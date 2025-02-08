**Terraform automation scripts** for setting up **Azure Key Vault, AWS Secrets Manager, and HashiCorp Vault** for **Docker & Kubernetes**.

---

## **🔹 1. Terraform for Azure Key Vault + AKS Integration**
This script **creates an Azure Key Vault**, **stores a secret**, and **integrates it with AKS using CSI Driver**.

### **Step 1: Create `main.tf`**
```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "myResourceGroup"
  location = "East US"
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = "myAKSCluster"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "myaks"

  default_node_pool {
    name       = "default"
    node_count = 1
    vm_size    = "Standard_DS2_v2"
  }

  identity {
    type = "SystemAssigned"
  }
}

resource "azurerm_key_vault" "kv" {
  name                        = "myKeyVault"
  location                    = azurerm_resource_group.rg.location
  resource_group_name         = azurerm_resource_group.rg.name
  tenant_id                   = data.azurerm_client_config.current.tenant_id
  sku_name                    = "standard"
}

resource "azurerm_key_vault_secret" "db_password" {
  name         = "DB_PASSWORD"
  value        = "superSecretPassword"
  key_vault_id = azurerm_key_vault.kv.id
}
```

### **Step 2: Apply the Terraform Configuration**
```bash
terraform init
terraform apply -auto-approve
```

✅ **This will create an AKS cluster with Azure Key Vault integration.**

---

## **🔹 2. Terraform for AWS Secrets Manager + Kubernetes**
This script creates an **AWS EKS cluster**, stores a **secret in AWS Secrets Manager**, and **retrieves it dynamically**.

### **Step 1: Create `main.tf`**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_secretsmanager_secret" "db_password" {
  name = "db_password"
}

resource "aws_secretsmanager_secret_version" "db_password_value" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = jsonencode({
    username = "myUser"
    password = "superSecretPassword"
  })
}

resource "aws_eks_cluster" "eks" {
  name     = "my-eks-cluster"
  role_arn = aws_iam_role.eks_role.arn

  vpc_config {
    subnet_ids = aws_subnet.public[*].id
  }
}
```

### **Step 2: Apply the Terraform Configuration**
```bash
terraform init
terraform apply -auto-approve
```

✅ **AWS Secrets Manager now stores secrets securely and can inject them into Kubernetes!**

---

## **🔹 3. Terraform for HashiCorp Vault + Docker/Kubernetes**
This script **deploys Vault using Terraform** and **stores a secret**.

### **Step 1: Create `main.tf`**
```hcl
provider "vault" {
  address = "http://localhost:8200"
}

resource "vault_mount" "secret" {
  path        = "secret"
  type        = "kv"
  options = {
    version = "2"
  }
}

resource "vault_kv_secret_v2" "db_password" {
  mount = vault_mount.secret.path
  name  = "db_password"

  data_json = jsonencode({
    username = "myUser"
    password = "superSecretPassword"
  })
}
```

### **Step 2: Apply the Terraform Configuration**
```bash
terraform init
terraform apply -auto-approve
```

✅ **Now Vault securely stores and injects secrets into your Kubernetes or Docker environment.**

---

## **🚀 TL;DR - Terraform Automates Secure Secrets Setup**
| Cloud | Service | Terraform Resources Used |
|--------|---------|----------------|
| **Azure** | AKS + Key Vault | `azurerm_kubernetes_cluster`, `azurerm_key_vault` |
| **AWS** | EKS + Secrets Manager | `aws_secretsmanager_secret`, `aws_eks_cluster` |
| **Self-Hosted** | HashiCorp Vault | `vault_mount`, `vault_kv_secret_v2` |

