# Plan de Reprise & Continuité d'Activité (PRA/PCA)

> **Statut** : V2 (6 Février 2026)
> **Objectif** : Formaliser les procédures de résilience, les objectifs de reprise et les scénarios de sinistre pour l'infrastructure uyoop.

---

## 1. Objectifs de Reprise

| Indicateur | Cible | Justification |
| :--- | :--- | :--- |
| **RPO** (Recovery Point Objective) | **< 24h** | Backup S3 quotidien (Restic/Velero). Perte max = dernière sauvegarde nocturne. |
| **RTO** (Recovery Time Objective) | **< 4h** | Re-provisioning Ansible + restauration Restic depuis S3. Testé via DRP. |
| **MTTR** (Mean Time To Repair) | **< 2h** (services critiques) | Mailcow + Nextcloud prioritaires. Ghost/SonarQube restaurés en second. |

---

## 2. Classification des Services (Criticité)

| Service | Criticité | RPO | RTO | Justification |
| :--- | :--- | :--- | :--- | :--- |
| **Mailcow** (mail.cjenti.com) | 🔴 Critique | < 24h | < 2h | Perte de mail = perte de communication. MX fallback possible (Proton 12.99€/mois). |
| **Nextcloud** (drive.cjenti.com) | 🔴 Critique | < 24h | < 2h | Données personnelles et professionnelles. Sync client = copie locale disponible. |
| **Ghost** (blog.uyoop.fr) | 🟡 Important | < 24h | < 4h | Vitrine publique. Indisponibilité tolérable quelques heures. |
| **Authelia** | 🟡 Important | < 24h | < 2h | Sans SSO, accès direct temporaire possible (bypass documenté). |
| **GitLab** (gitlab.com SaaS) | 🟢 Faible | N/A | N/A | SaaS — résilience gérée par GitLab Inc. Mirror local = backup passif. |
| **GitLab Runner** (VPS-2) | 🟢 Faible | N/A | < 4h | CI/CD peut attendre. Re-enregistrement Runner = 15 min. |
| **SonarQube / Open WebUI** (VPS-2) | 🟢 Faible | < 7j | < 1j | R&D — pas de données irremplaçables. Redéploiement Helm/Docker. |

---

## 3. Stratégie de Sauvegarde (Règle 3-2-1)

```
3 copies des données :
├── 1. NVMe local (VPS-3) — données chaudes
├── 2. OVH Object Storage S3 — backup chiffré (Restic, AES-256)
│      └── Object Lock activé (immuabilité anti-ransomware)
└── 3. Disque dur local Admin — copie froide mensuelle (Rclone sync)

2 supports différents : NVMe + S3 + HDD
1 copie hors-site : S3 OVH (datacenter distinct) + HDD local
```

### Détail des Sauvegardes

| Donnée | Outil | Fréquence | Rétention | Destination |
| :--- | :--- | :--- | :--- | :--- |
| **Mailcow** (mails + config) | `restic` + script cron | Quotidien 03h00 | 30 jours glissants | S3 OVH (Object Lock) |
| **Nextcloud** (fichiers + BDD) | `restic` + `mysqldump` | Quotidien 03h30 | 30 jours glissants | S3 OVH (Object Lock) |
| **Ghost** (content + BDD) | `restic` + `mysqldump` | Quotidien 04h00 | 14 jours glissants | S3 OVH |
| **K3s etcd / manifests** | `k3s etcd-snapshot` + Git | Quotidien 02h00 | 14 jours | S3 + gitlab.com |
| **Configs système** | Ansible playbooks (Git) | À chaque commit | Infini (Git history) | gitlab.com + GitHub mirror |
| **Snapshot VPS OVH** | OVH API / Console | Auto (1/jour inclus gamme 2026) | 1 jour (gratuit) ou 7 jours (3.96€/mois) | OVH infra |

---

## 4. Scénarios de Sinistre & Procédures

### Scénario A : Panne Service Unique (ex: Mailcow crash)
**Impact** : 1 service down, infra intacte.
**Procédure** :
1. Diagnostic : `docker compose logs` (Mailcow) ou `kubectl describe pod` (K3s)
2. Restart : `docker compose restart` ou `kubectl rollout restart deployment/<service>`
3. Si persistant : Restaurer depuis backup Restic :
   ```bash
   restic -r s3:s3.rbx.io.cloud.ovh.net/uyoop-backup restore latest --target /tmp/restore-mailcow
   # Vérifier intégrité, puis remplacer volumes
   docker compose down && rsync -av /tmp/restore-mailcow/ /opt/mailcow/ && docker compose up -d
   ```
