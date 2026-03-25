# Collections et documents

Les **collections** regroupent des **documents** découpés en **chunks** indexés. Cette structure est la base du RAG côté Albert API :

* recherche directe via `POST /v1/search` ;
* recherche native via **`SearchTool`** dans `POST /v1/chat/completions`.

## Hiérarchie

```
Collection
  └── Document
        └── Chunks (passages indexés, éventuellement enrichis de métadonnées)
```

## Collections (`/v1/collections`)

* **`GET /v1/collections`** — liste paginée ; filtres `name`, `visibility`, tri `order_by` / `order_direction`.
* **`POST /v1/collections`** — création avec corps JSON `CollectionRequest` :
  * `name` (requis) ;
  * `description` optionnelle ;
  * `visibility` : `private` (par défaut) ou `public`.
* **`GET /v1/collections/{collection_id}`** — détail.
* **`PATCH /v1/collections/{collection_id}`** — mise à jour des métadonnées.
* **`DELETE /v1/collections/{collection_id}`** — suppression.

### Visibilité `public` vs `private`

* **`private`** : vos documents restent dans votre périmètre ; la recherche et le RAG s’appliquent à votre compte/organisation.
* **`public`** : la collection est lisible par les autres utilisateurs (recherche et récupération de chunks selon les règles d’accès). Seul le propriétaire peut modifier/supprimer.

{% hint style="warning" %}
⚠️ À vérifier — La création de collections **publiques** peut être soumise à une permission spécifique côté plateforme. Confirmez auprès de votre gestionnaire accès.
{% endhint %}

## Documents (`/v1/documents`)

* **`GET /v1/documents`** — liste et filtres (collection, pagination).
* **`POST /v1/documents`** — création en **multipart/form-data** :
  * **`file`** : fichier source, typiquement **PDF**, **HTML**, **texte**, **Markdown** (formats et MIME acceptés selon le déploiement). Peut être omis si vous créez un document vide puis ajoutez des chunks ensuite ;
  * **`name`** : nom affiché (utile si vous n’envoyez pas `file`) ;
  * **`collection_id`** : collection cible (préféré) ;
  * **`collection`** : alias **déprécié** (préférez `collection_id`) ;
  * **Chunking** (découpage récursif type `RecursiveCharacterTextSplitter`) :
    * `disable_chunking` — désactive le découpage automatique (conceptuellement proche du “pas de splitter” de certains anciens tutoriels) ;
    * `chunk_size` — taille cible en **caractères** ;
    * `chunk_min_size` — taille minimale d’un chunk ;
    * `chunk_overlap` — chevauchement en caractères ;
    * `is_separator_regex` — traite `separators` comme des regex ;
    * `separators` — liste de délimiteurs (si non vide, `preset_separators` est ignoré) ;
    * `preset_separators` — presets de délimiteurs (ex. `markdown`, `python`, `html`).
  * **`metadata`** : chaîne JSON optionnelle, appliquée aux chunks lors de l’ingestion (à fournir sous forme stringifiée).
* **`DELETE /v1/documents/{document_id}`** — suppression.

{% hint style="warning" %}
⚠️ Métadonnées : `metadata` est attendu comme une **chaîne JSON** dans le multipart. Exemple :
`-F 'metadata={"source":"rapport","year":2024}'`.
{% endhint %}

### Exemple minimal : créer une collection et uploader un document

```bash
# 1) Collection
curl -sS "https://albert.api.etalab.gouv.fr/v1/collections" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "exemples",
    "visibility": "private"
  }'

# 2) Document
curl -sS "https://albert.api.etalab.gouv.fr/v1/documents" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -F "file=@mon-dossier.pdf" \
  -F "collection_id=REMPLACER_PAR_COLLECTION_ID" \
  -F "chunk_size=2048" \
  -F "chunk_overlap=200"
```

## Chunks

* **`GET /v1/documents/{document_id}/chunks`** — lecture des chunks.
* **`POST /v1/documents/{document_id}/chunks`** — ajout manuel si le document a été créé sans `file`.
* **`GET/DELETE /v1/documents/{document_id}/chunks/{chunk_id}`** — lecture / suppression unitaire.

### Route chunks dépréciée

Les routes **`GET /v1/chunks/{document}`** et apparentées sont **dépréciées**. Utilisez **`/v1/documents/{document_id}/chunks`**.

## Recherche directe sur corpus (`POST /v1/search`)

Hors chat, `CreateSearch` accepte une **`query`**, `collection_ids`, `document_ids`, des filtres de métadonnées, `limit` / `offset`, `method` (`semantic`, `lexical`, `hybrid`) et les paramètres `rff_k` / `score_threshold`.

Pour brancher cette même logique au modèle : [Outil de recherche RAG intégré](rag-search-tool.md).
