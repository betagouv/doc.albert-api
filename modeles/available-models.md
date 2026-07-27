

# Modèles disponibles

Albert API offre l’accès à une large gamme de modèle fondation open source d’IA générative. Nous mettons à jour régulièrement les modèles disponibles pour vous permettre d’utiliser les modèles qui font l’état de l’art.
 
| Modèle | Type | Aliases |
| --- | --- | --- |
| BAAI/bge-m3 | `text-embeddings-inference` | **openweight-embeddings** |
| BAAI/bge-reranker-v2-m3 | `text-classification` | **openweight-rerank** |
| deepseek-v4-flash | `text-generation` | deepseek-ai/DeepSeek-V4-Flash |
| lightonocr-2-1b | `image-text-to-text` | **openweight-ocr**, lighton/LightOn-OCR-2-1B |
| mistralai/Ministral-3-8B-Instruct-2512 | `image-text-to-text` | **openweight-small** |
| mistralai/Mistral-Small-3.2-24B-Instruct-2506 | `image-text-to-text` | **openweight-medium** |
| mistral-medium-2508 | `text-generation` | |
| mistral-ocr-2512 | `image-to-text` | |
| [openai/gpt-oss-120b](#openai-gpt-oss-120b) | `text-generation` | **openweight-large** |
| openai/whisper-large-v3 | `automatic-speech-recognition` | **openweight-audio** |
| qwen3-vl-embedding-8b | `text-embeddings-inference` | Qwen/Qwen3-VL-Embedding-8B |
| Qwen/Qwen3-Coder-30B-A3B-Instruct | `text-generation` | **openweight-code** |

{% hint style="info" %}
Pour en savoir plus sur les types de modèles, voir [Types de modèles](model-types.md).
{% endhint %}

Les recommandations d'usage ci-dessousproviennent de l'expérience des utilisateurs et ne sont pas garanties. Nous vous recommandons de tester les modèles différents modèles et leurs paramètres pour trouver ceux qui s'adapteront le mieux à vos besoins.

Vos retours nous sont précieux pour améliorer les modèles et les recommandations d'usage. N'hésitez pas nous les partager sur notre canal Tchap ou par mail (voir la page [Contact](https://ia.numerique.gouv.fr/outils-ia/albert-api/albert-api-contact/)).

## BAAI/bge-m3

*🏗️ En construction...*

| | | |
| --- | --- | --- |
| **Model card** | https://huggingface.co/BAAI/bge-m3 |
| **Nombre de paramètres** | 568M |
| **Type** | `text-embeddings-inference` |
| **Aliases** | `openweight-embeddings` |
| **Format d'entrées** | `text` |
| **Fenêtre de contexte** | 8192 |
| **Maximum de texte par batch** | 64 |


## BAAI/bge-reranker-v2-m3

*🏗️ En construction...*

| | | |
| --- | --- | --- |
| **Model card** | https://huggingface.co/BAAI/bge-reranker-v2-m3 |
| **Nombre de paramètres** | 568M |
| **Type** | `text-classification` |
| **Aliases** | openweight-rerank |
| **Format d'entrées** | `text` |
| **Fenêtre de contexte** | 8192 |
| **Maximum de texte par batch** | 64 |

## deepseek-v4-flash

*🏗️ En construction...*

| | | |
| --- | --- | --- |
| **Model card** | https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash |
| **Nombre de paramètres** | 284B |
| **Nombre de paramètres actifs** | 13B |
| **Type** | `image-text-to-text` |
| **Aliases** | deepseek-ai/DeepSeek-V4-Flash |
| **Format d'entrées** | `text` |
| **Fenêtre de contexte** | 393216 |
| **Fonctionnalités** | `tool calling`, `reasoning` |

## lightonocr-2-1b

*🏗️ En construction...*

| | | |
| --- | --- | --- |
| **Model card** | https://huggingface.co/lightonai/LightOnOCR-2-1B |
| **Nombre de paramètres** | 284B |
| **Nombre de paramètres actifs** | 13B |
| **Type** | `image-text-to-text` |
| **Aliases** | lighton/LightOn-OCR-2-1B |
| **Format d'entrées** | `text` |
| **Fenêtre de contexte** | 393216 |
| **Fonctionnalités** | `tool calling`, `reasoning` |

Nous observons certaines latences du modèle lorsque certains partern de pixel sont présents dans l'image (un QR code par exemple). Si vous observez des latences élevées, vous pouvez essayer de supprimer ces patterns de pixel de l'image avant de l'envoyer au modèle.

## mistralai/Ministral-3-8B-Instruct-2512

*🏗️ En construction...*

## mistralai/Mistral-Small-3.2-24B-Instruct-2506

*🏗️ En construction...*

## mistral-medium-2508

*🏗️ En construction...*

## mistral-ocr-2512

*🏗️ En construction...*

## openai/gpt-oss-120b

*🏗️ En construction...*

* `openai/gpt-oss-120b`

| Propriété | Valeur | Remarques |
| --- | --- | --- |
| Type | `text-generation` |
| Aliases | `openweight-large` |
| Fenêtre de contexte | 1024 |
| Température recommandée (`temperature`) | 0 |
| Maximum de tokens (`max_tokens`)| > 1024 | Même pour des résponses courtes, si le paramètre `max_tokens` est trop petit, les tokens de raisonnement consomme le maximum de tokens défini. |

| Capacités | | Remarques|
| --- | --- | --- | 
| Chat | Oui | Modèle pour des analyses complexes. | 
| Embeddings | Non |
| Analyse d'image | Non |
| Audio transcription | Non |
| Rerank | Non |
|  | Non |
| OCR | Non |

## openai/whisper-large-v3

*🏗️ En construction...*

## qwen3-vl-embedding-8b

*🏗️ En construction...*

## Qwen/Qwen3-Coder-30B-A3B-Instruct

*🏗️ En construction...*