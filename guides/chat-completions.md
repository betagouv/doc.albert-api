# Chat completions

L’endpoint **`POST /v1/chat/completions`** est le point d’entrée principal pour la génération de texte. Le corps de requête et la réponse s’inspirent du modèle **OpenAI Chat Completions** : `messages`, paramètres de décodage, `response_format`, streaming et outils (`tools` / `tool_calls`).

## Quel type de modèle ?

Pour **`POST /v1/chat/completions`**, vous devez utiliser un modèle de type **`text-generation`** (voir [Types de modèles](../modeles/model-types.md)).

{% hint style="warning" %}
⚠️ Les anciens tutoriels mélangaient parfois des modèles `image-text-to-text` / `image-to-text` avec le chat texte. En production, utilisez plutôt ces modèles pour l’OCR ou les endpoints adaptés, et gardez `text-generation` pour le chat standard.
{% endhint %}

## Avant de commencer

### 1) Définir votre clé API

```bash
export ALBERT_API_KEY="votre_jeton"
```

### 2) Choisir un modèle `text-generation`

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/models" \
  -H "Authorization: Bearer $ALBERT_API_KEY"
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

models = client.models.list().data
model = [m for m in models if m.type == "text-generation"][0].id
print("Modèle chat trouvé :", model)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const models = (await client.models.list()).data;
const model = models.find((m) => m.type === "text-generation")?.id;
console.log("Modèle chat trouvé :", model);
```
{% endtab %}
{% endtabs %}

## Unstreamed chat

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_TEXT_GENERATION",
    "messages": [
      {"role": "system", "content": "Tu réponds en français, de façon concise."},
      {"role": "user", "content": "Explique ce qu’est une API compatible OpenAI en deux phrases."}
    ],
    "stream": false
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
resp = client.chat.completions.create(
    model=model,
    messages=[
        {"role": "system", "content": "Tu réponds en français, de façon concise."},
        {"role": "user", "content": "Explique ce qu’est une API compatible OpenAI en deux phrases."},
    ],
    stream=False,
)

print(resp.choices[0].message.content)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const resp = await client.chat.completions.create({
  model,
  messages: [
    { role: "system", content: "Tu réponds en français, de façon concise." },
    { role: "user", content: "Explique ce qu’est une API compatible OpenAI en deux phrases." },
  ],
  stream: false,
});

console.log(resp.choices[0].message.content);
```
{% endtab %}
{% endtabs %}

## Streaming (SSE)

Pour activer le streaming, passez **`stream=True`**. Le SDK OpenAI renvoie une suite de chunks (deltas) jusqu’à la fin du flux.

{% tabs %}
{% tab title="curl" %}
```bash
curl -N "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_TEXT_GENERATION",
    "messages": [{"role": "user", "content": "Raconte une phrase sur la météo."}],
    "stream": true
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
stream = client.chat.completions.create(
    model=model,
    messages=[{"role": "user", "content": "Raconte une phrase sur la météo."}],
    stream=True,
)

for chunk in stream:
    delta = chunk.choices[0].delta
    if delta.content:
        print(delta.content, end="", flush=True)
print()
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const stream = await client.chat.completions.create({
  model,
  messages: [{ role: "user", content: "Raconte une phrase sur la météo." }],
  stream: true,
});

for await (const chunk of stream) {
  const delta = chunk.choices?.[0]?.delta;
  if (delta?.content) process.stdout.write(delta.content);
}
process.stdout.write("\n");
```
{% endtab %}
{% endtabs %}

Voir aussi le guide [Streaming](streaming.md).

## Format de réponse (`response_format`)

* **`{"type": "json_object"}`** : JSON valide (mode “JSON”). Vous devez aussi **demander explicitement du JSON** côté prompt (`system` / `user`) pour éviter des sorties longues inutiles.
* **`{"type": "json_schema", "json_schema": { ... }}`** : sorties structurées guidées par un schéma JSON (approche proche des Structured Outputs).

## Outils et `tool_calls`

Le champ **`tools`** permet d’activer :

* des fonctions au format JSON Schema ;
* des outils intégrés côté Albert, dont le **RAG natif** via `SearchTool`.

