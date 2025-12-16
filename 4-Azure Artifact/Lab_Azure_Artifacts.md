# Lab Azure Pipeline : Gestion des Artefacts Maven avec Azure Artifacts

## Objectifs du Lab

- Configurer un projet Java/Maven pour utiliser Azure Artifacts
- Créer un Feed Maven privé dans Azure DevOps
- Configurer l'authentification Maven (settings.xml et pom.xml)
- Créer un pipeline CI pour builder et publier un artefact (.jar/.war) dans le Feed
- Créer un pipeline CD pour consommer l'artefact et le déployer sur Azure Web App

---

## 0. Comprendre le Contexte : Les Objectifs "Cachés" de ce Lab

### La Grande Différence avec les Labs Précédents

Ce lab introduit un concept fondamental : **La gestion du cycle de vie des composants logiciels**, pas juste le déploiement d'une application.

#### Ce que les labs précédents faisaient

Dans les labs **Web App** (Node.js, .NET), **SQL Server**, et **Key Vault**, on utilisait **Pipeline Artifacts** :
- Construire → Empacker dans un ZIP → Déployer immédiatement
- L'artefact était **temporaire**, lié à l'exécution du build (30 jours max), puis supprimé
- Objectif : Passer rapidement du Code au Déploiement

#### Ce que ce lab fait différemment

Ici, on utilise **Azure Artifacts** (un vrai repository) :
- Construire → Publier dans un **Feed privé versionné** → Garder indéfiniment → Déployer quand on veut
- L'artefact est **permanent**, archivé avec une version (ex: `1.0.0`, `1.2.5-SNAPSHOT`)
- Objectif : Gérer une **bibliothèque privée d'entreprise** (votre propre "Maven Central")

### Les Bénéfices Concrets

#### 1. Créer une "Bibliothèque Privée" (Private Repository)
- L'équipe A développe un module de sécurité `.jar` → Publie v1.0.0 dans Azure Artifacts
- L'équipe B a besoin du même module → Elle l'ajoute simplement dans son `pom.xml`
- **Sans code source** ! Juste le package binaire, sécurisé, dans votre système privé

#### 2. La Gestion des Versions (Versioning et Traçabilité)
- Version `1.0.0` est **gravée dans le marbre** une fois publiée
- Bug en production sur v1.2.0 ? → Le pipeline peut redéployer exactement v1.0.0 stockée dans le Feed
- **Traçabilité totale** : Qui a déployé quelle version, quand, d'où ?

#### 3. La Sécurité de la Supply Chain
- Seuls les pipelines autorisés peuvent publier des packages
- Chaque publication est loggée et auditée
- Impossible qu'un développeur ait "oublié" une malveillance dans le code publié

### Analogie Concrète

| Concept | Lab Précédent (Pipeline Artifacts) | Ce Lab (Azure Artifacts) |
| :--- | :--- | :--- |
| **Analogie** | Cuisiner un plat et le servir immédiatement | Usine de mise en conserve (bouteilles étiquetées) |
| **Durée de vie** | Temporaire (consommation immédiate) | Permanent (archive indéfinie) |
| **Format** | Dossier ZIP ou fichiers en vrac | Package versionné (ex: `myapp-1.0.2.jar`) |
| **Accessibilité** | Pipeline → Deploy seulement | Toute l'entreprise peut l'utiliser |
| **Cas d'usage** | Déployer aujourd'hui | Partager, archiver, revendre à d'autres équipes |

### Tableau Comparatif Détaillé

| Caractéristique | Pipeline Artifacts | Azure Artifacts (ce lab) |
| :--- | :--- | :--- |
| **Mécanisme** | `PublishBuildArtifacts` task | `Maven deploy` (vers un Feed) |
| **Stockage** | Temporaire (30 jours par défaut) | **Permanent** |
| **Accessibilité** | Lié au "Run" du pipeline | Accessible globalement (via Feed) |
| **Versioning** | Un seul build = une version | Versioning explicite (1.0, 1.1, 2.0, etc.) |
| **Utilisation** | Passer build → deploy | Partager entre équipes et projets |
| **Sécurité** | Logique simple | Authentification (PAT, MavenAuthenticate) |

---

## 1. Schéma de l'Architecture

