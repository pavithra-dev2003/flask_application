# Flask Application Deployment on Azure Kubernetes Service (AKS) using GitHub Actions
 This project demonstrates end-to-end CI/CD pipeline to build, containerize, and deploy a Flask web application on Azure Kubernetes Service (AKS) using GitHub Actions.

## The setup automates:
* Building Docker image
* Pushing to Docker Hub
* Logging in to Azure
* Updating Kubernetes manifests
* Deploying to AKS cluster

## Table of Contents
#### Architecture

* Project Structure
* Technologies Used
* CI/CD Workflow
* Azure Setup
* GitHub Secrets
* Kubernetes Manifests
* Commands
* Accessing the App
* Troubleshooting
* Future Improvements

### Architecture
flowchart LR
    Dev[Developer Push Code] --> GH[GitHub Actions CI/CD]
   *  GH --> DH[Docker Hub - Push Image]
   *  GH --> AZ[Azure Login]
   *  AZ --> AKS[Azure Kubernetes Service]
   *  AKS --> User[User Access via LoadBalancer]


* Developer pushes code to main.
* GitHub Actions builds & pushes Docker image to Docker Hub.
* GitHub Actions authenticates with Azure (Service Principal).
* Updated deployment.yaml & service.yaml applied to AKS.
* Users access app via LoadBalancer External IP.

### Technologies Used

* Flask → Python web framework
* Docker → Containerization
* Docker Hub → Image registry
* Azure Kubernetes Service (AKS) → Managed Kubernetes cluster
* GitHub Actions → CI/CD automation
* kubectl → Kubernetes CLI

### CI/CD Workflow

* The GitHub Actions pipeline (ci-cd.yml) does:
* Checkout repository
* Login to Docker Hub
* Build and push Docker image
* Azure login using Service Principal
* Get AKS credentials
* Update Kubernetes manifests
* Deploy app to AKS

### Azure Setup
1. Create AKS Cluster
```
az group create --name pavithra-rg --location eastus
az aks create --resource-group pavithra-rg --name bettercluster1 --node-count 2 --generate-ssh-keys
```

3. Create Service Principal
 ```
az ad sp create-for-rbac --name "github-actions-sp" --role contributor \
    --scopes /subscriptions/<your-subscription-id> \
    --sdk-auth
```


This will generate JSON output. Example:
```

{
  "clientId": "11111111-2222-3333-4444-555555555555",
  "clientSecret": "your-secret-value-here",
  "subscriptionId": "66666666-7777-8888-9999-aaaaaaaaaaaa",
  "tenantId": "bbbbbbbb-cccc-dddd-eeee-ffffffffffff"
}
```

### GitHub Secrets

Go to GitHub Repo → Settings → Secrets → Actions and add:
* DOCKER_USERNAME → Docker Hub username
* DOCKER_PASSWORD → Docker Hub password or token
* AZURE_CREDENTIALS → Paste the JSON output from Service Principal

### Kubernetes Manifests
```
deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flaskweb-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - name: flask
        image: <docker-username>/flaskweb:latest
        ports:
        - containerPort: 5000
```
```
service.yaml
apiVersion: v1
kind: Service
metadata:
  name: service-demo
spec:
  type: LoadBalancer
  selector:
    app: front-end
  ports:
  - port: 80
    targetPort: 5000
```

### Useful Commands
#### Check deployments
```
kubectl get deployments
```

#### Check pods
```
kubectl get pods
```

#### Check services
```
kubectl get svc
```

#### Rollout status
```
kubectl rollout status deployment/flaskweb-demo
```

### Accessing the App
```
Run:

kubectl get svc
```


Wait for EXTERNAL-IP (can take 2–5 min).

Open http://<EXTERNAL-IP> in browser → Flask app is live! 

#### Troubleshooting

EXTERNAL-IP pending → Ensure AKS cluster supports LoadBalancer.

Login failure → Check AZURE_CREDENTIALS JSON values.

Pods not running → Run:

kubectl describe pod <pod-name>
kubectl logs <pod-name>

#### Future Improvements

Add Ingress Controller instead of LoadBalancer.
Use Azure Container Registry (ACR) instead of Docker Hub.
Implement Helm charts for deployment.

Add monitoring with Prometheus + Grafana.
