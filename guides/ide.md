# Utiliser un modèle de code dans un IDE

Albert API est compatible “OpenAI-like” sur l’endpoint de chat : vous pouvez donc brancher des modèles (notamment des modèles orientés code) sur des IDE et agents qui parlent l’API OpenAI.

{% hint style="warning" %}
⚠️ À vérifier — Ces plugins/éditeurs supportent parfois uniquement un sous-ensemble des champs OpenAI. Testez avec un `model` de type `text-generation` et l’URL base `.../v1`.
{% endhint %}

## Visual Studio Code (Continue)

[Continue](https://marketplace.visualstudio.com/items?itemName=Continuestral.continue) est une extension VS Code.

Après installation, configurez le fichier `.continue/config.yml` :

```yaml
name: Local Assistant
version: 1.0.0
schema: v1
models:
  - name: Albert API
    provider: openai
    model: <VOTRE_MODEL_ID>
    apiBase: https://albert.api.etalab.gouv.fr/v1
    apiKey: <VOTRE_API_KEY>
    roles:
      - chat

context:
  - provider: docs-legacy
  - provider: terminal
  - provider: problems
  - provider: folder
  - provider: codebase
  - provider: code
```

## IntelliJ / PyCharm (ProxyAI)

[ProxyAI](https://plugins.jetbrains.com/plugin/21056-proxyai) est un plugin JetBrains.

Après installation, configurez `~/.config/ProxyAI/config.yml` :

```yaml
name: Albert API
API Key: <VOTRE_API_KEY>
URL: https://albert.api.etalab.gouv.fr/v1
model: <VOTRE_MODEL_ID>
```

## Zed

[Zed](https://zed.dev/) permet de configurer des providers “OpenAI”.

Procédure (résumé) :

1. Ouvrir le panneau “Agent Panel”
2. Dans “Model”, sélectionner “Configure”
3. Provider : “OpenAI”
4. Renseigner `API Key` : `<VOTRE_API_KEY>`
5. Renseigner l’URL : `<YOUR_API_URL>` (typiquement `https://albert.api.etalab.gouv.fr/v1`)

Exemple de snippet (format JSON) :

```json
"language models": {
  "openai": {
    "api_url": "https://albert.api.etalab.gouv.fr/v1",
    "api_key": "<VOTRE_API_KEY>",
    "role": "chat",
    "available_models": [
      {
        "name": "<VOTRE_MODEL_ID>",
        "display_name": "<VOTRE_MODEL_ID>",
        "supports_tools": true
      }
    ],
    "version": "1"
  }
}
```

## Cursor (Kilo Code AI Agent)

Le plugin “Kilo Code AI Agent” (Cursor) se configure en “Custom OpenAI”.

Exemple :

```yaml
API Key: <VOTRE_API_KEY>
URL: https://albert.api.etalab.gouv.fr/v1/chat/completions
model: <VOTRE_MODEL_ID>
```

Si vous voulez valider que le `model` que vous sélectionnez correspond au bon type, commencez par la liste des modèles : [Guide “Modèles”](models.md).

Pour une vue “publique” des modèles (familles, licences, alias) vous pouvez aussi consulter :

[Catalogue des modèles](https://albert.sites.beta.gouv.fr/solutions/models/)

