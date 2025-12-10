# Lab Azure Pipeline : Gestion des Secrets avec Azure Key Vault

## Objectifs du Lab

- Comprendre la gestion sécurisée des secrets dans un pipeline CI/CD
- Créer et configurer une ressource **Azure Key Vault**
- Configurer les permissions d'accès pour Azure DevOps (Service Principal)
- Intégrer Azure Key Vault dans un pipeline YAML pour récupérer un mot de passe de base de données sans l'exposer
- Déployer une application ou une base de données en utilisant ce secret

---

## 1. Schéma de l'Architecture

```
┌──────────────────────┐          ┌────────────────────────────┐
│   Azure Key Vault    │          │    Azure DevOps Pipeline   │
│                      │          │                            │
│  ┌────────────────┐  │ Permissions   ┌────────────────────┐  │
│  │ Secret:        │◀─┼──────────┼─── │ Task: AzureKeyVault│  │
│  │ db-password    │  │  (Get/List)   │                    │  │
│  └────────────────┘  │          │    └─────────┬──────────┘  │
└──────────────────────┘          │              │ (Injecte)   │
                                  │              ▼             │
                                  │    ┌────────────────────┐  │
                                  │    │ Task: Deploy DB    │  │
                                  │    │ (Uses $(db-password)) │  │
                                  │    └────────────────────┘  │
                                  └────────────────────────────┘
```

### Description du flux

1. **Stockage** : Le mot de passe administrateur SQL est stocké uniquement dans Azure Key Vault.
2. **Accès** : Le pipeline Azure DevOps s'authentifie via une *Service Connection* (Service Principal).
3. **Récupération** : La tâche `AzureKeyVault` récupère le secret au moment de l'exécution.
4. **Utilisation** : Le secret est disponible comme une variable de pipeline (ex: `$(db-password)`) et injecté dans les tâches suivantes, sans jamais être écrit en dur dans le code.

---

## 2. Prérequis et Configuration des Ressources

### 2.1 Créer un Projet Azure DevOps

1. Si ce n'est pas fait, créez un projet : `AzureKeyVault-Demo`.
2. Importez votre repository contenant le code SQL ou applicatif (ou utilisez celui du lab précédent `SQLServer-Database-Demo`).

### 2.2 Créer l'Azure Key Vault

1. Accédez au **Portail Azure**.
2. Recherchez **Key Vaults** et cliquez sur **+ Créer**.
   - **Groupe de ressources** : `TestRG` (le même que votre base de données).
   - **Nom du coffre** : `kv-lab-[votre-nom]` (doit être unique).
   - **Région** : West Europe.
   - **Niveau de tarification** : Standard.
