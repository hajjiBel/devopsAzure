# Index des Laboratoires Azure DevOps Pipelines - Formation Complète

## 📚 Structure du Programme de Formation

Ce programme de formation est composé de **7 laboratoires indépendants** couvrant l'ensemble du cycle d'apprentissage Azure DevOps Pipelines.

---

## 📋 Fichiers de Laboratoires Créés

### LAB 1.1: Créer Votre Premier Pipeline Azure DevOps
**Fichier**: `LAB-1-1-Premier-Pipeline.md`
- **Durée**: 2 heures
- **Objectifs**: Mise en place initiale, création d'un pipeline YAML de base
- **Contenu**: Configuration du projet, déclenchement du pipeline, lecture des logs
- **Prérequis**: Compte Azure DevOps, Git
- **Livrables**: Pipeline fonctionnel, screenshots d'exécution

---

### LAB 2.1: Implémenter un Pipeline Multi-Étapes
**Fichier**: `LAB-2-1-Pipeline-Multi-Etapes.md`
- **Durée**: 3 heures
- **Objectifs**: Structure hiérarchique, dépendances, exécution conditionnelle
- **Contenu**: Stages, Jobs, Steps, variables, conditions
- **Prérequis**: LAB 1.1 complété
- **Livrables**: Pipeline Build → Test → Deploy, documentation des dépendances

---

### LAB 3.1: Gestion des Variables, Paramètres et Secrets
**Fichier**: `LAB-3-1-Variables-Secrets-Parametres.md`
- **Durée**: 2.5 heures
- **Objectifs**: Gestion sécurisée des variables, groupes de variables, permissions
- **Contenu**: Variables de pipeline, groupes de variables, variables secrètes, portée
- **Prérequis**: LAB 2.1 complété
- **Livrables**: Groupes de variables configurés, secrets masqués, documentation de sécurité

---

### LAB 4.1: Configuration et Gestion des Agents On-Premises
**Fichier**: `LAB-4-1-Agents-On-Premises.md`
- **Durée**: 3 heures
- **Objectifs**: Déploiement d'agents auto-hébergés, gestion des pools
- **Contenu**: Création de pools, PAT, installation agents (Windows/Linux), capacités
- **Prérequis**: LAB 3.1 complété, accès administrateur
- **Livrables**: Agent on-premises enregistré, pipeline exécuté sur agent personnalisé

---

### LAB 5.1: Tâches Intégrées, Tasks et Templates Réutilisables
**Fichier**: `LAB-5-1-Tasks-Templates.md`
- **Durée**: 3 heures
- **Objectifs**: Tasks du marketplace, templates réutilisables, paramétrisation
- **Contenu**: Tasks prédéfinies, création de templates (steps, jobs, stages), conditions
- **Prérequis**: LAB 4.1 complété
- **Livrables**: Templates réutilisables, pipeline modulaire, documentation des templates

---

### LAB 6.1: Sécurité, Permissions et Portes d'Approbation
**Fichier**: `LAB-6-1-Securite-Permissions-Approbations.md`
- **Durée**: 2.5 heures
- **Objectifs**: Modèle de sécurité, approvals, contrôle d'accès
- **Contenu**: Environnements, portes d'approbation, restrictions de branche, audit
- **Prérequis**: LAB 5.1 complété
- **Livrables**: Pipeline avec approvals en production, audit trail, documentation des permissions

---

### LAB 7.1: Déploiements Multi-Environnements et Stratégies Avancées
**Fichier**: `LAB-7-1-Deployments-Multi-Environnements.md`
- **Durée**: 4 heures
- **Objectifs**: Stratégies de déploiement, blue-green, canary, rollback
- **Contenu**: Multi-environnements, stratégies avancées, health checks, runbooks
- **Prérequis**: LAB 6.1 complété
- **Livrables**: Pipeline production-ready, runbooks de rollback, documentation complète

---

## 🎯 Parcours d'Apprentissage Recommandé

```
LAB 1.1 (2h)
    ↓
LAB 2.1 (3h)
    ↓
LAB 3.1 (2.5h)
    ↓
LAB 4.1 (3h)
    ↓
LAB 5.1 (3h)
    ↓
LAB 6.1 (2.5h)
    ↓
LAB 7.1 (4h)
    ↓
PROJET CAPSTONE (8h)
```

**Durée totale**: ~28.5 heures de labs + 8 heures capstone = **36.5 heures**

---

## 📊 Tableau Récapitulatif

| LAB | Titre | Durée | Concepts Clés |
|-----|-------|-------|--------------|
| 1.1 | Premier Pipeline | 2h | YAML, Trigger, Déploiement |
| 2.1 | Multi-Étapes | 3h | Stages, Jobs, Dépendances |
| 3.1 | Variables & Secrets | 2.5h | Variables, Groupes, Sécurité |
| 4.1 | Agents On-Prem | 3h | Pools, PAT, Auto-hébergement |
| 5.1 | Tasks & Templates | 3h | Tasks, Templates, Réutilisabilité |
| 6.1 | Sécurité & Approvals | 2.5h | Permissions, Approvals, Audit |
| 7.1 | Multi-Environnements | 4h | Stratégies, Blue-Green, Rollback |

---

## ✅ Compétences Acquises par LAB

### Après LAB 1.1
- Créer un projet Azure DevOps
- Comprendre la syntaxe YAML de base
- Déclencher automatiquement un pipeline

### Après LAB 2.1
- Structurer un pipeline multi-étapes
- Gérer les dépendances entre étapes
- Implémenter l'exécution conditionnelle

### Après LAB 3.1
- Gérer les variables à différents niveaux
- Sécuriser les données sensibles
- Comprendre la portée des variables

