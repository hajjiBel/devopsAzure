# Lab Azure Pipeline : Déploiement d'Application avec CI/CD

## Objectifs du Lab

- Créer un pipeline Azure avec étapes CI (Continuous Integration) et CD (Continuous Deployment)
- Comprendre l'architecture et le flux de travail CI/CD
- Configurer les déclencheurs et les étapes de déploiement
- Mettre en place des approvals pour les environnements sensibles

---

## 1. Schéma des Étapes CI/CD

┌─────────────────────────────────────────────────────────────────┐
│                     TRIGGER (Commit/PR)                         │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   STAGE 1: CI (Build)       │
        ├──────────────────────────────┤
        │  Job: Build & Test           │
        │  ├─ Checkout code            │
        │  ├─ Install dependencies     │
        │  ├─ Build application        │
        │  ├─ Run unit tests           │
        │  └─ Publish artifacts        │
        └──────────────┬───────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   STAGE 2: CD (Deploy)      │
        ├──────────────────────────────┤
        │  Job 1: Create App Service   │
        │  ├─ Créer Web App            │
        │  ├─ Configurer runtime       │
        │  └─ Générer URL déploiement  │
        └──────────────┬───────────────┘
                       │
        ┌──────────────▼──────────────┐
        │  Job 2: Deploy to App Svc   │
        │  ├─ Download artifacts       │
        │  ├─ Deploy via AzureWebApp   │
        │  └─ Run smoke tests          │
        └──────────────────────────────┘

### Description du flux

**TRIGGER** : Le pipeline démarre automatiquement lors d'un commit sur la branche `main`

**STAGE 1 - CI (Continuous Integration)** :
- Checkout du code source
- Installation des dépendances (npm, pip, etc.)
- Compilation/Build de l'application
- Exécution des tests unitaires
- Publication des artefacts (ZIP)

