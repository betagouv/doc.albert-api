# URL de base & versioning

## URL de base

Toutes les routes de la version courante de l’API sont exposées sous le préfixe **`/v1`** sur l’hôte suivant :

```
https://albert.api.etalab.gouv.fr
```

Exemple complet pour les modèles :

```
https://albert.api.etalab.gouv.fr/v1/models
```

Avec le SDK OpenAI, configurez :

```
base_url = "https://albert.api.etalab.gouv.fr/v1"
```

Le client ajoute alors automatiquement les chemins (`/chat/completions`, etc.).

## Versioning

Le numéro `v1` dans les chemins indique la **version majeure** de la surface HTTP exposée. En cas d’évolution breaking, une nouvelle version pourrait être publiée sous un autre préfixe ; les clients doivent **caler leur intégration** sur le préfixe documenté et surveiller les annonces de changement.

{% hint style="warning" %}
⚠️ À vérifier — Politique précise de dépréciation, période de grâce et calendrier des évolutions : consulter les canaux officiels et le [Changelog](../ressources/changelog.md).
{% endhint %}

## Santé et supervision (sans Bearer)

Des endpoints de monitoring existent en dehors de `/v1`, par exemple `GET /health` et `GET /metrics`. Ils servent surtout à l’observabilité de la plateforme et ne remplacent pas une authentification applicative sur les routes métier.

Pour le détail des chemins et schémas : [Référence OpenAPI](/broken/pages/vRZwVxl3vzeW4GecHnw7).
