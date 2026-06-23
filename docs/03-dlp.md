---
title: "Partie 3 — Politiques DLP : blocage de l’exfiltration de données | Gouvernance Shadow AI Microsoft 365"
description: "Politiques DLP Purview pour bloquer l'exfiltration de données vers les IA génératives : DLP principale, Endpoint DLP presse-papier, et DLP inline Edge for Business (v1.1) avec mode hybride."
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

# Partie 3 — Politiques DLP : blocage de l’exfiltration de données


## 3.1 Comprendre la DLP dans le contexte Shadow AI
La Data Loss Prevention (DLP) est le mécanisme qui empêche vos données sensibles de quitter le périmètre sécurisé de l'entreprise. Dans le contexte Shadow AI, la DLP couvre deux scénarios principaux :


| Scénario | Comment la DLP bloque |
|---|---|
| Utilisateur partage un fichier Confidentiel via un email vers une IA (ex: envoie à upload@chatgpt.com) | DLP détecte l'étiquette Confidentiel + destination externe et bloque l'envoi. |
| Utilisateur télécharge un fichier SharePoint pour l'uploader sur un site d'IA | DLP Endpoint détecte le téléchargement depuis SharePoint et bloque l'accès au clipboard ou au navigateur non autorisé. |
| Utilisateur copie-colle du texte contenant un numéro AVS dans une IA via le navigateur | DLP Endpoint détecte la présence d'un AVS dans le presse-papier et bloque le collage dans les applications non autorisées. |
| Script automatique appelle l'API OpenAI avec des données sensibles | MDCA (section 1) bloque au niveau réseau - DLP seul ne peut pas bloquer les appels API directs. |
| ⚠️ ANGLE MORT : Copier-coller de texte brut vers une IA via navigateur (~90 % des usages Shadow AI) Exemple : copier un extrait d’email, de contrat ou de code dans la zone de saisie de ChatGPT. Aucun fichier transféré, la DLP fichier est aveugle. Si la règle presse-papier Endpoint DLP n’est pas activée (souvent jugée trop intrusive en PME), la surveillance reste purement réactive (post-incident). | Couverture complète exige trois couches combinées :<br>(1) MDCA non-sanctionné + MDE (section 1.5 + toggle activé, section 1.3.4) : blocage de tous les sites IA catalogués, tous navigateurs, c’est la barrière primaire (blocage fort) ; (2) Endpoint DLP presse-papier activée sur les données sensibles (AVS, IBAN) pour les appareils MDE (section 3.3.3) — ⚠️ ceci reste un contrôle de conformité et de traçabilité, pas un blocage fort : il dépend de la détection SIT (faux négatifs possibles) et de l’activation effective sur le poste ; (3) sensibilisation utilisateurs (section 9.5) pour les cas résiduels. Note : le Web Content Filtering MDE (section 9.4) n’offre pas de catégorie IA en mai 2026. |




## 3.2 Créer la politique DLP principale - Blocage partage externe

| 1 | Créer une nouvelle politique DLP Purview → Protection contre la perte de données → Politiques. Cliquez + Créer une politique. |
|---|---|



| 2 | Choisir le modèle Dans la galerie de modèles, sélectionnez Personnalisé → Personnalisé (tout en bas de la liste). Cliquez Suivant. Note : il n’existe pas de modèle « Données personnelles suisses » dans Purview. Les règles de détection seront configurées manuellement à l’étape suivante. |
|---|---|



| 3 | Nommer la politique Nom : DLP - Blocage partage externe données sensibles (nLPD) Description : Bloque le partage de données personnelles suisses vers des destinataires externes. Couvre nLPD art. 8. Cliquez Suivant. |
|---|---|



| 4 | Sélectionner les emplacements 2026 : tous les emplacements sont cochés par défaut. Décochez : Appareils, Instances, Référentiels locaux. Activez uniquement : Courrier Exchange, Sites SharePoint, Comptes OneDrive, Messages Teams. Microsoft 365 Copilot est décoché par défaut — laisser ainsi (couvert par DLP-Protection-Copilot, guide Purview). ⚠️ Ne pas cocher Appareils — l’Endpoint DLP fait l’objet d’une politique dédiée configurée en section 3.3.2. L’activer ici créerait un conflit de politiques. |
|---|---|



