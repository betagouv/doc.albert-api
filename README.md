# Introduction

Albert API permet aux applications et agents d’appeler des modèles de **génération de texte**, d’**embeddings**, de **classification** (rerank), de **reconnaissance vocale**, d’**OCR** et des services de **recherche** sur des corpus indexés, via des routes et des schémas calqués sur l’**API OpenAI** là où c’est pertinent.

## À qui s’adresse cette documentation

Cette documentation vise les **développeurs et développeuses** qui intègrent l’API dans un produit ou un script : authentification, choix de modèles, appels HTTP et bonnes pratiques opérationnelles (quotas, erreurs, retries).

## URL et spécification

* **URL de base :** `https://albert.api.etalab.gouv.fr`
* **OpenAPI 3.1 :** [Référence OpenAPI interactive](/broken/pages/vRZwVxl3vzeW4GecHnw7) (fichier `openapi.json` à la racine du même hôte)

## Prochaines étapes

1. [Authentification](prise-en-main/authentication.md) — en-tête Bearer et obtention d’un jeton.
2. [Démarrage rapide](prise-en-main/quickstart.md) — premier appel `POST /v1/chat/completions`.
3. [Guides](guides/chat-completions.md) — paramètres avancés, streaming, outils et RAG.

## Ressources complémentaires

* [Notebooks OpenGateLLM](https://github.com/etalab-ia/opengatellm/tree/main/docs/tutorials) — pas-à-pas Colab/Jupyter (RAG, OCR, modèles, etc.) ; croiser avec la spec OpenAPI Albert pour les noms de champs à jour.
