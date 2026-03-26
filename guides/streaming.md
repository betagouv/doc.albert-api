# Streaming

Pour le chat, le streaming s’active avec **`"stream": true`** sur **`POST /v1/chat/completions`**. Le serveur renvoie des **Server-Sent Events** (SSE) : chaque événement contient un fragment JSON (delta) ; le flux se termine par une ligne `data: [DONE]` comme sur l’API OpenAI.

## Paramètre `stream_options`

Lorsque `stream` est à `true`, vous pouvez renseigner **`stream_options`** pour affiner le comportement du flux (par exemple demander l’inclusion d’informations d’usage en fin de stream, sur le modèle des clients OpenAI).

```json
{
  "model": "votre-modele-text-generation",
  "messages": [{"role": "user", "content": "Bonjour"}],
  "stream": true,
  "stream_options": {
    "include_usage": true
  }
}
```

{% hint style="warning" %}
⚠️ À vérifier — Clés exactes supportées dans `stream_options` et compatibilité `include_usage` selon les modèles : consulter le schéma `CreateChatCompletion` dans la [page de l’endpoint Chat](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/chat).
{% endhint %}

## Exemples (curl / Python / JavaScript)

{% tabs %}
{% tab title="curl" %}
```bash
curl -N "https://albert.api.etalab.gouv.fr/v1/chat/completions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "votre-modele-text-generation",
    "messages": [{"role": "user", "content": "Raconte une phrase sur la météo."}],
    "stream": true
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

stream = client.chat.completions.create(
    model="votre-modele-text-generation",
    messages=[{"role": "user", "content": "Raconte une phrase sur la météo."}],
    stream=True,
)

for chunk in stream:
    delta = chunk.choices[0].delta
    if delta.content:
        print(delta.content, end="", flush=True)
print()
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const stream = await client.chat.completions.create({
  model: "votre-modele-text-generation",
  messages: [{ role: "user", content: "Raconte une phrase sur la météo." }],
  stream: true,
});

for await (const chunk of stream) {
  const delta = chunk.choices?.[0]?.delta;
  if (delta?.content) process.stdout.write(delta.content);
}
process.stdout.write("\n");
```
{% endtab %}
{% endtabs %}

## Bonnes pratiques

* **Timeouts** : les streams longs exigent des timeouts côté client adaptés au contexte réseau.
* **Reconnexion** : en cas de coupure, la requête doit être relancée ; il n’y a pas de reprise automatique standard sur un identifiant de stream.
* **Erreurs mid-stream** : traiter les erreurs HTTP comme sur les appels non streamés ; voir [Codes d'erreur](../ressources/errors.md).
