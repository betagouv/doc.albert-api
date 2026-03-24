# OCR

L’endpoint **`POST /v1/ocr`** extrait du texte (et éventuellement de la structure) à partir d’un **document** ou d’une **image** référencés par URL dans le corps JSON. La forme des blocs **`document`** est **proche des conventions Mistral** (types `document_url` / URL d’image) ; pour le détail des clés, ouvrez le schéma `CreateOCR` dans la [Référence OpenAPI](../api-reference/reference.md).

## Exemple minimal (PDF distant)

```http
POST /v1/ocr HTTP/1.1
Host: albert.api.etalab.gouv.fr
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "model": "identifiant-modele-image-to-text-ou-vision",
  "document": {
    "type": "document_url",
    "document_url": "https://example.org/document.pdf"
  }
}
```

Vous pouvez aussi utiliser un **client Mistral** compatible en pointant `server_url` vers `https://albert.api.etalab.gouv.fr` et la même clé Bearer, si votre stack l’impose déjà.

## Corps de requête (`CreateOCR`)

Champs notables :

- **`document`** (requis) — soit un bloc `DocumentURLChunk`, soit un bloc `ImageURLChunk` (voir la spec pour les clés exactes `type` / `url`).
- **`model`** — identifiant du modèle OCR ; si `null`, le comportement par défaut dépend de la configuration plateforme.
- **`pages`** — liste d’entiers (indices de pages, **à partir de 0**) pour ne traiter qu’un sous-ensemble de pages.
- **`image_limit`**, **`image_min_size`**, **`include_image_base64`** — contrôle du nombre d’images extraites, taille minimale et inclusion éventuelle en base64 dans la réponse.
- **`document_annotation_format`**, **`bbox_annotation_format`** — `response_format` au même titre que pour le chat : `text`, `json_object`, `json_schema`, etc., pour contraindre la sortie.

## Cas d’usage

- Numérisation de PDF administratifs pour alimenter un RAG.
- Exploitation de captures d’écran ou scans dans un flux entièrement API.

## Dépréciation de `parse-beta`

L’endpoint **`POST /v1/parse-beta`** est **déprécié**. Pour tout nouveau développement, utilisez **`POST /v1/ocr`**.

La réponse suit les schémas `OCR`, `OCRPageObject`, etc., décrits dans la [Référence OpenAPI](../api-reference/reference.md).

{% hint style="warning" %}
⚠️ À vérifier — Quotas spécifiques, tailles maximales de document et formats d’URL autorisés (HTTP, HTTPS, authentification) peuvent varier selon l’offre ; valider sur votre compte.
{% endhint %}
