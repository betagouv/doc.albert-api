# Clés API

Les clés d’API Albert sont gérées sous **`/v1/me/keys`**. Elles servent de **jetons Bearer** pour toutes les routes protégées.

## Format des jetons

Les secrets d’accès sont souvent préfixés par **`sk-`** et peuvent correspondre à un **JWT** encodé (contenant typiquement des identifiants utilisateur et de clé). Traitez la chaîne complète comme **opaque** : ne la parsez pas côté client pour la logique métier.

## Créer une clé — `POST /v1/me/keys`

Corps JSON **`CreateKey`** :

* **`name`** (requis) — libellé pour retrouver la clé dans les listes ;
* **`expires`** — horodatage Unix **en secondes** après lequel la clé n’est plus valide, ou `null` pour absence d’expiration explicite.

Réponse **`CreateKeyResponse`** :

* **`id`** — identifiant entier de la clé ;
* **`token`** — secret **affiché intégralement une seule fois** à la création.

{% hint style="danger" %}
Le champ **`token`** n’est pas récupérable après coup par l’API documentée : enregistrez-le dans un coffre-fort de secrets (`ALBERT_API_KEY`, gestionnaire d’identifiants, vault). Toute perte implique la révocation et la création d’une nouvelle clé.
{% endhint %}

### Exemple

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/me/keys" \
  -H "Authorization: Bearer $ALBERT_EXISTANT" \
  -H "Content-Type: application/json" \
  -d '{"name": "ci-github", "expires": null}'
```

## Lister les clés — `GET /v1/me/keys`

Retourne une liste paginée (`offset`, `limit`, `order_by`, `order_direction`) d’objets **`Key`** (`id`, `name`, `token`, `expires`, `created`, …). Le champ `token` est présent dans le schéma public — **traitez toute valeur affichée comme sensible** et ne la journalisez pas côté client public.

{% hint style="warning" %}
⚠️ À vérifier — Politique réelle de masquage du secret sur les réponses `GET` en production (affichage complet vs préfixe) : valider sur votre compte avant d’afficher la liste à des utilisateurs finaux.
{% endhint %}

## Détail — `GET /v1/me/keys/{key}`

Consultation d’une entrée précise ; `key` est l’**identifiant entier** de la clé.

## Révoquer — `DELETE /v1/me/keys/{key}`

Supprime la clé identifiée par son **`id`**. Réponse **`204`** sans corps en cas de succès.

Pour le profil utilisateur (budget, limites) : [Quotas & limites](quotas.md).
