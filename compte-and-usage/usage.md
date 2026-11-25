# Usage & facturation

L’endpoint **`GET /v1/usage`** retourne l’historique de consommation du compte authentifié, regroupé **par jour** (UTC).


## Paramètres de requête

* **`start_time`** (requis) — timestamp Unix de **début** de la période.
* **`end_time`** (requis) — timestamp Unix de **fin** de la période.
* **`endpoint`** — filtre optionnel sur un endpoint : `/v1/audio/transcriptions`, `/v1/chat/completions`, `/v1/embeddings`, `/v1/ocr` ou `/v1/rerank`.
* **`model`** — filtre optionnel sur un modèle, par son nom tel que listé par `GET /v1/models` (les alias ne sont pas reconnus).
* **`key_id`** — filtre optionnel sur une clef d’API, par son identifiant (voir [Clefs API](api-keys.md)).
* **`offset`**, **`limit`** — pagination des jours renvoyés. Par défaut `offset=0` et `limit=10` ; `limit` vaut au maximum 100.

**Exemple de requête :** consommation de novembre 2026 sur `/v1/chat/completions`

```bash
curl -G "https://albert.api.etalab.gouv.fr/v1/usage" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -d start_time=1793491200 \
  -d end_time=1796083199 \
  -d endpoint=/v1/chat/completions
```

## Format de la réponse

```json
{
    "object": "list",
    "total": 2,
    "offset": 0,
    "limit": 10,
    "data": [
        {
            "object": "usage.bucket",
            "start_time": 1795564800,
            "end_time": 1795651200,
            "prompt_tokens": 182340,
            "completion_tokens": 24510,
            "total_tokens": 206850,
            "cost": 0.0,
            "requests": 214,
            "impacts": {
                "kWh": 0.0123,
                "kgCO2eq": 0.0008
            }
        },
        {
            "object": "usage.bucket",
            "start_time": 1795478400,
            "end_time": 1795564800,
            ...
        }
    ]
}
```

Chaque élément de `data` correspond à **un jour UTC** :

* **`start_time`**, **`end_time`** — début et fin (exclue) du jour, en timestamp Unix ;
* **`prompt_tokens`**, **`completion_tokens`**, **`total_tokens`** — somme des tokens consommés ce jour-là. Les tokens en cache ne sont pas enregistrés à part : `prompt_tokens` les inclut sans les distinguer ;
* **`requests`** — nombre de requêtes réussies ;
* **`cost`** — somme des coûts des requêtes, calculée avec les coûts par million de tokens du modèle (champ `costs` de `GET /v1/models`) ;
* **`impacts`** — estimation de l’empreinte environnementale : **`kWh`** (énergie consommée) et **`kgCO2eq`** (émissions de gaz à effet de serre).


{% hint style="info" %}
Les valeurs d’**empreinte environnementale** sont estimées par Albert API avec la méthodologie **EcoLogits**, à partir des caractéristiques du modèle, de sa zone d’hébergement, du nombre de tokens générés et de la durée de la requête. Elles restent **indicatives**, mais sont utiles pour les bilans environnementaux des services qui s’appuient sur Albert API.
{% endhint %}

Pour le détail complet des paramètres et du schéma de réponse, voir la [page de l’endpoint Usage](https://guides.ia.numerique.gouv.fr/albert-api/api-reference/liste-des-endpoint/usage#get-v1-usage).

Pour les plafonds appliqués à votre compte : [Quotas & limites](quotas.md).
