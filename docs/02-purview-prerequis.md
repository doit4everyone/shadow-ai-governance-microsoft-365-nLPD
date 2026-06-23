---
title: "Partie 2 — Prérequis Microsoft Purview | Gouvernance Shadow AI Microsoft 365"
description: "Trois scénarios de déploiement Purview (Cas A nouveau tenant, Cas B guide suivi, Cas C tenant existant) avec prérequis, inventaire et ordre de déploiement recommandé."
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

# Partie 2 — Prérequis Microsoft Purview

Avant de déployer la couche MDCA Shadow AI, vous devez déterminer dans quelle situation se trouve votre tenant Purview. Trois cas sont possibles. Lisez le tableau ci-dessous et suivez uniquement la section correspondant à votre situation.

| Votre situation | Description | Section à suivre |
|---|---|---|
| Cas A - Nouveau tenant | Purview n’a jamais été configuré. Aucune étiquette, aucune politique DLP, aucun audit. | Section 2.1 |
| Cas B - Guide Purview suivi | Votre tenant a été configuré en suivant le guide Configuration_Microsoft_Purview_2026.docx. | Section 2.2 |
| Cas C - Purview partiellement déployé | Votre tenant est une PME existante avec Purview en place, mais configuré de façon partielle ou différente du guide de référence. | Section 2.3 |



## 2.1 Cas A : Nouveau tenant (Purview non configuré)
Si Purview n’a jamais été configuré dans votre organisation, commencez impérativement par le guide de référence ci-dessous avant de revenir à ce document.

| 📄  DOCUMENT DE RÉFÉRENCE - Cas A Ce guide couvre la configuration complète depuis un tenant vierge : groupes de sécurité mail-enabled, 7 étiquettes de sensibilité avec chiffrement RMS, SITs personnalisés (AVS, IBAN, données médicales), politiques DLP, auto-labelling, Endpoint DLP, DSPM for AI, Insider Risk Management et gouvernance SharePoint. Téléchargez le guide ici : https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/ Une fois ce guide appliqué intégralement, revenez ici et suivez la section 2.2 (Cas B). |
|---|




## 2.2 Cas B : Guide Purview de référence déjà appliqué
Vérifiez que les éléments suivants sont en place. Ces prérequis sont nécessaires pour que les règles DLP Shadow AI (Partie 3) et les politiques DSPM for AI (Partie 4) fonctionnent correctement.

| ✔ | Prérequis | Réf. guide Purview |
|---|---|---|
| ☐ | Audit unifié activé, délai 48-72h avant pleine opérationnalité | Configuration_Microsoft_Purview_2026.docx — Partie 0, section 1.5. Vérification : Purview → Audit → Rechercher. Si le formulaire de recherche est actif, l'audit est activé. Si l'audit est désactivé, un bouton «Démarrer l'enregistrement» apparaît à la place. |
| ☐ | Étiquettes publiées (minimum Interne, Confidentiel, Secret) avec chiffrement sur les deux niveaux supérieurs | Configuration_Microsoft_Purview_2026.docx Partie 2. Vérification : Protection des données → Étiquettes de confidentialité <br>7 étiquettes doivent être présentes. Note 2026 : un bandeau «Migrez vers le modèle d'étiquette moderne» peut apparaître, ne pas cliquer en production. |
| ☐ | Politique DLP de base active couvrant SharePoint, OneDrive et Exchange | Configuration_Microsoft_Purview_2026.docx Partie 4. Vérification : Protection contre la perte de données → Stratégies.<br>Au moins une politique active (statut Synchronisation terminée). |
| ☐ | Politique d’auto-labelling configurée (AVS + IBAN - en simulation minimum) | Configuration_Microsoft_Purview_2026.docx Partie 3. Vérification : Stratégies → Stratégies d'étiquetage automatique<br>2 stratégies actives (Auto-Label-RH-Sensible et Auto-Label-Donnees-Sensibles-IBAN). |
| ☐ | Endpoint DLP avec groupe «Sites web d’IA générative» activé (action Bloquer ou Auditer) | Configuration_Microsoft_Purview_2026.docx Partie 9. Vérification : Protection contre la perte de données → Stratégies → DLP-Protection-Endpoint → règle → section Domaine de service → groupe «Sites web d'IA générative» configuré en Bloquer. |
| ☐ | Appareils onboardés dans Microsoft Defender for Endpoint | Section 1.3 de ce guide. Vérification : security.microsoft.com → Ressources → Inventaire des appareils — métrique clé : «Non intégré = 0». |




