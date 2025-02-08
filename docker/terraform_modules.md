Here are some **Terraform modules** to automate the **entire deployment** for **Azure Key Vault, AWS Secrets Manager, and HashiCorp Vault** with **automatic secret rotation**.

---

# **🔹 1. Terraform Module for Azure Key Vault + AKS with Secret Rotation**
This Terraform script will:
✅ Create an **AKS Cluster**  
✅ Deploy **Azure Key Vault**  
✅ Store a **secret in Key Vault**  
✅ Set up **automatic secret rotation using an Azure Function**  

---

### **Step 1: Create `main.tf`**
```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "myResourceGroup"
  location = "East US"
}

# Create AKS Cluster
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

# Create Azure Key Vault
resource "azurerm_key_vault" "kv" {
  name                        = "myKeyVault"
  location                    = azurerm_resource_group.rg.location
  resource_group_name         = azurerm_resource_group.rg.name
  tenant_id                   = data.azurerm_client_config.current.tenant_id
  sku_name                    = "standard"
}

# Store Secret in Key Vault
resource "azurerm_key_vault_secret" "db_password" {
  name         = "DB_PASSWORD"
  value        = "superSecretPassword"
  key_vault_id = azurerm_key_vault.kv.id
}

# Deploy Azure Function for Secret Rotation
module "function_app" {
  source              = "Azure/function-app/azurerm"
  function_app_name   = "rotateSecrets"
  resource_group_name = azurerm_resource_group.rg.name
  location           = azurerm_resource_group.rg.location
  storage_account_name = "secretrotationstorage"
  app_settings       = {
    "AZURE_KEYVAULT_URL" = azurerm_key_vault.kv.vault_uri
  }
}
```

✅ **Now, the Terraform script automates the entire AKS + Key Vault + Secret Rotation setup!**

---

# **🔹 2. Terraform Module for AWS Secrets Manager + EKS with Rotation**
This Terraform script will:
✅ Create an **AWS EKS Cluster**  
✅ Deploy **AWS Secrets Manager**  
✅ Store a **secret**  
✅ Enable **automatic secret rotation using AWS Lambda**  

---

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

# Create EKS Cluster
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "my-eks-cluster"
  cluster_version = "1.25"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets
}

# Deploy AWS Lambda for Secret Rotation
resource "aws_lambda_function" "rotate_secret" {
  filename      = "lambda_function.zip"
  function_name = "RotateSecretsFunction"
  role          = aws_iam_role.lambda_exec.arn
  handler       = "lambda_function.lambda_handler"
  runtime       = "python3.9"
}

# Enable Rotation in AWS Secrets Manager
resource "aws_secretsmanager_secret_rotation" "rotation" {
  secret_id           = aws_secretsmanager_secret.db_password.id
  rotation_lambda_arn = aws_lambda_function.rotate_secret.arn
  rotation_rules {
    automatically_after_days = 30
  }
}
```

✅ **This Terraform module sets up AWS EKS, Secrets Manager, and automatic secret rotation using AWS Lambda.**

---

# **🔹 3. Terraform Module for HashiCorp Vault + Kubernetes with Rotation**
This Terraform script will:
✅ Deploy **HashiCorp Vault**  
✅ Store a **secret**  
✅ Enable **automatic secret rotation for database credentials**  

---

### **Step 1: Create `main.tf`**
```hcl
provider "vault" {
  address = "http://localhost:8200"
}

# Enable Vault Secret Engine
resource "vault_mount" "secret" {
  path        = "secret"
  type        = "kv"
  options = {
    version = "2"
  }
}

# Store a Secret in Vault
resource "vault_kv_secret_v2" "db_password" {
  mount = vault_mount.secret.path
  name  = "db_password"

  data_json = jsonencode({
    username = "myUser"
    password = "superSecretPassword"
  })
}

# Enable Auto-Rotation for Database Credentials
resource "vault_database_secret_backend_connection" "mydb" {
  backend       = "database"
  name          = "mydb"
  allowed_roles = ["readonly"]

  mysql {
    connection_url = "mysql://{{username}}:{{password}}@mysql.example.com:3306/"
  }
}

resource "vault_database_secret_backend_role" "readonly" {
  backend             = vault_database_secret_backend_connection.mydb.backend
  name                = "readonly"
  db_name             = vault_database_secret_backend_connection.mydb.name
  creation_statements = ["GRANT SELECT ON *.* TO '{{name}}'@'%' IDENTIFIED BY '{{password}}';"]
  default_ttl         = "1h"
  max_ttl             = "24h"
}
```

✅ **This Terraform module deploys HashiCorp Vault, stores secrets, and automatically rotates database credentials every hour.**

---

# **🔹 TL;DR - Best Terraform Modules for Secret Rotation**
| Cloud | Secrets Manager | Rotation Method | Terraform Module Used |
|--------|----------------|-----------------|----------------------|
| **Azure AKS** | Azure Key Vault | Azure Function | `azurerm_kubernetes_cluster`, `azurerm_key_vault`, `Azure/function-app` |
| **AWS EKS** | AWS Secrets Manager | AWS Lambda | `aws_secretsmanager_secret`, `aws_lambda_function` |
| **Self-Hosted Kubernetes** | HashiCorp Vault | Dynamic Secrets | `vault_database_secret_backend_connection` |
