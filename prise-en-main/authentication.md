# Authentification

Albert API utilise le schéma de sécurité **HTTPBearer** : chaque requête authentifiée doit inclure un jeton dans l’en-tête `Authorization`.

## Obtenir un jeton

Les jetons d’API sont créés via l’endpoint `POST /v1/me/keys`. La procédure détaillée (création, liste, révocation) est décrite dans [Clés API](../compte-and-usage/api-keys.md).

## Format de l’en-tête

```
Authorization: Bearer <votre_jeton>
```

Ne commitez jamais de jeton dans un dépôt Git ; utilisez des variables d’environnement ou un gestionnaire de secrets.

## Exemples (curl / Python / JavaScript)

Remplacez `VOTRE_JETON` et, si besoin, l’identifiant de modèle par des valeurs valides pour votre compte.

Le SDK officiel `openai` accepte une URL de base personnalisée. Le paramètre `api_key` sert a transporter **votre jeton Albert** (ce n’est pas une clé du fournisseur OpenAI).

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/models" \
  -H "Authorization: Bearer VOTRE_JETON" \
  -H "Content-Type: application/json"
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

models = client.models.list()
print([m.id for m in models.data])
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
console.log(models.data.map((m) => m.id));
```
{% endtab %}
{% endtabs %}

La clé publique reçue à la création d’une clé API Albert se place dans `ALBERT_API_KEY`. Toutes les routes documentées sous `/v1/...` sont alors accessibles avec le même mécanisme que pour l’API OpenAI.
