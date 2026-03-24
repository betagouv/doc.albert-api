# Documentation Albert API

Albert API est l’API publique du service gouvernemental **Albert**, qui expose des modèles de langage et des capacités associées derrière une surface **compatible OpenAI** (chat, embeddings, etc.). Les intégrations existantes basées sur le SDK OpenAI peuvent en général être réorientées vers cet endpoint en changeant l’URL de base et la clé. L’API est opérée dans un périmètre d’hébergement souverain adapté aux exigences des administrations, incluant des offres qualifiées **SecNumCloud** pour les déploiements concernés.

| Domaine | Description |
| --- | --- |
| Chat | Complétions de chat (`POST /v1/chat/completions`), y compris streaming et appels d’outils. |
| Embeddings | Représentations vectorielles de texte (`POST /v1/embeddings`). |
| Recherche & RAG | Recherche sémantique, lexicale et hybride sur collections indexées ; RAG natif via l’outil intégré `SearchTool` dans le chat. |
| Audio | Transcription (`POST /v1/audio/transcriptions`, formats pris en charge côté API : mp3, wav). |
| OCR | Extraction de texte et structure depuis documents ou images (`POST /v1/ocr`). |
| Reranking | Réordonnancement de documents par pertinence (`POST /v1/rerank`). |
| Collections | Gestion de collections et de documents pour l’indexation et la recherche (`/v1/collections`, `/v1/documents`). |

**Démarrage :** suivez le [guide de démarrage rapide](getting-started/quickstart.md) pour obtenir un premier `chat.completion` en quelques minutes.

Des **notebooks Jupyter** (RAG, OCR, chat, audio, etc.) sont maintenus dans le dépôt open source du moteur : [tutoriels OpenGateLLM](https://github.com/etalab-ia/opengatellm/tree/main/docs/tutorials) (chemins et schémas exacts peuvent différer légèrement de l’instance Albert : se caler sur la [référence OpenAPI](api-reference/reference.md) pour la production).
