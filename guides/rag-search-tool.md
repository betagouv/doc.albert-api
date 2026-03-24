# Outil de recherche RAG intégré (SearchTool)

Albert API propose un **outil de recherche natif** utilisable directement dans le tableau **`tools`** de **`POST /v1/chat/completions`**. Il interroge des **chunks** indexés à partir de vos **collections** et **documents**, puis fournit des passages pertinents au modèle pour **ancrer** la réponse dans votre corpus.

{% hint style="warning" %}
⚠️ Ce mécanisme **n’est pas** une fonctionnalité standard OpenAI. Les clients et tutoriels génériques « OpenAI tools » ne le décrivent pas : intégrez explicitement un objet d’outil avec `type: "search"` et les paramètres ci-dessous.
{% endhint %}

## Schéma usuel dans `tools`

Ajoutez un objet respectant le schéma **`SearchTool`** (voir OpenAPI). Champs principaux :

| Champ | Description |
| --- | --- |
| `type` | Doit valoir la constante **`"search"`** pour déclencher l’outil intégré. |
| `collection_ids` | Liste d’identifiants de collections (entiers strictement positifs), jusqu’à 100 entrées. |
| `document_ids` | Liste optionnelle d’identifiants de documents pour restreindre le périmètre. |
| `metadata_filters` | Filtres sur les métadonnées des chunks (`ComparisonFilter` ou `CompoundFilter` dans la spec). |
| `limit` | Nombre maximal de résultats (par défaut 10, max 100). |
| `offset` | Pagination : nombre de résultats à sauter. |
| `method` | **`semantic`**, **`lexical`** ou **`hybrid`**. |
| `rff_k` | Constante de lissage pour la fusion RRF en recherche **hybride** (valeur par défaut 60 ; plage documentée jusqu’à 16384, recommandations souvent entre 10 et 100). |
| `score_threshold` | Seuil de similarité cosinus (0 à 1), applicable à la recherche **sémantique** — 0 désactive le filtrage par score. |

## Pipeline recommandé

1. **Créer une collection** — `POST /v1/collections` avec un nom et une visibilité `private` ou `public` (voir [Collections & documents](collections-documents.md)).
2. **Ingérer des documents** — `POST /v1/documents` en multipart (fichier + `collection_id`, paramètres de chunking).
3. **Appeler le chat avec SearchTool** — incluez l’objet `type: "search"` dans `tools` et laissez le modèle invoquer l’outil, ou forcez via `tool_choice` selon votre scénario.

En parallèle, l’endpoint **`POST /v1/search`** permet d’appeler la même logique de recherche **sans** passer par le modèle de chat (utile pour du debugging ou des UIs de recherche).

## Parcours type : RAG « manuel » (recherche puis chat)

Sans `SearchTool`, vous pouvez enchaîner **(1)** une recherche sur les chunks, **(2)** une construction de prompt avec les passages retournés, **(3)** un appel à **`POST /v1/chat/completions`**. C’est l’équivalent du tutoriel historique « semantic search + prompt ».

1. **Recherche** — `POST https://albert.api.etalab.gouv.fr/v1/search` avec un corps JSON aligné sur **`CreateSearch`** (ex. `query`, `collection_ids`, `method`, `limit`, `rff_k` si `hybrid`). Les résultats contiennent des chunks (texte + métadonnées).
2. **Prompt** — concaténez le contenu des chunks pertinents dans un message `user` (ou `system`) en rappelant la question initiale.
3. **Chat** — `POST /v1/chat/completions` avec un modèle `text-generation` uniquement.

Côté Python, enchaînez `requests` (ou `httpx`) pour la recherche et le SDK `openai` pour le chat, comme dans les notebooks du dépôt [opengatellm](https://github.com/etalab-ia/opengatellm/tree/main/docs/tutorials).

{% hint style="warning" %}
⚠️ Les anciens exemples qui utilisaient des champs du type `collections` / `prompt` au lieu de `collection_ids` / `query`, ou un chemin `/search` sans préfixe `/v1`, sont **obsolètes** : alignez-vous sur la [Référence OpenAPI](../api-reference/reference.md).
{% endhint %}

{% hint style="warning" %}
⚠️ À vérifier — Détails JSON exacts des filtres `metadata_filters` et comportement lorsque `collection_ids` et `document_ids` sont vides : se reporter aux schémas `ComparisonFilter`, `CompoundFilter` et `SearchTool` dans la [Référence OpenAPI](../api-reference/reference.md).
{% endhint %}

## Ancienne intégration chat (`search` / `search_args`)

Les champs **`search`** et **`search_args`** au niveau racine du corps de chat sont **dépréciés** au profit de ce pipeline explicite (collections + `SearchTool`). Ne les utilisez pas pour de nouvelles intégrations.
