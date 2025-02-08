Reference **environment variables** from **Kubernetes Secrets** in your **deployment YAML file**.
By default **Secrets** are only Base64-encoded by default. However, security can be improved by combining **environment variables** with a more secure **secrets management approach**.

---

## ✅ **Using Environment Variables from Secrets**
In your **Deployment YAML**, you can reference Secrets like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
        - name: my-container
          image: my-image
          env:
            - name: DB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: my-secret
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: my-secret
                  key: password
```

And the **Secret YAML** can look like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: bXlVc2VyTmFtZQ==  # Base64 encoded value of "myUserName"
  password: c3VwZXJTZWNyZXQ=  # Base64 encoded value of "superSecret"
```

💡 **This works, but it does not provide strong security** because the data is only Base64-encoded, meaning anyone with access to the cluster can decode it easily.

---

## 🔒 **How to Improve Security Beyond Base64 Encoding**
If you're using **Azure Kubernetes Service (AKS)**, **Minikube**, or **Docker-based Kubernetes**, consider these **enhanced security options**:

### **1️⃣ Use an External Secrets Manager**
Instead of storing secrets **inside Kubernetes**, you can store them in **Azure Key Vault** or **another external vault**, and inject them into your pod at runtime.

#### **Using Azure Key Vault with Kubernetes:**
1. **Create a Key Vault in Azure**
2. **Use the AKV Provider for Secrets Store CSI Driver**
3. **Mount the secret directly in your Pod**

Example of a pod referencing Azure Key Vault:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test
  annotations:
    secrets-store.csi.k8s.io/used: "true"
spec:
  containers:
    - name: busybox
      image: busybox
      command:
        - sleep
        - "1000000"
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
          secretProviderClass: "azure-kv"
```
✅ **Secrets are stored securely in Key Vault, not in Kubernetes YAML files!**

---

### **2️⃣ Encrypt Kubernetes Secrets at Rest**
Enable **Encryption at Rest** for Kubernetes Secrets so that even if someone gains access to the etcd database, they cannot read the secrets.

#### **Steps:**
1. **Create an encryption configuration file (`encryption-config.yaml`)**
2. **Configure AES encryption**
3. **Pass the file to the API server (`kube-apiserver --encryption-provider-config`)**

---

### **3️⃣ Use HashiCorp Vault or Another Secret Manager**
Instead of keeping secrets in Kubernetes, use **Vault** to securely inject them at runtime.

✅ **Benefits:**
- Strong **encryption**
- Secrets **never stored** in YAML or etcd
- **Dynamic secret rotation** is possible

---

### **TL;DR - The Best Approach for AKS, Minikube, or Docker-based Kubernetes**
| Approach | Security Level | Pros | Cons |
|----------|---------------|------|------|
| **Base64 Kubernetes Secrets** | ❌ Weak | Simple to use | Base64 is not encryption |
| **External Secret Managers (Azure Key Vault, AWS Secrets Manager, HashiCorp Vault, etc.)** | ✅✅✅ Strong | Secure & encrypted storage | Requires external setup |
| **Encryption at Rest (Kubernetes API server encryption)** | ✅✅ Medium | Protects secrets in etcd | Does not prevent runtime access |
| **Sealed Secrets** | ✅✅ Medium | Encrypts secrets before storing in Git | Requires controller in the cluster |

---

### **🚀 Recommended for AKS or Minikube**
1. **Use Azure Key Vault** (or HashiCorp Vault) to store secrets.
2. **Use CSI Driver** to mount secrets dynamically in Kubernetes.
3. **Enable Encryption at Rest** in Kubernetes.
4. **Use Sealed Secrets** if you must store secrets in Git.

