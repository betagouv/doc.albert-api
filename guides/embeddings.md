---
icon: "draw-polygon"
---
# Embeddings

Les **embeddings** sont des vecteurs numériques qui représentent le sens d'un contenu. Ils servent notamment à faire de la recherche sémantique, du clustering, de la déduplication, du RAG ou de la classification. Pour en savoir plus sur le RAG, voir la documentation dédiée [ici](rag.md).

Albert API expose l'endpoint **`POST /v1/embeddings`**, compatible avec le format OpenAI qui permet de transformer du texte ou des images en vecteurs.

{% hint style="info" %}
**Quel modèle choisir ?**

Albert API mets à disposition différents modèles d'embeddings selon la nature du contenu que vous souhaitez vectoriser.
Si votre corpus et vos requêtes sont **uniquement textuels**, utilisez le `openweight-embeddings`.

Si vous voulez représenter des contenus textuels et visuels dans le même espace vectoriel, alors deux méthodes sont possibles avec des modèles différents (voir [Vectorisation d'images](#vectorisation-d-images))

En savoir plus sur ces modèles d'embeddings sur [la page des modèles disponibles](../modeles/available-models.md).
{% endhint %}


{% hint style="warning" %}
⚠️ Il faut toujours vectoriser avec **le même modèle** ce que vous souhaitez comparer. N'indexez pas vos documents texte avec `openweight-embeddings` si vos images sont vectorisées avec `qwen3-vl-embedding-8b` : les vecteurs ne vivent pas dans le même espace et ne sont pas comparables !
{% endhint %}

## Vectorisation de texte

L'endpoint **`POST /v1/embeddings`** calcule un vecteur pour une chaîne unique ou pour une liste de chaînes. La réponse suit le format OpenAI classique : vous recevez un objet par entrée dans `data[]`, avec son `embedding` et son `index`, ainsi qu'un objet `usage` pour le suivi de consommation.

Pour le détail complet des clés et schémas, voir la [page de l'endpoint Embeddings](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/embeddings).

### Modèles recommandés

Les modèles qui supportent la vectorisation de texte sont ceux de type **`text-embeddings-inference`**. Vous pouvez les lister avec `GET /v1/models` (en savoir plus [ici](models.md)).

**Exemple de requête :**

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/embeddings" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openweight-embeddings",
    "input": [
      "Le chat est un animal domestique.",
      "La France a pour capitale Paris."
    ]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

response = client.embeddings.create(
    model="openweight-embeddings",
    input=[
        "Le chat est un animal domestique.",
        "La France a pour capitale Paris.",
    ],
    encoding_format="float",
)

for item in response.data:
    print("index=", item.index, "dim=", len(item.embedding))
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const response = await client.embeddings.create({
  model: "openweight-embeddings",
  input: [
    "Le chat est un animal domestique.",
    "La France a pour capitale Paris.",
  ],
  encoding_format: "float",
});

for (const item of response.data) {
  console.log("index=", item.index, "dim=", item.embedding.length);
}
```
{% endtab %}
{% endtabs %}

## Vectorisation d'images

Si votre objectif est de faire de la recherche ou du rapprochement entre des textes et des images ou entre des images entre elles, deux approches sont possibles avec Albert API :

- utiliser un modèle d'embeddings multimodal, c'est-à-dire un modèle qui peut vectoriser à la fois du texte et des images ;
- utiliser un LLM multimodal pour transformer les images en texte, puis vectoriser ce texte avec un modèle d'embeddings unimodal (autrement dit, un modèle qui ne peut vectoriser que du texte).

{% hint style="info" %}
**Comment choisir entre les deux approches ?**

Le bon choix ne peut pas être décidé uniquement "sur le papier" : il doit être fait après une étude sur un échantillon représentatif de vos documents, en observant les corrélations de similarité cosinus obtenues sur vos cas réels. Selon la typologie des documents (scans, formulaires, slides, schémas, captures, documents très textuels ou très visuels), la méthode la plus pertinente peut varier.

Constituez un petit jeu d'évaluation à partir d'extraits de documents représentatifs de votre corpus, puis comparez :

* la proximité cosinus entre les éléments que vous jugez similaires ;
* la séparation avec les éléments non pertinents ;
* la qualité du classement obtenu pour vos requêtes réelles.

Ne retenez la méthode qu'après cette vérification empirique.
{% endhint %}

### Méthode 1 : vectorisation multimodale

Cette approche consiste à utiliser modèle d'embeddings multimodal, c'est-à-dire un modèle qui peut vectoriser à la fois du texte et des images. Sur Albert API, ce rôle est assuré par **`qwen3-vl-embedding-8b`**.

{% hint style="info" %}
Pour en savoir plus sur ce modèle, voir [la documentation du modèle](../modeles/available-models.md#qwen3-vl-embedding-8b).
{% endhint %}

Pour le **texte**, vous pouvez continuer à utiliser `/v1/embeddings` avec un champ `input` standard. Pour les **images**, Albert API expose `/v1/embeddings` au format OpenAI classique, mais la gateway supporte aussi un champ vLLM **`messages`** pour certains usages multimodaux.

Cette variante image n'est pas documentée comme un usage standard OpenAI, car elle est **spécifique au modèle `qwen3-vl-embedding-8b`**.

**Particularités :**
* le corps de requête utilise **`messages`** au lieu de `input` ;
* la réponse reste un objet d'embeddings au format habituel ;
* **une seule image par requête** est supportée ;
* selon votre client, il peut être plus simple d'utiliser une requête HTTP directe plutôt que le helper `embeddings.create()`.

**Exemple de requête :**

{% tabs %}
{% tab title="curl" %}
```bash
IMAGE_BASE64=$(curl -sS "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg" | base64 | tr -d '\n')

curl -sS "https://albert.api.etalab.gouv.fr/v1/embeddings" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3-vl-embedding-8b",
    "messages": [
      {"role": "system", "content": "Represent the user's input."},
      {
        "role": "user",
        "content": [
          {
            "type": "image_url",
            "image_url": {"url": "data:image/jpeg;base64,${IMAGE_BASE64}"}
          }
        ]
      },
      {"role": "assistant", "content": ""}
    ],
    "encoding_format": "float",
    "continue_final_message": true,
    "add_special_tokens": true
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os

import requests

image_url = "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg"
image_base64 = base64.b64encode(requests.get(image_url).content).decode("utf-8")

response = requests.post(
    url="https://albert.api.etalab.gouv.fr/v1/embeddings",
    headers={"Authorization": f"Bearer {os.environ['ALBERT_API_KEY']}"},
    json={
        "model": "qwen3-vl-embedding-8b",
        "messages": [
            {"role": "system", "content": "Represent the user's input."},
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {"url": f"data:image/jpeg;base64,{image_base64}"},
                    }
                ],
            },
            {"role": "assistant", "content": ""},
        ],
        "encoding_format": "float",
        "continue_final_message": True,
        "add_special_tokens": True,
    },
)
response.raise_for_status()
data = response.json()

