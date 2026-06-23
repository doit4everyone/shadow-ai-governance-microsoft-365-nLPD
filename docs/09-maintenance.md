---
title: "Partie 9 — Gestion opérationnelle et maintenance | Gouvernance Shadow AI Microsoft 365"
description: "Gestion opérationnelle : vérifications périodiques, automatisation du marquage des nouvelles apps IA, communication aux utilisateurs (art. 19 nLPD), matrice des angles morts."
lang: fr
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

[← Retour à l’index](../)

# Partie 9 — Gestion opérationnelle et maintenance


## 9.1 Vérifications périodiques recommandées



| Étape | Description |
|---|---|
| 1 | Vérifications hebdomadaires Microsoft Defender (security.microsoft.com) → Incidents et alertes → Alertes : vérifiez les nouvelles alertes Shadow AI (indicateurs MDE, MDCA). Microsoft Purview → Protection des données → Alertes : vérifiez les correspondances de règles DLP (DLP rule matched) sur les emails et documents. MDCA → Tableau de bord → Alertes : vérifiez les nouvelles applications découvertes et les activités à risque. |
| 2 | Vérifications mensuelles MDCA → Applications cloud → Cloud Discovery → Applications découvertes : filtrez par Catégorie = IA générative + Balise = Aucune valeur — vérifiez que MDCA-ACT-Shadow-AI-AutoBlock a bien traité toutes les nouvelles apps (les a taguées automatiquement). Si des apps sans tag subsistent, taguez-les manuellement comme non-sanctionnées. MDE (security.microsoft.com) → Paramètres → Points de terminaison → Règles → Indicateurs → URL/Domaines : vérifiez si de nouveaux domaines API IA doivent être ajoutés (voir section 9.3). Intune (intune.microsoft.com) → Appareils → Configuration → Edge - Restrictions Shadow AI : vérifiez l’état de déploiement et ajoutez de nouveaux domaines web si nécessaire. admin.powerplatform.microsoft.com → Gérer → Power Automate → Inventaire : vérifiez si de nouveaux flux ont été créés, en particulier ceux utilisant des connecteurs IA. |


## 9.2 Automatiser le marquage des nouvelles applications IA dans MDCA



| Étape | Description |
|---|---|
| → | Cette configuration est documentée en section 1.6 de ce guide, directement après la classification manuelle des applications (section 1.5). Voir section 1.6 → Créer la stratégie MDCA-ACT-Shadow-AI-AutoBlock. |


## 9.3 Découvrir et gérer les nouveaux domaines API IA



| Étape | Description |
|---|---|
| 1 | Méthode 1 : MDCA Cloud Discovery (recommandée) MDCA → Applications cloud → Cloud Discovery → Tableau de bord → Applications découvertes. Filtrez par Catégorie = AI - Model Provider ou IA générative. MDCA identifie automatiquement tous les domaines (y compris les sous-domaines API) contactés par les appareils MDE-onboardés. Cliquez sur une application pour voir les domaines exacts utilisés. Si un nouveau domaine API apparaît, ajoutez-le dans MDE → Indicateurs → URL/Domaines. |
| 2 | Méthode 2 : MDE Advanced Hunting security.microsoft.com → Repérage avancé → exécutez la requête : DeviceNetworkEvents \| where RemoteUrl contains « ai » or RemoteUrl contains « openai » or RemoteUrl contains « anthropic » \| summarize count() by RemoteUrl \| sort by count_ desc |
| 3 | Méthode 3 : Veille externe : sources de référence Suivez ces sources pour anticiper les nouveaux fournisseurs IA et leurs endpoints API : - Microsoft Tech Community Security Blog : techcommunity.microsoft.com/t5/security-compliance — annonces des nouvelles apps IA détectées par MDCA et mises à jour du catalogue. - OWASP AI Security & Privacy Guide : owasp.org/www-project-ai-security — référence sur les vecteurs d’attaque IA émergents et les nouveaux services à risque. - MITRE ATLAS : atlas.mitre.org — base de données des techniques d’attaque spécifiques aux systèmes IA. Utile pour identifier les endpoints API utilisés dans des scénarios d’exfiltration. - Hugging Face : huggingface.co/models — inventaire des nouveaux modèles IA publiés et de leurs API d’inférence (ex. api-inference.huggingface.co). Toute API populaire devient rapidement un vecteur Shadow AI. |
| 4 | Processus opérationnel : Qui fait quoi et à quelle fréquence ? Tableau des responsabilités (adapté aux PME sans RSSI dédié — l’admin IT assure généralement les deux rôles) : Fréquence mensuelle — Admin IT : - Vérifier MDCA Cloud Discovery : nouvelles apps IA découvertes sans tag → taguer manuellement si MDCA-ACT-AutoBlock n’a pas encore détecté la première occurrence. - Consulter MDE Advanced Hunting : identifier le trafic vers des domaines IA non répertoriés dans les 6 indicateurs existants. - Mettre à jour les indicateurs MDE si un nouveau domaine API est détecté (chemin : section 6.3). Fréquence trimestrielle — Admin IT ou RSSI : - Consulter OWASP AI Security et MITRE ATLAS pour identifier les nouveaux vecteurs d’exfiltration IA émergents. - Passer en revue HuggingFace (modèles tendance) et vérifier si leurs endpoints API (api-inference.huggingface.co) doivent être ajoutés aux indicateurs MDE. - Revoir la liste des apps sanctionnées/non-sanctionnées dans MDCA en lien avec l’évolution des outils IA approuvés par la direction. À la demande — Admin IT : - Ajouter un nouveau domaine API dans MDE Indicateurs → URL/Domaines dès qu’un nouvel outil IA est détecté en Advanced Hunting ou signalé par un utilisateur. - Documenter chaque ajout avec la date, le domaine, le fournisseur et la justification nLPD art. 8. |


