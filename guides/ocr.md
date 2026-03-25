# OCR

L’endpoint **`POST /v1/ocr`** extrait du texte (et, selon votre configuration, des annotations/structure) à partir d’un **document** ou d’une **image** référencés par URL dans le corps JSON.

Pour le détail des clés et schémas, voir la page d’endpoint **OCR** :
https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/ocr

## Exemple minimal (document PDF par URL)

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_OCR",
    "document": {
      "type": "document_url",
      "document_url": "https://example.org/document.pdf"
    }
  }'
```

## Variante : OCR d’une image (URL)

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_OCR",
    "document": {
      "type": "image_url",
      "url": "https://example.org/image.png"
    },
    "include_image_base64": true
  }'
```

## Pages, images et indexation

La réponse retourne typiquement une liste de **pages**.

{% hint style="warning" %}
⚠️ Les indices de page sont **à partir de 0** (page 0 = première page).
{% endhint %}

Selon les options, la réponse peut aussi inclure des images (par exemple `images[]`) et leurs coordonnées de type bbox.

## Contrôler la sortie

Champs courants (voir `CreateOCR`) :

* **`pages`** — liste d’indices de pages à traiter uniquement.
* **`image_limit`** / **`image_min_size`** — contrôle du nombre de vues/images produites.
* **`include_image_base64`** — inclusion éventuelle des images en base64 (utile pour visualiser/archiver).
* **`document_annotation_format`** / **`bbox_annotation_format`** — format de sortie pour texte/annotations (même logique que `response_format` côté chat : `text`, `json_object`, `json_schema`, etc.).

## Dépréciation de `parse-beta`

L’endpoint **`POST /v1/parse-beta`** est **déprécié**. Pour tout nouveau développement, utilisez **`POST /v1/ocr`**.

{% hint style="warning" %}
⚠️ À vérifier — Quotas spécifiques, tailles maximales de document et formats d’URL autorisés (HTTP/HTTPS, authentification) peuvent varier selon l’offre. Validez sur votre compte.
{% endhint %}
