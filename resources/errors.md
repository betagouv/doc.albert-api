# Codes d'erreur

Albert API s’appuie sur FastAPI / Pydantic : les erreurs métier suivent en général le schéma **`HTTPExceptionModel`**, tandis que les requêtes mal formées peuvent produire une **`HTTPValidationError`**.

## Schémas `HTTPExceptionModel` (deux variantes)

La spec OpenAPI enregistre **deux** modèles homonymes pour les réponses d’exception HTTP :

### Variante « infrastructure » (`api.infrastructure.fastapi.documentation.HTTPExceptionModel`)

Utilisée par exemple pour certaines réponses **401** :

- **`status_code`** — entier HTTP ;
- **`detail`** — **chaîne** décrivant l’erreur (ex. clé invalide) ;
- **`headers`** — tableau associatif string → string optionnel.

Les champs **`status_code`** et **`detail`** sont **requis**. Le champ **`headers`** est optionnel et peut être absent ou `null`.

### Variante « exceptions API » (`api.schemas.exception.HTTPExceptionModel`)

Utilisée notamment pour les réponses **503** de type *« Model is too busy »* sur `POST /v1/chat/completions` et `POST /v1/ocr` :

- **`status_code`** — entier entre **100 et 599** (contrainte min/max dans le schéma) ;
- **`detail`** — **contenu quelconque** sérialisable JSON (pas limité à une chaîne) ;
- **`headers`** — map optionnelle ; dans ce schéma la clé **`headers`** doit être **présente** sur l’objet (champ requis au sens JSON Schema) mais sa valeur peut être **`null`** ou un dictionnaire de paires chaîne → chaîne.

En pratique, commencez par lire **`status_code`** et sérialisez **`detail`** en texte pour les logs sans supposer uniquement une chaîne plate.

## Erreurs de validation — `HTTPValidationError`

Corps typique :

- **`detail`** — liste d’objets **`ValidationError`** avec :
  - **`loc`** — chemin du champ fautif (chaînes et entiers mélangés) ;
  - **`msg`** — message lisible ;
  - **`type`** — code d’erreur Pydantic / JSON Schema.

Réponse HTTP associée : **422 Unprocessable Entity**.

## Table des statuts HTTP courants

| Code | Signification | Contexte |
| --- | --- | --- |
| **200** | OK | Lecture réussie (liste, détail, streaming entamé correctement). |
| **201** | Created | Ressource créée (ex. certaines collections). |
| **204** | No Content | Suppression ou action sans corps de réponse. |
| **401** | Unauthorized | Schéma d’authentification invalide ou jeton refusé. |
| **403** | Forbidden | Accès refusé au routeur ou à l’opération (permissions, limite à zéro, ou règle de droits). |
| **404** | Not Found | Ressource absente ou identifiant inconnu (y compris modèle indisponible pour le compte). |
| **422** | Validation Error | Corps ou paramètres hors schéma (`HTTPValidationError`). |
| **429** | Too Many Requests | Dépassement de quota / rate limiting (requêtes ou tokens selon configuration). |
| **503** | Service Unavailable | Saturation modèle : message *« Model is too busy, please try again later. »* |

## Cas particulier : 429 (limites / quotas)

En cas de **429**, réduisez le débit, mettez en file d’attente côté client, et relancez avec un délai croissant. Si l’API renvoie un **`Retry-After`**, respectez-le.

## Cas particulier : 503 « Model is too busy »

Lorsque le moteur d’inférence est saturé, les endpoints concernés peuvent répondre **503** avec le schéma `HTTPExceptionModel` « exceptions API ». Recommandations :

1. **Retry avec backoff exponentiel** — espacer les nouvelles tentatives (centaines de ms → secondes) et borner le nombre total d’essais.
2. **Éviter le thundering herd** — jitter aléatoire sur les clients multiples partageant le même compte.
3. **Idempotence** — pour les opérations non naturellement idempotentes, gardez un identifiant métier côté client avant de relancer.
4. **Streaming** — si vous ouvrez déjà un flux, anticipez une coupure réseau ou une erreur mid-stream identique à traiter comme échec retriable.

Pour une cartographie complète des réponses par route : [Référence OpenAPI](../api-reference/reference.md).
