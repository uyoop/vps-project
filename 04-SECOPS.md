# 04. SECOPS : SÉCURITÉ OPÉRATIONNELLE

> **Statut** : V1.0 (7 Février 2026)
> **Philosophie** : "Zero-Trust" — Ne jamais faire confiance, toujours vérifier.

## 1. Périmètre de Défense (Layered Defense)

### Couche 1 : Réseau & Firewall
*   **Infrastructure** : Le VPS n'expose que les ports strictement nécessaires.
*   **UFW (Uncomplicated Firewall)** : 
    *   Allow : 80/443 (Web), 22 (SSH - IP restreintes si possible), 25/587/993 (Mail), 51820 (WireGuard).
    *   Deny : Tout le reste (Défaut).
*   **CrowdSec** : IPS (Intrusion Prevention System) collaboratif.
    *   *Rôle* : Analyse les logs (Traefik, SSH, Authelia) et bannit les IP malveillantes.
    *   *Partage* : Les bans sont partagés avec la communauté mondiale (Intelligence Collective).

### Couche 2 : Identité & Accès (IAM)
*   **Authelia** : Portail d'authentification unique (SSO).
    *   *MFA* : 2FA obligatoire (TOTP/YubiKey) pour tous les services d'admin (Traefik Dashboard, Portainer, Longhorn).
*   **SSH** :
    *   Clés Ed25519 uniquement.
    *   `PermitRootLogin no`.
    *   `PasswordAuthentication no`.

### Couche 3 : Application (K8s)
*   **NetworkPolicies** : Isolation des namespaces. `prod` ne peut pas parler à `dev`.
*   **SecurityContext** : Les conteneurs ne tournent pas en `root` (sauf nécessité absolue).
*   **Scan** : Trivy scanne les images Docker pour les CVE connues.

### Couche 4 : Données & Secrets
*   **Chiffrement au repos** : Disques LUKS (si possible sur VPS) ou chiffrement applicatif (Nextcloud/Mailcow).
*   **Secrets Management** :
    *   **Phase 1** : Variables d'env via GitLab CI (Masquées).
    *   **Phase 2** : Sealed Secrets (Secrets chiffrés asymétriquement dans Git).
    *   **Phase 3** : HashiCorp Vault (Secrets dynamiques, rotation automatique).

## 2. Procédure de Réponse à Incident (IR)

### Cas 1 : Compromission SSH suspectée
1.  **Isoler** : Couper réseau (`ufw default deny incoming`).
2.  **Snapshot** : Faire un snapshot VPS via API OVH (Preuve légale).
3.  **Analyser** : Démarrer en mode Rescue, monter le disque, analyser logs (`/var/log/auth.log`, `.bash_history`).
4.  **Reconstruire** : Ne jamais "nettoyer" un serveur compromis. On le détruit et on redéploie via Ansible.

### Cas 2 : DDoS
*   **Mitigation** : Activer le "VAC" (Vacuum) OVH en mode permanent.
*   **Cloudflare** : Brouter le trafic DNS via Cloudflare (Mode "Under Attack") si l'IP est ciblée.

## 3. Conformité & Veille
*   **Veille** : Abonnement CERT-FR.
*   **Audit** : Scan régulier avec **OpenVAS** ou **Lyis**.
