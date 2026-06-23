---
title: "Partie 6 — Contrôles navigateur et Power Platform | Gouvernance Shadow AI Microsoft 365"
description: "Blocage réseau des sites IA : profil Edge URLBlocklist via Intune, indicateurs MDE URL/Domaines API pour bloquer les appels directs aux API d'IA générative."
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

# Partie 6 — Contrôles navigateur et Power Platform


## 6.1 Pourquoi Edge for Business est important pour bloquer le Shadow AI
ℹ️ Parties 6 et 7 groupées. Microsoft Edge for Business est le navigateur d'entreprise géré par Intune. Il offre des contrôles natifs pour bloquer ou restreindre l'accès aux sites d'IA directement dans le navigateur, en complément de MDCA.
⚠️ Limite de l'URLBlocklist Edge : les fonctions IA intégrées nativement au navigateur (Gemini sidebar dans Chrome, Copilot intégré dans Edge, extensions IA utilisant WebRTC) ne sont pas bloquées par la liste d’URL, car elles n’émettent pas de requête HTTP vers un domaine externe standard. Pour ces cas : désactivez les extensions non-approuvées via Intune → Profil Edge → paramètre ExtensionInstallBlocklist = * (bloque toutes les extensions sauf celles de la liste blanche).


## 6.2 Créer le profil Edge - Restrictions Shadow AI dans Intune

| 1 | Accéder à la configuration des profils Chemin : https://intune.microsoft.com → Appareils → Gestion des appareils → Configuration. Cliquez sur + Créer → + Nouvelle stratégie. Dans le panneau « Créer un profil » : - Plateforme : Windows 10 et versions ultérieures - Type de profil : Catalogue des paramètres (méthode recommandée 2026 — remplace l’ancien modèle « Paramètres administratifs ») Cliquez Créer. |
|---|---|



| 2 | Informations de base Nom : Edge - Restrictions Shadow AI Description : Bloque l’accès aux sites d’IA générative non autorisés via Edge for Business. Couvre nLPD art. 8. Plateforme : Windows (défini automatiquement). Cliquez Suivant. |
|---|---|




| 3 | Paramètres de configuration — ajouter le blocage d’URL Cliquez sur + Ajouter des paramètres → le Sélecteur de paramètres s’ouvre. Dans le champ de recherche, tapez : URLBlocklist Dans les résultats, cliquez sur la catégorie Microsoft Edge. 8 paramètres apparaîssent. Cochez : Bloquer l’accès à une liste d’URL (sous-élément de « Block access to a list of URLs » — version Device). Fermez le sélecteur. Le paramètre apparaît dans la section Microsoft Edge avec le toggle sur Disabled. Basculez le toggle sur Enabled. Un tableau de saisie « Bloquer l’accès à une liste d’URL » apparaît avec des lignes individuelles — une URL par ligne. Cliquez directement dans la ligne vide en bas du tableau pour saisir une URL (pas de bouton « Ajouter » séparé). Des actions sont disponibles en haut du tableau : Supprimer, Trier, Importer, Exporter. ⚠️ Note : l’info-bulle « des paramètres N et N+1 dans cette catégorie ne sont pas configurés » est un message informatif normal. Le Catalogue des paramètres Edge contient des milliers d’options ; seul URLBlocklist est nécessaire pour cette procédure. Saisissez les URLs à bloquer, une par ligne : chatgpt.com gemini.google.com claude.ai perplexity.ai aistudio.google.com ⚠️ Note : si votre organisation n’utilise pas Microsoft 365 Copilot, ajoutez également copilot.microsoft.com à la liste. Si Copilot M365 est déployé et autorisé, excluez ce domaine pour ne pas bloquer l’outil approuvé. Cliquez Suivant. |
|---|---|


ℹ️ Note v1.1 : si vous déployez la DLP inline Edge (section 3.4) en mode hybride, retirez de cette liste les domaines des apps passées en Mode B (ex. : chatgpt.com, gemini.google.com). L’URLBlocklist reste actif pour les apps maintenues en Mode A. Voir section 3.4.4 pour la procédure de transition.



| 4 | Balises d’étendue et Affectations Balises d’étendue : laissez Par défaut. Cliquez Suivant. Affectations → Groupes inclus : pour un déploiement en production, cliquez Ajouter tous les appareils. Pour un déploiement contrôlé (recommandé en phase de test), cliquez Ajouter des groupes et sélectionnez un groupe Entra ID dédié (ex. GRP-Test-Shadow-AI) contenant uniquement les appareils de test. Résultat : Groupe « Tous les appareils », État Actif, Filtre Aucune. Cliquez Suivant. |
|---|---|



