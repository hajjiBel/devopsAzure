# Lab Azure Pipeline : CI/CD pour Application .NET Core (ASP.NET)

## Schéma des Étapes CI/CD (.NET Core)

```
┌─────────────────────────────────────────────────────────────────┐
│                     TRIGGER (Commit/PR)                         │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   STAGE 1: CI (Build)       │
        ├──────────────────────────────┤
        │  Job: Build & Test           │
        │  ├─ Checkout code            │
        │  ├─ Setup .NET SDK           │
        │  ├─ Restore dependencies     │
        │  ├─ Build application        │
        │  ├─ Run unit tests           │
        │  ├─ Publish artefact (ZIP)   │
        │  └─ Publish build artifacts  │
        └──────────────┬───────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   STAGE 2: CD (Deploy)      │
        ├──────────────────────────────┤
        │  Job: Deploy to App Service  │
        │  ├─ Download artifacts       │
        │  ├─ Deploy via AzureWebApp   │
        │  └─ Run smoke tests          │
        └──────────────────────────────┘
```

## Objectifs du Lab

- Créer un projet Azure DevOps pour une application ASP.NET Core
- Configurer un pipeline CI (Continuous Integration) avec .NET CLI (remplaçant Maven)
- Créer une ressource Azure App Service (Web App) via le portail
- Configurer un pipeline CD (Continuous Deployment) en YAML
- Déployer automatiquement l'application compilée vers Azure

---

## Description du flux

**TRIGGER** : Le pipeline démarre automatiquement lors d'un commit sur la branche main.

**STAGE 1 - CI (Continuous Integration)** :
- **Checkout** : Récupération du code source.
- **Restore** : Restauration des paquets NuGet (équivalent `mvn restore`).
- **Build** : Compilation en mode Release (équivalent `mvn compile`).
- **Test** : Exécution des tests unitaires (équivalent `mvn test`).
- **Publish** : Génération des binaires et packaging en ZIP.

**STAGE 2 - CD (Continuous Deployment)** :
- **Deploy** : Envoi du package ZIP vers l'Azure App Service (Web App).
- **Smoke Test** : Vérification simple que l'application répond (HTTP 200).

---

## 1. Prérequis et Configuration des Ressources

### 1.1 Créer un Projet Azure DevOps

