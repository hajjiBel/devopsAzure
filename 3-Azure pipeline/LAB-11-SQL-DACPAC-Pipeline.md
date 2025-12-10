# Lab Azure Pipeline : CI/CD pour Base de Données SQL Server (DACPAC)

## Objectifs du Lab

- Créer un projet Azure DevOps pour une base de données SQL Server
- Configurer un pipeline CI (Continuous Integration) pour builder un projet SQL (.sqlproj)
- Générer un artefact DACPAC (Data-tier Application Package)
- Créer une ressource Azure SQL Database
- Configurer un pipeline CD (Continuous Deployment) en YAML pour déployer le schéma de base de données

---

## 1. Schéma des Étapes CI/CD (SQL Server / DACPAC)

```
┌─────────────────────────────────────────────────────────────────┐
│                     TRIGGER (Commit/PR)                         │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   STAGE 1: CI (Build)       │
        ├──────────────────────────────┤
        │  Job: Build Database Project │
        │  ├─ Checkout code            │
        │  ├─ Setup MSBuild / VSBuild  │
        │  ├─ Build Solution (.sln)    │
        │  │  ➔ Output: .dacpac file   │
        │  ├─ Publish Build Artifacts  │
        └──────────────┬───────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   STAGE 2: CD (Deploy)      │
        ├──────────────────────────────┤
        │  Job: Deploy to Azure SQL    │
        │  ├─ Download artifacts       │
        │  ├─ Deploy SQL DACPAC Task   │
        │  │  ➔ Updates Schema         │
        │  │  ➔ Updates Stored Procs   │
        │  └─ Verify Connection        │
        └──────────────────────────────┘
```

### Description du flux

**TRIGGER** : Le pipeline démarre automatiquement lors d'un commit sur la branche main.

**STAGE 1 - CI (Continuous Integration)** :
- **Checkout** : Récupération du code source (Projet SQL Server Data Tools)
- **Build** : Compilation de la solution .sln ou du projet .sqlproj via MSBuild
- **Output** : Génération d'un fichier DACPAC. Ce fichier contient la définition complète du schéma de la base de données
- **Publish** : Mise à disposition du fichier .dacpac comme artefact pour le déploiement

**STAGE 2 - CD (Continuous Deployment)** :
- **Deploy** : Utilisation de la tâche SqlAzureDacpacDeployment pour appliquer le schéma DACPAC sur l'Azure SQL Database cible. Cette tâche compare l'état actuel de la base avec le DACPAC et génère un script de mise à jour (delta) pour appliquer les changements

---

## 2. Prérequis et Configuration des Ressources

### 2.1 Créer un Projet Azure DevOps

