# Transcription audio

L’endpoint **`POST /v1/audio/transcriptions`** transcrit (et selon les modèles, traduit) un fichier audio via un modèle de type **`automatic-speech-recognition`**.

## Corps multipart

Le schéma attend un formulaire **`multipart/form-data`** avec notamment :

* **`file`** (requis) — contenu binaire du média ;
* **`model`** (requis) — identifiant du modèle ASR ;
* **`language`** — code ISO-639-1 optionnel (par ex. `fr`, `en`) ;
* **`prompt`** — consigne textuelle optionnelle ;
* **`response_format`** — `json`, `text` ou `verbose_json` ;
* **`temperature`** — tirage entre `0` et `1`.

Les formats explicitement mentionnés dans la spec sont **`mp3`** et **`wav`**.

{% hint style="warning" %}
⚠️ À vérifier — Autres formats MIME réellement acceptés en production (m4a, flac, etc.) : testez sur votre environnement.
{% endhint %}

## Choisir `response_format`

Par défaut, vous récupérez généralement une forme “texte” (selon SDK). Si vous avez besoin d’une sortie structurée, passez par `response_format`.

```bash
curl -sS "https://albert.api.etalab.gouv.fr/v1/audio/transcriptions" \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  -F "file=@enregistrement.mp3" \
  -F "model=REMPLACER_PAR_MODELE_ASR" \
  -F "language=fr" \
  -F "response_format=json"
```

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
        # response_format="json",  # optionnel
    )
print(tr.text)
```

Pour le détail des champs de réponse (`verbose_json`, etc.), voir la page de l’endpoint **Audio** :
[page de l’endpoint Audio](https://doc.incubateur.net/alliance/albert-api/api-reference/liste-des-endpoint/audio).
