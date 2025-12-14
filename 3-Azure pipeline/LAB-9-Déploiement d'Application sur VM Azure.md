# Lab Azure Pipeline : Déploiement d'Application sur VM Azure

## Objectifs du Lab

- Créer un pipeline Azure avec étapes CI (Continuous Integration) et CD (Continuous Deployment)
- Comprendre l'architecture et le flux de travail CI/CD
- Configurer les déclencheurs et les étapes de déploiement
- Déployer une application sur une Machine Virtuelle Azure via un Deployment Group

***

## 1. Schéma des Étapes CI/CD (Déploiement sur VM)

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
        │  ├─ Install dependencies     │
        │  ├─ Build application        │
        │  ├─ Run unit tests           │
        │  └─ Publish artifacts        │
        └──────────────┬───────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   STAGE 2: CD (Deploy)      │
        ├──────────────────────────────┤
        │  Job: Deploy to VM           │
        │  ├─ Download artifacts       │
        │  ├─ Deploy via Deployment    │
        │  │  Group (SSH/WinRM)        │
        │  └─ Run smoke tests          │
        └──────────────────────────────┘
```


### Description du flux

**TRIGGER** : Le pipeline démarre automatiquement lors d'un commit sur la branche `main`

**STAGE 1 - CI (Continuous Integration)** :

- Checkout du code source
- Installation des dépendances (npm, pip, etc.)
- Compilation/Build de l'application
- Exécution des tests unitaires
- Publication des artefacts (ZIP)

**STAGE 2 - CD (Continuous Deployment)** :

- Téléchargement de l'artefact ZIP
- Déploiement sur la VM via le **Deployment Group** (environnement d'exécution)
- Exécution des scripts sur la VM (extraction, installation, démarrage)
- Validation via smoke tests (vérification de l'application)

***

## 2. Configuration YAML du Pipeline

### 2.1 Structure de Base

Créez un fichier `azure-pipelines.yml` à la racine de votre repository :

```yaml

trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  artifactName: 'app-artifact'

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

          # Étape 2: Installer Node.js
          - task: NodeTool@0
            displayName: 'Install Node.js'
            inputs:
              versionSpec: '18.x'

          # Étape 3: Installer les dépendances
          - script: npm install
            displayName: 'Install dependencies'

          # Étape 4: Tests unitaires
          - script: npm test
            displayName: 'Run unit tests'
            continueOnError: true

          # Étape 5: Archiver l'application
          - task: ArchiveFiles@2
            displayName: 'Archive application files'
            inputs:
              rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
              includeRootFolder: false
              archiveType: 'zip'
              archiveFile: '$(Build.ArtifactStagingDirectory)/$(artifactName).zip'
              replaceExistingArchive: true

          # Étape 6: Publier les artefacts
          - task: PublishBuildArtifacts@1
            displayName: 'Publish artifacts'
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'

  - stage: CD
    displayName: 'Continuous Deployment'
    dependsOn: CI
    condition: succeeded()
    jobs:
      - deployment: DeployToVM
        displayName: 'Deploy to Virtual Machine'
        environment:
          name: 'prod-vm'
          resourceType: 'VirtualMachine'
          tags: 'web'
        strategy:
          runOnce:
            deploy:
              steps:
                # Étape 1: Télécharger les artefacts
                - download: current
                  artifact: 'drop'
                  displayName: 'Download artifacts'

                # Étape 2: Debug - Afficher le contenu du workspace
                - bash: |
                    echo "========== WORKSPACE DEBUG =========="
                    echo "Pipeline.Workspace: $(Pipeline.Workspace)"
                    echo ""
                    echo "Contenu de $(Pipeline.Workspace)/drop/:"
                    ls -la $(Pipeline.Workspace)/drop/
                    echo ""
                    echo "Contenu du ZIP:"
                    unzip -l $(Pipeline.Workspace)/drop/$(artifactName).zip | head -20
                    echo "========================================="
                  displayName: 'Debug workspace'

                # Étape 3: Nettoyer et préparer le répertoire
                - bash: |
                    echo "Cleaning deployment directory..."
                    rm -rf /home/azureuser/app
                    mkdir -p /home/azureuser/app
                    cd /home/azureuser/app
                    echo "✅ Directory prepared: $(pwd)"
                  displayName: 'Prepare directory'

                # Étape 4: Extraire le ZIP - VERSION CORRIGÉE
                - bash: |
                    echo "Extracting artifact..."
                    cd /home/azureuser/app

                    # Extraire le ZIP
                    unzip -q $(Pipeline.Workspace)/drop/$(artifactName).zip

                    # Vérifier la structure
                    echo "Contenu après extraction:"
                    ls -la

                    # SI les fichiers sont dans un sous-dossier 's', les remonter
                    if [ -d "s" ] && [ -f "s/package.json" ]; then
                        echo "⚠️ Fichiers trouvés dans le dossier 's/', remontée..."
                        mv s/* .
                        rm -rf s
                        echo "✅ Fichiers remontés"
                    fi

                    # Vérifier que package.json existe
                    if [ ! -f "package.json" ]; then
                        echo "❌ ERREUR: package.json non trouvé!"
                        echo "Contenu du répertoire:"
                        ls -la
                        exit 1
                    fi

                    echo "✅ Extraction réussie"
                    echo "Fichiers présents:"
                    ls -la
                  displayName: 'Extract and fix artifact'

                # Étape 5: Installer les dépendances
                - bash: |
                    echo "Installing dependencies..."
                    cd /home/azureuser/app

                    # Vérifier qu'on est au bon endroit
                    if [ ! -f "package.json" ]; then
                        echo "❌ ERREUR: package.json absent!"
                        pwd
                        ls -la
                        exit 1
                    fi

                    npm install --production
                    echo "✅ Dependencies installed"
                  displayName: 'Install dependencies'

                # Étape 6: Arrêter l'ancienne application
                - bash: |
                    echo "Stopping old application..."
                    pkill -f "node" || echo "No running process found"
                    sleep 3
                    echo "✅ Old application stopped"
                  displayName: 'Stop old application'

                # Étape 7: Démarrer l'application
                - bash: |
                    echo "Starting application..."
                    cd /home/azureuser/app

                    export NODE_ENV=production
                    export PORT=3000

                    nohup npm start > /tmp/app.log 2>&1 &
                    sleep 5

                    if pgrep -f "node" > /dev/null; then
                        echo "✅ Application started"
                        ps aux | grep node | grep -v grep
                    else
                        echo "❌ Failed to start application"
                        cat /tmp/app.log
                        exit 1
                    fi
                  displayName: 'Start application'

                # Étape 8: Health check avec retry
                - bash: |
                    echo "Running health check..."

                    for i in {1..10}; do
                        echo "Attempt $i/10..."

                        STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/health 2>/dev/null || echo "000")

                        if [ "$STATUS" == "200" ]; then
                            echo "✅ Health check PASSED (HTTP $STATUS)"
                            curl -s http://localhost:3000/health | head -20
                            exit 0
                        fi

                        echo "⏳ HTTP $STATUS - Retrying in 5s..."
                        sleep 5
                    done

                    echo "❌ Health check FAILED after 10 attempts"
                    echo ""
                    echo "Application logs:"
                    tail -50 /tmp/app.log
                    exit 1
                  displayName: 'Health check'

                # Étape 9: Résumé du déploiement
                - bash: |
                    echo ""
                    echo "╔════════════════════════════════════════╗"
                    echo "║     DEPLOYMENT SUCCESSFUL! ✅          ║"
                    echo "╚════════════════════════════════════════╝"
                    echo ""
                    echo "📍 Application Directory: /home/azureuser/app"
                    echo "🏠 Host: $(hostname)"
                    echo "🌐 IP: $(hostname -I | awk '{print $1}')"
                    echo ""
                    echo "📊 Running Process:"
                    ps aux | grep node | grep -v grep
                    echo ""
                    echo "📡 Listening on:"
                    netstat -tulpn 2>/dev/null | grep :3000 || echo "Checking ports..."
                    echo ""
                    echo "🔗 Access URL:"
                    echo "   http://$(hostname -I | awk '{print $1}'):3000"
                    echo ""
                    echo "📝 Recent logs:"
                    tail -20 /tmp/app.log
                    echo ""
                  displayName: 'Deployment summary'


```


### Explication des sections principales

| Section | Description |
| :-- | :-- |
| **trigger** | Le pipeline se déclenche automatiquement sur les commits à la branche `main` |
| **pool** | Utilise un agent Linux (ubuntu-latest) pour exécuter les tâches de build |
| **variables** | Définit les constantes réutilisables dans le pipeline |
| **stages** | Divise le pipeline en deux étapes logiques : CI et CD |
| **deployment job** | Déploiement spécialisé qui s'exécute sur les VMs du Deployment Group |
| **environment** | Référence l'environnement "prod-vm" avec les VMs enregistrées |
| **tags** | Filtre les VMs par tags (ex: 'web') pour cibler seulement certaines VMs |


***

## 3. Configuration de l'Environnement VM dans Azure DevOps

### 3.1 Créer la Machine Virtuelle Azure

#### Étape 1 : Créer la VM via le Portail Azure

1. Accédez au **Portail Azure** → **Machines virtuelles**
2. Cliquez sur **+ Créer**
3. Configurez :

```
Abonnement : Votre abonnement
Groupe de ressources : rg-pipeline-vm-lab
Nom : vm-app-deploy
Région : West Europe
Image : Ubuntu 22.04 LTS
Taille : Standard_B2s (2 vCPU, 4 GB RAM)
Authentification : Clé publique SSH ou Mot de passe
Nom d'utilisateur : azureuser
```

4. **Configurer le disque** : Gardez les valeurs par défaut
5. **Configurer la mise en réseau** :
    - Créer un groupe de sécurité réseau (NSG)
    - Ajouter une règle d'entrée :
        - Port : 3000 (ou le port de votre application)
        - Source : Internet
        - Action : Autoriser
    - Ajouter une règle pour SSH (port 22)
6. Cliquez sur **Vérifier + créer** → **Créer**

#### Étape 2 : Attendre le déploiement

Attendez que la VM soit déployée (2-3 minutes). Une fois prête :

- Notez l'**Adresse IP publique** : `20.x.x.x`
- Notez le **Nom de la VM** : `vm-app-deploy`


### 3.2 Créer l'Environnement dans Azure DevOps

1. Accédez à votre projet Azure DevOps
2. Allez à **Pipelines** → **Environments**
3. Cliquez sur **Create environment**
4. Configurez :

```
Nom : prod-vm
Ressource : Virtual Machine
```

5. Cliquez sur **Create**

### 3.3 Enregistrer la VM auprès du Deployment Group

1. Sur la page d'environnement `prod-vm`, cliquez sur **Add resource**
2. Sélectionnez **Virtual Machine**
3. Choisissez **Linux**
4. Un script d'enregistrement s'affiche :
```bash
#!/bin/bash

# Exemple de script généré (à adapter)
mkdir ~/myagent && cd ~/myagent
wget https://vstsagentpackage.azureedge.net/agent/3.x.x/vsts-agent-linux-x64-3.x.x.tar.gz
tar zxvf vsts-agent-linux-x64-3.x.x.tar.gz
./config.sh --unattended \
  --agent "MyVM" \
  --url "https://dev.azure.com/YOUR_ORG" \
  --auth pat \
  --token YOUR_PAT_TOKEN \
  --pool "Default" \
  --work _work
sudo ./svc.sh install azureuser
sudo ./svc.sh start
```


#### Exécuter le script sur la VM

1. **Se connecter à la VM via SSH** :

```bash
ssh -i /chemin/vers/clé.pem azureuser@20.x.x.x
```

2. **Copier-coller le script d'enregistrement**
3. **Exécuter le script** :

```bash
chmod +x config.sh
./config.sh --unattended ...
```

4. **Vérifier l'enregistrement** :
    - Retourner à Azure DevOps
    - La VM doit apparaître dans l'environnement `prod-vm`
    - Ajouter un **tag** : `web` (pour filtrer les VMs dans le pipeline)

***

## 4. Étapes d'Exécution

### 4.1 Prérequis

- ✅ VM Azure créée et accessible via SSH
- ✅ Agent Azure DevOps enregistré sur la VM
- ✅ Environnement `prod-vm` créé dans Azure DevOps
- ✅ Service Connection Azure fonctionnel


### 4.2 Créer le Pipeline dans Azure DevOps

1. Accédez à **Pipelines** → **New Pipeline**
2. Sélectionnez votre repository
3. Choisissez **Existing Azure Pipelines YAML file**
4. Sélectionnez le fichier `azure-pipelines.yml`
5. Cliquez sur **Continue** puis **Save and Run**

### 4.3 Déclencher le Pipeline

Le pipeline se déclenche automatiquement lors d'un commit sur `main` :

```bash
git add azure-pipelines.yml
git commit -m "Add CI/CD pipeline with VM deployment"
git push origin main
```


### 4.4 Suivre l'Exécution

1. Accédez à **Pipelines** → **Runs**
2. Cliquez sur le run en cours
3. Visualisez les stages CI et CD
4. Dans le stage CD, vérifiez que le job `DeployToVM` s'exécute sur votre VM

### 4.5 Vérifier le Déploiement sur la VM

Après l'exécution réussie du pipeline :

```bash
# Se connecter à la VM
ssh -i /chemin/vers/clé.pem azureuser@20.x.x.x

# Vérifier le contenu du répertoire de déploiement
ls -la /home/azureuser/app

# Vérifier l'état du processus Node
ps aux | grep node

# Vérifier les logs de l'application
tail -f /tmp/app.log

# Tester l'application
curl http://localhost:3000
```


***

## 5. Monitoring et Troubleshooting

### 5.1 Erreurs Courantes

| Erreur | Cause | Solution |
| :-- | :-- | :-- |
| `Agent offline` | Agent Azure DevOps non enregistré | Réenregistrer l'agent via le script |
| `Permission denied` | Droits insuffisants sur les répertoires | Vérifier les permissions avec `chmod` |
| `Port already in use` | Application déjà en cours d'exécution | Arrêter le processus précédent : `pkill -f node` |
| `Artifact not found` | Artefact non publié par le CI | Vérifier les logs du stage CI |
| `Connection timeout` | Groupe de sécurité réseau (NSG) bloque le port | Ajouter une règle d'entrée dans le NSG |

### 5.2 Vérifier les Logs

Dans Azure DevOps :

1. Accédez au run du pipeline
2. Cliquez sur le stage **CD**
3. Cliquez sur le job **DeployToVM**
4. Consultez les logs détaillés de chaque étape

### 5.3 Tester la Connectivité SSH

```bash
# Depuis la machine locale
ssh -vvv -i /chemin/vers/clé.pem azureuser@20.x.x.x

# Sur la VM, vérifier les répertoires créés
ls -la /home/azureuser/app
```


***

## 6. Améliorations Optionnelles

### 6.1 Utiliser PM2 pour la Gestion des Processus

Au lieu de `npm start`, utiliser PM2 pour une meilleure résilience :

```bash
# Installer PM2 sur la VM
npm install -g pm2

# Démarrer l'application avec PM2
pm2 start /home/azureuser/app/server.js --name "my-app"

# Configurer le démarrage automatique
pm2 startup
pm2 save
```


### 6.2 Ajouter des Approvals Manuelles

Avant le déploiement en production :

```yaml
- deployment: DeployToVM
  displayName: 'Deploy to Production VM'
  environment:
    name: 'prod-vm'
    resourceType: 'VirtualMachine'
  strategy:
    ...
```

Configurez les approvals dans l'environnement :

- Allez à **Environments** → **prod-vm** → **Approvals and checks** → **Add approval**


### 6.3 Déploiement sur Plusieurs VMs

Utilisez des **tags** pour déployer sur plusieurs VMs :

```yaml
tags: 'web,prod'  # Déployer sur toutes les VMs avec ces tags
```


***

## 7. Points de Contrôle Qualité

| Étape | Critère de succès | Validation |
| :-- | :-- | :-- |
| VM créée | VM visible dans Portail Azure | ✅ Status : "En cours d'exécution" |
| Agent enregistré | VM apparaît dans l'environnement | ✅ Environments → prod-vm → Targets |
| Pipeline créé | Fichier YAML poussé | ✅ Visible dans Azure DevOps |
| Stage CI | Build et tests passants | ✅ Artefact ZIP généré |
| Stage CD | Déploiement sur VM réussi | ✅ Logs : "Artifact extracted successfully" |
| Application active | Processus Node en cours d'exécution | ✅ `ps aux \| grep node` affiche le processus |
| Smoke test | HTTP 200 retourné | ✅ ✅ Health check passed |


***

## 8. Ressources Supplémentaires

- [Documentation Azure Pipelines - Deploying to VMs](https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/deploy-linux-vm)
- [Azure VM Deployment Groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/deployment-group-phases)
- [YAML Schema - Deployment Jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/jobs-deployment)
- [Secure Shell (SSH) Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/ssh)

***

**Fin du Lab**
