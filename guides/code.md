---
icon: "terminal"
---
# Coder avec Albert API

Albert API expose une API compatible OpenAI. Vous pouvez donc configurer la plupart des agents de code (en terminal et dans votre éditeur) pour qu'ils utilisent les modèles souverains de l'État hébergés sur AlbertAPI

Cette page liste les configurations pour les principaux outils actuels.

## Principe général

Albert API répond sur `POST /v1/chat/completions`. Tout outil capable d'appeler un point d'accès compatible OpenAI, avec une URL de base et une clé configurables, fonctionne.

* URL de base : `https://albert.api.etalab.gouv.fr/v1`
* Clé : votre clé Albert API, à obtenir sur [https://albert.playground.etalab.gouv.fr](https://albert.playground.etalab.gouv.fr)

Ne mettez jamais votre clé dans un fichier de configuration versionné. Passez par une variable d'environnement.

## Choix des modèles

Le catalogue de modèles évolue régulièrement (ajout, suppression) et les identifiants des modèles peuvent changer de version. Les identifiants cités sur cette page servent d'exemple. Avant de configurer un outil, consultez :

* la page [Modèles disponibles](https://guides.ia.numerique.gouv.fr/albert-api/modeles/available-models) pour les modèles, leurs capacités et leurs fenêtres de contexte ;
* le [Changelog](https://guides.ia.numerique.gouv.fr/albert-api/ressources/changelog) pour les ajouts, retraits et migrations annoncés.

Récupérez ensuite l'identifiant exact avec la liste des modèles, et utilisez le champ **`id`**, pas une valeur du champ `aliases` : les alias sont une commodité d'affichage et changent sans préavis.

```bash
curl -s https://albert.api.etalab.gouv.fr/v1/models \
  -H "Authorization: Bearer $ALBERT_API_KEY" \
  | jq -r '.data[] | select(.type=="text-generation" or .type=="image-text-to-text") | .id'
```

Pour du code agentique, choisissez un modèle qui prend en charge les appels d'outils (voir [Function calling & outils](https://guides.ia.numerique.gouv.fr/albert-api/guides/function-calling)) et dispose d'une fenêtre de contexte large. Les exemples ci-dessous utilisent `deepseek-v4-flash-0731` (131 072 tokens de contexte au 16 septembre 2026).

<details>

<summary><strong>OpenCode V2</strong></summary>

* Site officiel et installation : [https://opencode.ai/v2/docs/](https://opencode.ai/v2/docs/)
* Documentation de configuration : [https://opencode.ai/v2/docs/config](https://opencode.ai/v2/docs/config)
* Fichier de configuration utilisateur : `~/.config/opencode/opencode.json`

Définissez la variable d'environnement `ALBERT_API_KEY` avec votre clé Albert API. Le champ `env` permet à OpenCode de l'utiliser pour authentifier les requêtes.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "albert/deepseek-v4-flash-0731",
  "providers": {
    "albert": {
      "name": "Albert API (DINUM)",
      "env": ["ALBERT_API_KEY"],
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "https://albert.api.etalab.gouv.fr/v1"
      },
      "models": {
        "deepseek-v4-flash-0731": {
          "name": "DeepSeek V4 Flash (Albert)",
          "limit": {
            "context": 131072,
            "output": 32768
          }
        }
      }
    }
  }
}
```

La clé `shell` ci-dessus concerne Windows avec Git Bash. Supprimez-la sous macOS et Linux.

</details>

<details>

<summary><strong>OpenCode V1 </strong></summary>

{% hint style="info" %}
OpenCode a lancé [OpenCode V2](https://opencode.ai/v2/docs) en Septembre 2026. L'installation ci-dessous fonctionne avec [OpenCode V1](https://opencode.ai/docs).
{% endhint %}

* Site officiel et installation : [https://opencode.ai/](https://opencode.ai/)
* Documentation de configuration : [https://opencode.ai/docs/config/](https://opencode.ai/docs/config/)
* Fichier de configuration utilisateur : `~/.config/opencode/opencode.json`

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "albert/deepseek-v4-flash-0731",
  "small_model": "albert/deepseek-v4-flash-0731",
  "provider": {
    "albert": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Albert API (DINUM)",
      "options": {
        "apiKey": "{env:ALBERT_API_KEY}",
        "baseURL": "https://albert.api.etalab.gouv.fr/v1"
      },
      "models": {
        "deepseek-v4-flash-0731": {
          "name": "DeepSeek V4 Flash (Albert)",
          "tool_call": true,
          "reasoning": true,
          "limit": { "context": 131072, "output": 32768 },
          "options": {
            "chat_template_kwargs": { "thinking": true }
          }
        }
      }
    }
  }
}
```

Sous Windows avec Git Bash, ajoutez `"shell": "C:\\Program Files\\Git\\bin\\bash.exe"` à la racine du fichier. Sous macOS et Linux, cette clé est inutile.

</details>

<details>

<summary><strong>Pi (terminal)</strong></summary>

* Site officiel et installation : [https://pi.dev/](https://pi.dev/)
* Documentation de configuration : [https://pi.dev/docs/latest/models](https://pi.dev/docs/latest/models)
* Fichier de configuration utilisateur : `~/.pi/agent/models.json`

```json
{
  "providers": {
    "albert": {
      "baseUrl": "http://127.0.0.1:8080/v1",
      "api": "openai-completions",
      "apiKey": "none",
      "compat": {
        "supportsDeveloperRole": false,
        "maxTokensField": "max_tokens"
      },
      "models": [
        {
          "id": "deepseek-v4-flash-0731",
          "name": "Albert · DeepSeek V4 Flash",
          "reasoning": false,
          "input": ["text"],
          "contextWindow": 131072,
          "maxTokens": 32768
        },
        {
          "id": "gpt-oss-120b",
          "name": "Albert · gpt-oss 120B",
          "reasoning": true,
          "input": ["text"],
          "contextWindow": 131072,
          "maxTokens": 32768,
          "compat": { "thinkingFormat": "openai" }
        }
      ]
    }
  }
}
```


</details>

<details>

<summary><strong>Cline (terminal)</strong></summary>

Site officiel et installation : [https://cline.bot/](https://cline.bot/)

* Documentation de configuration : [https://docs.cline.bot/provider-config/openai-compatible](https://docs.cline.bot/provider-config/openai-compatible)
* Fichier de configuration utilisateur : `~/.cline/data/settings/providers.json`

```json
{
  "version": 1,
  "lastUsedProvider": "openai-compatible",
  "providers": {
    "openai-compatible": {
      "settings": {
        "provider": "openai-compatible",
        "apiKey": "votre_clé_albert",
        "model": "deepseek-v4-flash-0731",
        "baseUrl": "https://albert.api.etalab.gouv.fr/v1"
      }
    }
  }
}
```

</details>

<details>

<summary><strong>Kilo Code (terminal)</strong></summary>

* Site officiel et installation : [https://kilo.ai/](https://kilo.ai/)
* Documentation de configuration : [https://kilo.ai/docs/code-with-ai/agents/custom-models](https://kilo.ai/docs/code-with-ai/agents/custom-models)
* Fichier de configuration utilisateur : `~/.config/kilo/kilo.jsonc`

```jsonc
{
  "$schema": "https://app.kilo.ai/config.json",
  "model": "albert/deepseek-v4-flash-0731",
  "provider": {
    "albert": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Albert API (DINUM)",
      "options": {
        "apiKey": "{env:ALBERT_API_KEY}",
        "baseURL": "https://albert.api.etalab.gouv.fr/v1"
      },
      "models": {
        "deepseek-v4-flash-0731": {
          "name": "DeepSeek V4 Flash (Albert)",
          "tool_call": true,
          "limit": { "context": 131072, "output": 32768 }
        }
      }
    }
  }
}
```

</details>

## Dans un IDE (VS Code, Cursor, etc.)

<details>

<summary><strong>Avec l'extension OpenCode</strong></summary>

Si vous avez déjà configuré OpenCode comme ci-dessus alors l'extension officielle OpenCode fonctionne dans **VS Code, Cursor, Windsurf et VSCodium**. Elle s'installe automatiquement la première fois que vous lancez `opencode` dans le terminal intégré de l'éditeur, ou manuellement depuis la place de marché des extensions.

Elle apporte le raccourci d'ouverture, le partage automatique de votre sélection ou de l'onglet courant avec l'agent, et l'insertion de références de fichiers du type `@Fichier#L37-42`.

Documentation : [https://opencode.ai/docs/ide/](https://opencode.ai/docs/ide/)

</details>

<details>

<summary><strong>Les extensions Cline et Kilo Code</strong></summary>

Cline et Kilo Code proposent aussi en extension pour VS Code, Cursor, Windsurf, VSCodium et JetBrains. Si vous avez configuré l'outil comme expliqué précédemment vous n'avez rien d'autre à régler.

* **Cline** : recherchez « Cline » dans le panneau des extensions (`Ctrl+Shift+X` ou `Cmd+Shift+X`). Vous pouvez aussi configurer Albert API depuis ses réglages : **API Provider** `OpenAI Compatible`, puis **Base URL**, **API Key** et **Model ID**. À la première ouverture, Cline propose de se connecter à son propre service : ce n'est pas nécessaire avec votre clé AlbertAPI.\
  Documentation : [https://docs.cline.bot/getting-started/installing-cline](https://docs.cline.bot/getting-started/installing-cline)
* **Kilo Code** : recherchez « Kilo Code » ; sous VS Code, choisissez **Install Pre-Release Version** dans le menu du bouton **Install** (canal recommandé par l'éditeur). Vous pouvez aussi configurer Albert API depuis **Settings** → **Providers** → **Custom provider** (Provider API : `OpenAI Compatible`).\
  Documentation : [https://kilo.ai/docs/getting-started/installing](https://kilo.ai/docs/getting-started/installing)

</details>

<details>

<summary><strong>GitHub Copilot</strong></summary>

Simon Roux maintient une extension VS Code qui **enregistre AlbertAPI comme fournisseur de modèles à l'intérieur de GitHub Copilot Chat**. Ce n'est pas une extension autonome : elle suppose que GitHub Copilot est déjà installé et connecté, et VS Code en version 1.125 ou ultérieure.

[https://forge.apps.education.fr/rouxsimon/albert-vscode](https://forge.apps.education.fr/rouxsimon/albert-vscode)

</details>

### Compatibilité “théorique” (non garantie)

De façon générale, **n’importe quel plugin / IDE** capable d’appeler un modèle via `POST /v1/chat/completions`, et/ou un SDK “OpenAI” peut **théoriquement** fonctionner avec Albert API. En revanche, nous ne pouvons pas **garantir** la compatibilité de toutes les combinaisons outil/IDE, ni les champs supportés par chaque client.

## Ce qui ne fonctionne pas par défaut

Albert API ne répond pas sur ces endpoints :

* `POST /v1/responses`, le format récent d'OpenAI, utilisé notamment par Codex CLI
* `POST /v1/messages`, le format d'Anthropic, utilisé par Claude Code
* `POST /v1/completions`, le format historique utilisé pour l'autocomplétion&#x20;

Ces outils ne peuvent donc pas être branchés directement sur Albert API. llm-proxy (voir la section Pi ci-dessus) expose ces formats en local et les traduit vers Albert API. Le dépot de llm-proxy contient un guide par outil.&#x20;