print("dim=", len(data["data"][0]["embedding"]))
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const imageUrl = "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg";
const imageBuffer = await (await fetch(imageUrl)).arrayBuffer();
const imageBase64 = Buffer.from(imageBuffer).toString("base64");

const response = await fetch("https://albert.api.etalab.gouv.fr/v1/embeddings", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "qwen3-vl-embedding-8b",
    messages: [
      { role: "system", content: "Represent the user's input." },
      {
        role: "user",
        content: [
          {
            type: "image_url",
            image_url: { url: `data:image/jpeg;base64,${imageBase64}` },
          },
        ],
      },
      { role: "assistant", content: "" },
    ],
    encoding_format: "float",
    continue_final_message: true,
    add_special_tokens: true,
  }),
});

if (!response.ok) throw new Error(await response.text());
const data = await response.json();

console.log("dim=", data.data[0].embedding.length);
```
{% endtab %}
{% endtabs %}

### Méthode 2 : vectorisation avec un LLM multimodal

L'autre approche consiste à transformer d'abord l'image en texte à l'aide d'un LLM multimodal, puis à vectoriser ce texte avec un modèle d'embeddings unimodal (autrement dit, un modèle qui ne peut vectoriser que du texte).

La chaîne de traitement est alors la suivante :

1. un LLM multimodal décrit l'image ou en extrait les informations utiles (ex. `openweight-medium`);
2. le texte produit est ensuite vectorisé avec un modèle d'embeddings texte (ex. `openweight-embeddings`);

{% hint style="info" %}
Pour en savoir plus sur les LLM multimodaux disponibles sur Albert API, voir [la documentation des modèles disponibles](../modeles/available-models.md).
{% endhint %}

**Exemple de requête :**

{% tabs %}
{% tab title="curl" %}
```bash
IMAGE_BASE64=$(curl -sS "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg" | base64 | tr -d '\n')

