# LAB 4.1: Configuration et Gestion des Agents On-Premises

## Durée
3 heures

## Objectifs d'Apprentissage
À la fin de ce laboratoire, vous serez capable de:
- Créer un pool d'agents personnalisé
- Générer un jeton d'accès personnel (PAT)
- Déployer un agent auto-hébergé sur une machine Windows ou Linux
- Enregistrer l'agent auprès d'Azure DevOps
- Vérifier l'état et les capacités de l'agent
- Exécuter des pipelines sur un agent auto-hébergé

## Prérequis
- Avoir complété LAB 3.1
- Accès administrateur à une machine Windows ou Linux
- Accès à Azure DevOps avec permissions d'administration
- Connectivité réseau entre la machine agent et Azure DevOps
- Git installé sur la machine (pour les tests)

## Concepts Clés

### Types d'Agents
- **Agents Hébergés par Microsoft**: Gérés par Microsoft, automatiquement mis à jour
- **Agents Auto-Hébergés**: Sur votre infrastructure, contrôle total

### Architecture Agent
```
Azure DevOps (Cloud)
        ↓
    Pool d'Agents
        ↓
   Agent 1 (On-Prem)
   Agent 2 (Cloud)
   Agent 3 (On-Prem)
```

### Flux d'Authentification
1. Création d'un PAT (Personal Access Token)
2. L'agent utilise le PAT pour s'authentifier
3. L'agent s'enregistre dans le pool
4. Le pipeline assign le job à un agent disponible

## Instructions Étape par Étape

### Étape 1: Créer un Pool d'Agents Personnalisé
1. Dans Azure DevOps, allez à **Paramètres de l'Organisation** (en bas à gauche)
2. Sélectionnez **Pools d'agents**
3. Cliquez sur **Ajouter un pool**
4. Remplissez les champs:
   - Type de pool: **Auto-hébergé**
   - Nom: **OnPremAgentPool**
   - Description: "Pool d'agents on-premises pour formation"
5. Cliquez sur **Créer**

**Temps estimé**: 5 minutes

### Étape 2: Générer un Jeton d'Accès Personnel (PAT)
1. Dans Azure DevOps, cliquez sur votre profil (en haut à droite)
2. Sélectionnez **Jetons d'accès personnel**
3. Cliquez sur **+ Nouveau jeton**
4. Remplissez les informations:
   - Nom: **AgentFormation**
   - Organisation: Sélectionnez votre organisation
   - Expiration: **90 jours** (ou selon votre politique)
   - Portée: Sélectionnez **Pools d'agents** (lire et gérer)
5. Cliquez sur **Créer**

**⚠️ ATTENTION**: Le PAT n'est affiché qu'une seule fois!
- **Copiez-le immédiatement** et conservez-le dans un endroit sûr
- Vous en aurez besoin à l'étape 4

**Temps estimé**: 5 minutes

### Étape 3: Télécharger et Préparer l'Agent

#### Pour Windows:
1. Retournez à **Paramètres** → **Pools d'agents** → **OnPremAgentPool**
2. Cliquez sur **Nouvel agent**
3. Sélectionnez **Windows**
4. Cliquez sur **Télécharger**
5. Attendez la fin du téléchargement (fichier zip)
6. Créez un dossier sur votre machine:
   ```powershell
   New-Item -Type Directory -Path "C:\agents" -Force
   cd C:\agents
   ```
7. Extrayez le fichier téléchargé dans ce dossier

#### Pour Linux:
1. Même processus, mais sélectionnez **Linux**
2. Créez le dossier:
   ```bash
   sudo mkdir -p /opt/agents
   cd /opt/agents
   ```
3. Extrayez le fichier

**Temps estimé**: 10 minutes

### Étape 4: Configurer l'Agent

#### Configuration sur Windows:
1. Ouvrez PowerShell en tant qu'administrateur
2. Naviguez vers le dossier de l'agent:
   ```powershell
   cd C:\agents
   ```
3. Exécutez le script de configuration:
   ```powershell
   .\config.cmd
   ```
