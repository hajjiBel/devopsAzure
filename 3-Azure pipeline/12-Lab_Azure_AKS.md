# Lab Azure Pipeline : Déploiement d'Application sur Azure Kubernetes Service (AKS)

## Objectifs du Lab

- Créer et configurer un registre Azure Container Registry (ACR)
- Créer un cluster Azure Kubernetes Service (AKS) attaché à l'ACR
- Construire une image Docker personnalisée et la pousser vers l'ACR
- Configurer un pipeline CI/CD Azure DevOps pour automatiser le build et le déploiement
- Déployer l'application sur AKS via des manifestes Kubernetes (Deployment, Service, HPA)

---

## 1. Schéma de l'Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                AZURE CLOUD ENVIRONMENT                          │
├─────────────────────────────────────────────┬───────────────────┤
│  Azure Container Registry (ACR)             │   AKS Cluster     │
│                                             │                   │
│  ┌──────────────┐     Pull Image            │  ┌─────────────┐  │
│  │ Docker Image │ ───────────────────────▶  │  │ Pod (App)  │  │
│  │ (Nginx App)  │                           │  └─────────────┘  │
│  └──────▲───────┘                           │         ▲         │
│         │                                   │         │         │
└─────────┼───────────────────────────────────┴─────────┼─────────┘
          │ Push Image (CI)                             │ Deploy (CD)
          │                                             │
┌─────────┴─────────────────────────────────────────────┴─────────┐
│                      AZURE DEVOPS PIPELINE                      │
├─────────────────────────────────────────────────────────────────┤
│  STAGE 1: CI (Build & Push)    │   STAGE 2: CD (Deploy)         │
│                                │                                │
│  1. Build Docker Image         │   1. Create/Update Secrets     │
│  2. Push to ACR                │   2. Apply Deployment.yml      │
│  3. Publish Artifacts          │   3. Apply Service.yml         │
│     (Manifests)                │   4. Apply HPA.yml             │
└────────────────────────────────┴────────────────────────────────┘
```

### Description du flux

1. **Infrastructure** : Un registre d'images (ACR) et un cluster Kubernetes (AKS) sont déployés sur Azure.
2. **CI (Continuous Integration)** : Le pipeline construit l'image Docker de l'application et la pousse vers l'ACR.
3. **CD (Continuous Deployment)** : Le pipeline récupère les fichiers manifestes (YAML) et demande à AKS de mettre à jour l'application en utilisant la nouvelle image disponible dans l'ACR.
4. **HPA (Horizontal Pod Autoscaler)** : Une règle d'autoscaling ajuste automatiquement le nombre de pods en fonction de la charge CPU.

---

## 2. Préparation de l'Environnement Azure

### 2.1 Connexion et Création des Ressources

Exécutez les commandes suivantes dans Azure Cloud Shell ou votre terminal local :

```bash
# 1. Connexion à Azure
az login
az account set --subscription "Votre-ID-Abonnement"

# 2. Création du Groupe de Ressources
az group create --name rg-aks-lab --location francecentral

# 3. Création de l'Azure Container Registry (ACR)
# Note: Le nom de l'ACR doit être unique globalement. Remplacez 'monacr123' par un nom unique.
ACR_NAME="aksdemoacr$RANDOM"
az acr create --name $ACR_NAME --resource-group rg-aks-lab --sku Basic

# 4. Création du Cluster AKS attaché à l'ACR
# L'argument --attach-acr configure automatiquement les permissions (AcrPull)
AKS_NAME="aks-cluster-lab"
az aks create \
  --name $AKS_NAME \
  --resource-group rg-aks-lab \
  --node-count 1 \
  --node-vm-size Standard_B2s \
  --generate-ssh-keys \
  --attach-acr $ACR_NAME

# 5. Récupérer les identifiants pour kubectl (pour tester localement)
az aks get-credentials --resource-group rg-aks-lab --name $AKS_NAME
```

---

## 3. Préparation de l'Application et des Manifestes

### 3.1 Structure du Projet

Créez un dossier pour votre projet et organisez-le comme suit :

```
my-aks-app/
├── Dockerfile
├── index.html
├── manifests/
│   ├── deployment.yml
│   ├── service.yml
│   └── hpa.yml
└── azure-pipelines.yml
```

### 3.2 Fichiers de l'Application

**index.html** :
```html
<!DOCTYPE html>
<html>
<head>
    <title>AKS Demo</title>
</head>
<body>
    <h1>Bienvenue sur mon application AKS v1 !</h1>
    <p>Déployé via Azure DevOps.</p>
