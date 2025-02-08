Here are step by step set up for **Azure Key Vault, AWS Secrets Manager, and HashiCorp Vault** with **Docker and Kubernetes** to securely manage your secrets.

---

## **🔹 1. Azure Key Vault with Kubernetes (AKS)**
Azure Key Vault securely stores secrets and integrates with **Azure Kubernetes Service (AKS)** using **the CSI driver**.

### **Step 1: Enable Azure Key Vault CSI Driver on AKS**
```bash
az aks enable-addons --resource-group <RESOURCE_GROUP> --name <CLUSTER_NAME> --addons azure-keyvault-secrets-provider
```

### **Step 2: Create a Key Vault and Store Secrets**
```bash
az keyvault create --name <KEYVAULT_NAME> --resource-group <RESOURCE_GROUP> --location <LOCATION>
az keyvault secret set --vault-name <KEYVAULT_NAME> --name "DB_PASSWORD" --value "superSecretPassword"
```

### **Step 3: Create a Kubernetes SecretProviderClass**
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kv-secret-provider
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    keyvaultName: "<KEYVAULT_NAME>"
    tenantId: "<TENANT_ID>"
    objects: |
      array:
        - |
          objectName: "DB_PASSWORD"
          objectType: "secret"
```

### **Step 4: Mount Key Vault Secrets into a Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
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
          secretProviderClass: "azure-kv-secret-provider"
```
✅ **The secret is securely stored in Azure Key Vault and mounted inside the container at `/mnt/secrets/DB_PASSWORD`.**

---

## **🔹 2. AWS Secrets Manager with Kubernetes & Docker**
AWS Secrets Manager securely stores and manages sensitive information and integrates with **EKS or standalone Docker containers**.

### **Step 1: Store a Secret in AWS Secrets Manager**
```bash
aws secretsmanager create-secret --name my-secret --secret-string '{"username":"myUser","password":"superSecretPassword"}'
```

### **Step 2: Retrieve Secrets from AWS CLI (for Docker)**
Run your container and inject the secret dynamically:
```bash
docker run -e DB_PASSWORD=$(aws secretsmanager get-secret-value --secret-id my-secret --query 'SecretString' --output text) my-image
```

### **Step 3: Use AWS Secrets Manager in Kubernetes with External Secrets Operator**
Install the **External Secrets Operator**:
```bash
kubectl apply -f https://github.com/external-secrets/external-secrets/releases/latest/download/crds.yaml
kubectl apply -f https://github.com/external-secrets/external-secrets/releases/latest/download/install.yaml
```

### **Step 4: Create an ExternalSecret Resource**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret
spec:
  refreshInterval: "1h"
  secretStoreRef:
    name: aws-secret-store
    kind: ClusterSecretStore
  target:
    name: my-secret
    creationPolicy: Owner
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: my-secret
        property: password
```
✅ **Now, Kubernetes will automatically retrieve the secret from AWS Secrets Manager and inject it into your pods.**

---

## **🔹 3. HashiCorp Vault with Docker & Kubernetes**
HashiCorp Vault securely manages secrets and integrates with both **Docker** and **Kubernetes**.

### **Step 1: Run HashiCorp Vault Locally**
```bash
docker run --cap-add=IPC_LOCK -d --name=dev-vault -e "VAULT_DEV_ROOT_TOKEN_ID=root" -p 8200:8200 hashicorp/vault
```
Access Vault UI at: [http://localhost:8200](http://localhost:8200)

### **Step 2: Enable the Kubernetes Authentication Method**
```bash
vault auth enable kubernetes
vault write auth/kubernetes/config \
    kubernetes_host="https://kubernetes.default.svc.cluster.local"
```

### **Step 3: Store a Secret in Vault**
```bash
vault kv put secret/my-app username=myUser password=superSecretPassword
```

### **Step 4: Deploy a Vault Agent in Kubernetes**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vault-agent
spec:
  template:
    spec:
      containers:
        - name: vault-agent
          image: hashicorp/vault
          command: ["vault", "agent", "-config=/etc/vault/config.hcl"]
```

### **Step 5: Inject Secrets into Pods Using Vault Sidecar**
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
```
✅ **Now, Vault securely injects secrets into the running container without storing them in Kubernetes!**

---

## **🔥 TL;DR - Best Secure Secrets Approach**
| Method | Best For | Security Level |
|--------|---------|---------------|
| **Azure Key Vault** + CSI Driver | AKS (Azure Kubernetes Service) | ✅✅✅ Very Secure |
| **AWS Secrets Manager** + External Secrets Operator | EKS & Docker | ✅✅✅ Very Secure |
| **HashiCorp Vault** + Vault Agent | Any Kubernetes Cluster | ✅✅✅ Very Secure |
| **Docker Env Vars** (`-e` flag) | Standalone Docker (NOT Recommended) | ❌ Weak |

---

### **🚀 Final Recommendation**
✅ **If using Azure Kubernetes Service (AKS):**
- **Use Azure Key Vault** + **CSI Driver**  
- Secrets are never stored in YAML files

✅ **If using AWS EKS or Docker on AWS:**
- Use **AWS Secrets Manager** + **External Secrets Operator**
- Secrets are injected dynamically at runtime

✅ **If using Docker standalone or generic Kubernetes:**
- **Use HashiCorp Vault** with Kubernetes Vault Agent to inject secrets into containers.

