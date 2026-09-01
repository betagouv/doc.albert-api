---
description: Cette page résume l’évolution fonctionnelle et technique visible d'Albert API.
icon: clock-rotate-left
---

# Changelog

## Référence de version

La **version d'Albert API** exposée sur `https://albert.api.etalab.gouv.fr/openapi.json` figure dans le champ `info.version` du document (par exemple `0.4.x` selon les déploiements). C’est la **source de vérité** pour les routes et schémas disponibles sur un environnement donné.

***

## Versions

<details>

<summary><strong>0.6.0 (beta)</strong></summary>

<mark style="color:red;">**Date de déploiement : 31 août 2026**</mark>

#### **🔐 Connexion au Playground via ProConnect**

Dans un souci d'améliorer la sécurité d'Albert API, la connexion au Playground se fait désormais via ProConnect, le SSO de l'Etat.\
\
Vos comptes existants et vos clés d'API sont conservés : aucune action de migration n'est nécessaire.\
\
⚠️ Connectez-vous avec l'adresse professionnelle exacte associée à votre compte actuel (celle de votre email de confirmation). Une autre adresse créera un nouveau compte, vide, sans vos clés. **Si votre email Albert API ne correspond pas à votre adresse ProConnect, merci de nous contacter pour faire le matching (albert.api@numerique.gouv.fr).**\
\
Si l'accès vous est refusé, rendez-vous sur notre [FAQ](faq.md#pourquoi-mon-compte-nest-pas-autorise) pour connaître la démarche à suivre.

#### 🎁 Phase de test pour Gemma 4

Un nouveau modèle vient s'ajouter au catalogue des modèles : `gemma-4-31-b-it`. Ce modèle est candidat au remplacement du modèle `openweight-medium` durant du phase de test allant jusqu'au 1er décembre 2026.

Nous attendons vos retours sur ce nouveau modèle. Vous retrouvez toutes les spécifications techniques de ce modèle dans notre [documentation](../modeles/available-models.md#gemma-4-31b-it).

**Retrouvez le calendrier des migrations de modèle dans notre documentation** [**ici**](../modeles/available-models.md#calendrier-des-migrations)**.**

</details>

<details>

<summary><strong>0.5.0 (beta)</strong></summary>

<mark style="color:red;">**Date de déploiement : 27 juillet 2026**</mark>

La version 0.5.0 d'Albert API introduit une phase d'expérimentation de nouveaux modèles ainsi que la suppression définitive de plusieurs fonctionnalités dépréciées depuis la version 0.4.1.

#### 🎁 Phase de test de nouveaux modèles

À compter du **25 juillet 2026**, nous lançons une phase de test de trois nouveaux modèles. Cette expérimentation se poursuivra jusqu'au **1er octobre 2026**, date à laquelle nous déciderons, en fonction de vos retours et de nos évaluations, de pérenniser tout ou partie de ces modèles.

**Nous attendons avec impatience vos retours sur le canal de support !**

* [LightOnOCR-2-1B](https://huggingface.co/lightonai/LightOnOCR-2-1B) (`openweight-ocr`)

Ce modèle, accessible via l'endpoint `/v1/chat/completions`, est spécialisé dans l'OCR. Il permet d'extraire du texte à partir d'images, notamment de documents ou de pages de PDF numérisées, grâce à ses capacités multimodales.

Une documentation détaillée sera publiée le **25 juillet**.

* [DeepSeek-V4-Flash](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash)

Ce modèle de **284 milliards de paramètres** est évalué comme successeur du modèle **Qwen3-Coder-30B-A3B-Instruct** pour les usages liés au développement logiciel.

Si les résultats de la phase de test sont concluants, il remplacera le modèle actuellement proposé sous l'alias `openweight-code`.

**⚠️ Nous ne recommandons pas d'utiliser ce modèle pour d'autres usages que le code.**

* [Qwen3-VL-Embedding-8B](https://huggingface.co/Qwen/Qwen3-VL-Embedding-8B)

Ce modèle d'embeddings multimodal, accessible via l'endpoint `/v1/embeddings`, vous permettra de représenter de manière vectorielle des textes, des images, des screenshots et des vidéos. Cela vous permettra de réaliser dans votre vector store du RAG avec différents formats d'entrée.

Une documentation détaillée sera publiée le **25 juillet**.

#### &#x20;🗑️ Suppression de fonctionnalités dépréciées

Comme annoncé lors de la sortie de la version **0.4.1**, plusieurs fonctionnalités dépréciées seront définitivement supprimées avec la version **0.5.0**.

Ces évolutions visent à simplifier et homogénéiser les contrats d'API.

Nous vous invitons à vérifier les changements ci-dessous et à mettre à jour vos intégrations si vous êtes concernés.

* L'argument `collection` de l'endpoint `POST /v1/documents` est supprimé au profit de `collection_id`.
* L'endpoint `GET /v1/chunks/{document}/{chunk}` est remplacé par `GET /v1/documents/{document_id}/chunks/{chunk_id}`.
* L'endpoint `GET /v1/chunks` est remplacé par `GET /v1/documents/{document_id}/chunks`.
* L'endpoint `POST /v1/parse-beta` est supprimé.
* L'argument `collections` de l'endpoint `POST /v1/search` est remplacé par `collection_ids`.
* L'argument `search` de l'endpoint `POST /v1/chat/completions` est supprimé au profit de l'outil `search`, à utiliser via le champ `tools`.
* L'argument `prompt` de l'endpoint `POST /v1/search` est remplacé par `query` et l'argument `collections` par `collections`
* L'argument `collections` de l'endpoint `POST /v1/search` est remplacé supprimé par `collection_ids`.

#### 📦 Fonctionnalité dépréciées

* Le tool `search` de l'endpoint `POST /v1/chat/completions` est déprécié. La fonctionnalité est maintenue pour le moment mais ne fera plus l'objet d'améliorations, elle sera supprimée lors de la version 1.0.0 (Q4 2026). Nous souhaitons privilégier l'usage de la combinaison de l'usage de `/v1/search` et `/v1/chat/completion` pour un RAG maitrisé.

</details>
