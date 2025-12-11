# LAB 6.1: Sécurité, Permissions et Portes d'Approbation

## Durée
2.5 heures

## Objectifs d'Apprentissage
À la fin de ce laboratoire, vous serez capable de:
- Configurer les permissions des groupes de variables
- Implémenter les portes d'approbation (approval gates)
- Gérer les permissions d'environnement
- Implémenter la validation manuelle
- Comprendre le modèle de sécurité des pipelines
- Auditer les accès et approvals

## Prérequis
- Avoir complété LAB 5.1
- Accès à la gestion des permissions dans Azure DevOps
- Au minimum deux comptes utilisateurs pour tester les approvals (optionnel)
- Compréhension des groupes de variables

## Concepts Clés

### Modèle de Sécurité
```
Organisation
└── Projet
    └── Pipeline/Environnement
        └── Permissions
            ├── Lecteur
            ├── Contributeur
            └── Administrateur
```

### Types de Contrôles de Sécurité
- **Permissions de Pipeline**: Qui peut créer/modifier
- **Permissions de Groupe de Variables**: Qui peut accéder aux secrets
- **Permissions d'Environnement**: Qui peut déployer
- **Portes d'Approbation**: Validation avant déploiement

## Instructions Étape par Étape

### Étape 1: Créer des Environnements
1. Allez à votre projet Azure DevOps
2. Accédez à **Pipelines** → **Environnements**
3. Cliquez sur **Créer un environnement**

4. Créez les environnements suivants:
   - **Développement**: Pour les builds de test
   - **Staging**: Environnement de pré-production
   - **Production**: Environnement de production

Pour chaque environnement:
- Nom: comme ci-dessus
- Description: "Environnement de [type]"
- Cliquez sur **Créer**

**Temps estimé**: 10 minutes

### Étape 2: Configurer les Portes d'Approbation
1. Sélectionnez l'environnement **Production**
2. Cliquez sur **Approvals and checks** 
3. Sélectionnez **Approvals**
4. Cliquez sur **+ Ajouter**
5. Remplissez les champs:
   - **Approuveurs**: Sélectionnez vous-même ou un collègue
   - **Instructions**: "Vérifier que le déploiement est sûr avant d'approuver"
   - **Délai d'expiration des approbations (en minutes)**: 1440 (24 heures)
   - **Le déploiement peut être approuvé par le demandeur**: Décochez cette case (meilleure pratique)

6. Cliquez sur **Créer**

**Temps estimé**: 5 minutes

### Étape 3: Configurer les Permissions du Groupe de Variables
1. Allez à **Pipelines** → **Bibliothèque** → **Groupes de variables**
2. Sélectionnez le groupe **SecretsPipeline** (créé en LAB 3.1)
3. Cliquez sur **Permissions du pipeline**

4. Vous verrez une liste de pipelines:
   - Par défaut: **Pas d'autorisation** pour tous
   - Cliquez sur **+ Ajouter**
   - Sélectionnez votre pipeline
   - Cliquez sur **Autoriser**

**Résultat**: Seul ce pipeline peut accéder aux secrets

5. Documentez les permissions:
   - Pipeline A: Autorisé ✅
   - Pipeline B: Interdit ❌

**Temps estimé**: 5 minutes

### Étape 4: Créer un Pipeline avec Portes d'Approbation
1. Ouvrez votre fichier **azure-pipelines.yml**
2. Remplacez le contenu par:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: SecretsPipeline
  - name: buildConfiguration
    value: 'Release'

stages:
- stage: Build
  displayName: 'Stage de Build'
  jobs:
  - job: BuildJob
    displayName: 'Compiler l''Application'
    steps:
    - script: echo "Compilation de l'application..."
      displayName: 'Compiler'
    
    - script: echo "Build réussi!"
      displayName: 'Confirmation'

- stage: Test
  displayName: 'Stage de Test'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - job: TestJob
    displayName: 'Tests de Qualité'
    steps:
    - script: echo "Exécution des tests..."
      displayName: 'Tests Unitaires'
    
    - script: echo "Couverture: 85%"
      displayName: 'Rapport de Couverture'

- stage: DeployDev
  displayName: 'Déployer en Développement'
  dependsOn: Test
  condition: succeeded()
  jobs:
  - deployment: DeployToDev
    displayName: 'Déploiement Développement'
    environment: 'Développement'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Déploiement en Développement..."
            displayName: 'Deploy Dev'