| 5 | Configurer les règles de la politique Cliquez Créer une règle. Nom : Blocage - Données personnelles suisses vers l'externe Conditions : -  Le contenu contient : Types d'informations sensibles : Swiss Social Security Number AHV (confiance moyenne, min 1). ℹ️ Cas B (guide Configuration_Microsoft_Purview_2026.docx appliqué) : remplacez Swiss Social Security Number AHV par le SIT personnalisé AVS-Suisse-PME (rechercher «AVS» dans la liste). Ce SIT détecte les numéros AVS par format regex 756.XXXX.XXXX.XX, sans validation du checksum EAN-13, avec mots-clés de soutien en 4 langues nationales + anglais. Voir Configuration_Microsoft_Purview_2026.docx — Annexe C. -  ET Le contenu est partagé avec : des personnes extérieures à mon organisation. Actions : -  Restreindre l'accès ou chiffrer le contenu dans les emplacements Microsoft 365. -  Bloquer l'envoi d'email (pour Exchange). Notifications à l'utilisateur : -  Cochez Notifier les utilisateurs avec des conseils de stratégie. -  Message : Ce document contient des données personnelles protégées par la nLPD suisse. Le partage externe est bloqué. Rapport d'incident : Envoyer à l'administrateur sécurité. Cliquez Enregistrer. |
|---|---|



| 6 | Démarrer en mode Test Dans les options de la politique, sélectionnez Mode Test. Cochez Afficher les conseils de stratégie en mode test. Cliquez Suivant → Soumettre. Attendez 48h, consultez les correspondances, puis activez. |
|---|---|




## 3.3 DLP Endpoint - Bloquer le copier-coller vers des IA
La DLP Endpoint s'applique directement sur les appareils Windows et bloque les actions risquées comme le copier-coller de données sensibles dans le navigateur ou des applications non autorisées.


### 3.3.1 Prérequis - Onboarding des appareils

| 1 | Vérifier l'onboarding des appareils Purview → Protection contre la perte de données → Intégration de l'appareil. Vérifiez que vos appareils Windows 10/11 apparaissent dans la liste. Si des appareils manquent, consultez la section 1.3.3 pour l'onboarding via Intune. |
|---|---|



### 3.3.2 Créer la politique Endpoint DLP

| 1 | Créer la politique Endpoint DLP Purview → Protection contre la perte de données → Politiques → + Créer une politique. Modèle : Personnalisé → Personnalisé (tout en bas de la liste). Cliquez Suivant. Nom : DLP-Endpoint-Shadow-AI-CopyPaste Description : Bloque le copier-coller de données sensibles vers des applications IA non autorisées. Couvre nLPD art. 8. Cliquez Suivant. Unités d’administration : Répertoire complet. Cliquez Suivant. |
|---|---|
| 2 | Sélectionner l’emplacement Page Emplacements : décochez TOUT, puis cochez uniquement : - ✅ Appareils Laissez Exchange, SharePoint, OneDrive, Teams décochés — ils sont couverts par la politique 3.2. Une politique Endpoint DLP distincte évite les conflits de règles. Cliquez Suivant. |
| 3 | Créer la règle de blocage Page Paramètres de stratégie → sélectionnez Créer ou personnaliser des règles DLP avancées. Cliquez Suivant, puis + Créer une règle. Nom de la règle : Blocage - Copier-coller données sensibles vers IA Conditions — Le contenu inclut → Types d’informations sensibles : - Swiss Social Security Number AHV (confiance moyenne, min 1) - International Banking Account Number (IBAN) (confiance haute, min 1) Actions — activités Endpoint → cochez : - Copier dans le Presse-papiers : Bloquer - Copier dans une application non autorisée : Bloquer (la liste des applications non autorisées est définie dans les paramètres globaux — section 3.3.3 à configurer immédiatement après) - Télécharger vers un service cloud non autorisé : Bloquer Notification utilisateur : cochez Notifier les utilisateurs. Message : Ce contenu est protégé. Le copier-coller vers des applications non autorisées est bloqué (nLPD art. 8). Cliquez Enregistrer. Cliquez Suivant. |
| 4 | Démarrer en mode Test Page Mode de la stratégie → sélectionnez Mode test. Cochez Afficher les conseils de stratégie en mode test. Cliquez Suivant → Soumettre. Attendez 48h, consultez les correspondances dans Explorateur d’activités (Purview → Protection des informations → Explorateur d’activités), puis activez la politique. ⚠️ Prérequis indispensable : les appareils doivent être onboardés dans MDE et la stratégie de conformité Intune active (section 1.3.3). Sans onboarding, la politique Endpoint DLP est silencieuse. |



### 3.3.3 Configurer les activités surveillées par Endpoint DLP