3. Cliquez sur **Suivant : Configuration d'accès**.
   - **Modèle d'autorisation** : Sélectionnez **Stratégie d'accès au coffre** (Vault access policy) pour simplifier ce lab (ou RBAC si vous êtes à l'aise).
4. Cliquez sur **Vérifier + créer** puis **Créer**.

### 2.3 Créer le Secret

1. Une fois le Key Vault créé, allez dans **Objets** > **Secrets**.
2. Cliquez sur **+ Générer/Importer**.
   - **Options de chargement** : Manuel.
   - **Nom** : `sql-admin-password`.
   - **Valeur secrète** : Saisissez votre mot de passe SQL (ex: `P@ssw0rd1234!`).
3. Cliquez sur **Créer**.

---

## 3. Configuration de la Sécurité (Crucial)

Pour que Azure DevOps puisse lire ce secret, il faut lui donner la permission explicite.

### 3.1 Identifier le Service Principal

1. Dans Azure DevOps, allez dans **Project Settings** > **Service connections**.
2. Repérez votre connexion `ServiceConnection-Lab`.
3. Cliquez sur **Manage Service Principal** (cela ouvre le portail Azure sur l'App Registration).
4. Copiez l'**ID d'application (client)** ou le nom de l'application.

### 3.2 Ajouter une Stratégie d'Accès dans Key Vault

1. Retournez dans votre ressource **Key Vault** sur le portail Azure.
2. Allez dans **Stratégies d'accès** (Access policies).
3. Cliquez sur **+ Créer**.
   - **Autorisations de secret** : Cochez **Obtenir (Get)** et **Lister (List)**.
   - **Principal** : Cliquez sur "Aucune sélectionnée", puis recherchez le nom de votre Service Principal (trouvé à l'étape 3.1) ou `TestRG-ServiceConnection` (le nom par défaut).
4. Cliquez sur **Suivant**, puis **Créer** (ou Ajouter).
5. **IMPORTANT** : Cliquez sur **Enregistrer** en haut de la page Stratégies d'accès.

---

## 4. Configuration YAML du Pipeline CD

Nous allons modifier le pipeline CD (du lab précédent SQL ou App) pour qu'il tire le mot de passe du Key Vault.

### 4.1 Pipeline YAML

Créez ou modifiez `azure-pipelines.yml` :

```yaml
trigger:
- main

pool:
  vmImage: 'windows-latest'

variables:
  azureSubscription: 'ServiceConnection-Lab'
  # Nom de votre Key Vault
  keyVaultName: 'kv-lab-[votre-nom]'
  # Autres variables
  sqlServerName: 'sql-server-lab-[votre-nom].database.windows.net'
  databaseName: 'Products'
  sqlAdminLogin: 'sqladmin'
  # Note : sqlAdminPassword n'est plus défini ici !

steps:
# 1. Récupérer les secrets depuis Key Vault
- task: AzureKeyVault@2
  displayName: 'Get Secrets from Key Vault'
  inputs:
    azureSubscription: '$(azureSubscription)'
    KeyVaultName: '$(keyVaultName)'
    SecretsFilter: 'sql-admin-password' # Nom exact du secret dans Azure
    RunAsPreJob: false

# 2. Vérification (Optionnel - pour debug uniquement, n'affiche pas le secret en clair)
- script: |
    echo "Secret recovered successfully."
    echo "Login: $(sqlAdminLogin)"
    # La ligne suivante affichera '***' dans les logs
    echo "Password Length: $(sql-admin-password)" 
  displayName: 'Debug Secret Presence'

# 3. Utilisation du secret pour déployer (Exemple SQL DACPAC)
- task: SqlAzureDacpacDeployment@1
  displayName: 'Deploy SQL DACPAC'
  inputs:
    azureSubscription: '$(azureSubscription)'
    AuthenticationType: 'server'
    ServerName: '$(sqlServerName)'
    DatabaseName: '$(databaseName)'
    SqlUsername: '$(sqlAdminLogin)'
    # On utilise ici la variable générée par la tâche Key Vault
    SqlPassword: '$(sql-admin-password)' 
    deployType: 'DacpacTask'
    DeploymentAction: 'Publish'
    DacpacFile: '$(Pipeline.Workspace)/drop/**/*.dacpac' # Assurez-vous d'avoir l'artefact
    IpDetectionMethod: 'AutoDetect'
```

---

## 5. Exécution et Validation

1. **Lancer le Pipeline** : Exécutez le pipeline dans Azure DevOps.
2. **Observer les Logs** :
   - Regardez la tâche `AzureKeyVault`. Vous devriez voir "Downloaded secret: sql-admin-password".
   - Regardez la tâche de déploiement. Elle devrait s'authentifier avec succès.
3. **Sécurité** : Vérifiez que le mot de passe n'apparaît **jamais** en clair dans les logs. Azure DevOps masque automatiquement les variables marquées comme secrètes (il affiche `***`).

---

## 6. Alternative : Variable Groups (Méthode Recommandée pour Production)

Au lieu d'utiliser la tâche `AzureKeyVault` directement dans le YAML, vous pouvez lier un **Variable Group**.

1. **Pipelines** > **Library** > **+ Variable group**.
2. Nom : `KV-Secrets`.
3. Cochez **Link secrets from an Azure key vault as variables**.
4. Sélectionnez votre connexion et votre Key Vault.
5. Ajoutez le secret `sql-admin-password` et cliquez sur **Save**.

**Utilisation dans le YAML :**

```yaml
variables:
- group: KV-Secrets # Les secrets sont maintenant accessibles directement

steps:
- script: echo $(sql-admin-password)
```

Cette méthode est plus propre car elle sépare la configuration des secrets de la logique du pipeline.

---

## 7. Troubleshooting

| Erreur | Solution |
|--------|----------|
| **403 Forbidden** | Le Service Principal n'a pas de *Access Policy* sur le Key Vault. Retournez à l'étape 3.2 et n'oubliez pas de cliquer sur **Enregistrer**. |
| **Secret not found** | Le nom dans `SecretsFilter` ne correspond pas exactement au nom du secret dans Azure (sensible à la casse). |
| **Service Connection** | Assurez-vous d'utiliser la même Service Connection dans le pipeline et dans la stratégie d'accès Key Vault. |
