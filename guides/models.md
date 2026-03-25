# Récupérer la liste des modèles

Ce guide montre comment interroger **`GET /v1/models`** avec le SDK OpenAI (version “compatible OpenAI”) pour récupérer les modèles disponibles, puis filtrer par type (chat, embeddings, OCR, audio, etc.).

## Pré-requis

1. Définir votre clé API :

```bash
export ALBERT_API_KEY="votre_jeton"
```

2. Installer le SDK :

```bash
pip install -U openai
```

## Liste des modèles (`GET /v1/models`)

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

models = client.models.list().data

for m in models[:3]:
    print(
        f"ID: {m.id}\n"
        f"Type: {m.type}\n"
        f"Owner: {m.owned_by}\n"
        f"Max tokens: {m.max_context_length}\n"
    )
```

## Filtrer selon l’usage

En pratique, choisissez un modèle selon son champ `type` :

* `text-generation` : **chat** (`POST /v1/chat/completions`)
* `text-embeddings-inference` : **embeddings** (`POST /v1/embeddings`)
* `automatic-speech-recognition` : **audio** (`POST /v1/audio/transcriptions`)
* `image-to-text` ou variantes `*ocr*` : **OCR** (`POST /v1/ocr`)

{% hint style="warning" %}
⚠️ À vérifier — Le mapping exact entre `type` et les endpoints dépend de la configuration de votre instance. Utilisez la liste des modèles et vérifiez le comportement sur votre environnement.
{% endhint %}

## Détail d’un modèle (`GET /v1/models/{model}`)

```python
model_id = models[0].id

model = client.models.retrieve(model=model_id)

print(
    f"ID: {model.id}\n"
    f"Type: {model.type}\n"
    f"Owner: {model.owned_by}\n"
    f"Max tokens: {model.max_context_length}\n"
)
```

Pour la documentation endpoint, voir la [page “Models”](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/models).