| 1 | Accéder aux paramètres Endpoint DLP Purview → Paramètres → Protection contre la perte de données → Paramètres de l'appareil. Vérifiez que les activités suivantes sont surveillées : -  Copier dans le presse-papier. -  Copier sur un support amovible (USB). -  Télécharger vers un service cloud non autorisé. -  Imprimer. -  Copier dans une application non autorisée. |
|---|---|



| 2 | Ajouter les navigateurs non autorisés Dans Paramètres de l'appareil → Restrictions des navigateurs non autorisés. Si un utilisateur accède à ChatGPT via Firefox au lieu d'Edge, Endpoint DLP peut bloquer le téléchargement. Ajoutez Firefox, Brave, Opera dans la liste des navigateurs à restreindre pour le téléchargement de fichiers sensibles. Laissez Edge (géré par Defender) comme navigateur autorisé. ⚠️ Limite importante : ces restrictions s’appliquent aux téléchargements de fichiers depuis des sites sensibles. Elles ne bloquent pas le copier-coller manuel de texte dans la zone de saisie d’un service IA (ChatGPT, Gemini, etc.) via le navigateur, qui reste le vecteur Shadow AI le plus courant. Pour réduire ce risque : combinez avec l’Endpoint DLP section 3.3.2 («Copier dans le presse-papier») et la sensibilisation des utilisateurs (section 9.5). |
|---|---|



| 3 | Ajouter les applications non autorisées Dans Paramètres de l'appareil → Applications non autorisées. Ajoutez les applications qui ne doivent pas recevoir de données sensibles : -  Notepad++ (si utilisé pour exfiltrer via paste-and-upload) -  Applications de messagerie personnelle non managées. Cliquez Enregistrer. |
|---|---|




## 3.4 DLP inline Edge for Business — Protection prompt-level (v1.1)
Cette section introduit une approche complémentaire au blocage total (sections 1 à 6) : au lieu de bloquer l’accès aux sites IA, la DLP inline analyse les prompts en temps réel dans Edge for Business et bloque uniquement ceux contenant des données sensibles (AVS, IBAN). L’utilisateur peut utiliser ChatGPT pour des tâches légitimes ; seuls les prompts sensibles sont interceptés.


### 3.4.1 Choix du mode de protection
Trois modes de protection sont disponibles. Le mode hybride (recommandé) combine les avantages des deux premiers.

| Critère | Mode A — Blocage total (v1.0) |
|---|---|
| Principe | Aucun accès aux IA non autorisées. Les sites sont bloqués au niveau réseau (MDCA + MDE + URLBlocklist). Aucun bypass possible. |
| Prérequis | Business Premium + Defender and Purview Suites (aucun coût supplémentaire). |
| Force | Blocage dur — le seul qui empêche physiquement l’accès. Aucune donnée ne peut sortir vers le site bloqué. |
| Limite | Frustration des collaborateurs → contournement (smartphone, hotspot). L’admin ne voit pas les tentatives de contournement hors réseau. |



| Critère | Mode B — Usage contrôlé (v1.1) |
|---|---|
| Principe | Accès autorisé dans Edge for Business. Les prompts contenant des SITs (AVS, IBAN) sont bloqués avec notification Purview. L’utilisateur peut cliquer Continuer (override logué). |
| Prérequis | Business Premium + Defender and Purview Suites + abonnement Azure avec facturation à l’utilisation (PAYG). Coût variable selon le volume de prompts analysés — négligeable en PME. |
| Force | Le besoin est couvert → le contournement diminue. Traçabilité complète dans Purview Activity Explorer (utilisateur, timestamp, SIT détecté, action). |
| Limite | Le blocage est « Block with override » par design (Microsoft) — pas un blocage dur. L’utilisateur averti peut passer outre. Seuls les SITs reconnus sont détectés — le texte brut sans SIT passe. |



| Critère | Mode Hybride — Recommandé pour PME |
|---|---|
| Principe | Apps à fort usage (ChatGPT, Gemini) en Mode B (sanctionnées + DLP inline). Apps sans besoin métier (Grok, DeepSeek) en Mode A (bloquées). Copilot M365 sanctionné nativement + DSPM. |
| Prérequis | Identiques au Mode B (Azure PAYG). |
| Force | Défense en profondeur réaliste : blocage dur là où c’est nécessaire, friction + traçabilité là où c’est utile. Démontrable en audit nLPD art. 24. |
| Limite | Complexité admin moyenne — 3 couches à coordonner par app (MDCA, URLBlocklist, MDE indicateurs). |


⚠️ Le Mode A reste la recommandation par défaut si l’organisation n’a pas de besoin identifié d’IA générative. Le Mode Hybride s’applique quand les collaborateurs utilisent déjà des IA — mieux vaut contrôler que ignorer.


