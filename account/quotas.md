# Quotas et limites

Les paramètres de consommation sont portés par l’objet **`UserInfo`** (`GET /v1/me/info`) et structurés à l’aide de **`Limit`** pour les plafonds par routeur de modèles.

## Objet `Limit`

Chaque entrée du tableau **`limits`** décrit une contrainte :

| Champ | Signification |
| --- | --- |
| **`router`** | Identifiant entier du **routeur** de modèles auquel la limite s’applique. |
| **`type`** | Type énuméré **`LimitType`** : **`tpm`** (tokens par minute), **`tpd`** (tokens par jour), **`rpm`** (requêtes par minute), **`rpd`** (requêtes par jour). |
| **`value`** | Entier positif ou `null` — plafond associé ; la signification exacte dépend du `type` (tokens vs requêtes). |

Une même politique peut combiner plusieurs entrées `Limit` pour différents routeurs ou granularités temps.

Les compteurs s’appliquent en général **par utilisateur** (chaque consommation est suivie individuellement, et non pas mélangée entre comptes).

### Dépassement de limites

Lorsque le trafic ou les tokens dépassent les plafonds configurés pour votre compte, l’API peut répondre **429 Too Many Requests** (détail dans le corps ou les en-têtes selon la version). Stratégie recommandée : **backoff exponentiel**, respect éventuel d’un en-tête **`Retry-After`**, et réduction du parallélisme.

{% hint style="warning" %}
⚠️ À vérifier — Codes HTTP exacts (429 vs autre) et format du `detail` pour les dépassements sur l’instance Albert : consulter la [Référence OpenAPI](../api-reference/reference.md) et les réponses observées en préproduction.
{% endhint %}

## Budget (`UserInfo.budget`)

- Valeur **numérique** — enveloppe budgétaire restante ou autorisée selon les règles plateforme (coûts agrégés).
- **`null`** — d’après la spec, **budget illimité** côté modèle de données (cela ne supprime pas d’éventuelles limites techniques `limits`).

## Priorité (`UserInfo.priority`)

Entier **`priority`** : plus la valeur est élevée, plus l’utilisateur est **prioritaire** dans la file d’attente des invocations **non stream** (ordonnancement / scheduling). La valeur par défaut documentée est `0`.

## Qui configure ces valeurs ?

Les champs `limits`, `budget`, `priority` et droits associés sont **fixés par l’administration de la plateforme** (ou les mécanismes internes liés à votre compte organisationnel). Ils ne sont en général **pas modifiables** par simple appel public documenté ici.

{% hint style="warning" %}
⚠️ À vérifier — Interprétation monétaire précise du `budget`, fenêtres temporelles des compteurs (`tpd`, `rpd`) et comportement lorsque plusieurs limites entrent en conflit : demander la fiche d’exploitation à votre référent ou consulter les engagements contractuels.
{% endhint %}

Pour suivre la consommation réelle : [Usage & facturation](usage.md).
