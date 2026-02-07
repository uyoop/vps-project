# 03. STRATÉGIE DEVOPS & CI/CD

> **Statut** : V1.0 (7 Février 2026)
> **Objectif** : Industrialiser le déploiement de l'infrastructure et des applications pour garantir reproductibilité et sécurité.

## 1. Philosophie "GitOps"
Tout part du code. L'état souhaité de l'infrastructure est défini dans le dépôt Git.
*   **Source de Vérité** : `gitlab.com/uyoop/infrastructure`
*   **Synchronisation** : 
    *   **Push-based** (Phase 1/2) : GitLab CI pousse les changements via `kubectl` / `ansible`.
    *   **Pull-based** (Phase 3 - Cible) : **FluxCD** ou **ArgoCD** surveille le dépôt et applique les changements (réconciliation automatique).

## 2. Structure du Dépôt (Monorepo)
```
.
├── ansible/                # Configuration des VMs (OS, Docker, K3s base)
│   ├── inventory/
│   ├── roles/
│   │   ├── common/         # Users, SSH, Fail2Ban
│   │   ├── security/       # UFW, CrowdSec
│   │   └── k3s/            # Setup Cluster
│   └── playbooks/
├── k8s/                    # Manifestes Kubernetes (Applications)
│   ├── base/               # Configs communes
│   ├── overlays/
│   │   ├── prod/           # Namespaces Prod
│   │   └── dev/            # Namespaces Test
│   ├── charts/             # Helm Charts locaux
│   └── secrets/            # SealedSecrets (Chiffrés)
├── docker/                 # Dockerfiles spécifiques (app métier)
└── docs/                   # Documentation technique (Ce dossier)
```

## 3. Pipelines CI/CD (GitLab)

### A. Pipeline d'Infrastructure (Ansible)
Déclenché sur modification dans `/ansible/*`.
1.  **Lint** : `ansible-lint` vérifie la syntaxe.
2.  **Dry-Run** : `ansible-playbook --check` simule l'exécution.
3.  **Deploy** (Manuel/Master) : Applique la configuration sur les VPS.

### B. Pipeline Applications (K8s)
Déclenché sur modification dans `/k8s/*`.
1.  **Test** : `kubeval` ou `pluto` vérifie la validité des manifests.
2.  **Scan** : `trivy config` scanne les mauvaises configurations (ex: run as root).
3.  **Deploy** : `kubectl apply` met à jour le cluster.

## 4. Gestion des Environnements

| Environnement | Branche Git | Cluster K3s | URL Cible |
| :--- | :--- | :--- | :--- |
| **Development** | `feature/*` | Namespace `dev-lab` | `*.dev.uyoop.fr` |
| **Staging** | `develop` | Namespace `staging` | `*.pre.uyoop.fr` |
| **Production** | `main` | Namespace `prod-*` | `*.uyoop.fr` |

## 5. Outils de la Chaîne
*   **Code** : GitLab (SaaS)
*   **CI Runner** : Self-hosted sur VPS-2 (Docker Executor).
*   **Code Quality** : SonarQube (Static Analysis).
*   **Artifacts** : GitLab Container Registry (Images Docker privées).
*   **Secrets** : Sealed Secrets (Bitnami) pour commiter des secrets chiffrés.