## 2.3 Cas C : Tenant existant avec Purview partiellement déployé
C’est le cas le plus fréquent en PME : Purview a été activé lors de la souscription à Business Premium, mais seule une partie des fonctionnalités a été configurée. Avant de déployer la couche Shadow AI, vous devez faire un inventaire de l’existant et combler les manques critiques.

| ⚠️  ATTENTION : Ne jamais modifier à l’aveugle Toute modification d’une étiquette existante ou d’une politique DLP active impacte immédiatement tous les utilisateurs et tous les fichiers déjà étiquetés. Lisez entièrement cette section avant de toucher quoi que ce soit. Travaillez toujours en mode Simulation pour les nouvelles politiques DLP avant de les activer. |
|---|



### 2.3.1 Inventaire de l’existant : à faire en premier
Étape préalable obligatoire avant toute configuration. Notez les résultats dans un tableau ou un document texte, vous en aurez besoin pour la suite.


| Ce qu’il faut vérifier | Où et comment |
|---|---|
| Étiquettes de sensibilité existantes | Purview → Protection des informations → Étiquettes. Notez : noms, niveaux, chiffrement activé ou non, groupes assignés. |
| Politiques DLP actives | Purview → Protection contre la perte de données → Politiques. Notez : emplacements couverts, actions (bloquer / avertir / auditer), mode (Test ou Actif). |
| Politiques d’auto-labelling | Purview → Protection des informations → Politiques d’étiquetage automatique. Notez : SITs détectés, étiquettes appliquées, emplacements. |
| Audit unifié | PowerShell : Get-AdminAuditLogConfig \| Select-Object UnifiedAuditLogIngestionEnabled. Résultat attendu : UnifiedAuditLogIngestionEnabled = True.<br>Si False : exécutez Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true et attendez 48-72h. |
| Politiques MDCA existantes | Portail Defender (security.microsoft.com) → Applications cloud → Stratégies → Gestion des stratégies. Vérifiez si des politiques de découverte ou de session sont déjà en place. Note 2026 : depuis juin 2025, le modèle de détection des menaces est dynamique, les anciennes stratégies de détection ont été migrées automatiquement. |
| Appareils onboardés MDE | Portail Defender → Ressources → Inventaire des appareils. Vérifiez que vos postes Windows apparaissent dans la liste. Métrique clé : «Non intégré = 0» indique que tous les appareils sont onboardés. |




### 2.3.2 Prérequis obligatoires Purview partiel
Après l’inventaire, vérifiez que chaque élément ci-dessous est présent. Pour chaque prérequis manquant, la colonne «Action» indique exactement ce que vous devez faire avant de continuer.


| Prérequis | Situation acceptable | Action si manquant |
|---|---|---|
| Audit unifié activé | UnifiedAuditLogIngestionEnabled = True | PowerShell : Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true. Attendez 48-72h avant de continuer. |
| Étiquettes de sensibilité - Minimum 3 niveaux | Au minimum : un niveau «Interne», un «Confidentiel» et un «Secret» (ou équivalent). Le chiffrement RMS doit être activé sur Confidentiel et Secret. | Si les étiquettes sont absentes ou sans chiffrement : suivre la Partie 2 du guide Configuration_Microsoft_Purview_2026.docx Si des étiquettes existent avec d’autres noms : les mapper avec les niveaux du guide avant de continuer. |
| Groupes de sécurité mail-enabled pour le chiffrement RMS | Les groupes assignés aux étiquettes chiffrées sont de type MailUniversalSecurityGroup (vérifiable via PowerShell : Get-DistributionGroup \| Select-Object Name, DisplayName, RecipientTypeDetails). Résultat attendu : 3 groupes avec RecipientTypeDetails = MailUniversalSecurityGroup (GRP-Direction-Purview, GRP-RH-Purview, GRP-Finances-Purview). Les groupes de sécurité simples Entra ID affichent GroupMailbox ou SecurityGroup. Ils ne fonctionnent pas avec le chiffrement RMS. | Les groupes de sécurité simples (Entra ID) ne fonctionnent pas avec le chiffrement RMS. Créer des groupes mail-enabled dans l’AD local (hybride) ou via Exchange Online (cloud-only). Voir Partie 2.2 du guide Configuration_Microsoft_Purview_2026.docx. |
| Politique DLP couvrant l’externe | Au moins une politique DLP active couvrant Exchange + SharePoint + OneDrive avec une règle de blocage du partage externe sur les données sensibles. | Créer une politique DLP minimale en suivant la Partie 3 de ce guide (section 3.2). La passer en mode Test pendant 48h avant de l’activer. |
| SITs suisses détectés (AVS, IBAN) | Votre politique d’auto-labelling ou DLP détecte le Numéro AVS suisse et l’IBAN suisse (SITs natifs Microsoft). | Ajouter ces SITs à votre politique d’auto-labelling existante ou en créer une nouvelle. Voir Partie 3 du guide Configuration_Microsoft_Purview_2026.docx. |
| Appareils onboardés dans MDE | Les postes Windows apparaissent dans Defender → Appareils avec statut Actif. C’est nécessaire pour Cloud Discovery MDCA et Endpoint DLP. | Suivre la section 1.3.3 de ce guide pour l’onboarding via Intune. |



