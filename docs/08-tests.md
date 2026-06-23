---
title: "Partie 8 — Tests et validation | Gouvernance Shadow AI Microsoft 365"
description: "Tests de validation T1 à T4 : blocage navigateur, blocage API, auto-étiquetage automatique, gouvernance Power Platform."
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

# Partie 8 — Tests et validation


## 8.1 Test T1 Gouvernance Shadow AI navigateur et API





| Étape | Description |
|---|---|
| 1 | T1a — Blocage Edge URLBlocklist (chatgpt.com) Prérequis : appareil onboardé dans le groupe GRP-Test-Shadow-AI (profil Intune Edge - Restrictions Shadow AI déployé). Depuis la VM de test, ouvrez Microsoft Edge et naviguez vers https://chatgpt.com. Résultat attendu : page « Cette page est bloquée — Votre organization ne vous permet pas d’afficher ce site ». Ce blocage est géré par l’Edge URLBlocklist (politique Intune, section 6.2). Note : ce blocage ne s’applique qu’à Microsoft Edge. Firefox et Chrome ne sont pas couverts par cette politique pour les URL web grand public — cette couverture est assurée par MDCA (sections 1.5 et 1.3.4) une fois les applis marquées « non-sanctionnées » et propagées (24-48h). |
| 2 | T1b — Blocage MDE Network Protection (api.openai.com dans Edge) et comportement Firefox/Chrome Depuis la VM de test, ouvrez Microsoft Edge et naviguez vers https://api.openai.com. Résultat attendu : page « Ce site web est bloqué par votre organisation — Hébergé par api.openai.com » avec la mention « Sécurité Microsoft » en bas de page. Ce blocage est géré par les indicateurs URL/Domaines MDE (section 6.3), pas par l’Edge URLBlocklist. La différence visuelle entre les deux messages de blocage est importante : le message Edge URLBlocklist est générique, le message MDE affiche « Sécurité Microsoft ». Comportement Firefox et Chrome (validé en tenant lab, mai 2026) : le blocage ne génère pas une page d’administration propre mais une erreur SSL — « connexion non sécurisée avec le serveur ». Ce comportement est normal : MDE intercepte la négociation TLS. Le site est bien bloqué — le message est simplement moins explicite. Prévoyez une communication spécifique à destination des utilisateurs Firefox/Chrome (section 9.5). Trace d’alerte Defender : chaque tentative génère une alerte « Connection to a custom network indicator » dans security.microsoft.com → Incidents et réponse → Alertes. L’alerte contient le processus (ex. firefox.exe), les domaines bloqués, l’appareil et l’horodatage — traçabilité nLPD art. 24. Source : « Indicateur de domaine personnalisé » — Technologie : « Détecteur réseau ». |
| 3 | T1c — Blocage MDE Network Protection cross-navigateur (api.openai.com dans Firefox) Depuis la VM de test, ouvrez Firefox et naviguez vers https://api.openai.com. Résultat attendu : Firefox affiche une erreur de connexion sécurisée ET une notification Windows apparaît : « Sécurité Windows — Ce contenu est bloqué par votre administrateur informatique. Pour votre protection, votre administrateur informatique ne vous autorise pas à accéder au contenu de api.openai.com. » Ce test confirme que MDE Network Protection opère au niveau OS, indépendamment du navigateur utilisé. Les indicateurs URL/Domaines bloquent toutes les applications (navigateurs, scripts, appels API programmatiques) sur l’appareil géré. |


## 8.2 Test T2 Détection DLP sur partage de données sensibles



| Étape | Description |
|---|---|
| 1 | Procédure de test Depuis Outlook, envoyez un email à une adresse externe (ex. adresse personnelle) avec le texte suivant dans le corps : 756.1234.5678.97 (numéro AVS fictif). L’email est envoyé (la politique DLP est en mode TestWithNotifyUser — voir note ci-dessous). Attendez 15 à 30 minutes, puis vérifiez dans Microsoft Purview → Solutions → Protection des données → Explorateur d’activités. |
| 2 | Résultat attendu — Explorateur d’activités Activité « DLP rule matched » visible pour l’email test avec : - Type d’infos sensibles : Swiss Social Security Number AHV - Stratégie : DLP - Blocage partage externe données sensibles (nLPD) - Règle : DLP - Blocage partage externe données sensibles (nLPD) - Mode : TestWithNotifyUser - Actions : NotifyUser + GenerateAlert - Destinataire : adresse externe (IncludeExternalUsers) En parallèle, plusieurs activités « Label applied » par « Auto-labeling policy » (Auto-Label-RH-S...) confirment que l’étiquette 4 — RH-Confidentiel a été appliquée automatiquement. |


> **⚠️ Mode TestWithNotifyUser, passage en mode Blocage La politique DLP est actuellement en mode « TestWithNotifyUser » : elle détecte et notifie mais ne bloque pas. Pour activer le blocage réel, modifiez la politique dans Purview → Protection des données → Stratégies → DLP - Blocage partage externe données sensibles (nLPD) → Mode : Activer la stratégie DLP. Cette modification doit être précédée d’une communication aux utilisateurs (art. 19 nLPD).**




## 8.3 Test T3 Auto-étiquetage automatique


| Étape | Description |
|---|---|
| 1 | Validation via Explorateur d’activités Le même email test (section 8.2) déclenche simultanément la validation T3 : les activités « Label applied » par Auto-labeling policy (Auto-Label-RH-S...) confirment que l’étiquette 4 — RH-Confidentiel a été appliquée automatiquement sur détection du numéro AVS. L’email reçu par le destinataire affiche l’en-tête « RH — CONFIDENTIEL — Usage interne uniquement » et le pied de page « Document protégé nLPD — Ne pas diffuser », et est marqué « Chiffré — Autorisation accordée par : admin@votredomaine.ch ». La protection IRM liée à l’étiquette est active même si le DLP ne bloque pas : le destinataire externe doit s’authentifier pour accéder au contenu. |


## 8.4 Test T4 Gouvernance Power Platform (DLP-ConnecteursProd)



| Étape | Description |
|---|---|
| 1 | Vérification de la politique active Accédez à admin.powerplatform.microsoft.com → Sécurité → Données et confidentialité → Stratégie de données. La politique DLP-ConnecteursProd doit apparaître avec Étendue : Organisation (client) et Appliqué à : Tous les environnements. |
| 2 | Test de blocage connecteur OpenAI Dans Power Automate (make.powerautomate.com), créez un nouveau flux instantané (Test-DLP-OpenAI). Ajoutez une action avec le connecteur « OpenAI GPT (Independent Publisher) » ou « Azure OpenAI ». Tentez de sauvegarder le flux avec une connexion valide à un connecteur bloqué : la sauvegarde échoue avec un message de violation DLP indiquant que le connecteur n’est pas autorisé dans cet environnement. Note lab : la validation complète de ce test nécessite une clé API valide pour établir la connexion. Sans connexion valide, Power Automate refuse l’enregistrement pour cause de « connexion manquante » avant même d’atteindre la vérification DLP. ⚠️ Validation fonctionnelle non réalisée en lab (clé API requise). Le comportement de blocage DLP est documenté par la présence de la politique active avec 17 connecteurs bloqués, mais le test de blocage effectif n’a pas pu être exécuté faute de connexion valide. À valider en production lors de la mise en œuvre. |


---

[← Partie 7 — Gouvernance Power Platform et Copilot Studio](07-power-platform.md) | [Partie 9 — Gestion opérationnelle et maintenance →](09-maintenance.md)
