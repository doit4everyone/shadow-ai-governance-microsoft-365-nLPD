---
title: "Partie 5 — Détection comportementale (Insider Risk Management) | Gouvernance Shadow AI Microsoft 365"
description: "Détection comportementale du Shadow AI via Insider Risk Management : création de la stratégie IRM-Shadow-AI-Navigation et configuration des indicateurs."
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

# Partie 5 — Détection comportementale (Insider Risk Management)


## 5.1 Comprendre Insider Risk Management
ℹ️ Cette partie s’applique à tous les cas (A, B et C). IRM détecte spécifiquement les comportements Shadow AI : téléchargements massifs vers des IA externes, exfiltration de données, rétrogradation d’étiquettes pour contourner les DLP.  Insider Risk Management (IRM) est la fonctionnalité Purview qui détecte les comportements internes à risque - y compris les employés qui tentent d'exfiltrer des données via des outils IA non autorisés.


> **Cas d'usage Shadow AI pour IRM**
>
> - Employé qui télécharge massivement des fichiers clients avant de les uploader sur une IA externe.
>
> - Employé en préavis de départ qui copie de grandes quantités de données sensibles.
>
> - Employé qui accède régulièrement à des documents Secret en dehors de ses heures habituelles.
>
> - Employé qui rétrograde délibérément des étiquettes de sensibilité pour contourner les DLP.




## 5.2 Créer la stratégie IRM-Shadow-AI-Navigation

> **Accès : Microsoft Purview → Gestion des risques internes**
>
> Chemin : Microsoft Purview → Gestion des risques internes → Stratégies → + Créer une stratégie. Le wizard s’ouvre avec 7 sections de navigation (détaillées ci-dessous en 12 étapes numérotées, certaines sections comportant plusieurs pages) dans la barre de gauche : Modèle de stratégie, Nom et description, Utilisateurs et groupes, Contenu à prioriser, Événement déclencheur, Indicateurs, Terminer.
















