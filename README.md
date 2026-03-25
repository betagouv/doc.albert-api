# Introduction

Albert API permet aux applications et agents d’appeler des modèles de **génération de texte**, d’**embeddings**, de **classification** (rerank), de **reconnaissance vocale**, d’**OCR** et des services de **recherche** sur des corpus indexés, via des routes et des schémas calqués sur l’**API OpenAI** là où c’est pertinent.

## À qui s’adresse cette documentation

Cette documentation vise les **développeurs et développeuses** qui intègrent l’API dans un produit ou un script : authentification, choix de modèles, appels HTTP et bonnes pratiques opérationnelles (quotas, erreurs, retries).

## URL et spécification

* **URL de base :** `https://albert.api.etalab.gouv.fr`
* **OpenAPI 3.1 :** [Référence OpenAPI interactive et schémas](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint)

## Prochaines étapes

1. [Authentification](prise-en-main/authentication.md) — en-tête Bearer et obtention d’un jeton.
2. [Démarrage rapide](prise-en-main/quickstart.md) — premier appel `POST /v1/chat/completions`.
3. [Guides](guides/chat-completions.md) — paramètres avancés, streaming, outils et RAG.

## Ressources complémentaires

* [Notebooks OpenGateLLM](https://github.com/etalab-ia/opengatellm/tree/main/docs/tutorials) — pas-à-pas Colab/Jupyter (RAG, OCR, modèles, etc.) ; croiser avec la spec OpenAPI Albert pour les noms de champs à jour.

## Obtenir un accès

Si vous êtes agent de la fonction publique d’État, l’accès à Albert API est ouvert via une demande en ligne : vous recevrez un mail avec vos identifiants et la documentation dans les 24 heures.

[Demander un accès](https://albert.sites.beta.gouv.fr/access/)

## Tarifs & quotas (vision par modèle)

La page “Tarifs et limites” liste, pour des familles de modèles, des quotas d’usage en mode **expérimentation** et **production** (RPM/RPD/TPM/TPD).

[Tarifs et limites](https://albert.sites.beta.gouv.fr/prices/)

{% hint style="warning" %}
⚠️ Les quotas indiqués sur le site sont une vue “tarifs” : pour connaître vos limites exactes (compte, routeurs, fenêtres), consultez l’objet `limits` dans **`GET /v1/me/info`** et la page [Quotas & limites](compte-and-usage/quotas.md).
{% endhint %}

## Sécurité & hébergement souverain

Albert API bénéficie d’un environnement cloud souverain (certification **SecNumCloud** via Outscale) et a des engagements de traitement des données :

* Albert API ne conserve aucune trace des conversations envoyées aux modèles.
* Albert API n’envoie aucune de vos données sur Internet.

[Sécurité & hébergement](https://albert.sites.beta.gouv.fr/solutions/security/)

## Statut en production

Pour suivre le statut opérationnel de l’API (quand publié), voir :

[Statut en production](https://albert.sites.beta.gouv.fr/about/status/)
