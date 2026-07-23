# Modèles disponibles

## Caractéristiques des modèles

Albert API propose plusieurs modèles avec des capacités différentes. Pour connaitre les caractéristiques de chaque modèle, rendez-vous sur notre guide [Modèles (liste)](../guides/models.md).

## Recommandations d'usage des modèles

{ % hint style="warning" %}
Les recommandations d'usage proviennent de l'expérience des utilisateurs et ne sont pas garanties. Nous vous recommandons de tester les modèles différents modèles et leurs paramètres pour trouver ceux qui s'adapteront le mieux à vos besoins.

Vos retours nous sont précieux pour améliorer les modèles et les recommandations d'usage. N'hésitez pas nous les partager sur notre canal Tchap ou par mail (voir la page [Contact](https://ia.numerique.gouv.fr/outils-ia/albert-api/albert-api-contact/)).
{ % endhint %}

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
