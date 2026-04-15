# Outil de recherche RAG intégré (SearchTool)

Albert API expose un **outil de recherche natif** utilisable directement dans la liste **`tools`** de **`POST /v1/chat/completions`**. L’outil interroge des **chunks** indexés via vos **collections** et **documents**, puis le modèle s’en sert pour **ancrer** sa réponse dans votre corpus.

{% hint style="warning" %}
⚠️ `SearchTool` n’est pas une fonctionnalité standard “OpenAI tools”. Les clients génériques ne savent pas l’interpréter : ajoutez bien `type: "search"` dans `tools` et utilisez les paramètres attendus par Albert.
{% endhint %}

## Schéma usuel dans `tools`

Ajoutez un objet correspondant à **`SearchTool`** (voir la [page de l’endpoint Search](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/search)).

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

### Utiliser une collection publique

Albert API met à disposition des **collections publiques** prêtes à l'emploi. Il suffit de récupérer l'identifiant de la collection souhaitée, puis de l'utiliser directement dans `SearchTool`.

Ces collections sont les suivantes :

| Nom de la collection | Contenu |
| --- | --- |
| `mediatech-legifrance` | Législation et réglementation nationale consolidée : codes officiels (73 en vigueur), lois, décrets, ordonnances depuis 1945, arrêtés sélectionnés depuis 1990. Source : legifrance.gouv.fr |
| `mediatech-fiches-travail-emploi` | Fiches pédagogiques du Ministère du travail sur le droit du travail. Source : travail-emploi.gouv.fr |
| `mediatech-fiches-service-public` | Fiches pratiques à destination des usagers (droits, démarches administratives, formulaires). Source : service-public.gouv.fr |
| `mediatech-annuaire-services-publics-locaux` | Annuaire de plus de 86 000 guichets publics locaux (mairies, organismes sociaux, services de l'État) avec coordonnées et horaires. Source : data.gouv.fr |
| `mediatech-annuaire-services-publics-nationaux` | Référentiel de l'organisation administrative de l'État : ~6 000 organismes, missions, hiérarchie, coordonnées et responsables. Source : data.gouv.fr |
| `mediatech-dossiers-legislatifs` | Dossiers législatifs : lois depuis juin 2002, ordonnances depuis 2002, projets et propositions de loi en préparation. Source : data.gouv.fr |
| `mediatech-decisions-conseil-constitutionnel` | Décisions du Conseil constitutionnel depuis 1958 (DC, QPC, contentieux électoral, etc.). Source : data.gouv.fr |
| `mediatech-decisions-cnil` | Délibérations de la CNIL depuis 1979. Source : echanges.dila.gouv.fr |

L'exemple ci-dessous interroge la collection **`mediatech-decisions-cnil`** :

```python
import requests

BASE_URL = "https://albert.api.etalab.gouv.fr/v1"
API_KEY = "***"
headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

# 1. Récupérer l'ID de la collection publique voulue
resp = requests.get(f"{BASE_URL}/collections?name=mediatech-decisions-cnil", headers=headers)
collection_id = resp.json()["data"][0]["id"]

# 2. Chat avec RAG via SearchTool
prompt = "Quelles sont les durées de conservation des données dans les fichiers de police ? Précise les sources des délibérations CNIL utilisées"
response = requests.post(
    f"{BASE_URL}/chat/completions",
    headers=headers,
    json={
        "model": "openweight-large",
        "messages": [
            {"role": "user", "content": prompt},
        ],
        "tools": [
            {
                "type": "search",
                "collection_ids": [collection_id],
                "method": "hybrid",
                "limit": 6,
            }
        ],
        "tool_choice": "auto",
    },
)
print(response.json()["choices"][0]["message"]["content"])
```

Pour lister toutes les collections publiques disponibles et obtenir leur description, appelez `GET /v1/collections?visibility=public`.

## Parcours type : RAG manuel

Il est aussi possible de faire du RAG manuel, où la recherche du contexte est une étape dédiée, non intégrée à l'appel `/chat/completions`.
Pour ce faire, `SearchTool` n'est plus utilisé, et les étapes sont :

1. **Recherche** — `POST /v1/search` avec un JSON aligné sur `CreateSearch` (`query`, `collection_ids`, `method`, `limit`, `rff_k` si `hybrid`, etc.) ;
2. **Prompt** — concaténation des `chunk.content` pour construire un prompt incluant le contexte;
3. **Chat** — `POST /v1/chat/completions` avec un modèle **`text-generation`**.

### Exemple avec `SearchTool`

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
        "method": "hybrid",
        "limit": 5
      }
    ],
    "tool_choice": "auto"
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
from openai import OpenAI
import os

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

resp = client.chat.completions.create(
    model="REMPLACER_PAR_MODELE_TEXT_GENERATION",
    messages=[{"role": "user", "content": "Résume les points clés du document."}],
    tools=[{"type": "search", "collection_ids": [123], "method": "hybrid", "limit": 5}],
    tool_choice="auto",
)
print(resp.choices[0].message.content)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const resp = await client.chat.completions.create({
  model: "REMPLACER_PAR_MODELE_TEXT_GENERATION",
  messages: [{ role: "user", content: "Résume les points clés du document." }],
  tools: [{ type: "search", collection_ids: [123], method: "hybrid", limit: 5 }],
  tool_choice: "auto",
});

console.log(resp.choices[0].message.content);
```
{% endtab %}
{% endtabs %}

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
⚠️ Les anciens exemples utilisant `collections` / `prompt` (au lieu de `collection_ids` / `query`), ou un chemin `/search` sans le préfixe `/v1`, sont **obsolètes**. Alignez votre corps JSON sur la [page de l’endpoint Search](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/search).
{% endhint %}

{% hint style="warning" %}
⚠️ À vérifier — Détails exacts de `metadata_filters` (filtres simples vs composés) et comportement lorsque `collection_ids` / `document_ids` sont vides : consultez les schémas (voir pages **Search** et **Chat**).
{% endhint %}

## Ancienne intégration chat (`search` / `search_args`)

Les champs **`search`** et **`search_args`** au niveau racine du corps de `POST /v1/chat/completions` sont **dépréciés** au profit du pipeline explicite (ingestion collections + `SearchTool` dans `tools`).
