# Planning de Migration & Feuille de Route V2

> **Statut** : V2 (6 Février 2026) — Intègre stratégie GitLab hybride, Mailcow Docker Compose, corrections
> **Stratégie** : "Continuité d'abord, Code ensuite".
> **Contexte de Travail** : Projet mené en parallèle du stage. Charge de travail adaptée.

---

## 🔥 PHASE 1 : "OPÉRATION SOCLE" (Immédiat - 25 Mars)
**Objectif Critique** : Continuité de service Mail/Data avant expiration hébergement o2switch (25/03).
**Note** : Sauvegarde Proton déjà sécurisée (domaines supprimés sans conséquence). Focus total sur la migration.

### Semaine 06 (Infrastructure as Code Init)
*   [x] **Repo** : Création du dépôt principal sur **gitlab.com** (Free Tier). Structure : `ansible/`, `k8s-manifests/`, `docker/`, `docs/`.
*   [ ] **Souscription** : Commande VPS Core-Prod (OVH) et domaines.
*   [ ] **Provisioning** : Création des rôles **Ansible** de base (`common`, `security`, `docker`, `k3s`).
    *   *Sécurité* : Installation auto de **UFW** + **CrowdSec** + SSH Hardening.

### Semaine 07 (Services Vitaux)
*   [ ] **Cluster** : Déploiement K3s via Ansible (Rôle `k3s-ansible`).
*   [ ] **Mail** : Déploiement Mailcow (**Docker Compose standalone** — seul mode supporté officiellement).
    *   *Test* : Validation Score SpamCheck (SPF/DKIM/DMARC).
*   [ ] **Data** : Déploiement Nextcloud (K3s).
    *   *Migration* : Upload manuel des 24 Go critiques.
*   [ ] **Mirror Git** : Cron `git clone --mirror` de gitlab.com vers VPS-3 (réversibilité souveraine).
*   [ ] **Bascule** : Changement DNS MX `cjenti.com` (mail perso) (Target : 15/03).

> *Détails techniques dans `03-DEVOPS.md` et `04-SECOPS.md`.*

---

## 🛠️ PHASE 2 : "L'USINE LOGICIELLE" (Mars - Avril)
**Objectif Titre** : Industrialisation et Sécurité (DevSecOps).

### Mars : Identité & Web
*   [ ] **Migration Web** : Transfert domaines O2Switch -> OVH (`uyoop.fr`/`.com` pro + `cjenti.fr`/`.com` perso).
*   [ ] **SSO** : Déploiement **Authelia** (LDAP Mailcow backend).
*   [ ] **BizOps** : Déploiement **Dolibarr** (Namespace `prod-gestion`) + **Mautic**.
*   [ ] **Maintenance** : Activation de **Renovate Bot** pour suivi des mises à jour.

> *Détails fonctionnels BizOps dans `07-BIZOPS.md`.*

### Avril : IA & Qualité (Extension "AI-Lab")
*   [ ] **Infra** : Provisioning VPS-2 (6 vCores / 12 Go RAM / 100 Go NVMe) via Ansible Playbook réutilisé.
*   [ ] **Réseau** : Configuration Tunnel WireGuard Mesh entre les nœuds (MTU 1420, PersistentKeepalive=25).
*   [ ] **CI/CD** : **GitLab Runner** self-hosted installé sur VPS-2, enregistré sur gitlab.com.
*   [ ] **Qualité** : SonarQube avec Quality Gate stricte, intégré au pipeline gitlab.com.
*   [ ] **Monitoring** : Déploiement **Prometheus** + **Grafana** + **AlertManager** (namespace `monitoring`). Alertes Discord/Mail sur CPU, RAM, disque, certificats.
*   [ ] **Secrets** : Déploiement **Sealed Secrets** (Bitnami) pour chiffrer les secrets dans Git.

> *Détails IA dans `05-AIOPS.md`.*

---

## 🚀 PHASE 3 : "CONSOLIDATION & RÉSILIENCE" (Mai - Juin)
**Objectif Certification** : Preuves de robustesse et Documentation finale.

### Mai : Big Data & FinOps
*   [ ] **Storage** : Activation S3 Object Storage et règles de cycle de vie.
*   [ ] **Archivage** : Migration OneDrive -> S3 (Rclone).
*   [ ] **Observabilité** : Dashboard Grafana "FinOps" (Coûts OVH API) + dashboards services (Mailcow, Nextcloud, K3s).
*   [ ] **Vault** : Déploiement **HashiCorp Vault** (secrets dynamiques runtime, rotation, injection sidecar). Complète Sealed Secrets.

### Juin : Soutenance & DRP
*   [ ] **Crash Test (DRP)** : "Journée du Chaos".
    *   *Scénario* : Suppression volontaire du namespace `prod` et restauration depuis Backup S3.
    *   *Preuve* : Vidéo du rétablissement pour la soutenance.
*   [ ] **Livrable** : Repo Git nettoyé et public (ou accès Jury).

---

## 🌐 PHASE 4 : "LANCEMENT & PERSPECTIVES" (Juillet - Septembre)
**Objectif** : Mise en ligne publique de la solution uyoop et démarrage de l'activité professionnelle.

### Fin Juin : Pause
*   [ ] **Repos** : Quelques jours de décompression post-certification.

### Juillet-Août : Finalisation Plateforme
*   [ ] **Hardening** : Finalisation complète de l'infrastructure (sécurité, performance, contenus).
*   [ ] **Produit** : Mise en ligne de l'offre publique : **vitrine consulting DevSecOps + produit technique**.
*   [ ] **Contenu** : Rédaction d'articles blog Ghost (retours d'expérience, études de cas, veille sécurité).
*   [ ] **Portfolio** : Repos publics GitHub (mirror GitLab → GitHub) comme vitrine technique.

### Septembre : Phase d'Action Intensive
Deux branches parallèles :
*   **Branche A — CDI** : Ciblage d'entreprises françaises dans les secteurs **sécurité, souveraineté/défense, intelligence artificielle**. La plateforme uyoop sert de portfolio vivant.
*   **Branche B — Freelance** : Consultant DevSecOps indépendant. Déploiement des compétences auprès de cibles sectorielles à définir. La plateforme uyoop sert de vitrine commerciale et d'outil de production.

> **Objectif fin août** : Solution uyoop **en ligne et accessible au grand public** — vitrine + offre consulting + produit technique opérationnel.

---

## Calendrier des Risques & Mitigations

| Échéance | Risque Identifié | Impact | Plan de Mitigation |
| :--- | :--- | :--- | :--- |
| **21 Février** | Migration Mail inachevée | Perte de mails entrants | Activation **Proton Mensuel (12.99€)** (Buffer). |
| **15 Mars** | Transferts Domaines bloqués | Coupure Web | Vérification codes EPP à J-30. |
| **Mai** | Crash Disque VPS | Perte Données | Backups S3 quotidiens + Test de restauration (DRP). |
| **Soutenance** | Panne "Effet Démo" | Échec Présentation | Environnement de secours ou Vidéo enregistrée. |