## 9.4 Filtrage du contenu web MDE (blocage dynamique par catégorie)


| Étape | Description |
|---|---|
| ⚠️ | Catégorie « IA générative » non disponible, constat terrain (mai 2026) Le filtrage du contenu web MDE (security.microsoft.com → Paramètres → Points de terminaison → Règles → Filtrage du contenu web) est actif dans le tenant lab, mais les catégories disponibles en mai 2026 sont : Contenu pour adultes, Bande passante élevée, Responsabilité légale, Loisir, Sans catégorie. Aucune catégorie « IA générative » ou « Intelligence artificielle » n’est disponible. Le blocage des sites IA est assuré par les mécanismes suivants, qui couvrent l’ensemble des navigateurs (Edge, Firefox, Chrome) : - MDCA non-sanctionné → MDE (section 1.5 + toggle MDE activé — voir section 1.3.4) : blocage dynamique de toutes les apps IA du catalogue, tous navigateurs. - MDE indicateurs URL/Domaines (section 6.3) : blocage des domaines API IA (api.openai.com, api.anthropic.com, etc.), tous navigateurs. - Edge URLBlocklist via Intune (section 6.2) : blocage immédiat pour Edge uniquement, sans délai de propagation. À réévaluer lors des prochaines mises à jour Microsoft — une catégorie IA pourrait être ajoutée au catalogue WCF. |


## 9.5 Communication aux utilisateurs (art. 19 nLPD)



| Étape | Description |
|---|---|
| 1 | Obligations de transparence nLPD L’art. 19 nLPD impose d’informer les employés sur le traitement de leurs données comportementales. Les mécanismes déployés dans ce guide (MDCA, IRM, DLP, Indicateurs MDE) constituent un traitement de données personnelles. Documents à préparer avant mise en production : - Charte informatique mise à jour : mentionne explicitement la surveillance des accès aux outils IA non autorisés. - Communication employés : liste des outils IA autorisés (Copilot M365, Copilot Studio) et des outils bloqués (ChatGPT grand public, Gemini, Claude, etc.). - Procédure de demande d’exception : comment un employé peut demander l’accès à un outil IA spécifique pour un usage professionnel légitime. Message type à envoyer aux employés lors de l’activation du blocage DLP (mode Block) : « À compter du [date], l’envoi de documents ou d’informations sensibles (numéros AVS, données client, informations financières) vers des destinataires externes via email ou outils IA grand public sera bloqué automatiquement. Pour toute question, contactez [responsable IT]. » Note Firefox/Chrome : informez les collaborateurs que sous ces navigateurs, les sites IA bloqués affichent une erreur de type « connexion non sécurisée » ou « problème de sécurité ». Ce message signifie que le blocage fonctionne — pas qu’il y a une panne réseau. |


## 9.5.1 Cycle de vie d’un outil IA. Processus de demande et validation
Ce processus s’applique à tout nouvel outil IA dont un collaborateur souhaite l’usage dans le cadre professionnel. Il garantit la conformité nLPD et la cohérence avec les contrôles techniques déployés.


