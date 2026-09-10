---
icon: key
---

# Clefs API

Albert API utilise un système de clef pour authentifier vos requêtes. Ces clefs sont des **Bearer** tokens.

### Création d'une clef

Vous pouvez créer des clefs d'API de 2 manières, par API ou sur le Playground.

{% tabs %}
{% tab title="Playground" icon="globe-pointer" %}
Connectez-vous sur le [playground](https://albert.playground.gouv.fr/), puis rendez-vous dans la page _API Keys_.
{% endtab %}

{% tab title="API" icon="square-terminal" %}
Vous pouvez créer une clef d'API en appelant le endpout POST `/v1/keys`.<br>

```
curl -x 
```
{% endtab %}
{% endtabs %}





#### Expiration des clefs

**Toutes les clefs API ont une date d'expiration de maximum 1 an à compter de la date de création.**&#x20;

Il est impossible d'obtenir une clef sans expiration.







## Format des jetons

Les secrets d’accès sont souvent préfixés par **`sk-`** et peuvent correspondre à un **JWT** encodé (contenant typiquement des identifiants utilisateur et de clé). Traitez la chaîne complète comme **opaque** : ne la parsez pas côté client pour la logique métier.

## Créer une clé — `POST /v1/me/keys`

Corps JSON **`CreateKey`** :

* **`name`** (requis) — libellé pour retrouver la clé dans les listes ;
* **`expires`** — horodatage Unix **en secondes** après lequel la clé n’est plus valide, ou `null` pour absence d’expiration explicite.

Réponse **`CreateKeyResponse`** :

* **`id`** — identifiant entier de la clé ;
* **`token`** — secret **affiché intégralement une seule fois** à la création (selon configuration / environnement).

{% hint style="danger" %}
Le champ **`token`** n’est pas récupérable après coup par l’API documentée : enregistrez-le dans un coffre-fort de secrets (`ALBERT_API_KEY`, gestionnaire d’identifiants, vault). Toute perte implique la révocation et la création d’une nouvelle clé.
{% endhint %}

{% hint style="warning" %}
⚠️ Comportement observé en test (runner) : `POST /v1/me/keys` peut renvoyer **uniquement** `id` (sans `token`). Dans ce cas, la méthode recommandée pour récupérer une nouvelle clé utilisable est de la générer via le **Playground** (qui affiche la clé une seule fois), puis de la stocker en lieu sûr.
{% endhint %}

**Exemple de requête :**

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/me/keys" \
  -H "Authorization: Bearer $ALBERT_EXISTANT" \
  -H "Content-Type: application/json" \
  -d '{"name": "ci-github", "expires": null}'
```
{% endtab %}

{% tab title="Python" %}
```python
import os
import requests

resp = requests.post(
    "https://albert.api.etalab.gouv.fr/v1/me/keys",
    headers={
        "Authorization": f"Bearer {os.environ['ALBERT_EXISTANT']}",
        "Content-Type": "application/json",
    },
    json={"name": "ci-github", "expires": None},
)
resp.raise_for_status()
print(resp.json())
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const resp = await fetch("https://albert.api.etalab.gouv.fr/v1/me/keys", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_EXISTANT}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ name: "ci-github", expires: null }),
});

if (!resp.ok) throw new Error(await resp.text());
console.log(await resp.json());
```
{% endtab %}
{% endtabs %}

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
