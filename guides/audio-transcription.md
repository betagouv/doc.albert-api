# Transcription audio

L’endpoint **`POST /v1/audio/transcriptions`** transcrit un fichier audio via un modèle de type **`automatic-speech-recognition`** ([en savoir plus sur les types de modèles](../modeles/model-types.md)).

## Transcrire un fichier audio

Le endpoint `/v1/audio/transcriptions` supporte les fichiers `mp3` et `wav` jusqu'à 20 Mb par fichier. 

Pour plus d'information sur les paramètres du endpoint, voir la page [API reference - Audio](https://guides.ia.numerique.gouv.fr/albert-api/api-reference/liste-des-endpoint/audio).

**Exemple de requête :**
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/audio/transcriptions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -F "file=@enregistrement.mp3" \
  -F "model=REMPLACER_PAR_MODELE_ASR" \
  -F "language=fr" \
  -F "response_format=json"
```

## Exemples (curl / Python / JavaScript)

{% tabs %}
{% tab title="curl" %}
```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/audio/transcriptions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -F "file=@enregistrement.mp3" \
  -F "model=REMPLACER_PAR_MODELE_ASR" \
  -F "language=fr"
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

with open("enregistrement.mp3", "rb") as f:
    tr = client.audio.transcriptions.create(
        model="REMPLACER_PAR_MODELE_ASR",
        file=f,
        language="fr",
    )
print(tr.text)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import fs from "node:fs";
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://albert.api.etalab.gouv.fr/v1",
  apiKey: process.env.ALBERT_API_KEY,
});

const tr = await client.audio.transcriptions.create({
  model: "REMPLACER_PAR_MODELE_ASR",
  file: fs.createReadStream("enregistrement.mp3"),
  language: "fr",
});

console.log(tr.text);
```
{% endtab %}
{% endtabs %}

## Choisir `response_format`

Le paramètre `response_format` permet de choisir le format de la réponse. Ce format dépend de l'usage que vous souhaitez en faire.

| `response_format` | Description |
|---|---|
| `json` (defaut) | Réponse au format JSON, contient le texte transcrit. |
| `text` | Réponse au format TXT, contient uniquement le texte transcrit. |
| `verbose_json` | Réponse au format JSON en indiquant le début et la fin de chaque phrase. |
| `srt` | Réponse au format SRT. |
| `vtt` | Réponse au format VTT. |
| `diarized_json` | Réponse au format JSON en indiquant le début et la fin de chaque phrase et les changements de speakers. Ce format utilisé pour les transcriptions des réunions.|