```
┌──────────────────────┐          ┌────────────────────────────┐
│    Azure Artifacts   │          │    Azure DevOps Pipeline   │
│   (Maven Feed)       │          │                            │
│  ┌────────────────┐  │ Publish  │  ┌────────────────────┐    │
│  │ Artifact:      │◀─┼──────────┼──│ Task: Maven Deploy │    │
│  │ app-1.0.0.jar  │  │ (CI)     │  │ (settings.xml + PAT)    │
│  └────────────────┘  │          │  └────────────────────┘    │
│          │           │          │                            │
└──────────┼───────────┘          │                            │
           │ Consume              │                            │
           │ (CD)                 │                            │
           ▼                      │                            │
┌──────────────────────┐          │  ┌────────────────────┐    │
│   Azure Web App      │◀─────────┼──│ Task: AzureWebApp  │    │
│                      │  Deploy  │  │ (Download Artifact)     │
│                      │          │  └────────────────────┘    │
└──────────────────────┘          └────────────────────────────┘
```

### Description du flux

1. **Stockage** : Le code source Java est stocké dans Azure Repos.
2. **Build & Publish (CI)** : Le pipeline CI compile le code, exécute les tests et publie l'artefact généré (`.jar`) dans un **Feed Azure Artifacts** privé et sécurisé.
3. **Deploy (CD)** : Le pipeline CD (ou le stage de déploiement) récupère l'artefact depuis le Feed et le déploie sur une Azure Web App.

---

## 2. Prérequis et Configuration des Ressources

### 2.1 Créer un Projet Azure DevOps et le Feed

1. **Créer le Projet** : Dans Azure DevOps, créez un projet nommé `AppWebMaven`.
2. **Créer le Feed Artifacts** :
   - Allez dans **Artifacts** dans le menu de gauche.
   - Cliquez sur **+ Create Feed**.
   - Nom : `AppWebMavenFeed`.
   - Visibilité : **Project scoped** (ou Organization si vous préférez).
   - Décochez "Include packages from common public sources" (sauf si vous voulez proxy Maven Central).
   - Cliquez sur **Create**.

### 2.2 Préparer le Code Source

1. **Cloner le Projet** : Importez ou clonez le projet Java Spring Boot fourni (`Java-SpringBoot-Maven`) dans votre Azure Repo.
   https://github.com/hajjiBel/simple-node-js
3. **Configurer `pom.xml`** :
   - Dans Azure Artifacts, cliquez sur **Connect to Feed**.
   - Sélectionnez **Maven**.
   - Copiez la section `<repositories>` et `<distributionManagement>` fournie.
   - Collez-ces sections dans votre fichier `pom.xml` à la racine du projet.

   ```xml
   <!-- Exemple (à adapter avec vos valeurs) -->
   <repositories>
     <repository>
       <id>AppWebMavenFeed</id>
       <url>https://pkgs.dev.azure.com/votre-org/AppWebMaven/_packaging/AppWebMavenFeed/maven/v1</url>
       <releases><enabled>true</enabled></releases>
       <snapshots><enabled>true</enabled></snapshots>
     </repository>
   </repositories>
   <distributionManagement>
     <repository>
       <id>AppWebMavenFeed</id>
       <url>https://pkgs.dev.azure.com/votre-org/AppWebMaven/_packaging/AppWebMavenFeed/maven/v1</url>
     </repository>
   </distributionManagement>
   ```

### 2.3 Créer l'Azure Web App

1. Accédez au **Portail Azure**.
2. Créez une **Web App** :
   - Nom : `webapp-maven-lab-[votre-nom]`.
   - Runtime : **Java 8** (ou 11/17 selon votre projet) avec **Tomcat** ou **Java SE**.
   - OS : **Linux**.
   - Plan : **F1 (Free)**.

---

## 3. Configuration du Pipeline CI (Build & Publish)

Créez un fichier `azure-pipelines.yml` à la racine. L'astuce est d'utiliser la tâche `MavenAuthenticate@0` pour gérer l'authentification au Feed sans avoir à gérer manuellement un fichier `settings.xml` avec un PAT.

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  # Nom de votre feed (ProjectName/FeedName si project-scoped)
  artifactFeed: 'AppWebMaven/AppWebMavenFeed' 
  azureSubscription: 'ServiceConnection-Lab'
  webAppName: 'webapp-maven-lab-[votre-nom]'

