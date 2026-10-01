---
description: >-
  Cette page s'adresse aux entreprises qui proposent une solution logicielle à
  des administrations de l'État et qui souhaitent s'appuyer sur Albert API pour
  les fonctionnalités d'IA générative.
icon: plug
---

# Entreprises : se connecter à Albert API

### En bref

* Albert API est compatible avec le standard OpenAI. Une solution qui sait déjà appeler une API OpenAI peut s'y brancher sans développement spécifique, en mode « _bring your own model_ ».
* Pour vos clients publics, Albert API est une brique IA gratuite et de confiance, compatible avec le traitement de données sensibles au titre de l'article 31 de la loi SREN.
* L'équipe Albert API peut vous fournir une clé d'intégration temporaire pour vos tests. En production, votre solution utilise le compte Albert API de l'administration cliente.

### Rappel : à qui est destiné Albert API

Albert API est un service réservé aux agents de la fonction publique d'État. Une entreprise ne peut donc pas l'utiliser pour son propre compte ni le revendre. Elle peut en revanche configurer sa solution pour que ses clients publics y accèdent avec leurs propres identifiants.

### Dans quels cas intégrer Albert API

L'intégration est particulièrement pertinente lorsque votre solution est déjà hébergée dans un environnement de confiance (on-premise chez l'administration ou en cloud qualifié SecNumCloud) et que seule la brique IA manque pour que l'ensemble respecte les exigences de souveraineté. Albert API complète alors votre architecture sans faire sortir les données de cet environnement de confiance.

Deux configurations types :

| Configuration                                    | Description                                                                                                          |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| Solution hébergée en SecNumCloud                 | Votre application tourne chez un hébergeur qualifié et appelle Albert API pour l'inférence.                          |
| Solution déployée dans le SI de l'administration | Votre application est installée on-premise chez votre client et appelle Albert API depuis son système d'information. |

### Prérequis

Avant d'engager une intégration, vérifiez les points suivants :

1. **Compatibilité technique.** Votre produit sait appeler une API compatible OpenAI. Selon votre architecture, cela peut passer par une configuration propre à chaque client, par l'ajout d'Albert API à votre catalogue de modèles, ou par un paramétrage « bring your own model » directement accessible.
2. **Adéquation des modèles.** Les modèles et les limites de débit (rate limits) proposés par Albert API correspondent aux besoins de votre fonctionnalité. Consultez la [liste des modèles disponibles](https://guides.ia.numerique.gouv.fr/albert-api/modeles/available-models).
3. **Projet client identifié.** Vous êtes en discussion avancée avec une ou plusieurs administrations, et celles-ci ont validé le principe d'utiliser Albert API. Les intégrations ne sont pas ouvertes à titre exploratoire, sans projet client identifié.
4. **Fonctionnement en « bring your own model ».** En production, c'est l'administration cliente qui détient l'accès à Albert API. Votre produit doit donc pouvoir fonctionner avec une clé API fournie par le client, même si ce paramétrage peut rester invisible pour l'utilisateur final.

### Exemple de configuration

Dans la plupart des outils compatibles OpenAI, l'intégration se résume à renseigner trois paramètres :

| Paramètre   | Valeur                                                                                                                                   |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| URL de base | `https://albert.api.etalab.gouv.fr/v1`                                                                                                   |
| Clé API     | la clé du compte Albert API de l'administration cliente                                                                                  |
| Modèle      | l'identifiant d'un modèle de la [liste des modèles disponibles](https://guides.ia.numerique.gouv.fr/albert-api/modeles/available-models) |

Par exemple, avec le SDK Python OpenAI :

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://albert.api.etalab.gouv.fr/v1",
    api_key="sk-...",  # clé du compte Albert API de l'administration
)

response = client.chat.completions.create(
    model="<identifiant-du-modele>",
    messages=[{"role": "user", "content": "Bonjour"}],
)
print(response.choices[0].message.content)
```

Des outils open source comme OpenCode ou OpenWorker proposent ce paramétrage directement dans leur interface, via un « fournisseur personnalisé » compatible OpenAI.

### Démarche

1. **Prise de contact.** Ecrivez à l'équipe Albert API : albert.api@numerique.gouv.fr. Présentez-y rapidement votre projet, en précisant l'administration cliente concernée. L'équipe vérifie éventuellement avec vous que les prérequis ci-dessus sont remplis.
2. **Création d'un compte ProConnect.** Créez un compte [ProConnect](https://proconnect.crisp.help/fr/article/comment-creer-un-compte-sur-proconnect-q4h0ci/) pour les personnes qui réaliseront l'intégration. Cette opération peut être réalisée en parallèle de l'étape 1.
3. **Autorisation de votre domaine.** L'équipe Albert API ajoute le nom de domaine de votre entreprise à sa liste d'autorisation. Vous aurez accès à Albert API pendant une durée définie avec votre interlocuteur.
4. **Accès au playground.** Vous pourrez ensuite accéder au [playground Albert API](https://albert.playground.etalab.gouv.fr/) pour générer des clés APIs de développement.
5. **Mise en production.** Une fois la solution déployée chez une administration, celle-ci devra créer une clé API via le playground, comme vous l'avez fait. La clé d'intégration n'est pas destinée à un usage en production.

### Pour aller plus loin

* [Documentation Albert API](https://guides.ia.numerique.gouv.fr/albert-api)
* [Modèles disponibles](https://guides.ia.numerique.gouv.fr/albert-api/modeles/available-models)
