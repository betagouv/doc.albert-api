# Modèles disponibles

## Lister les modèles — `GET /v1/models`

Retourne la liste des modèles exposés sur l’instance Albert à laquelle vous êtes connectés. Chaque entrée suit le schéma **`Model`** :

* **`id`** — identifiant à passer aux endpoints (`model` des requêtes chat, embeddings, ASR, rerank, OCR…) ;
* **`type`** — famille fonctionnelle (`text-generation`, `text-embeddings-inference`, etc.) ;
* **`aliases`** — autres noms acceptés pour référencer le même modèle ;
* **`max_context_length`** — taille maximale de contexte en **tokens** ;
* **`costs`** — tarification indicielle (`prompt_tokens`, `completion_tokens` **par million de tokens** pour la composante correspondante).

## Détail — `GET /v1/models/{model}`

Récupère la fiche d’un modèle précis à partir de son **identifiant ou d’un alias** accepté par la plateforme.

## Choisir un modèle

1. Filtrez mentalement par **`type`** selon l’endpoint (voir [Types de modèles](model-types.md)).
2. Vérifiez **`max_context_length`** pour dimensionner prompts + sortie.
3. Comparez **`costs`** si votre compte est soumis à budget (voir aussi [Quotas & limites](../compte-and-usage/quotas.md)).

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/models" \
  -H "Authorization: Bearer $ALBERT_API_KEY"
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

models = client.models.list().data
for model in models:
    print(model.id, model.type)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const models = await client.models.list();
for (const model of models.data) {
  console.log(model.id, model.type);
}
```
{% endtab %}
{% endtabs %}

Les modèles réellement déployés varient selon l’environnement (recette, production interministérielle, etc.) : la liste HTTP fait **foi** pour votre intégration.