### Après LAB 4.1
- Déployer un agent auto-hébergé
- Gérer les pools d'agents
- Exécuter des pipelines sur infrastructure personnalisée

### Après LAB 5.1
- Utiliser les tasks du marketplace
- Créer des templates réutilisables
- Implémenter des pipelines modulaires

### Après LAB 6.1
- Configurer des portes d'approbation
- Gérer les permissions finement
- Auditer les accès et approvals

### Après LAB 7.1
- Déployer en multi-environnements
- Implémenter des stratégies avancées
- Créer des runbooks opérationnels

---

## 🎓 Structure de Chaque Laboratoire

Chaque fichier de lab suit cette structure standardisée:

1. **En-tête**: Titre, durée, objectifs
2. **Prérequis**: Conditions pour commencer
3. **Concepts Clés**: Explications théoriques
4. **Instructions Étape par Étape**: Procédure détaillée
5. **Code/Exemples**: Fichiers YAML complets
6. **Résultats Attendus**: Critères de succès
7. **Livrables**: Éléments à soumettre
8. **Dépannage**: Solutions aux problèmes courants
9. **Points Clés**: À retenir absolument
10. **Étapes Suivantes**: Progression du parcours

---

## 📝 Évaluation et Certification

### Modèle d'Évaluation

| Composant | Poids | Critères |
|-----------|-------|----------|
| **Complétude des Labs** | 40% | Tous les 7 labs complétés |
| **Livrables Pratiques** | 30% | Screenshots, fichiers YAML |
| **Documentation** | 20% | Explications et runbooks |
| **Compréhension** | 10% | Q&A, démonstration |

### Critères de Réussite par LAB

- ✅ Objectifs atteints
- ✅ Tous les livrables fournis
- ✅ Code YAML valide et fonctionnel
- ✅ Screenshots de confirmation
- ✅ Documentation complète

---

## 🛠️ Outils et Ressources Nécessaires

### Logiciels Requis
- Visual Studio Code ou éditeur similaire
- Git CLI
- Azure CLI (optionnel)
- PowerShell ou Bash

### Comptes et Accès
- Compte Azure DevOps (gratuit)
- Organisation Azure DevOps
- Abonnement Azure (pour certains labs optionnels)

### Documentation de Référence
- [Azure DevOps Pipelines Documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines)
- [YAML Schema Reference](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema)
- [Azure DevOps Marketplace Tasks](https://marketplace.visualstudio.com/azuredevops)

---

## 🎯 Recommandations Pédagogiques

### Pour les Formateurs
- Parcourir chaque lab avant la formation
- Adapter les durées selon le niveau des participants
- Prévoir des sessions Q&R après chaque lab
- Encourager l'expérimentation supplémentaire

### Pour les Apprenants
- Prendre du temps pour comprendre chaque concept
- Expérimenter au-delà des instructions
- Documenter ses découvertes
- Partager avec le groupe
- Demander de l'aide si nécessaire

### Rythme Recommandé
- **1-2 labs par jour**: Formation intensive (5 jours)
- **1 lab par semaine**: Formation étalée (7 semaines)
- **Auto-formation**: À votre rythme

---

## 📞 Support et Ressources

### En cas de Problème
1. Consulter la section "Dépannage Courant" du lab
2. Revoir les prérequis
3. Consulter la documentation officielle Microsoft
4. Demander support à l'équipe de formation

### Ressources Supplémentaires
- Forums Azure DevOps Community
- Stack Overflow (tag: azure-devops)
- Microsoft Learn modules
- GitHub repositories d'exemples

---

## 🎁 Bonus: Projet Capstone

### Description
Implémenter un pipeline CI/CD complet pour une application réelle, intégrant tous les concepts des 7 labs.

### Durée
8 heures (peut être étalé)

### Livrables
- Pipeline YAML complet
- Documentation d'architecture
- Runbooks opérationnels
- Présentation de 30 minutes
- Démonstration en direct

### Évaluation
- Fonctionnalité: 30%
- Sécurité: 25%
- Bonnes pratiques: 20%
- Performance: 15%
- Présentation: 10%

---

## 📄 Version et Maintenance

**Version**: 1.0
**Date**: Décembre 2025
**Dernière mise à jour**: 2025-12-01

### Mises à Jour Futures
- Intégration des nouveaux templates Azure DevOps
- Ajout de labs sur le DevSecOps
- Intégration avancée avec Kubernetes
- Exemples avec différents langages

---

## 📌 Résumé Final

Cette formation Azure DevOps Pipelines est une progression pédagogique complète du **débutant au professionnel**, couvrant:

✅ Fondamentaux (LAB 1-2)
✅ Concepts avancés (LAB 3-5)
✅ Production-Ready (LAB 6-7)
✅ Projet intégré (Capstone)

**Tous les fichiers sont en français, structurés académiquement, et prêts pour une utilisation en formation professionnelle.**

---

## 📋 Fichiers Disponibles

1. `LAB-1-1-Premier-Pipeline.md`
2. `LAB-2-1-Pipeline-Multi-Etapes.md`
3. `LAB-3-1-Variables-Secrets-Parametres.md`
4. `LAB-4-1-Agents-On-Premises.md`
5. `LAB-5-1-Tasks-Templates.md`
6. `LAB-6-1-Securite-Permissions-Approbations.md`
7. `LAB-7-1-Deployments-Multi-Environnements.md`

**Tous les fichiers peuvent être téléchargés et utilisés comme matériel de formation professionnel.**

---

*Formation Azure DevOps Pipelines - Tous droits réservés - Utilisation pédagogique uniquement*
