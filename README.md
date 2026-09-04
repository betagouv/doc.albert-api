---
icon: stars
---

# Qu'est-ce qu'Albert API ?

Albert API permet aux applications et agents d’appeler des modèles de **génération de texte**, d’**embeddings**, de **classification** (rerank), de **reconnaissance vocale**, d’**OCR** et des services de **recherche** sur des corpus indexés, via des routes et des schémas calqués sur l’**API OpenAI** là où c’est pertinent.

## À qui s’adresse cette documentation

Cette documentation vise les **développeurs et développeuses** qui intègrent l’API dans un produit ou un script : authentification, choix de modèles, appels HTTP et bonnes pratiques opérationnelles (quotas, erreurs, retries).

## URL et spécification

* **URL de base :** `https://albert.api.etalab.gouv.fr`
* **OpenAPI 3.1 :** [Référence OpenAPI interactive et schémas](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint)

## Prochaines étapes

1. [Authentification](/broken/pages/kAnuH9jbzC0DENsy17Pl) — en-tête Bearer et obtention d’un jeton.
2. [Démarrage rapide](prise-en-main/quickstart.md) — premier appel `POST /v1/chat/completions`.
3. [Guides](guides/chat-completions.md) — paramètres avancés, streaming, outils et RAG.

## Obtenir un accès

Si vous êtes agent de la fonction publique d’État, l’accès à Albert API se fait automatiquement via ProConnect. Si vous rencontrez des problèmes de connexion, veuillez consulter notre [FAQ](https://guides.ia.numerique.gouv.fr/albert-api/ressources/faq#pourquoi-mon-compte-nest-pas-autorise)

## Tarifs & quotas (vision par modèle)

La page “Tarifs et limites” liste, pour des familles de modèles, des quotas d’usage en mode **expérimentation** et **production** (RPM/RPD/TPM/TPD).

[Tarifs et limites](https://ia.numerique.gouv.fr/outils-ia/albert-api/tarifs-et-limites/)

{% hint style="warning" %}
⚠️ Les quotas indiqués sur le site sont une vue “tarifs” : pour connaître vos limites exactes (compte, routeurs, fenêtres), consultez l’objet `limits` dans **`GET /v1/me/info`** et la page [Quotas & limites](compte-and-usage/quotas.md).
{% endhint %}

## Sécurité & hébergement souverain

Albert API bénéficie d’un environnement cloud souverain (certification **SecNumCloud** via Outscale) et a des engagements de traitement des données :

* Albert API ne conserve aucune trace des conversations envoyées aux modèles (voir [la page dédiée](ressources/absence-de-retention-des-donnees-metier-sur-le-chemin-dinference.md))
* Albert API n’envoie aucune de vos données sur Internet.

[Sécurité & hébergement](https://ia.numerique.gouv.fr/outils-ia/albert-api/infrastructure-s%C3%A9curis%C3%A9e/)

## Compatibilité OpenAI

Albert API est **compatible OpenAI** sur les endpoints courants : en conservant votre code basé sur le SDK ou sur des appels HTTP OpenAI, vous changez en principe uniquement la **base URL** (ici `https://albert.api.etalab.gouv.fr/v1`) et la **clé** (jeton Albert).

Pour aller plus loin : [Chat completions](guides/chat-completions.md), [page de l’endpoint Chat](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/chat).

## Statut en production

Pour suivre le statut opérationnel de l’API (quand publié), voir :

[Statut en production](https://ia.numerique.gouv.fr/outils-ia/albert-api/statuts-de-lapi/)