**STAGE 2 - CD (Continuous Deployment)** :
- **Job 1** : Création dynamique d'une Azure App Service Web App (si elle n'existe pas)
- **Job 2** : Déploiement de l'artefact ZIP vers la Web App créée
- Validation via smoke tests (vérification de l'endpoint `/health`)

---

## 2. Configuration YAML du Pipeline

### 2.1 Structure de Base

Créez un fichier `azure-pipelines.yml` à la racine de votre repository :

# Trigger: Déclenche le pipeline sur les commits vers main
trigger:
  - main

# Pool: Définit l'agent de build
pool:
  vmImage: 'ubuntu-latest'

# Variables globales
variables:
  buildConfiguration: 'Release'
  artifactName: 'app-artifact'
  azureSubscription: 'ServiceConnection-Lab'
  webAppName: 'my-pipeline-lab-$(Build.BuildId)'
  resourceGroupName: 'rg-pipeline-lab'

# Stages: Étapes du pipeline
stages:
  - stage: CI
    displayName: 'Continuous Integration'
    jobs:
      - job: Build
        displayName: 'Build & Test'
        steps:
          # Étape 1: Récupérer le code
          - checkout: self
            displayName: 'Checkout code'

          # Étape 2: Installer les dépendances
          - task: NodeTool@0
            displayName: 'Install Node.js'
            inputs:
              versionSpec: '18.x'
          
          - script: npm install
            displayName: 'Install dependencies'

          # Étape 3: Build
          - script: npm run build
            displayName: 'Build application'
          
          # Étape 4: Tests unitaires
          - script: npm run test
            displayName: 'Run unit tests'
          
          # Étape 5: Publier les artefacts
          - task: PublishBuildArtifacts@1
            displayName: 'Publish artifacts'
            inputs:
              pathToPublish: 'dist'
              artifactName: $(artifactName)

  - stage: CD
    displayName: 'Continuous Deployment'
    dependsOn: CI
    condition: succeeded()
    jobs:
      # Job 1: Déployer vers App Service
      - deployment: DeployToAppService
        displayName: 'Deploy to Azure App Service'
        environment: 'dev'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadBuildArtifacts@1
                  displayName: 'Download artifacts'
                  inputs:
                    buildType: 'current'
                    downloadPath: '$(Pipeline.Workspace)'

                - task: AzureWebApp@1
                  displayName: 'Deploy to Web App'
                  inputs:
                    azureSubscription: '$(azureSubscription)'
                    appType: 'webAppLinux'
                    appName: '$(webAppName)'
                    package: '$(Pipeline.Workspace)/drop/$(artifactName).zip'
                    startupCommand: 'npm start'

                - script: |
                    echo "Deployment completed successfully!"
                    echo "Application URL: https://$(webAppName).azurewebsites.net"
                  displayName: 'Deployment summary'

                - script: |
                    sleep 10
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://$(webAppName).azurewebsites.net)
                    if [ $STATUS -eq 200 ]; then
                      echo "✅ Health check passed: HTTP $STATUS"
                    else
                      echo "⚠️ Health check returned: HTTP $STATUS"
                    fi
                  displayName: 'Health check (Smoke test)'

### Explication des sections principales

| Section | Description |
|---------|-------------|
| **trigger** | Le pipeline se déclenche automatiquement sur les commits à la branche `main` |
| **pool** | Utilise un agent Linux (ubuntu-latest) pour exécuter les tâches |
| **variables** | Définit les constantes réutilisables dans le pipeline |
| **stages** | Divise le pipeline en deux étapes logiques : CI et CD |
| **jobs** | Groupe de tâches exécutées séquentiellement ou en parallèle |
| **steps** | Tâches individuelles (scripts, tâches Azure, etc.) |

---

## 3. Composants Clés du Pipeline

### 3.1 Trigger
trigger:
  - main
  # Optionnel: configurer les branches et les chemins
  paths:
    include:
      - src/**
      - azure-pipelines.yml
Le trigger déclenche le pipeline lors d'un commit sur `main` ou sur les fichiers spécifiés.

### 3.2 Pool (Agent de Build)
pool:
  vmImage: 'ubuntu-latest'  # Linux
  # vmImage: 'windows-latest'  # Windows
  # vmImage: 'macos-latest'   # macOS
Le pool définit l'environnement d'exécution des tâches.

### 3.3 Variables
variables:
  buildConfiguration: 'Release'
  azureSubscription: 'ServiceConnection-Lab'
  webAppName: 'my-pipeline-lab-$(Build.BuildId)'
  # Variables secrètes (créées dans l'interface Azure DevOps)
  # - deploymentKey: $(SECRET_KEY)
Les variables réutilisables dans le pipeline. Les variables secrètes sont définies dans les paramètres Azure DevOps.

### 3.4 Stages
Deux étapes logiques :
- **CI Stage** : Compilation, tests unitaires, publication des artefacts
- **CD Stage** : Création de l'App Service et déploiement vers Azure

### 3.5 Jobs et Steps
- **Job** : Ensemble de tâches exécutées séquentiellement
- **Step** : Tâche individuelle (script, task Azure CLI, task Azure Web App)

#### Exemple de dépendance entre jobs
- job: CreateAppService
  displayName: 'Create Azure App Service'

- deployment: DeployToAppService
  dependsOn: CreateAppService  # S'exécute après CreateAppService

### 3.6 Tâches principales du CD

| Tâche | Objectif | Exemple |
|-------|----------|---------|
| `AzureCLI@2` | Exécuter des commandes Azure CLI | Créer un groupe de ressources |
| `AzureWebApp@1` | Déployer vers Azure App Service | Déployer l'artefact ZIP |
| `DownloadBuildArtifacts@1` | Télécharger l'artefact du CI | Récupérer le fichier ZIP |

---

## 4. Étapes d'Exécution

### 4.1 Créer la Web App manuellement via le Portail Azure

#### Étape 1 : Créer le Groupe de Ressources

1. Accédez au **Portail Azure** → **Groupes de ressources**
2. Cliquez sur **+ Créer**
3. Configurez :
   - **Abonnement** : Votre abonnement
   - **Nom** : `rg-pipeline-lab`
   - **Région** : `West Europe`
4. Cliquez sur **Vérifier + créer** → **Créer**

#### Étape 2 : Créer le Plan App Service

1. Portail Azure → **App Service Plans**
2. Cliquez sur **+ Créer**
3. Configurez :
   - **Ressource** : `plan-lab-[votrenom]`
   - **Groupe de ressources** : `rg-pipeline-lab`
   - **Système d'exploitation** : Linux
   - **Région** : `West Europe`
   - **Niveau de tarification** : F1 (gratuit)
4. Cliquez sur **Créer**

#### Étape 3 : Créer la Web App

1. Portail Azure → **App Services**
2. Cliquez sur **+ Créer**
3. Configurez :
   - **Nom** : `my-pipeline-lab-[votrenom]` (doit être unique)
   - **Publier** : Code
   - **Runtime** : Node 18 LTS
   - **Groupe de ressources** : `rg-pipeline-lab`
   - **Plan App Service** : `plan-lab-[votrenom]`
4. Cliquez sur **Créer**
5. **Attendre 2-3 minutes** que la Web App soit déployée
6. Notez l'URL : `https://my-pipeline-lab-[votrenom].azurewebsites.net`

### 4.2 Créer le Pipeline dans Azure DevOps

1. Accédez à **Pipelines > New Pipeline**
2. Sélectionnez votre repository
3. Choisissez **Existing Azure Pipelines YAML file**
4. Sélectionnez le fichier `azure-pipelines.yml`
5. Cliquez sur **Continue** puis **Save and Run**

### 4.3 Configurer l'Environment dans Azure DevOps

1. Accédez à **Pipelines > Environments**
2. Cliquez sur **Create environment**
3. Créez un environment nommé **dev**
4. Cliquez sur **Create**

### 4.4 Mettre à jour les Variables du Pipeline

1. Accédez à votre pipeline
2. Cliquez sur **Edit**
3. Allez à **Variables** et ajoutez/modifiez :
   - `webAppName` = `my-pipeline-lab-[votrenom]` (nom exact de votre Web App)
   - `resourceGroupName` = `rg-pipeline-lab`
   - `azureSubscription` = `ServiceConnection-Lab`

### 4.5 Déclencher le Pipeline

Le pipeline se déclenche automatiquement lors d'un commit sur la branche `main` :

git add azure-pipelines.yml
git commit -m "Add CI/CD pipeline"
git push origin main

Le pipeline démarre automatiquement et vous pouvez suivre son avancement dans **Pipelines > Runs**.

### 4.6 Vérifier le déploiement

Une fois le pipeline réussi, vérifiez le déploiement :

# Accéder à l'URL de l'application
https://my-pipeline-lab-[votrenom].azurewebsites.net

# Ou via curl
curl https://my-pipeline-lab-[votrenom].azurewebsites.net

**Résultat attendu** : La page de l'application s'affiche (ou page par défaut Azure App Service si première déploiement)

---

## 5. Monitoring et Troubleshooting

### 5.1 Visualiser l'Exécution

1. Accédez à **Pipelines > Runs**
2. Cliquez sur le run pour voir les détails
3. Visualisez les logs pour chaque étape
4. Vérifiez les artefacts produits

### 5.2 Résoudre les Erreurs

| Erreur | Cause | Solution |
|--------|-------|----------|
| Build failed | Dépendances manquantes | Vérifier `npm install` |
| Tests failed | Code défaillant | Vérifier les logs des tests |
| Deploy failed | Permissions insuffisantes | Vérifier les credentials |
| Approval timeout | Absence d'approbateur | Assigner les approbateurs |

### 5.3 Vérifier les Logs

Cliquez sur chaque job pour consulter les logs détaillés de chaque étape.

---

## 6. Bonnes Pratiques

✅ **À Faire** :
- Utiliser des variables secrètes pour les credentials
- Diviser le pipeline en étapes logiques
- Ajouter des tests automatisés à chaque étape
- Configurer les approvals pour les environnements sensibles
- Monitorer les logs et les erreurs

❌ **À Éviter** :
- Hardcoder les secrets dans le YAML
- Déployer directement en production sans tests
- Créer des pipelines trop complexes
- Ignorer les alertes et les erreurs

---

## 6. Points de contrôle qualité

| Étape | Critère de succès | Validation |
|-------|------------------|-----------|
| Web App créée | Web App visible dans le portail Azure | ✅ Portail Azure → App Services |
| Pipeline créé | Fichier `azure-pipelines.yml` pushé | ✅ Visible dans Azure DevOps |
| Stage CI | Build et tests passants | ✅ Artefact ZIP généré |
| Déploiement | Application en ligne sur Web App | ✅ Accès à `https://my-pipeline-lab-[votrenom].azurewebsites.net` |
| Smoke test | Health check réussi | ✅ HTTP 200 retourné |

## 7. Ressources Supplémentaires

- [Documentation Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/what-is-azure-pipelines)
- [YAML Schema Reference](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema)
- [Azure App Service Deployment](https://learn.microsoft.com/en-us/azure/app-service/deploy-azure-pipelines)
- [Azure CLI - Web App Management](https://learn.microsoft.com/en-us/cli/azure/webapp)

---

**Fin du Lab**

Durée estimée : **45-60 minutes**  
Niveau : Intermédiaire  
Prérequis : Compte Azure, Azure DevOps, Git