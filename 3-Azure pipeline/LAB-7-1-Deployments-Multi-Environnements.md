# LAB 7.1: Déploiements Multi-Environnements et Stratégies Avancées

## Durée
4 heures

## Objectifs d'Apprentissage
À la fin de ce laboratoire, vous serez capable de:
- Configurer des déploiements dans plusieurs environnements
- Implémenter la stratégie Blue-Green
- Mettre en œuvre une stratégie de déploiement Canary
- Configurer les vérifications de santé post-déploiement
- Implémenter la procédure de rollback
- Documenter les runbooks opérationnels

## Prérequis
- Avoir complété LAB 6.1
- Compréhension des environnements et permissions
- Familiarité avec les déploiements et les stratégies

## Concepts Clés

### Stratégies de Déploiement
- **Blue-Green**: Deux environnements identiques, basculement instantané
- **Canary**: Déploiement progressif avec trafic limité
- **Rolling**: Remplacement progressif des instances
- **Feature Flags**: Activation/désactivation de features

### Architecture Multi-Environnements
```
Dev (Déploiement Auto)
  ↓
Staging (Déploiement Auto)
  ↓
Production (Déploiement Manuel)
  ↓
Monitoring & Alerting
```

## Instructions Étape par Étape

### Étape 1: Configurer les Variables par Environnement
1. Créez un fichier de configuration **config/environments.yml**:

```yaml
environments:
  Development:
    name: dev
    vmSize: B1s
    instanceCount: 1
    autoScale: false
    healthCheckUrl: "https://dev.example.com/health"
    deploymentTimeout: 15
  
  Staging:
    name: staging
    vmSize: B2s
    instanceCount: 2
    autoScale: true
    healthCheckUrl: "https://staging.example.com/health"
    deploymentTimeout: 20
  
  Production:
    name: prod
    vmSize: P1v2
    instanceCount: 3
    autoScale: true
    healthCheckUrl: "https://app.example.com/health"
    deploymentTimeout: 30
    requiresApproval: true
    blueGreenEnabled: true
```

2. Commitez le fichier:
   ```bash
   git add config/
   git commit -m "Ajout de la configuration multi-environnements"
   ```

**Temps estimé**: 5 minutes

### Étape 2: Implémenter un Pipeline Multi-Environnements
1. Créez le fichier **azure-pipelines.yml** avec la configuration complète:

```yaml
trigger:
  - main

parameters:
- name: environment
  displayName: 'Environnement de Déploiement'
  type: string
  default: 'Development'
  values:
  - Development
  - Staging
  - Production

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  buildNumber: $(Build.BuildNumber)

stages:
- stage: Build
  displayName: 'Compilation'
  jobs:
  - job: BuildApplication
    displayName: 'Build l''Application'
    steps:
    - script: echo "Compilation pour ${{ parameters.environment }}..."
      displayName: 'Initialiser le Build'
    
    - script: echo "Build v$(buildNumber) créé"
      displayName: 'Numéro de Build'
    
    - script: mkdir -p $(Build.ArtifactStagingDirectory)/app
      displayName: 'Préparer Artifacts'
    
    - script: echo "Artefact v$(buildNumber)" > $(Build.ArtifactStagingDirectory)/app/version.txt
      displayName: 'Créer Fichier de Version'
    
    - task: PublishBuildArtifacts@1
      displayName: 'Publier Artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'

- stage: Test
  displayName: 'Tests'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - job: RunTests
    displayName: 'Exécuter les Tests'
    steps:
    - script: echo "Tests unitaires en cours..."
      displayName: 'Tests Unitaires'
    
    - script: echo "Couverture: 92%"
      displayName: 'Rapport de Couverture'
    
    - script: echo "Tests d'intégration en cours..."
      displayName: 'Tests Intégration'
    
    - script: echo "Tous les tests réussis!"
      displayName: 'Résumé des Tests'

- stage: PreDeployment
  displayName: 'Validation Pré-Déploiement'
  dependsOn: Test
  condition: succeeded()
  jobs:
  - job: SecurityScan
    displayName: 'Analyse de Sécurité'
    steps:
    - script: |
        echo "Analyse de vulnérabilités..."
        echo "✓ Pas de vulnérabilités critiques"
        echo "✓ Dépendances à jour"
        echo "✓ Secrets non exposés"
      displayName: 'Scan de Sécurité'
    
    - script: echo "Validation de la configuration..."
      displayName: 'Validation Configuration'

- stage: Deploy_${{ replace(parameters.environment, ' ', '_') }}
  displayName: 'Déployer en ${{ parameters.environment }}'
  dependsOn: PreDeployment
  condition: succeeded()
  variables:
    deploymentEnvironment: ${{ parameters.environment }}
  jobs:
  - deployment: Deploy
    displayName: 'Déploiement ${{ parameters.environment }}'
    environment: ${{ parameters.environment }}
    strategy:
      runOnce:
        preDeploy:
          steps:
          - script: echo "Préparation du déploiement en ${{ parameters.environment }}..."
            displayName: 'Pré-Déploiement'
          
          - download: current
            artifact: drop
            displayName: 'Télécharger Artifacts'
        
        deploy:
          steps:
          - script: |
              echo "Déploiement de l'application en ${{ parameters.environment }}"
              echo "Version: $(buildNumber)"
              echo "Configuration: Release"
            displayName: 'Déployer Application'
          
          - script: |
              echo "Configuration des paramètres..."
              echo "Environnement: ${{ parameters.environment }}"
            displayName: 'Configuration Environnement'
        
        postDeploy:
          steps:
          - script: echo "Vérifications post-déploiement..."
            displayName: 'Post-Déploiement'
          
          - script: |
              echo "Vérification de santé..."
              echo "✓ Endpoints accessibles"
              echo "✓ Base de données connectée"
              echo "✓ Services prêts"
            displayName: 'Health Check'

- stage: Monitoring
  displayName: 'Monitoring Post-Déploiement'
  dependsOn: Deploy_${{ replace(parameters.environment, ' ', '_') }}
  condition: succeeded()
  jobs:
  - job: MonitorHealth
    displayName: 'Surveiller la Santé'
    steps:
    - script: |
        echo "Vérification de l'application..."
        echo "Taux d'erreur: 0.02%"
        echo "Disponibilité: 99.98%"
        echo "Temps de réponse: 145ms"
      displayName: 'Métriques de Santé'
    
    - script: echo "Pas d'alertes"
      displayName: 'Vérifier Alertes'
```

