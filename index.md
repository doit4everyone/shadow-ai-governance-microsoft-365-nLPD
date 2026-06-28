---
title: "Gouvernance Shadow AI Microsoft 365 — Guide MDCA + Purview pour PME suisses (nLPD) | DoIt4Everyone"
description: "Guide complet de gouvernance du Shadow AI en environnement Microsoft 365 Business Premium : MDCA, Purview DLP, DSPM for AI, Insider Risk Management, DLP inline Edge for Business. Conformité nLPD pour PME suisses."
lang: fr
permalink: /
---
<style>
  header, footer { display: none !important; }
  .wrapper {
    max-width: 900px !important;
    margin: 0 auto !important;
    float: none !important;
    position: relative !important;
    padding: 40px 20px !important;
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif !important;
    font-size: 1.1em !important;
  }
  section {
    width: 100% !important;
    float: none !important;
    margin: 0 !important;
    padding-top: 0 !important;
  }
  h1, h2 { text-align: center; }
  table { width: 100%; display: table; margin: 20px 0; }
</style>

# 🛡️ Gouvernance Shadow AI Microsoft 365

<p align="center">
<strong>MDCA + Microsoft Purview — Guide complet Environnement Microsoft 365</strong><br>
<em>Conformité nLPD — PME Suisse 🇨🇭 — Version 1.1</em>
</p>

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Disponible-brightgreen)](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## 📖 À propos

| | |
|---|---|
| **Licences de base** | Microsoft 365 Business Premium |
| **Extension sécurité** | Microsoft Defender and Purview Suites for Microsoft 365 Business Premium |
| **Environnement** | Hybride (AD + Entra ID) et tenant pur Microsoft 365 |
| **Réglementation** | nLPD Suisse — Loi fédérale sur la protection des données |
| **Niveau** | Moyen — chaque étape expliquée en détail |
| **Version** | 1.1 — 2026 |
| **Objectif** | Gouverner et surveiller le Shadow AI. Protection des données et conformité nLPD |

> ℹ️ **Guide complémentaire requis (Cas A) :** si Microsoft Purview n’a jamais été configuré dans votre organisation, consultez d’abord [Configuration_Microsoft_Purview_2026](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/) avant de suivre ce guide — voir Partie 2.

---


## Chronologie de déploiement
Cette chronologie permet un déploiement progressif sans interruption de service. Chaque semaine représente un palier stable : si vous devez vous arrêter, l’architecture reste cohérente à la fin de chaque semaine.


| Période | Actions |
|---|---|
| Semaine 1 | Activation MDCA + onboarding MDE : activer le toggle MDCA-MDE (section 1.3.4), vérifier Cloud Discovery, onboarder les appareils Windows dans Defender for Endpoint. Network Protection en mode Block via Intune (section 1.3.5). Ces deux contrôles sont les plus critiques : sans eux, toute la chaîne de blocage est silencieuse. |
| Semaine 2 | Gouvernance MDCA : sanctionner les apps IA approuvées (Copilot M365), non-sanctionner les apps IA non autorisées (ChatGPT grand public, Gemini, etc.), créer la stratégie AutoBlock (section 1.6) et la politique de détection MDCA-DET-Shadow-AI-Generative (section 1.7.1). Politique BYOD Conditional Access (section 1.8.1). |
| Semaine 3 | DLP Purview : créer la politique principale (section 3.2) en mode Test, créer la politique Endpoint DLP (section 3.3.2) en mode Test, configurer les activités surveillées (section 3.3.3). Blocage réseau : Edge URLBlocklist via Intune (section 6.2), 6 indicateurs MDE URL/Domaines API (section 6.3). Valider les tests T1a et T1b (section 8). |
| Semaine 4 | DSPM for AI (section 4), IRM Shadow-AI-Navigation (section 5), Power Platform DLP-ConnecteursProd (section 7). Communication aux utilisateurs obligatoire avant activation IRM (art. 19 nLPD, section 9.5). Passer les politiques DLP de mode Test en mode Actif (bloquant) après validation des résultats dans l’Activity Explorer. État final cible : toutes les politiques DLP en mode Actif (Enable) avec actions Block ou Block with override. Le mode Test n’est qu’une phase transitoire de validation, pas un état permanent. Planifier les vérifications mensuelles (section 9.1). Semaine 5 (optionnelle, v1.1) : lier l’abonnement Azure PAYG (section 3.4.2), créer la politique DLP inline Edge (section 3.4.3), configurer le mode hybride pour les apps IA à usage autorisé (section 3.4.4). Tester le blocage prompt dans Edge avant passage en mode actif. |


## Positionnement de la solution
Cette solution est une architecture de contrôle centrée sur le réseau et la donnée dans l’écosystème Microsoft 365. Elle n’est pas un système de sécurité périmétrique universel. Les contrôles déployés voyagent avec les données et les flux réseau connus — pas avec les comportements invisibles au réseau.
✅ Ce que cette solution fait
- Blocage réseau des sites IA non autorisés sur les navigateurs gérés (Edge, Firefox, Chrome) via MDCA + MDE Network Protection
- Détection des applications IA non catalogées via Cloud Discovery
- Blocage du copier-coller de données sensibles (AVS, IBAN) sur les appareils Windows via Endpoint DLP
- Détection comportementale des usages IA via Insider Risk Management
- Gouvernance des connecteurs Power Platform et Copilot Studio (17 connecteurs bloqués)
- Surveillance des interactions Copilot M365 via DSPM for AI
- Blocage des prompts contenant des données sensibles (AVS, IBAN) dans les apps IA (ChatGPT, Gemini, DeepSeek) via DLP inline Edge for Business — avec notification Microsoft Purview et traçabilité complète (section 3.4, v1.1). Nécessite un abonnement Azure PAYG
- Traçabilité audit nLPD (art. 8, 9, 19, 24) sur les vecteurs couverts — rétention des journaux : 1 an avec Microsoft Defender and Purview Suites (90 jours par défaut sans l’add-on)

❌ Ce qu’elle ne remplace pas
- Antivirus / EDR : cette solution ne détecte pas les malwares ni les menaces sur les postes. Microsoft Defender (ou équivalent) reste un prérequis indépendant.
- SIEM : la corrélation des événements de sécurité relève d’une couche distincte (Microsoft Sentinel ou équivalent). MDCA génère des alertes, pas une détection unifiée.
- Structure de permissions : si un collaborateur a accès à un fichier qu’il ne devrait pas voir, cette solution ne corrige pas cette erreur de gouvernance. Une structure de droits cohérente est un préalable indispensable.
- Mobile iOS/Android : l’app Microsoft Defender existe sur iOS/Android et remonte des données Cloud Discovery, mais les indicateurs URL personnalisés et le blocage réseau ne sont pas supportés sur ces plateformes. Les smartphones personnels hors Intune ne sont pas couverts. La protection mobile (Intune App Protection Policy / MAM) est hors périmètre de ce guide.
- LLM locaux : Ollama, GPT4All et équivalents tournent sans trafic réseau externe. MDCA, les indicateurs MDE et l’URLBlocklist sont aveugles à ce vecteur. ⚠️ Nuance juridique : l’absence d’exfiltration réseau ne signifie pas absence de risque nLPD — le traitement local de données sensibles (ex. : alimenter un LLM local avec des dossiers RH ou médicaux) peut violer les principes de finalité et de minimisation (art. 6 nLPD) si ce traitement n’a pas été déclaré ou autorisé. Contrôle possible : AppLocker/WDAC via Intune, hors périmètre de ce guide.
- SaaS tiers à IA native : Notion AI, Salesforce Einstein, HubSpot AI utilisent les sous-domaines des apps SaaS autorisées — trafic invisible, transfert non maîtrisé au sens nLPD art. 8.
- Copier-coller de texte brut sans donnée sensible détectable : si le texte collé ne contient pas de SIT reconnu (AVS, IBAN), aucun contrôle ne le détecte. C’est le vecteur le plus fréquent et le moins contrôlable techniquement.

⚠️ Déployée sur des fondations fragiles (permissions SharePoint incohérentes, appareils non onboardés, Network Protection désactivé), cette solution donnera une fausse impression de sécurité. Déployée correctement, elle constitue une défense en profondeur efficace et démontrable face au PFPDT — dans un périmètre réaliste de 70 à 85 % de couverture sur un environnement Windows géré (voir matrice de couverture, section 9.7). Ce n’est pas une défense complète : mobile, SaaS IA native et LLM locaux restent hors périmètre.


---

## 🗂️ Sommaire complet

| Chapitre | Description |
|---|---|
| [Introduction — Le Shadow AI et les risques](docs/introduction.md) | Qu’est-ce que le Shadow AI, exemples concrets, risques de fuite de données, vecteur API, risques nLPD (art. 5, 6, 8, 9, 16, 24, 62), architecture de la solution, modèle de menace |
| [Partie 0 — Avant de commencer](docs/00-prerequis.md) | Licences requises, comptes et rôles, portails utilisés, installation PowerShell, checklist de départ |
| [Partie 1 — Microsoft Defender for Cloud Apps (MDCA)](docs/01-mdca.md) | Cloud Discovery, onboarding Defender for Endpoint, sanction des apps IA, AutoBlock, Conditional Access BYOD |
| [Partie 2 — Prérequis Microsoft Purview](docs/02-purview-prerequis.md) | Cas A (nouveau tenant), Cas B (guide suivi), Cas C (tenant existant) |
| [Partie 3 — Politiques DLP](docs/03-dlp.md) | DLP principale, Endpoint DLP presse-papier, DLP inline Edge for Business (v1.1), mode hybride |
| [Partie 4 — DSPM for AI](docs/04-dspm.md) | Surveillance des interactions IA via Purview DSPM |
| [Partie 5 — Insider Risk Management](docs/05-irm.md) | Détection comportementale du Shadow AI |
| [Partie 6 — Contrôles navigateur et Power Platform](docs/06-reseau.md) | Edge URLBlocklist Intune, indicateurs MDE URL/Domaines API |
| [Partie 7 — Power Platform et Copilot Studio](docs/07-power-platform.md) | Gouvernance des connecteurs — DLP-ConnecteursProd |
| [Partie 8 — Tests et validation](docs/08-tests.md) | Tests T1 à T4 — validation du déploiement |
| [Partie 9 — Gestion opérationnelle et maintenance](docs/09-maintenance.md) | Vérifications périodiques, communication utilisateurs (art. 19 nLPD), matrice des angles morts |

---

## 📚 Guide complémentaire

| Document | Description |
|---|---|
| [Configuration_Microsoft_Purview_2026](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/) | Configuration complète de Microsoft Purview depuis un tenant vierge : étiquettes de sensibilité, SITs personnalisés suisses (AVS, IBAN, données médicales), politiques DLP, auto-labelling, Endpoint DLP, DSPM for AI, Insider Risk Management |

---

## ⚖️ Avertissement

Ce guide constitue une aide à la mise en conformité technique. Il ne remplace pas un conseil juridique. Les références à la nLPD sont fournies à titre informatif et doivent être validées avec votre conseil juridique ou votre préposé à la protection des données avant mise en production.

---

ℹ️ *Références et aide à la rédaction assistées par IA, avec validation humaine finale et tests en tenant lab réel.*
