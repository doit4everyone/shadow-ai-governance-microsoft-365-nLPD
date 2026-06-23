---
title: "Introduction — Le Shadow AI et les risques pour l’entreprise | Gouvernance Shadow AI Microsoft 365"
description: "Qu'est-ce que le Shadow AI, exemples concrets en entreprise, risques de fuite de données, vecteur API non contrôlé, risques réglementaires nLPD (art. 5, 6, 8, 9, 16, 24, 62), architecture de la solution MDCA + Purview, modèle de menace pour PME suisse."
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

# Introduction — Le Shadow AI et les risques pour l’entreprise


## Qu'est-ce que le Shadow AI ?
Le terme Shadow AI (ou IA fantôme) désigne l'utilisation non contrôlée et non autorisée d'outils d'intelligence artificielle par les collaborateurs d'une organisation, sans validation préalable du service informatique ni de la direction.
Tout comme le « Shadow IT » désignait l'utilisation non autorisée de logiciels ou services cloud, le Shadow AI représente la même problématique appliquée aux outils d'IA générative : ChatGPT, Claude.ai, Gemini, Midjourney, Perplexity, et des dizaines d'autres services accessibles depuis n'importe quel navigateur web.


### Exemples concrets de Shadow AI en entreprise
Voici des situations réelles que vous pouvez rencontrer dans votre organisation :
- Un collaborateur copie un contrat client dans ChatGPT pour en obtenir un résumé rapide.
- Un comptable colle des données financières dans Gemini pour générer un tableau de bord.
- Un responsable RH utilise une IA externe pour rédiger des évaluations de performance contenant des données personnelles d'employés.
- Un développeur utilise GitHub Copilot sans licence validée par l'entreprise, exposant le code source propriétaire. ⚠️ Angle mort : GitHub Copilot est une extension IDE (VS Code, JetBrains) qui communique via api.github.com, invisible à MDCA Cloud Discovery et non bloqué par Edge BlockList ni Defender Web Content Filtering. Contrôle possible : bloquer les domaines copilot.github.com et api.github.com/copilot via MDE Network Protection, et définir une politique sur les extensions IDE autorisées.
- Un médecin dans une clinique utilise un chatbot IA pour reformuler des notes médicales contenant des données de patients.


> **🚫  IMPORTANT Dans tous ces cas, les données quittent le périmètre de sécurité de l'entreprise et sont transmises à des serveurs situés hors de la Suisse, hors de l'Union Européenne, sans accord de traitement des données, sans chiffrement garanti, et potentiellement utilisées pour réentraîner les modèles d'IA.**




## Les risques liés au Shadow AI

