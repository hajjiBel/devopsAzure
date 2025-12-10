# Index des Laboratoires Azure DevOps Pipelines 

## 📚 Structure du Programme de Formation

Ce programme de formation est composé de **13 laboratoires indépendants** couvrant l'ensemble du cycle d'apprentissage Azure DevOps Pipelines, du **débutant au professionnel**, incluant des applications pratiques et l'orchestration Kubernetes.

---

## 📋 Fichiers de Laboratoires Créés

### LAB 1.1: Créer Votre Premier Pipeline Azure DevOps
**Fichier**: `LAB-1-1-Premier-Pipeline.md`
- **Durée**: 2 heures
- **Niveau**: Débutant
- **Objectifs**: Mise en place initiale, création d'un pipeline YAML de base
- **Contenu**: Configuration du projet, déclenchement du pipeline, lecture des logs
- **Prérequis**: Compte Azure DevOps, Git
- **Livrables**: Pipeline fonctionnel, screenshots d'exécution

---

### LAB 2.1: Implémenter un Pipeline Multi-Étapes
**Fichier**: `LAB-2-1-Pipeline-Multi-Etapes.md`
- **Durée**: 3 heures
- **Niveau**: Débutant
- **Objectifs**: Structure hiérarchique, dépendances, exécution conditionnelle
- **Contenu**: Stages, Jobs, Steps, variables, conditions
- **Prérequis**: LAB 1.1 complété
- **Livrables**: Pipeline Build → Test → Deploy, documentation des dépendances

---

### LAB 3.1: Gestion des Variables, Paramètres et Secrets
**Fichier**: `LAB-3-1-Variables-Secrets-Parametres.md`
- **Durée**: 2.5 heures
- **Niveau**: Intermédiaire
- **Objectifs**: Gestion sécurisée des variables, groupes de variables, permissions
- **Contenu**: Variables de pipeline, groupes de variables, variables secrètes, portée
- **Prérequis**: LAB 2.1 complété
- **Livrables**: Groupes de variables configurés, secrets masqués, documentation de sécurité

---

### LAB 4.1: Configuration et Gestion des Agents On-Premises
**Fichier**: `LAB-4-1-Agents-On-Premises.md`
- **Durée**: 3 heures
- **Niveau**: Intermédiaire
- **Objectifs**: Déploiement d'agents auto-hébergés, gestion des pools
- **Contenu**: Création de pools, PAT, installation agents (Windows/Linux), capacités
- **Prérequis**: LAB 3.1 complété, accès administrateur
- **Livrables**: Agent on-premises enregistré, pipeline exécuté sur agent personnalisé

---

### LAB 5.1: Tâches Intégrées, Tasks et Templates Réutilisables
**Fichier**: `LAB-5-1-Tasks-Templates.md`
- **Durée**: 3 heures
- **Niveau**: Intermédiaire
- **Objectifs**: Tasks du marketplace, templates réutilisables, paramétrisation
- **Contenu**: Tasks prédéfinies, création de templates (steps, jobs, stages), conditions
- **Prérequis**: LAB 4.1 complété
- **Livrables**: Templates réutilisables, pipeline modulaire, documentation des templates

---

### LAB 6.1: Sécurité, Permissions et Portes d'Approbation
**Fichier**: `LAB-6-1-Securite-Permissions-Approbations.md`
- **Durée**: 2.5 heures
- **Niveau**: Intermédiaire
- **Objectifs**: Modèle de sécurité, approvals, contrôle d'accès
- **Contenu**: Environnements, portes d'approbation, restrictions de branche, audit
- **Prérequis**: LAB 5.1 complété
- **Livrables**: Pipeline avec approvals en production, audit trail, documentation des permissions

---

### LAB 7.1: Déploiements Multi-Environnements et Stratégies Avancées
**Fichier**: `LAB-7-1-Deployments-Multi-Environnements.md`
- **Durée**: 4 heures
- **Niveau**: Avancé
- **Objectifs**: Stratégies de déploiement, blue-green, canary, rollback
- **Contenu**: Multi-environnements, stratégies avancées, health checks, runbooks
- **Prérequis**: LAB 6.1 complété
- **Livrables**: Pipeline production-ready, runbooks de rollback, documentation complète

---

### LAB 8: Déploiement d'Application Node.js (CI/CD Complet)
**Fichier**: `LAB-8.-Deploiement-d-Application-node-js.md`
- **Durée**: 2-3 heures
- **Niveau**: Intermédiaire
- **Objectifs**: Mettre en pratique CI/CD pour application Node.js en production
- **Contenu**: Pipeline CI (build, tests, artefacts), Pipeline CD (création App Service, déploiement, smoke tests)
- **Prérequis**: LAB 2.1 complété, notions de Node.js
- **Livrables**: Application Node.js déployée sur Azure App Service, pipeline CI/CD fonctionnel

---

### LAB 10: Pipeline CI/CD pour Application .NET Core / ASP.NET
**Fichier**: `LAB-10-Azure-Pipeline-DOTNET.md`
- **Durée**: 2-3 heures
- **Niveau**: Intermédiaire
- **Objectifs**: Implémenter CI/CD pour framework .NET moderne
- **Contenu**: .NET CLI, restauration NuGet, build Release, tests unitaires, déploiement App Service
- **Prérequis**: LAB 2.1 complété, notions de .NET Core
- **Livrables**: Application ASP.NET Core déployée, pipeline complet avec smoke tests

---

### LAB 11: Pipeline CI/CD pour Base de Données SQL Server (DACPAC)
**Fichier**: `LAB-11-SQL-DACPAC-Pipeline.md`
- **Durée**: 3 heures
- **Niveau**: Avancé
- **Objectifs**: Automatiser les déploiements de schéma de base de données SQL Server
- **Contenu**: Build SQL .sqlproj, génération DACPAC, déploiement Azure SQL Database déclaratif
- **Prérequis**: LAB 2.1 complété, notions SQL Server
- **Livrables**: Pipeline CI/CD pour base de données, DACPAC généré et déployé

---

### LAB 12: Déploiement sur Azure Kubernetes Service (AKS)
**Fichier**: `12-Lab_Azure_AKS.md`
- **Durée**: 4-5 heures
- **Niveau**: Avancé
- **Objectifs**: Conteneuriser et déployer sur Kubernetes via Azure DevOps
- **Contenu**: 
  - Build image Docker et push vers Azure Container Registry (ACR)
  - Déploiement sur cluster AKS
  - Manifestes Kubernetes (Deployment, Service, HPA)
  - Autoscaling horizontal (HPA)
- **Prérequis**: LAB 7.1 complété, notions Docker/Kubernetes
- **Livrables**: Image Docker en ACR, application déployée sur AKS, HPA configuré

---

### LAB 13: Gestion des Secrets avec Azure Key Vault
**Fichier**: `LAB-13_Azure_KeyVault.md`
- **Durée**: 2-3 heures
- **Niveau**: Avancé
- **Objectifs**: Sécuriser les secrets dans les pipelines CICD
- **Contenu**: 
  - Création Azure Key Vault
  - Permissions Service Principal
  - Récupération secrets dans pipeline
  - Variable Groups avec Key Vault
- **Prérequis**: LAB 3.1 complété, recommandé LAB 11 (pour contexte SQL)
- **Livrables**: Pipeline sécurisé avec gestion des secrets, Key Vault intégré

---