curl -sS "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openweight-medium",
    "messages": [
      {
        "role": "user",
        "content": [
          {"role": "system", "content": "Décris l'image en quelques phrases."},
          {
            "type": "image_url",
            "image_url": {"url": "data:image/jpeg;base64,${IMAGE_BASE64}"},
          },
        ],
      },
    ],
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os

import requests

image_url = "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg"
image_base64 = base64.b64encode(requests.get(image_url).content).decode("utf-8")
```
```python
import base64
import os

import requests

image_url = "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg"
image_base64 = base64.b64encode(requests.get(image_url).content).decode("utf-8")

response = requests.post(
    url="https://albert.api.etalab.gouv.fr/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['ALBERT_API_KEY']}"},
    json={
        "model": "openweight-medium",
        "messages": [
            {"role": "system", "content": "Décris l'image en quelques phrases."},
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {"url": f"data:image/jpeg;base64,{image_base64}"},
                    }
                ],
            },
        ],
    },
)
response.raise_for_status()
data = response.json()

print(data["choices"][0]["message"]["content"])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const imageUrl = "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg";
const imageBuffer = await (await fetch(imageUrl)).arrayBuffer();
const imageBase64 = Buffer.from(imageBuffer).toString("base64");
```
```javascript
const imageUrl = "https://raw.githubusercontent.com/AtharvaTaras/Cat-Images-Dataset/refs/heads/master/cute%20cat/Image_102.jpg";
const imageBuffer = await (await fetch(imageUrl)).arrayBuffer();
const imageBase64 = Buffer.from(imageBuffer).toString("base64");

const response = await fetch("https://albert.api.etalab.gouv.fr/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "openweight-medium",
    messages: [
      { role: "system", content: "Décris l'image en quelques phrases." },
      {
        role: "user",
        content: [
          { type: "image_url", image_url: { url: `data:image/jpeg;base64,${imageBase64}` } },
        ],
      },
    ],
  }),
});

if (!response.ok) throw new Error(await response.text());
const data = await response.json();

console.log(data.choices[0].message.content);
```
{% endtab %}
{% endtabs %}

Par exemple, vous pouvez utiliser un LLM multimodal tel que **`openweight-medium`** pour produire une description structurée de l'image, puis envoyer cette description vers un modèle d'embeddings textuels.

Cette approche peut être particulièrement utile lorsque la pertinence dépend davantage du contenu textuel ou sémantique explicitable de l'image que de ses caractéristiques visuelles brutes.

En contrepartie, elle ajoute une étape de transformation : la qualité finale dépend donc aussi de la qualité de la description produite par le LLM. Là encore, le choix doit être validé en mesurant la similarité cosinus sur des extraits représentatifs de votre base documentaire.