| Phase | Acteur | Durée typ. | Actions | Outils / Traces |
|---|---|---|---|---|
| Demande | Employé / Manager | 1-2 j | Soumettre via formulaire ou email dédié. Indiquer le cas d’usage et les données traitées. | Formulaire SharePoint ou ticket IT |
| Évaluation | Admin IT + DPO | 3-5 j | Vérifier conformité nLPD (art. 5, 8, 9). Analyser si l’outil est dans le catalogue MDCA. Vérifier CGU et localisation des données. | Catalogue MDCA + DSPM for AI |
| Validation | Direction / DPO | 1-2 j | Approuver ou refuser. Si approuvé : sanctionner dans MDCA et informer les utilisateurs (art. 19 nLPD). | MDCA → Sanctionner |
| Déploiement | Admin IT | 1 j | Ajouter à la liste des outils approuvés. Former les utilisateurs. Documenter les mesures de protection actives. | Intune + communication 9.5 |
| Monitoring | Admin IT | Continu | Vérifications mensuelles MDCA Cloud Discovery. Alertes MDCA-DET actives. Revoir l’usage via DSPM for AI. | Section 9.1 + 9.3 |
| Retrait | Admin IT + Manager | À la demande | Non-sanctionner dans MDCA. Informer les utilisateurs. Archiver les logs d’usage (nLPD art. 24). | MDCA → Non-sanctionner |


## 9.6 À surveiller : Shadow AI dans le Centre d’administration M365 (préversion)
Microsoft a lancé plusieurs surfaces de surveillance et de gouvernance des usages IA en 2026. Le tableau ci-dessous distingue ce qui est réellement accessible avec Business Premium + Defender and Purview Suites de ce qui exige une licence supérieure. Distinction validée en tenant lab (juin 2026).


| Surface | Disponibilité et périmètre |
|---|---|
| Tableau de bord de sécurité pour l’IA | Accès : ai.security.microsoft.com (ou via les portails Defender, Entra, Purview). ✅ Accessible avec Business Premium + Defender and Purview Suites — aucune licence supplémentaire. Consolide les signaux IA de Defender, Entra et Purview. Trois onglets : Vue d’ensemble, Inventaire de l’IA, Risque lié à l’IA. ⚠️ Note Cas A : le tableau reste vide tant qu’aucun agent Copilot Studio ni interaction Copilot M365 n’existe. Il surveille les agents IA et les interactions Copilot, pas les accès web grand public à ChatGPT ou Claude (suivis dans MDCA Cloud Discovery, section 1.4). |
| Fonctionnalités d’évaluation (preview MDCA) | Accès : security.microsoft.com → Paramètres → Applications cloud → Général → Fonctionnalités d’évaluation. ✅ Accessible avec Business Premium + Defender and Purview Suites. Activer le toggle et cocher les trois composants : Microsoft Defender XDR + Defender for Identity, Microsoft Defender for Endpoint, Microsoft Defender for Cloud Apps. Chemin spécifique à MDCA — ne pas confondre avec le toggle global Settings → Microsoft Defender XDR → General, qui n’est disponible qu’en mode Defender for Endpoint Plan 2 (non actif par défaut en Business Premium, qui tourne en mode Defender for Business). |
| Page Shadow AI (Centre d’administration M365) | Accès théorique : admin.microsoft.com → Agents → Shadow AI (programme Frontier). ❌ HORS PÉRIMÈTRE Business Premium (validé en tenant lab, juin 2026). Deux blocages cumulés : (1) le programme Frontier requiert une licence Microsoft 365 Copilot ou E3 pour s’activer ; (2) la page Shadow AI elle-même requiert une licence Microsoft 365 E3 minimum. Avec Business Premium seul, le menu Agents → Shadow AI n’apparaît pas. Périmètre fonctionnel (si licence E3) : détecte les agents IA locaux non autorisés (OpenClaw aujourd’hui ; Cursor, Claude Code CLI, Codex CLI annoncés) sur les appareils Windows gérés via Intune. Ne couvre pas les accès web ChatGPT/Claude/Gemini. |
| Conditional Access pour Agent Identities | Accès : Entra ID → Protection → Conditional Access → + New policy. Nom : CA-Block-High-Risk-Agent-Identities. Voir la procédure complète dans le tableau ci-dessous. ⚠️ Preview. Nécessite une licence Microsoft 365 Copilot. Cible les agents IA autonomes ayant une identité Entra — pas les collaborateurs humains. À activer si le tenant évolue vers Copilot Studio. |

