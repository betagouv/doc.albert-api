# Function calling et outils

Albert API permet au modèle de chat de proposer des **appels d’outils** (`tool_calls`) décrits dans le corps de la requête. Le flux est le même que sur l’API OpenAI : le modèle émet une complétion avec des appels ; le client exécute la logique métier ; une réponse au rôle `tool` est renvoyée dans `messages` ; le modèle produit alors une réponse utilisateur finale.

## Déclarer des outils

Le champ **`tools`** est une liste d’objets. Chaque outil de type fonction comporte au minimum :

- un **nom** ;
- une **description** ;
- des **paramètres** au format JSON Schema (`parameters`) décrivant les arguments attendus.

Le détail des schémas suit la convention **Chat Completions** OpenAI. Consultez le schéma `CreateChatCompletion` dans la [Référence OpenAPI](../api-reference/reference.md) pour la forme exacte attendue par le serveur.

## `tool_choice`

- **`none`** — pas d’appel d’outil (comportement par défaut lorsqu’aucun outil n’est fourni).
- **`auto`** — le modèle choisit entre répondre en texte ou appeler un ou plusieurs outils (défaut lorsque des outils sont présents).
- **`required`** — le modèle doit invoquer au moins un outil.
- Sélection ciblée — forcer un outil précis via un objet du type `{"type": "function", "function": {"name": "nom_fonction"}}`.

## Enchaînement typique

1. **Requête initiale** — `messages` + `tools` + `tool_choice` adéquat.
2. **Réponse assistant** — peut contenir `tool_calls` avec des `arguments` JSON à parser.
3. **Exécution** — votre code appelle bases de données, HTTP internes, etc.
4. **Message `tool`** — vous ajoutez un message avec `role: "tool"`, le `tool_call_id` correspondant et le **résultat** (souvent une chaîne JSON).
5. **Nouvelle requête** — vous renvoyez toute l’historique mis à jour pour obtenir la réponse finale de l’assistant.

## Outils intégrés Albert

Outre les fonctions « génériques », Albert expose l’outil natif de **recherche documentaire** (`SearchTool`, `type: "search"`). Il n’est **pas** équivalent à un connecteur OpenAI hébergé : pipeline, paramètres et sémantique sont ceux d’Albert. Voir [Outil de recherche RAG intégré](rag-search-tool.md).

## Erreurs et robustesse

- Validez et **sandboxez** les arguments produits par le modèle avant toute action sensible.
- Limitez la profondeur des enchaînements d’outils pour éviter boucles et surcoûts tokens.
