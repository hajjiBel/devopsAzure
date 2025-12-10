# LAB 5.1: Tâches Intégrées, Tasks et Templates Réutilisables

## Durée
3 heures

## Objectifs d'Apprentissage
À la fin de ce laboratoire, vous serez capable de:
- Comprendre les tâches prédéfinies (tasks) disponibles
- Ajouter et configurer les tasks dans le pipeline
- Créer des templates d'étapes réutilisables
- Créer des templates de jobs
- Créer des templates d'étapes complets
- Implémenter des templates paramétrés

## Prérequis
- Avoir complété LAB 4.1
- Compréhension des variables et des dépendances
- Connaissance de base du marketplace Azure DevOps

## Concepts Clés

### Types de Tasks
- **Tâches de Script**: Bash, PowerShell, Cmd
- **Tâches de Build**: Compilation, Maven, Gradle, MSBuild
- **Tâches de Déploiement**: App Service, Kubernetes, VM
- **Tâches de Test**: xUnit, Jest, Pytest
- **Tâches Utilitaires**: Opérations sur fichiers, REST API

### Structure des Templates
```
Templates
├── steps-templates/
│   └── build-steps.yml
├── jobs-templates/
│   └── build-job.yml
└── stages-templates/
    └── build-stage.yml
```

## Instructions Étape par Étape

### Étape 1: Explorez le Marketplace des Tasks
1. Dans votre pipeline YAML dans Azure DevOps
2. Cliquez sur l'onglet **Tasks** à gauche
3. Explorez les catégories:
   - **Utilitaires**: Fichiers, FTP, HTTP
   - **Build**: Compilateurs, frameworks
   - **Test**: Frameworks de test
   - **Déploiement**: Cloud, On-Prem
   - **Package**: NPM, NuGet, Maven

4. Recherchez **"Docker"**
5. Observez les tasks disponibles:
   - Docker CLI Installer
   - Docker Build
   - Docker Push

**Temps estimé**: 5 minutes

### Étape 2: Ajouter une Task Docker au Pipeline
1. Ouvrez votre référentiel local
2. Éditez le fichier **.azure-pipelines/azure-pipelines.yml**
3. Remplacez le contenu par:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: 'Stage de Build avec Tasks'
  jobs:
  - job: BuildJob
    displayName: 'Build avec Tasks Intégrées'
    steps:
    # Task 1: Utiliser une task prédéfinie
    - task: UseDotNet@2
      displayName: 'Installer .NET SDK'
      inputs:
        version: '6.x'
        packageType: sdk
    
    # Task 2: Afficher la version de .NET
    - script: dotnet --version
      displayName: 'Afficher la version de .NET'
    
    # Task 3: Restaurer les dépendances (exemple si vous aviez un projet)
    - task: DotNetCoreCLI@2
      displayName: 'Restaurer les Dépendances NuGet'
      inputs:
        command: 'restore'
        projects: '**/*.csproj' # Si vous aviez un projet C#
      continueOnError: true
    
    # Task 4: Utiliser une task Docker
    - task: DockerInstaller@0
      displayName: 'Installer Docker CLI'
      inputs:
        dockerVersion: 'latest'
    
    # Task 5: Vérifier Docker
    - script: docker --version
      displayName: 'Vérifier Installation Docker'

- stage: Test
  displayName: 'Stage de Test'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - job: TestJob
    displayName: 'Tests avec Tasks'
    steps:
    # Task de test
    - script: |
        echo "Exécution des tests unitaires..."
        echo "Tests réussis (simulation)"
      displayName: 'Exécuter Tests'
