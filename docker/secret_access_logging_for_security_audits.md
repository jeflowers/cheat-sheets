### **🚀 Configuring Secret Access Logging for Security Audits**
To track **who accessed your secrets, when, and from where**, we need **audit logging** for **Azure Key Vault, AWS Secrets Manager, and HashiCorp Vault**.

---

# **🔹 1. Azure Key Vault - Enable Access Logging**
✅ **Azure Monitor & Log Analytics** allow tracking **secret access events**.

### **Step 1: Enable Key Vault Diagnostic Settings in Terraform**
```hcl
resource "azurerm_monitor_diagnostic_setting" "keyvault_logs" {
  name                       = "keyvault-diagnostics"
  target_resource_id         = azurerm_key_vault.kv.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.logs.id

  log {
    category = "AuditEvent"
    enabled  = true
  }
}
```

### **Step 2: Query Logs in Azure Monitor**
Use **Azure Log Analytics** to query **Key Vault access logs**:
```kusto
AzureDiagnostics
| where ResourceType == "VAULTS"
| where Category == "AuditEvent"
| project TimeGenerated, CallerIpAddress, OperationName, ResultType
| order by TimeGenerated desc
```

✅ **Now, all secret access events are logged in Azure Monitor.**

---

# **🔹 2. AWS Secrets Manager - Enable Access Logging**
✅ **AWS CloudTrail** allows tracking secret access events.

### **Step 1: Enable CloudTrail Logging for AWS Secrets Manager**
```hcl
resource "aws_cloudtrail" "secrets_audit" {
  name           = "secrets-access-trail"
  s3_bucket_name = aws_s3_bucket.secrets_logs.id
  include_global_service_events = true
  is_multi_region_trail = true
}

resource "aws_cloudwatch_log_group" "secrets_manager_logs" {
  name = "/aws/secretsmanager/audit"
}
```

### **Step 2: Query Secret Access Logs**
Run this AWS CLI command:
```bash
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventSource,AttributeValue=secretsmanager.amazonaws.com
```

✅ **Now, all secret access attempts are logged in CloudTrail and CloudWatch.**

---

# **🔹 3. HashiCorp Vault - Enable Audit Logging**
✅ **Vault supports built-in audit logging**.

### **Step 1: Enable Vault Audit Logging**
```bash
vault audit enable file file_path=/var/log/vault_audit.log
```

### **Step 2: Query Logs for Secret Access**
```bash
cat /var/log/vault_audit.log | grep "secrets/data/db_password"
```

✅ **Now, every secret read/write operation is logged in Vault’s audit logs.**

---

# **🔹 TL;DR - Best Secret Access Logging Methods**
| Cloud | Secrets Manager | Logging Method | Log Query Tool |
|--------|----------------|----------------|----------------|
| **Azure AKS** | Azure Key Vault | Azure Monitor | Log Analytics |
| **AWS EKS** | AWS Secrets Manager | AWS CloudTrail | AWS CLI |
| **Self-Hosted Kubernetes** | HashiCorp Vault | Vault Audit Logs | grep & log files |
