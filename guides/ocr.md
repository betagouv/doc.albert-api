---
icon: glasses
---
# OCR - Reconnaissance Optique de Caractères

L'**OCR** (*Optical Character Recognition*, reconnaissance optique de caractères) permet d'extraire le texte contenu dans une image ou un document scanné (PDF, photo, capture d'écran, etc.). C'est ce qui permet de rendre exploitables des contenus qui n'existent pas nativement sous forme de texte numérique : archives papier scannées, formulaires, factures, courriers reçus par mail sous forme d'image... Une fois le texte extrait, vous pouvez le rechercher, l'indexer, l'analyser ou l'envoyer à un LLM.

## Les deux façons de faire de l'OCR avec Albert API

Albert API propose deux façons de faire de l'OCR :

1. **`/v1/chat/completions`** avec un LLM multimodal fine-tuné pour l'OCR, comme **`openweight-ocr`** : le modèle "lit" l'image envoyée dans le message et retourne le texte extrait (éventuellement accompagné d'un raisonnement).
2. **`/v1/ocr`** avec **`mistral-ocr-2512`** : cet endpoint reproduit l'[endpoint `/v1/ocr` de l'API SaaS de Mistral AI](https://mistral.ai/news/mistral-ocr/) et donne accès à la pipeline complète d'OCR de Mistral AI, que nous hébergeons nous-mêmes.

{% hint style="warning" %}
⚠️ **`mistral-ocr-2512`** n'est **pas accessible à l'ensemble des utilisateurs**. Si vous n'y avez pas accès, utilisez `/v1/chat/completions` avec **`openweight-ocr`**, qui couvre la grande majorité des besoins d'OCR et est disponible par défaut.
{% endhint %}

{% hint style="info" %}
**Quelle méthode choisir ?**

Par défaut, utilisez `/v1/chat/completions` avec `openweight-ocr` : c'est le modèle que nous mettons à disposition de tous les utilisateurs, aussi bien pour extraire du texte que pour combiner cette extraction avec du raisonnement (résumé, question-réponse, extraction de champs précis...) en une seule requête.

Si vous avez accès à `mistral-ocr-2512` et que vous avez besoin de la pipeline d'OCR complète de Mistral AI (extraction fidèle et structurée du texte d'un document entier, annotations, etc.), utilisez `/v1/ocr`, notamment pour l'intégrer ensuite dans un pipeline de [RAG](rag.md).
{% endhint %}

Pour connaître les modèles compatibles avec ces endpoints, voir [notre documentation dédiées aux modèles disponibles](../modeles/available-models.md).

## Méthode 1 (par défaut) : OCR avec l'endpoint `/v1/chat/completions`

Cette méthode s'appuie sur un LLM multimodal capable d'analyser des images (voir la section [Multimodal](chat-completions.md#multimodal-image-texte-via-chat-completions) du guide Chat Completions). Albert API propose **`openweight-ocr`**, un modèle multimodal fine-tuné spécifiquement pour l'OCR, disponible pour tous les utilisateurs par défaut.

Cette méthode est adaptée si vous voulez, en une seule requête, extraire le texte d'un document **et**, si besoin, en tirer une information (résumé, réponse à une question, etc.).

**Exemple : extraire le texte d'une image distante.**

{% tabs %}
{% tab title="curl" %}
```bash
IMG_BASE64=$(curl -sS "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg" | base64 -w 0)

curl -sS "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"openweight-ocr\",
    \"messages\": [
      {
        \"role\": \"user\",
        \"content\": [
          {\"type\": \"image_url\", \"image_url\": {\"url\": \"data:image/jpeg;base64,${IMG_BASE64}\"}}
        ]
      }
    ],
    \"max_tokens\": 4096,
    \"temperature\": 0.2,
    \"top_p\": 0.9
  }"
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os

import requests
from openai import OpenAI

image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg"
image_base64 = base64.b64encode(requests.get(image_url).content).decode("utf-8")

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

response = client.chat.completions.create(
    model="openweight-ocr",
    messages=[
        {
            "role": "user",
            "content": [{"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image_base64}"}}],
        }
    ],
    max_tokens=4096,
    temperature=0.2,
    top_p=0.9,
)
print(response.choices[0].message.content)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const imageUrl = "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg";
const imageBuffer = await (await fetch(imageUrl)).arrayBuffer();
const imageBase64 = Buffer.from(imageBuffer).toString("base64");

const response = await client.chat.completions.create({
  model: "openweight-ocr",
  messages: [
    {
      role: "user",
      content: [{ type: "image_url", image_url: { url: `data:image/jpeg;base64,${imageBase64}` } }],
    },
  ],
  max_tokens: 4096,
  temperature: 0.2,
  top_p: 0.9,
});

console.log(response.choices[0].message.content);
```
{% endtab %}
{% endtabs %}

{% hint style="tip" %}
**OCR d'un PDF**

Si votre document est un PDF, convertissez d'abord chaque page à traiter en image avant de l'envoyer à `/v1/chat/completions`. En Python, vous pouvez utiliser `pypdfium2` :

```python
import base64
import io

import pypdfium2 as pdfium
import requests

pdf_data = requests.get("https://arxiv.org/pdf/2412.13663").content
pdf = pdfium.PdfDocument(pdf_data)
page = pdf[0]

# Rendu à 200 DPI (facteur d'échelle = 200/72 ≈ 2.77)
pil_image = page.render(scale=2.77).to_pil()

buffer = io.BytesIO()
pil_image.save(buffer, format="PNG")
image_base64 = base64.b64encode(buffer.getvalue()).decode("utf-8")
```

Conseils :
* Rendez vos PDF à 200 DPI, avec une dimension la plus longue ciblée autour de 1540px.
* Conservez le ratio d'aspect de la page d'origine pour ne pas déformer la géométrie du texte.
{% endhint %}

## Méthode 2 (accès restreint) : OCR avec l'endpoint `/v1/ocr`

{% hint style="warning" %}
⚠️ Cette méthode repose sur **`mistral-ocr-2512`**, modèle **non accessible à l'ensemble des utilisateurs**. Si vous n'y avez pas accès, utilisez plutôt la [Méthode 1](#méthode-1-par-défaut-ocr-avec-lendpoint-v1chatcompletions) avec `openweight-ocr`.
{% endhint %}

L'endpoint **`POST /v1/ocr`** reproduit l'endpoint `/v1/ocr` de l'API SaaS de Mistral AI et donne accès à la pipeline complète d'OCR de Mistral AI, hébergée par nos soins (offre Mistral Entreprise on-premise). Il prend en entrée un **document** ou une **image** référencé(e) dans le corps de la requête JSON et retourne, pour chaque page, le texte extrait (au format markdown) ainsi que, selon la configuration, des **annotations**.

Pour le détail complet des clés et schémas, voir la [page de l'endpoint OCR](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/ocr).

### Modèle disponible

Le seul modèle disponible sur cet endpoint est **`mistral-ocr-2512`**, de type `image-to-text` (voir le guide [Types de modèles](../modeles/model-types.md)). Vous pouvez vérifier votre accès en récupérant la liste des modèles disponibles sur votre instance avec `GET /v1/models` (voir le guide [Modèles (liste)](models.md)).

### Paramètres principaux

* **`model`** (requis) — `mistral-ocr-2512`.
* **`document`** (requis) — objet décrivant la source à traiter :
  * `{"type": "document_url", "document_url": "https://..."}` pour un PDF/document (URL classique ou data URL `data:application/pdf;base64,...`) ;
  * `{"type": "image_url", "image_url": "https://..."}` pour une image (URL classique ou data URL `data:image/...;base64,...`).
* **`pages`** — liste d'indices de pages à traiter.
* **`include_image_base64`** — inclut les images des pages en base64 dans la réponse (utile pour archiver ou afficher les pages traitées).
* **`document_annotation_format`** / **`bbox_annotation_format`** — format de sortie des annotations (texte libre, JSON, JSON Schema), sur le même principe que le `response_format` de `/v1/chat/completions`.

{% hint style="warning" %}
⚠️ Les indices de page du paramètre `pages` sont **à partir de 0** (`0` = première page).
{% endhint %}

### Exemple : OCR d'un PDF référencé par URL

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral-ocr-2512",
    "document": {
      "type": "document_url",
      "document_url": "https://www.princexml.com/samples/magic6/magic.pdf"
    },
    "include_image_base64": true
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os
import requests

response = requests.post(
    url="https://albert.api.etalab.gouv.fr/v1/ocr",
    headers={"Authorization": f"Bearer {os.environ['ALBERT_API_KEY']}"},
    json={
        "model": "mistral-ocr-2512",
        "document": {
            "type": "document_url",
            "document_url": "https://www.princexml.com/samples/magic6/magic.pdf",
        },
        "include_image_base64": True,
    },
)
response.raise_for_status()
data = response.json()

print("Pages traitées :", len(data["pages"]))
print(data["pages"][0]["markdown"])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const response = await fetch("https://albert.api.etalab.gouv.fr/v1/ocr", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "mistral-ocr-2512",
    document: {
      type: "document_url",
      document_url: "https://www.princexml.com/samples/magic6/magic.pdf",
    },
    include_image_base64: true,
  }),
});

if (!response.ok) throw new Error(await response.text());
const data = await response.json();

console.log("Pages traitées :", data.pages.length);
console.log(data.pages[0].markdown);
```
{% endtab %}
{% endtabs %}

### Variante : OCR d'un PDF encodé en base64 (data URL)

Cette variante est utile lorsque le document n'est pas accessible par une URL publique (fichier local, par exemple).

{% tabs %}
{% tab title="curl" %}
```bash
PDF_BASE64=$(curl -sS "https://www.princexml.com/samples/textbook/somatosensory.pdf" | base64 -w 0)

curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"mistral-ocr-2512\",
    \"document\": {
      \"type\": \"document_url\",
      \"document_url\": \"data:application/pdf;base64,${PDF_BASE64}\"
    },
    \"include_image_base64\": true
  }"
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os
import requests

