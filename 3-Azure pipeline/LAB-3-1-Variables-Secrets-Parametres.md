# LAB 3.1: Gestion des Variables, Paramètres et Secrets

## Durée
2.5 heures

## Objectifs d'Apprentissage
À la fin de ce laboratoire, vous serez capable de:
- Créer et utiliser des variables de pipeline
- Définir des groupes de variables
- Implémenter des variables secrètes sécurisées
- Gérer la portée des variables (pipeline, étape, job)
- Utiliser les paramètres de template

## Prérequis
- Avoir complété LAB 2.1
- Accès à la section Pipelines → Bibliothèque d'Azure DevOps
- Permissions pour créer des groupes de variables

## Concepts Clés

### Types de Variables
- **Variables Pipeline**: Accessibles dans tout le pipeline
- **Variables d'Environnement**: Variables au niveau du système d'exploitation
- **Variables Secrètes**: Chiffrées et masquées dans les logs
- **Variables de Sortie**: Définies par une étape, utilisées par une autre
- **Groupes de Variables**: Variables partagées entre plusieurs pipelines

### Portée des Variables
```
Organisation
└── Projet
    └── Pipeline
        └── Étape
            └── Job
                └── Step
```
Les variables définies à un niveau sont accessibles aux niveaux inférieurs.

### Syntaxe d'Accès
- Dans YAML: `$(nomVariable)`
- Dans les scripts: Dépend du shell (bash vs PowerShell)

## Instructions Étape par Étape

### Étape 1: Créer un Groupe de Variables
1. Dans Azure DevOps, allez à **Pipelines** → **Library** → **Groupes de variables**
2. Cliquez sur **+ Groupe de variables**
3. Nommez le groupe: **"SecretsPipeline"**
4. Cliquez sur **Enregistrer**

**Temps estimé**: 3 minutes

### Étape 2: Ajouter des Variables Secrètes
1. Le groupe "SecretsPipeline" est maintenant ouvert
2. Cliquez sur **+ Ajouter une variable**
3. Nom: `DatabasePassword`
4. Valeur: `MonMotDePasse123!` (exemple)
5. Cliquez sur l'icône cadenas pour marquer comme secret
6. Cliquez sur **OK**

Répétez pour ajouter:
- Variable: `ApiKey` - Valeur: `sk-1234567890abcdef`
- Variable: `EnvironmentUrl` - Valeur: `https://api.example.com`

7. Cliquez sur **Enregistrer** pour valider le groupe

**Temps estimé**: 5 minutes

### Étape 3: Créer les Variables de Pipeline
1. Ouvrez votre fichier **azure-pipelines.yml** localement
2. Remplacez le contenu par:

