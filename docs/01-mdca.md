---
title: "Partie 1 — Microsoft Defender for Cloud Apps (MDCA) | Gouvernance Shadow AI Microsoft 365"
description: "Configuration complète Microsoft Defender for Cloud Apps (MDCA) : Cloud Discovery, onboarding Defender for Endpoint, sanction/non-sanction des apps IA, stratégies AutoBlock, Conditional Access BYOD."
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

# Partie 1 — Microsoft Defender for Cloud Apps (MDCA)


## 1.1 Comprendre MDCA et son rôle dans la réduction des usages Shadow AI
Microsoft Defender for Cloud Apps (MDCA) est votre principal outil de détection et de blocage des applications IA non autorisées. En mode Cloud Discovery via Microsoft Defender for Endpoint, il analyse les métadonnées réseau (domaines contactés, volumes, horodatages) de tous les appareils onboardés. Note : ce n’est pas un proxy inline, il ne lit pas le contenu des flux HTTPS. Le proxy inline (CAAC) est une configuration avancée séparée, non couverte dans ce guide de base.


> **Ce que MDCA fait concrètement**
>
> 1. Il découvre automatiquement la majorité des applications cloud visibles réseau utilisées par vos collaborateurs (ChatGPT, Claude, Gemini, Perplexity, etc.). Limite : Cloud Discovery reste borné aux flux DNS/HTTP(S) observables — il ne couvre pas les WebSockets ni les applications encapsulées (apps Teams, Electron).
>
> 2. Il les évalue sur 90+ critères de sécurité (certification ISO, localisation des données, chiffrement, etc.).
>
> 3. Il vous permet de sanctionner (autoriser) ou non-sanctionner (bloquer) chaque application.
>
> 4. Il crée des alertes automatiques quand un utilisateur accède à une IA non autorisée.
>
> 5. Il peut bloquer le téléchargement de fichiers sensibles depuis ces applications.



## 1.2 Première connexion au portail MDCA




| Étape | Description |
|---|---|
| 1 | Accéder au portail Microsoft Defender Ouvrez votre navigateur (Edge ou Chrome). Allez sur : https://security.microsoft.com Connectez-vous avec votre compte administrateur. Dans le menu de gauche, développez Applications cloud, c’est la section MDCA. |
| 2 | Vérifier que MDCA est activé Dans le menu de gauche : Applications cloud → Découverte cloud. Si vous voyez que la page Découverte cloud s’affiche avec les boutons, créer un nouveau rapport et Configurer le chargement automatique, MDCA est actif. Continuez à la section 1.3. Note : la section Sécurité cloud du même menu correspond à Defender for Cloud (CSPM Azure/AWS/GCP). C’est un produit différent de MDCA, ne pas confondre les deux. |


## 1.3 Configurer Cloud Discovery, Inventaire du Shadow AI
Cloud Discovery est la fonctionnalité qui analyse les logs réseau pour découvrir automatiquement toutes les applications cloud utilisées dans votre organisation. ⚠️ Choisissez UNE seule option selon votre environnement : si vos appareils sont joints à un Active Directory on-premise → section 1.3.1 (Option A). Si votre tenant est pur Microsoft 365 (Entra ID uniquement, sans AD on-premise) → section 1.3.2 (Option B). Ne faites pas les deux.


### 1.3.1 Option A : Environnement hybride (avec Active Directory on-premise)
Dans un environnement hybride, vos collaborateurs utilisent des appareils joints au domaine Active Directory. La méthode de découverte recommandée est l'intégration avec Microsoft Defender for Endpoint.




| Étape | Description |
|---|---|
| 1 | Activer l'intégration Defender for Endpoint Dans le portail Defender : Paramètres → Cloud Apps. Cliquez sur Microsoft Defender for Endpoint dans le menu Paramètres. Cochez la case Appliquer l’accès aux applications. Passez la Gravité de l’alerte de Information à Moyen. Cliquez Enregistrer. |
| 2 | Vérifier que les appareils remontent des données Attendez 24-48 heures (les données de découverte ne sont pas instantanées). Allez dans Applications cloud → Découverte cloud. Vous devriez voir apparaître les applications découvertes avec le nombre d'utilisateurs et de transactions. |


