# Chat completions

L’endpoint **`POST /v1/chat/completions`** est le point d’entrée principal pour la génération de texte. Le corps de requête et la forme de la réponse suivent le modèle **OpenAI Chat Completions** : messages, paramètres de décodage, format de sortie, streaming et outils.

Pour **`POST /v1/chat/completions`**, la spec attend un modèle de type **`text-generation`** (voir [Types de modèles](../models/model-types.md)). Les modèles **`image-text-to-text`** ou **`image-to-text`** servent à d’autres usages (p. ex. OCR), pas au chat texte standard — ne vous fiez pas aux anciens tutoriels qui les mélangeaient avec le chat sur des instances différentes.

## Messages

La conversation est une liste `messages` ordonnée. Rôles courants :

| Rôle | Rôle technique |
| --- | --- |
| `system` | Consignes stables pour le modèle (ton, format, contraintes). |
| `user` | Entrée de l’utilisateur ou de l’application. |
| `assistant` | Réponses précédentes du modèle (historique multi-tours). |
| `tool` | Réponse d’un outil après un `tool_calls` (voir [Function calling](function-calling.md)). |

## Paramètres de génération

- **`temperature`** — entropie du tirage (par défaut typiquement autour de `0.7` dans la spec).
- **`top_p`** — échantillonnage par masse cumulée (*nucleus sampling*).
- **`max_completion_tokens`** — plafond de tokens **générés** pour la complétion.
- **`frequency_penalty`**, **`presence_penalty`**, **`stop`**, **`seed`**, **`logprobs`** — mêmes usages que sur l’API OpenAI.

## Format de réponse (`response_format`)

- **`{"type": "json_object"}`** — mode JSON : le modèle doit produire un JSON valide. Il faut aussi **demander explicitement du JSON** dans un message `system` ou `user`, sinon la génération peut se prolonger artificiellement.
- **`{"type": "json_schema", "json_schema": { ... }}`** — sorties structurées guidées par un schéma JSON (équivalent conceptuel aux Structured Outputs OpenAI).

## Streaming

Passez **`"stream": true`** pour recevoir des deltas au format **Server-Sent Events**. Voir le guide [Streaming](streaming.md).

## Outils et `tool_calls`

- **`tools`** — liste d’outils que le modèle peut invoquer (fonctions au schéma JSON, et outils intégrés côté Albert).
- **`tool_choice`** — `none`, `auto`, `required`, ou choix d’une fonction nommée.
- **`parallel_tool_calls`** — exécution parallèle ou séquentielle des outils lorsque c’est supporté.

L’outil de recherche **RAG natif** (`SearchTool`) est décrit dans [Outil de recherche RAG intégré](rag-search-tool.md).

## Compatibilité OpenAI

En dehors des extensions propres à Albert (SearchTool, endpoints métiers supplémentaires), les champs et le comportement général sont pensés pour être **interopérables** avec les clients OpenAI : mêmes noms de champs pour les paramètres usuels, mêmes rôles de messages, mêmes conventions pour le streaming SSE.

{% hint style="warning" %}
⚠️ À vérifier — Champs additionnels exacts et valeurs par défaut par modèle : se référer au schéma `CreateChatCompletion` dans la [Référence OpenAPI](../api-reference/reference.md).
{% endhint %}

## Paramètres de recherche dépréciés sur le chat

Le corps de requête peut encore mentionner **`search`** et **`search_args`** : ces champs sont **dépréciés** au profit du pipeline collections + documents + **`SearchTool`** dans `tools`. Ne les utilisez pas pour de nouveaux développements.
