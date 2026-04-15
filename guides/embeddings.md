# Embeddings

L’endpoint **`POST /v1/embeddings`** calcule des vecteurs pour du texte (ou des entrées tokenisées) à l’aide d’un modèle dont le type est **`text-embeddings-inference`**.

## Requête

Champs principaux du schéma **`EmbeddingsRequest`** :

* **`model`** : identifiant d’un modèle d’embeddings (obtenu via `GET /v1/models`) ;
* **`input`** : chaîne unique ou **liste de chaînes** (batch) ; chaque entrée doit respecter la limite du modèle (**`max_context_length`** côté `Model`) ;
* **`dimensions`** : dimension vectorielle souhaitée (si supportée par le modèle) ;
* **`encoding_format`** : format des valeurs (en pratique, `float` est le format attendu).

## Réponse

La réponse suit le format OpenAI embeddings : `data[]` contient des objets avec `embedding` et `index` ; et un objet `usage` de facturation/monitoring.

Pour les schémas exacts, voir la [page de l’endpoint Embeddings](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/embeddings).

## Exemples (curl / Python / JavaScript)

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/embeddings" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_EMBEDDINGS",
    "input": [
      "Le chat est un animal domestique.",
      "La France a pour capitale Paris."
    ]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

r = client.embeddings.create(
    model="REMPLACER_PAR_MODELE_EMBEDDINGS",
    input=["Première phrase.", "Deuxième phrase."],
)

for item in r.data:
    print("index=", item.index, "dim=", len(item.embedding))
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const r = await client.embeddings.create({
  model: "REMPLACER_PAR_MODELE_EMBEDDINGS",
  input: ["Première phrase.", "Deuxième phrase."],
});

for (const item of r.data) {
  console.log("index=", item.index, "dim=", item.embedding.length);
}
```
{% endtab %}
{% endtabs %}

## Bon à savoir

Si votre objectif est de faire de la recherche RAG “bout en bout”, vous pouvez aussi éviter de gérer un index vectoriel vous-même : utilisez `POST /v1/search` et/ou `SearchTool`, voir [Génération Augmentée par Récupération (RAG)](rag.md).