✅ Recommandation pour un tenant Business Premium : utilisez ai.security.microsoft.com comme tableau de bord IA consolidé, et MDCA Cloud Discovery (section 1.4) comme source de vérité pour les usages web Shadow AI. La page Shadow AI Frontier et la détection d’agents locaux (OpenClaw, CLI) restent à réévaluer si vous passez à une licence E3 ou Copilot.



## 9.7 Matrice des angles morts. Vecteurs, couverture et efficacité
Vue consolidée conçue pour un audit nLPD. Légende : ✅ couvert \| ⚠️ partiel \| ❌ hors périmètre. L’efficacité estimée suppose Network Protection en mode Block (section 1.3.5) actif. Ces estimations sont basées sur des retours terrain et les caractéristiques connues de chaque contrôle ; elles dépendent du niveau de configuration réel et peuvent varier selon l’environnement. Type de contrôle par vecteur : Web navigateur, API IA externe, BYOD = prévention dure (blocage effectif, MDE Network Protection). Power Automate, Email/SharePoint = visibilité et gouvernance (MDCA, DLP en mode audit/blocage partiel). Copier-coller, Browser-Integrated AI = friction avec contournement possible (Endpoint DLP, DLP inline Edge — Block with override). SaaS tiers, Mobile, LLM local = détection ou absence de contrôle.

| Vecteur | Couverture guide | Risque résiduel | Urgence | Action complémentaire | Efficacité estimée |
|---|---|---|---|---|---|
| Web (navigateur géré) | ✅ Couvert | Faible | OK | Sections 1.5, 6.2, 1.3.4-5 | 95% — si Network Protection Block actif |
| API IA externe | ✅ Couvert | Faible | OK | Section 6.3 — 6 domaines bloqués | 90% — nouveaux endpoints à surveiller |
| BYOD non conforme | ✅ Couvert | Faible si actif | OK | Section 1.8.1 — vérifier périodiquement | 95% — si politique CA active |
| Power Automate / no-code | ✅ Couvert | Moyen | OK | Section 7 — revoir trimestriellement | 85% — nouveaux connecteurs possibles |
| Copier-coller | ⚠️ Partiel | Élevé | Surveiller | Section 3.3.3 — DLP presse-papier | 60% — données sensibles seulement |
| Browser-Integrated AI | ⚠️ Partiel | Élevé | Surveiller | Section 6.1 + ExtensionBlocklist Intune | 70% — protocoles non-HTTP non couverts |
| Email / SharePoint | ⚠️ Partiel | Élevé | Surveiller | Section 3 — passer en mode Block | 75% — en mode Audit actuellement |
| SaaS tiers avec IA native | ❌ Non couvert | Élevé | Évaluer | MDCA détecte si trafic visible. Notion AI, Salesforce Einstein, HubSpot AI utilisent des sous-domaines des apps SaaS autorisées, le trafic passe via des canaux déjà approuvés. Dans ces cas, les données sont traitées côté fournisseur SaaS sans visibilité réseau ni contrôle Microsoft direct ce qui constitue un transfert de données non maîtrisé au sens de la nLPD art. 8. | 10% — détection indirecte seulement |
| Mobile iOS/Android | ❌ Non couvert | Très élevé | PRIORITÉ | Intune App Protection Policy (hors périmètre). Note terrain : si Intune App Protection Policy (MAM) est active sur les apps gérées (Word, Outlook, Teams), le copier-coller vers une app non gérée est bloqué. Couverture partielle du vecteur copy-paste mobile. Configuration MAM hors périmètre de ce guide. | 0% — aucun contrôle actif |
| LLM local (Ollama, GPT4All) | ❌ Non couvert | Modéré (pas d'exfiltration) | Surveiller | AppLocker/WDAC + MDE DeviceProcessEvents | Pas un vecteur d'exfiltration ; risque = évasion des contrôles de contenu |

⚠️ Prérequis fondamental : Network Protection en mode Block (section 1.3.5) est requis pour toute efficacité réelle sur les vecteurs ✅. Vecteurs PRIORITÉ (orange) : Mobile et SaaS tiers, qui font réellement sortir des données, sont les investissements prioritaires du prochain cycle. Le LLM local ne fait pas sortir de données du périmètre : ce n'est pas un vecteur d'exfiltration mais une surface d'évasion (reformuler une donnée sensible pour déjouer la DLP par contenu avant de l'exfiltrer par un autre canal). À surveiller via AppLocker/WDAC et MDE DeviceProcessEvents, sans le traiter comme une fuite directe.


---

[← Partie 8 — Tests et validation](08-tests.md) | [Retour à l’index →](../)
