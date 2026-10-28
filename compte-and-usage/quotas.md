---
icon: "gauge-simple-max"
---
# Quotas et limites

Chaque compte dispose de limites de consommation. Ces limites sont configurées par la DINUM. Elles sont de 2 types : limites par token et limites par requête. 

Les limites comptabilisés uniquement sur les endpoints qui appellent des modèles : `/v1/audio/transcriptions`, `/v1/chat/completions`, `/v1/embeddings`, `/v1/ocr`et `/v1/rerank`. 

Vous pouvez consulter les limites par token et par requête pour chaque modèle sur la documentation des modèles disponibles [ici](../modeles/available-models.md). **Les limites sont définis selon des niveaux d’accès : expérimentation, production limitée ou partenaire.**

## Niveaux d'accès (tiers)

### Expérimentation

L’accès à Albert API est ouvert à tous les agents de la fonction publique d'État des fins d'expérimentation.
Ces expérimentations sont toutefois soumises à certaines limitations d’usage :

* Accès aux modèles principaux d’Albert API
* Quotas d'utilisation bas, pouvant être augmenté selon le besoin (voir détails ci-dessous)
* Pas de garantie de disponibilité
* Accès restreint en période de forte demande

### Production limitée

Si vos quotas d'expérimentation s'avèrent insuffisants et que vous souhaitez utiliser Albert API avec des volumes plus importants, deux étapes sont nécessaires :

