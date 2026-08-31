---
icon: rocket
---

# Démarrage rapide

Pour envoyer des requêtes à Albert API vous devez posséder créer une clef d'API depuis l'interface [Playground](https://albert.playground.etalab.gouv.fr/) d'Albert API.&#x20;

{% hint style="info" %}
Une clef d'API est un identifiant de connexion (_Bearer token_) permettant de vous authentifier lors de vos appels API. Elle commence par `sk-eyJhbG...`.
{% endhint %}

{% stepper %}
{% step %}
### Se connecter au playground

Rendez vous sur [https://albert.playground.etalab.gouv.fr](https://albert.playground.etalab.gouv.fr/) et connectez vous avec Proconnect ([en savoir plus](https://www.proconnect.gouv.fr/)). \
\
Si vous êtes redirigez sur une page vous indiquant que votre accès est refusé, merci de consulter [la section de la FAQ dédiée](../ressources/faq.md#pourquoi-mon-compte-nest-pas-autorise).
{% endstep %}

{% step %}
### Créer une clef d'API

Une fois connectez à l'interface Playground, rendez-vous sur la page [_API Keys_](https://albert.playground.etalab.gouv.fr/keys). Sur cette page créer vous une clef en lui attribuant un nom. Vous pouvez configurer la durée d'expiration de votre clef jusqu'à un an.&#x20;

<figure><img src="../.gitbook/assets/Screenshot 2026-08-19 at 11.16.03.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Après création, la clé est affichée **une seule fois**. Copiez-la immédiatement et conservez-la dans un gestionnaire de secrets (ou variable d’environnement), car **vous ne pourrez pas l'afficher** ensuite.
{% endhint %}
{% endstep %}

{% step %}
### Appeler un modèle

Pour cette première requête nous allons appelez le modèle `openweight-small`. Ouvrez un terminal et exécutez le code ci-dessous.&#x20;

{% hint style="info" %}
Remplacez la valeur `$ALBERT_API_KEY` par la valeur de votre clef API.&#x20;
{% endhint %}

{% tabs %}
{% tab title="Bash" icon="square-terminal" %}
```bash
curl -x POST "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "REMPLACER_PAR_ID_MODELE",
    "messages": [
      {"role": "system", "content": "Tu réponds en français, de façon concise."},
      {"role": "user", "content": "Explique ce qu’est une API compatible OpenAI en deux phrases."}
    ]
  }'

```
{% endtab %}

{% tab title="Python" icon="python" %}
```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key=os.environ["ALBERT_API_KEY"],
)

r = client.chat.completions.create(
    model="REMPLACER_PAR_ID_MODELE",
    messages=[
        {"role": "system", "content": "Tu réponds en français, de façon concise."},
        {"role": "user", "content": "Explique ce qu’est une API compatible OpenAI en deux phrases."},
    ],
)
print(r.choices[0].message.content)
```
{% endtab %}

{% tab title="JavaScript" icon="js" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const r = await client.chat.completions.create({
  model: "REMPLACER_PAR_ID_MODELE",
  messages: [
    { role: "system", content: "Tu réponds en français, de façon concise." },
    { role: "user", content: "Explique ce qu’est une API compatible OpenAI en deux phrases." }
  ],
});

console.log(r.choices[0].message.content);
```
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### Explorer les fonctionnalités et les modèles disponibles

* [Consulter les endpoints disponibles](https://app.gitbook.com/s/3Kt6ArO8RlWHs4cxZZIu/api-reference)
* [Consulter les modèles disponibles](../modeles/available-models.md)&#x20;
* [Consulter nos guides thématiques](https://app.gitbook.com/s/3Kt6ArO8RlWHs4cxZZIu/guides)
{% endstep %}
{% endstepper %}
