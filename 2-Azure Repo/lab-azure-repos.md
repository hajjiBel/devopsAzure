# LAB Azure Repos

---

## 📋 TABLE DES MATIÈRES

1. [Module 1 : Présentation d'Azure Repos](#module-1--présentation-dazure-repos)
2. [Module 2 : Navigation et Interface](#module-2--navigation-et-interface)
3. [Module 3 : Clone et Push - Premiers Pas](#module-3--clone-et-push---premiers-pas)
4. [Module 4 : Politiques de Branche Avancées](#module-4--politiques-de-branche-avancées)
5. [Module 5 : Pull Requests Complètes](#module-5--pull-requests-complètes)


---

## 🎯 MODULE 1 : PRÉSENTATION D'AZURE REPOS

### Qu'est-ce qu'Azure Repos ?

**Azure Repos** est le service de gestion de version intégré à Azure DevOps. Il offre des capacités complètes de contrôle de version pour gérer votre code source de manière centralisée et sécurisée, peu importe l'ampleur de votre projet.

Un système de gestion de version est un **logiciel fondamental** qui vous permet de :
- **Tracer toutes les modifications** apportées à votre code
- **Maintenir un historique complet** de l'évolution du projet
- **Collaborer efficacement** au sein d'une équipe
- **Revenir à des versions antérieures** si nécessaire

---

## 🔄 Systèmes de Contrôle de Version

Azure Repos supporte deux approches principales de gestion de version :

### 1️⃣ Git (Contrôle de Version Distribué) ⭐ RECOMMANDÉ

**Caractéristiques principales** :
- **Système décentralisé** : Chaque développeur dispose d'une copie locale complète du référentiel
- **Indépendance** : Permet de travailler hors ligne avec toute l'historique disponible localement
- **Performance** : Opérations locales ultrarapides (commit, branches, fusions)
- **Flexibilité** : Gestion avancée des branches et des flux de travail
- **Scalabilité** : Parfaitement adapté aux petites et grandes équipes

**Pourquoi Git domine le secteur** :
- ✅ **Norme industrielle** acceptée et utilisée par la majorité des développeurs
- ✅ **Écosystème riche** d'outils et d'intégrations
- ✅ **Communauté active** avec documentation abondante
- ✅ **Compatibilité universelle** avec tous les environnements DevOps modernes
- ✅ **Workflows avancés** (feature branches, pull requests, code reviews)

### 2️⃣ TFVC (Team Foundation Version Control) - Centralisé

**Caractéristiques** :
- Système **centralisé** avec un serveur principal unique
- Chaque développeur récupère uniquement les fichiers sur lesquels il travaille
- Meilleur pour les projets avec des fichiers volumineux (binaires)
- Moins flexible que Git pour les workflows modernes

**Quand utiliser TFVC** :
- Projets legacy avec dépendances critiques sur le serveur central
- Environnements exigeant un contrôle d'accès très strict
- Travail principalement sur fichiers binaires volumineux

---

## 💡 Avantages d'Azure Repos avec Git

### Intégration Azure DevOps Complète
- **Pipelines automatisés** : Déclenchez directement vos pipelines CI/CD depuis Git
- **Boards liés** : Connectez vos commits et pull requests aux éléments de travail
- **Policies avancées** : Imposez des règles de qualité avant la fusion de code

### Gestion des Branches Professionnelle
- **Protection des branches** : Empêchez les push directs sur main/master
- **Stratégies de branche** : Build automatiques, approbations obligatoires
- **Nommage standardisé** : Conventions feature/, bugfix/, hotfix/

### Pull Requests Puissantes
- **Code reviews intégrées** : Approvals et commentaires threading
- **Validation automatique** : Tests et vérifications avant fusion
- **Traçabilité complète** : Historique détaillé de chaque changement

### Sécurité et Conformité
- **Contrôle d'accès granulaire** : Par branche, par équipe, par ressource
- **Audit trail complet** : Qui a changé quoi et quand
- **Secrets management** : Intégration avec Azure Key Vault

### Collaboration Distribuée
- **Travail hors ligne** : Commits sans connexion réseau
- **Fusion intelligente** : Gestion efficace des conflits
- **Blame et history** : Trouver facilement l'origine d'une modification

---

## 🏗️ Architecture d'Azure Repos

```
┌─────────────────────────────────────────────────────────┐
│           Azure DevOps Organization                      │
│                                                           │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Azure DevOps Project                    │   │
│  │                                                   │   │
│  │  ┌──────────────────────────────────────────┐   │   │
│  │  │  Azure Repos (Git Repository)             │   │   │
│  │  │                                            │   │   │
│  │  │  📁 Main Branch (Production-Ready)        │   │   │
│  │  │     ├── v1.0 release tag                  │   │   │
│  │  │     └── v1.1 release tag                  │   │   │
│  │  │                                            │   │   │
│  │  │  📁 Develop Branch (Intégration)          │   │   │
│  │  │     └── commit history                    │   │   │
│  │  │                                            │   │   │
│  │  │  📁 Feature Branches (Travail en cours)   │   │   │
│  │  │     ├── feature/login-module              │   │   │
│  │  │     └── feature/payment-integration       │   │   │
│  │  │                                            │   │   │
│  │  └──────────────────────────────────────────┘   │   │
│  │                                                   │   │
│  │  Connecté à :                                    │   │
│  │  • Azure Pipelines (CI/CD)                       │   │
│  │  • Azure Boards (Work Items)                     │   │
│  │  • Azure Artifacts (Package Management)         │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 🔑 Concepts Fondamentaux de Git

### Repository (Dépôt)
Un **repository** est l'espace de stockage central où votre code et son historique complet sont sauvegardés. Chaque développeur a une copie locale complète du repository.

```
Repository Centralisé (Azure Repos)
        ↑
        │ pull, push
        │ 
   ┌────┴────┐
   │   Git   │
   └────┬────┘
        ↓
   Dépôt Local (votre ordinateur)
   - Code source
   - Historique complet
   - Branches
```

### Branch (Branche)
Une **branche** est une copie indépendante du code permettant de travailler en parallèle sans affecter le code principal. Les branches permettent l'isolation des changements jusqu'à leur intégration.

**Branches essentielles** :
- `main` ou `master` → Code en production
- `develop` → Intégration continue des features
- `feature/*` → Nouvelles fonctionnalités
- `bugfix/*` → Corrections de bugs

### Commit
Un **commit** est une sauvegarde d'un ensemble cohérent de modifications avec un message descriptif. C'est l'unité de base de l'historique du projet.

```
Timeline des Commits
─────────────────────────────────────
│ 001: Initial project setup
│ 002: Add login functionality
│ 003: Fix authentication bug
│ 004: Add user dashboard
│ 005: Optimize database queries ← Current
─────────────────────────────────────
```

### Pull Request (Demande de Pull)
Une **pull request** (PR) est une demande formelle de fusionner les changements d'une branche à une autre. Elle permet les code reviews et validations avant l'intégration.

**Cycle d'une Pull Request** :
```
1. Créer une feature branch
2. Pusher les commits
3. Ouvrir une Pull Request
4. Code review (approbations)
5. Tests automatiques (pipelines)
6. Fusionner dans main/develop
7. Supprimer la branche
```

---

## 🖥️ MODULE 2 : NAVIGATION ET INTERFACE

### Objectif
Maîtriser l'interface d'Azure Repos et ses différentes sections.

### Structure de l'Interface

#### Section 1 : Files (Fichiers)
C'est votre **point d'entrée principal** dans Azure Repos.

**Caractéristiques :**
- Deux volets d'affichage :
  1. **Volet de gauche** : Arborescence des fichiers et dossiers (vue en arbre)
  2. **Volet de droite** : Contenu du fichier actuellement sélectionné

**Opérations disponibles :**
- 📁 Naviguer dans les dossiers
- 📝 Voir le contenu des fichiers
- ➕ Ajouter des fichiers
- ➕ Ajouter des dossiers
- 📤 Importer des fichiers
- 🔄 Renommer des fichiers
- 🗑️ Supprimer des fichiers
- 💾 Télécharger le projet en ZIP
- 🔀 Fork du projet

**Bonnes Pratiques :**
- Utilisez plutôt un **IDE ou Git CLI** pour éditer les fichiers
- L'interface web est surtout pour la **consultation**
- Travaillez **en local** d'abord, puis poussez sur le serveur

#### Section 2 : History (Historique)
Permet de voir les **différents événements** relatifs à un fichier ou dossier spécifique.

**Informations affichées :**
- 👤 Qui a modifié le fichier
- 📅 Quand il a été modifié
- 💬 Message du commit
- 🔗 ID du commit

#### Section 3 : Branches
Vue complète de **toutes les branches** du dépôt.

**Informations visibles :**
- 🌳 **Branche par défaut** (généralement "main" ou "master")
- 🔀 **Branches de développement** (ex: develop, feature/*, etc.)
- 🏷️ **Tags** créés (ex: v1.0.0, v0.0.1)
- 📅 Date de création de chaque branche
- 👤 Auteur de la branche

**Actions possibles :**
- Créer une nouvelle branche
- Voir la liste complète des branches
- Consulter les tags

#### Section 4 : Commits
**Historique complet de tous les commits** effectués.

**Informations disponibles :**
- 🆔 ID du commit
- 💬 Message du commit
- 👤 Auteur du commit
- 📅 Date du commit
- 🔗 Lien vers la branche associée
- 🔀 Indication des merges (fusions)

**Fonctionnalités :**
- Recherche par ID de commit
- Filtrage par message
- Voir qui a fait quelle modification

#### Section 5 : Pushes
**Historique de tous les pushs** effectués vers le serveur.

**Données disponibles :**
- 📅 Quand le push a eu lieu
- 👤 Qui a fait le push
- 🔀 Quelle branche a été poussée
- 📊 Nombre de commits dans le push

#### Section 6 : Pull Requests
**Liste de toutes les pull requests** (PR) du projet.

**Pour chaque PR, vous verrez :**
- 📖 Titre et description
- 🔀 Branche source et branche destination
- ✅ Statut de la PR (ouverte, approuvée, fusionnée, rejetée)
- 👤 Auteur et reviewers
- 📝 Commentaires et feedback

**Onglets dans une PR :**
- **Overview** : Vue globale et status
- **Files** : Fichiers modifiés avec diff
- **Updates** : Historique des modifications
- **Commits** : Liste des commits associés

### Section 7 : Setup Build (Configuration de Build)
Pour générer les **pipelines Azure Pipelines** (CI/CD) liés au projet.

### Étapes Pratiques

#### Étape 1 : Explorer la Section Files
1. Allez sur **Repos** → **Files**
2. Observez l'arborescence des fichiers
3. Cliquez sur différents fichiers pour voir leur contenu
4. Notez les opérations disponibles (clone, fork, upload, etc.)

#### Étape 2 : Consulter l'Historique
1. Allez sur **Repos** → **History**
2. Observez qui a modifié les fichiers
3. Notez les dates et messages de commit

#### Étape 3 : Examiner les Branches
1. Allez sur **Repos** → **Branches**
2. Identifiez la branche par défaut (main)
3. Observez les autres branches (develop, feature/*, etc.)
4. Notez les tags existants

#### Étape 4 : Parcourir les Commits
1. Allez sur **Repos** → **Commits**
2. Lisez les messages de commit
3. Identifiez les fusiones (merges) dans l'historique
4. Notez qui a fait chaque commit

#### Étape 5 : Vérifier les Pull Requests
1. Allez sur **Repos** → **Pull Requests**
2. Observez les PR existantes
3. Ouvrez une PR complète pour voir sa structure
4. Consultez les commentaires et approbations

### Points Clés à Mémoriser
- L'interface Files est votre **tableau de bord principal**
- Chaque section offre une **vue différente** des données
- L'historique est **complet et inaltérable**
- Les PR centralisent toute la **collaboration**

---

## 🚀 MODULE 3 : CLONE ET PUSH - PREMIERS PAS

### Objectif
Maîtriser les opérations fondamentales : cloner un dépôt, créer des fichiers, committer et pousser.

### Contexte Théorique

#### Clonage d'un Dépôt
Le **clonage** crée une copie **complète** du dépôt distant sur votre machine locale.

**Deux méthodes disponibles :**

**1. HTTPS (Plus simple, mais moins sécurisé)**
```bash
git clone https://dev.azure.com/organization/project/_git/repository
```
- ✅ Fonctionne partout
- ✅ Pas de configuration de clé nécessaire
- ❌ Demande le mot de passe à chaque push
- ❌ Moins recommandé pour un usage professionnel

**2. SSH (Plus complexe, mais plus sécurisé)**
```bash
git clone git@ssh.dev.azure.com:v3/organization/project/repository
```
- ✅ Sécurisé (clés publique/privée)
- ✅ Pas besoin de mot de passe à chaque fois
- ✅ Recommandé pour les équipes
- ❌ Requiert la configuration initiale de clés SSH

#### Options de Clonage

**Option 1 : Cloner un dépôt existant**
- Clone une **copie complète** du dépôt
- Vous obtenez **tout l'historique**
- Idéal pour rejoindre un projet existant

**Option 2 : Importer depuis un autre dépôt**
- Importe du code d'un **GitHub**, d'un **GitLab**, ou d'un autre **Azure Repo**
- Azure Repos **absorbe** le projet
- Utile pour **migrer des projets**

#### Authentication Sécurisée

**Clés SSH (Recommandé)**
1. Générez une paire de clés publique/privée
2. Fournissez la **clé publique** à Azure DevOps
3. Gardez votre **clé privée** en sécurité
4. Configurez votre client Git pour utiliser SSH

**Personal Access Tokens (PATs)**
- Utile pour les **scripts** et **automatisations**
- À utiliser à la place du mot de passe
- Plus sécurisé que les mots de passe en clair

**Webhooks**
- Pour connecter Azure Repos à d'autres services
- Déclenche des actions à chaque événement Git

### Étapes Pratiques

#### Étape 1 : Copier le Lien de Clonage

**Sur Azure DevOps :**
0. Allez sur **Repos** → Initializer un dépôt Git avec un fichier README
1. Allez sur **Repos** → **Files**
2. Cliquez sur le bouton **Clone**
3. Choisissez **HTTPS** ou **SSH**
4. Copiez l'URL

**Exemple HTTPS :**
```
https://dev.azure.com/myorg/myproject/_git/myrepo
```

#### Étape 2 : Cloner le Dépôt Localement

**Ouvrez votre terminal/Git Bash :**

```bash
# Créez un répertoire pour votre projet
mkdir mes-projets
cd mes-projets

# Clonez le dépôt
git clone https://dev.azure.com/myorg/myproject/_git/myrepo
cd myrepo

# Vérifiez que vous êtes dans le bon répertoire
ls -la
```

**Résultat attendu :**
- Un nouveau dossier est créé
- Le contenu du dépôt y est copié
- Un dossier `.git` caché contient l'historique

#### Étape 3 : Vérifier la Configuration Git

```bash
# Vérifiez la branche actuelle
git branch

# Vérifiez l'URL distante
git remote -v

# Résultat attendu :
# origin https://dev.azure.com/myorg/myproject/_git/myrepo (fetch)
# origin https://dev.azure.com/myorg/myproject/_git/myrepo (push)
```

#### Étape 4 : Créer un Fichier de Test

```bash
# Créez un fichier simple
echo "Ceci est mon premier test" > test.txt

# Vérifiez l'état de Git
git status

# Vous devriez voir :
# Untracked files:
#   test.txt
```

#### Étape 5 : Ajouter et Committer

```bash
# Ajoutez le fichier à l'index (staging area)
git add test.txt

# Vérifiez à nouveau
git status
# Vous devriez voir :
# Changes to be committed:
#   new file: test.txt

# Committez avec un message descriptif
git commit -m "Ajout de fichier de test initial"

# Vérifiez le statut
git status
# Vous devriez voir :
# On branch master
# nothing to commit, working tree clean
```

#### Étape 6 : Voir l'Historique Local

```bash
# Affichage compact
git log --oneline

# Affichage détaillé
git log
```

#### Étape 7 : Configurer les Credentials (si nécessaire)

**Pour HTTPS :**
```bash
# Azure DevOps vous demandera un mot de passe
# Allez sur Azure DevOps → User Settings → Personal access tokens
# Créez un token avec accès à "Code (read & write)"
# Utilisez ce token comme mot de passe
```

#### Étape 8 : Pousser vers le Serveur

```bash
# Poussez les modifications vers la branche main/master
git push origin master

# Ou si vous êtes sur main :
git push origin main

# Entrez votre mot de passe ou token si demandé

# Résultat attendu :
# Enumerating objects: 3, done.
# Counting objects: 100% (3/3), done.
# Writing objects: 100% (3/3), 250 bytes | 250.00 KiB/s, done.
# Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
# To https://dev.azure.com/myorg/myproject/_git/myrepo
#  * [new branch]      master -> master
```

#### Étape 9 : Vérifier le Push sur Azure DevOps

1. Retournez à **Azure DevOps** → **Repos** → **Files**
2. Rafraîchissez la page (F5)
3. Vous devriez voir votre fichier `test.txt` apparu
4. Allez dans **Commits**
5. Vous devriez voir votre commit "Ajout de fichier de test initial"

#### Étape 10 : Vérifier l'Historique

```bash
# Affichage de l'historique local
git log --oneline

# Affichage avec plus de détails
git log --graph --oneline --all
```

### Points Clés à Mémoriser
- Le **clonage** télécharge l'**historique complet**
- SSH est plus sûr qu'HTTPS pour un usage professionnel
- Les commits sont d'abord locaux
- Le **push** envoie les commits au serveur
- L'historique est **traçable** et **immuable**

---

## 🛡️ MODULE 4 : POLITIQUES DE BRANCHE AVANCÉES

### Objectif
Maîtriser les politiques de branche pour améliorer la qualité du code et sécuriser les branches critiques.

### Contexte Théorique

Une **Pull Request (PR)** est une **demande formelle** de fusionner le code d'une branche vers une autre.

**Avantages :**
- ✅ **Révision de code** par les pairs
- ✅ **Validation** avant fusion
- ✅ **Collaboration** structurée
- ✅ **Traçabilité** complète
- ✅ **Documentation** automatique

#### Policies Disponibles dans Azure DevOps

**1. Minimum Number of Reviewers (Nombre minimum de reviewers)**
- Exiger qu'un certain nombre de personnes (ex: 2-5) approuvent avant fusion
- Options :
  - Rejeter les PRs non approuvées
  - Permettre au requêteur d'approuver ses propres changements
  - Exiger que les approbations soient récentes

**2. Require Linked Work Items (Liaison avec Work Items)**
- Exiger qu'une PR soit **liée à un work item**
- Garantit le **traçabilité** du travail
- Lie le code à la gestion de projet
- Très important pour les équipes quality-focused

**3. Require Comment Resolution (Résolution des Commentaires)**
- Exiger que tous les commentaires soient **résolus** avant fusion
- Garantit que les problèmes soulevés sont **adressés**
- Crée une **documentation** complète du code
- Améliore la communication

**4. Automatically Include Code Reviewers (Include automatique de reviewers)**
- Ajouter automatiquement des reviewers basés sur des critères :
  - Propriétaires du code
  - Historique des commits
  - Patterns de fichiers

**5. Require Successful Builds (Builds réussis)**
- Pipeline Azure Pipelines doit **réussir** avant fusion
- Garantit le bon **fonctionnement** du code
- Détecte les **compilations échouées**
- Très important pour la CI

**6. Enforce a Merge Strategy (Stratégie de fusion)**
- Limiter les types de merge possibles :
  - Merge commit (conserve l'historique)
  - Squash (compacte les commits)
  - Rebase (réécrit l'historique)

**7. Require Code Review Only (Code Review seulement)**
- Exiger une **approbation explicite** avant fusion
- Peut être conditionnée au rejet automatique après N jours

**8. Limit Commit Author to PR Creator (Auteur = Créateur)**
- Restreindre à ce que seul le créateur puisse merger

### Étapes Pratiques

#### Étape 1 : Accéder aux Branch Policies

1. Allez sur **Repos** → **Branches**
2. Trouvez la branche `develop` ou `main`
3. Cliquez sur les **trois points** (...)
4. Sélectionnez **Branch Policies**

#### Étape 2 : Activer le Nombre Minimum de Reviewers

**Configuration :**
1. Trouvez l'option **"Require a minimum number of reviewers"**
2. Activez-la avec **ON**
3. Définissez le nombre : **2** reviewers minimum
4. Options additionnelles :
   - ☐ "Allow requestors to approve their own changes" (généralement DÉCOCHÉ)
   - ☑️ "Prohibit the most recent pusher from approving their own changes"
5. Cliquez sur **Save**

**Effet :**
- Les PRs ne peuvent être fusionnées que si 2+ personnes ont approuvé
- Le requêteur ne peut **pas** approuver ses propres changements
- Cela **force la collaboration**

#### Étape 3 : Exiger la Liaison avec Work Items

**Configuration :**
1. Trouvez l'option **"Require linked work items"**
2. Activez-la avec **ON**
3. Cliquez sur **Save**

**Effet :**
- Une PR **ne peut pas être fusionnée** sans être liée à un work item
- Force les développeurs à **documenter** leur travail
- Crée une **traçabilité complète**


#### Étape 4 : Exiger la Résolution des Commentaires

**Configuration :**
1. Trouvez l'option **"Require resolution on comments"**
2. Activez-la avec **ON**
3. Options :
   - All comments must be resolved (recommandé)
   - Comments from code reviewers must be resolved
4. Cliquez sur **Save**

**Effet :**
- Chaque commentaire doit être **marqué comme résolu**
- Garantit que les **problèmes sont adressés**
- Crée une **documentation** automatique

#### Étape 5 : Exiger la Réussite du Build

**Configuration :**
1. Trouvez l'option **"Build validation"**
2. Activez-la avec **ON**
3. Sélectionnez votre **pipeline Azure Pipelines**
4. Options :
   - "Require successful build"
   - "Automatic approval if build succeeds"
5. Cliquez sur **Save**

**Effet :**
- Le build automatisé **doit réussir** avant fusion
- **Prévient** les codes cassés d'être mergés
- Très recommandé pour la qualité

#### Étape 6 : Limiter les Stratégies de Fusion

**Configuration :**
1. Trouvez l'option **"Enforce a merge strategy"**
2. Activez-la avec **ON**
3. Sélectionnez les stratégies autorisées :
   - ☑️ Basic merge (No-ff merge)
   - ☑️ Squash commit
   - ☐ Rebase and fast-forward
   - ☐ Rebase with merge commit
4. Cliquez sur **Save**

**Recommandations :**
- Pour conserver l'**historique complet** → Autorisez "Basic merge"
- Pour des PR simples → Autorisez "Squash"
- Pour le **cleanliness** → Limitez les options

### Étape 7 : Tester les Policies

#### Test 1 : Merger avec approbations

1. Créez une branche `feature/test-approvals` et poussez au moins un commit.
2. Ouvrez une **Pull Request** de `feature/test-approvals` vers `main`.
3. Ajoutez **2 reviewers** à la PR (membres du groupe de reviewers si une politique existe).
4. Demandez à ces 2 personnes d’ouvrir la PR, de vérifier les changements puis de cliquer sur **Approve**.
5. Vérifiez que, une fois les 2 approbations obtenues, le bouton **Complete** devient actif sur la PR.
6. Cliquez sur **Complete** (choisissez le type de merge si nécessaire), puis validez.
7. **Résultat attendu** : la PR passe à l’état **Completed** et les commits sont fusionnés dans la branche `main`.

---

#### Test 1 : Vérifier l’obligation de Work Item lié

1. Créez une branche `feature/no-workitem` et poussez au moins un commit.
2. Ouvrez une **Pull Request** de `feature/no-workitem` vers `main` **sans lier de Work Item** (ne pas associer de User Story, Bug ou Task).
3. Ajoutez des reviewers si la politique l’exige et obtenez les approbations nécessaires.
4. Essayez de cliquer sur **Complete** pour terminer la PR.
5. **Résultat attendu** : la complétion est bloquée et un message indique qu’un Work Item doit être lié (ex. “Requires linked work item”).
6. Liez un Work Item existant (ou créez-en un depuis la PR et associez-le).
7. Relancez l’action **Complete** sur la PR.
8. **Résultat attendu** : la PR est fusionnée avec succès et le Work Item est automatiquement lié à la PR.


### Configuration Recommandée par Niveau

**🟢 Débutant (Permissif)**
- Minimum 1 reviewer
- Pas de work item requis
- Pas de commentaires requis

**🟡 Intermédiaire (Équilibré)**
- Minimum 2 reviewers
- Work items liés recommandés
- Commentaires doivent être résolus
- Build validation obligatoire

**🔴 Avancé (Strict)**
- Minimum 3-5 reviewers
- Work items liés obligatoires
- Tous les commentaires résolus
- Build successful requis
- Specific merge strategy (squash)
- Auto-complete désactivé

### Points Clés à Mémoriser
- Les policies **protègent** les branches critiques
- Le nombre minimum de reviewers **force la collaboration**
- Les work items **assurent la traçabilité**
- Les policies **appliquent** la discipline de l'équipe

---


### Étapes Pratiques

#### Étape 1 : Créer une Branche Feature

```bash
# Basculez sur la branche main
git checkout main

# Créez une nouvelle branche
git checkout -b feature/new-header

# Vérifiez votre branche actuelle
git branch
# Vous devriez voir :
# * feature/new-header
#   main
```

#### Étape 2 : Faire des Modifications

**Modifiez un fichier (ex: index.html) :**
```html
<!-- Avant -->
<title>Old Training Academy</title>
<h1>Welcome</h1>

<!-- Après -->
<title>Training Academy 2025</title>
<h1>Welcome to Training Academy</h1>
<p>Professional Development Programs</p>
```

#### Étape 3 : Committer les Modifications

```bash
# Vérifiez les changements
git status

# Ajoutez les fichiers modifiés
git add index.html

# Committez avec un message descriptif
git commit -m "Update header with new academy branding"

# Vérifiez le commit
git log --oneline -1
```

#### Étape 4 : Pousser la Branche

```bash
# Poussez votre branche vers le serveur
git push origin feature/new-header

# Résultat attendu :
# remote: To create a pull request, visit:
# remote:   https://dev.azure.com/...
```

#### Étape 5 : Créer la Pull Request via Azure DevOps

**Via l'interface :**
1. Allez sur **Repos** → **Pull Requests**
2. Cliquez sur **New Pull Request**
3. Vérifiez les paramètres :
   - Source : `feature/new-header`
   - Target : `main`
4. Remplissez les informations :
   - **Titre** : "Update academy branding and header"
   - **Description** : Détaillez les changements et le contexte
5. Assignez des **Reviewers** (2-3 personnes)
6. Liez un **Work Item** si nécessaire 
7. Cliquez sur **Create**

**Exemple de description :**
```
## Description
Update the site header to reflect the new 2025 branding for Easy Training Academy.

## Changes Made
- Updated page title to "Easy Training Academy 2025"
- Enhanced main heading with academy name
- Added descriptive tagline

## Type of Change
- [ ] Bug fix
- [x] New feature
- [ ] Breaking change
- [x] Documentation update

## Testing
Manual testing completed in Chrome and Firefox.
Responsive design verified for mobile and tablet.

## Related to
Fixes AB#123
```

#### Étape 6 : Examiner les Modifications (en tant que Reviewer)

**Onglet Overview :**
1. Consultez le résumé général
2. Vérifiez le status des policies
3. Lisez la description

**Onglet Files :**
1. Allez à l'onglet **Files**
2. Observez les **changements en couleur** :
   - 🔴 Lignes supprimées
   - 🟢 Lignes ajoutées
3. Cliquez sur une ligne pour **ajouter un commentaire**
4. Rédigez le commentaire :
   ```
   Great update! The new heading is much clearer.
   Suggestion: Consider adding a subtitle for better hierarchy.
   ```
5. Cliquez sur **Comment**

#### Étape 7 : Approuver la PR

**En tant que Reviewer :**
1. Allez à l'onglet **Overview**
2. Lisez les commentaires et réponses
3. Cliquez sur le bouton **Approve** ou **Vote** (selon la configuration)
4. Options possibles :
   - ✅ **Approve** : Approbation complète
   - ⚠️ **Approve with suggestions** : Approbation conditionnelle
   - ❓ **Wait for author** : En attente de clarification
   - ❌ **Reject** : Rejet (exceptionnel)
5. Sélectionnez **Approve**
6. Ajoutez un commentaire :
   ```
   Code review completed.
   Changes look good and follow our design guidelines.
   Ready to merge.
   ```

#### Étape 8 : Observer les Événements

Vous verrez une timeline montrant :
- ✅ Approval by John Doe
- 💬 Comments added
- 🔄 Changes requested
- ✔️ All policies met

#### Étape 9 : Compléter la Fusion

1. Une fois que les conditions sont remplies :
   - ✅ Minimum reviewers approuvé
   - ✅ Policies satisfaites
   - ✅ Aucun conflit
2. Cliquez sur le bouton **Complete**
3. Options de fusion :
   - ☑️ **Delete branch after merge** (recommandé : OUI)
   - 📝 **Add merge message** (optionnel)
   - Merge type : Basic merge / Squash / Rebase
4. Cliquez sur **Complete merge**

#### Étape 10 : Vérifier la Fusion

```bash
# Basculez sur main
git checkout main

# Mettez à jour depuis le serveur
git pull origin main

# Vérifiez que les changements sont présents
cat index.html | grep "Training Academy 2025"

# Vérifiez que la branche a été supprimée
git branch -a
# feature/new-header ne devrait pas apparaître
```



### Points Clés à Mémoriser
- Une PR crée un **espace de discussion structuré**
- Les modifications sont visibles de manière **contextuelle**
- L'approbation est une **responsabilité** du reviewer
- Les commentaires doivent être **constructifs**
- L'historique des PRs est **permanent et traçable**

---




---

