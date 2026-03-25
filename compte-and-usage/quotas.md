# Quotas & limites

Les paramètres de consommation sont portés par l’objet **`UserInfo`** (`GET /v1/me/info`) et structurés à l’aide de **`Limit`** pour les plafonds par routeur de modèles.

## Objet `Limit`

Chaque entrée du tableau **`limits`** décrit une contrainte :

| Champ        | Signification                                                                                                                                              |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`router`** | Identifiant entier du **routeur** de modèles auquel la limite s’applique.                                                                                  |
| **`type`**   | Type énuméré **`LimitType`** : **`tpm`** (tokens par minute), **`tpd`** (tokens par jour), **`rpm`** (requêtes par minute), **`rpd`** (requêtes par jour). |
| **`value`**  | Entier positif ou `null` — plafond associé ; la signification exacte dépend du `type` (tokens vs requêtes).                                                |

Une même politique peut combiner plusieurs entrées `Limit` pour différents routeurs ou granularités temps.

Les compteurs s’appliquent en général **par utilisateur** (chaque consommation est suivie individuellement, et non pas mélangée entre comptes).

### Dépassement de limites

Lorsque le trafic ou les tokens dépassent les plafonds configurés pour votre compte, l’API peut répondre **429 Too Many Requests** (détail dans le corps ou les en-têtes selon la version). Stratégie recommandée : **backoff exponentiel**, respect éventuel d’un en-tête **`Retry-After`**, et réduction du parallélisme.

{% hint style="warning" %}
⚠️ À vérifier — Codes HTTP exacts (429 vs autre) et format du `detail` pour les dépassements sur l’instance Albert : consulter la page de l’endpoint **Me** :
[page de l’endpoint Me](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/me)
{% endhint %}

## Quotas “tarifs” (par famille de modèles)

En complément de vos limites propres au compte (champ `limits` dans `GET /v1/me/info`), la page officielle **Tarifs et limites** fournit une vue indicative par type de modèle et par niveau d’accès (**expérimentation** vs **production**).

[Tarifs et limites](https://albert.sites.beta.gouv.fr/prices/)

| Famille de modèles (ex.) | Accès | RPM | RPD | TPM | TPD |
| --- | --- | --- | --- | --- | --- |
| `openai/gpt-oss-120b` (chat) | expérimentation | 10 | 1000 | 128 000 | 1 280 000 |
| `openai/gpt-oss-120b` (chat) | production | 50 | 5000 | 246 000 | Illimité |
| `mistralai/Mistral-Small-*` / `Ministral-*` (chat) | expérimentation | 50 | 1000 | 128 000 | 2 460 000 |
| `mistralai/Mistral-Small-*` / `Ministral-*` (chat) | production | 100 | 50 000 | 246 000 | illimité |
| `Qwen/Qwen3-Coder-*` (code) | expérimentation | 50 | 1000 | 128 000 | 2 460 000 |
| `Qwen/Qwen3-Coder-*` (code) | production | 100 | 50 000 | 246 000 | illimité |
| `openai/whisper-large-v3` (audio) | expérimentation | 50 | 1000 | illimité | illimité |
| `openai/whisper-large-v3` (audio) | production | 100 | 5000 | illimité | illimité |
| `BAAI/bge-m3` (embeddings) | expérimentation | 500 | 50 000 | illimité | illimité |
| `BAAI/bge-m3` (embeddings) | production | 2000 | 200 000 | illimité | illimité |
| `BAAI/bge-reranker-v2-m3` (rerank) | expérimentation | 500 | 50 000 | illimité | illimité |
| `BAAI/bge-reranker-v2-m3` (rerank) | production | 2000 | 200 000 | illimité | illimité |

{% hint style="warning" %}
⚠️ À vérifier — la page “tarifs” peut évoluer. Les compteurs `Limit` dans `GET /v1/me/info` restent la source de vérité opérationnelle.
{% endhint %}

## Budget (`UserInfo.budget`)

* Valeur **numérique** — enveloppe budgétaire restante ou autorisée selon les règles plateforme (coûts agrégés).
* **`null`** — d’après la spec, **budget illimité** côté modèle de données (cela ne supprime pas d’éventuelles limites techniques `limits`).

## Priorité (`UserInfo.priority`)

Entier **`priority`** : plus la valeur est élevée, plus l’utilisateur est **prioritaire** dans la file d’attente des invocations **non stream** (ordonnancement / scheduling). La valeur par défaut documentée est `0`.

## Qui configure ces valeurs ?

Les champs `limits`, `budget`, `priority` et droits associés sont **fixés par l’administration de la plateforme** (ou les mécanismes internes liés à votre compte organisationnel). Ils ne sont en général **pas modifiables** par simple appel public documenté ici.

{% hint style="warning" %}
⚠️ À vérifier — Interprétation monétaire précise du `budget`, fenêtres temporelles des compteurs (`tpd`, `rpd`) et comportement lorsque plusieurs limites entrent en conflit : demander la fiche d’exploitation à votre référent ou consulter les engagements contractuels.
{% endhint %}

Pour suivre la consommation réelle : [Usage & facturation](usage.md).
