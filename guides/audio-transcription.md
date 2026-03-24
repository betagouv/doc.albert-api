# Transcription audio

L’endpoint **`POST /v1/audio/transcriptions`** transcrit (ou traduit) un fichier audio à l’aide d’un modèle de type **`automatic-speech-recognition`**.

## Corps multipart

Le schéma attend un formulaire **`multipart/form-data`** avec notamment :

- **`file`** (requis) — flux binaire du média ;
- **`model`** (requis) — identifiant de modèle ASR ;
- **`language`** — code ISO-639-1 optionnel (par ex. `fr`, `en`) pour cibler la langue de sortie et réduire la latence ;
- **`prompt`** — consigne textuelle optionnelle pour guider le modèle ;
- **`response_format`** — `json`, `text` ou `verbose_json` ;
- **`temperature`** — tirage entre `0` et `1`.

Les formats de fichier explicitement mentionnés dans la spec sont **`mp3`** et **`wav`**.

{% hint style="warning" %}
⚠️ À vérifier — Autres formats MIME réellement acceptés en production (m4a, flac, etc.) : tester sur votre environnement ou consulter les notes opérationnelles associées au modèle.
{% endhint %}

## Exemple curl

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/audio/transcriptions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -F "file=@enregistrement.mp3" \
  -F "model=REMPLACER_PAR_MODELE_ASR" \
  -F "language=fr"
```

## Exemple Python (SDK OpenAI)

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

Pour le détail des champs de réponse (`verbose_json`, etc.), ouvrez le schéma `AudioTranscription` dans la [Référence OpenAPI](../api-reference/reference.md).