### Risque de fuite de données (Data Exfiltration)
Lorsqu'un collaborateur colle des données dans une IA externe ces données sont envoyées vers les serveurs du fournisseur de l'IA, généralement aux États-Unis ou dans d'autres pays hors Suisse. Le fournisseur peut :
- Stocker ces données de manière permanente.
- Les utiliser pour améliorer ses modèles (selon ses conditions d'utilisation).
- Les exposer en cas de violation de sécurité (data breach).
- Les transmettre à des tiers dans le cadre de partenariats commerciaux.

### Le vecteur d'exfiltration souvent ignoré : les API non contrôlées
Un point particulièrement important pour les équipes techniques : Purview seul ne peut pas bloquer les API appelées directement depuis du code ou des scripts. Voici pourquoi c'est dangereux :


> **Exemple concret : API non bloquée par Purview**
>
> Un développeur crée un script Python qui appelle directement l'API OpenAI (api.openai.com) depuis son poste de travail.
>
> Il envoie automatiquement les fichiers du dossier /Projets vers GPT-4 pour analyse.
>
> Purview voit cette activité comme du trafic réseau chiffré standard, il ne peut pas inspecter le contenu des appels HTTPS directs.
>
> Les données sensibles quittent l'entreprise sans aucune alerte DLP.
>
> => C'est pourquoi MDCA (Microsoft Defender for Cloud Apps) est indispensable : il identifie et catégorise ces applications via Cloud Discovery. Le blocage effectif des appels API est ensuite appliqué par MDE Network Protection (section 1.3.5) sur la base de cette catégorisation : MDCA identifie, MDE bloque.


Les vecteurs d'exfiltration via API incluent :
- Appels directs à api.openai.com, api.anthropic.com, generativelanguage.googleapis.com, etc.
- Intégrations via des outils no-code (Zapier, Make.com, n8n) qui utilisent des clés API personnelles.
- Extensions de navigateur qui envoient le contenu des pages web vers des IA externes.
- Plugins Microsoft Office tiers non validés qui intègrent des fonctions IA.
- IA intégrées nativement aux navigateurs (Browser-Integrated AI) : Gemini sidebar dans Chrome, Copilot dans Edge, ou extensions IA utilisant WebRTC ou des protocoles de communication non-HTTP. Ces outils contournent le filtrage d’URL classique car ils ne génèrent pas de requête HTTP vers un domaine bloqué, ils communiquent via des canaux chiffrés natifs du navigateur. Contrôle possible : désactiver les extensions non-approuvées via Intune (Profil Edge → ExtensionInstallBlocklist = *) et utiliser uniquement Edge for Business avec la liste d’extensions approuvées.
- LLM locaux (angle mort, mais pas d'exfiltration) : des outils comme Ollama, GPT4All, LM Studio ou Jan permettent de faire tourner un modèle IA directement sur le poste de travail, sans aucun trafic réseau externe. Important : comme la donnée ne quitte pas la machine, ce n’est pas une exfiltration au sens de la nLPD art. 8 ; le risque réel est un traitement non maîtrisé et l’usage du LLM comme étape d’évasion (reformuler une donnée sensible pour déjouer la DLP par contenu avant de l’exfiltrer par un autre canal). MDCA, les indicateurs MDE et l’URLBlocklist sont aveugles à ce vecteur. Un collaborateur peut analyser des données sensibles sans générer la moindre alerte. Contrôle possible : bloquer l’installation via Intune (AppLocker ou WDAC) et surveiller via MDE Advanced Hunting (DeviceProcessEvents). Ce vecteur échappe à toute détection basée sur le réseau, y compris MDCA et les indicateurs MDE. Il reste hors périmètre de la solution décrite dans ce guide.
- Applications mobiles synchronisées avec des données d'entreprise (OneDrive, SharePoint) et utilisant des IA locales ou cloud. Smartphones iOS/Android : un collaborateur peut copier-coller du contenu Teams/Outlook Mobile vers ChatGPT Mobile sans qu'aucun mécanisme de ce guide ne le bloque (MDE s'applique uniquement aux appareils Windows onboardés). Solution : Intune App Protection Policy, hors périmètre de ce guide.


### Risques réglementaires spécifiques à la Suisse, nLPD
La Suisse dispose depuis le 1er septembre 2023 de la nouvelle Loi fédérale sur la Protection des Données (nLPD), révisée pour s'aligner en grande partie sur le RGPD européen. Le Shadow AI crée des violations directes de plusieurs articles :




| Article nLPD | Risque lié au Shadow AI |
|---|---|
| Art. 5 let. c - Données sensibles | Les données de santé, biométriques, d'appartenance syndicale, religieuse ou politique envoyées à une IA externe constituent une violation grave. |
| Art. 6 - Finalité & Proportionnalité | Les données collectées pour un usage précis ne peuvent pas être réutilisées pour entraîner un modèle IA externe. |
| Art. 8 - Sécurité des données | L'obligation de mesures techniques et organisationnelles (MTO) est violée si des données peuvent librement quitter le périmètre sécurisé. |
| Art. 9 - Privacy by Design & by Default | La protection doit être intégrée dès la conception (by design) ET les paramètres les plus protecteurs s’appliquent par défaut (by default). Un collaborateur utilisant une IA non validée contourne ces deux principes. |
| Art. 16 - Transfert à l'étranger | Transférer des données personnelles vers un fournisseur IA aux USA exige un mécanisme de transfert valable : soit le fournisseur est certifié au titre du Swiss-U.S. Data Privacy Framework reconnu par la Suisse, soit des clauses contractuelles types (SCC) sont en place. À défaut, le transfert est illicite. La certification DPF doit être vérifiée fournisseur par fournisseur ; ce point doit être validé avec votre conseil juridique. En pratique, la quasi-totalité des fournisseurs d’IA générative grand public (OpenAI, Anthropic, Google DeepMind, Mistral, Meta) ne sont pas certifiés au volet suisse du DPF — ce qui rend le transfert illicite par défaut et justifie précisément les mesures de blocage de ce guide. |
| Art. 24 - Notification PFPDT | En cas de violation entraînant un risque élevé pour les personnes concernées, vous devez notifier le Préposé Fédéral dès que possible (art. 24 nLPD). Note : seules les violations à risque élevé nécessitent une notification PFPDT ; les violations de faible risque font l’objet d’une documentation interne uniquement. Sans journaux d’audit, vous ne pouvez pas reconstituer l’incident ni respecter cette obligation. |
| Art. 62 - Secret professionnel | Les professions soumises au secret (médecins, avocats, banquiers) risquent des sanctions pénales en cas de transmission de données clients à une IA externe. |


> **nLPD SUISSE La violation de la nLPD peut entraîner des sanctions pénales allant jusqu'à CHF 250'000.- pour les personnes physiques responsables (art. 60-63 nLPD). L'amende s'applique à la personne physique (le responsable), Exception : l'art. 64 nLPD prévoit que l'entreprise peut être condamnée à payer une amende jusqu'à CHF 50'000.- si l'identification de la personne physique responsable exigerait des efforts disproportionnés. Le Préposé Fédéral à la Protection des Données et à la Transparence (PFPDT) dispose de pouvoirs d'enquête et de sanction renforcés depuis la révision de la loi.**



## Pourquoi MDCA + Purview ? Une approche en deux couches
Pour bloquer efficacement le Shadow AI, il faut agir sur deux niveaux complémentaires :
Ce guide se concentre sur la couche MDCA (gouvernance du flux). La couche Purview (protection de la donnée — étiquettes, DLP, chiffrement RMS) est traitée dans le guide complémentaire Configuration_Microsoft_Purview_2026.docx — obligatoire uniquement si Purview n’a jamais été configuré dans votre organisation (Cas A, voir section 2.1). Plusieurs sections de ce guide y font référence sans en dupliquer le contenu.



| Couche | Rôle |
|---|---|
| Microsoft Defender for Cloud Apps (MDCA) | Surveille et contrôle le trafic réseau. Détecte les applications cloud non autorisées (dont les IA externes). Peut bloquer l'accès à des domaines ou des applications entières. Visible dans le portail Microsoft Defender. |
| Microsoft Purview | Protège les données elles-mêmes. Classe et étiquette les fichiers sensibles. Applique des règles DLP qui empêchent le partage de fichiers confidentiels. Audite toutes les activités. Surveille les interactions avec Copilot M365. |


> **Analogie simple pour comprendre**
>
> Imaginez votre entreprise comme un bâtiment sécurisé :
>
> - MDCA = le garde à l'entrée qui contrôle qui peut sortir et vers où.
>
> - Purview = les casiers à clé à l'intérieur qui protègent les documents sensibles.
>
> Les deux sont nécessaires : un garde sans casiers laisse les documents accessibles à l'intérieur. Des casiers sans garde n'empêchent pas de copier le contenu avant de sortir.




## Architecture de la solution

> **🎯  À LIRE EN PREMIER - Périmètre réel de protection Correctement déployée avec Network Protection en mode Block, cette pile native Microsoft (Business Premium + extension Defender et Purview) traite efficacement les vecteurs où la donnée sort par le réseau : sites d'IA dans les navigateurs gérés, appels API directs, partage externe par email/SharePoint, appareils BYOD non conformes et flux Power Platform. Elle ne couvre pas totalement trois vecteurs, qui restent le risque résiduel principal : le copier-coller manuel de texte vers un outil IA autorisé ou non encore catalogué (l'usage le plus fréquent), le mobile (smartphones personnels hors MDE) et les SaaS tiers à IA native (Notion AI, Salesforce Einstein) dont le trafic passe par des canaux déjà approuvés. Ces vecteurs exigent des mesures complémentaires (Intune App Protection pour le mobile, sensibilisation, revue contractuelle), détaillées en section 9.7. Objectif réaliste : réduire fortement les fuites par les canaux réseau, documenter la surveillance pour la nLPD et identifier clairement ce qui reste ouvert. Ce guide ne prétend pas à une couverture totale du Shadow AI, et la section 9.7 énonce sans détour les angles morts.**


Ce guide couvre la configuration complète des éléments suivants :




| Composant | Ce que vous allez configurer |
|---|---|
| MDCA - Discovery | Inventaire automatique de toutes les applications IA utilisées dans votre organisation. |
| MDCA - App Control | Blocage ou restriction des applications IA non approuvées. |
| MDCA - Policies | Alertes automatiques en cas d'utilisation d'une IA non autorisée. |
| Purview - Labels | Étiquettes de sensibilité sur vos documents (Public, Interne, Confidentiel, Secret). |
| Purview - DLP | Règles qui empêchent l'envoi de fichiers sensibles vers l'extérieur. |
| Purview - DSPM for AI | Surveillance spécifique des interactions IA (Copilot M365 inclus). |
| Conditional Access | Bloquer l'accès aux ressources M365 depuis les appareils non gérés (BYOD). Note : le CA ne peut pas bloquer les URLs publiques (chatgpt.com, etc.), utilisez MDCA + MDE Network Protection (sections 1.3.4-5) et Edge BlockList (section 6) pour cela. Note : le Web Content Filtering MDE (section 9.4) ne dispose pas de catégorie IA en mai 2026. |
| Endpoint DLP | Bloquer le copier-coller de données sensibles vers des applications non autorisées. |


## Modèle de menace Shadow AI, Classification pour PME suisse
Cette classification permet de comprendre quels profils sont couverts et lesquels représentent un risque résiduel. Elle sert de base pour prioriser les investissements complémentaires hors périmètre Microsoft.


| Profil | Description | Exemple PME suisse | Probabilité | Impact / Couverture guide |
|---|---|---|---|---|
| Opportuniste | Employé productif sans intention malveillante | Comptable colle un No. AVS dans Claude pour résumer | Élevée | Élevé : ✅ Couvert : DLP + MDCA non-sanctionné |
| Automatisé | Script ou pipeline CI/CD qui appelle une API IA | Développeur avec pipeline GitHub → api.openai.com | Moyenne | Élevé : ✅ Couvert : indicateurs MDE URL/API |
| Intégré | IA native dans un outil SaaS autorisé ou LLM local | Copilot Teams ou Ollama installé sur le laptop | Croissante | Très élevé : ⚠️ Partiel : SaaS natif = fuite possible ; LLM local = évasion (pas d'exfiltration) |
| Malveillant | Exfiltration délibérée via canal IA avant départ | Employé en partance reformulant des données via LLM local avant exfiltration par un canal personnel | Faible | Critique : ❌ Non couvert ; exfiltration via canal personnel (USB, webmail), le LLM local n'étant qu'une étape d'évasion |

⚠️ Le LLM local ne fait pas sortir de données du périmètre : ce n'est pas un vecteur d'exfiltration mais une surface d'évasion (reformuler une donnée sensible pour déjouer la DLP par contenu). Le vrai risque résiduel le plus élevé est l'insider malveillant qui exfiltre par un canal personnel (USB, webmail), à traiter par AppLocker/WDAC + surveillance comportementale MDE + politique RH.


## Architecture de contrôle Shadow AI. Couverture par couche

Ce modèle mappe chaque couche de l’architecture IT aux menaces Shadow AI correspondantes et aux contrôles déployés dans ce guide. Il permet de prouver la couverture systémique et d’identifier les lacunes résiduelles. Note de lecture : cette architecture représente la couverture théorique cible ; la section 9.7 documente la couverture réelle avec les risques résiduels et l’efficacité estimée par vecteur.



| Couche | Menace Shadow AI | Contrôle déployé | Section | Statut |
|---|---|---|---|---|
| Utilisateur | Usage volontaire ou inconscient d’IA non validée | Communication nLPD + charte + sensibilisation | 9.5 | ⚠️ Partiel, dépend de l’adoption |
| Appareil | Apps IA installées localement (LLM, extensions) | MDE onboarding + Intune + Network Protection Block | 1.3, 1.3.5 | ✅ Windows gérés ❌ Mobile hors périmètre |
| Réseau | Trafic vers domaines IA externes | MDCA non-sanctionné + indicateurs MDE + URLBlocklist Edge | 1.5, 6.2, 6.3 | ✅ Couverture effective sur endpoints Windows gérés, sans contournement réseau (VPN, DNS externe, DoH) (Network Protection actif). Hors périmètre : iOS/Android |
| Application | Apps cloud IA non autorisées | MDCA Cloud Discovery + sanctionner/non-sanctionner + AutoBlock | 1.5, 1.6 | ✅ Couvert catalogue MDCA ❌ SaaS tiers hors catalogue |
| Données | Fuite de données sensibles (AVS, IBAN, santé) | DLP Purview + IRM + DSPM for AI + Endpoint DLP | 3, 4, 5 | ✅ Couvert sur appareils MDE ⚠️ Copier-coller résiduel |
| Comportement | Patterns anormaux non détectés | MDCA alertes + MDE Advanced Hunting + MDCA-DET policy | 1.7, 9.3 | ✅ Détection, réponse manuelle requise |


---

[← Retour à l’index](../) | [Partie 0 — Avant de commencer →](00-prerequis.md)
