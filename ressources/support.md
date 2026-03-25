# Support

## Documentation technique

* Ce site (GitBook) — parcours guidés et la [liste des endpoints GitBook](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint)
* Fichier **`openapi.json`** — comportement exact des schémas et énumérations pour votre environnement.
* Statut en production : [Statut de l’API](https://albert.sites.beta.gouv.fr/about/status/)
* [Notebooks OpenGateLLM](https://github.com/etalab-ia/opengatellm/tree/main/docs/tutorials) — exemples pas-à-pas (à recouper avec la spec Albert pour les champs exacts).

## Signaler un problème

{% hint style="warning" %}
⚠️ À vérifier — Canal unique de support développeur (ticketting interne, issue tracker GitHub `betagouv/doc.albert-api` ou dépôt applicatif amont) : utilisez la procédure définie par votre organisation ou le programme **beta.gouv** dont dépend votre accès à Albert API.
{% endhint %}

**Bonnes pratiques pour un signalement utile :**

* fenêtre horaire et fuseau ;
* `request_id` ou extraits de **logs** sans secrets (tracez un hash de jeton, pas la clé complète) ;
* code HTTP, extrait JSON d’erreur (`detail`) ;
* modèle et endpoint concernés ;
* reproductibilité minimale (curl anonymisé).

## Ressources liées

* Obtenir ou gérer des clés : [Clés API](../compte-and-usage/api-keys.md).
* Comprendre les erreurs : [Codes d'erreur](errors.md).