> **ℹ️  INFO Pour que Cloud Discovery fonctionne en environnement hybride, les appareils doivent être inscrits dans Microsoft Defender for Endpoint (onboardés). Si ce n'est pas encore le cas, la section 1.3.3 explique comment onboarder les appareils.**



### 1.3.2 Option B - Tenant pur Microsoft 365 (cloud-only)
Dans un environnement cloud-only, tous vos appareils sont gérés via Intune et joints à Entra ID. L'intégration est plus simple.




| Étape | Description |
|---|---|
| 1 | Activer Cloud Discovery en environnement cloud-only (via Defender for Endpoint) Dans le portail Defender : Paramètres → Cloud Apps → Collecte de données. Activez l’intégration en suivant la même procédure que la section 1.3.1 : Paramètres → Applications cloud → Découverte cloud → Microsoft Defender pour point de terminaison. Cliquez Enregistrer. L'intégration est automatique, aucune configuration supplémentaire n'est nécessaire. |


### 1.3.3 Onboarding des appareils dans Defender for Endpoint
Si vos appareils ne sont pas encore onboardés dans Defender for Endpoint, voici comment le faire en masse via Intune :







| Étape | Description |
|---|---|
| 1 | Créer le profil d'onboarding dans Intune Allez sur : https://intune.microsoft.com Sécurité des points de terminaison → Microsoft Defender for Endpoint. Cliquez sur Connecter pour lier Intune à Defender. Activez l'option Connexion à Microsoft Defender pour point de terminaison. Cliquez sur Enregistrer. |
| 2 | Créer la stratégie de conformité Defender Chemin : Intune (intune.microsoft.com) → Sécurité des points de terminaison → Microsoft Defender for Endpoint. Créez une stratégie de conformité : Plateforme = Windows 10 et versions ultérieures. Activez l’option Connexion à Microsoft Defender pour point de terminaison. Assignez au groupe Tous les appareils. Délai d’application : 4 à 8h. Vérification : Portail Defender → Ressources → Inventaire des appareils métrique clé : « Non intégré = 0 ». Erreur fréquente : l’onboarding passe par Sécurité des points de terminaison → Microsoft Defender for Endpoint, pas par Antivirus. |


### 1.3.4 Vérification critique : Toggle MDCA dans MDE Fonctionnalités avancées



| Étape | Description |
|---|---|
| ! | Étape obligatoire : sans ce toggle, toute la chaîne MDCA→MDE est silencieusement inactive Constat terrain (mai 2026) : ce toggle est Désactivé par défaut dans les nouveaux tenants. Sans lui, MDCA ne reçoit pas les signaux MDE et ne peut pas pousser les règles de blocage vers les appareils. Le statut « Non sanctionné » appliqué en section 1.5 reste purement informatif, aucun blocage réel sur Firefox ou Chrome. Chemin : security.microsoft.com → Paramètres → Points de terminaison → Fonctionnalités avancées (scroll vers le bas). Activez les deux toggles suivants, puis cliquez Enregistrer les préférences : 1 Microsoft Defender for Cloud Apps → Activé (permet à MDE d’envoyer ses signaux à MDCA et de recevoir les règles de blocage des apps non-sanctionnées). 2 Filtrage du contenu web → Activé (prérequis pour toute stratégie WCF future). Note : la description indique « licence E5 requise », c’est inexact. Cette intégration fonctionne avec Microsoft 365 Business Premium + Microsoft Defender and Purview Suites, confirmé en tenant lab (mai 2026). |


### 1.3.5 Étape obligatoire : Activer Network Protection en mode Block (Intune)



