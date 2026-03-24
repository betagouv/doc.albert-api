# Embeddings

L’endpoint **`POST /v1/embeddings`** calcule des vecteurs pour du texte (ou des entrées tokenisées) à l’aide d’un modèle dont le type est **`text-embeddings-inference`**.

## Requête

Champs principaux du schéma **`EmbeddingsRequest`** :

- **`model`** — identifiant retourné par `GET /v1/models` pour un modèle d’embeddings.
- **`input`** — chaîne unique, **tableau de chaînes** (plusieurs phrases en un appel), ou représentations en tokens selon la spec. La chaîne vide n’est pas valide ; chaque entrée doit respecter la limite du modèle (**`max_context_length`** côté objet `Model`).
- **`dimensions`** — dimension vectorielle souhaitée lorsque le modèle le permet (sinon `null`).
- **`encoding_format`** — seule la valeur **`float`** est supportée (c’est la valeur par défaut).

## Réponse

La réponse suit la forme habituelle **OpenAI embeddings** : liste d’objets avec `embedding`, `index`, et métadonnées d’usage (`usage`). Consultez le schéma `Embeddings` dans la [Référence OpenAPI](../api-reference/reference.md).

## Exemple curl

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/embeddings" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_EMBEDDINGS",
    "input": "Le chat embeddings sert à la recherche sémantique."
  }'
```

## Exemple Python

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

r = client.embeddings.create(
    model="REMPLACER_PAR_MODELE_EMBEDDINGS",
    input=["Première phrase.", "Deuxième phrase."],
)
for item in r.data:
    print(item.index, len(item.embedding))
```

Les vecteurs obtenus alimentent typiquement une base vectorielle ou la couche de recherche décrite dans [Outil de recherche RAG intégré](rag-search-tool.md) et [Collections & documents](collections-documents.md).