- stage: DeployStaging
  displayName: 'Déployer en Staging'
  dependsOn: DeployDev
  condition: succeeded()
  jobs:
  - deployment: DeployToStaging
    displayName: 'Déploiement Staging'
    environment: 'Staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Déploiement en Staging..."
            displayName: 'Deploy Staging'

- stage: DeployProduction
  displayName: 'Déployer en Production'
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - deployment: DeployToProd
    displayName: 'Déploiement Production'
    environment: 'Production'  # Cet environnement a une porte d'approbation
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Déploiement APPROUVÉ en Production!"
            displayName: 'Deploy Production'
          
          - script: |
              echo "======================================="
              echo "Déploiement en Production - Approuvé"
              echo "======================================="
            displayName: 'Notification de Déploiement'
```

**Explication**:
- `environment: 'Production'` - Cet étage utilisera l'environnement avec porte d'approbation
- Avant cette étape, le pipeline demande une approbation manuelle
- Sans approbation, le pipeline ne peut pas continuer

****Configuration du Stage****
- **stage: DeployDev** : Identifiant unique du stage utilisé pour référencer cette étape dans le pipeline et établir des dépendances avec d'autres stages.

- **displayName:** 'Déployer en Développement' : Nom descriptif affiché dans l'interface Azure DevOps pour faciliter la lecture et le suivi de l'exécution du pipeline.

- **dependsOn: Test** : Indique que ce stage ne peut démarrer qu'après la complétion du stage nommé "Test", créant ainsi une séquence ordonnée dans votre pipeline (Build → Test → DeployDev).

- **condition: succeeded()** : Le déploiement ne s'exécutera que si le stage "Test" s'est terminé avec succès, évitant ainsi de déployer du code défaillant.

Configuration du Job de Déploiement
- **deployment: DeployToDev** : Type de job spécial pour les déploiements qui offre des fonctionnalités avancées comme l'historique des déploiements, le suivi et les approbations.

- **environment**: 'Développement' : Référence l'environnement Azure DevOps nommé "Développement", permettant de configurer des approbations, des contrôles de sécurité et de suivre l'historique des déploiements sur cet environnement.

Stratégie de Déploiement
- **strategy: runOnce** : Stratégie de déploiement la plus simple qui exécute les étapes une seule fois sans phases de pré-déploiement, déploiement ou post-déploiement complexes. Cette stratégie convient parfaitement à l'environnement de développement où la rapidité prime sur les contrôles élaborés.

- **deploy: steps:** : Section contenant les étapes concrètes du déploiement, ici un simple script d'écho qui simule le déploiement (dans un cas réel, vous y placeriez vos commandes de déploiement comme az webapp deployment, kubectl apply, ou autres).

### Étape 5: Ajouter une Validation Avant Déploiement
1. Modifiez le pipeline pour ajouter une étape de vérification:

```yaml
- stage: DeployProduction
  displayName: 'Déployer en Production'
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - job: PreDeploymentValidation
    displayName: 'Validation Pré-Déploiement'
    steps:
    - script: |
        echo "Vérifications de sécurité:"
        echo "✓ Tous les tests ont réussi"
        echo "✓ Pas de vulnérabilités détectées"
        echo "✓ Configuration validée"
      displayName: 'Vérifications de Sécurité'
    
    - script: |
        echo "Prêt pour l'approbation manuelle"
      displayName: 'Prêt pour Production'
  
  - deployment: DeployToProd
    displayName: 'Déploiement Production'
    dependsOn: PreDeploymentValidation
    condition: succeeded()
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "DÉPLOIEMENT EN PRODUCTION - APPROUVÉ!"
            displayName: 'Deploy Production'
