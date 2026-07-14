# Gouvernance Shadow AI — Microsoft 365 (nLPD)

Guide complet de gouvernance du **Shadow AI** en environnement **Microsoft 365 Business Premium**, destiné aux PME suisses. Couvre MDCA, Purview DLP (y compris la DLP inline Edge for Business), DSPM for AI, Insider Risk Management et la gouvernance Power Platform, avec une cartographie complète des obligations **nLPD** (art. 5, 6, 7, 8, 9, 16, 19, 22, 24, 32, 61, 62, 64).

📖 **Site complet :** https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/

---

## Contenu du dépôt

```
.
├── _config.yml              # Configuration Jekyll (GitHub Pages)
├── index.md                  # Page d'accueil — sommaire et positionnement
├── docs/
│   ├── introduction.md        # Le Shadow AI, risques nLPD, architecture
│   ├── 00-prerequis.md        # Licences, comptes, rôles, checklist
│   ├── 01-mdca.md             # Microsoft Defender for Cloud Apps
│   ├── 02-purview-prerequis.md
│   ├── 03-dlp.md              # Politiques DLP + DLP inline Edge for Business
│   ├── 04-dspm.md             # DSPM for AI
│   ├── 05-irm.md              # Insider Risk Management
│   ├── 06-reseau.md           # URLBlocklist Edge, indicateurs MDE
│   ├── 07-power-platform.md   # Gouvernance Power Platform / Copilot Studio
│   ├── 08-tests.md            # Tests de validation
│   └── 09-maintenance.md      # Maintenance, matrice des angles morts
├── MDCA_Shadow_AI_v1.1.docx   # Version Word téléchargeable
└── MDCA_Shadow_AI_v1.1.pdf    # Version PDF téléchargeable
```

## Guide complémentaire

Ce guide suppose que **Microsoft Purview** est déjà configuré (étiquettes de sensibilité, SITs personnalisés, DLP de base). Si ce n'est pas le cas, commencez par :

📄 [Configuration_Microsoft_Purview_2026](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/)

## Méthodologie

Chaque procédure a été validée en tenant lab réel (Microsoft 365 Business Premium + Defender and Purview Suites). Les écarts entre le comportement observé en portail et la documentation Microsoft officielle sont signalés explicitement (⚠️ *finding terrain*) tout au long du guide.

## Statut

| | |
|---|---|
| **Version** | 1.1 — 2026 |
| **Licences de référence** | Microsoft 365 Business Premium + Defender and Purview Suites |
| **Réglementation** | nLPD (Suisse) |

## Licence

MIT — voir [LICENSE](LICENSE).

## Avertissement

Ce guide constitue une aide à la mise en conformité technique. Il ne remplace pas un conseil juridique. Les références à la nLPD doivent être validées avec votre conseil juridique ou votre préposé à la protection des données avant mise en production.

---

ℹ️ *Références et aide à la rédaction assistées par IA, avec validation humaine finale et tests en tenant lab réel.*
