# Perspectives : L'IA au service du DevSecOps V2

> **Statut** : V2 (6 Février 2026)
> **Vision** : Transformer une infrastructure d'hébergement classique en une plateforme intelligente d'assistance aux opérations.

## 1. Pourquoi l'IA dans ce projet ?
Dans une démarche **"Stratégie à votre image"**, l'IA ne doit pas être une boîte noire externe (ChatGPT/Copilot SaaS) qui ingère nos données sensibles. Elle doit être un composant interne de l'infrastructure (`ai-node`), capable de comprendre le contexte métier (Documentation, Code propriétaire) sans fuite de données.

## 2. Cas d'Usage Implémentés (Phase 2)

### A. Moteur de Recherche Documentaire (RAG)
**Problème** : La documentation technique est dispersée (Markdown GitLab, Docs Nextcloud, Notes).
**Solution** : **Perplexica** (ou équivalent Open Source).
*   **Fonctionnement** : Indexation vectorielle des dépôts GitLab et des documents Nextcloud.
*   **Usage** : Interface "Chat" permettant de poser des questions en langage naturel : *"Quelle est la procédure de rotation des clés SSH ?"* ou *"Résume l'architecture réseau du namespace prod"*.
*   **Souveraineté** : L'index reste local. Aucune donnée ne part chez OpenAI.

### B. Analyse Statique Augmentée (SAST)
**Problème** : Détecter les vulnérabilités dans le code avant déploiement.
**Solution** : **SonarQube + Plugins IA**.
*   **Pipeline** : À chaque `git push` sur gitlab.com, le Runner self-hosted sur VPS-2 déclenche l'analyse.
*   **Apport IA** : Détection de patterns complexes, suggestions de refactoring et explication des failles de sécurité aux développeurs juniors.

## 3. Architecture Logique du Nœud IA
Le VPS-2 (6 vCores / 12 Go RAM / 100 Go NVMe) est configuré comme un "Worker" spécialisé.

| Couche | Technologie | Rôle |
| :--- | :--- | :--- |
| **Interface** | UI Web / API | Point d'entrée pour l'utilisateur et les webhooks |
| **Cerveau** | **Ollama** | Exécution des modèles (LLM) optimisés (ex: Mistral-7B, Llama3) |
| **Mémoire** | **Qdrant** | Base de données vectorielle pour le contexte (RAG) |
| **Calcul** | CPU (AVX2) | Inférence sur CPU (Pas de GPU, modèles quantifiés Q4_K_M) |

## 4. Futur : Vers l'Auto-Remédiation ?
À terme (Post-Stage), l'objectif est de coupler le monitoring (AlertManager) à l'IA pour proposer des diagnostics automatiques en cas d'incident :
*   *Alerte* : "Disk Usage High on /var/lib/docker".
*   *IA* : "Analyse : Logs conteneur Mailcow anormalement volumineux. Suggestion : `docker system prune` ou rotation logs."

---
*"L'IA n'est pas une finalité, c'est un membre de l'équipe Ops."*