### 3.4.2 Prérequis — Abonnement Azure avec paiement à l’utilisation
La DLP inline Edge for Business est une fonctionnalité à paiement à l’utilisation (PAYG). Sans abonnement Azure lié au tenant, les toggles Edge for Business et Réseau restent désactivés dans l’assistant de création DLP. Finding terrain validé en juin 2026.

| 1 | Lier l’abonnement Azure Purview → Protection contre la perte de données → Stratégies. Cliquez sur le bandeau orange « Configurez la facturation avec paiement à l’utilisation » → Prise en main. Sélectionnez votre Souscription Azure et un Groupe de ressources existant (ou créez-en un, ex. : SHADOWS-IA). Cochez « Je confirme avoir lu et accepté les Conditions générales » → Activer. Purview crée automatiquement une ressource Azure (ex. : BSCULIER-AZURE-SERVICES dans le groupe de ressources). La propagation peut prendre quelques minutes. |
|---|---|
| ⚠️ | Coût et facturation Le coût est à l’utilisation uniquement — pas de frais fixes, pas d’engagement. En PME (<50 utilisateurs), le coût est négligeable. Consultez la facturation dans le portail Azure → votre groupe de ressources. |



### 3.4.3 Créer la politique DLP SHADOWS AI

| 1 | Démarrer l’assistant DLP Purview → Protection contre la perte de données → Stratégies → + Créer une stratégie. L’assistant propose deux chemins : « Appareils et applications d’entreprise » et « Trafic web en ligne ». Sélectionnez Trafic web en ligne. |
|---|---|
| 2 | Modèle et nom Catégorie : Personnalisé → Personnalisé. Suivant. Nom : DLP SHADOWS AI Description : Protège les prompts IA contenant des données sensibles nLPD via Edge for Business. V1.1. |
| 3 | Sélectionner les applications cloud Cliquez + Ajouter des applications cloud → onglet Périmètres d’application adaptatifs. Cochez : Toutes les applications d’IA non gérées (inclut Anthropic Claude, ChatGPT, Google Gemini, DeepSeek, et d’autres). Cliquez Add → Suivant. |
| 4 | Activer Edge for Business Page « Choisissez où appliquer la stratégie » — deux toggles apparaissent : ✅ Activez Edge for Business (« Détectez et protégez les données sensibles partagées avec des applications cloud non gérées, y compris les applications d’IA »). ⚠️ Le toggle « Réseau et navigateurs sécurisés non Microsoft » est disponible mais hors périmètre de cette procédure (nécessite SASE/SSE). Laissez-le Désactivé. |
| 5 | Créer la règle de blocage Paramètres de la stratégie → Créer ou personnaliser des règles de DLP avancées → + Créer une règle. Nom : Blocage - Prompts IA contenant des données sensibles (nLPD) Conditions → + Ajouter une condition → Le contenu inclut → Types d’infos confidentielles : - Swiss Social Security Number AHV (confiance moyenne, min 1) - International Banking Account Number IBAN (confiance moyenne, min 1) Cas B (guide Purview appliqué) : ajoutez aussi AVS-Suisse-PME et Medical-RH-Suisse. |
| 6 | Configurer les actions — IMPORTANT Actions → + Ajouter une action → Restreindre les activités du navigateur et du réseau. 4 options apparaissent. Cochez et configurez en Block uniquement les deux premières : ✅ Texte envoyé ou partagé avec des applications cloud ou IA → Block ✅ Fichier téléchargé ou partagé avec des applications cloud ou IA → Block ❌ Décochez : Texte reçu de l’application IA ou cloud ❌ Décochez : Fichier reçu de l’application IA ou cloud ⚠️ Finding terrain : si Block est activé sur « Texte reçu », le site IA est bloqué entièrement (la page ne charge plus). Ne cocher que « envoyé » pour obtenir une protection prompt-level sans bloquer le site. |
| 7 | Mode de la stratégie Démarrez en mode simulation (Exécuter la stratégie en mode simulation). Après validation dans l’Activity Explorer (section 8), passez en mode Activer immédiatement la stratégie. Cliquez Envoyer. |
| ℹ️ | Ce que Purview crée automatiquement Un bandeau jaune confirme : « Des stratégies supplémentaires en dehors de Purview seront créées [...] Les utilisateurs inclus dans la stratégie ne pourront pas utiliser de navigateurs non protégés. » Purview crée automatiquement les politiques Intune/Edge nécessaires sans intervention manuelle. Firefox et Chrome sont bloqués automatiquement pour les utilisateurs concernés. |



