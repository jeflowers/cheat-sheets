### **🚀 Injecting Secrets into Containers at Runtime Using Terraform**
**automated secret storage** in **Azure Key Vault, AWS Secrets Manager, and HashiCorp Vault**, let's ensure **Docker and Kubernetes workloads retrieve these secrets securely at runtime**.

---

## **🔹 1. Injecting Secrets from Azure Key Vault into AKS Pods**
We'll **mount secrets dynamically** inside Kubernetes using the **Azure Key Vault CSI Driver**.

### **Step 1: Modify `main.tf` to Add CSI Driver**
Add the **SecretProviderClass** to your AKS Terraform script:

```hcl
resource "kubernetes_manifest" "azure_secret_provider" {
  manifest = {
    apiVersion = "secrets-store.csi.x-k8s.io/v1"
    kind       = "SecretProviderClass"
    metadata = {
      name      = "azure-keyvault-provider"
      namespace = "default"
    }
    spec = {
      provider = "azure"
      parameters = {
        keyvaultName = azurerm_key_vault.kv.name
        tenantId     = data.azurerm_client_config.current.tenant_id
        objects = <<EOT
          array:
            - |
              objectName: "DB_PASSWORD"
              objectType: "secret"
        EOT
      }
    }
  }
}
```

### **Step 2: Modify Kubernetes Deployment to Use Key Vault Secret**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: my-image
          volumeMounts:
            - name: secrets-store
              mountPath: "/mnt/secrets"
              readOnly: true
      volumes:
        - name: secrets-store
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: "azure-keyvault-provider"
```

✅ **Now, your application can access `DB_PASSWORD` from `/mnt/secrets/DB_PASSWORD` inside the pod.**

---

## **🔹 2. Injecting AWS Secrets Manager Secrets into EKS Pods**
We'll use the **External Secrets Operator** to sync AWS Secrets into Kubernetes.

### **Step 1: Modify `main.tf` to Install External Secrets Operator**
```hcl
resource "kubernetes_manifest" "aws_secret_provider" {
  manifest = {
    apiVersion = "external-secrets.io/v1beta1"
    kind       = "ExternalSecret"
    metadata = {
      name      = "aws-db-secret"
      namespace = "default"
    }
    spec = {
      refreshInterval = "1h"
      secretStoreRef = {
        name = "aws-secrets-store"
        kind = "ClusterSecretStore"
      }
      target = {
        name            = "aws-db-secret"
        creationPolicy  = "Owner"
      }
      data = [
        {
          secretKey = "DB_PASSWORD"
          remoteRef = {
            key      = aws_secretsmanager_secret.db_password.name
            property = "password"
          }
        }
      ]
    }
  }
}
```

### **Step 2: Modify Kubernetes Deployment to Use AWS Secret**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: my-image
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: aws-db-secret
                  key: DB_PASSWORD
```

✅ **Now, AWS Secrets Manager dynamically injects the secret as an environment variable inside your pod.**

---

## **🔹 3. Injecting HashiCorp Vault Secrets into Kubernetes Pods**
We'll use the **Vault Agent Injector** to automatically inject secrets.

### **Step 1: Modify `main.tf` to Enable Vault Injector**
```hcl
resource "kubernetes_manifest" "vault_agent" {
  manifest = {
    apiVersion = "apps/v1"
    kind       = "Deployment"
    metadata = {
      name      = "vault-agent"
      namespace = "default"
    }
    spec = {
      replicas = 1
      selector = {
        matchLabels = {
          app = "vault-agent"
        }
      }
      template = {
        metadata = {
          labels = {
            app = "vault-agent"
          }
        }
        spec = {
          containers = [
            {
              name  = "vault-agent"
              image = "hashicorp/vault"
              command = ["vault", "agent", "-config=/etc/vault/config.hcl"]
            }
          ]
        }
      }
    }
  }
}
```

### **Step 2: Modify Kubernetes Deployment to Use Vault Secret**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "my-role"
    vault.hashicorp.com/agent-inject-secret-db_password: "secret/my-app"
spec:
  containers:
    - name: my-container
      image: my-image
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: vault-db-secret
              key: DB_PASSWORD
```

✅ **Now, HashiCorp Vault dynamically injects the `DB_PASSWORD` secret at runtime.**

---

## **🔹 4. Injecting Secrets from HashiCorp Vault into Docker**
If you're running **Docker without Kubernetes**, you can retrieve **Vault secrets at runtime**.

### **Step 1: Store a Secret in Vault**
```bash
vault kv put secret/my-app password="superSecretPassword"
```

### **Step 2: Retrieve the Secret in Docker**
```bash
docker run -e DB_PASSWORD=$(vault kv get -field=password secret/my-app) my-image
```

✅ **Now, the Docker container dynamically retrieves secrets from Vault instead of using hardcoded values.**

---

## **🚀 TL;DR - The Best Ways to Inject Secrets into Containers**
| Cloud | Secrets Manager | How Secrets Are Injected |
|--------|----------------|--------------------------|
| **Azure AKS** | Azure Key Vault | CSI Driver (mounts secrets as files) |
| **AWS EKS** | AWS Secrets Manager | External Secrets Operator (injects as env vars) |
| **Self-Hosted Kubernetes** | HashiCorp Vault | Vault Agent Injector (injects secrets dynamically) |
| **Docker Standalone** | HashiCorp Vault | Secrets retrieved at runtime via CLI |
