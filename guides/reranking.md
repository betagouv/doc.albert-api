# Reranking

L’endpoint **`POST /v1/rerank`** réordonne une liste de documents textuels selon leur pertinence par rapport à une **requête**, via un modèle de type **`text-classification`**.

## Requête (`CreateRerank`)

* **`query`** (requis) — la question ou requête utilisateur ;
* **`documents`** (requis) — tableau de chaînes à classer ;
* **`model`** (requis) — identifiant d’un modèle `text-classification` retourné par `GET /v1/models` ;
* **`top_n`** — nombre max de résultats à retourner.

## Intégration typique

1. **Recherche large** — récupérez des candidats avec `POST /v1/search` (ou un moteur de recherche externe) ;
2. **Rerank** — appelez `POST /v1/rerank` pour améliorer l’ordre ;
3. **Chat / UI** — utilisez les meilleurs résultats pour générer une réponse ou afficher une liste ordonnée.

## Exemple curl

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/rerank" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_RERANK",
    "query": "Quelle est la capitale de la France ?",
    "documents": [
      "Paris est la capitale.",
      "Le croissant est une viennoiserie.",
      "Lyon est une ville française."
    ],
    "top_n": 2
  }'
```

## Exemple Python (SDK OpenAI)

```python
import os
import requests

base_url = "https://albert.api.etalab.gouv.fr/v1"
api_key = os.environ["ALBERT_API_KEY"]
headers = {"Authorization": f"Bearer {api_key}"}

resp = requests.post(
    url=f"{base_url}/rerank",
    headers=headers,
    json={
        "model": "REMPLACER_PAR_MODELE_RERANK",
        "query": "Quelle est la capitale de la France ?",
        "documents": [
            "Paris est la capitale.",
            "Le croissant est une viennoiserie.",
            "Lyon est une ville française.",
        ],
        "top_n": 2,
    },
)
resp.raise_for_status()
print(resp.json())
```

Pour la structure exacte de la réponse (scores, indices, etc.), voir la page de l’endpoint **Rerank** :
[page de l’endpoint Rerank](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/rerank).