```yaml
# ========================================
# PIPELINE : Gestion des Variables
# ========================================

trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

# ========================================
# VARIABLES GLOBALES
# ========================================
variables:
  # Groupe de variables secret (défini dans Azure DevOps)
  - group: SecretsPipeline
  
  # Variables locales (publiques)
  - name: buildConfiguration
    value: 'Release'
  - name: developerName
    value: 'FormationAzure'
  - name: projectEnvironment
    value: 'Training'

# ========================================
# STAGES
# ========================================
stages:
  # ========================================
  # STAGE 1 : BUILD
  # ========================================
  - stage: Build
    displayName: '🔨 Stage de Build'
    
    # Variables au niveau du stage
    variables:
      buildPlatform: 'Any CPU'
    
    jobs:
      - job: BuildJob
        displayName: "Tâche d'Affichage des Variables"
        
        steps:
          # Étape 1 : Afficher variables globales
          - script: |
              echo "=========================================="
              echo "=== Variables Globales ==="
              echo "=========================================="
              echo "Configuration: $(buildConfiguration)"
              echo "Développeur: $(developerName)"
              echo "Environnement: $(projectEnvironment)"
              echo "=========================================="
            displayName: 'Afficher Variables Globales'
          
          # Étape 2 : Afficher variables du stage
          - script: |
              echo "=========================================="
              echo "=== Variables du Stage ==="
              echo "=========================================="
              echo "Plateforme de build: $(buildPlatform)"
              echo "=========================================="
            displayName: "Afficher Variables du Stage"
          
          # Étape 3 : Vérifier variables secrètes (masquées)
          - script: |
              echo "=========================================="
              echo "=== Variables Secrètes (Masquées) ==="
              echo "=========================================="
              echo "Mot de passe DB: $(DatabasePassword)"
              echo "Remarque: Le secret n'est jamais affiché en clair"
              echo "=========================================="
            displayName: "Vérifier Variables Secrètes"

  # ========================================
  # STAGE 2 : TEST
  # ========================================
  - stage: Test
    displayName: '✅ Stage de Test'
    dependsOn: Build
    condition: succeeded()
    
    jobs:
      - job: TestJob
        displayName: 'Tâche de Test des Paramètres'
        
        # Variables au niveau du job
        variables:
          testConfiguration: 'Debug'
        
        steps:
          - script: |
              echo "=========================================="
              echo "=== Test de la Portée des Variables ==="
              echo "=========================================="
              echo "Configuration (globale): $(buildConfiguration)"
              echo "Configuration (job): $(testConfiguration)"
              echo "Environnement (globale): $(projectEnvironment)"
              echo "Plateforme (stage Build): $(buildPlatform)"
              echo "=========================================="
            displayName: 'Tester Portée des Variables'

  # ========================================
  # STAGE 3 : DEPLOY
  # ========================================
  - stage: Deploy
    displayName: '🚀 Stage de Déploiement'
    dependsOn: Test
    condition: succeeded()
    
    jobs:
      - job: DeployJob
        displayName: 'Tâche de Déploiement'
        
        # Variables au niveau du job
        variables:
          deploymentTimeout: '30'
          deploymentRegion: 'westeurope'
        
        steps:
          - script: |
              echo "=========================================="
              echo "=== Préparation du Déploiement ==="
              echo "=========================================="
              echo "Environnement cible: $(projectEnvironment)"
              echo "Configuration: $(buildConfiguration)"
              echo "Timeout déploiement: $(deploymentTimeout) minutes"
              echo "Région: $(deploymentRegion)"
              echo "=========================================="
            displayName: 'Préparer le Déploiement'
          
          - script: |
              echo "Simulation du déploiement en cours..."
              sleep 2
              echo "Déploiement terminé avec succès!"
            displayName: 'Exécuter le Déploiement'


```

**Points importants**:
- `group: SecretsPipeline` - Charge les variables du groupe créé
- Les secrets sont **automatiquement masqués** dans les logs
- Les variables déclarées à un niveau supérieur sont accessibles en dessous
- `$(nomVariable)` est remplacé par sa valeur au runtime

**Temps estimé**: 10 minutes

### Étape 4: Valider et Pousser
1. Vérifiez la syntaxe YAML (indentation correcte)
2. Depuis le terminal à la racine du projet:
   ```bash
   git add .azure-pipelines/azure-pipelines.yml
   git commit -m "Ajout de la gestion des variables et groupes de variables"
   git push origin main
   ```

**Temps estimé**: 3 minutes

### Étape 5: Exécuter et Analyser
1. Retournez à Azure DevOps
2. Allez à **Pipelines** et attendez le nouveau pipeline
3. Cliquez dessus pour voir l'exécution

**Points à observer**:
- Les variables globales affichées correctement
- Les variables secrètes **masquées** par `***` dans les logs
- Les variables spécifiques à chaque étape
- Aucun secret en clair nulle part

4. Cliquez sur la première étape "Afficher Variables Globales"
5. Observez les logs:
   ```
   Configuration: Release
   Développeur: FormationAzure
   Environnement: IEasyTraining
   ```

6. Cliquez sur "Vérifier Variables Secrètes"
7. Vous verrez: `Les secrets ne seront pas affichés: ***`

**Temps estimé**: 8 minutes

### Étape 6: Configurer les Permissions du Groupe (Sécurité)
1. Retournez à **Pipelines** → **Library** → **Groupes de variables**
2. Sélectionnez le groupe "SecretsPipeline"
3. Cliquez sur **Permissions du Pipeline**
4. Cliquez sur **+ Ajouter**
5. Sélectionnez votre pipeline
6. Cliquez sur **Autoriser** ou **Refuser** selon vos besoins

**Concepts de sécurité**:
- Par défaut, aucun pipeline ne peut accéder au groupe
- Vous devez **autoriser explicitement** chaque pipeline
- Cela fonctionne comme un pare-feu pour les secrets


### Étape 7: Utiliser les Paramètres de Template (Avancé)
1. Créez un dossier `templates` à la racine du projet:
   ```bash
   mkdir templates
   ```