```

**Temps estimé**: 5 minutes

### Étape 6: Commiter et Tester
1. Commitez les modifications:
   ```bash
   git add .azure-pipelines/azure-pipelines.yml
   git commit -m "Ajout des étapes de déploiement avec portes d'approbation"
   git push origin main
   ```

2. Attendez que le pipeline s'exécute
3. Observez que:
   - Build → Test → DeployDev → DeployStaging s'exécutent automatiquement
   - DeployProduction **pause** et demande une approbation

**Temps estimé**: 10 minutes

### Étape 7: Approuver le Déploiement
1. Attendez que le pipeline atteigne l'étape Production
2. Vous devriez voir une notification d'approbation
3. Cliquez sur **Examiner** (Review) ou **Approuver**
4. Entrez éventuellement une raison: "Déploiement validé et sûr"
5. Cliquez sur **Approuver**

**Résultat**: Le déploiement Production s'exécute

**Temps estimé**: 2 minutes

### Étape 8: Configurer les Restrictions d'Utilisateurs
1. Retournez à l'environnement **Production**
2. Cliquez sur **Approvals and checks**
3. Cliquez sur **Branch control**
4. Cliquez sur **+ Ajouter**
5. Paramètres:
   - **Branche autorisée**: main (seulement les déploiements de main)
   - Cliquez sur **Créer**

**Résultat**: Seule la branche main peut être déployée en production

**Temps estimé**: 3 minutes

### Étape 9: Implémenter une Validation de Temps
1. Retournez à l'environnement **Production**
2. Cliquez sur **Approvals and checks**
3. Cliquez sur **Business hours check**
4. Cliquez sur **+ Ajouter**
5. Paramètres:
   - **Fuseau horaire**: UTC+1 (ou votre fuseau)
   - **Heure de début**: 08:00
   - **Heure de fin**: 18:00
   - **Jours**: Lu-Ve
   - Cliquez sur **Créer**

**Résultat**: Les déploiements en production ne peuvent se faire que durant les heures de bureau


## Résultats Attendus

À la fin de ce laboratoire, vous devriez:
- ✅ Avoir créé des environnements (Dev, Staging, Prod)
- ✅ Avoir configuré une porte d'approbation sur Production
- ✅ Avoir configuré les permissions des groupes de variables
- ✅ Avoir exécuté un pipeline avec approbation manuelle
- ✅ Avoir approuvé et complété un déploiement
- ✅ Avoir audité les décisions d'approbation

## Livrables

1. **Capture d'écran 1**: Environnements créés (Dev, Staging, Prod)
2. **Capture d'écran 2**: Porte d'approbation configurée
3. **Capture d'écran 3**: Pipeline en attente d'approbation
4. **Capture d'écran 4**: Approbation fournie
5. **Capture d'écran 5**: Audit des approbations
6. **Fichier**: azure-pipelines.yml avec les étapes de déploiement

## Matrice de Permissions - Bonnes Pratiques

| Rôle | Build | Test | Dev | Staging | Prod |
|------|-------|------|-----|---------|------|
| **Développeur** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **QA Lead** | ✅ | ✅ | ✅ | ✅ | ❌ |
| **DevOps** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Approver** | ✅ | ✅ | ✅ | ✅ | ✅* |

*Approbation uniquement, pas de modification

## Checklist de Sécurité

- [ ] Porte d'approbation configurée sur Production
- [ ] Approbateurs désignés clairement
- [ ] Branche control limité à main
- [ ] Pas d'auto-approbation en production
- [ ] Audit logging activé
- [ ] Permissions du groupe de variables restrictives
- [ ] Jetons d'accès rotés régulièrement
- [ ] Documentation des procédures d'approbation

## Dépannage Courant

### "L'environnement n'a pas de porte d'approbation"
- **Cause**: Non configuré
- **Solution**: Allez à Environnements → Approvals and checks → Ajouter

### "Approbation refusée sans raison"
- **Cause**: Validation du contrôle de branche échouée
- **Solution**: Vérifiez que vous déployez depuis la branche autorisée

### "Le déploiement continue sans approbation"
- **Cause**: L'étape n'utilise pas `environment:`
- **Solution**: Ajoutez `environment: 'Production'` à l'étape

## Points Clés à Retenir

1. **Environnements**: Conteneurs logiques pour les contrôles de sécurité
2. **Approvals**: Validation manuelle avant déploiement critique
3. **Permissions**: Modèle granulaire par ressource
4. **Audit**: Traçabilité de toutes les actions
5. **Responsabilité**: Clarifier qui approuve quoi

## Stratégies de Déploiement Sûr

### Déploiement en Escalier
```
Dev (Auto) → Staging (Auto) → Prod (Manuel)
```

### Déploiement en Heures de Bureau
- Production déployable uniquement 8h-18h
- Heures non-bureau: Pas de déploiement critique

### Politique de Rotation des Approuveurs
- Au moins 2 personnes désignées
- Personne qui build ne peut pas approuver sa propre build

## Étapes Suivantes

Une fois ce laboratoire terminé:
- LAB 7.1: Déploiements en environnements multiples
- Implémenter des stratégies de déploiement avancées
- Configurer les alertes sur les approbations

---

**Durée totale du laboratoire**: ~50 minutes
**Heure de lab supplémentaire**: Pour tester différents scénarios d'approbation et affiner les permissions
