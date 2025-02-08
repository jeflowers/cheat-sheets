Set up **automatic secret rotation** for **Azure Key Vault, AWS Secrets Manager, and HashiCorp Vault** to ensure secrets are regularly updated without manual intervention.

---

# **🔹 1. Azure Key Vault - Secret Rotation in AKS**
Azure Key Vault supports **automatic rotation** for secrets when used with **Azure Functions or Azure Automation**.

### **Method 1: Using Azure Key Vault Managed Identity Rotation**
Azure Key Vault allows **auto-rotation** for **certificates, but not for secrets natively**. However, you can use an **Azure Function** to rotate secrets.

### **Step 1: Create an Azure Function to Rotate Secrets**
```python
import os
import datetime
import logging
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Set up logging
logging.basicConfig(level=logging.INFO)

# Initialize Azure Key Vault Client
vault_url = os.getenv("AZURE_KEYVAULT_URL")
credential = DefaultAzureCredential()
client = SecretClient(vault_url=vault_url, credential=credential)

def rotate_secret(secret_name):
    new_value = "new-secret-value-" + datetime.datetime.now().strftime("%Y%m%d%H%M%S")
    client.set_secret(secret_name, new_value)
    logging.info(f"Secret {secret_name} rotated successfully!")

rotate_secret("DB_PASSWORD")
```

### **Step 2: Deploy the Function to Azure**
1. Deploy this Python script as an **Azure Function**.
2. Schedule it using an **Azure Timer Trigger (e.g., every 30 days).**
3. Azure Key Vault will now **auto-rotate the secret every X days**.

✅ **AKS will continue to fetch the updated secret dynamically via the CSI driver.**

---

# **🔹 2. AWS Secrets Manager - Automatic Rotation**
AWS Secrets Manager **supports automatic rotation** natively for database credentials, API keys, and more.

### **Step 1: Enable Rotation for an AWS Secret**
```bash
aws secretsmanager rotate-secret --secret-id my-secret --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:RotateSecretsFunction
```

### **Step 2: Create an AWS Lambda Function to Rotate Secrets**
```python
import boto3
import json

secrets_client = boto3.client("secretsmanager")

def lambda_handler(event, context):
    secret_id = event["SecretId"]
    
    # Generate new secret value
    new_secret_value = f"NewSecret-{context.aws_request_id}"
    
    # Store new secret in AWS Secrets Manager
    secrets_client.put_secret_value(SecretId=secret_id, SecretString=new_secret_value)
    
    return {"status": "Secret rotated successfully"}
```

### **Step 3: Configure AWS Secrets Rotation Schedule**
- Go to **AWS Secrets Manager** > Select Secret
- Click **Enable Rotation** > Choose your **Lambda Function**
- Set rotation schedule (e.g., every 30 days)

✅ **AWS EKS will dynamically pick up the rotated secret when injected via the External Secrets Operator.**

---

# **🔹 3. HashiCorp Vault - Automatic Secret Rotation**
Vault supports **dynamic secrets** with auto-rotation for **databases, SSH, cloud credentials**, and more.

### **Method 1: Enable Database Secret Rotation**
```bash
vault write database/config/mydb \
    plugin_name=mysql-database-plugin \
    connection_url="mysql://{{username}}:{{password}}@mysql.example.com:3306/" \
    allowed_roles="readonly"

vault write database/roles/readonly \
    db_name=mydb \
    creation_statements="GRANT SELECT ON *.* TO '{{name}}'@'%' IDENTIFIED BY '{{password}}';" \
    default_ttl="1h" \
    max_ttl="24h"
```

### **Method 2: Use Vault Lease Auto-Renewal**
```bash
vault lease renew database/creds/readonly
```
- Vault **automatically rotates** the database credentials every **1 hour**.
- Kubernetes **fetches the new secret dynamically** when Vault Injector is used.

✅ **No need for manual intervention!**

---

# **🔹 TL;DR - Which Rotation Method Should You Use?**
| Cloud | Secrets Manager | Rotation Method |
|--------|----------------|--------------------------|
| **Azure AKS** | Azure Key Vault | Azure Function auto-rotation |
| **AWS EKS** | AWS Secrets Manager | AWS Lambda auto-rotation |
| **Self-Hosted Kubernetes** | HashiCorp Vault | Dynamic Secrets & Lease Renewal |