2. Créez le fichier `templates/build-params.yml`:
   ```yaml
   parameters:
     - name: buildConfig
       type: string
       default: 'Release'
     - name: runTests
       type: boolean
       default: true
   
   jobs:
   - job: BuildWithParams
     displayName: 'Build avec Paramètres'
     steps:
     - script: echo "Configuration: ${{ parameters.buildConfig }}"
       displayName: 'Afficher Configuration du Paramètre'
     
     - ${{ if parameters.runTests }}:
       - script: echo "Exécution des tests..."
         displayName: 'Tests Conditionnels'
   ```

3. Modifiez votre pipeline pour utiliser ce template:
   ```yaml
   stages:
   - stage: BuildWithTemplate
     displayName: 'Build avec Template'
     jobs:
     - template: templates/build-params.yml
       parameters:
         buildConfig: 'Debug'
         runTests: true
   ```

4. Poussez et exécutez:
   ```bash
   git add .
   git commit -m "Ajout des templates avec paramètres"
   git push origin main
   ```

**Temps estimé**: 10 minutes

## Résultats Attendus

À la fin de ce laboratoire, vous devriez:
- ✅ Avoir créé un groupe de variables
- ✅ Avoir ajouté des variables secrètes
- ✅ Avoir utilisé les variables dans le pipeline
- ✅ Avoir observé les secrets masqués dans les logs
- ✅ Avoir compris la portée des variables
- ✅ Avoir configuré les permissions du groupe

## Livrables

1. **Capture d'écran 1**: Groupe de variables "SecretsPipeline" avec variables
2. **Capture d'écran 2**: Logs du pipeline montrant les variables affichées
3. **Capture d'écran 3**: Logs montrant les secrets masqués avec `***`
4. **Capture d'écran 4**: Permissions du pipeline configurées
5. **Fichier**: Le fichier azure-pipelines.yml mis à jour

## Tableau de Portée des Variables

| Niveau | Accessible Depuis | Exemple |
|--------|------------------|---------|
| Pipeline | Toutes les étapes | Défini avec `variables:` au top niveau |
| Étape | Tous les jobs de cette étape | Défini dans `variables:` de stage |
| Job | Toutes les étapes du job | Défini dans `variables:` du job |
| Étape | Seulement cette étape | Défini dans la step |

## Bonnes Pratiques de Sécurité

1. **Jamais de secrets en clair dans le YAML**
   - ❌ Mauvais: `password: MonMotDePasse123`
   - ✅ Bon: Utiliser un groupe de variables avec variable secrète

2. **Autoriser explicitement les pipelines**
   - Les permissions par défaut sont restrictives
   - Accordez seulement le nécessaire

3. **Rotation des secrets**
   - Changez régulièrement les clés API et mots de passe
   - Utilisez Azure Key Vault pour les environnements production

4. **Audit des accès**
   - Consultez régulièrement qui a accès aux groupes de variables
   - Supprimez les accès non utilisés

## Dépannage Courant

### "Le groupe de variables est introuvable"
- **Cause**: Le pipeline n'a pas accès au groupe
- **Solution**: Configurez les permissions du groupe

### Les variables ne se substituent pas
- **Cause**: Syntaxe incorrecte ou variable mal nommée
- **Solution**: Vérifiez `$(exactNomVariable)` avec la casse correcte

### Les secrets apparaissent dans les logs
- **Cause**: Affichage accidentel dans un script
- **Solution**: Azure DevOps masque automatiquement, mais évitez d'afficher intentionnellement

## Points Clés à Retenir

1. **Portée hiérarchique**: Les variables définies haut sont accessibles bas
2. **Groupes partagés**: Économisent la duplication et centralisent les secrets
3. **Sécurité automatique**: Les variables secrètes sont masquées dans les logs
4. **Permissions explicites**: Nécessaires pour accéder aux groupes
5. **Paramètres de template**: Rendent les pipelines plus flexibles et réutilisables

## Étapes Suivantes

Une fois ce laboratoire terminé:
- LAB 4.1: Configurer les agents on-premises
- LAB 5.1: Implémenter les tâches et templates avancés
- LAB 6.1: Configurer les permissions et les portes d'approbation

---

**Durée totale du laboratoire**: ~45 minutes
**Heure de lab supplémentaire**: Pour explorer les templates avancés et la sécurité