| Étape | Description |
|---|---|
| ! | Sans cette étape, MDCA et les indicateurs MDE détectent et notifient mais ne bloquent jamais Constat terrain (mai 2026) : Network Protection est désactivé par défaut dans Intune. Sans lui, les apps non-sanctionnées MDCA et les indicateurs MDE URL/Domaines génèrent une notification Windows (« Ce contenu est bloqué par votre administrateur ») mais la page reste accessible. Ce comportement peut créer une fausse impression de sécurité. Chemin : intune.microsoft.com → Appareils → Gestion des appareils → Configuration → + Créer → Windows 10 et versions ultérieures → Catalogue des paramètres. Nom de la stratégie : MDE-NetworkProtection-Block-ShadowAI Recherchez « Network Protection » dans le Catalogue des paramètres et configurez : - Enable Network Protection → Enabled (block mode) - Allow Network Protection Down Level → Network protection will be enabled downlevel Affectations : Tous les appareils. Délai de propagation : 15 à 30 minutes. Vérification : accéder à chatgpt.com dans Firefox, la page doit être bloquée (pas seulement notifiée). Note : « Protection du réseau » n’est pas disponible comme profil direct dans Réduction de la surface d’attaque, passez obligatoirement par le Catalogue des paramètres. |


## 1.4 Identifier et analyser les applications IA dans Cloud Discovery





| Étape | Description |
|---|---|
| 1 | Accéder au tableau de bord Cloud Discovery Dans le portail Defender : Applications cloud → Découverte cloud. Vous voyez la liste de toutes les applications cloud détectées. |
| 2 | Filtrer les applications d'IA générative Cliquez sur Filtres en haut de la liste. Dans le filtre Catégorie, sélectionnez IA générative. Vous voyez maintenant uniquement les applications IA utilisées dans votre organisation. Notez le nombre d'utilisateurs et le volume de données transférées pour chaque application. |
| 3 | Consulter le score de risque d'une application Cliquez sur le nom d'une application (ex: ChatGPT). Vous voyez le score de risque sur 10 et le détail des critères : localisation des données, certification ISO 27001, politique de rétention, etc. Un score inférieur à 7 indique une application à risque élevé (seuil cohérent avec la politique de détection configurée en section 1.7.1). En dessous de 5, le risque est critique : bloquez immédiatement. |


> **Applications IA à risque élevé - Liste de référence**
>
> Les applications suivantes sont fréquemment détectées et présentent un risque élevé pour les données d'entreprise :
>
> - ChatGPT (OpenAI) - Entraînement possible sur vos données sans opt-out explicite
>
> - Gemini (Google) - Données stockées aux États-Unis
>
> - Claude.ai (Anthropic) - Version grand public, sans DPA entreprise
>
> - Perplexity AI - Historique des conversations conservé
>
> - Character.ai - Pas de certifications entreprise
>
> - Midjourney - Discord comme interface, logs non contrôlables
>
> Note : Les versions Enterprise de ces services (ChatGPT Enterprise, Claude for Work) ont des garanties contractuelles différentes.




## 1.5 Sanctionner et non-sanctionner les applications IA
La notion de sanction dans MDCA signifie 'application approuvée par l'entreprise'. Une application non-sanctionnée peut être bloquée automatiquement.




| Étape | Description |
|---|---|
| 1 | Marquer les outils IA approuvés comme Sanctionnés Dans Cloud Discovery, recherchez Microsoft Copilot. Cliquez sur les trois points (...) à droite de l'application. Sélectionnez Sanctionner. Faites de même pour les autres outils IA officiellement approuvés par votre organisation. |
| 2 | Marquer les outils IA non approuvés comme Non sanctionnés Pour chaque application IA non autorisée (ChatGPT grand public, Gemini, etc.) : Cliquez sur Annulation de l’approbation dans la fiche de l’application. Confirmez en cliquant Enregistrer dans le dialogue Marquer comme non approuvé ?. Le statut passe à Application non sanctionnée (icône rouge ⦸). MDCA envoie automatiquement la règle de blocage à Defender for Endpoint si l’intégration est active (section 1.3.1). |


> **⚠️  ATTENTION Le blocage effectif via 'Non sanctionner' nécessite que l'intégration Defender for Endpoint soit active (section 1.3). Sans cette intégration, le statut 'Non sanctionné' est uniquement informatif — il ne bloque pas l'accès.**




## 1.6 Automatiser le marquage des nouvelles applications IA