stages:
# ========================================================
# STAGE 1: CI (BUILD & PUBLISH TO ARTIFACTS)
# ========================================================
- stage: CI
  displayName: 'Build and Publish Artifact'
  jobs:
  - job: Build
    steps:
    # 1. Authentification Maven (Génère un settings.xml temporaire)
    - task: MavenAuthenticate@0
      displayName: 'Maven Authenticate'
      inputs:
        artifactsFeeds: 'AppWebMavenFeed' # Nom simple du feed

    # 2. Build & Deploy vers Azure Artifacts
    - task: Maven@3
      displayName: 'Maven Build & Deploy'
      inputs:
        mavenPomFile: 'pom.xml'
        goals: 'deploy' # 'deploy' pousse vers distributionManagement (Azure Artifacts)
        options: '-DskipTests=true' # Optionnel, pour accélérer le lab
        publishJUnitResults: false
        javaHomeOption: 'JDKVersion'
        jdkVersionOption: '1.8'
        mavenVersionOption: 'Default'
        mavenAuthenticateFeed: true # Redondant avec MavenAuthenticate mais bonne pratique
        effectivePomSkip: false
        sonarQubeRunAnalysis: false

    # 3. Publier aussi comme Pipeline Artifact (pour le CD immédiat)
    # (Optionnel si on ne veut que consommer depuis le Feed, mais utile pour le CD task)
    - task: CopyFiles@2
      displayName: 'Copy Artifact'
      inputs:
        SourceFolder: '$(system.defaultworkingdirectory)/target'
        Contents: '**/*.jar'
        TargetFolder: '$(build.artifactstagingdirectory)'

    - task: PublishBuildArtifacts@1
      displayName: 'Publish Pipeline Artifact'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'
```

---

## 4. Configuration du Pipeline CD (Deploy)

Ajoutez le stage de déploiement à la suite du fichier YAML.

```yaml
# ========================================================
# STAGE 2: CD (DEPLOY TO WEB APP)
# ========================================================
- stage: CD
  displayName: 'Deploy to Azure Web App'
  dependsOn: CI
  condition: succeeded()
  jobs:
  - deployment: Deploy
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          # 1. Télécharger l'artefact (Depuis le Pipeline Artifact pour simplicité)
          - download: current
            artifact: 'drop'

          # 2. Déployer vers Azure Web App
          - task: AzureWebApp@1
            displayName: 'Azure Web App Deploy'
            inputs:
              azureSubscription: '$(azureSubscription)'
              appType: 'webAppLinux'
              appName: '$(webAppName)'
              package: '$(Pipeline.Workspace)/drop/**/*.jar'
              # Commande de démarrage pour Spring Boot (si Java SE)
              # startupCommand: 'java -jar /home/site/wwwroot/*.jar --server.port=80' 
```

---

## 5. Exécution et Validation

1. **Commit & Push** : Poussez le code avec le `pom.xml` modifié et le fichier YAML.
2. **Vérification CI** :
   - Le pipeline se lance.
   - Vérifiez la tâche `Maven Deploy`.
   - Allez dans le menu **Artifacts** : vous devriez voir votre package (ex: `com.example:demo`) avec la version `1.0.0` (ou `1.0.0-SNAPSHOT`).
3. **Vérification CD** :
   - Le stage CD déploie l'application.
   - Vérifiez l'URL de votre Web App.

---

## 6. Points Clés & Troubleshooting

| Erreur | Solution |
|--------|----------|
| **401 Unauthorized (Maven)** | Assurez-vous d'utiliser la tâche `MavenAuthenticate@0` **avant** la tâche `Maven@3`. Vérifiez que le nom du feed dans `pom.xml` correspond à celui dans la tâche. |
| **Snapshot vs Release** | Si votre version dans `pom.xml` finit par `-SNAPSHOT`, Maven cherchera à publier dans le repo `snapshotRepository`. Assurez-vous que votre `distributionManagement` dans le `pom.xml` a bien les deux sections (`repository` et `snapshotRepository`) pointant vers le même feed (ou différents si configuré). |
| **Settings.xml** | Ne créez pas manuellement de `settings.xml` dans le repo si vous utilisez `MavenAuthenticate`. Cette tâche en génère un dynamiquement avec un token valide pour la durée du build. |
| **Web App Error** | Si l'application ne démarre pas (erreur 5xx), vérifiez les logs de la Web App via Kudu (`https://<webapp-name>.scm.azurewebsites.net`) ou le Log Stream dans le portail Azure. Souvent lié à un port incorrect (Spring Boot par défaut 8080, Azure attend 80). Ajoutez `server.port=80` dans `application.properties` ou dans la commande de démarrage. |

---

## 7. Récapitulatif des Compétences Acquises

À la fin de ce lab, vous maîtriserez :

✅ **Gestion des Artefacts** : Créer et maintenir une bibliothèque privée  
✅ **Versioning** : Comprendre les versions SNAPSHOT vs Release  
✅ **Authentification Sécurisée** : Utiliser `MavenAuthenticate` pour masquer les credentials  
✅ **Cycle de Vie Complet** : Build → Archive → Déploiement contrôlé  
✅ **Supply Chain Security** : Publier uniquement via pipeline autorisé  

**Niveau de Maturité DevOps** : Vous êtes passé de "Déployer du code" à "Gérer des composants logiciels réutilisables"