pdf_url = "https://www.princexml.com/samples/textbook/somatosensory.pdf"
pdf_base64 = base64.b64encode(requests.get(pdf_url).content).decode("utf-8")

response = requests.post(
    url="https://albert.api.etalab.gouv.fr/v1/ocr",
    headers={"Authorization": f"Bearer {os.environ['ALBERT_API_KEY']}"},
    json={
        "model": "mistral-ocr-2512",
        "document": {
            "type": "document_url",
            "document_url": f"data:application/pdf;base64,{pdf_base64}",
        },
        "include_image_base64": True,
    },
)
response.raise_for_status()
data = response.json()

print(data["pages"][0]["markdown"][:200])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const pdfBuffer = await (await fetch("https://www.princexml.com/samples/textbook/somatosensory.pdf")).arrayBuffer();
const pdfBase64 = Buffer.from(pdfBuffer).toString("base64");

const response = await fetch("https://albert.api.etalab.gouv.fr/v1/ocr", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "mistral-ocr-2512",
    document: {
      type: "document_url",
      document_url: `data:application/pdf;base64,${pdfBase64}`,
    },
    include_image_base64: true,
  }),
});

if (!response.ok) throw new Error(await response.text());
const data = await response.json();

console.log(data.pages[0].markdown.slice(0, 200));
```
{% endtab %}
{% endtabs %}

### Exemple : OCR d'une image référencée par URL

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral-ocr-2512",
    "document": {
      "type": "image_url",
      "image_url": "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg"
    },
    "include_image_base64": true
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os
import requests

image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg"

response = requests.post(
    url="https://albert.api.etalab.gouv.fr/v1/ocr",
    headers={"Authorization": f"Bearer {os.environ['ALBERT_API_KEY']}"},
    json={
        "model": "mistral-ocr-2512",
        "document": {"type": "image_url", "image_url": image_url},
        "include_image_base64": True,
    },
)
response.raise_for_status()
data = response.json()

print(data["pages"][0]["markdown"][:200])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const response = await fetch("https://albert.api.etalab.gouv.fr/v1/ocr", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "mistral-ocr-2512",
    document: {
      type: "image_url",
      image_url: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg",
    },
    include_image_base64: true,
  }),
});

if (!response.ok) throw new Error(await response.text());
const data = await response.json();

console.log(data.pages[0].markdown.slice(0, 200));
```
{% endtab %}
{% endtabs %}