```

**Points importants**:
- `task:` - Utilise une task du marketplace
- `inputs:` - Paramètres de configuration de la task
- `continueOnError: true` - Continue même si l'étape échoue
- `displayName` - Nom affiché

4. Commitez et poussez:
   ```bash
   git add .azure-pipelines/azure-pipelines.yml
   git commit -m "Ajout de tasks prédéfinies (Docker, .NET)"
   git push origin main
   ```

**Temps estimé**: 10 minutes

### Étape 3: Créer un Template d'Étapes Réutilisable
1. Créez le dossier des templates:
   ```bash
   mkdir -p templates/steps
   ```

2. Créez le fichier **templates/steps/build-steps.yml**:
   ```yaml
   parameters:
     - name: buildConfiguration
       type: string
       default: 'Release'
     - name: buildPlatform
       type: string
       default: 'Any CPU'
   
   steps:
   - script: echo "Initialisation du build..."
     displayName: 'Étape 1: Initialiser'
   
   - script: |
       echo "Configuration: ${{ parameters.buildConfiguration }}"
       echo "Plateforme: ${{ parameters.buildPlatform }}"
     displayName: 'Étape 2: Configuration'
   
   - script: echo "Compilation en cours..."
     displayName: 'Étape 3: Compiler'
   
   - script: echo "Build terminé avec succès!"
     displayName: 'Étape 4: Finaliser'
   ```

3. Créez le fichier **templates/steps/test-steps.yml**:
   ```yaml
   parameters:
     - name: testFramework
       type: string
       default: 'unittest'
   
   steps:
   - script: echo "Préparation des tests..."
     displayName: 'Préparer Tests'
   
   - script: |
       echo "Framework de test: ${{ parameters.testFramework }}"
       echo "Exécution des tests..."
     displayName: 'Exécuter Tests'
   
   - script: echo "Rapport de couverture généré"
     displayName: 'Générer Rapport'
   ```

**Temps estimé**: 10 minutes

### Étape 4: Créer un Template de Job
1. Créez le dossier:
   ```bash
   mkdir -p templates/jobs
   ```

2. Créez le fichier **templates/jobs/build-job.yml**:
   ```yaml
   parameters:
     - name: jobName
       type: string
       default: 'DefaultJob'
     - name: displayName
       type: string
       default: 'Build Job'
     - name: configuration
       type: string
       default: 'Release'
     - name: platform
       type: string
       default: 'Any CPU'
   
   jobs:
   - job: ${{ parameters.jobName }}
     displayName: ${{ parameters.displayName }}
     steps:
     - template: ../steps/build-steps.yml
       parameters:
         buildConfiguration: ${{ parameters.configuration }}
         buildPlatform: ${{ parameters.platform }}
   ```

**Temps estimé**: 5 minutes

### Étape 5: Créer un Template d'Étape Complète
1. Créez le fichier **templates/stages/build-stage.yml**:
   ```yaml
   parameters:
     - name: stageName
       type: string
       default: 'Build'
     - name: displayName
       type: string
       default: 'Build Stage'
     - name: dependsOn
       type: string
       default: ''
     - name: condition
       type: string
       default: ''
   
   stages:
   - stage: ${{ parameters.stageName }}
     displayName: ${{ parameters.displayName }}
     ${{ if parameters.dependsOn }}:
       dependsOn: ${{ parameters.dependsOn }}
     ${{ if parameters.condition }}:
       condition: ${{ parameters.condition }}
     jobs:
     - template: ../jobs/build-job.yml
       parameters:
         jobName: '${{ parameters.stageName }}Job'
         displayName: '${{ parameters.displayName }} - Job'
   ```

**Temps estimé**: 5 minutes

### Étape 6: Utiliser les Templates dans le Pipeline Principal
1. Remplacez le contenu de **azure-pipelines.yml** par:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

# Déclaration des groupes de variables
variables:
- name: buildConfiguration
  value: 'Release'
- name: buildPlatform
  value: 'Any CPU'

stages:
# Stage 1: Build utilisant le template
- template: templates/stages/build-stage.yml
  parameters:
    stageName: 'Build'
    displayName: 'Build Stage (Template)'
    condition: succeeded()

# Stage 2: Test utilisant le template
- template: templates/stages/build-stage.yml
  parameters:
    stageName: 'Test'
    displayName: 'Test Stage (Template)'
    dependsOn: 'Build'
    condition: succeeded()

# Stage 3: Deploy utilisant le template
- template: templates/stages/build-stage.yml
  parameters:
    stageName: 'Deploy'
    displayName: 'Deploy Stage (Template)'
    dependsOn: 'Test'
    condition: succeeded()
```

**Temps estimé**: 5 minutes

### Étape 7: Améliorer les Templates avec des Conditions
1. Créez un template avancé **templates/jobs/conditional-job.yml**:
   ```yaml
   parameters:
     - name: jobName
       type: string
     - name: displayName
       type: string
     - name: runTests
       type: boolean
       default: true
     - name: environment
       type: string
       default: 'dev'
   
   jobs:
   - job: ${{ parameters.jobName }}
     displayName: ${{ parameters.displayName }}
     steps:
     - script: echo "Job exécuté pour l'environnement: ${{ parameters.environment }}"
       displayName: 'Afficher Environnement'
     
     # Étapes conditionnelles
     - ${{ if parameters.runTests }}:
       - script: echo "Exécution des tests..."
         displayName: 'Tests Conditionnels'
       - script: echo "Tests réussis!"
         displayName: 'Rapport de Tests'
     
     # Condition basée sur l'environnement
     - ${{ if eq(parameters.environment, 'prod') }}:
       - script: echo "Déploiement en PRODUCTION"
         displayName: 'Déploiement Production'
     
     - ${{ if ne(parameters.environment, 'prod') }}:
       - script: echo "Déploiement en ${{ parameters.environment }}"
         displayName: 'Déploiement Développement'
   ```

2. Utilisez ce template dans le pipeline:
   ```yaml
   stages:
   - stage: DeployTest
     displayName: 'Deploy avec Conditions'
     jobs:
     - template: templates/jobs/conditional-job.yml
       parameters:
         jobName: 'DevDeploy'
         displayName: 'Deploy Dev'
         runTests: true
         environment: 'dev'
     
     - template: templates/jobs/conditional-job.yml
       parameters:
         jobName: 'ProdDeploy'
         displayName: 'Deploy Production'
         runTests: true
         environment: 'prod'
   ```

