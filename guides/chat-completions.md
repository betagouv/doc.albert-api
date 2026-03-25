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

## Unstreamed chat

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

## Streaming (SSE)

Pour activer le streaming, passez **`stream=True`**. Le SDK OpenAI renvoie une suite de chunks (deltas) jusqu’à la fin du flux.

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

## Compatibilité OpenAI

En dehors des extensions propres à Albert (notamment `SearchTool` et certains endpoints métiers), les conventions générales restent proches des clients OpenAI : mêmes noms de champs, mêmes rôles de `messages`, mêmes conventions SSE.

{% hint style="warning" %}
⚠️ À vérifier — Champs additionnels exacts et valeurs par défaut : se référer au schéma `CreateChatCompletion` dans la [page de l’endpoint Chat](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/chat).
{% endhint %}

## Champs de recherche dépréciés

Le corps de requête peut encore contenir **`search`** et **`search_args`** au niveau racine. Ces champs sont **dépréciés** au profit du pipeline explicite :

1. collections + documents (ingestion),
2. `SearchTool` dans `tools`,
3. ensuite génération avec le chat.
