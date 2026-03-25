# Table of contents

## Prise en main

* [Introduction](README.md)
* [Authentification](prise-en-main/authentication.md)
* [Démarrage rapide](prise-en-main/quickstart.md)
* [URL de base & versioning](prise-en-main/base-url.md)

## API REFERENCE

* [Liste des Endpoint](api-reference/liste-des-endpoint/README.md)
  * ```yaml
    type: builtin:openapi
    props:
      models: true
      downloadLink: true
    dependencies:
      spec:
        ref:
          kind: openapi
          spec: albert-api
    ```

## Guides

* [Chat completions](guides/chat-completions.md)
* [Streaming](guides/streaming.md)
* [Function calling & outils](guides/function-calling.md)
* [Outil de recherche RAG intégré](guides/rag-search-tool.md)
* [Embeddings](guides/embeddings.md)
* [Transcription audio](guides/audio-transcription.md)
* [OCR](guides/ocr.md)
* [Reranking](guides/reranking.md)
* [Collections & documents](guides/collections-documents.md)

## Compte & usage

* [Clés API](compte-and-usage/api-keys.md)
* [Quotas & limites](compte-and-usage/quotas.md)
* [Usage & facturation](compte-and-usage/usage.md)

## Modèles

* [Modèles disponibles](modeles/available-models.md)
* [Types de modèles](modeles/model-types.md)

## Ressources

* [Codes d'erreur](ressources/errors/README.md)
  * ```yaml
    type: builtin:openapi
    props:
      models: true
      downloadLink: true
    dependencies:
      spec:
        ref:
          kind: openapi
          spec: albert-api
    ```
* [Changelog](ressources/changelog.md)
* [Support](ressources/support.md)
