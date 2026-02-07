# Analyse FinOps & Budget Prévisionnel V2

> **Statut** : V2 (6 Février 2026) — Coûts vérifiés OVH, budget annualisé, gamme VPS 2026 documentée
> **Approche** : Rationalisation des coûts par la maîtrise technologique. Passage d'un modèle OPEX SaaS (pay-per-user) à un modèle IaaS (pay-per-resource).

## 1. Situation Initiale (2025 - Legacy)
Coûts dispersés, dépendance forte aux éditeurs, aucun contrôle sur la localisation précise des données.

| Fournisseur | Services Fournis | Coût Annuel HT | Mensuel TTC (Est.) |
| :--- | :--- | :--- | :--- |
| **Proton** | Mail, VPN, Drive (Suisse) | 119.88 € | ~12.00 € |
| **Microsoft** | Office 365, Exchange (US/EU) | 66.24 € | ~6.62 € |
| **O2Switch** | Hébergement mutualisé (France) | 84.00 € | ~8.40 € |
| **Registrars** | Portfolio Noms de domaine (uyoop.fr/com pro + cjenti.fr/com perso) | 125.00 € | ~12.50 € |
| **TOTAL** | | **395.12 € HT** | **~39.52 € TTC** |

---

## 2. Infrastructure Cible (2026 - Souveraine)
Basée sur OVHcloud (Roubaix/Gravelines), facturation prévisible et évolutive.

### Coûts Fixes (Infrastructure)
| Ressource | Rôle | Coût Mensuel TTC | Note |
| :--- | :--- | :--- | :--- |
| **VPS-3** | Nœud Core (Prod) | 14.28 € | Base 2026 stable |
| **Domaines** | Identité Numérique (uyoop.fr/com + cjenti.fr/com) | ~10.00 € | Centralisés OVH |
| **TOTAL IaaS** | | **~24.28 €** | Socle de base |

### Coûts Variables & Options (Flexibilité)
| Ressource | Type | Coût | Stratégie |
| :--- | :--- | :--- | :--- |
| **VPS-2 (IA)** | R&D (6 vCores / 12 Go / 100 Go NVMe) | ~7.14 € TTC / mois | *Optimisation* : Facturation horaire (Cloud) possible durant tests. |
| **S3 Object Storage** | Backup & Archives | ~0.01 €/Go/mois (One-Zone) | Stockage froid chiffré. ~1-3 €/mois selon volume. Pas de frais API/trafic chez OVH. |
| **Backup Premium VPS** | Sécurité | 3.96 € TTC/mois (optionnel) | Sauvegarde 7 jours glissants. 1 jour inclus gratuitement dans la gamme 2026. |
| **Risque Proton** | Assurance | 12.99 € / mois | **Plan B** : Abonnement mensuel si migration retardée (Budget "Sérénité"). |

> **Note gamme VPS OVH 2026** (engagement 12 mois) :
> VPS-1 (4 vCores/8 Go/75 Go) = 4.58€ TTC | VPS-2 (6 vCores/12 Go/100 Go) = 7.14€ TTC | **VPS-3 (8 vCores/24 Go/200 Go) = 14.28€ TTC** | VPS-4 (12 vCores/48 Go/300 Go) = 25.50€ TTC

---

## 3. Synthèse ROI & Valeur Ajoutée

### Gains Financiers
*   **Total Cible Optimisé** : ~24.28 € / mois (VPS-3 + Domaines, hors IA).
*   **Total Cible "Full Package"** : ~32.42 € / mois (avec VPS IA + S3 permanent).
*   **Budget Annuel Socle** : 24.28 × 12 = **291.36 € TTC/an**.
*   **Budget Annuel Full** : 32.42 × 12 = **389.04 € TTC/an**.
*   **Legacy 2025 annualisé TTC** : 395.12 × 1.20 = **474.14 € TTC/an**.
*   **Économie nette Full Package** : 474.14 - 389.04 = **~85 € TTC/an (18%)** à périmètre fonctionnel **largement supérieur** (Puissance de calcul dédiée, IA, CI/CD).
*   **Économie nette Socle seul** : 474.14 - 291.36 = **~183 € TTC/an (39%)**.

> **Astuce FinOps** : OVH propose **200€ de crédit Public Cloud** pour le premier projet. Utilisable pour tester l'Object Storage S3 et les AI Endpoints sans surcoût. ⚠️ *Offre promotionnelle, non garantie — ne pas intégrer dans le budget de base.*

### Budget "Worst Case" (Scénario Pessimiste)
Si tous les coûts variables et plans de secours sont activés simultanément :

| Poste | Coût Mensuel TTC |
| :--- | :--- |
| VPS-3 (Core-Prod) | 14.28 € |
| VPS-2 (AI-Lab) | 7.14 € |
| Domaines (4 NDD) | ~10.00 € |
| S3 Object Storage (~100 Go) | ~1.00 € |
| Backup Premium VPS (7 jours) | 3.96 € |
| Plan B Proton Unlimited | 12.99 € |
| **TOTAL Worst Case** | **~49.37 € / mois** |
| **Annualisé** | **~592 € TTC/an** |

> Ce scénario est temporaire (1-2 mois max pendant la migration). Il reste inférieur au coût Legacy + VPS combinés. Dès la migration terminée, Proton est résilié et le budget redescend en zone "Full Package".

### Coût One-Shot : Transfert Domaines
Le transfert de 4 NDD depuis O2Switch/registrars vers OVH implique un renouvellement anticipé d'un an par domaine.

| Domaine | Coût Transfert (1 an incl.) |
| :--- | :--- |
| uyoop.fr | ~7 € |
| uyoop.com | ~10 € |
| cjenti.fr | ~7 € |
| cjenti.com | ~10 € |
| **Total one-shot** | **~34 €** |

> Ce coût inclut 1 an de renouvellement. Il ne s'ajoute pas au coût annuel domaines, il le remplace pour la première année.

### Valeur Métier (Intangible)
Cette transition génère une valeur inestimable pour le profil professionnel :
1.  **Souveraineté** : Données 100% hébergées en France, code auditable.
2.  **Compétence** : Preuve technique d'un savoir-faire "Architecte Cloud" et culture FinOps.
3.  **Innovation** : Capacité à intégrer des LLM privés sans surcoût de licence (ex: Copilot à 19$/mois économisé).
4.  **BizOps Economie** : L'auto-hébergement de **Dolibarr** + **Mautic** économise environ **50€/mois** de licences SaaS (type Quickbooks + Brevo).

![Graphique ROI](https://quickchart.io/chart?c={type:'bar',data:{labels:['Legacy_2025_(47€)','Cible_SaaS_(90€)','Cible_uyoop_(24€)'],datasets:[{label:'Coût_Mensuel_TTC',data:[39.52,90.00,24.28]}]}})
*(Graphique généré automatiquement pour illustration FinOps)*
