# Collections et documents

Les **collections** regroupent des **documents** découpés en **chunks** indexés pour la recherche (`POST /v1/search`) et pour l’outil **`SearchTool`** du chat. C’est la base du RAG côté Albert API.

## Hiérarchie

```text
Collection
  └── Document
        └── Chunks (passages indexés, éventuellement enrichis de métadonnées)
```

## Collections (`/v1/collections`)

- **`GET /v1/collections`** — liste paginée ; filtres `name`, `visibility`, tri `order_by` / `order_direction`.
- **`POST /v1/collections`** — création avec corps JSON **`CollectionRequest`** :
  - `name` (requis) ;
  - `description` optionnelle ;
  - `visibility` : **`private`** (par défaut) ou **`public`** — les collections publiques sont visibles de tous les utilisateurs de la plateforme, les privées restent cantonnées au propriétaire.
- **`GET /v1/collections/{collection_id}`** — détail.
- **`PATCH /v1/collections/{collection_id}`** — mise à jour des métadonnées.
- **`DELETE /v1/collections/{collection_id}`** — suppression.

### Visibilité `public` vs `private`

- **`private`** — la collection et ses documents sont gérés par le propriétaire ; la recherche et le RAG s’appliquent à **votre** périmètre.
- **`public`** — la collection est **lisible** par les autres utilisateurs de la plateforme (recherche et récupération de chunks selon les règles d’accès). Seul le **propriétaire** peut modifier ou supprimer la collection et ses documents.

{% hint style="warning" %}
⚠️ À vérifier — La création de collections **publiques** peut être soumise à des **permissions** spécifiques côté plateforme (équivalent « create public collection » dans l’administration OpenGateLLM). Confirmer auprès de votre gestionnaire accès.
{% endhint %}

## Documents (`/v1/documents`)

- **`GET /v1/documents`** — liste et filtres (collection, pagination).
- **`POST /v1/documents`** — création, en **multipart/form-data** (`Body_create_document_v1_documents_post`) :
  - **`file`** — fichier source ; formats courants pour l’ingestion : **PDF**, **HTML**, **texte**, **Markdown** (extensions et types MIME acceptés selon le déploiement). Peut être omis pour créer un document vide puis le remplir via les routes de chunks ;
  - **`name`** — nom affiché ou substitution si pas de fichier ;
  - **`collection_id`** — rattachement à une collection ; le document sera vectorisé avec le modèle associé à cette collection ;
  - **`collection`** — alias **déprécié**, préférer `collection_id` ;
  - **Chunking** — découpage récursif façon [**RecursiveCharacterTextSplitter**](https://python.langchain.com/docs/how_to/recursive_text_splitter/) (LangChain) :
    - `disable_chunking` — désactive le découpage automatique ;
    - `chunk_size` — taille cible en **caractères** (défaut 2048 dans la spec) ;
    - `chunk_min_size` — taille minimale d’un chunk ;
    - `chunk_overlap` — chevauchement en caractères ;
    - `is_separator_regex` — traiter les séparateurs comme des expressions régulières ;
    - `separators` — liste de délimiteurs personnalisés ; si non vide, **`preset_separators` est ignoré** ;
    - `preset_separators` — énumération de jeux de séparateurs (`markdown`, `python`, `html`, etc.) ;
  - `metadata` — chaîne JSON optionnelle conforme au schéma `Metadata`, appliquée aux chunks lorsqu’un fichier est fourni.

- **`DELETE /v1/documents/{document_id}`** — suppression.

## Chunks

- **`GET /v1/documents/{document_id}/chunks`** — lecture des chunks d’un document.
- **`POST /v1/documents/{document_id}/chunks`** — création manuelle ou ajout de contenu lorsque le document a été créé sans fichier.
- **`GET/DELETE /v1/documents/{document_id}/chunks/{chunk_id}`** — lecture / suppression unitaire.

### Route chunks dépréciée

Les routes **`GET /v1/chunks/{document}`** et apparentées sont **dépréciées**. Utilisez les chemins sous **`/v1/documents/{document_id}/chunks`**.

## Recherche directe sur corpus (`POST /v1/search`)

Hors chat, le corps **`CreateSearch`** accepte une **`query`**, des `collection_ids`, `document_ids`, filtres de métadonnées, pagination (`limit`, `offset`), `method` (**`semantic`**, **`lexical`**, **`hybrid`**) et les paramètres **`rff_k`** / **`score_threshold`** comme pour `SearchTool`. Un paramètre de requête `required` (booléen, défaut `true`) modulateur est également documenté sur l’opération — se référer à la description OpenAPI pour la sémantique exacte.

Pour brancher la même logique dans le modèle : [Outil de recherche RAG intégré](rag-search-tool.md).