**Explication**:
- `parameters.environment`: Permet de choisir l'environnement au lancement
- `runOnce` strategy: Garantit une exécution unique et idempotente
- Phases: preDeploy, deploy, postDeploy

**Temps estimé**: 20 minutes

### Étape 3: Implémenter la Stratégie Blue-Green
1. Créez un template pour Blue-Green **templates/strategies/blue-green.yml**:

```yaml
parameters:
  - name: environment
    type: string
  - name: blueSlotName
    type: string
    default: 'blue'
  - name: greenSlotName
    type: string
    default: 'green'
  - name: appServiceName
    type: string

steps:
# Phase 1: Déployer sur la fente verte
- script: |
    echo "Déploiement de la nouvelle version sur le slot VERT..."
    echo "App Service: ${{ parameters.appServiceName }}"
    echo "Slot: ${{ parameters.greenSlotName }}"
  displayName: 'Phase 1: Déployer sur Vert'

# Phase 2: Tests de smoke sur le slot vert
- script: |
    echo "Exécution des tests de smoke sur le slot VERT..."
    echo "✓ Endpoints accessibles"
    echo "✓ Pas d'erreurs critiques"
    echo "✓ Performance acceptable"
  displayName: 'Phase 2: Tests de Smoke'

# Phase 3: Basculer le trafic
- script: |
    echo "Basculage du trafic vers le slot VERT..."
    echo "Le BLEU devient l'ancienne version"
    echo "Le VERT devient la version active"
  displayName: 'Phase 3: Basculer Trafic'

# Phase 4: Monitoring
- script: |
    echo "Monitoring de la version active..."
    echo "Attendant 5 minutes pour stabilité..."
  displayName: 'Phase 4: Monitoring'

# Phase 5: Rollback en cas de problème
- script: |
    echo "Si problème détecté, préparation du rollback..."
    echo "Basculage inversé possible en < 1 minute"
  displayName: 'Phase 5: Préparation Rollback'
```

2. Utilisez ce template dans votre pipeline:
   ```yaml
   - stage: DeployBlueGreen
     displayName: 'Déploiement Blue-Green'
     jobs:
     - deployment: BlueGreenDeploy
       displayName: 'Exécuter Blue-Green'
       environment: Production
       strategy:
         runOnce:
           deploy:
             steps:
             - template: templates/strategies/blue-green.yml
               parameters:
                 environment: Production
                 appServiceName: 'myapp-prod'
   ```

**Temps estimé**: 10 minutes

### Étape 4: Implémenter la Stratégie Canary
1. Créez un template Canary **templates/strategies/canary.yml**:

```yaml
parameters:
  - name: canaryTrafficPercentage
    type: number
    default: 10
  - name: monitoringDurationMinutes
    type: number
    default: 10
  - name: errorThreshold
    type: number
    default: 1.0

steps:
# Phase 1: Déploiement Canary
- script: |
    echo "Déploiement Canary (nouvelle version)"
    echo "Trafic: ${{ parameters.canaryTrafficPercentage }}%"
  displayName: 'Phase 1: Déployer Canary'

# Phase 2: Monitoring
- script: |
    echo "Monitoring pendant ${{ parameters.monitoringDurationMinutes }} minutes..."
    echo "Seuil d'erreur acceptable: ${{ parameters.errorThreshold }}%"
  displayName: 'Phase 2: Monitoring'

# Phase 3: Augmentation du Trafic
- script: |
    echo "Trafic augmenté à 25%..."
    echo "Monitoring supplémentaire..."
  displayName: 'Phase 3: Augmenter Trafic'

# Phase 4: Déploiement Complet
- script: |
    echo "Déploiement complet en 100%"
    echo "Version Canary devient la version active"
  displayName: 'Phase 4: Déploiement Complet'

# Phase 5: Cleanup
- script: |
    echo "Arrêt des anciennes instances"
    echo "Nettoyage des ressources temporaires"
  displayName: 'Phase 5: Cleanup'
```