### 2.3.3 Points de vigilance spécifiques aux tenants existants
Même avec les prérequis en place, plusieurs situations fréquentes en PME peuvent bloquer le déploiement Shadow AI ou créer des conflits silencieux.


| Situation fréquente | Ce qu’il faut faire |
|---|---|
| Étiquettes avec des noms différents du guide Configuration_Microsoft_Purview_2026.docx (ex : Privé, Restreint, Très secret) | Ne pas renommer les étiquettes existantes, cela réinitialise la protection sur tous les fichiers déjà étiquetés. Mapper les noms existants aux niveaux du guide et documenter la correspondance. Utiliser ces noms dans les règles DLP Shadow AI de la Partie 3. |
| Politiques DLP déjà actives en mode Blocage | Ajouter les nouvelles règles Shadow AI à une politique existante plutôt que d’en créer une nouvelle. Passer la règle en mode Test uniquement, pas la politique entière. Vérifier que la nouvelle règle ne crée pas de doublon de blocage sur les mêmes conditions. |
| MDCA déjà configuré avec des politiques de découverte existantes | Faire l’inventaire des politiques MDCA avant d’en créer de nouvelles (section 1.7). Vérifier si des applications IA sont déjà marquées Sanctionnées. Ne pas modifier ces statuts sans validation métier. |
| Conditional Access App Control déjà actif | Vérifier les politiques CA existantes avant de créer celles de la section 1.8. Des politiques en conflit peuvent bloquer des utilisateurs légitimes. Tester en mode Rapport uniquement (Report-only) au moins 1 semaine. |
| Purview Suite absente, uniquement Business Premium | DSPM for AI, Insider Risk Management et Endpoint DLP avancé ne sont pas disponibles sans l’add-on Purview Suite. Cet add-on est un prérequis obligatoire de ce guide (voir section 0.1). Si votre organisation ne l’a pas encore acquis, la couche MDCA (Partie 1) reste fonctionnelle, mais les Parties 3 à 5 ne pourront pas être déployées. |




### 2.3.4 Ordre de déploiement recommandé pour un tenant existant

Sur un tenant existant, déployez les éléments dans cet ordre strict pour éviter les effets de bord.

| Ordre | Action |
|---|---|
| 1 | Faire l’inventaire complet (section 2.3.1). Ne rien modifier avant. |
| 2 | Activer l’audit si pas encore fait. Attendre 48-72h. |
| 3 | Vérifier ou compléter les étiquettes et groupes mail-enabled (prérequis RMS). |
| 4 | Déployer MDCA, Cloud Discovery et politiques de détection Shadow AI (Partie 1 de ce guide). Phase la plus sûre : uniquement surveillance, pas de blocage. |
| 5 | Ajouter les règles DLP Shadow AI en mode Test (Partie 3). Attendre 48h et valider avant d’activer. |
| 6 | Configurer DSPM for AI et Insider Risk Management si Purview Suite est disponible (Parties 4 et 5). |
| 7 | Activer le blocage MDCA (Non-sanction + Defender for Endpoint). C’est l’étape la plus visible pour les utilisateurs, prévenir en amont. |



---

[← Partie 1 — Microsoft Defender for Cloud Apps (MDCA)](01-mdca.md) | [Partie 3 — Politiques DLP : blocage de l’exfiltration de données →](03-dlp.md)