| Étape | Description |
|---|---|
| 1 | Créer la stratégie de découverte automatique MDCA-ACT-Shadow-AI-AutoBlock Le catalogue MDCA est mis à jour en continu par Microsoft. Sans automatisation, chaque nouvelle application IA découverte reste en statut neutre et n’est pas bloquée via MDE. Chemin : MDCA → Applications cloud → Stratégies → + Créer une stratégie → Stratégie de découverte d’application. Configuration réalisée (12 mai 2026) : Nom de la stratégie : MDCA-ACT-Shadow-AI-AutoBlock - Filtre 1 : Catégorie est égal à IA générative - Filtre 2 : Balise de l’application est égal à Aucune valeur (apps sans tag) - Action : Baliser l’application comme non approuvée Important : cette condition « Aucune valeur » garantit que les applications déjà marquées « Application approuvée » (Microsoft Copilot, M365 Copilot, Copilot Studio, Microsoft Security Copilot, Microsoft 365 Copilot Chat) ne seront pas affectées. ⚠️ Limite importante : cette stratégie s’applique uniquement aux applications nouvellement découvertes (premier trafic détecté après la création de la stratégie). Elle ne s’applique pas rétroactivement aux applications déjà présentes dans Cloud Discovery sans tag. Action manuelle requise lors de la mise en place initiale : MDCA → Applications cloud → Cloud Discovery → Applications découvertes → filtrer par Catégorie = IA générative + Balise = Aucune valeur → taguer manuellement chaque application comme non-sanctionnée. |


## 1.7 Créer des politiques MDCA pour le Shadow AI
Les politiques MDCA permettent de recevoir des alertes et d'automatiser des actions lorsqu'un collaborateur utilise une application IA non autorisée.


### 1.7.1 Politique de détection d'applications IA non autorisées





| Étape | Description |
|---|---|
| 1 | Créer la politique Dans le portail Defender : Applications cloud → Stratégies → Gestion des stratégies. Cliquez sur + Créer une stratégie (flèche déroulante). Types disponibles : Stratégie d’activité, Stratégie de fichier, Stratégie de découverte d’application, Stratégie d’accès, Stratégie de session, Stratégie d’application OAuth. Sélectionnez Stratégie de détection d’application. |
| 2 | Configurer les paramètres de la politique Nom : MDCA-DET-Shadow-AI-Generative Gravité : Moyen Catégorie : Cloud Discovery Filtre : Catégorie est égal à IA générative (cochez aussi AI - MCP Server et AI - Model Provider pour couvrir l’ensemble du périmètre IA) Appliquer à : Tous les rapports continus Actions de gouvernance : aucune cochée (détection seule en phase initiale) |
| 3 | Configurer les alertes Dans la section Alertes : La case Créer une alerte pour chaque événement est cochée par défaut. Cochez éventuellement Envoyer une alerte par courrier électronique. Laissez la limite quotidienne à 5. Cliquez Créer. |


### 1.7.2 Stratégie de surveillance des transferts de données vers des IA

> **⛔  SECTION OPTIONNELLE - Non déployée dans ce guide. À lire avant de configurer quoi que ce soit. La stratégie de session décrite ci-dessous ne fonctionne que si le proxy CAAC (Conditional Access App Control) est déployé, ce que ce guide ne fait pas. Sans ce proxy, elle est silencieusement inactive et ne bloque rien. Elle n'apporte aucune couverture supplémentaire face aux vecteurs Shadow AI réels (le copier-coller, le mobile et les API directes restent hors de sa portée). Si vous suivez ce guide pour la première fois, ne configurez pas cette section : passez directement à la section 1.8.1 (politique BYOD). Le contenu ci-dessous est conservé à titre de référence pour les organisations qui exploitent déjà un proxy CAAC.**