| Étape | Description |
|---|---|
| 1 | Modèle de stratégie : Utilisation à risque de l’IA Dans la liste des modèles, sélectionnez : Utilisation à risque de l’IA. Ce modèle est spécifiquement conçu pour détecter les comportements à risque liés aux outils d’IA générative : navigation vers des sites d’IA non autorisés, interactions risquées avec Copilot, saisie d’invites sensibles. Cliquez Suivant. |
| 2 | Nom et description Nom : IRM-Shadow-AI-Navigation Description : Détecte la navigation vers des sites d’IA générative non autorisés et les interactions à risque avec des outils IA externes. Couvre nLPD art. 8. Cliquez Suivant. |
| 3 | Utilisateurs et groupes Utilisateurs, groupes et étendues adaptatives : All (tous les utilisateurs). Utilisateurs et groupes exclus : Aucun. Cliquez Suivant. |
| 4 | Contenu à prioriser — Étiquettes de confidentialité à hiérarchiser Toute activité associée au contenu portant ces étiquettes recevra un score de risque plus élevé. Cliquez + Ajouter ou modifier des étiquettes de confidentialité et sélectionnez les 3 étiquettes suivantes : - 3 — Direction-Confidentiel - 4 — RH-Confidentiel - 5 — Finances-Confidentiel Résultat attendu : 3 éléments. Cliquez Suivant. |
| 5 | Contenu à prioriser — Types d’informations sensibles à hiérarchiser Toute activité associée au contenu contenant ces informations sensibles recevra un score de risque plus élevé. Cliquez + Ajouter ou modifier des types d’informations sensibles et sélectionnez : - Credit Card Number - International Banking Account Number (IBAN) - Swiss Social Security Number AHV Résultat attendu : 3 éléments. Cliquez Suivant. |
| 6 | Contenu à prioriser — Score Page : « Décider s’il faut noter uniquement l’activité avec un contenu prioritaire » Sélectionnez : ● Obtenir des alertes pour toutes les activités (option par défaut, recommandée). Cette option génère des alertes pour toute navigation vers un outil d’IA non autorisé, que du contenu étiqueté soit impliqué ou non. Le contenu prioritaire (3 SITs + 3 étiquettes) augmentera uniquement le score de risque quand il est présent. Cliquez Suivant. |
| 7 | Événement déclencheur : Navigation vers des sites web d’IA générative Page : « Choisir un événement déclencheur pour cette stratégie » Cochez : ☑ L’utilisateur a accédé à un site Web potentiellement dangereux. Sous-option à cocher : ☑ Navigation vers des sites web d’IA générative. Note : Un avertissement indique que certains indicateurs sont désactivés. Cet avertissement concerne les indicateurs Copilot et Enterprise AI (nécessitent une facturation Azure pay-per-use). L’indicateur « Navigation vers des sites web d’IA générative » fait partie du groupe « Indicateurs de navigation à risque (préversion) » (14/14 actifs) et est pleinement opérationnel. Laissez toutes les autres options non cochées (Messages inappropriés, Compte Entra supprimé, Copilot, Connecteur RH). Cliquez Suivant. |
| 8 | Seuils de déclenchement des événements Page : « Choisir les seuils de déclenchement des événements » Sélectionnez : ● Appliquer des seuils intégrés (RECOMMANDÉ). Les seuils Microsoft sont calibrés sur le nombre d’événements par jour. Cette option évite les faux positifs tout en détectant les comportements anormaux. Cliquez Suivant. |
| 9 | Indicateurs : 4/11 sélectionnés Page : « Indicateurs » — affiche les indicateurs actifs pour cette stratégie. Indicateurs actifs (4/11) : - Indicateurs de navigation à risque (1/1) : Navigation vers des sites web d’IA générative ☑ - Expériences Microsoft Copilot (2/2) : Réception d’une réponse sensible de Copilot ☑ + Entrée d’une invite à risque dans Copilot ☑ - Indicateurs d’alerte DLP (1/1) : Génération d’alertes à partir des stratégies DLP sélectionnées ☑ Indicateurs non disponibles (7/11) : Applications IA d’entreprise (0/2), Autres applications IA (0/1), Indicateurs de sécurité du contenu Azure AI (0/2) — nécessitent une facturation Azure pay-per-use non configurée dans ce tenant. Cliquez Suivant. |
| 10 | Options de détection Page : « Options de détection » — détections de séquence et amplificateurs de résultats de risque. Laissez tous les paramètres par défaut (☑ Sélectionner tout coché, sous-options en préversion grisées). Cliquez Suivant. |
| 11 | Seuils d’indicateur Page : « Choisir le type de seuil pour les indicateurs » Sélectionnez : ● Appliquer les seuils fournis par Microsoft. Les seuils Microsoft sont basés sur le nombre d’événements enregistrés par activité par jour et déterminent la gravité des alertes générées (faible, moyenne, élevée). Cliquez Suivant. |
| 12 | Vérifier les paramètres et terminer Page : « Vérifier les paramètres et terminer » — récapitulatif complet de la stratégie. Vérifiez les paramètres suivants : - Modèle : Utilisation à risque de l’IA - Nom : IRM-Shadow-AI-Navigation - Utilisateurs : All / Exclusions : Aucun - Contenu prioritaire : Credit Card Number, IBAN, Swiss Social Security Number AHV, 3—Direction-Confidentiel, 4—RH-Confidentiel, 5—Finances-Confidentiel Événement déclencheur : Événement déclencheur d’utilisation risquée du navigateur intégré - Indicateurs : 4/11 sélectionnés, aucun seuil personnalisé Cliquez Envoyer pour créer la stratégie. Confirmation : « Votre stratégie est créée ». La stratégie prendra effet immédiatement mais peut prendre jusqu’à 24 heures pour générer les premières alertes. |


> **nLPD SUISSE Insider Risk Management traite des données comportementales des employés (historique de navigation, accès aux fichiers, activité Copilot). En Suisse, cela constitue un traitement de données personnelles au sens de l’art. 5 nLPD. Obligations préalables : information des employés (art. 19 nLPD) sur la surveillance et ses finalités, et potentiellement consultation des représentants du personnel selon le règlement intérieur. Consultez votre service juridique avant d’activer cette fonctionnalité en production.**



---

[← Partie 4 — Surveillance Shadow AI dans Purview (DSPM for AI)](04-dspm.md) | [Partie 6 — Contrôles navigateur et Power Platform →](06-reseau.md)