4. Répondez aux questions interactives:

```
Serveur Azure DevOps:
> https://dev.azure.com/[VotrOrganisation]

Type d'authentification:
> PAT

Jeton:
> [Collez votre PAT ici]

Pool d'agents:
> OnPremAgentPool

Nom de l'agent:
> [NomDeLaMachine]

Dossier de travail:
> _work (appuyez sur Entrée pour accepter)

Exécuter en tant que service?
> Y (Oui, recommandé)

Compte de service:
> [Sélectionnez selon votre configuration]
```

#### Configuration sur Linux:
1. Ouvrez un terminal
2. Naviguez vers le dossier:
   ```bash
   cd /opt/agents
   ```
3. Donnez des permissions d'exécution:
   ```bash
   chmod +x config.sh
   ```
4. Exécutez le script:
   ```bash
   ./config.sh
   ```
5. Suivez les mêmes étapes que pour Windows

**Temps estimé**: 15 minutes

### Étape 5: Démarrer l'Agent en Tant que Service

#### Pour Windows:
1. L'agent s'est installé en tant que service lors de la configuration
2. Vérifiez que le service est démarré:
   ```powershell
   Get-Service | Where-Object {$_.Name -like "*vstsagent*"}
   ```
3. Vous devriez voir un service actif

#### Pour Linux:
1. Installez le service:
   ```bash
   sudo /opt/agents/svc.sh install agent-user
   ```
2. Démarrez le service:
   ```bash
   sudo /opt/agents/svc.sh start
   ```
3. Vérifiez l'état:
   ```bash
   sudo /opt/agents/svc.sh status
   ```

**Temps estimé**: 5 minutes

### Étape 6: Vérifier l'Enregistrement de l'Agent
1. Retournez à Azure DevOps
2. Allez à **Paramètres** → **Pools d'agents** → **OnPremAgentPool**
3. Vous devriez voir votre agent listé
4. Vérifiez l'état: **En ligne** (vert)
5. Cliquez sur l'agent pour voir ses **Capacités** détectées automatiquement:
   - Système d'exploitation
   - Version du runtime
   - Outils disponibles (Git, Node.js, etc.)

**Points à vérifier**:
- État: **En ligne** (pas Hors ligne)
- Agent actif
- Capacités détectées (au moins quelques-unes)

**Temps estimé**: 5 minutes

### Étape 7: Créer un Pipeline de Test
1. Retournez à votre projet dans Azure DevOps
2. Allez à **Repos** et ouvrez votre fichier **azure-pipelines.yml**
3. Modifiez le pool pour utiliser l'agent on-premises:

```yaml
trigger:
  - main

pool:
  name: OnPremAgentPool
  demands: []

stages:
- stage: TestOnPremAgent
  displayName: 'Test Agent On-Premises'
  jobs:
  - job: ValidateAgent
    displayName: 'Valider l''Agent'
    steps:
    - script: |
        echo "Agent on-premises fonctionne!"
        echo "Système d'exploitation:"
        hostname
      displayName: 'Vérifier l''Agent'
    
    - script: |
        echo "Version de Git:"
        git --version
      displayName: 'Vérifier Git'
    
    - script: |
        echo "Répertoire de travail:"
        pwd
      displayName: 'Vérifier Répertoire'
```

4. Commitez et poussez:
   ```bash
   git add .
   git commit -m "Configuration du pipeline pour l'agent on-premises"
   git push origin main
   ```

**Temps estimé**: 5 minutes

### Étape 8: Exécuter et Observer
1. Retournez à **Pipelines** et lancez le pipeline
2. Attendez son exécution
3. Observez:
   - Le pipeline s'exécute sur **OnPremAgentPool**
   - Les étapes s'exécutent sans erreur
   - Votre agent on-premises a traité le job

**Temps estimé**: 5-10 minutes