### Étape 8: Documentation Complète
1. Créez **docs/deployment-guide.md**:

```markdown
# Guide de Déploiement

## Prérequis
- Accès à Azure DevOps
- Permissions d'approbation pour Production
- Connaissance des stratégies de déploiement

## Procédure de Déploiement Standard

### 1. Préparation
- [ ] Vérifier que tous les tests passent
- [ ] Vérifier la couverture de code (> 80%)
- [ ] Vérifier qu'aucune vulnérabilité critique

### 2. Déploiement en Development
- Automatique après build
- Vérifier les logs
- Tester fonctionnalités critiques

### 3. Déploiement en Staging
- Automatique après test réussi
- Tests de performance
- Tests de charge
- Vérifier l'intégration

### 4. Déploiement en Production
- Demande une approbation manuelle
- **Strategy**: Blue-Green recommandée
- **Monitoring**: Actif pendant 15 minutes post-déploiement
- **Rollback**: Disponible jusqu'à 1 heure après

## Stratégies de Déploiement

### Blue-Green (Recommandée)
- Temps d'arrêt: 0
- Risque: Moyen
- Temps de rollback: 1 minute

### Canary (Pour les changements majeurs)
- Temps d'arrêt: 0
- Risque: Bas
- Temps de rollback: 1 minute
- Trafic initial: 10%

### Rolling (Pour les mises à jour de dépendances)
- Temps d'arrêt: 0
- Risque: Moyen-Bas
- Temps de rollback: 5 minutes
```

**Temps estimé**: 5 minutes

## Résultats Attendus

À la fin de ce laboratoire, vous devriez:
- ✅ Avoir configuré un pipeline multi-environnements
- ✅ Avoir implémenté la stratégie Blue-Green
- ✅ Avoir implémenté la stratégie Canary
- ✅ Avoir créé une procédure de rollback
- ✅ Avoir mis en place des health checks
- ✅ Avoir documenté les procédures opérationnelles

## Livrables

1. **Capture d'écran 1**: Pipeline multi-environnements en exécution
2. **Capture d'écran 2**: Déploiement réussi en tous les environnements
3. **Fichiers**: Tous les templates (Blue-Green, Canary, Health Check)
4. **Documentation**: Runbook complet et guide de déploiement
5. **Capture d'écran 3**: Monitoring et health checks

## Tableau Comparatif des Stratégies

| Aspect | Blue-Green | Canary | Rolling |
|--------|-----------|--------|---------|
| **Temps d'arrêt** | 0 | 0 | 0 |
| **Complexité** | Moyenne | Élevée | Basse |
| **Rollback rapide** | Oui | Oui | Oui |
| **Impact utilisateurs** | Aucun | Limité | Progressif |
| **Idéal pour** | Changements majeurs | Nouvelles features | Updates mineures |

## Checklist de Déploiement Production

- [ ] Tous les tests passent (100%)
- [ ] Code review approuvé
- [ ] Security scan réussi
- [ ] Performance validée
- [ ] Notification équipe
- [ ] Fenêtre de maintenance confirmée
- [ ] Plan de rollback documenté
- [ ] Monitoring configuré
- [ ] On-call engineer disponible
- [ ] Health checks en place

## Dépannage Courant

### "Le health check échoue après déploiement"
- **Cause**: Application pas prête ou lenteur de démarrage
- **Solution**: Augmenter délai de retry ou nombre de tentatives

### "Trafic ne bascule pas en Blue-Green"
- **Cause**: Slot non préparé correctement
- **Solution**: Vérifier la configuration de l'App Service

### "Canary détecte trop d'erreurs"
- **Cause**: Seuil trop stricte ou problème réel
- **Solution**: Analyser les erreurs et ajuster le seuil ou fixer le code

## Points Clés à Retenir

1. **Multi-environnements**: Dev → Staging → Prod
2. **Blue-Green**: Zéro downtime, facile à rollback
3. **Canary**: Test réel avec utilisateurs
4. **Health Checks**: Essentiels pour la fiabilité
5. **Rollback Plan**: Toujours prêt
6. **Documentation**: Runbooks vitales

## Étapes Suivantes

Une fois ce laboratoire terminé:
- Implémenter le monitoring avancé avec Application Insights
- Configurer les alertes automatiques
- Mettre en place les dashboards de déploiement
- Automatiser les rollbacks conditionnels

---

**Durée totale du laboratoire**: ~90 minutes
**Heure de lab supplémentaire**: Pour tester les stratégies en production et optimiser les métriques de monitoring
