# Table of contents

## Prise en main

* [Introduction](README.md)
* [Authentification](prise-en-main/authentication.md)
* [Création de compte & accès](prise-en-main/creation-compte.md)
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
* [Modèles (liste)](guides/models.md)
* [Streaming](guides/streaming.md)
* [Function calling & outils](guides/function-calling.md)
* [RAG - Génération Augmentée par Récupération](guides/rag.md)
* [Embeddings](guides/embeddings.md)
* [Transcription audio](guides/audio-transcription.md)
* [OCR](guides/ocr.md)
* [Reranking](guides/reranking.md)
* [Collections & documents](guides/collections-documents.md)
* [IDE (code models)](guides/ide.md)

## Compte & usage

* [Clés API](compte-and-usage/api-keys.md)
* [Quotas & limites](compte-and-usage/quotas.md)
* [Usage & facturation](compte-and-usage/usage.md)

## Modèles

* [Modèles disponibles](modeles/available-models.md)
* [Types de modèles](modeles/model-types.md)

## Ressources

* [Codes d'erreur](ressources/errors.md)
* [Changelog](ressources/changelog.md)
* [Support](ressources/support.md)
