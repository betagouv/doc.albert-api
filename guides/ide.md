# Utiliser un modèle de code dans un IDE

Albert API est compatible “OpenAI-like” sur l’endpoint de chat : vous pouvez donc brancher des modèles (notamment des modèles orientés code) sur des IDE et agents qui parlent l’API OpenAI.

Aujourd’hui, nous **maintenons et testons** principalement **2 façons** d’intégrer Albert API à un outil de développement.

## VS Code (Continue)

[Continue](https://marketplace.visualstudio.com/items?itemName=Continue.continue) est une extension VS Code qui peut se configurer avec un provider “OpenAI”.

Après installation, configurez le fichier `config.yaml` (Continue) :

- **macOS / Linux** : `~/.continue/config.yaml`
- **Windows** : `%USERPROFILE%\.continue\config.yaml`

Vous pouvez aussi ouvrir la configuration directement depuis le plugin. Voir la documentation Continue : [How to Configure Continue](https://docs.continue.dev/customize/deep-dives/configuration).

Exemple de configuration (Continue) :

```yaml
models:
  - name: OpenGateLLM-Chat
    provider: openai
    model: Qwen/Qwen3-Coder-30B-A3B-Instruct
    apiBase: https://albert.api.etalab.gouv.fr/v1
    apiKey: PUT_ALBERT_KEY_HERE
```

Nous recommandons l’usage de **openweight-code** avec Continue. À date, **openweight-code ne propose pas de compatibilité avec la fonctionnalité d’auto-complétion** (autocomplete) : l’intégration est donc principalement pertinente pour les usages “chat / assistance”.

## Agentic coding (opencode)

Pour les usages **agentiques** (édition multi-fichiers, planification, exécution d’actions), nous avons testé **opencode**.

- **Téléchargement / installation** : [OpenCode](https://opencode.ai/)
- **Fichier de configuration** (préférences utilisateur) : `~/.config/opencode/opencode.json`
- **Documentation** : [Configuration (OpenCode)](https://opencode.ai/docs/config/)

Exemple de configuration (opencode) :

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "litellm": {
      "models": {
        "Qwen/Qwen3-Coder-30B-A3B-Instruct": {
          "name": "Qwen/Qwen3-Coder-30B-A3B-Instruct"
        },
        "openai/gpt-oss-120b": {
          "name": "openai/gpt-oss-120b"
        }
      },
      "options": {
        "apiKey": "PUT ALBERT KEY HERE",
        "baseURL": "https://albert.api.etalab.gouv.fr/v1"
      }
    }
  },
  "mode": {
    "build": {
      "model": "Qwen/Qwen3-Coder-30B-A3B-Instruct",
      "tools": {
        "write": true,
        "edit": true,
        "bash": false
      }
    }
  }
}
```

## Compatibilité “théorique” (non garantie)

De façon générale, **n’importe quel plugin / IDE** capable d’appeler un modèle via :

- `POST /v1/chat/completions`, et/ou
- un SDK “OpenAI”

peut **théoriquement** fonctionner avec Albert API. En revanche, nous ne pouvons pas **garantir** la compatibilité de toutes les combinaisons outil/IDE, ni les champs supportés par chaque client.

Si vous voulez valider que le `model` que vous sélectionnez correspond au bon type, commencez par la liste des modèles : [Guide “Modèles”](models.md). Pour une vue “publique” des modèles (familles, licences, alias) vous pouvez aussi consulter : [Catalogue des modèles](https://albert.sites.beta.gouv.fr/solutions/models/).

{% hint style="info" %}
À date, le support **`v1/responses` (OpenAI)** et **`v1/messages` (Anthropic)** n’est pas encore disponible dans Albert API. En conséquence, l’intégration avec certains outils de coding (ex. **Claude Code**) n’est pas disponible “par défaut”.

Certains utilisateurs ont cependant réussi via leur propre **proxy** qui traduit/adapte les requêtes et réponses d’Albert API vers l’assistant de leur choix. Nous ne pouvons pas maintenir ni lister l’ensemble des proxies existants : pour des conseils et des liens vers les proxies maintenus par la communauté, vous pouvez demander sur [Tchap](https://tchap.gouv.fr/#/room/#albert:agent.dinum.tchap.gouv.fr).
{% endhint %}

