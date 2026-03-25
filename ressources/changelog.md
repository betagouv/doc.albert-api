# Changelog

Cette page résume l’évolution documentaire et fonctionnelle visible côté consommateurs de l’API Albert.

## Référence de version

La **version du contrat OpenAPI** exposée sur `https://albert.api.etalab.gouv.fr/openapi.json` figure dans le champ `info.version` du document (par exemple `0.4.x` selon les déploiements). C’est la **source de vérité** pour les routes et schémas disponibles sur un environnement donné.

## Évolutions récentes mentionnées dans la spec

* **Outil `SearchTool`** — recherche documentaire intégrée au chat via `tools` avec `type: "search"`.
* **Dépréciations** — anciens champs `search` / `search_args` sur le chat ; endpoint `POST /v1/parse-beta` au profit de `POST /v1/ocr` ; routes `GET /v1/chunks/{document}` au profit de `GET /v1/documents/{document_id}/chunks`.

{% hint style="warning" %}
⚠️ À vérifier — Journal officiel des versions (releases GitHub, fichier CHANGELOG du dépôt moteur ou page communication beta.gouv) : consolidez le lien canonique utilisé par votre organisation à partir des dépôts `betagouv` / `etalab-ia` associés au produit Albert.
{% endhint %}

## Tutoriels Jupyter

Le dépôt [etalab-ia/opengatellm — `docs/tutorials/`](https://github.com/etalab-ia/opengatellm/tree/main/docs/tutorials) contient des notebooks (**RAG**, **OCR**, **chat completions**, **audio**, **modèles**) alignés sur la même stack que l’API. **Adaptez** les URLs et les noms de champs JSON à la spec OpenAPI de l’instance Albert (`/v1/...`, `collection_ids`, `query`, etc.).

## Documentation GitBook

Les mises à jour de ce livre sont **versionnées dans le dépôt Git** synchronisé avec GitBook. Comparez les commits sur la branche `main` pour l’historique détaillé du contenu éditorial.
