# Démarrage rapide

Objectif : obtenir une réponse du modèle via **`POST /v1/chat/completions`** en moins de cinq minutes.

## Étape 1 — Obtenir un jeton

Créez une clé API avec `POST /v1/me/keys` (voir [Clés API](../compte-and-usage/api-keys.md)). Exportez-la dans votre environnement, par exemple :

```bash
export ALBERT_API_KEY="votre_jeton"
```

## Étape 2 — Choisir un modèle

Listez les modèles disponibles :

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/models" \
  -H "Authorization: Bearer $ALBERT_API_KEY"
```

Choisissez un modèle dont le champ `type` est **`text-generation`** (voir [Types de modèles](../modeles/model-types.md)). Notez son `id` (par ex. `Meta-Llama-3.1-8B-Instruct` — l’offre exacte dépend de la plateforme).

## Étape 3 — Envoyer une complétion de chat

### curl

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_ID_MODELE",
    "messages": [
      {"role": "system", "content": "Tu réponds en français, de façon concise."},
      {"role": "user", "content": "Explique ce qu’est une API compatible OpenAI en deux phrases."}
    ]
  }'
```

### Python (SDK OpenAI)

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

r = client.chat.completions.create(
    model="REMPLACER_PAR_ID_MODELE",
    messages=[
        {"role": "system", "content": "Tu réponds en français, de façon concise."},
        {"role": "user", "content": "Explique ce qu’est une API compatible OpenAI en deux phrases."},
    ],
)
print(r.choices[0].message.content)
```

## Compatibilité OpenAI

Albert API est **compatible OpenAI** sur les endpoints courants : en conservant votre code basé sur le SDK ou sur des appels HTTP OpenAI, vous changez en principe uniquement la **base URL** (ici `https://albert.api.etalab.gouv.fr/v1`) et la **clé** (jeton Albert). Les extensions spécifiques — par exemple l’outil natif **SearchTool** pour le RAG — sont documentées à part ; elles ne sont pas équivalentes aux outils hébergés par OpenAI.

Pour aller plus loin : [Chat completions](../guides/chat-completions.md), [page de l’endpoint Chat](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/chat).