### Étape 9: Configurer les Capacités de l'Agent (Avancé)
1. Si vous avez besoin d'ajouter des capacités personnalisées:
2. Sur le dossier de l'agent, éditez: `.agent/capabilities.json` (Windows) ou `.agent/capabilities` (Linux)
3. Exemple:
   ```json
   {
     "Agent.OS": "Windows_NT",
     "Agent.OSVersion": "10.0.19041",
     "Agent.Version": "2.x.x",
     "CustomCapability": "SpecialTool"
   }
   ```
4. Redémarrez l'agent:
   ```powershell
   # Windows
   Restart-Service "*vstsagent*"
   
   # Linux
   sudo /opt/agents/svc.sh stop
   sudo /opt/agents/svc.sh start
   ```

**Temps estimé**: 10 minutes (optionnel)

## Résultats Attendus

À la fin de ce laboratoire, vous devriez:
- ✅ Avoir créé un pool d'agents personnalisé
- ✅ Avoir généré un jeton d'accès personnel
- ✅ Avoir installé et enregistré un agent auto-hébergé
- ✅ Avoir vérifié l'état "En ligne" de l'agent
- ✅ Avoir exécuté un pipeline sur l'agent on-premises
- ✅ Avoir compris la gestion des pools d'agents

## Livrables

1. **Capture d'écran 1**: Pool d'agents "OnPremAgentPool" créé
2. **Capture d'écran 2**: Agent listé et en ligne dans le pool
3. **Capture d'écran 3**: Capacités détectées de l'agent
4. **Capture d'écran 4**: Pipeline exécuté sur l'agent on-premises
5. **Capture d'écran 5**: Logs de l'exécution réussie

## Tableau Comparatif: Agents Hébergés vs Auto-Hébergés

| Aspect | Hébergé Microsoft | Auto-Hébergé |
|--------|------------------|--------------|
| **Gestion** | Microsoft | Vous-même |
| **Coût** | Inclus (limité) | Infrastructure + maintenance |
| **Personnalisation** | Limitée | Complète |
| **Performance** | Standard | Selon votre hardware |
| **Disponibilité** | Réseau Microsoft | Votre réseau |
| **Sécurité** | Partagé | Isolé |
| **Mises à jour** | Automatiques | Manuelles |

## Bonnes Pratiques

1. **Nommage clair**: Utilisez des noms significatifs pour les agents
2. **Monitoring**: Surveillez régulièrement l'état des agents
3. **Maintenance**: Mettez à jour les agents régulièrement
4. **Sécurité**: Conservez le PAT en toute sécurité
5. **Documentation**: Documentez votre configuration d'agent

## Dépannage Courant

### L'agent reste "Hors ligne"
- **Cause**: Problème de connectivité ou d'authentification
- **Solution**: Vérifiez le PAT, la connectivité réseau, le pare-feu

### "Le pool d'agents est introuvable"
- **Cause**: Mauvais nom de pool dans le pipeline
- **Solution**: Vérifiez le nom exact du pool (sensible à la casse)

### L'agent s'arrête après redémarrage
- **Cause**: Service non configuré correctement
- **Solution**: Réexécutez la configuration avec permissions d'administrateur

### Erreur "Access Denied" lors de l'enregistrement
- **Cause**: PAT expiré ou permissions insuffisantes
- **Solution**: Générez un nouveau PAT avec les bonnes permissions

## Points Clés à Retenir

1. **Pool d'agents**: Conteneur logique pour grouper les agents
2. **PAT**: Nécessaire pour l'authentification sécurisée
3. **Service système**: L'agent s'exécute continûment en arrière-plan
4. **Capacités automatiques**: Azure DevOps détecte automatiquement les outils disponibles
5. **Flexibilité**: Les agents on-premises permettent plus de contrôle et de customisation

## Étapes Suivantes

Une fois ce laboratoire terminé:
- LAB 5.1: Implémenter les tâches et templates
- LAB 6.1: Configurer les permissions et portes d'approbation
- Déployer plusieurs agents pour la haute disponibilité

---

**Durée totale du laboratoire**: ~60 minutes
**Heure de lab supplémentaire**: Pour configurer plusieurs agents et mettre à jour les capacités
