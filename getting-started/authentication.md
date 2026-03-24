# Authentification

Albert API utilise le schéma de sécurité **HTTPBearer** : chaque requête authentifiée doit inclure un jeton dans l’en-tête `Authorization`.

## Obtenir un jeton

Les jetons d’API sont créés via l’endpoint `POST /v1/me/keys`. La procédure détaillée (création, liste, révocation) est décrite dans [Clés API](../account/api-keys.md).

## Format de l’en-tête

```text
Authorization: Bearer <votre_jeton>
```

Ne commitez jamais de jeton dans un dépôt Git ; utilisez des variables d’environnement ou un gestionnaire de secrets.

## Exemple avec curl

Remplacez `VOTRE_JETON` et, si besoin, l’identifiant de modèle par des valeurs valides pour votre compte.

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/models" \
  -H "Authorization: Bearer VOTRE_JETON" \
  -H "Content-Type: application/json"
```

## Exemple avec le SDK OpenAI (Python)

Le SDK officiel `openai` accepte une URL de base personnalisée. Le paramètre `api_key` sert à transporter **votre jeton Albert** (ce n’est pas une clé du fournisseur OpenAI).

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

La clé publique reçue à la création d’une clé API Albert se place dans `ALBERT_API_KEY`. Toutes les routes documentées sous `/v1/...` sont alors accessibles avec le même mécanisme que pour l’API OpenAI.