⚠️ PRÉREQUIS OBLIGATOIRE - Conditional Access App Control (CAAC)
Une politique de session MDCA n’intercepte le trafic QUE si l’application est préalablement routée à travers le proxy MDCA. Cela exige une politique d’Accès Conditionnel Entra ID avec le contrôle de session « Utiliser le contrôle d’application par accès conditionnel». Sans cette configuration, la session policy n’a aucun effet — elle ne voit aucun trafic.
Procédure dans Entra ID (https://entra.microsoft.com) → Accès conditionnel → Stratégies → + Nouvelle stratégie : (1) Nom : CAAC-Shadow-AI-Session-Control. (2) Utilisateurs ou assistants → Tous les utilisateurs. (3) Ressources cibles → Toutes les ressources (anciennement « Toutes les applications cloud »). (4) Contrôles d’accès → Session → cochez Utiliser le contrôle d’application par accès conditionnel → sélectionnez Utiliser une stratégie personnalisée... (5) Activer une stratégie → Rapport uniquement. (6) Cliquez Créer.






| Étape | Description |
|---|---|
| 1 | Créer la politique de session Applications cloud → Stratégies → Gestion des stratégies. Cliquez sur + Créer une stratégie → Stratégie de session. Nom : Shadow AI - Surveillance transferts données vers IA externe Gravité : Moyen |
| 2 | Configurer les conditions Type d’activité : Chargement de fichier (Upload), pour surveiller les fichiers envoyés vers des services IA externes. Attention : “Téléchargement” (Download) désigne le sens inverse (depuis l’app vers le poste) et ne couvre pas ce cas d’usage. Application : sélectionnez les applications IA non sanctionnées. Filtre Utilisateur : Tous les utilisateurs (ou un groupe spécifique). Action : Bloquer (ou Surveiller selon votre niveau de maturité initial). |
| 3 | Activer la politique Vérifiez la configuration. Cliquez Créer. ⚠️ HORS PÉRIMÈTRE DE CE GUIDE — Configuration avancée non déployée. IMPORTANT : cette configuration n’est pas utilisée dans ce guide et est fournie à titre informatif uniquement. Elle ne remplace pas les contrôles actifs des sections précédentes. Elle n’est pas requise pour la stratégie décrite dans ce guide et n’apporte pas de couverture significative supplémentaire face aux vecteurs Shadow AI identifiés (LLM locaux, API directes, usages mobiles restent hors périmètre CAAC). Cette stratégie de session n’est opérationnelle que si le proxy CAAC est déployé (Paramètres → Applications cloud → Contrôle des applications à accès conditionnel). Sans ce proxy : la politique est silencieusement inactive et ne bloque rien. Pour les PME en début de déploiement : finalisez d’abord les sections 1.3 à 1.7.1 et la Partie 3 (DLP) qui couvrent 80% du risque Shadow AI. |


## 1.8 Accès Conditionnel — Périmètre et limites pour la gouvernance IA
⚠️ Note architecturale importante : Le Conditional Access (CA) d’Entra ID s’applique uniquement aux applications enregistrées dans le tenant Microsoft (applications SAML/OIDC managées). Il ne peut pas bloquer directement les URLs de sites publics comme chatgpt.com ou claude.ai, qui ne sont pas des applications Entra ID. Pour bloquer ces sites, utilisez les mécanismes décrits en section 1.5 (non-sanction via Defender for Endpoint) et section 6.2 (liste d’URL dans Edge/Intune). Cette section couvre néanmoins le cas des applications cloud managées que vous souhaitez soumettre à un contrôle de session via le proxy MDCA (Conditional Access App Control).




| Étape | Description |
|---|---|
| 1 | Accéder aux politiques d'accès conditionnel Allez sur : https://entra.microsoft.com Accès conditionnel → Stratégies (menu gauche d’Entra ID). Cliquez sur + Nouvelle stratégie. |
| 2 | Configurer la politique de blocage IA Aucune politique CA supplémentaire n’est requise pour bloquer les sites IA grand public (chatgpt.com, claude.ai, etc.) car le Conditional Access ne peut pas bloquer ces URLs non-Entra. Le blocage est assuré par MDCA + MDE (sections 1.5 et 6.2). → Passez directement à la section 1.8.1 pour configurer la politique BYOD. |


> **✅  CONSEIL Conseil pour les débutants : Commencez obligatoirement en mode Rapport uniquement. Attendez au minimum 1 semaine et consultez les Journaux de connexion (Entra ID → Supervision → Journaux de connexion) pour vérifier l’impact avant d’activer. Note 2026 : l’octroi d’application cliente approuvée est mis hors service, ne pas utiliser ce contrôle dans les nouvelles stratégies CA.**




## 1.8.1 Politique BYOD, Bloquer les appareils non gérés
Angle mort critique : si un collaborateur accède à Outlook Web, SharePoint ou Teams depuis un ordinateur personnel non onboardé dans Defender for Endpoint, l’intégralité des contrôles Endpoint DLP et MDCA devient inopérante. La solution est une politique d’Accès Conditionnel ciblant les appareils non conformes.




| Étape | Description |
|---|---|
| 1 | Accéder à la création de politique Allez sur https://entra.microsoft.com → Protection → Accès conditionnel → Stratégies → + Nouvelle stratégie. La page Nouveau affiche les sections : Affectations (Utilisateurs ou assistants, Ressources cibles, Réseau, Conditions) et Contrôles d’accès (Octroyer, Session). |
| 2 | Nommer la politique Dans le champ Nom, saisissez : BLOCK - Accès depuis appareils non conformes (BYOD) |
| 3 | Configurer les utilisateurs Cliquez sur Utilisateurs ou assistants (Préversion). Dropdown «À quoi cette stratégie s’applique-t-elle ?» : sélectionnez Utilisateurs et groupes. Onglet Inclure : sélectionnez Tous les utilisateurs. ⚠️ Un bandeau d’avertissement apparaît : «Ne bloquez pas votre accès !» — Microsoft recommande de tester d’abord sur un petit groupe d’utilisateurs. |
| 4 | Configurer les ressources cibles Cliquez sur Ressources cibles. Dropdown : sélectionnez Ressources (anciennement applications...). Onglet Inclure : sélectionnez Toutes les ressources (anciennement «Toutes les applications cloud»). |
| 5 | Configurer le filtre appareils Cliquez sur Conditions → Filtre pour les appareils. Basculez Configurer sur Oui. Sélectionnez Exclure les appareils filtrés de la stratégie. Ajoutez deux expressions : Ligne 1 : Propriété = IsCompliant / Opérateur = Est égal à / Valeur = True. Cliquez + Ajouter une expression. Ligne 2 : Et/Ou = Ou / Propriété = TrustType / Opérateur = Est égal à / Valeur = Jointure hybride Microsoft Entra. La syntaxe générée dans le champ «Syntaxe de la règle» affiche : device.isCompliant -eq True -or device.trustType -eq "ServerAD". Cliquez Terminé. Logique résultante : les appareils conformes Intune OU hybrid-joined sont exclus de la stratégie, tout autre appareil (BYOD) est bloqué. ⚠️ Cette configuration suppose que les règles de conformité Intune sont strictes : un appareil hybrid-joined mais non conforme (chiffrement désactivé, antivirus absent, etc.) reste exclu de la stratégie par la clause OR. Vérifiez vos politiques de conformité Intune (section 0) avant de vous appuyer sur cette règle. |
| 6 | Configurer le contrôle d’accès Cliquez sur Octroyer. Sélectionnez Bloquer l’accès. Cliquez Sélectionner. |
| 7 | Activer en mode test puis en production Section Activer une stratégie en bas de page : laissez sur Rapport uniquement pendant 1 semaine. ⚠️ Bandeau rouge : «Ne bloquez pas votre accès !» l’option Exclure l’utilisateur actuel (votre compte admin) est présélectionnée, conservez cette exclusion. Cliquez Créer. Après 1 semaine, consultez les journaux (Entra ID → Supervision → Journaux de connexion) et passez sur Activé. |
| ⚠️ | ERREUR FRÉQUENTE — Ne pas utiliser «Inclure les appareils filtrés» avec la logique ET entre IsCompliant et TrustType : cela bloquerait les appareils conformes non-hybrides et les hybrides non-conformes. La logique correcte est «Exclure les appareils filtrés» avec un OU entre les deux conditions. |


## 1.9 Tableau de bord MDCA - Suivi du Shadow AI






| Étape | Description |
|---|---|
| 1 | Accéder à Cloud Discovery Portail Defender (security.microsoft.com) → Applications cloud → Découverte cloud. Si aucune intégration n’est encore configurée, la page affiche un écran d’accueil avec trois actions : Créer un nouveau rapport, Configurer le chargement automatique, Afficher l’exemple de rapport. Une fois l’intégration MDE active (section 1.3), le tableau de bord se peuple automatiquement avec les métriques : nombre d’applications découvertes, utilisateurs, IPs, volume de trafic, niveaux de risque et top utilisateurs. |
| 2 | Créer un rapport d’instantané (chargement manuel) Cliquez sur Créer un nouveau rapport pour analyser des logs existants. Le wizard se déroule en 4 étapes : Vue d’ensemble → Détails du rapport (nom, description, source) → Charger les journaux de trafic → Terminer. Dans le champ Source, sélectionnez l’appliance correspondant à votre pare-feu ou proxy : Blue Coat, Barracuda, Check Point, McAfee, Palo Alto, Sophos, Zscaler, etc. Pour les appareils non listés, utilisez Generic W3C log, Generic CEF log ou Generic LEEF log. Note : Microsoft Defender for Endpoint n’apparaît pas dans cette liste, son intégration est automatique via la section 1.3 (chargement continu). |
| 3 | Consulter les alertes MDCA Chemin : Enquête et réponse → Incidents et alertes → Alertes. Filtrez par type « Connection to a custom network indicator » pour identifier les tentatives d’accès bloquées par les indicateurs MDE chaque alerte contient le processus, les domaines tentés, l’appareil et l’utilisateur (traçabilité nLPD art. 24). ⚠️ Note Cas A : dans un tenant vierge, cette liste sera vide jusqu'à ce que les politiques DLP et MDCA génèrent leurs premières alertes (compter 24-48h après activation). Les alertes visibles dans votre lab proviennent des politiques créées dans le guide Configuration_Microsoft_Purview_2026.docx (DLP-Protection-nLPD-demo, DLP-Bloquer-Copilot-Confidentiel) et non de MDCA (si vous avez configuré Purview). Le tableau affiche : Nom de l’alerte, Balises, Gravité, État de l’examen, État, Catégorie, Source de détection, Ressources affectées. Utilisez Ajouter un filtre pour filtrer par : Gravité, Catégories (ex. Exfiltration), Sources de service/détection, Étiquette de confidentialité. Pour chaque alerte : cliquez dessus, examinez l’activité, et choisissez de résoudre ou d’escalader. Note : depuis juin 2025, MDCA utilise un modèle dynamique de détection des menaces, les stratégies de détection ont été migrées automatiquement. |



| ✓ | Action | Statut |
|---|---|---|
| ☐ | MDCA activé et tableau de bord visible dans le portail Defender | À faire |
| ☐ | Intégration Defender for Endpoint ou Intune configurée pour Cloud Discovery | À faire |
| ☐ | Applications IA d’entreprise marquées Sanctionnées via le bouton Sanction (Copilot Studio, M365 Copilot Chat, Microsoft Copilot). Remarque : GitHub Copilot n’est pas listé ici car il est invisible à Cloud Discovery (extension IDE via api.github.com) et ne peut donc pas être sanctionné dans MDCA ; il se traite côté MDE Network Protection et politique d’extensions IDE (voir Introduction). | À faire |
| ☐ | Applications IA grand public marquées Non sanctionnées (statut : Application non sanctionnée) | À faire |
| ☐ | Stratégie MDCA-DET-Shadow-AI-Generative créée et active | À faire |
| ☐ | Stratégie MDCA-ACT-Shadow-AI-AutoBlock créée et active (section 1.6) | À faire |
| ☐ | Politique de session pour les transferts de fichiers créée | À faire |
| ☐ | Alertes email configurées pour le responsable sécurité | À faire |
| ☐ | Premier rapport Cloud Discovery consulté et archivé | À faire |


---

[← Partie 0 — Avant de commencer](00-prerequis.md) | [Partie 2 — Prérequis Microsoft Purview →](02-purview-prerequis.md)
