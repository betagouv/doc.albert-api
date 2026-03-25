# Reranking

L’endpoint **`POST /v1/rerank`** réordonne une liste de documents textuels selon leur pertinence par rapport à une **requête**, via un modèle de type **`text-classification`**.

## Corps de requête (`CreateRerank`)

* **`query`** (requis) — texte de la question ou de la requête utilisateur.
* **`documents`** (requis) — tableau de chaînes non vides à classer.
* **`model`** (requis) — identifiant d’un modèle `text-classification` listé sous `/v1/models`.
* **`top_n`** — nombre maximal de résultats à retourner ; si absent, la spec indique que **tous** les documents peuvent être renvoyés réordonnés.

La description OpenAPI mentionne que **`query`** et **`prompt`** ne peuvent pas être fournis ensemble ; seul `query` apparaît comme propriété documentée dans le schéma public — en cas de doute, ne passez que `query`.

## Intégration typique

1. **Recherche large** — `POST /v1/search` ou moteur texte classique pour récupérer un ensemble candidat.
2. **Rerank** — `POST /v1/rerank` pour affiner l’ordre avant envoi au modèle de chat ou affichage utilisateur.

## Exemple minimal

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/rerank" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_MODELE_RERANK",
    "query": "Quelle est la capital de la France ?",
    "documents": [
      "Paris est la capitale.",
      "Le croissant est une viennoiserie.",
      "Lyon est une ville française."
    ],
    "top_n": 2
  }'
```

La structure exacte de la réponse (scores, indices) est définie dans le schéma de réponse associé à l’opération `rerank` dans la [Référence OpenAPI](/broken/pages/vRZwVxl3vzeW4GecHnw7).
