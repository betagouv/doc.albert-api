# Types de modèles

L’énumération **`ModelType`** décrit le rôle d’un modèle exposé par `GET /v1/models`. Le tableau ci-dessous relie chaque valeur aux **endpoints principaux** qui l’utilisent.

| `ModelType`                    | Endpoint(s) typique(s)                      | Rôle                                                               |
| ------------------------------ | ------------------------------------------- | ------------------------------------------------------------------ |
| `text-generation`              | `POST /v1/chat/completions`                 | Génération de texte conversationnel, outils inclus.                |
| `text-embeddings-inference`    | `POST /v1/embeddings`                       | Vecteurs de texte pour recherche, clustering, features.            |
| `text-classification`          | `POST /v1/rerank`                           | Scoring de paires (requête, document) pour réordonnancement.       |
| `automatic-speech-recognition` | `POST /v1/audio/transcriptions`             | Transcription (et éventuelle traduction contrôlée via `language`). |
| `image-text-to-text`           | `POST /v1/ocr` (selon configuration modèle) | Modèles vision + langage sur documents ou images.                  |
| `image-to-text`                | `POST /v1/ocr` (selon configuration modèle) | Spécialisation plus étroite image → texte.                         |

{% hint style="warning" %}
⚠️ À vérifier — Affectation exacte des modèles `image-text-to-text` vs `image-to-text` à l’endpoint OCR et liste blanche par environnement : contrôlez les fiches `Model` retournées par `/v1/models`.
{% endhint %}

## Champs utiles sur `Model`

* **`max_context_length`** — borne haute **tokens** pour la fenêtre contextuelle ; requis pour dimensionner prompts + complétions ou embeddings massifs.
* **`aliases`** — raccourcis acceptés à la place de `id` dans les requêtes.
* **`costs`** — objet **`ModelCosts`** :
  * **`prompt_tokens`** — coût monétaire **par million de tokens** consommés en entrée ;
  * **`completion_tokens`** — coût **par million de tokens** générés en sortie.

Les unités monétaires et la décrementation du **budget** utilisateur dépendent de la politique commerciale / interministerielle appliquée à votre compte.

Pour l’exploration dynamique : [Modèles disponibles](available-models.md).
