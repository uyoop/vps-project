# 07. BIZOPS : GESTION D'ENTREPRISE (DOLIBARR)

> **Statut** : V1.0 (7 Février 2026)
> **Objectif** : Doter l'activité professionnelle d'un ERP/CRM souverain, robuste et intégré.

## 1. Pourquoi Dolibarr ?
Dans une approche **FinOps** et **Souveraine**, l'usage d'un SaaS tiers (Quickbooks, Tiime) pose problème : coût mensuel (20-50€/mois), données hébergées hors contrôle.
**Dolibarr** (Open Source Français) couvre 100% des besoins :
*   **CRM** : Gestion prospects/clients.
*   **Ventes** : Devis, Commandes, Factures (Conforme Loi Anti-Fraude TVA).
*   **Projets** : Suivi des temps (Timesheets) pour la facturation freelance.
*   **Comptabilité** : Export FEC pour le comptable.

## 2. Architecture Technique
Dolibarr est déployé dans le namespace `prod-gestion` sur le cluster K3s (VPS-3).

| Composant | Technologie | Specs | Rôle |
| :--- | :--- | :--- | :--- |
| **App** | Docker `dolibarr/dolibarr` | PHP 8.x | Frontend/Backend |
| **Database** | MariaDB 10.11 | `prod-db` | Stockage structuré |
| **Documents** | Volume Persistent (PVC) | NVMe / S3 | Stockage PDF Factures/Devis |
| **Sécurité** | Traefik + Authelia | OIDC | Protection de l'accès (2FA obligatoire) |

## 3. Stratégie de Résilience (Critique)
Les données de facturation sont **vitales**.
*   **RPO (Perte Max)** : < 1 heure.
*   **Backup DB** : Dump SQL toutes les heures (CronJob K8s) -> S3 (Object Lock).
*   **Backup Fichiers** : Restic sur le volume `/var/www/documents`.
*   **Plan de Secours** : En cas de crash total du VPS, possibilité de remonter un Dolibarr local (Docker Desktop) et d'injecter le dump SQL pour facturer en urgence.

## 4. Intégration BizOps (Mautic <-> Dolibarr)
L'objectif est d'automatiser le cycle de vie client :
1.  **Marketing (Mautic)** : Capture du lead (Formulaire Web), Nurturing (Newsletter).
2.  **Conversion** : Le prospect devient "Chaud".
3.  **Sync (Zapier/n8n/Script)** : Création automatique du Tiers/Contact dans **Dolibarr**.
4.  **Vente (Dolibarr)** : Émission du Devis -> Signature -> Facture.

## 5. Maintenance & Mises à jour
*   **Règle d'or** : Pas de `latest`. On fige les versions mineures (ex: `20.0.x`).
*   **Procédure Upgrade** :
    1.  Clone de la DB Prod -> Dev.
    2.  Test de la migration Dolibarr en Dev.
    3.  Validation (Génération PDF, Login).
    4.  Application en Prod.

---
*Ce module transforme l'infrastructure technique en outil de production économique réel.*
