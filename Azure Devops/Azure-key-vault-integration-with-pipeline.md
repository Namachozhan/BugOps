# Integrating Azure Key Vault with Azure DevOps Pipeline

## Overview
This document provides a detailed step-by-step guide to integrating Azure Key Vault with an Azure DevOps pipeline using both **Managed Identity** and **Service Principal** authentication methods. It includes instructions for setting up Azure Key Vault, storing secrets, granting necessary permissions, and retrieving secrets in an Azure DevOps pipeline. Additionally, common issues and fixes are documented.

---

## 1. Creating Azure Key Vault

### **Using Azure Portal (GUI):**
1. Go to the **Azure Portal**.
2. Navigate to **Key Vaults**.
3. Click **Create**.
4. Select the **Subscription** and **Resource Group**.
5. Provide a **Key Vault Name**.
6. Choose the appropriate **Region**.
7. Set **Pricing Tier** to "Standard".
8. Click **Review + Create**, then **Create**.

### **Using Azure CLI:**
```sh
az keyvault create --name <KEYVAULT_NAME> --resource-group <RESOURCE_GROUP> --location <LOCATION>
```

---

## 2. Creating and Storing Secrets in Azure Key Vault

### **Using Azure Portal (GUI):**
1. Open **Azure Key Vault**.
2. Navigate to **Secrets** → **Generate/Import**.
3. Enter a **Name** (e.g., `vm-username`).
4. Enter a **Value** (e.g., `admin-user`).
5. Click **Create**.

### **Using Azure CLI:**
```sh
az keyvault secret set --vault-name <KEYVAULT_NAME> --name "vm-username" --value "admin-user"
az keyvault secret set --vault-name <KEYVAULT_NAME> --name "vm-password" --value "SecurePassword123"
```

---

## 3. Configuring Managed Identity for Self-Hosted Agent VM

### **Assigning System-Managed Identity (GUI):**
1. Go to **Azure Portal** → **Virtual Machines**.
2. Select your **Self-hosted Agent VM**.
3. Go to **Identity** → **System Assigned**.
4. Click **On** and **Save**.
5. Copy the **Object ID** of the Managed Identity.

### **Using Azure CLI:**
```sh
az vm identity assign --name <VM_NAME> --resource-group <RESOURCE_GROUP>
```

---

## 4. Granting Key Vault Access to Managed Identity

### **Using Azure Portal (GUI):**
1. Open **Azure Key Vault**.
2. Navigate to **Access Control (IAM)**.
3. Click **Add role assignment**.
4. Select **Key Vault Secrets User**.
5. Choose **Managed Identity** → **Select your VM’s identity**.
6. Click **Save**.

### **Using Azure CLI:**
```sh
az role assignment create --assignee <MANAGED_IDENTITY_OBJECT_ID> --role "Key Vault Secrets User" --scope /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEYVAULT_NAME>
```

---

## 5. Configuring Azure DevOps Pipeline to Use Key Vault Secrets

### **Adding Azure Key Vault to DevOps Library (GUI):**
1. Open **Azure DevOps**.
2. Go to **Project Settings** → **Service Connections**.
3. Click **New Service Connection** → **Azure Resource Manager**.
4. Select **Managed Identity** and provide details.
5. Save the connection.

### **Pipeline YAML Configuration:**
```yaml
trigger:
- main

pool:
  name: Default  # Self-hosted agent pool

variables:
  - group: MyKeyVaultVariables  # Reference Key Vault

steps:
- task: AzureKeyVault@1
  inputs:
    azureSubscription: '<SERVICE_CONNECTION>'
    KeyVaultName: '<KEYVAULT_NAME>'
    SecretsFilter: '*'  # Fetch all secrets

- script: |
    echo "Username: $(vm-username)"
    echo "Password: $(vm-password)"
  displayName: 'Display Secrets'
```

---

## 6. Alternative: Using Service Principal for Authentication

### **Creating a Service Principal (CLI):**
```sh
az ad sp create-for-rbac --name "sp-keyvault-access" --role "Key Vault Secrets User" --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEYVAULT_NAME>
```
- Save the **App ID**, **Password**, and **Tenant ID**.

### **Granting Key Vault Access (CLI):**
```sh
az keyvault set-policy --name <KEYVAULT_NAME> --spn <APP_ID> --secret-permissions get list
```

### **Azure DevOps Service Connection (GUI):**
1. Go to **Azure DevOps** → **Project Settings**.
2. Open **Service Connections** → **New Service Connection**.
3. Select **Azure Resource Manager** → **Service Principal**.
4. Provide **App ID**, **Tenant ID**, and **Password**.
5. Click **Verify and Save**.

---

## 7. Common Issues & Fixes

### **Issue: Self-hosted agent lacks permissions**
**Fix:** Ensure the VM has **Managed Identity enabled** and the Key Vault has an **Access Policy** granting **Key Vault Secrets User** role.
```sh
az role assignment create --assignee <MANAGED_IDENTITY_OBJECT_ID> --role "Key Vault Secrets User" --scope /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEYVAULT_NAME>
```

### **Issue: Azure DevOps cannot fetch secrets**
**Fix:**
1. Verify **Service Connection** is correctly configured in DevOps.
2. Ensure **Key Vault Access Policy** allows **get/list** permissions.
3. Run:
```sh
az keyvault show --name <KEYVAULT_NAME>
az keyvault secret list --vault-name <KEYVAULT_NAME>
```

### **Issue: Unable to locate executable file pwsh**
**Fix:** Explicitly set script type as PowerShell in pipeline YAML.
```yaml
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      echo "Username: $(vm-username)"
      echo "Password: $(vm-password)"
```

---

## Conclusion
This guide provides a comprehensive setup for integrating Azure Key Vault with Azure DevOps using both **Managed Identity** and **Service Principal** authentication methods. By following these steps, you can securely store and retrieve sensitive data in your pipeline. If you face issues, refer to the **Common Issues & Fixes** section.

