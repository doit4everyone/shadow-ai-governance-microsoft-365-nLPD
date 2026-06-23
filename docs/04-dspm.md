---
title: "Partie 4 — Surveillance Shadow AI dans Purview (DSPM for AI) | Gouvernance Shadow AI Microsoft 365"
description: "Surveillance des interactions IA via DSPM for AI : tableaux de bord, tâches de configuration, stratégies actives, Explorateur d'activités."
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

# Partie 4 — Surveillance Shadow AI dans Purview (DSPM for AI)

⚠️ Copilot M365 = IA approuvée, mais à risque résiduel : Sanctionner Copilot dans MDCA ne suffit pas à éliminer tous les risques. Trois risques résiduels spécifiques : (1) Hallucinations sur données classifiées — Copilot peut générer des synthèses incorrectes de documents sensibles ; (2) Mauvaise interprétation des niveaux de classification — un document Confidentiel cité dans un contexte non-classifié ; (3) Extraction indirecte — un prompt mal formulé peut exposer des informations issues d’autres utilisateurs via le contexte partagé de l’organisation. DSPM for AI (cette partie) surveille ces interactions et détecte les usages anormaux de Copilot.

## 4.1 Comprendre DSPM for AI
ℹ️ Cette partie s’applique à tous les cas (A, B et C). DSPM for AI est le tableau de bord central pour surveiller le Shadow AI. DSPM for AI (Data Security Posture Management for AI) est la fonctionnalité Purview dédiée à la surveillance des interactions avec les outils d'IA. Elle surveille :
- Les interactions avec Microsoft Copilot M365.
- Les interactions avec Copilot Studio (agents personnalisés).
- Les accès aux données sensibles via des requêtes IA.
- Les anomalies comportementales liées à l'IA (ex: un utilisateur qui interroge Copilot sur des milliers de documents en peu de temps).


> **nLPD SUISSE nLPD Art. 62 - Secret professionnel : DSPM for AI permet de prouver que vous surveillez activement les accès aux données sensibles via les outils IA, ce qui est essentiel pour la conformité nLPD en cas d'audit du PFPDT.**




## 4.2 Vérifier et activer les tâches de configuration DSPM





| Étape | Description |
|---|---|
| 1 | Accéder à DSPM Chemin : Microsoft Purview → DSPM (menu gauche) → page « Gestion de la posture de sécurité des données ». Le tableau de bord affiche trois métriques de posture : Découverte des données, Protection des données, Enquête sur les données. Ces métriques sont alimentées automatiquement par les politiques DLP, IRM et les étiquettes de confidentialité déjà en place. |
| 2 | Vérifier les tâches de configuration DSPM → Tâches et actions → Tâches de configuration. La page affiche toutes les tâches de configuration avec leur statut (Non démarré, Ignoré, Terminé). Vérifiez que ces deux tâches sont ✅ Terminé : - Activer Microsoft Purview Audit (Obligatoire) — indispensable pour que DSPM ait accès aux journaux d’activité. - Protégez vos données avec des étiquettes de sensibilité (Recommandé) — confirme que les étiquettes du guide Purview sont en place. Les autres tâches (facturation, extension navigateur, intégration appareils) sont recommandées mais nécessitent des licences supplémentaires ou une facturation Azure. Elles peuvent être activées progressivement. |
| 3 | Vérifier les stratégies actives DSPM → Stratégies. La page consolide toutes les stratégies Purview liées à la sécurité IA, groupées par solution : - Protection contre la perte de données (2) : DLP-Protection-Copilot (créée dans Configuration_Microsoft_Purview_2026.docx — Partie 4.4) et DLP-Endpoint-Shadow-AI-CopyPaste (section 3.3.2 de ce guide) — statut : Activé ✅ Note : si vous êtes en Cas A et n’avez pas encore appliqué le guide Configuration_Microsoft_Purview_2026.docx, DLP-Protection-Copilot n’existe pas encore dans DSPM — c’est normal à ce stade. Elle sera créée lors du déploiement Purview. - Gestion des risques internes (3) : IRM-Shadow-AI-Navigation (section 5.2 de ce guide), IRM-Risque-IA et IRM-Fuites-Données-PME (créées dans Configuration_Microsoft_Purview_2026.docx) — statut : Activé ✅. Note Cas A : si vous n’avez pas encore appliqué le guide Purview, seule IRM-Shadow-AI-Navigation existera à ce stade Si une stratégie manque ou est désactivée, cliquez dessus pour diagnostiquer et corriger depuis cette vue centrée. |


## 4.3 Utiliser l’Explorateur d’activités pour le suivi




| Étape | Description |
|---|---|
| 1 | Accéder à l’Explorateur d’activités DSPM → Découvrir → Explorateur d’activités. La page propose deux onglets : Types d’activité (activités DLP et étiquettes) et Activités d’IA (interactions Copilot et IA). |
| 2 | Onglet « Types d’activité » — surveiller les activités DLP et étiquettes Utilisez les filtres disponibles pour isoler les événements critiques : - Activité : Étiquette rétrogadée — détecte les tentatives de contournement des politiques DLP. - Activité : Email sent to external recipient — partages externes depuis Exchange. - Étiquettes de sensibilité : filtrez par étiquette (ex. 4 — RH-Confidentiel) pour identifier les fichiers sensibles concernés. Cliquez sur Exporter pour générer un rapport CSV ou PDF utilisable comme preuve de surveillance pour un audit nLPD (PFPDT). |
| 3 | Onglet « Activités d’IA » — surveiller les interactions Copilot Cet onglet est spécifique aux interactions avec les outils IA. Il affiche : - Type d’activité : AI Interaction (invites et réponses Copilot), DLP rule match, Sensitive info types. - Catégorie d’application IA : Copilot experiences & agents. - Applications : M365Copilot - Apps, Microsoft 365 Copilot and Copilot Chat, M365Copilot - Bizhat. - Participant utilisateur : identifie l’utilisateur à l’origine de l’interaction. - Étiquette de confidentialité : indique si du contenu sensible étiqueté a été impliqué dans l’interaction. Filtrez par période et par utilisateur pour investiguer un incident spécifique. Utilisez Exporter pour archiver les preuves. |


---

[← Partie 3 — Politiques DLP : blocage de l’exfiltration de données](03-dlp.md) | [Partie 5 — Détection comportementale (Insider Risk Management) →](05-irm.md)