### 3.4.4 Configurer le mode hybride — 3 couches à coordonner
Pour passer une app IA du Mode A (bloqué) au Mode B (contrôlé), trois couches de blocage doivent être désactivées simultanément pour cette app. Si une seule couche reste active, le blocage persiste. Finding terrain validé en juin 2026.

| App IA | Mode recommandé |
|---|---|
| ChatGPT | Mode B (sanctionné + DLP inline) — usage le plus répandu, besoin réel en PME |
| Google Gemini | Mode B (sanctionné + DLP inline) — alternative courante |
| Grok, DeepSeek, Meta AI | Mode A (bloqué) — pas de besoin PME identifié |
| Perplexity AI | Au choix — évaluer le besoin métier |
| Copilot M365 | Sanctionné nativement — protégé par DSPM (section 4) |



| 1 | MDCA — Sanctionner l’app security.microsoft.com → Applications cloud → Catalogue d’applications cloud → cherchez l’app (ex. : ChatGPT) → changez de Non sanctionné → Sanctionné. |
|---|---|
| 2 | Intune URLBlocklist — Retirer le domaine intune.microsoft.com → Appareils → Configuration → profil Edge - Restrictions Shadow AI → retirez chatgpt.com de la liste URLBlocklist → Enregistrer. |
| 3 | MDE Indicateurs — Désactiver l’indicateur security.microsoft.com → Paramètres → Points de terminaison → Indicateurs → onglet URL/Domaines → désactivez ou supprimez l’indicateur chatgpt.com (et api.openai.com si présent). ⚠️ Les indicateurs auto-générés par MDCA (« Unsanctioned cloud app access was blocked ») sont supprimés automatiquement quand l’app est sanctionnée. Les indicateurs créés manuellement doivent être retirés manuellement. |
| ⚠️ | Réversibilité Pour revenir au Mode A, rétablissez les 3 couches dans l’ordre inverse : (1) non-sanctionnez l’app dans MDCA, (2) ajoutez le domaine dans l’URLBlocklist, (3) les indicateurs MDE se recréent automatiquement. |



### 3.4.5 Comportement par navigateur — Findings terrain

| Navigateur | Comportement observé (validé juin 2026) |
|---|---|
| Edge for Business (Mode B) | Site accessible. Les prompts contenant des SITs (AVS, IBAN) déclenchent une notification Microsoft Purview : « Votre organisation a protégé le contenu que vous essayez de coller » avec boutons Continuer (override logué) et Annuler. |
| Firefox (Mode B) | Application entièrement bloquée — message « Cette application a été bloquée par votre administrateur système ». Purview bloque automatiquement Firefox comme navigateur non protégé sans configuration Intune manuelle. |
| Chrome (Mode B) | Même comportement que Firefox — bloqué automatiquement. |
| Edge for Business (Mode A) | Site bloqué entièrement par MDCA + MDE Network Protection. Page « Cette page est bloquée ». |
| Firefox/Chrome (Mode A) | Site bloqué par Network Protection. Page de blocage ou erreur SSL (Firefox). |



### 3.4.6 Finding critique — « Block » = « Block with override » (by design)
La documentation Microsoft Endpoint DLP confirme que les actions clipboard/paste dans le navigateur sont conçues pour la détection et l’audit, pas pour le blocage silencieux. L’option « Block » pour les opérations de texte dans Edge se comporte comme « Block with override » — l’utilisateur peut toujours cliquer « Continuer ». L’override est logué dans Purview Activity Explorer avec l’identité de l’utilisateur, le timestamp et le SIT détecté.

| Type de blocage | Bypass possible ? |
|---|---|
| MDE Network Protection (Mode A) | ❌ Aucun bypass — blocage dur |
| DLP inline Edge — texte envoyé | ✅ Block with override (by design Microsoft) |
| DLP inline Edge — fichier upload | ❌ Blocage direct possible |
| Endpoint DLP — paste depuis doc étiqueté | ✅ Block with override |


C’est précisément pourquoi le mode hybride est la seule approche défendable : le Mode A fournit le blocage dur là où c’est nécessaire (apps sans besoin métier), le Mode B fournit la friction + traçabilité là où c’est utile (apps autorisées). Les deux couches DLP (inline Edge + Endpoint) se déclenchent en cascade : d’abord l’Endpoint DLP sur le paste, puis l’inline Edge sur l’envoi. Chaque événement est logué indépendamment dans l’Activity Explorer.


---

[← Partie 2 — Prérequis Microsoft Purview](02-purview-prerequis.md) | [Partie 4 — Surveillance Shadow AI dans Purview (DSPM for AI) →](04-dspm.md)