4. Vérification : test envoi/réception mail, check SpamScore.

**Temps estimé** : 30 min - 1h.

### Scénario B : Perte Complète VPS-3 (Crash Disque / Compromission)
**Impact** : Tous les services down.
**Procédure** :
1. **Commander** un nouveau VPS-3 via OVH (ou restaurer snapshot si disponible).
2. **Provisionner** : `ansible-playbook -i inventory/prod site.yml` (rôles `common`, `security`, `docker`, `k3s`).
3. **Restaurer Mailcow** : Restic restore depuis S3 → `/opt/mailcow/` → `docker compose up -d`.
4. **Restaurer K3s** : Restore etcd snapshot → redéployer manifests depuis Git.
5. **Restaurer Nextcloud/Ghost** : Restic restore volumes + import BDD dumps.
6. **DNS** : Vérifier pointage vers nouvelle IP (si changement). TTL court recommandé (300s).
7. **Validation** : Check tous les services, certificats Let's Encrypt, scores mail.

**Temps estimé** : 2h - 4h (objectif RTO).

### Scénario C : Compromission Sécurité (Intrusion détectée)
**Impact** : Intégrité douteuse de l'ensemble.
**Procédure** :
1. **Isolation immédiate** : `ufw deny incoming` ou shutdown VPS via console OVH.
2. **Forensics** : Snapshot disque pour analyse post-mortem (ne pas écrire dessus).
3. **Rebuild from scratch** : Nouveau VPS → Ansible → Restore backups S3 **antérieurs à la compromission**.
4. **Rotation** : Tous les secrets (clés SSH, tokens GitLab, mots de passe BDD, Sealed Secrets re-seal).
5. **Post-mortem** : Document d'analyse, mise à jour des règles CrowdSec, renforcement UFW.

**Temps estimé** : 4h - 8h (inclut forensics).

### Scénario D : Indisponibilité prolongée (>24h)
**Plan de continuité** :
*   **Mail** : Activation Plan B **Proton Unlimited** (12.99€/mois). Import backup mails.
*   **Drive** : Clients Nextcloud desktop ont une copie locale. Fallback temporaire sur OneDrive/Google Drive.
*   **Web** : Page statique "maintenance" hébergée sur GitHub Pages.
*   **Code** : gitlab.com SaaS non impacté. Mirror GitHub disponible.

---

## 5. Test DRP (Disaster Recovery Plan)

### Exercice Planifié : "Journée du Chaos" (Juin 2026, pré-soutenance)
**Objectif** : Prouver la capacité de restauration complète devant le jury.

**Protocole** :
1. Suppression volontaire du namespace `prod` sur K3s.
2. Arrêt de Mailcow Docker Compose.
3. Chrono lancé ⏱️.
4. Restauration complète depuis backups S3 uniquement.
5. Validation : tous les services up + test fonctionnel.

**Livrable** : Vidéo screen-capture + rapport chronométré (temps réel vs RTO cible).

### Fréquence des Tests
| Type | Fréquence | Responsable |
| :--- | :--- | :--- |
| Restore unitaire (1 service) | Trimestriel | Admin (cj) |
| DRP complet (tous services) | Biannuel | Admin (cj) |
| Vérification intégrité backups (`restic check`) | Mensuel (cron) | Automatisé |
| Test restauration backup aléatoire | Mensuel | Admin (cj) |

---

## 6. Matrice de Communication

| Événement | Qui prévenir | Canal | Délai |
| :--- | :--- | :--- | :--- |
| Panne service (< 1h) | Personne (auto-résolu) | — | — |
| Panne service (> 1h) | Utilisateurs impactés | Mail perso / Signal | < 30 min |
| Perte VPS complète | Tous utilisateurs | Mail fallback (Proton) + page maintenance GitHub Pages | < 1h |
| Compromission sécurité | Tous utilisateurs + CNIL (si données perso) | Mail + téléphone | < 24h (RGPD) |

---

*"Un backup non testé est un backup qui n'existe pas."*
