Kubernetes **Secrets** are only encoded in Base64, which is **not encryption**—it’s just a simple way to store and transmit data. If someone gains access to your cluster, they can easily decode the Secrets with tools like CyberChef. To actually secure sensitive data in Kubernetes, consider the six options below:

### **1. Use Kubernetes Secrets with Encryption at Rest**
By default, Kubernetes supports **Encryption at Rest** for Secrets. You can enable it by configuring **encryption providers** in the API server.

- **Enable Encryption Providers** in Kubernetes by creating an encryption configuration file:
  
  ```yaml
  apiVersion: apiserver.config.k8s.io/v1
  kind: EncryptionConfiguration
  resources:
    - resources:
        - secrets
      providers:
        - aescbc:
            keys:
              - name: key1
                secret: <base64-encoded-32-byte-key>
        - identity: {}
  ```

- Then, **pass this file** to the API server with the `--encryption-provider-config` flag.

### **2. Use an External Secrets Management System**
Instead of storing sensitive credentials in Kubernetes Secrets directly, you can use an **external secret manager**:

- **AWS Secrets Manager**
- **Google Secret Manager**
- **HashiCorp Vault**
- **Azure Key Vault**
- **Sealed Secrets** (a K8s-native approach)

These tools **store and encrypt secrets outside of Kubernetes** and inject them at runtime.

### **3. Hashing is NOT a Solution for API Keys or Passwords**
- **Hashing works for passwords** because they are meant to be stored and compared (like in authentication).
- **Hashing API keys is useless** because you need the original value to use them.

### **4. Sealed Secrets (Recommended)**
[Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) allows you to **encrypt Secrets** using a controller that only Kubernetes can decrypt.

- Install `kubeseal` CLI tool.
- Encrypt secrets **before** adding them to the cluster.
- Even if someone gets your YAML file, they **cannot decrypt** the values without access to the cluster's private key.

### **5. Use Environment Variables from a Secure Source**
Rather than hardcoding secrets in your YAML file, pull them from **a vault** at runtime:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

If using **an external secret manager**, you can have the secret injected dynamically.

### **6. Use Mutating Webhooks to Inject Secrets**
- A **mutating admission webhook** can intercept pod creation requests and inject secrets dynamically.
- This prevents secrets from being stored in **plaintext YAML files**.

### **Too Long; Didn't Read (TL;DR) - Secure Your Kubernetes Secrets**
| Method | Pros | Cons |
|--------|------|------|
| **Encryption at Rest** | Easy to enable, built into K8s | Doesn't prevent in-cluster access |
| **External Secret Managers** | Highly secure, integrates with cloud IAM | Extra setup required |
| **Sealed Secrets** | Encrypts secrets before storing in Git | Requires a controller in cluster |
| **Mutating Webhooks** | Injects secrets dynamically | Requires custom webhook development |

🔒 **For production, the best approach is to use an external secrets manager or Sealed Secrets to avoid exposing credentials in plaintext.**
