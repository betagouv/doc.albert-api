# Outil de recherche RAG intégré (SearchTool)

Albert API expose un **outil de recherche natif** utilisable directement dans la liste **`tools`** de **`POST /v1/chat/completions`**. L’outil interroge des **chunks** indexés via vos **collections** et **documents**, puis le modèle s’en sert pour **ancrer** sa réponse dans votre corpus.

{% hint style="warning" %}
⚠️ `SearchTool` n’est pas une fonctionnalité standard “OpenAI tools”. Les clients génériques ne savent pas l’interpréter : ajoutez bien `type: "search"` dans `tools` et utilisez les paramètres attendus par Albert.
{% endhint %}

## Schéma usuel dans `tools`

Ajoutez un objet correspondant à **`SearchTool`** (voir la page de l’endpoint **Search** : https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/search).

| Champ              | Description |
| --- | --- |
| `type` | Doit valoir `"search"` pour déclencher l’outil intégré |
| `collection_ids` | Liste d’IDs de collections (entiers strictement positifs), jusqu’à 100 |
| `document_ids` | IDs optionnels de documents pour restreindre |
| `metadata_filters` | Filtres sur les métadonnées des chunks (`ComparisonFilter` / `CompoundFilter`) |
| `limit` | Nombre max de résultats (défaut 10, max 100) |
| `offset` | Décalage de pagination |
| `method` | `"semantic"`, `"lexical"` ou `"hybrid"` |
| `rff_k` | Paramètre RRF pour `hybrid` (souvent entre 10 et 100) |
| `score_threshold` | Seuil de score (0 à 1), utile pour `semantic` |

## Pipeline recommandé (RAG “intégré” au chat)

1. **Créer une collection** — `POST /v1/collections` (voir [Collections & documents](collections-documents.md)).
2. **Ingérer des documents** — `POST /v1/documents` en `multipart/form-data` (fichier + `collection_id` + options de chunking).
3. **Appeler le chat avec `SearchTool`** — ajoutez l’objet `type: "search"` dans `tools` et laissez le modèle invoquer l’outil (ou forcez via `tool_choice` si besoin).

En pratique, l’endpoint **`POST /v1/search`** permet aussi d’exécuter la recherche **sans** passer par le chat (debug, UIs, pipeline “manuel”).

## Parcours type : RAG manuel (recherche puis chat)

Sans `SearchTool`, vous pouvez enchaîner :

1. **Recherche** — `POST /v1/search` avec un corps JSON aligné sur `CreateSearch` (`query`, `collection_ids`, `method`, `limit`, `rff_k` si `hybrid`, etc.) ;
2. **Prompt** — concaténation des `chunk.content` renvoyés dans un prompt ;
3. **Chat** — `POST /v1/chat/completions` avec un modèle **`text-generation`**.

### Exemple Python : recherche puis chat

Cet exemple illustre le flux “manuel” avec `requests` pour la recherche et le SDK OpenAI pour le chat.

```python
import os
import requests
from openai import OpenAI

base_url = "https://albert.api.etalab.gouv.fr/v1"
api_key = os.environ["ALBERT_API_KEY"]

headers = {"Authorization": f"Bearer {api_key}"}

client = OpenAI(base_url=base_url, api_key=api_key)
```

#### 1) Créer une collection

```python
collection = requests.post(
    url=f"{base_url}/collections",
    json={"name": "tutorial", "visibility": "private"},
    headers=headers,
)
collection.raise_for_status()
collection_id = collection.json()["id"]
```

#### 2) Ingérer un document (multipart)

```python
pdf_url = "https://www.princexml.com/samples/magic6/magic.pdf"
pdf_bytes = requests.get(pdf_url).content

files = {"file": ("magic.pdf", pdf_bytes, "application/pdf")}

doc = requests.post(
    url=f"{base_url}/documents",
    headers=headers,
    files=files,
    data={"collection_id": collection_id},
)
doc.raise_for_status()
document_id = doc.json()["id"]
```

#### 3) Rechercher des chunks (`/v1/search`)

```python
search = requests.post(
    url=f"{base_url}/search",
    headers=headers,
    json={
        "query": "Quel est le sujet du document ?",
        "collection_ids": [collection_id],
        "method": "semantic",
        "limit": 5,
    },
)
search.raise_for_status()

results = search.json()["data"]
```

#### 4) Construire un prompt avec les passages

```python
chunks = []
for r in results:
    chunk = r["chunk"]
    chunks.append(chunk["content"])

context = "\n\n---\n\n".join(chunks)
prompt = (
    "Réponds uniquement en t'appuyant sur les extraits fournis.\n\n"
    f"Question : Quel est le sujet du document ?\n\n"
    f"Extraits :\n{context}"
)
```

#### 5) Réponse par chat (`/v1/chat/completions`)

```python
models = client.models.list().data
chat_model = [m for m in models if m.type == "text-generation"][0].id

resp = client.chat.completions.create(
    model=chat_model,
    messages=[{"role": "user", "content": prompt}],
    stream=False,
)

print(resp.choices[0].message.content)
```

{% hint style="warning" %}
⚠️ Les anciens exemples utilisant `collections` / `prompt` (au lieu de `collection_ids` / `query`), ou un chemin `/search` sans le préfixe `/v1`, sont **obsolètes**. Alignez votre corps JSON sur la page **Search** :
https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/search
{% endhint %}

{% hint style="warning" %}
⚠️ À vérifier — Détails exacts de `metadata_filters` (filtres simples vs composés) et comportement lorsque `collection_ids` / `document_ids` sont vides : consultez les schémas (voir pages **Search** et **Chat**).
{% endhint %}

## Ancienne intégration chat (`search` / `search_args`)

Les champs **`search`** et **`search_args`** au niveau racine du corps de `POST /v1/chat/completions` sont **dépréciés** au profit du pipeline explicite (ingestion collections + `SearchTool` dans `tools`).
