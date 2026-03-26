# OCR

L’endpoint **`POST /v1/ocr`** extrait du texte (et selon la configuration) des **annotations** à partir d’un **document** ou d’une **image** référencé(e) dans le corps JSON.

Pour le détail des clés et schémas : la [page d’endpoint OCR](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/ocr).

## Choisir un modèle OCR

Le modèle utilisé pour l’OCR doit correspondre à un type de modèle adapté (souvent `image-to-text` ou une variante liée à l’OCR).

Voir aussi :

* [Guide “Modèles (liste)”](models.md) pour récupérer `GET /v1/models` et filtrer par `type`.

{% hint style="warning" %}
⚠️ À vérifier — Le champ `type` exact et/ou l’alias du modèle OCR peut varier selon l’instance. Sélectionnez un `model` observé dans `GET /v1/models`.
{% endhint %}

## Paramètres importants

Champs courants :

* **`document`** (requis) — objet de type :
  * `{"type": "document_url", "document_url": "https://..."}` pour un PDF/document ;
  * `{"type": "image_url", "url": "https://..."}` pour une image.
* **`pages`** — liste d’indices de pages à traiter (indexation **à partir de 0**).
* **`include_image_base64`** — inclusion éventuelle d’images en base64 dans la réponse (utile si vous souhaitez archiver/visualiser).
* **`document_annotation_format`** / **`bbox_annotation_format`** — format de sortie pour texte/annotations (logique proche de `response_format` côté chat : `text`, `json_object`, `json_schema`, etc.).

{% hint style="warning" %}
⚠️ Les indices de page sont **à partir de 0** (page 0 = première page).
{% endhint %}

## Exemple : OCR d’un PDF par URL (document_url)

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_OCR",
    "document": {
      "type": "document_url",
      "document_url": "https://www.princexml.com/samples/magic6/magic.pdf"
    },
    "include_image_base64": true
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os
import requests

server_url = "https://albert.api.etalab.gouv.fr"
api_key = os.environ["ALBERT_API_KEY"]

headers = {"Authorization": f"Bearer {api_key}"}

model = "REMPLACER_PAR_MODELE_OCR"
pdf_url = "https://www.princexml.com/samples/magic6/magic.pdf"

resp = requests.post(
    url=f"{server_url}/v1/ocr",
    headers=headers,
    json={
        "model": model,
        "document": {"type": "document_url", "document_url": pdf_url},
        "include_image_base64": True,
    },
)
resp.raise_for_status()
data = resp.json()

# Exemples de champs attendus (selon la config/format) :
# print(data["pages"][0]["markdown"])
print("Pages disponibles :", len(data.get("pages", [])))
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const resp = await fetch("https://albert.api.etalab.gouv.fr/v1/ocr", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "REMPLACER_PAR_MODELE_OCR",
    document: {
      type: "document_url",
      document_url: "https://www.princexml.com/samples/magic6/magic.pdf",
    },
    include_image_base64: true,
  }),
});

if (!resp.ok) throw new Error(await resp.text());
const data = await resp.json();
console.log("Pages disponibles :", data.pages?.length ?? 0);
```
{% endtab %}
{% endtabs %}

## Variante : OCR d’un PDF encodé en base64 (data URL)

{% tabs %}
{% tab title="curl" %}
```bash
PDF_BASE64=$(curl -sS "https://www.princexml.com/samples/textbook/somatosensory.pdf" | base64 -w 0)

curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"REMPLACER_PAR_MODELE_OCR\",
    \"document\": {
      \"type\": \"document_url\",
      \"document_url\": \"data:application/pdf;base64,${PDF_BASE64}\"
    },
    \"include_image_base64\": true
  }"
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import os
import requests

server_url = "https://albert.api.etalab.gouv.fr"
api_key = os.environ["ALBERT_API_KEY"]

headers = {"Authorization": f"Bearer {api_key}"}
model = "REMPLACER_PAR_MODELE_OCR"

pdf_url = "https://www.princexml.com/samples/textbook/somatosensory.pdf"
pdf_bytes = requests.get(pdf_url).content
pdf_base64 = base64.b64encode(pdf_bytes).decode("utf-8")

resp = requests.post(
    url=f"{server_url}/v1/ocr",
    headers=headers,
    json={
        "model": model,
        "document": {
            "type": "document_url",
            "document_url": f"data:application/pdf;base64,{pdf_base64}",
        },
        "include_image_base64": True,
    },
)
resp.raise_for_status()
data = resp.json()

print("Markdown (ex.) :", (data.get("pages", [{}])[0]).get("markdown", "")[:200])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const pdfBuffer = await (await fetch("https://www.princexml.com/samples/textbook/somatosensory.pdf")).arrayBuffer();
const pdfBase64 = Buffer.from(pdfBuffer).toString("base64");

const resp = await fetch("https://albert.api.etalab.gouv.fr/v1/ocr", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "REMPLACER_PAR_MODELE_OCR",
    document: {
      type: "document_url",
      document_url: `data:application/pdf;base64,${pdfBase64}`,
    },
    include_image_base64: true,
  }),
});

if (!resp.ok) throw new Error(await resp.text());
const data = await resp.json();
console.log("Markdown (ex.) :", (data.pages?.[0]?.markdown ?? "").slice(0, 200));
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
⚠️ À vérifier — Support “data URLs” (format `data:application/pdf;base64,...`) selon votre instance et votre politique d’ingestion.
{% endhint %}

## Exemple : OCR d’une image par URL (image_url)

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/ocr" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_OCR",
    "document": {
      "type": "image_url",
      "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg"
    },
    "include_image_base64": true
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os
import requests

server_url = "https://albert.api.etalab.gouv.fr"
api_key = os.environ["ALBERT_API_KEY"]

headers = {"Authorization": f"Bearer {api_key}"}

model = "REMPLACER_PAR_MODELE_OCR"
image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg"

resp = requests.post(
    url=f"{server_url}/v1/ocr",
    headers=headers,
    json={
        "model": model,
        "document": {"type": "image_url", "url": image_url},
        "include_image_base64": True,
    },
)
resp.raise_for_status()
data = resp.json()

print("Markdown (ex.) :", (data.get("pages", [{}])[0]).get("markdown", "")[:200])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const resp = await fetch("https://albert.api.etalab.gouv.fr/v1/ocr", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.ALBERT_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "REMPLACER_PAR_MODELE_OCR",
    document: {
      type: "image_url",
      url: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/ReceiptSwiss.jpg/1280px-ReceiptSwiss.jpg",
    },
    include_image_base64: true,
  }),
});

if (!resp.ok) throw new Error(await resp.text());
const data = await resp.json();
console.log("Markdown (ex.) :", (data.pages?.[0]?.markdown ?? "").slice(0, 200));
```
{% endtab %}
{% endtabs %}

## Dépréciation de `parse-beta`

L’endpoint **`POST /v1/parse-beta`** est **déprécié**. Pour tout nouveau développement, utilisez **`POST /v1/ocr`**.

{% hint style="warning" %}
⚠️ À vérifier — quotas spécifiques, tailles maximales de document et formats d’URL autorisés (HTTP/HTTPS, authentification) peuvent varier selon l’offre.
{% endhint %}
