---
title: "Partie 0 — Avant de commencer | Gouvernance Shadow AI Microsoft 365"
description: "Licences requises, comptes et rôles, portails utilisés, installation PowerShell Exchange Online, checklist de départ pour le déploiement Shadow AI Microsoft 365."
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

# Partie 0 — Avant de commencer


## 0.1 Licences requises - Vérification préalable
Avant de commencer, vérifiez que vous disposez bien des licences nécessaires. Ce guide utilise les deux licences suivantes :


| Licence | Ce qu'elle inclut pour ce guide |
|---|---|
| Microsoft 365 Business Premium | Entra ID P1, Intune, Defender for Business, Exchange Online Plan 2, SharePoint, OneDrive, Teams. |
| Microsoft Defender and Purview Suites for Microsoft 365 Business Premium | MDCA (Defender for Cloud Apps), Purview Suite complète : DLP avancée, Labels de sensibilité, DSPM for AI, Endpoint DLP, Insider Risk Management, Defender for Identity, Purview Information Protection, Purview Audit. |



| ℹ️  INFO Comment vérifier vos licences : Connectez-vous à https://admin.microsoft.com → Facturation → Vos produits. Vous devez voir les deux licences listées ci-dessus avec un statut Actif.<br>Note licences Copilot M365 : DSPM for AI peut surveiller les interactions Copilot M365, mais Microsoft 365 Copilot n’est pas inclus dans Business Premium ni dans Defender and Purview Suite. Il fait l’objet d’une licence distincte : «Microsoft 365 Business Premium and Microsoft 365 Copilot Business». Sans cette licence, DSPM for AI sera actif mais n’aura pas de données d’interaction Copilot M365 à surveiller. |
|---|



## 0.2 Comptes et rôles nécessaires
Vous avez besoin d'un compte avec les droits administrateur. Voici comment attribuer les rôles nécessaires :


| 1 | Accéder au centre d'administration Entra ID Ouvrez un navigateur (Edge ou Chrome recommandé). Allez sur : https://entra.microsoft.com Connectez-vous avec votre compte administrateur global. |
|---|---|



| 2 | Attribuer les rôles administrateur Dans le menu de gauche, cliquez sur Identité → Rôles et administrateurs → Rôles et administrateurs. Recherchez et attribuez les rôles suivants à votre compte (ou à un compte dédié) : -  Administrateur général (Global Administrator) → pour l'accès complet. -  Administrateur de conformité (Compliance Administrator) → pour Purview. -  Administrateur de sécurité (Security Administrator) → pour MDCA. Cliquez sur le rôle → Ajouter des attributions → sélectionnez votre compte → Ajouter. |
|---|---|



| ⚠️  ATTENTION Bonne pratique : Évitez d'utiliser le compte Administrateur général au quotidien. Créez un compte dédié aux tâches d'administration (ex: admin-securite@votre-domaine.ch) et attribuez-lui uniquement les rôles nécessaires. |
|---|



## 0.3 Portails utilisés dans ce guide

| Portail | URL |
|---|---|
| Centre d'administration Microsoft 365 | https://admin.microsoft.com |
| Microsoft Entra ID (Azure AD) | https://entra.microsoft.com |
| Microsoft Purview | https://purview.microsoft.com |
| Microsoft Defender (MDCA inclus) | https://security.microsoft.com |
| Microsoft Intune | https://intune.microsoft.com |
| PowerShell Exchange Online | Via PowerShell, voir section 0.4 |



## 0.4 Installation de PowerShell Exchange Online (pour l'audit Purview)
Certaines configurations nécessitent PowerShell. Voici comment l'installer correctement depuis zéro :


| 1 | Ouvrir PowerShell en tant qu'administrateur Appuyez sur la touche Windows. Tapez : PowerShell Cliquez droit sur Windows PowerShell → Exécuter en tant qu'administrateur. Cliquez Oui si Windows demande une confirmation. |
|---|---|



| 2 | Installer le module Exchange Online Dans la fenêtre PowerShell, tapez exactement la commande suivante et appuyez sur Entrée : Install-Module -Name ExchangeOnlineManagement -Force -AllowClobber Si le système vous demande d'installer NuGet ou PSGallery, tapez O (Oui) et appuyez sur Entrée. L'installation prend 1 à 3 minutes selon votre connexion internet. |
|---|---|



| 3 | Se connecter à Exchange Online Tapez la commande suivante : Connect-ExchangeOnline -UserPrincipalName admin@votre-domaine.ch Remplacez admin@votre-domaine.ch par votre adresse email d'administrateur. Une fenêtre de connexion Microsoft s'ouvre. Entrez votre mot de passe et validez le MFA si demandé. Une fois connecté, vous voyez une invite PowerShell normale - la connexion est établie. |
|---|---|



| ℹ️  INFO Si la commande Install-Module échoue avec une erreur de sécurité, tapez d'abord : Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser Puis relancez l'installation du module. |
|---|





## 0.5 État requis avant de continuer. Checklist de départ

| ✓ | Action | Statut |
|---|---|---|
| ☐ | Licences M365 Business Premium actives pour tous les utilisateurs concernés | À faire |
| ☐ | Licence Microsoft Defender and Purview Suites active sur le tenant | À faire |
| ☐ | Compte administrateur avec rôles Conformité + Sécurité + Global Admin | À faire |
| ☐ | Accès confirmé à https://purview.microsoft.com (page Purview visible) | À faire |
| ☐ | Accès confirmé à https://security.microsoft.com (page Defender visible) | À faire |
| ☐ | PowerShell Exchange Online installé et connexion testée avec succès | À faire |
| ☐ | Toggle « Microsoft Defender for Cloud Apps » activé dans MDE Fonctionnalités avancées (section 1.3.4). Critique : sans lui, le blocage MDCA→MDE est inactif | À faire |
| ☐ | Network Protection activé en mode Block via Intune Catalogue des paramètres (section 1.3.5). Critique : sans lui, aucun blocage réel n’est effectif | À faire |
| ☐ | Navigateur Edge ou Chrome à jour \| \| □ \| Abonnement Azure lié au tenant avec facturation à l’utilisation (PAYG) — requis uniquement pour la section 3.4 (DLP inline Edge). Sans ce prérequis, les sections 1 à 9 restent pleinement fonctionnelles | À faire |



---

[← Introduction](introduction.md) | [Partie 1 — Microsoft Defender for Cloud Apps (MDCA) →](01-mdca.md)