| 5 | Vérifier + créer Page de récapitulatif — vérifiez : - Nom : Edge - Restrictions Shadow AI - Paramètre : Block access to a list of URLs — Enabled - URLs bloquées : chatgpt.com, gemini.google.com, claude.ai, perplexity.ai, aistudio.google.com - Affectation : Tous les appareils — Actif Cliquez Créer. Le profil se déploie automatiquement sur les appareils Intune dans les 4 à 8 heures. |
|---|---|




## 6.3 Blocage des appels API IA, MDE Indicateurs personnalisés

| 1 | Prérequis — Vérifier que les indicateurs réseau sont actifs Chemin : https://security.microsoft.com → Système → Paramètres → Points de terminaison → Général → Fonctionnalités avancées. Vérifiez que « Indicateurs de réseau personnalisés » est sur Actif. Ce paramètre est nécessaire pour que les indicateurs URL/Domaines soient appliqués sur les appareils MDE-onboardés. Note : la fonctionnalité « Filtrage du contenu web » est également visible sur cette page. Elle ne joue aucun rôle actif dans ce guide : aucune catégorie « IA générative » n’est disponible dans le WCF (voir section 9.4). Le WCF reste un prérequis technique (moteur réseau) pour les indicateurs personnalisés ci-dessous. |
|---|---|



| 2 | Accéder aux indicateurs URL/Domaines Chemin : Paramètres → Points de terminaison → Règles → Indicateurs → onglet URL/Domaines. La page affiche la capacité disponible (0/15000 initialement) et les onglets : Codes de hachage de fichiers, Adresses IP, URL/Domaines, Certificats. Cliquez sur + Ajouter un élément pour créer un indicateur de blocage. |
|---|---|



| 3 | Créer un indicateur de blocage — exemple api.openai.com Le wizard « Ajouter un indicateur » comprend 4 étapes : Indicateur → Action → Détails de l’alerte → Étendue de l’organisation → Résumé. Étape 1 — Indicateur : - URL/Domaine : api.openai.com - Titre : Shadow AI - Blocage API OpenAI - Description : Bloque les appels API directs vers OpenAI (ChatGPT, GPT-4). Couvre nLPD art. 8. - Expire le : Jamais Étape 2 — Action : sélectionnez Bloquer l’exécution. Étape 3 — Détails de l’alerte : Gravité Medium, Catégorie Exfiltration. Cochez Générer une alerte. Étape 4 — Étendue : Tous les appareils de mon organisation. Cliquez Envoyer. |
|---|---|



| 4 | Créer les 5 indicateurs restants — mêmes paramètres Répétez l’opération pour chaque domaine avec les mêmes paramètres (Bloquer l’exécution / Medium / Exfiltration / Tous les appareils) : - api.anthropic.com — Shadow AI - Blocage API Anthropic (Claude) - api.mistral.ai — Shadow AI - Blocage API Mistral - api.groq.com — Shadow AI - Blocage API Groq - zapier.com — Shadow AI - Blocage Zapier (no-code IA) - make.com — Shadow AI - Blocage Make.com (no-code IA) Résultat attendu : onglet URL/Domaines affiche 6 indicateurs, capacité 6/15000. |
|---|---|




| ⚠️ Maintenance et limites Ces indicateurs constituent une liste statique. Elle couvre les fournisseurs IA majeurs connus au moment de la rédaction. Pour les domaines web grand public (chatgpt.com, claude.ai...), MDCA (Partie 1) gère dynamiquement le blocage via son catalogue d'applications cloud, mis à jour en continu par Microsoft (plusieurs milliers d'applications, dont la catégorie IA générative), sans maintenance manuelle requise pour ce vecteur. Pour les nouveaux fournisseurs API, ajoutez leur domaine API dans cet onglet. Pour les nouvelles applications web IA, elles sont généralement ajoutées automatiquement au catalogue MDCA. Prérequis indispensable : les appareils doivent être onboardés dans Microsoft Defender for Endpoint (section 1.3.3). Sans MDE, les indicateurs n’ont aucun effet. |
|---|



---

[← Partie 5 — Détection comportementale (Insider Risk Management)](05-irm.md) | [Partie 7 — Gouvernance Power Platform et Copilot Studio →](07-power-platform.md)