**Temps estimé**: 10 minutes

### Étape 8: Valider et Tester
1. Créez la structure de répertoires:
   ```bash
   mkdir -p templates/steps
   mkdir -p templates/jobs
   mkdir -p templates/stages
   ```

2. Commitez tous les fichiers:
   ```bash
   git add .
   git commit -m "Ajout des templates d'étapes, jobs et stages réutilisables"
   git push origin main
   ```

3. Attendez l'exécution du pipeline
4. Vérifiez que tous les stages s'exécutent correctement
5. Observez que les templates sont utilisés correctement

**Temps estimé**: 5 minutes

### Étape 9: Bonnes Pratiques des Templates
1. Documentez vos templates avec des commentaires:
   ```yaml
   # Template de Build Réutilisable
   # Paramètres:
   #   - buildConfiguration: Configuration de build (Debug/Release)
   #   - buildPlatform: Plateforme cible (Any CPU/x64)
   parameters:
     - name: buildConfiguration
       type: string
       default: 'Release'
   ```

2. Versionnez vos templates:
   ```bash
   git tag -a "templates-v1.0" -m "Version initiale des templates"
   git push origin --tags
   ```

**Temps estimé**: 5 minutes

## Résultats Attendus

À la fin de ce laboratoire, vous devriez:
- ✅ Comprendre les tasks prédéfinies disponibles
- ✅ Avoir créé des templates d'étapes réutilisables
- ✅ Avoir créé des templates de jobs
- ✅ Avoir créé des templates d'étapes complets
- ✅ Avoir utilisé les templates dans le pipeline
- ✅ Avoir implémenté des conditions dans les templates

## Livrables

1. **Capture d'écran 1**: Tasks disponibles dans le marketplace
2. **Capture d'écran 2**: Pipeline avec tasks intégrées en exécution
3. **Fichiers templates**: Tous les fichiers YAML des templates
4. **Capture d'écran 3**: Pipeline utilisant les templates en exécution
5. **Capture d'écran 4**: Logs montrant l'utilisation des templates

## Tableau des Tasks Utiles

| Catégorie | Task | Utilité |
|-----------|------|---------|
| **Build** | UseDotNet | Installer .NET SDK |
| **Build** | DockerInstaller | Installer Docker CLI |
| **Deploy** | AzureWebApp | Déployer vers App Service |
| **Deploy** | KubernetesManifest | Déployer vers Kubernetes |
| **Test** | PublishTestResults | Publier résultats de tests |
| **Utilitaire** | PublishBuildArtifacts | Publier des artifacts |
| **Utilitaire** | DownloadBuildArtifacts | Télécharger des artifacts |

## Exemples de Templates Avancés

### Template avec Matrices
```yaml
parameters:
  - name: nodeVersions
    type: object
    default:
      - 14.x
      - 16.x
      - 18.x

jobs:
- ${{ each version in parameters.nodeVersions }}:
  - job: TestNode_${{ replace(version, '.', '_') }}
    displayName: 'Test Node.js ${{ version }}'
    steps:
    - script: echo "Testing with Node ${{ version }}"
```

### Template avec Variables d'Environnement
```yaml
parameters:
  - name: environmentVariables
    type: object

jobs:
- job: JobWithEnvVars
  displayName: 'Job avec Variables'
  steps:
  - script: |
      ${{ each var in parameters.environmentVariables }}:
        echo "${{ var.key }}=${{ var.value }}"
```

## Dépannage Courant

### Le template n'est pas trouvé
- **Cause**: Chemin incorrect dans `template:`
- **Solution**: Vérifiez le chemin relatif (par rapport au fichier courant)

### Erreur de paramètre non défini
- **Cause**: Paramètre requis non fourni
- **Solution**: Vérifiez la définition du paramètre et sa transmission

### Les conditions ne fonctionnent pas
- **Cause**: Syntaxe `${{ if }}` incorrecte
- **Solution**: Utilisez `succeeded()`, `failed()` ou comparaisons

## Points Clés à Retenir

1. **Tasks du Marketplace**: Simplifient les tâches courantes
2. **Templates Réutilisables**: Réduisent la duplication
3. **Paramètres**: Rendent les templates flexibles
4. **Conditions**: Contrôlent l'exécution conditionnelle
5. **Documentation**: Essentielles pour la maintenance

## Étapes Suivantes

Une fois ce laboratoire terminé:
- LAB 6.1: Configurer les permissions et portes d'approbation
- LAB 7.1: Implémenter les déploiements en environnements multiples
- Explorer d'autres tasks du marketplace

---

**Durée totale du laboratoire**: ~55 minutes
**Heure de lab supplémentaire**: Pour explorer d'autres tasks et créer des templates personnalisés avancés
