# Usage & facturation

L’endpoint **`GET /v1/me/usage`** retourne l’historique de consommation du compte authentifié.

## Paramètres de requête

* **`start_time`** — timestamp Unix **début** ; si absent, la spec indique un défaut à **30 jours avant** l’instant présent.
* **`end_time`** — timestamp Unix **fin** ; si absent, défaut à **maintenant**.
* **`endpoint`** — filtre optionnel sur un **sous-ensemble** d’endpoints :\
  `/v1/audio/transcriptions`, `/v1/chat/completions`, `/v1/embeddings`, `/v1/ocr`, `/v1/rerank`, `/v1/search`.
* **`offset`**, **`limit`** — pagination des enregistrements d’usage.

## Objet `Usage` (élément de liste)

Chaque entrée peut inclure notamment :

* **`model`**, **`key`**, **`endpoint`**, **`method`**, **`status`** — contexte de la requête ;
* **`created`** — horodatage Unix (secondes) ;
* **`usage`** — objet **`UsageDetail`** avec agrégats de jetons, coût, empreinte carbone et métriques.

## `UsageDetail`

* **`prompt_tokens`**, **`completion_tokens`**, **`total_tokens`** — compteurs de jetons ;
* **`cost`** — coût total attribué à l’appel (unité et politique internes à la plateforme) ;
* **`carbon`** — objet **`CarbonFootprintUsage`** avec :
  * **`kWh`** — `min` / `max` ;
  * **`kgCO2eq`** (objet min/max d’équivalent **CO₂**, GWP).
* **`metrics`** — champs comme **`latency`**, **`ttft`** (temps jusqu’au premier token) lorsque disponibles.

{% hint style="info" %}
Le volet **empreinte carbone** (kWh et kgCO₂eq avec fourchettes min/max) est une particularité utile pour les bilans environnementaux des services qui s’appuient sur Albert API. Côté moteur OpenGateLLM, les estimations peuvent s’appuyer sur des bibliothèques du type **EcoLogits** ; les valeurs retournées par l’API restent **indicatives** et dépendent des paramètres de modèle et d’hébergement configurés sur la plateforme.
{% endhint %}

## Conteneur de réponse

La réponse **`Usages`** renvoie un tableau `data` d’objets `Usage`. Pour le détail des champs optionnels, ouvrez les schémas `Usages`, `Usage` et `UsageDetail` dans la page de l’endpoint **Me** :
https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/me

Pour les plafonds appliqués à votre compte : [Quotas & limites](quotas.md).