* Remplir le formulaire suivant : [Formulaire production limitée](https://grist.numerique.gouv.fr/o/albert/forms/dteMmqusoagbJ68woTRTZF/11)
* Nous contacter à l'adresse : albert.api@numerique.gouv.fr
* Nous appliquerons ensuite les quotas de « Production limitée » sur le compte indiqué

### Production (Partenaire)

En décembre 2025, la DINUM a publié à destination des autres ministères une note visant au cofinancement des infrastructures d'Albert API.

Les agents des ministères partenaires bénéficient de conditions d'utilisation d'Albert API pour la production :

* Accès à l'intégralité des modèles d'Albert API
* Garantie de disponibilité (SLA)
* Accompagnement personnalisé
* Quotas d'utilisation élevé, pouvant être augmenté selon le besoin (voir détails ci-dessous)

{% hint style="info" icon="lightbulb" %}
Pourquoi un cofinancement ?
* Les coûts d’hébergement, de calcul (GPU, licences LLM) et de maintenance sont directement liés au nombre d’utilisateurs.
* La mutualisation permet une meilleure efficience de la dépense publique et évite aux ministères d’investir individuellement dans des infrastructures coûteuses et complexes.
* Albert API est une solution clé en main, sécurisée permettant aux ministères de se concentrer sur leurs missions métiers.
{% endhint %}

## Limites par token

Dans Albert API, les limites par tokens sont exprimées en **tokens par minute (TPM) et par jour (TPD)**.

{% hint style="info" icon="lightbulb" %}
Si vous vous demandez ce qu'est un token ou comment le texte est découpé en tokens, consultez notre FAQ *[Qu'est ce qu'un token ?](../ressources/faq.md#quest-ce-quun-token)*.
{% endhint %}

### Comment sont comptabilisés les tokens ?

Les tokens sont comptabilisés sur les endpoints qui appellent des modèles : `/v1/audio/transcriptions`, `/v1/chat/completions`, `/v1/embeddings`, `/v1/ocr`et `/v1/rerank`. Ils sont comptabilisés différement selon les endpoints.

Vous retrouvez un champ `usage` en réponse de chacun de ces endpoints vous indiquants le nombre de tokens consommés par la requête.
De plus, les en-têtes de la réponse indiquent l'état de vos limites, voir [Suivre sa consommation en temps réel](#suivre-sa-consommation-en-temps-reel).

{% tabs %}
{% tab title="/v1/chat/completions" %}

**Les tokens sont comptabilisés sur tous les messages de la requête ainsi que sur tous les tokens générés par le modèle (content, reasoning, tool calls, etc.)**.

* `prompt_tokens` : tokens de tous les messages de la requête (content, reasoning, etc.)
* `completion_tokens` : tokens générés par le modèle (content, reasoning, tool calls, etc.)

Voici un exemple de requête avec sa réponse:

```json
{
    "model": "openweight-large",
    "messages": [
        {
            "role": "system",
            "content": "Tu es un assistant de chat qui répond à des questions." → 12 tokens
        },
        {
            "role": "user",
            "content": "Bonjour, comment allez-vous ?" → 6 tokens
        },
        {
            "role": "assistant",
            "content": "Je vais bien, merci pour votre question." → 9 tokens
        },
        {
            "role": "user",
            "content": "Quel est votre nom ?" → 5 tokens
        }
    ]
}
```

**Réponse**
```json
{
    "id": "chatcmpl-1234567890",
    "object": "chat.completion",
    "created": 1717000000,
    "model": "openweight-large",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "Je m'appelle Albert." → 5 token
            },
            "finish_reason": "stop"
        }
    ],
   "usage": {
        "prompt_tokens": 32,
        "completion_tokens": 5,
        "total_tokens": 37,
        "prompt_tokens_details": {
            "cached_tokens": 0
            },
        "cost": 0,
        "impacts": {
            "kWh": 0,
            "kgCO2eq": 0
        }
    }
  }
```


{% endtab %}
{% tab title="/v1/embeddings" %}

Les tokens sont comptabilisés **uniquement sur les inputs** de la requête.

* `prompt_tokens` : tokens des textes passés dans `input` (ou dans `messages`)
* `completion_tokens` : toujours 0

Voici un exemple de requête :
```json
{
    "model": "openweight-embeddings-large",
    "input": [
        "Albert API est un outil de IA open source.", → 10 tokens
        "Il est développé par la communauté de l'IA open source." → 13 tokens
    ]
}
```

{% endtab %}
{% tab title="/v1/rerank" %}

Les tokens sont comptabilisés **uniquement sur le query et les documents** de la requête.

* `prompt_tokens` : tokens de `query` + tokens de chaque document
* `completion_tokens` : toujours 0

Voici un exemple de requête :
```json
{
    "model": "openweight-rerank-large",
    "query": "Albert API est un outil de IA open source.", → 10 tokens
    "documents": [
        "Albert API est un outil de IA open source.", → 10 tokens
        "Il est développé par la communauté de l'IA open source." → 13 tokens
    ]
}
```
{% endtab %}

{% tab title="/v1/audio/transcriptions" %}

Les tokens sont comptabilisés sur le **`prompt` optionnel** (si fourni) et sur le **texte transcrit** renvoyé par le modèle. Le fichier audio lui-même n'est **pas** tokenisé.

* `prompt_tokens` : tokens du champ `prompt` (0 si le prompt est vide ou omis)
* `completion_tokens` : tokens du texte de la transcription

Voici un exemple de requête avec sa réponse :

```json
{
    "model": "whisper-large-v3",
    "file": "enregistrement.mp3",
    "language": "fr",
    "prompt": "Réunion d'équipe, vocabulaire technique." → 8 tokens
}
```

**Réponse** (`response_format=json`)
```json
{
    "id": "audio-1234567890",
    "model": "whisper-large-v3",
    "text": "Bonjour à tous, passons au point suivant de l'ordre du jour." → 14 tokens,
    "usage": {
        "prompt_tokens": 8,
        "completion_tokens": 14,
        "total_tokens": 22,
        "prompt_tokens_details": {
            "cached_tokens": 0
        },
        "cost": 0,
        "impacts": {
            "kWh": 0,
            "kgCO2eq": 0
        }
    }
}
```

{% endtab %}

{% tab title="/v1/ocr" %}

Les tokens sont comptabilisés sur le **`document_annotation_prompt` optionnel** (si fourni) et sur le **texte extrait** (markdown des pages + annotation document éventuelle). Le document ou l'image en entrée n'est **pas** tokenisé.

* `prompt_tokens` : tokens du champ `document_annotation_prompt` (0 si omis)
* `completion_tokens` : tokens du markdown de chaque page, plus ceux de `document_annotation` le cas échéant

Voici un exemple de requête avec sa réponse :

```json
{
    "model": "mistral-ocr-2512",
    "document": {
        "type": "document_url",
        "document_url": "https://example.com/facture.pdf"
    },
    "document_annotation_prompt": "Extrais le numéro de facture." → 6 tokens
}
```

**Réponse**
```json
{
    "id": "ocr-1234567890",
    "model": "mistral-ocr-2512",
    "pages": [
        {
            "index": 0,
            "markdown": "# Facture\nNuméro : FAC-2024-42\nMontant : 120 €" → 18 tokens,
            "images": []
        }
    ],
    "document_annotation": "{\"invoice_number\": \"FAC-2024-42\"}" → 8 tokens,
    "usage": {
        "prompt_tokens": 6,
        "completion_tokens": 26,
        "total_tokens": 32,
        "prompt_tokens_details": {
            "cached_tokens": 0
        },
        "cost": 0,
        "impacts": {
            "kWh": 0,
            "kgCO2eq": 0
        }
    }
}
```

{% endtab %}
{% endtabs %}

### Décompte du modèle ou d'Albert API

Lorsque le modèle renvoie son propre décompte de tokens (champ `usage` de sa réponse), Albert API reprend ce décompte, calculé avec le tokenizer du modèle. Sinon, Albert API compte les tokens avec son propre tokenizer, voir la FAQ *[Qu'est ce qu'un token ?](../ressources/faq.md#quest-ce-quun-token)*.

### Tokens en cache

Lorsqu'une partie du prompt a déjà été traitée récemment par le modèle (par exemple un long message système identique d'une requête à l'autre), le modèle la sert depuis son cache. Ces tokens sont indiqués dans `usage.prompt_tokens_details.cached_tokens` et sont déjà inclus dans `prompt_tokens`. Si le modèle ne renvoie pas ce champ, `cached_tokens` vaut 0.

Les tokens en cache demandent moins de ressources au modèle : seuls 20 % d'entre eux sont décomptés de vos limites TPM et TPD.

```
tokens décomptés = prompt_tokens + completion_tokens - 0,8 × cached_tokens (arrondi à l'entier)
```

Par exemple, une requête de 1 000 tokens de prompt, dont 800 en cache, et 200 tokens générés décompte 1 000 + 200 - 0,8 × 800 = 560 tokens de vos limites. Cette réduction ne s'applique qu'aux limites : `prompt_tokens` et `cost` restent inchangés.

## Limites par requête

Dans Albert API, les limites par requêtes sont exprimées en **requêtes par minute (RPM) et par jour (RPD)**. 
Les requêtes sont comptabilisées uniquement sur les endpoints qui appellent un modèle.

## Quand les limites sont-elles décomptées ?

1. **Avant l'appel au modèle** : la requête est refusée (**429**) s'il ne vous reste plus de requête dans votre limite RPM ou RPD, ou s'il vous reste moins de tokens dans votre limite TPM ou TPD que n'en compte le prompt. Le prompt est alors estimé avec le tokenizer d'Albert API, car le modèle n'a pas encore répondu.
2. **Après une réponse réussie** : la requête compte pour 1 dans vos limites RPM et RPD, et ses tokens sont décomptés de vos limites TPM et TPD.

Une requête refusée ou en échec n'est jamais décomptée. En streaming, la requête est décomptée à la fin du flux ; si vous coupez le flux avant la fin, seuls les tokens déjà reçus sont décomptés.

## Comment suivre sa consommation ?

### Voir ses limites et ce qu’il en reste

Deux moyens de consulter vos limites et ce que vous en avez consommé, pour chaque modèle auquel vous avez accès :

{% tabs %}
{% tab title="Playground" icon="globe-pointer" %}
Connectez-vous sur le [Playground Albert API](https://albert.playground.etalab.gouv.fr) et ouvrez la page **Limits**. Pour chaque modèle, elle affiche la valeur de chaque limite (RPM, RPD, TPM et TPD) et ce que vous en avez consommé dans la fenêtre en cours. La page se met à jour toutes les 30 secondes.
{% endtab %}

{% tab title="API" icon="square-terminal" %}
Appelez l’endpoint [GET /v1/limits](https://guides.ia.numerique.gouv.fr/albert-api/api-reference/liste-des-endpoint/limits#get-v1-limits) :

```bash
curl -X GET "https://albert.api.etalab.gouv.fr/v1/limits" \
  -H "Authorization: Bearer $API_KEY"
```

La réponse contient un élément par modèle auquel vous avez accès :

```json
{
    "object": "list",
    "data": [
        {
            "object": "model_limits",
            "router_id": 1,
            "model": "openweight-large",
            "limits": [
                {"type": "tpm", "value": 100000, "remaining": 87500},
                {"type": "tpd", "value": null, "remaining": 0},
                {"type": "rpm", "value": 60, "remaining": 58},
                {"type": "rpd", "value": 1000, "remaining": 942}
            ]
        }
    ]
}
```

* `value` : valeur de la limite. `null` signifie qu’il n’y a pas de limite sur ce type ; `remaining` n’a alors pas de sens.
* `remaining` : ce qu’il reste de la limite dans la fenêtre en cours (minute ou jour).
{% endtab %}
{% endtabs %}

Les limites TPM, TPD, RPM et RPD de chaque modèle par niveau d’accès sont aussi listées sur la page [Modèles disponibles](../modeles/available-models.md).

### Suivre sa consommation en temps réel

À chaque appel vers un modèle, que la requête réussisse ou soit refusée (429), les en-têtes de la réponse indiquent l’état de vos limites :

```
x-ratelimit-limit-requests: 60
x-ratelimit-remaining-requests: 59
x-ratelimit-reset-requests: 49s
x-ratelimit-limit-tokens: 1000000
x-ratelimit-remaining-tokens: 12000
x-ratelimit-reset-tokens: 5h12m30s
```

| En-tête                          | Contenu                                                    |
| -------------------------------- | ---------------------------------------------------------- |
| `x-ratelimit-limit-requests`     | Valeur de la limite de requêtes (RPM ou RPD)               |
| `x-ratelimit-remaining-requests` | Requêtes restantes dans la fenêtre                         |
| `x-ratelimit-reset-requests`     | Temps avant la réinitialisation de la fenêtre, ex. `49s`   |
| `x-ratelimit-limit-tokens`       | Valeur de la limite de tokens (TPM ou TPD)                 |
| `x-ratelimit-remaining-tokens`   | Tokens restants dans la fenêtre                            |
| `x-ratelimit-reset-tokens`       | Temps avant la réinitialisation de la fenêtre, ex. `5h12m30s` |

* **Fenêtre indiquée** : si une limite par minute et une limite par jour sont définies, les en-têtes indiquent celle où il vous reste le moins. Les en-têtes de requêtes et de tokens peuvent donc décrire des fenêtres différentes, comme dans l’exemple ci-dessus (requêtes par minute, tokens par jour).
* **Pas de limite** : si aucune limite de requêtes (ou de tokens) n’est définie pour le modèle, les en-têtes correspondants sont absents.

Le corps de la réponse contient aussi un champ `usage` avec le détail des tokens consommés par la requête :

```json
"usage": {
    "prompt_tokens": int,
    "completion_tokens": int,
    "total_tokens": int,
    "prompt_tokens_details": {
        "cached_tokens": int
    },
    "cost": int,
    "impacts": {
        "kWh": float,
        "kgCO2eq": float
    }
}
```

### Consulter l’historique de consommation

Deux moyens de consulter votre consommation cumulée :

{% tabs %}
{% tab title="Playground" icon="globe-pointer" %}
Connectez-vous sur le [Playground Albert API](https://albert.playground.etalab.gouv.fr) et ouvrez la page **Usage**.

<figure><img src="../.gitbook/assets/screenshots/playground-usage-page.png" alt="Page Usage du Playground Albert API"><figcaption>Page Usage du Playground</figcaption></figure>
{% endtab %}

{% tab title="API" icon="square-terminal" %}
Appelez l’endpoint [GET /v1/me/usage](https://guides.ia.numerique.gouv.fr/albert-api/api-reference/liste-des-endpoint/me#get-v1-me-usage) :

```bash
curl -X GET "https://albert.api.etalab.gouv.fr/v1/me/usage" \
  -H "Authorization: Bearer $API_KEY"
```

Pour le détail des paramètres et du format de réponse : [Usage & facturation](usage.md).
{% endtab %}
{% endtabs %}

### Dépassement de limites

Lorsqu’une de vos limites est atteinte, l’API répond **429 Too Many Requests** et le message d’erreur indique la limite dépassée. Stratégie recommandée : attendre la durée indiquée par `x-ratelimit-reset-requests` ou `x-ratelimit-reset-tokens` avant de réessayer, appliquer un **backoff exponentiel** et réduire le parallélisme.