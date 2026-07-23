---
icon: ranking-star
---

# Reranking

Albert API propose un endpoint POST `/v1/rerank` qui permet d'appeler des modèles de reranking. Ces modèles sont particulièrement intéressants pour améliorer votre pipeline de RAG (Retrieval Augmented Generation).

{% hint style="info" %}
Albert API propose un endpoint POST `/v1/rerank` sur les convention de l'API v2 de Cohere (voir [Cohere API - Reranking](https://docs.cohere.com/reference/rerank)). Cette convention permet de s'intégrer nativement dans la plupart des frameworks de IA générative comme par exemple [OpenWebUI](https://openwebui.com/).
{% endhint %}

**Dans ce tutoriel nous allons voir :**

* **pourquoi utiliser un modèle de reranking,**
* **qu'est-ce qu'un modèle reranking**,
* **quand l'utiliser**
* **comment l'utiliser.**

## Prérequis

* Connaissance des notions de RAG, de vector-store, d'embeddings et de chunks
* Connaissance de l'upload et de la recherche de documents avec Albert API.

Pour en savoir plus sur ces notions, voir le guide Construire un RAG avec Albert API _(à venir)_.

## Pourquoi utiliser le reranking ?

Lorsque vous construisez un système de RAG, vous devez :

1. Rechercher des documents pertinents dans un vector store
2. Sélectionner les meilleurs passages
3. Les envoyer au modèle de génération

Le problème : la recherche vectorielle retourne souvent **des résultats approximatifs**. 

En effet, que ce soit avec une recherche sémantique, lexicale ou hybride, **la recherche dans un vector store est une recherche à larges mailles**. Elle permet ainsi de retrouver les _chunks_ qui gravitent autour de la requête recherchée mais cela peut prendre dans le filet des chunks relativement éloignés contextuellement.

En conséquence, un moteur vectoriel est très rapide, mais il peut retourner :

* des résultats **un peu hors sujet**
* des passages **moins pertinents que d'autres**
* des résultats dans **un ordre sous-optimal**

Ainsi si certains résultats de recherche sont pertinents, d'autres beaucoup moins. C'est précisément le rôle de l'endpoint **`/v1/rerank`** : **réordonner les résultats de recherche pour ne garder que les passages les plus pertinents pour une requête.**

***

## Qu'est ce que le reranking ?

Le **reranking** utilise un petit modèle puissant pour :

* comparer **la requête utilisateur**
* avec **chaque document trouvé**
* et produire **un score de pertinence**

Albert API propose un modèle de reranking _**openweight-rerank**_. Pour plus d'informations sur ce modèle, voir les [modèles disponibles](https://albert.sites.beta.gouv.fr/solutions/models/).

Pour connaître quels sont les modèles de reranking disponibles, faites une requête au endpoint `/v1/models` . Parmi ceux retourné, les modèles de reranking ont le type `text-classification`.

{% hint style="info" %}
Un modèle de reranking est généralement un modèle de classification de quelques millions de paramètres spécialement entraîné sur des paires questions-documents. Lors de son entrainement le modèle a appris à inférer sur la base de millions d'exemples des patterns permettant d'attribuer un score à la pertinence d'un document vis-à-vis d'une question.
{% endhint %}

## Quand utiliser le reranking ?

Le reranking est utile lorsque :

* vous avez beaucoup de documents
* le contexte du LLM est limité (ex : openweight-large)
* les capacités du LLM à analyser un grand contexte sont limitées (ex : openweight-small)

***

## Comment utiliser le reranking ?

### Fonctionnement général

L'endpoint `/v1/rerank` prend :

* une **query**
* une **liste de documents**

et retourne les documents **triés par pertinence.** Chaque document reçoit un **score de pertinence**.

### Pipeline de RAG avec le reranking

Si vous souhaitez un RAG avec 5 chunks pour augmenter le contexte de la requête utilisateur, alors dans un premier temps récupérez 40 chunks proches avec `/v1/search`. Puis vous affinez avec `/v1/rerank` pour obtenir les 5 chunks les plus pertinents.

<pre class="language-mermaid"><code class="lang-mermaid"><strong>graph TD
</strong>  user_request(Requête utilisateur)
  response(Réponse)
  
  
  user_request 
  --> /v1/search 
  --"40 chunks" --> /v1/rerank 
  -- "5 chunks" --> /v1/chat/completions 
  --> response
  
  
</code></pre>

### Étape 1. Recherche vectorielle

On commence par récupérer les documents les plus proches de la requête utilisateur.

{% tabs %}
{% tab title="Python" %}
```python
import os
import requests

response = requests.post(
    url="https://api.albert.etalab.gouv.fr/v1/rerank",
    headers={"Authorization": f"Bearer {os.environ.get('ALBERT_API_KEY')}"},
    json={
        "query": "Comment déployer une application Docker ?",
        "method": "semantic",
        "limit": 40,
    },
)

data = response.json()["data"]
chunks = [chunk.content for chunk in data.chunk]
```
{% endtab %}
{% endtabs %}

À ce stade, les documents sont **proches sémantiquement**, mais pas forcément les plus pertinents pour répondre précisément à la question.

C’est pourquoi on ajoute une étape de **reranking**.

{% hint style="tip" %}
Le reranking est utile quelque soit la méthode de recherche utilisée (sémantique, lexicale, hybride). Attention toutefois, il peut être inutile, voir contre-productif. Voir les [bonnes pratiques](#bonnes-pratiques) pour plus d'informations.
{% endhint %}

### Étape 2. Reranking

Le reranker réévalue les documents retournés par la recherche vectorielle afin de déterminer **les plus pertinents pour la requête**.

{% tabs %}
{% tab title="Python" %}
```python
import os
import requests

user_query = "Comment déployer une application Docker ?"

response = requests.post(
    url="https://api.albert.etalab.gouv.fr/v1/rerank",
    headers={"Authorization": f"Bearer {os.environ.get('ALBERT_API_KEY')}"},
    json={
        "model": "openweight-rerank",
        "query": user_query,
        "documents": chunks,
        "top_n": 5,
    },
)
data = response.json()
```
{% endtab %}
{% endtabs %}

**Réponse**

```json
{
  "results": [
    {
      "index": 2, // Position du document dans la liste d'entrée
      "relevance_score": 0.98 // Score de pertinence calculé par le modèle
    },
    ...
  ]
}
``` 

### Étape 3. Reconstruction des documents triés

On récupère les documents dans le bon ordre à partir des indices retournés.

{% tabs %}
{% tab title="Python" %}
```
ranked_chunks = [chunks[result["index"]] for result in data["results"]]
```
{% endtab %}
{% endtabs %}

### Étape 4. Génération avec le contexte

On injecte les documents les plus pertinents dans le prompt du LLM comme _openweight-small_ par exemple.

{% tabs %}
{% tab title="Python" %}
```python
rag_template = """Réponds à la question en te basant sur les documents suivants. 
Si la réponse à la question n'est pas  dans les documents, indique que tu ne 
t'es pas appuyé sur ces documents pour générer la réponse, sinon cite les 
sources de ta réponse.

[QUESTION]
{user_query}

[DOCS]
{chunks}
"""

query = rag_template % {"user_query": user_query, "chunks": "\n\n".join(chunks)}

response = requests.post(
    url="https://api.albert.etalab.gouv.fr/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ.get('ALBERT_API_KEY')}"},
    json={
        "model": "openweight-small",
        "messages": [{"role": "user", "content": query}],
    },
)
data = response.json()
```
{% endtab %}
{% endtabs %}

Le modèle utilise alors **le contexte fourni pour générer une réponse plus fiable et précise**.

***

## Bonnes pratiques

### 1. Reranker plus de documents que nécessaire

Pipeline recommandée :
* Search : limit = 20 à 50
* Rerank : top_n = 3 à 10

Cela permet au reranker de choisir les documents les plus pertinents parmi un ensemble suffisamment large.

### 2. Utiliser des chunks courts

La qualité du reranking dépend fortement de la taille des documents. Nous recommandons 100 à 500 tokens par chunk.

Les documents trop longs diluent l’information importante et réduisent la précision du reranker.

### 3. Le reranking peut être inutile ou contre-productif

Le reranking est un outil puissant, mais il ne faut pas l'utiliser à chaque fois. 
Vous devez déterminer si le reranking est nécessaire pour votre cas d'usage en testant des requêtes sur vos documents. Par exemple, la recherche hybride peut obtenir des scores de pertinence dans certains cas plus précis que le reranking. Dans ce cas, ce dernier va éliminer des documents pertinents.

Comme pour toute technique, il faut tester et mesurer les performances à l'aide d'un processus de d'évaluation.
