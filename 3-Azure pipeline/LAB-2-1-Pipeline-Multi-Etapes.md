# LAB 2.1: Implémenter un Pipeline Multi-Étapes


## Objectifs d'Apprentissage
À la fin de ce laboratoire, vous serez capable de:
- Créer une structure multi-étapes (Build, Test, Deploy)
- Implémenter des dépendances entre étapes
- Configurer l'exécution conditionnelle des étapes
- Définir des variables au niveau des étapes
- Gérer l'ordre d'exécution des jobs

## Prérequis
- Avoir complété LAB 1.1
- Compréhension de la structure hiérarchique YAML
- Familiarité avec l'indentation YAML (2 espaces obligatoires)

## Concepts Clés

### Hiérarchie des Pipelines
```
Pipeline
├── Stages (Étapes principales)
│   ├── Build
│   ├── Test
│   └── Deploy
├── Jobs (Tâches)
│   ├── BuildJob
│   ├── TestJob
│   └── DeployJob
└── Steps (Étapes élémentaires)
    ├── Script
    └── Task
```

### Définitions
- **Stage (Étape)**: Division logique majeure du pipeline (exemple: Build, Test, Deploy)
- **Job (Tâche)**: Collection d'étapes s'exécutant sur un agent
- **Step (Étape)**: Action élémentaire (script ou tâche prédéfinie)
- **Dépendance**: Relation entre étapes (une dépend du succès d'une autre)

## Instructions Étape par Étape

### Étape 1: Préparer le Fichier Pipeline
1. Ouvrez votre référentiel local dans Visual Studio Code
2. Ouvrez le fichier **.azure-pipelines/azure-pipelines.yml**
3. Sélectionnez tout le contenu et supprimez-le

### Étape 2: Créer la Structure Multi-Étapes
1. Tapez le code suivant dans le fichier:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  environmentName: 'Training'

stages:
- stage: Build
  displayName: 'Stage de Build'
  jobs:
  - job: BuildJob
    displayName: 'Tâche de Build'
    steps:
    - script: echo Initialisation du Build...
      displayName: 'Initialiser le Build'
    
    - script: echo Configuration $(buildConfiguration)
      displayName: 'Afficher la Configuration'
    
    - script: echo Build terminé avec succès!
      displayName: 'Compilation du Code'

- stage: Test
  displayName: 'Stage de Test'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - job: TestJob
    displayName: 'Tâche de Test'
    steps:
    - script: echo Exécution des tests unitaires...
      displayName: 'Tests Unitaires'
    
    - script: echo Exécution des tests d'intégration...
      displayName: 'Tests d''Intégration'
    
    - script: echo Tous les tests ont réussi!
      displayName: 'Vérification des Tests'

- stage: Deploy
  displayName: 'Stage de Déploiement'
  dependsOn: Test
  condition: succeeded()
  jobs:
  - job: DeployJob
    displayName: 'Tâche de Déploiement'
    steps:
    - script: echo Déploiement en cours...
      displayName: 'Déployer l''Application'
    #corriger
        -      script: echo Application déployée vers $(environmentName)
                                displayName: 'Vérifier le Déploiement'
```

**Explications clés**:
- `dependsOn: Build` - Cette étape attend que Build soit terminée
- `condition: succeeded()` - L'étape s'exécute seulement si l'étape précédente a réussi
- `variables` - Variables réutilisables dans tout le pipeline
- `displayName` - Nom affiché dans l'interface pour plus de clarté



### Étape 3: Valider la Syntaxe YAML
1. Vérifiez que chaque ligne commence avec le bon nombre d'espaces (2 espaces par niveau)
2. Vérifiez que les tirets `-` sont correctement alignés
3. Pas de tabulations (utiliser uniquement des espaces)

**Points d'attention**:
- Les `stage:` doivent être au même niveau d'indentation
- Les `jobs:` doivent être indentés sous `stage:`
- Les `steps:` doivent être indentés sous `jobs:`

### Étape 4: Commiter et Pousser
1. Ouvrez le terminal à la racine du projet
2. Vérifiez les modifications:
   ```bash
   git status
   ```
3. Ajoutez les fichiers modifiés:
   ```bash
   git add .azure-pipelines/azure-pipelines.yml
   ```
4. Créez un commit:
   ```bash
   git commit -m "Ajout du pipeline multi-étapes avec Build, Test, Deploy"
   ```
5. Poussez vers le serveur:
   ```bash
   git push origin main
   ```



### Étape 5: Observer l'Exécution du Pipeline
1. Retournez à Azure DevOps
2. Allez à **Pipelines** → Sélectionnez votre pipeline
3. Vous devriez voir une nouvelle exécution qui a démarré
4. Attendez que le pipeline se termine

**Ce que vous devez observer**:
- 3 étapes listées dans l'ordre: Build → Test → Deploy
- Chaque étape indiquant "Réussi"
- Progression des étapes dans le temps


### Étape 6: Analyser en Détail les Étapes
1. Cliquez sur **Build** dans la visualisation
2. Observez les 3 étapes du job BuildJob
3. Vérifiez que les variables sont affichées correctement

4. Cliquez sur **Test**
5. Vérifiez que cette étape a attendu que Build se termine
6. Observez les messages de test

7. Cliquez sur **Deploy**
8. Vérifiez que cette étape s'est exécutée en dernier
9. Observez l'affichage de la variable `environmentName`


### Étape 7: Tester les Conditions (Avancé)
1. Retournez au fichier **azure-pipelines.yml** localement
2. Modifiez une étape pour observer le comportement des conditions:

```yaml
- stage: Build
  displayName: 'Stage de Build'
  jobs:
  - job: BuildJob
    displayName: 'Tâche de Build'
    steps:
    - script: exit 1  # Cela va faire échouer l'étape
      displayName: 'Étape qui échoue'
      continueOnError: true
```

3. Poussez cette modification:
   ```bash
   git add .
   git commit -m "Test des conditions avec étape échouée"
   git push origin main
   ```

4. Observez que les étapes Test et Deploy ne s'exécutent pas (car Build a échoué)

5. Restaurez le fichier à son état précédent:
   ```bash
   git checkout .
   git push origin main
   ```


## Résultats Attendus

À la fin de ce laboratoire, vous devriez:
- ✅ Avoir un pipeline avec 3 étapes distinctes
- ✅ Avoir observé l'exécution séquentielle des étapes
- ✅ Avoir compris le concept de dépendances
- ✅ Avoir utilisé des variables au niveau du pipeline
- ✅ Avoir testé les conditions d'exécution

## Livrables

1. **Capture d'écran 1**: Vue générale du pipeline avec 3 étapes
2. **Capture d'écran 2**: Détails du stage Build avec ses 3 jobs
3. **Capture d'écran 3**: Détails du stage Test (montrant la dépendance)
4. **Capture d'écran 4**: Visualisation complète du pipeline exécuté
5. **Fichier**: Le fichier azure-pipelines.yml finalisé

## Concept Avancé: Variables par Étape

Vous pouvez aussi définir des variables spécifiques à une étape:

```yaml
- stage: Build
  displayName: 'Stage de Build'
  variables:
    buildConfiguration: 'Debug'  # Surcharge la variable globale
  jobs:
  - job: BuildJob
    steps:
    - script: echo Configuration $(buildConfiguration)
```

## Dépannage Courant

### L'étape Test ne s'exécute pas
- **Cause**: Build a échoué ou la condition n'est pas respectée
- **Solution**: Vérifiez les logs du stage Build

### Erreur "Indentation invalide"
- **Cause**: Utilisation de tabulations au lieu d'espaces
- **Solution**: Assurez-vous d'utiliser exactement 2 espaces pour chaque niveau

### Les variables ne s'affichent pas
- **Cause**: Syntaxe incorrecte `$(variableName)`
- **Solution**: Vérifiez que vous avez mis les parenthèses correctement

## Points Clés à Retenir

1. **Progression linéaire**: Les étapes s'exécutent dans l'ordre défini
2. **Dépendances**: Utilisez `dependsOn:` pour expliciter les relations
3. **Conditions**: Limitez l'exécution avec `condition: succeeded()`
4. **Nommage clair**: Les `displayName` améliorent la traçabilité
5. **Réutilisabilité**: Les variables économisent du code et améliorent la maintenance

## Étapes Suivantes

Une fois ce laboratoire terminé:
- LAB 3.1: Gérer les variables et les paramètres
- LAB 4.1: Configurer les agents on-premises
- Ajouter des tâches et des scripts plus complexes

---