1. Accédez à [Azure DevOps](https://dev.azure.com/)
2. Cliquez sur **Create project**
3. Remplissez les champs :
   - **Nom** : `DotNet-AspNetCore-Lab`
   - **Visibilité** : Private
4. Cliquez sur **Create**

### 1.2 Préparer le Code Source (Azure Repos)

Vous pouvez importer un projet existant ou en créer un nouveau.

#### Option A : Importer un repo exemple (Recommandé)

1. Dans Azure DevOps, allez dans **Repos > Import**
2. URL de clonage : `https://github.com/microsoftlearning/mslearn-aspnet-core` (ou tout autre projet .NET Core simple)
3. Cliquez sur **Import**

#### Option B : Créer un projet simple via Cloud Shell

```bash
dotnet new webapp -n MyDotNetApp
dotnet new xunit -n MyDotNetApp.Tests
# (Ajouter ensuite le code dans votre repo Azure)
```

### 1.3 Créer la Web App Azure (Manuellement)

1. Accédez au [Portail Azure](https://portal.azure.com/)
2. Recherchez **App Services** et cliquez sur **+ Créer**
3. Remplissez les informations suivantes :
   - **Abonnement** : Votre abonnement
   - **Groupe de ressources** : Créer nouveau `rg-dotnet-lab`
   - **Nom** : `lab-dotnet-[votre-nom]` (doit être unique)
   - **Publier** : Code
   - **Pile d'exécution (Runtime)** : .NET 8 (LTS)
   - **Système d'exploitation** : Linux
   - **Région** : West Europe (Europe)
   - **Plan App Service** : Sélectionnez ou créez un plan (F1 Gratuit est suffisant)
4. Cliquez sur **Vérifier + créer** puis **Créer**
5. Une fois créée, notez le **nom exact** de votre Web App

---

## 2. Configuration YAML du Pipeline

Créez un fichier nommé **`azure-pipelines.yml`** à la racine de votre repository avec le contenu suivant.

**⚠️ Important** : Mettez à jour la variable `webAppName` avec le nom de la ressource créée à l'étape 1.3.

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  # Nom exact de votre Web App créée dans le portail Azure
  webAppName: 'lab-dotnet-votre-nom' 
  # Nom de votre Service Connection dans Azure DevOps
  azureSubscription: 'ServiceConnection-Lab' 

stages:
  # ========================================================
  # STAGE 1: CI (BUILD & TEST)
  # ========================================================
  - stage: CI
    displayName: 'Build & Test (.NET)'
    jobs:
      - job: Build
        displayName: 'Build Job'
        steps:
          # 1. Installer le SDK .NET
          - task: UseDotNet@2
            displayName: 'Install .NET 8 SDK'
            inputs:
              version: '8.x'
              performMultiLevelLookup: true

          # 2. Restaurer les dépendances (Equivalent Maven Restore)
          - task: DotNetCoreCLI@2
            displayName: 'Restore dependencies'
            inputs:
              command: 'restore'
              projects: '**/*.csproj'

          # 3. Compiler l'application (Equivalent Maven Compile)
          - task: DotNetCoreCLI@2
            displayName: 'Build App'
            inputs:
              command: 'build'
              projects: '**/*.csproj'
              arguments: '--configuration $(buildConfiguration) --no-restore'

          # 4. Exécuter les tests (Equivalent Maven Test)
          # Note: Cette étape peut échouer si aucun projet de test n'existe. 
          # Commentez-la si vous utilisez un projet sans tests.
          - task: DotNetCoreCLI@2
            displayName: 'Run Tests'
            inputs:
              command: 'test'
              projects: '**/*[Tt]ests/*.csproj'
              arguments: '--configuration $(buildConfiguration) --no-build'

          # 5. Publier l'application (Packaging)
          - task: DotNetCoreCLI@2
            displayName: 'Publish App'
            inputs:
              command: 'publish'
              publishWebProjects: true
              arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
              zipAfterPublish: true

          # 6. Publier l'artefact pour le stage suivant
          - task: PublishBuildArtifacts@1
            displayName: 'Upload Artifacts'
            inputs:
              PathtoPublish: '$(Build.ArtifactStagingDirectory)'
              ArtifactName: 'drop'
              publishLocation: 'Container'

  # ========================================================
  # STAGE 2: CD (DEPLOY)
  # ========================================================
  - stage: CD
    displayName: 'Deploy to Azure'
    dependsOn: CI
    condition: succeeded()
    jobs:
      - deployment: DeployWeb
        displayName: 'Deploy Web App'
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                # 1. Télécharger l'artefact
                - download: current
                  artifact: 'drop'
                
                # 2. Déployer vers Azure App Service
                - task: AzureWebApp@1
                  displayName: 'Azure Web App Deploy'
                  inputs:
                    azureSubscription: '$(azureSubscription)'
                    appType: 'webAppLinux'
                    appName: '$(webAppName)'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
                    runtimeStack: 'DOTNETCORE|8.0'

                # 3. Smoke Test (Vérification)
                - script: |
                    echo "Testing URL: https://$(webAppName).azurewebsites.net"
                    sleep 10
                    STATUS_CODE=$(curl -o /dev/null -s -w "%{http_code}\n" https://$(webAppName).azurewebsites.net)
                    echo "Status Code: $STATUS_CODE"
                    if [ "$STATUS_CODE" -eq 200 ] || [ "$STATUS_CODE" -eq 302 ]; then
                        echo "✅ Deployment Success"
                        exit 0
                    else
                        echo "⚠️ Warning: Application returned $STATUS_CODE"
                        # On ne fail pas le build ici pour le lab, mais en prod on pourrait faire 'exit 1'
                    fi
                  displayName: 'Smoke Test'
```

---

## 3. Exécution et Validation

### 3.1 Lancer le Pipeline

1. Dans Azure DevOps, allez dans **Pipelines**
2. Cliquez sur **New Pipeline**
3. Sélectionnez **Azure Repos Git > Votre Repository**
4. Choisissez **Existing Azure Pipelines YAML file** et sélectionnez le fichier créé
5. Cliquez sur **Save and Run**

### 3.2 Suivi

1. Cliquez sur le **Run** en cours
2. Observez la complétion du **stage CI**
3. Observez le démarrage et la complétion du **stage CD**

### 3.3 Validation Finale

1. Une fois le déploiement terminé (vert), allez sur le portail Azure ou utilisez votre navigateur
2. Ouvrez l'URL : `https://lab-dotnet-[votre-nom].azurewebsites.net`
3. Vous devriez voir la page d'accueil de votre application ASP.NET Core ("Welcome", "Hello World", etc.)

---

## 4. Tableau d'équivalence (Java/Maven vs .NET)

Pour vous aider à comprendre les étapes par rapport à vos labs précédents en Java :

| Étape | Commande Maven (Java) | Commande .NET CLI | Tâche YAML Azure |
|-------|----------------------|-------------------|------------------|
| Dépendances | `mvn restore` | `dotnet restore` | `DotNetCoreCLI@2` (restore) |
| Compilation | `mvn compile` | `dotnet build` | `DotNetCoreCLI@2` (build) |
| Tests | `mvn test` | `dotnet test` | `DotNetCoreCLI@2` (test) |
| Packaging | `mvn package` (.jar/.war) | `dotnet publish` (.zip) | `DotNetCoreCLI@2` (publish) |
| Déploiement | Azure App Service Deploy | Azure App Service Deploy | `AzureWebApp@1` |

---

## 5. Résolution de problèmes courants

### Le pipeline échoue à la restauration des dépendances

- **Cause** : Fichier `.csproj` manquant ou chemin incorrect
- **Solution** : Vérifiez que vos fichiers `.csproj` sont à la racine ou mettez à jour le chemin dans le YAML

### Le déploiement échoue avec erreur d'authentification

- **Cause** : Service Connection non configurée
- **Solution** : 
  1. Allez dans **Project Settings > Service Connections**
  2. Créez une nouvelle connection **Azure Resource Manager**
  3. Remplacez `azureSubscription` par le nom exact de votre connection

### Le Smoke Test échoue (Status Code 500 ou 502)

- **Cause** : L'application n'a pas démarré correctement
- **Solution** :
  1. Vérifiez les logs via le portail Azure (App Service > Log stream)
  2. Assurez-vous que le runtime .NET 8 est configuré
  3. Attendez 20-30 secondes après le déploiement avant de tester

### Aucun projet de test trouvé

- **Cause** : Votre structure de projet n'inclut pas de tests
- **Solution** : Commentez l'étape "Run Tests" dans le YAML ou créez un projet de test XUnit

---

## 6. Prochaines étapes

- Configurer les **notifications de pipeline** (Teams, Email)
- Ajouter des **étapes de sécurité** (SonarQube, dependency scanning)
- Implémenter un **pipeline de test automatisé** plus complet
- Configurer les **environnements multiples** (Staging, Production)
- Mettre en place des **approvals** avant le déploiement en production