Le mécanisme `tools` / `tool_choice` est décrit dans [Function calling](function-calling.md) et l’outil intégré dans [Outil de recherche RAG intégré](rag-search-tool.md).

{% hint style="warning" %}
⚠️ `SearchTool` n’est pas une fonctionnalité “OpenAI standard”. Utilisez bien `type: "search"` dans `tools` et des paramètres conformes à la spec Albert.
{% endhint %}

Exemple minimal d’activation de `SearchTool` :

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_TEXT_GENERATION",
    "messages": [{"role": "user", "content": "Résume les points clés du document."}],
    "tools": [
      {
        "type": "search",
        "collection_ids": [123],
        "method": "semantic",
        "limit": 5
      }
    ],
    "tool_choice": "auto"
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
tools = [
    {
        "type": "search",
        "collection_ids": [123],
        "method": "semantic",
        "limit": 5,
    }
]

resp = client.chat.completions.create(
    model=model,
    messages=[{"role": "user", "content": "Résume les points clés du document."}],
    tools=tools,
    tool_choice="auto",
)
print(resp.choices[0].message.content)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const tools = [
  {
    type: "search",
    collection_ids: [123],
    method: "semantic",
    limit: 5,
  },
];

const resp = await client.chat.completions.create({
  model,
  messages: [{ role: "user", content: "Résume les points clés du document." }],
  tools,
  tool_choice: "auto",
});

console.log(resp.choices[0].message.content);
```
{% endtab %}
{% endtabs %}

## Compatibilité OpenAI

En dehors des extensions propres à Albert (notamment `SearchTool` et certains endpoints métiers), les conventions générales restent proches des clients OpenAI : mêmes noms de champs, mêmes rôles de `messages`, mêmes conventions SSE.

## Multimodal (image + texte) via `chat/completions`

Albert API peut accepter des entrées multimodales via `POST /v1/chat/completions` en passant, dans un message `user`, un `content` **structuré** (liste) combinant :

* `{"type": "text", "text": "..."}`
* `{"type": "image_url", "image_url": {"url": "data:image/png;base64,..."}}`

Choisissez un modèle dont le `type` est généralement **`image-text-to-text`** (ou `image-to-text` selon l’instance).

{% tabs %}
{% tab title="curl" %}
```bash
IMG_BASE64=$(base64 -w 0 "archi_mistral.png")

curl -sS "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"REMPLACER_PAR_MODELE_MULTIMODAL\",
    \"messages\": [
      {
        \"role\": \"user\",
        \"content\": [
          {\"type\": \"text\", \"text\": \"Décris l'image et lis le texte visible (si présent). Réponds en français.\"},
          {\"type\": \"image_url\", \"image_url\": {\"url\": \"data:image/png;base64,${IMG_BASE64}\"}}
        ]
      }
    ],
    \"max_tokens\": 500
  }"
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

with open("archi_mistral.png", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode("utf-8")

r = client.chat.completions.create(
    model="REMPLACER_PAR_MODELE_MULTIMODAL",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Décris l'image et lis le texte visible (si présent). Réponds en français."},
                {"type": "image_url", "image_url": {"url": f"data:image/png;base64,{img_b64}"}},
            ],
        }
    ],
    max_tokens=500,
)
print(r.choices[0].message.content)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import fs from "node:fs";
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const imgB64 = fs.readFileSync("archi_mistral.png").toString("base64");

const r = await client.chat.completions.create({
  model: "REMPLACER_PAR_MODELE_MULTIMODAL",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Décris l'image et lis le texte visible (si présent). Réponds en français." },
        { type: "image_url", image_url: { url: `data:image/png;base64,${imgB64}` } },
      ],
    },
  ],
  max_tokens: 500,
});

console.log(r.choices[0].message.content);
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
⚠️ À vérifier — Champs additionnels exacts et valeurs par défaut : se référer au schéma `CreateChatCompletion` dans la page de l’endpoint Chat :
[page de l’endpoint Chat](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/chat)
{% endhint %}

## Champs de recherche dépréciés

Le corps de requête peut encore contenir **`search`** et **`search_args`** au niveau racine. Ces champs sont **dépréciés** au profit du pipeline explicite :

1. collections + documents (ingestion),
2. `SearchTool` dans `tools`,
3. ensuite génération avec le chat.