1. Accédez à [Azure DevOps](https://dev.azure.com)
2. Créez un nouveau projet nommé : **SQLServer-Database-Demo**
3. Visibilité : **Private**

### 2.2 Préparer le Code Source (Azure Repos)

1. Dans Azure DevOps, allez dans **Repos**
2. Importez le repository suivant (contenant un projet SQL) :
   - URL : `https://github.com/HassenJRIDI/SqlServer-Database`
3. Cliquez sur **Import**

### 2.3 Créer les Ressources Azure (Azure SQL Database)

1. Accédez au [Portail Azure](https://portal.azure.com/)
2. Créez un **Groupe de ressources** : **TestRG**
3. Créez une ressource **Azure SQL Database** :
   - **Nom de la base** : Products
   - **Serveur** : Cliquez sur Créer un nouveau
     - **Nom** : sql-server-lab-[votre-nom] (doit être unique)
     - **Authentification** : Utiliser l'authentification SQL uniquement
     - **Login admin** : sqladmin
     - **Mot de passe** : P@ssw0rd1234 (ou autre mot de passe fort)
   - **Niveau de calcul** : Basic ou Standard S0 (pour minimiser les coûts)
   - **Réseau** : Dans l'onglet Réseau, cochez **"Autoriser les services et les ressources Azure à accéder à ce serveur"** (C'est CRITIQUE pour que le pipeline puisse déployer)
4. Cliquez sur **Créer**

---

## 3. Configuration YAML du Pipeline

Contrairement à l'approche "Éditeur Classique", nous allons utiliser une approche moderne YAML.

### 3.1 Créer le fichier azure-pipelines.yml

Créez un fichier `azure-pipelines.yml` à la racine de votre repo avec le contenu suivant :

```yaml
trigger:
- main

pool:
  vmImage: 'windows-latest'  # Windows est requis pour les projets SQL (DACPAC)

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'
  # Variables de déploiement (A modifier selon vos ressources)
  azureSubscription: 'ServiceConnection-Lab'
  sqlServerName: 'sql-server-lab-[votre-nom].database.windows.net'
  databaseName: 'Products'
  sqlAdminLogin: 'sqladmin'
  sqlAdminPassword: 'P@ssw0rd1234' # Idéalement à mettre dans les variables secrètes

stages:
# ========================================================
# STAGE 1: CI (BUILD DACPAC)
# ========================================================
- stage: CI
  displayName: 'Build Database Project'
  jobs:
  - job: Build
    displayName: 'Build DACPAC'
    steps:
    # 1. Restore NuGet packages (optionnel pour SQL mais bonne pratique)
    - task: NuGetToolInstaller@1

    - task: NuGetCommand@2
      inputs:
        restoreSolution: '$(solution)'

    # 2. Build Solution to generate DACPAC
    # L'argument /p:OutDir force la sortie dans le dossier de staging
    - task: VSBuild@1
      displayName: 'Build SQL Project'
      inputs:
        solution: '$(solution)'
        platform: '$(buildPlatform)'
        configuration: '$(buildConfiguration)'
        msbuildArgs: '/p:OutDir=$(Build.ArtifactStagingDirectory)'

    # 3. Publish Artifact (Le fichier .dacpac)
    - task: PublishBuildArtifacts@1
      displayName: 'Publish Artifact'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'

# ========================================================
# STAGE 2: CD (DEPLOY DACPAC)
# ========================================================
- stage: CD
  displayName: 'Deploy to Azure SQL'
  dependsOn: CI
  condition: succeeded()
  jobs:
  - deployment: DeployDB
    displayName: 'Deploy DACPAC'
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          # 1. Download Artifact
          - download: current
            artifact: 'drop'

          # 2. Deploy DACPAC to Azure SQL
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Deploy SQL DACPAC'
            inputs:
              azureSubscription: '$(azureSubscription)'
              AuthenticationType: 'server' # Utilisation login/mdp SQL
              ServerName: '$(sqlServerName)'
              DatabaseName: '$(databaseName)'
              SqlUsername: '$(sqlAdminLogin)'
              SqlPassword: '$(sqlAdminPassword)'
              deployType: 'DacpacTask'
              DeploymentAction: 'Publish'
              DacpacFile: '$(Pipeline.Workspace)/drop/**/*.dacpac'
              IpDetectionMethod: 'AutoDetect' # Ajoute temporairement l'IP de l'agent au firewall
```

---

## 4. Exécution et Validation

### 4.1 Créer la Connexion de Service

Avant de lancer, assurez-vous d'avoir une Service Connection valide vers votre abonnement Azure.

1. Allez dans **Project Settings > Service connections**
2. Cliquez sur **New service connection**
3. Sélectionnez **Azure Resource Manager > Service principal (automatic)**
4. Sélectionnez votre abonnement et le resource group **TestRG**
5. Nommez-la **ServiceConnection-Lab**

### 4.2 Lancer le Pipeline

1. Allez dans **Pipelines > New Pipeline**
2. Sélectionnez **Azure Repos Git**
3. Choisissez votre fichier YAML
4. Cliquez sur **Run**

### 4.3 Validation du Déploiement

1. Connectez-vous à votre base de données via **SQL Server Management Studio (SSMS)** ou l'éditeur de requête du Portail Azure
2. Vérifiez que les tables définies dans le projet GitHub (par exemple `Sales.Customer`, `Production.Product`) ont bien été créées
3. Si vous modifiez une table dans le code (ex: ajout d'une colonne dans un fichier .sql), commitez le changement : le pipeline se relancera et appliquera uniquement cette modification (ALTER TABLE)

---

## 5. Points Clés à Retenir

| Point | Description |
|-------|-------------|
| **DACPAC** | Archive contenant le schéma de la base de données (pas les données) |
| **Stage CI** | Compile le projet SQL et génère le fichier .dacpac |
| **Stage CD** | Déploie le DACPAC sur Azure SQL Database |
| **Service Connection** | Authentifie Azure DevOps vers votre souscription Azure |
| **IpDetectionMethod** | Permet au pipeline d'accéder au firewall d'Azure SQL |
| **Déploiement déclaratif** | Le DACPAC compare l'état actuel et applique les changements nécessaires |

---

## 6. Dépannage Courant

| Problème | Solution |
|---------|----------|
| **Erreur : "Service connection not found"** | Vérifier le nom de la Service Connection dans le YAML |
| **Erreur : "dacpac file not found"** | Vérifier que le chemin `$(Pipeline.Workspace)/drop/**/*.dacpac` est correct |
| **Erreur : "Firewall IP not authorized"** | Cocher "Autoriser les services Azure" dans les paramètres réseau de SQL Server |
| **Build échoue** | Vérifier que MSBuild est disponible sur l'agent Windows |
| **Déploiement lent** | Vérifier le niveau de calcul de la base (Basic peut être lent) |

---

## 7. Améliorations Futures

- ✅ Ajouter des étapes de test automatisé (tSQLt)
- ✅ Implémenter des approbations manuelles avant déploiement en Production
- ✅ Utiliser des variables secrètes pour les mots de passe
- ✅ Ajouter des notifications d'alerte en cas d'erreur
- ✅ Mettre en place des environnements multiples (Dev, Test, Prod)
