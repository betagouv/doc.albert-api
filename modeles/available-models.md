# Modèles disponibles

Albert API offre l’accès à une large gamme de modèles fondation open source d’IA générative. Nous mettons à jour régulièrement les modèles disponibles pour vous permettre d’utiliser les modèles qui font l’état de l’art.

{% hint style="info" %}
Chaque modèle est associé à un type de modèle. Pour en savoir plus sur les types de modèles, voir [Types de modèles](model-types.md).
{% endhint %}

Les recommandations d'usage ci-dessous proviennent de l'expérience des utilisateurs et ne sont pas garanties. Nous vous recommandons de tester différents modèles et leurs paramètres pour trouver ceux qui s'adapteront le mieux à vos besoins.

Vos retours nous sont précieux pour améliorer les modèles et les recommandations d'usage. N'hésitez pas à nous les partager sur notre canal Tchap ou par mail (voir la page [Contact](https://ia.numerique.gouv.fr/outils-ia/albert-api/albert-api-contact/)).

<details>

<summary>BAAI/bge-m3</summary>

Model d'embeddings de référence pour la vectorisation de texte.

<a href="https://huggingface.co/BAAI/bge-m3" class="button secondary">Model card</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 568M |
| **Type** | `text-embeddings-inference` |
| **Aliases** | `openweight-embeddings` |
| **Licence** | MIT |
| **Fenêtre de contexte** | 8192 |
| **Dimension maximale des vecteurs** | 1024 |
| **Maximum de texte par batch** | 64 |

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Text vectorisation</strong></td>
      <td>/v1/embeddings</td>
      <td><a href="../guides/embeddings.md">embeddings.md</a></td>
      <td><a href="../.gitbook/assets/icons/arrow-up-1-9.svg">arrow-up-1-9.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>BAAI/bge-reranker-v2-m3</summary>

Modèle de reranking multilingue basé sur `bge-m3`. Il est le modèle de référence pour le reranking sur Albert API.

<a href="https://huggingface.co/BAAI/bge-reranker-v2-m3" class="button secondary">Model card</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 568M |
| **Type** | `text-classification` |
| **Licence** | MIT |
| **Aliases** | `openweight-rerank` |
| **Fenêtre de contexte** | 8192 |
| **Maximum de texte par batch** | 64 |

{% hint style="success" icon="lightbulb" %}
La limite architecturale est de 8192 tokens, toutefois le modèle a été finetuné pour être utilisé avec des textes de longueur maximale de 1024 tokens. Nous recommandons de ne pas dépasser cette longueur lors de la construction des chunks pour le reranking.
{% endhint %}

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Reranking</strong></td>
      <td>/v1/reranking</td>
      <td><a href="../guides/reranking.md">reranking.md</a></td>
      <td><a href="../.gitbook/assets/icons/ranking-star.svg">ranking-star.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>deepseek-v4-flash</summary>

MoE spécialisé pour les tâches de code.

<a href="https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash" class="button secondary">Model card</a>
<a href="https://albert.playground.etalab.gouv.fr" class="button primary">Essayer dans Playground</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 284B |
| **Nombre de paramètres actifs** | 13B |
| **Licence** | MIT |
| **Type** | `text-generation` |
| **Aliases** | `deepseek-ai/DeepSeek-V4-Flash` |
| **Fenêtre de contexte** | 393216 |
| **Température recommandée** | 1.0 |

{% hint style="warning" %}
Ce modèle est en phase d'expérimentation du 26 juillet 2026 au 1er octobre 2026. Candidat au remplacement du modèle `openweight-code` à l'issue de cette période.

Ce modèle extra-européen peut présenter certaines formes de censure des réponses. Nous recommandons de l'utiliser uniquement pour les cas d'usage de génération de code et non comme un modèle conversationnel généraliste.

Cette phase d'expérimentation a notamment pour objectif d'évaluer l'impact de ces limitations afin de déterminer si elles sont compatibles avec une intégration pérenne dans Albert API. Vos retours sur ce sujet seront particulièrement précieux.
{% endhint %}

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Code</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/ide.md">ide.md</a></td>
      <td><a href="../.gitbook/assets/icons/terminal.svg">terminal.svg</a></td>
    </tr>
    <tr>
      <td><strong>Reasoning</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/reasoning.md">reasoning.md</a></td>
      <td><a href="../.gitbook/assets/icons/brain.svg">brain.svg</a></td>
    </tr>
    <tr>
      <td><strong>Tool calling</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/function-calling.md">function-calling.md</a></td>
      <td><a href="../.gitbook/assets/icons/screwdriver-wrench.svg">screwdriver-wrench.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>lightonocr-2-1b</summary>

Modèle multimodal spécialisé pour l'OCR. Il est le modèle de référence pour l'OCR sur Albert API.

<a href="https://huggingface.co/lightonai/LightOnOCR-2-1B" class="button secondary">Model card</a>
<a href="https://albert.playground.etalab.gouv.fr" class="button primary">Essayer dans Playground</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 1B |
| **Licence** | `Apache 2.0` |
| **Type** | `image-text-to-text` |
| **Aliases** | `lighton/LightOn-OCR-2-1B`, `openweight-ocr` |
| **Fenêtre de contexte** | 16384 |

{% hint style="success" icon="lightbulb" %}
Nous observons certaines latences lorsque certains patterns de pixels sont présents dans l'image (un QR code par exemple). Si vous observez des latences élevées, essayez de supprimer ces patterns avant d'envoyer l'image au modèle.
{% endhint %}

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>OCR (chat)</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/ocr.md">ocr.md</a></td>
      <td><a href="../.gitbook/assets/icons/images.svg">images.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>mistralai/Ministral-3-8B-Instruct-2512</summary>

Modèle multimodal généraliste de petite taille, idéal pour des tâches simples comme de la classification ou de correction de texte.

<a href="https://huggingface.co/mistralai/Ministral-3-8B-Instruct-2512" class="button secondary">Model card</a>
<a href="https://albert.playground.etalab.gouv.fr" class="button primary">Essayer dans Playground</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 8.9B |
| **Type** | `text-generation` |
| **Licence** | `Apache 2.0` |
| **Aliases** | `mistralai/Ministral-3-8B-Instruct-2512`, `openweight-small` |
| **Format d'entrées** | `text`, `image` |
| **Fenêtre de contexte** | 262144 |
| **Température recommandée** | 0.1  |

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Chat</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/chat-completions.md">chat-completions.md</a></td>
      <td><a href="../.gitbook/assets/icons/message.svg">message.svg</a></td>
    </tr>
    <tr>
      <td><strong>Analyse d'image</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/chat-completions.md">chat-completions.md</a></td>
      <td><a href="../.gitbook/assets/icons/eye.svg">eye.svg</a></td>
    </tr>
    <tr>
      <td><strong>Tool calling</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/function-calling.md">function-calling.md</a></td>
      <td><a href="../.gitbook/assets/icons/screwdriver-wrench.svg">screwdriver-wrench.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>mistral-small-3-2-24b-instruct-2506</summary>

<a href="https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2506" class="button secondary">Model card</a>
<a href="https://albert.playground.etalab.gouv.fr" class="button primary">Essayer dans Playground</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 24B |
| **Type** | `text-generation` |
| **Licence** | `Apache 2.0` |
| **Aliases** | `mistralai/Mistral-Small-3.2-24B-Instruct-2506`, `openweight-medium` |
| **Format d'entrées** | `text`, `image` |
| **Fenêtre de contexte** | 131072 |
| **Température recommandée** | 0.15 |

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Chat</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/chat-completions.md">chat-completions.md</a></td>
      <td><a href="../.gitbook/assets/icons/message.svg">message.svg</a></td>
    </tr>
    <tr>
      <td><strong>Analyse d'image</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/chat-completions.md">chat-completions.md</a></td>
      <td><a href="../.gitbook/assets/icons/eye.svg">eye.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>mistral-medium-2508</summary>

{% hint style="warning" %}
Ce modèle accessible uniquement aux ministères partenaires qui en font la demande. Pour en savoir plus, consultez la page [Tarifs et limites](https://ia.numerique.gouv.fr/outils-ia/albert-api/tarifs-et-limites/) de notre site.
{% endhint %}

<a href="https://docs.mistral.ai/models/model-cards/mistral-medium-3-1-25-08" class="button secondary">Model card</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 123B |
| **Type** | `text-generation` |
| **Licence** | `propriétaire` |
| **Aliases** | `mistral-medium-2508` |
| **Format d'entrées** | `text`, `image` |
| **Fenêtre de contexte** | 131072 |

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Chat</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/chat-completions.md">chat-completions.md</a></td>
      <td><a href="../.gitbook/assets/icons/message.svg">message.svg</a></td>
    </tr>
    <tr>
      <td><strong>Analyse d'image</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/chat-completions.md">chat-completions.md</a></td>
      <td><a href="../.gitbook/assets/icons/eye.svg">eye.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>mistral-ocr-2512</summary>

{% hint style="warning" %}
Ce modèle accessible uniquement aux ministères partenaires qui en font la demande. Pour en savoir plus, consultez la page [Tarifs et limites](https://ia.numerique.gouv.fr/outils-ia/albert-api/tarifs-et-limites/) de notre site.
{% endhint %}

<a href="https://mistral.ai/news/mistral-ocr-3/" class="button secondary">Model card</a>
<a href="https://albert.playground.etalab.gouv.fr" class="button primary">Essayer dans Playground</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 3B |
| **Type** | `image-to-text` |
| **Licence** | `propriétaire` |
| **Aliases** | `mistral-ocr-2512` |
| **Fenêtre de contexte** | 16384 |

**Mistral OCR 3**, accessible via `POST /v1/ocr` (accès restreint selon les comptes). Pour la plupart des usages OCR, préférer `openweight-ocr` via `/v1/chat/completions` — voir le guide [OCR](../guides/ocr.md).

### Capacités

</details>

<details>

<summary>gpt-oss-120b</summary>
<a href="https://huggingface.co/openai/gpt-oss-120b" class="button secondary">Model card</a>
<a href="https://albert.playground.etalab.gouv.fr" class="button primary">Essayer dans Playground</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 117B |
| **Nombre de paramètres actifs** | 5.1B |
| **Type** | `text-generation` |
| **Aliases** | `openai/gpt-oss-120b`, `openweight-large` |
| **Fenêtre de contexte** | 131072 |

MoE open weight (OpenAI). Les tokens de raisonnement consomment le budget `max_tokens` : prévoyez une marge confortable même pour des réponses courtes.

### Capacités

</details>

<details>

<summary>whisper-large-v3</summary>
<a href="https://huggingface.co/openai/whisper-large-v3" class="button secondary">Model card</a>
<a href="https://transcripts.numerique.gouv.fr/" class="button primary">Essayer dans Transcripts</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 1.55B |
| **Type** | `automatic-speech-recognition` |
| **Licence** | `Apache 2.0` |
| **Aliases** | `openai/whisper-large-v3`, `openweight-audio` |
| **Fenêtre de contexte** | Fichier audio jusqu'à 20 MB |

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Transcription</strong></td>
      <td>/v1/audio/transcriptions</td>
      <td><a href="../guides/audio-transcriptions.md">audio-transcriptions.md</a></td>
      <td><a href="../.gitbook/assets/icons/microphone.svg">microphone.svg</a></td>
    </tr>
    <tr>
      <td><strong>Diarization</strong></td>
      <td>/v1/audio/transcriptions</td>
      <td><a href="../guides/audio-transcriptions.md">audio-transcriptions.md</a></td>
      <td><a href="../.gitbook/assets/icons/comments.svg">comments.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>qwen3-vl-embedding-8b</summary>
<a href="https://huggingface.co/Qwen/Qwen3-VL-Embedding-8B" class="button secondary">Model card</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 8B |
| **Type** | `text-embeddings-inference` |
| **Licence** | `Apache 2.0` |
| **Aliases** | `Qwen/Qwen3-VL-Embedding-8B`|
| **Fenêtre de contexte** | 32768 |
| **Dimension maximale des vecteurs** | 4096* |

Embeddings multimodaux (texte, images, captures d’écran, vidéos). Ne mélangez pas ces vecteurs avec ceux de `openweight-embeddings` : ce ne sont pas le même espace.

*La dimension des vecteurs n'est pas modifiable avec ce modèle.

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Text vectorisation</strong></td>
      <td>/v1/embeddings</td>
      <td><a href="../guides/embeddings.md">embeddings.md</a></td>
      <td><a href="../.gitbook/assets/icons/arrow-up-1-9.svg">arrow-up-1-9.svg</a></td>
    </tr>
  </tbody>
</table>

</details>

<details>

<summary>qwen3-coder-30b-a3b-instruct</summary>
<a href="https://huggingface.co/Qwen/Qwen3-Coder-30B-A3B-Instruct" class="button secondary">Model card</a>
<a href="https://albert.playground.etalab.gouv.fr" class="button primary">Essayer dans Playground</a>

| | |
| --- | --- |
| **Nombre de paramètres** | 30.5B |
| **Nombre de paramètres actifs** | 3.3B |
| **Type** | `text-generation` |
| **Licence** | `Apache 2.0` |
| **Aliases** | `Qwen/Qwen3-Coder-30B-A3B-Instruct`, `openweight-code` |
| **Fenêtre de contexte** | 262144 |

MoE orienté code / agentic coding (function calling).

### Capacités

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
      <th data-hidden data-card-cover data-type="files"></th>
    </tr>
    <tr>
      <td><strong>Code</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/ide.md">ide.md</a></td>
      <td><a href="../.gitbook/assets/icons/terminal.svg">terminal.svg</a></td>
    </tr>
    <tr>
      <td><strong>Tool calling</strong></td>
      <td>/v1/chat/completions</td>
      <td><a href="../guides/function-calling.md">function-calling.md</a></td>
      <td><a href="../.gitbook/assets/icons/screwdriver-wrench.svg">screwdriver-wrench.svg</a></td>
    </tr>
  </thead>
  <tbody>

</details>