</body>
</html>
```

**Dockerfile** :
```dockerfile
FROM nginx:alpine
WORKDIR /usr/share/nginx/html
# Supprimer la page par défaut
RUN rm -rf ./*
# Copier notre page
COPY index.html .
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 3.3 Manifestes Kubernetes

**manifests/deployment.yml** :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 2 # Nombre initial de pods
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        # __IMAGE_NAME__ sera remplacé par le pipeline
        image: __IMAGE_NAME__
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"
```

**manifests/service.yml** :
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: LoadBalancer # Expose l'app sur une IP publique
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
```

**manifests/hpa.yml** (Autoscaling) :
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## 4. Configuration du Pipeline CI/CD

### 4.1 Créer la Service Connection pour Docker (ACR)

1. Dans Azure DevOps > Project Settings > Service connections.
2. **New service connection** > **Docker Registry** > **Azure Container Registry**.
3. Sélectionnez votre abonnement et l'ACR créé (`aksdemoacr...`).
4. Nommez la connexion : `AcrConnection`.

### 4.2 Créer la Service Connection pour Kubernetes (AKS)

1. **New service connection** > **Kubernetes**.
2. Sélectionnez **Azure Subscription**.
3. Sélectionnez votre cluster `aks-cluster-lab`.
4. Nommez la connexion : `AksConnection`.

### 4.3 Fichier `azure-pipelines.yml`

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  # Remplacez par le nom de votre ACR (sans .azurecr.io)
  acrName: 'aksdemoacrXXXX' 
  imageRepository: 'my-app'
  tag: '$(Build.BuildId)'
  # Nom des Service Connections créées précédemment
  dockerRegistryServiceConnection: 'AcrConnection'
  kubernetesServiceConnection: 'AksConnection'

stages:
# STAGE 1: CI (Build & Push Docker Image)
- stage: Build
  displayName: Build and Push
  jobs:
  - job: Build
    displayName: Build
    steps:
    - task: Docker@2
      displayName: Build and Push to ACR
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: '**/Dockerfile'
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
          latest

    # Publier les manifestes pour le stage suivant
    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Pipeline.Workspace)/s/manifests'
        artifact: 'manifests'
        publishLocation: 'pipeline'

# STAGE 2: CD (Deploy to AKS)
- stage: Deploy
  displayName: Deploy to AKS
  dependsOn: Build
  jobs:
  - deployment: Deploy
    displayName: Deploy to AKS
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadPipelineArtifact@2
            inputs:
              buildType: 'current'
              artifactName: 'manifests'
              targetPath: '$(Pipeline.Workspace)/manifests'

          # Remplacer le placeholder d'image dans le deployment.yml
          - script: |
              sed -i "s|__IMAGE_NAME__|$(acrName).azurecr.io/$(imageRepository):$(tag)|g" $(Pipeline.Workspace)/manifests/deployment.yml
            displayName: 'Update Image Tag in Manifest'

          # Appliquer les manifestes Kubernetes
          - task: Kubernetes@1
            displayName: 'Apply Manifests'
            inputs:
              connectionType: 'Kubernetes Service Connection'
              kubernetesServiceEndpoint: $(kubernetesServiceConnection)
              command: 'apply'
              arguments: '-f $(Pipeline.Workspace)/manifests/'
```

---

## 5. Exécution et Validation

1. **Pousser le code** : Commitez et poussez tous les fichiers vers votre repository Azure Repos (ou GitHub).
2. **Lancer le Pipeline** :
   - Allez dans **Pipelines** > **New Pipeline**.
   - Sélectionnez votre repo et le fichier YAML existant.
   - Lancez le run.

3. **Vérifier le Déploiement** :
   - Une fois le pipeline terminé, allez dans le portail Azure ou utilisez `kubectl`.
   - Récupérez l'IP publique du service :
     ```bash
az aks get-credentials --resource-group <RG> --name <AKS_NAME> --overwrite-existing
kubectl get nodes
kubectl get service
kubectl get svc my-app-service
     ```
   - Ouvrez l'IP externe (`EXTERNAL-IP`) dans votre navigateur. Vous devriez voir "Bienvenue sur mon application AKS v1 !".

4. **Tester l'Autoscaling** :
   - Vérifiez que le HPA est actif :
     ```bash
     kubectl get hpa
     ```

---

## 6. Points Clés & Troubleshooting

| Erreur | Cause Possible | Solution |
|--------|----------------|----------|
| **ImagePullBackOff** | AKS n'arrive pas à tirer l'image depuis l'ACR. | Vérifiez que l'AKS est bien attaché à l'ACR (`az aks update --attach-acr ...`) ou que le secret `imagePullSecrets` est configuré si vous n'utilisez pas l'intégration native. |
| **Pending (Service)** | L'IP publique met du temps à être allouée. | Attendez quelques minutes. Si cela persiste, vérifiez les quotas d'IP publiques dans votre région Azure. |
| **Pipeline Fail** | Erreur de connexion Docker/Kubernetes. | Vérifiez que les noms des Service Connections dans le YAML correspondent exactement à ceux créés dans Project Settings. |
| **Sed error** | Le remplacement de `__IMAGE_NAME__` échoue. | Vérifiez le chemin du fichier `deployment.yml` dans l'étape `sed`. |

---

## 7. Nettoyage des Ressources

Pour éviter des coûts inutiles, supprimez le groupe de ressources à la fin du TP :

```bash
az group delete --name rg-aks-lab --yes --no-wait
```
