# Lab Académique : Déployer Node.js sur Azure DevOps

## Objectifs du Lab

À la fin de ce lab, tu seras capable de :
- Créer une application Node.js simple
- Configurer un repo Git dans Azure DevOps
- Construire un pipeline CI/CD YAML
- Déployer l'app sur Azure App Service
- Valider le déploiement en accédant à l'application

---

## Prérequis

- Compte Azure (avec abonnement actif)
- Compte Azure DevOps
- Git installé localement
- Node.js installé (v16 ou plus)
- CLI Azure installé (optionnel mais recommandé)

---

## Partie 1 : Créer l'application Node.js

### 1.1 Initialiser le projet

```bash
mkdir my-nodejs-app
cd my-nodejs-app
npm init -y
```

### 1.2 Installer Express

```bash
npm install express
```

### 1.3 Créer le fichier app.js

Crée un fichier `app.js` à la racine du projet :

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello from Node.js on Azure DevOps! 🚀');
});

app.get('/api/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date() });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

### 1.4 Mettre à jour package.json

Modifie le script `start` dans `package.json` :

```json
{
  "name": "my-nodejs-app",
  "version": "1.0.0",
  "description": "Simple Node.js app on Azure DevOps",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"Error: no test specified\" && exit 0"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### 1.5 Créer .gitignore

```
node_modules/
.env
*.log
.DS_Store
```

### 1.6 Tester localement

```bash
npm start
```

Visite `http://localhost:3000` dans ton navigateur. Tu devrais voir le message de bienvenue.

---

## Partie 2 : Initialiser le repo Azure DevOps

### 2.1 Créer un projet Azure DevOps

1. Va sur [dev.azure.com](https://dev.azure.com)
2. Clique sur **+ New project**
3. Donne un nom (ex: `nodejs-lab`)
4. Sélectionne **Git** comme système de contrôle de version
5. Clique **Create**

### 2.2 Initialiser Git localement

```bash
git init
git add .
git commit -m "Initial commit: Node.js app"
```

### 2.3 Connecter au repo Azure DevOps

Dans Azure DevOps, dans l'onglet **Repos**, tu verras les commandes. Exécute :

```bash
git remote add origin https://dev.azure.com/[organization]/[project]/_git/[repository]
git branch -M main
git push -u origin main
```

---

## Partie 3 : Créer l'infrastructure Azure (App Service)

### 3.1 Créer un Resource Group

```bash
az group create --name rg-nodejs-lab --location eastus
```

### 3.2 Créer un App Service Plan

```bash
az appservice plan create \
  --name plan-nodejs-lab \
  --resource-group rg-nodejs-lab \
  --sku B1 \
  --is-linux
```

### 3.3 Créer une Web App

```bash
az webapp create \
  --resource-group rg-nodejs-lab \
  --plan plan-nodejs-lab \
  --name app-nodejs-lab-[random] \
  --runtime "node|18-lts"
```

**Note :** Remplace `[random]` par quelque chose d'unique (ex: ton nom + date).

---

## Partie 4 : Créer le Pipeline CI/CD YAML

### 4.1 Créer le fichier azure-pipelines.yml

À la racine du projet, crée un fichier `azure-pipelines.yml` :

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  nodeVersion: '18.x'
  buildConfiguration: 'Release'

stages:
  - stage: Build
    displayName: 'Build Stage'
    jobs:
      - job: BuildJob
        displayName: 'Build Node.js App'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)
            displayName: 'Install Node.js'

          - script: |
              npm install
            displayName: 'Install Dependencies'

          - script: |
              npm test
            displayName: 'Run Tests'

          - task: ArchiveFiles@2
            inputs:
              rootFolderOrFile: '$(Build.SourcesDirectory)'
              includeRootFolder: false
              archiveType: 'zip'
              archiveFile: '$(Build.ArtifactStagingDirectory)/app-$(Build.BuildId).zip'
              replaceExistingArchive: true
            displayName: 'Create Artifact'

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'nodejs-app'
            displayName: 'Publish Artifact'

  - stage: Deploy
    displayName: 'Deploy Stage'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployToAppService
        displayName: 'Deploy to App Service'
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadBuildArtifacts@1
                  inputs:
                    buildType: 'current'
                    downloadType: 'single'
                    artifactName: 'nodejs-app'
                    downloadPath: '$(Pipeline.Workspace)'
                  displayName: 'Download Artifact'

                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'Azure Connection'
                    appType: 'webAppLinux'
                    appName: 'app-nodejs-lab-[random]'
                    package: '$(Pipeline.Workspace)/nodejs-app/app-*.zip'
                  displayName: 'Deploy to Azure App Service'
```

**Important :** Remplace `app-nodejs-lab-[random]` et `Azure Connection` avec tes vraies valeurs.

### 4.2 Committer et pousser

```bash
git add azure-pipelines.yml
git commit -m "Add Azure DevOps pipeline"
git push
```

---

## Partie 5 : Configurer la connexion Azure dans DevOps

### 5.1 Créer une Service Connection

1. Dans Azure DevOps, va dans **Project Settings** → **Service connections**
2. Clique **New service connection** → **Azure Resource Manager**
3. Sélectionne **Service Principal (automatic)**
4. Remplis les infos de ton abonnement Azure
5. Donne un nom (ex: `Azure Connection`)
6. Clique **Save**

### 5.2 Vérifier les permissions

Assure-toi que le Service Principal a les rôles :
- **Contributor** sur le Resource Group `rg-nodejs-lab`

---

## Partie 6 : Créer et exécuter le Pipeline

### 6.1 Créer le pipeline dans Azure DevOps

1. Va dans **Pipelines** → **Create Pipeline**
2. Sélectionne **Azure Repos Git**
3. Sélectionne ton repo `nodejs-lab`
4. Choisis **Existing Azure Pipelines YAML file**
5. Sélectionne `azure-pipelines.yml` depuis la branche `main`
6. Clique **Continue** puis **Save and run**

### 6.2 Attendre le résultat

Le pipeline va :
1. **Build** : Installer les dépendances, lancer les tests, créer un artifact ZIP
2. **Deploy** : Télécharger l'artifact et le déployer sur App Service

Tu peux voir la progression en temps réel dans l'interface de Azure DevOps.

---

## Partie 7 : Valider le déploiement

### 7.1 Accéder à l'application

Une fois le pipeline vert, va sur :

```
https://app-nodejs-lab-[random].azurewebsites.net
```

Tu devrais voir : **Hello from Node.js on Azure DevOps! 🚀**

### 7.2 Tester l'endpoint health

```
https://app-nodejs-lab-[random].azurewebsites.net/api/health
```

Tu devrais voir :
```json
{
  "status": "healthy",
  "timestamp": "2025-12-11T13:17:00.000Z"
}
```

---

## Points clés expliqués

### Trigger
```yaml
trigger:
  - main
```
Le pipeline se lance automatiquement quand tu pushes sur la branche `main`.

### Pool
```yaml
pool:
  vmImage: 'ubuntu-latest'
```
Les jobs s'exécutent sur une VM Linux hébergée par Microsoft.

### Stages & Jobs
- **Build stage** : Compile et teste
- **Deploy stage** : Déploie sur Azure (runOnce = une seule vague, pas de canary/rolling)

### runOnce
```yaml
strategy:
  runOnce:
```
Le déploiement se fait une seule fois sur l'environnement, sans phases progressives.





---

## Troubleshooting

| Problème | Solution |
|----------|----------|
| Pipeline échoue à la connexion Azure | Vérifie que le Service Connection existe et a les bonnes permissions |
| App Service ne démarre pas | Check les logs : `az webapp log tail --resource-group rg-nodejs-lab --name app-nodejs-lab-[random]` |
| Port ne correspond pas | Azure utilise la variable `PORT`. Express l'écoute avec `process.env.PORT \|\| 3000` ✓ |
| Artifact non trouvé au déploiement | Vérifie que le nom du ZIP dans la task de déploiement correspond au pattern du build |

---

## Résumé du workflow

```
Code local (app.js, package.json)
    ↓
Git push vers Azure DevOps
    ↓
Pipeline YAML déclenché (trigger: main)
    ↓
Stage Build : npm install, npm test, créer ZIP
    ↓
Publish artifact
    ↓
Stage Deploy : Télécharger ZIP, déployer sur App Service
    ↓
App accessible via HTTPS sur Azure
```

---

**Fin du lab. Bravo! 🎉**