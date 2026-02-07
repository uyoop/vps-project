# Adéquation Projet / Référentiel Compétences RNCP V2

> **Statut** : V2 (6 Février 2026)
> **Titre visé** : Administrateur Systèmes DevOps (RNCP 36061).
> **Objectif** : Ce chapitre démontre la couverture des blocs de compétences du titre par les réalisations concrètes du projet uyoopVPS.

---

## Bloc 1 : Automatiser le déploiement d'une infrastructure (RNCP36061BC01)

| Compétence Référentiel | Réalisation Concrète Projet uyoop | Livrable Dossier |
| :--- | :--- | :--- |
| **Automatiser la création de serveurs** | Provisioning des VPS (Core & AI) via scripts de bootstrap (Cloud-init / Bash). Configuration initiale automatisée. | Scripts de setup (`setup.sh`), configuration SSH/User. |
| **Automatiser le déploiement** | Utilisation de l'approche **GitOps** : pipelines **GitLab CI/CD** (gitlab.com SaaS + Runner self-hosted) et **Helm Charts** pour déployer K3s, Mailcow (Docker Compose), Nextcloud. L'état désiré est défini dans le code. | Dépôt Git d'Infrastructure, Manifests Kubernetes, `.gitlab-ci.yml`. |
| **Sécuriser l'infrastructure** | Architecture **Zero-Trust**. Mise en place de **Authelia** (2FA/SSO), **Sealed Secrets** (chiffrement Git-safe) + **HashiCorp Vault** (secrets dynamiques runtime), **NetworkPolicies** (Isolation Namespaces), et durcissement OS (**UFW** Firewall + **CrowdSec** IDS). | Chap. 01 (Architecture), config Firewall, Politiques K8s, Policy Vault. |
| **Mettre en prod. dans le cloud** | Gestion de l'exposition Internet via **Traefik** (Ingress Controller), gestion DNS (OVH) et certificats TLS auto (Cert-Manager / Let's Encrypt). Approche **hybride** : SaaS (gitlab.com) + IaaS (OVH VPS). Stratégie multi-domaines : `uyoop.fr/com` (pro) + `cjenti.fr/com` (perso). | URL accessibles (`mail.cjenti.com`, `blog.uyoop.fr`, etc), Dashboards Traefik. |

---

## Bloc 2 : Déployer en continu une application (RNCP36061BC02)

| Compétence Référentiel | Réalisation Concrète Projet uyoop | Livrable Dossier |
| :--- | :--- | :--- |
| **Préparer environn. de test** | Création d'un namespace dédié `dev-lab` ou utilisation du VPS-2 (AI-Lab) pour valider les mises à jour avant la prod. | Configuration Namespaces K8s, Scénarios de test. |
| **Gérer le stockage des données** | Implémentation d'une stratégie de stockage hybride : **PV/PVC Kubernetes** (`local-path-provisioner`, inclus K3s) sur NVMe + **Object Storage S3** pour les archives. | Chap. 01 (Stratégie Données), Config StorageClasses. |
| **Gérer des containers** | Conteneurisation de tous les services (Mailcow en Docker Compose standalone, Nextcloud/Ghost/Authelia en K3s). Gestion des images Docker et Registre privé (gitlab.com Container Registry). | `docker-compose.yml`, Pods K8s, Dockerfiles. |
| **Automatiser la mise en prod** | Mise en place de pipelines **GitLab CI/CD** (gitlab.com SaaS + Runner self-hosted sur VPS-2) : Build, Scan de sécurité (Trivy/Sonar), Déploiement auto sur Cluster K3s. Démontre la compétence "infra hybride" (SaaS + On-premise). | Fichiers `.gitlab-ci.yml`, Rapport de scan SonarQube. |
| **Intégrer les services métier** | Déploiement et maintien en condition opérationnelle de l'ERP **Dolibarr** (Gestion commerciale/comptable) et du Marketing Automation **Mautic**. Intégration du portail **uHub** (DevSecAiOps). Preuve d'alignement IT/Métier ("BizOps"). | Section `07-BIZOPS`, `08-INTEGRATION-UHUB`, Screenshots. |

---

## Bloc 3 : Superviser les services déployés (RNCP36061BC03)

| Compétence Référentiel | Réalisation Concrète Projet uyoop | Livrable Dossier |
| :--- | :--- | :--- |
| **Définir statistiques services** | Identification des indicateurs clés (KPI) : Disponibilité Web, Espace Disque (Alerte NVMe), Charge CPU (notamment sur Node IA). | Liste des métriques surveillées, SLA définis. |
| **Exploiter solution supervision** | Déploiement de la stack d'observabilité : **Prometheus** (Collecte), **Grafana** (Visualisation), **AlertManager** (Notifications Discord/Mail). | Screenshots Dashboards Grafana, Exemples d'alertes. |
| **Echanger en anglais** | La totalité de la documentation technique du code (Commentaires, Commits Git) et la veille technologique sont réalisées en anglais international. | Commits Git, README techniques des repos. |

---

## Synthèse de la Couverture
Le projet uyoopVPS couvre **100% des compétences techniques** du titre.
Il dépasse le cadre standard par l'intégration d'une dimension **DevSecOps** (Sécurité avancée) et **AIOps** (Intégration Intelligence Artificielle), apportant une valeur ajoutée distinctive présentée au Jury.
