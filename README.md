Here’s the cleaned-up **GitHub README.md** version of your CLI Chapter Breakdown, with **all emojis removed**:

---

```markdown
# CI/CD Pipeline with Node.js, Terraform, Jenkins, ACR, AKS & ArgoCD

This project demonstrates a complete DevOps workflow:

1. Containerizing a Node.js app  
2. Provisioning AKS with Terraform  
3. Configuring Terraform remote backend with Azure Blob  
4. Authenticating Azure access via Jenkins  
5. Building & pushing Docker images to Azure Container Registry (ACR)  
6. Deploying to AKS using ArgoCD (GitOps)  
7. Cleaning up resources  

---

## Chapter 1: Containerizing Our Node.js App

```bash
# Clone the Node.js app from GitHub
git clone https://github.com/your-org/your-node-app.git
cd your-node-app

# Run the app locally to test
npm install
npm start

# Build a Docker image
docker build -t my-node-app:latest .

# Test Docker image locally
docker run -p 3000:3000 my-node-app:latest
```

---

## Chapter 2: Terraform and AKS (with Remote Backend)

### Set up Terraform project

```bash
mkdir terraform-aks && cd terraform-aks
touch main.tf backend.tf
```

---

### Create `create-backend.sh` Script

```bash
cat <<EOF > create-backend.sh
#!/bin/bash

RESOURCE_GROUP_NAME=tfstate
STORAGE_ACCOUNT_NAME=lilitfstatesan
CONTAINER_NAME=tfstate

# Create resource group
az group create --name \$RESOURCE_GROUP_NAME --location eastus

# Create storage account
az storage account create --resource-group \$RESOURCE_GROUP_NAME --name \$STORAGE_ACCOUNT_NAME --sku Standard_LRS --encryption-services blob

# Create blob container
az storage container create --name \$CONTAINER_NAME --account-name \$STORAGE_ACCOUNT_NAME
EOF
```

### Run the Script

```bash
chmod +x create-backend.sh
./create-backend.sh
```

---

## Chapter 3: Authenticating to Azure (Terraform + Jenkins)

### Create a Service Principal

```bash
az login

az ad sp create-for-rbac \
  --name "jenkins-terraform-sp" \
  --role="Contributor" \
  --scopes="/subscriptions/<your-subscription-id>" \
  --sdk-auth
```

### Export Environment Variables

```bash
export ARM_CLIENT_ID="<clientId>"
export ARM_CLIENT_SECRET="<clientSecret>"
export ARM_SUBSCRIPTION_ID="<subscriptionId>"
export ARM_TENANT_ID="<tenantId>"
```

---

### Jenkins: Add Azure Credentials

1. Go to: Jenkins → Manage Jenkins → Credentials → (global)  
2. Add:  
   - Kind: Secret Text  
   - Secret: Paste JSON from `--sdk-auth`  
   - ID: `AZURE_CREDENTIALS`

---

## Chapter 4: Define AKS Cluster + Configure Backend

### Configure Terraform Backend (`backend.tf`)

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

### Define AKS Cluster (`main.tf`)

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = "myAKSCluster"
  location            = "East US"
  resource_group_name = "myResourceGroup"
  dns_prefix          = "myaks"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_DS2_v2"
  }

  identity {
    type = "SystemAssigned"
  }
}
```

### Provision Infrastructure

```bash
terraform init
terraform plan
terraform apply -auto-approve

# Get AKS kubeconfig
az aks get-credentials --resource-group <rg> --name <cluster>
kubectl get nodes
```

---

## Chapter 5: Jenkins – The Automation Engine

### Run Jenkins in Docker (local)

```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
-v jenkins_home:/var/jenkins_home \
jenkins/jenkins:lts
```

---

### Jenkinsfile (Azure Auth + Terraform)

```groovy
pipeline {
  agent any
  environment {
    AZURE_CREDENTIALS = credentials('AZURE_CREDENTIALS')
  }
  stages {
    stage('Azure Login') {
      steps {
        writeFile file: 'azureauth.json', text: "${AZURE_CREDENTIALS}"
        sh '''
        export ARM_CLIENT_ID=$(jq -r .clientId azureauth.json)
        export ARM_CLIENT_SECRET=$(jq -r .clientSecret azureauth.json)
        export ARM_SUBSCRIPTION_ID=$(jq -r .subscriptionId azureauth.json)
        export ARM_TENANT_ID=$(jq -r .tenantId azureauth.json)

        az login --service-principal \
            --username $ARM_CLIENT_ID \
            --password $ARM_CLIENT_SECRET \
            --tenant $ARM_TENANT_ID
        '''
      }
    }
    stage('Terraform Apply') {
      steps {
        sh '''
        terraform init
        terraform apply -auto-approve
        '''
      }
    }
  }
}
```

---

## Chapter 6: Build and Push Docker Image

```bash
# Build image
docker build -t myapp:1 .

# Tag for ACR
docker tag myapp:1 myacrlil.azurecr.io/myapp:1

# Login to ACR
az acr login --name myacrlil

# Push image
docker push myacrlil.azurecr.io/myapp:1
```

---

## Chapter 7: GitOps with ArgoCD

```bash
# Login to ArgoCD
argocd login <ARGOCD_SERVER> --username admin --password <password>

# Create App
argocd app create my-app \
--repo https://github.com/your-org/my-app.git \
--path manifests/production \
--dest-server https://kubernetes.default.svc \
--dest-namespace my-app

# Sync App
argocd app sync my-app

# Get App status
argocd app get my-app
```

---

## Chapter 8: Cleanup

```bash
terraform destroy -auto-approve
kubectl config delete-context <aks-cluster>
```

---

## Summary

This project integrates:
- Terraform for IaC  
- Docker & ACR for containerization  
- Jenkins for CI  
- AKS for orchestration  
- ArgoCD for GitOps CD  

Ready to push to production or scale your CI/CD learning to the next level.

---

## Project Structure Suggestion

```
.
├── create-backend.sh
├── backend.tf
├── main.tf
├── Jenkinsfile
└── README.md
```

---

## License

MIT © Liliane Konissi
```


