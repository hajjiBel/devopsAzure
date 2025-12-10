# LAB 1.1: Créer Votre Premier Pipeline Azure DevOps

## Durée


## Objectifs d'Apprentissage
À la fin de ce laboratoire, vous serez capable de:
- Configurer un projet Azure DevOps
- Créer un référentiel Git
- Définir un pipeline YAML de base
- Déclencher l'exécution automatique du pipeline
- Interpréter les résultats d'exécution

## Prérequis
- Compte Azure DevOps actif (https://dev.azure.com)
- Accès à une organisation Azure DevOps
- Connaissance de base de Git
- Visual Studio Code ou éditeur de texte similaire
- Git CLI installé sur votre machine locale

## Concepts Clés

### Qu'est-ce qu'Azure Pipeline?
Azure Pipeline est un service cloud qui automatise les tests, les builds et les déploiements de projets de code. Il permet la Continuous Integration (CI) et la Continuous Deployment (CD).

### Composants Essentiels
- **Projet Azure DevOps**: Container pour votre code et vos pipelines
- **Référentiel Git**: Stockage de votre code source
- **Fichier YAML**: Définition déclarative de votre pipeline
- **Déclencheur (Trigger)**: Événement qui lance le pipeline

## Instructions Étape par Étape

### Étape 1: Naviguer vers Azure DevOps
1. Ouvrez https://dev.azure.com dans votre navigateur
2. Connectez-vous avec vos identifiants Microsoft
3. Sélectionnez votre organisation Azure DevOps
4. Cliquez sur "+ Nouveau projet"

### Étape 2: Créer un Nouveau Projet
1. Entrez le nom du projet: **"ProjetFormationAzure"**
2. Description: "Formation pratique sur les Pipelines Azure"
3. Visibilité: Privé (recommandé)
4. Contrôle de version: Git
5. Processus de travail: Agile
6. Cliquez sur "Créer"

 
### Étape 3: Initialiser le Référentiel Git
1. Une fois le projet créé, allez à la section **Repos**
2. Vous verrez une option "Initialiser avec un fichier README"
3. Cliquez sur **"Initialiser"**
4. Acceptez les paramètres par défaut
5. Attendez que la branche main soit créée



### Étape 4: Cloner le Référentiel Localement
1. Cliquez sur le bouton bleu **"Cloner"** en haut à droite
2. Copiez l'URL du référentiel
3. Ouvrez un terminal/PowerShell sur votre machine
4. Accédez au dossier de votre choix
5. Exécutez la commande:
   ```bash
   git clone [URL_COPIÉE]
   cd ProjetFormationAzure
   ```



### Étape 5: Créer le Fichier Pipeline
1. Créez un dossier **.azure-pipelines** dans le référentiel local:
   ```bash
   mkdir .azure-pipelines
   cd .azure-pipelines
   ```

2. Créez le fichier **azure-pipelines.yml**:
   ```bash
   # Windows
   
   
   # Linux/Mac
   touch azure-pipelines.yml
   ```

3. Ouvrez le fichier dans Visual Studio Code:
   ```bash
   code azure-pipelines.yml
   ```

4. Collez le contenu suivant:
   ```yaml
   trigger:
     - main

   pool:
     vmImage: 'ubuntu-latest'

   steps:
   - script: echo Bonjour, Bienvenue à la Formation Azure DevOps!
     displayName: 'Afficher le Message de Bienvenue'
   
   - script: echo Votre pipeline fonctionne correctement!
     displayName: 'Message de Confirmation'
   ```

**Explications du code**:
- `trigger`: Définit les événements qui déclenchent le pipeline (ici: changements sur main)
- `pool`: Spécifie l'environnement d'exécution (ubuntu-latest)
- `steps`: Liste des actions à exécuter
- `script`: Exécute une commande shell
- `displayName`: Nom affiché dans l'interface Azure DevOps


### Étape 6: Valider et Pousser les Modifications
1. Depuis le terminal, positionnez-vous à la racine du projet:
   ```bash
   cd ..
   ```

2. Vérifiez l'état:
   ```bash
   git status
   ```

3. Ajoutez les modifications:
   ```bash
   git add .
   ```

4. Créez un commit:
   ```bash
   git commit -m "Ajout du premier pipeline YAML"
   ```

5. Poussez vers le serveur:
   ```bash
   git push origin main
   ```


### Étape 7: Vérifier l'Exécution du Pipeline
1. Retournez à Azure DevOps dans le navigateur
2. Accédez à la section **Pipelines**
3. Vous devriez voir votre pipeline listé
4. Cliquez sur le pipeline pour voir son exécution
5. Attendez que l'exécution se termine (généralement 2-3 minutes)
6. Vérifiez que les deux étapes se sont exécutées avec succès

**Ce que vous devez observer**:
- Statut: Réussi (bouton vert)
- Deux étapes listées avec le statut "Réussi"
- Logs affichant les messages que vous avez définis


### Étape 8: Analyser les Logs
1. Cliquez sur la première étape: "Afficher le Message de Bienvenue"
2. Observez la sortie: "Bonjour, Bienvenue à la Formation Azure DevOps!"
3. Retournez et cliquez sur la deuxième étape
4. Observez sa sortie également

## Résultats Attendus

À la fin de ce laboratoire, vous devriez:
- ✅ Avoir un projet Azure DevOps fonctionnel
- ✅ Avoir un référentiel Git initialisé
- ✅ Avoir créé un fichier azure-pipelines.yml valide
- ✅ Avoir déclenché l'exécution automatique du pipeline
- ✅ Avoir observé l'exécution complète et réussie

## Livrables

1. **Capture d'écran 1**: Vue d'ensemble du projet Azure DevOps
2. **Capture d'écran 2**: Page Pipelines montrant votre pipeline créé
3. **Capture d'écran 3**: Exécution complète du pipeline avec les deux étapes
4. **Capture d'écran 4**: Logs détaillés montrant les messages affichés

## Dépannage Courant

### Le pipeline ne se déclenche pas automatiquement
- **Cause**: Le fichier YAML n'est pas à la bonne localisation
- **Solution**: Vérifiez que le fichier est dans le dossier racine ou .azure-pipelines

### Erreur: "Le fichier azure-pipelines.yml est introuvable"
- **Cause**: Le fichier n'a pas été validé correctement
- **Solution**: Vérifiez que vous avez poussé (git push) vos modifications

### Erreur de syntaxe YAML
- **Cause**: Indentation incorrecte
- **Solution**: L'indentation en YAML est critique. Utilisez 2 espaces, pas des tabulations

## Points Clés à Retenir

1. **Déclencheur**: Sans `trigger:`, le pipeline ne s'exécutera que manuellement
2. **Pool**: L'image VM détermine l'environnement d'exécution
3. **Noms d'affichage**: Les `displayName` améliorent la lisibilité dans les logs
4. **Versioning**: Stockez toujours vos pipelines dans le contrôle de version

## Étapes Suivantes

Une fois ce laboratoire terminé, vous serez prêt pour:
- LAB 2.1: Implémenter des pipelines multi-étapes
- Ajouter des étapes de build et de test
- Configurer des environnements de déploiement

---
