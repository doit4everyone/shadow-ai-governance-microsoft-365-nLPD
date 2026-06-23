---
title: "Partie 7 — Gouvernance Power Platform et Copilot Studio | Gouvernance Shadow AI Microsoft 365"
description: "Gouvernance des connecteurs Power Platform et Copilot Studio : politique DLP-ConnecteursProd bloquant 17 connecteurs IA non autorisés."
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

# Partie 7 — Gouvernance Power Platform et Copilot Studio


## 7.1 Pourquoi gouverner Power Platform pour le Shadow AI

| ! | Vecteur Shadow AI souvent ignoré Power Platform (Power Automate, Power Apps, Copilot Studio) permet aux utilisateurs de créer des automatisations et des agents IA sans intervention IT. Sans gouvernance, un collaborateur peut créer un flux Power Automate qui envoie automatiquement des données SharePoint vers une API d’IA externe (OpenAI, Anthropic, Mistral) — contournant complètement MDCA, Purview et les indicateurs MDE. La politique DLP Power Platform est indépendante de Microsoft Purview. Elle se configure dans le Centre d’administration Power Platform et s’applique aux connecteurs utilisés dans les flux Power Automate et agents Copilot Studio. |
|---|---|



## 7.2 Inventaire préalable obligatoire

| ⚠️ PRÉREQUIS : Inventaire AVANT la politique DLP La politique DLP Power Platform est active immédiatement et sans mode simulation. Tout flux utilisant un connecteur bloqué cessera de fonctionner dès la création. Effectuez l’inventaire ci-dessous AVANT d’appliquer la politique. |
|---|



| 1 | Inventaire Power Automate Chemin : admin.powerplatform.microsoft.com → icône Gérer → Power Automate → onglet Inventaire. La liste affiche tous les flux existants avec leur propriétaire, environnement et type. Pour chaque flux : cliquez sur son nom pour vérifier les connecteurs utilisés (section « Connexions » à droite). Exemple tenant lab : 1 flux existant de type notification Teams (connecteur unique : Microsoft Teams). Aucun connecteur HTTP. Aucun risque. |
|---|---|



| 2 | Inventaire Copilot Studio Chemin : admin.powerplatform.microsoft.com → icône Gérer → Copilot Studio → onglet Assistants. Exemple tenant lab : 0 assistant Copilot Studio. Aucun risque. Si des assistants existent : vérifiez qu’aucun n’utilise « Knowledge source with public websites » ou un connecteur IA externe. Si c’est le cas, contactez le propriétaire avant d’appliquer la politique. |
|---|---|




## 7.3 Créer la politique DLP-ConnecteursProd

| 1 | Accéder aux stratégies de données Chemin : admin.powerplatform.microsoft.com → Sécurité → Données et confidentialité → Stratégie de données → + Nouvelle stratégie. Note : si accès refusé, attribuez le rôle « Administrateur de Power Platform » via Entra ID → Rôles et administrateurs. Attendez 5 à 10 minutes. Étape 1 — Nom de la stratégie : DLP-ConnecteursProd → Suivant. |
|---|---|



| 2 | Connecteurs prédéfinis — groupe Professionnel (Entreprise) Recherchez et déplacez vers « Professionnel » (clic ⋮ → Passer à Entreprise) : SharePoint, OneDrive Entreprise, Microsoft Teams, Office 365 Outlook, Office 365 Groups Mail, Microsoft Entra ID, OneNote (professionnel). Résultat attendu : onglet Professionnel affiche (7). Note : SharePoint, OneDrive et Teams ne sont pas blocables (Blocable = Non) — seul « Passer à Entreprise » est disponible pour ces connecteurs. |
|---|---|



| 3 | Connecteurs prédéfinis — groupe Bloqué Recherchez et déplacez vers « Bloqué » (clic ⋮ → Bloquer) : Connecteurs HTTP (filet universel) : HTTP, HTTP Webhook, HTTP avec Microsoft Entra ID, When a HTTP request is received. Garde-fou : le connecteur HTTP générique sert aussi à de nombreuses intégrations légitimes (webhooks, API internes, outils métier). Le bloquer sur tous les environnements, sans simulation et avec effet immédiat, peut casser des automatisations en production. Si vous n'avez pas la certitude qu'aucun flux légitime ne l'utilise, ne bloquez d'abord que les connecteurs IA générative tiers (ci-dessous), validez l'absence d'impact, puis ajoutez les connecteurs HTTP dans un second temps. L'inventaire de la section 7.2 est ici impératif, pas optionnel. Connecteurs Copilot Studio : Knowledge source with public websites and data in Copilot Studio, Knowledge source with SharePoint and OneDrive in Copilot Studio. Connecteurs IA générative tiers : OpenAI Assistants (éditeur indépendant), OpenAI GPT (éditeur indépendant), OpenAI (Independent Publisher), Azure OpenAI, Anthropic (éditeur indépendant), Hugging Face (éditeur indépendant), Mistral (éditeur indépendant), Cohère (éditeur indépendant), Perplexity AI (éditeur indépendant), Zapier MCP, Zapier NLA (éditeur indépendant). Résultat attendu : onglet Bloqué affiche (17). Note : Azure OpenAI est le connecteur Microsoft officiel. Si l’organisation utilise Azure OpenAI de façon approuvée, déplacez-le vers Professionnel plutôt que Bloqué. |
|---|---|



| 4 | Connecteurs personnalisés, Étendue et création Connecteurs personnalisés : laissez la règle par défaut Ordre 1 / Ignorer / * → Suivant. Étendue : sélectionnez « Ajouter tous les environnements » → Suivant. Revoir : vérifiez (7) Entreprise, (+1600) Hors entreprise, (17) Bloqué, Étendue = Tous les environnements. Cliquez « Créer une stratégie ». Confirmation : « Une nouvelle stratégie a bien été créée. » La politique est active immédiatement. |
|---|---|



### 7.4 Blocage des agents IA tiers — Copilot Store

| 1 | Accéder aux paramètres agents Centre d’administration M365 (admin.microsoft.com) → Agents → Settings → Allowed agent types. ⚠️ Cette section n’est visible que si votre tenant est inscrit au programme Frontier (mai 2026). Si elle n’apparaît pas encore, les contrôles de la Partie 7 restent actifs indépendamment. |
|---|---|
| 2 | Bloquer les agents tiers Page Allowed agent types → décocher Allow apps and agents built by external publishers. Cliquez Save. Résultat : tous les agents IA tiers (Asana, Jira Cloud, Webex AI Agent, etc.) sont bloqués et indisponibles dans le Copilot Store de votre organisation. |
| ℹ️ | Complémentarité avec DLP-ConnecteursProd (section 7.3) La politique Power Platform (section 7.3) bloque les connecteurs HTTP des flux Power Automate. Le blocage des agents tiers agit à une couche différente : il bloque les agents IA autonomes dans le Copilot Store, indépendamment des connecteurs. Ces deux contrôles sont complémentaires et non redondants. |




---

[← Partie 6 — Contrôles navigateur et Power Platform](06-reseau.md) | [Partie 8 — Tests et validation →](08-tests.md)
