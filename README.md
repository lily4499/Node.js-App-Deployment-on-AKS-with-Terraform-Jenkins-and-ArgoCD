# Node.js-App-Deployment-on-AKS-with-Terraform-Jenkins-and-ArgoCD

---

```markdown
# 🚀 Node.js App Deployment on AKS with Terraform, Jenkins, and ArgoCD

This project demonstrates how to:

- Push a Node.js web app to GitHub
- Provision an Azure Kubernetes Service (AKS) cluster using Terraform
- Configure a remote Terraform backend on Azure Blob Storage
- Build and push Docker images to Azure Container Registry (ACR) using Jenkins
- Deploy the application on AKS using ArgoCD (GitOps)

---

## 📁 Project Structure

```
.
├── terraform/
│   ├── main.tf
│   ├── providers.tf
│   ├── variables.tf
│   └── output.tf
├── Jenkinsfile
└── nodejs-app/
```

---

## 🧰 Prerequisites

- Azure CLI
- Terraform CLI
- Jenkins (with ACR credentials configured)
- ArgoCD CLI
- Node.js app (push to GitHub if not already)

---

## 1️⃣ Push Your Node.js App to GitHub

```bash
cd your-nodejs-app
git init
git remote add origin https://github.com/<your-username>/<repo-name>.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

---

## 2️⃣ Authenticate Terraform with Azure

### Option 1: Azure CLI (Interactive)
```bash
az login
```

### Option 2: Azure Service Principal (CI/CD)

```bash
az ad sp create-for-rbac \
  --role="Contributor" \
  --scopes="/subscriptions/<SUBSCRIPTION_ID>"
```

Export environment variables:
```bash
export ARM_CLIENT_ID=<appId>
export ARM_CLIENT_SECRET=<password>
export ARM_SUBSCRIPTION_ID=<subscriptionId>
export ARM_TENANT_ID=<tenant>
```

---

## 3️⃣ Create Azure Remote Backend

### Create `create-backend.sh`

```bash
#!/bin/bash

RESOURCE_GROUP_NAME=tfstate
STORAGE_ACCOUNT_NAME=tfstate$RANDOM
CONTAINER_NAME=tfstate

az group create --name $RESOURCE_GROUP_NAME --location westeurope

az storage account create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name $STORAGE_ACCOUNT_NAME \
  --sku Standard_LRS \
  --encryption-services blob

az storage container create \
  --name $CONTAINER_NAME \
  --account-name $STORAGE_ACCOUNT_NAME
```

### Run Script

```bash
chmod +x create-backend.sh
./create-backend.sh
```

---

## 4️⃣ Configure Terraform Backend

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate"
    storage_account_name = "<STORAGE_ACCOUNT_NAME>"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}
```

---

## 5️⃣ Create AKS Cluster with Terraform

```bash
cd terraform
terraform init
terraform apply -auto-approve
```

---

## 6️⃣ Configure Jenkins to Push Image to ACR

Update your `Jenkinsfile` as shown in this example:  
📎 [Jenkinsfile Sample](https://github.com/lily4499/nodejs-webapp-2/blob/main/Jenkinsfile)

ACR Login (optional test):
```bash
az acr login --name <yourACRName>
```

---

## 7️⃣ Get AKS Credentials

```bash
az aks get-credentials --resource-group my-rg --name karo-aks --overwrite-existing
kubectl get nodes
```

---

## 8️⃣ Deploy with ArgoCD CLI

### Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Get Initial Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d && echo
```

### Port-Forward ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### ArgoCD CLI Login

```bash
argocd login 127.0.0.1:8080
# Username: admin
# Password: (from previous step)
```

### Create ArgoCD App

```bash
argocd app create nodejs-app \
--repo https://github.com/ooghenekaro/argocd-amazon-manifest.git \
--path ./ \
--dest-server https://kubernetes.default.svc \
--dest-namespace default \
--sync-policy automatic
```

### Manage App

```bash
argocd app list
argocd app sync nodejs-app
argocd app get nodejs-app
argocd app rollback nodejs-app <revision>
```

---

## 🧹 Cleanup

```bash
az group delete --name tfstate
```

---

## 📸 Screenshots (Optional)

> Include UI screenshots of Jenkins Pipeline, ArgoCD UI, and kubectl output if available.

---

## 🧠 Links

- Terraform AKS Sample: [lily4499/aks-demo](https://github.com/ooghenekaro/aks-demo)
- Jenkins Pipeline Sample: [lily4499/nodejs-webapp-2](https://github.com/ooghenekaro/nodejs-webapp-2)
- ArgoCD Docs: [argo-cd](https://argo-cd.readthedocs.io/en/stable/)

---

---
