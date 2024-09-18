# Azure Service Principal in Azure DevOps Pipelines 🚀

Using a Service Principal in Azure DevOps pipelines allows you to securely authenticate and interact with Azure resources. Here’s a step-by-step guide on creating and using a Service Principal, with instructions for both `command line` and `GUI` methods.

### **Prerequisites 🛠️**

Before you start, ensure you have the following:

1. **Azure CLI**: Install Azure CLI from the [official documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
2. **PowerShell**: Ensure you have PowerShell installed. You can download it from [Microsoft’s site](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-windows).
3. **Azure Subscription**: You need an active Azure subscription to create resources.

### **1. Create a Service Principal**

#### **Using Azure CLI in PowerShell:**

1. **Sign in to Azure:** 🔑
   ```bash
   az login
   ```
   
   - **Sample Output:**
     ```c
     [
       {
         "cloudName": "AzureCloud",
         "homeTenantId": "<your-tenant-id>",
         "id": "<your-user-id>",
         "isDefault": true,
         "managedByTenants": [],
         "name": "<your-user-name>",
         "state": "Enabled",
         "tenantId": "<your-tenant-id>",
         "user": {
           "name": "<your-user-email>",
           "type": "user"
         }
       }
     ]
     ```

2. **Get the Subscription ID:** 🆔
   - If you don’t know your subscription ID, list your subscriptions with:
     ```powershell
     $subscriptionId = az account list --query "[?isDefault].id" -o tsv
     ```
   
   - **Sample Output:**
     ```
     <your-subscription-id>
     ```

3. **Create the Service Principal and Generate a Secret:** 🔐
   - Run the following command to create a Service Principal and generate a client secret:
     ```powershell
     $sp = az ad sp create-for-rbac --name <your-app-name> --role Contributor --scopes /subscriptions/$subscriptionId | ConvertFrom-Json
     ```
     - Replace `<your-app-name>` with a name for your Service Principal.
   - **Sample Output:**
     ```json
     {
       "appId": "<your-client-id>",
       "displayName": "<your-app-name>",
       "password": "<your-client-secret>",
       "tenant": "<your-tenant-id>"
     }
     ```
   - **Store these values securely.** 

4. **Store the Client Secret in a PowerShell Variable:** 💾
   ```powershell
   $clientId = $sp.appId
   $clientSecret = $sp.password
   $tenantId = $sp.tenant
   ```
   
   - **Sample Output:**
     ```c
     $clientId = <your-client-id>
     $clientSecret = <your-client-secret>
     $tenantId = <your-tenant-id>
     ```

#### **Using Azure Portal (GUI):**

1. **Sign in to Azure Portal:**
   - Go to the [Azure Portal](https://portal.azure.com) and log in with your credentials. 🌐

2. **Register a New Application:**
   - Navigate to **Azure Active Directory** > **App registrations**.
   - Click **New registration**.
   - Enter a name for your Service Principal (e.g., `MyPipelineSP`).
   - Choose **Accounts in this organizational directory only** as the supported account type.
   - Click **Register**. 📝

3. **Generate a Client Secret:**
   - Select the application you just created.
   - Go to **Certificates & secrets**.
   - Click **New client secret**.
   - Add a description (e.g., `MySecret`) and select an expiration period.
   - Click **Add**.
   - **Copy the secret value** immediately (store it securely). 🔑
   - **Sample Output:**
     ```
     Description: MySecret
     Value: <your-client-secret>
     Expires: <expiration-date>
     ```

4. **Assign Roles to the Service Principal:**
   - Navigate to the resource or resource group you want to grant access to.
   - Go to **Access control (IAM)**.
   - Click **Add role assignment**.
   - Choose a role (e.g., `Contributor`).
   - Search for your Service Principal by name, select it, and click **Save**. 🛠️

### **2. Use the Service Principal in Azure DevOps Pipelines**

#### **Add Service Principal Credentials to Azure DevOps:**

1. **Navigate to Your Azure DevOps Project:**
   - Go to your Azure DevOps project and click on **Pipelines**. 🏗️

2. **Create or Edit a Variable Group:**
   - Go to **Library** > **+ Variable group** (or edit an existing one).
   - Add two variables:
     - `AZURE_CLIENT_ID`: Enter the Application (client) ID from your Service Principal.
     - `AZURE_CLIENT_SECRET`: Enter the client secret value from your Service Principal.
   - Click **Save**. 💾

#### **Authenticate and Run Commands in the Pipeline:**

1. **Edit Your Pipeline:**
   - Go to **Pipelines** and select the pipeline you want to edit. ✏️

2. **Add a Script Task for Authentication:**
   ```yaml
   steps:
     - script: |
         az login --service-principal -u $(AZURE_CLIENT_ID) -p $(AZURE_CLIENT_SECRET) --tenant <tenant-id>
       displayName: 'Azure Login'
   ```
   Replace `<tenant-id>` with your Azure Active Directory tenant ID. 🔑

   - **Sample Output:**
     ```json
     [
       {
         "cloudName": "AzureCloud",
         "homeTenantId": "<your-tenant-id>",
         "id": "<your-client-id>",
         "isDefault": true,
         "managedByTenants": [],
         "name": "<your-app-name>",
         "state": "Enabled",
         "tenantId": "<your-tenant-id>",
         "user": {
           "name": "<your-service-principal-name>",
           "type": "servicePrincipal"
         }
       }
     ]
     ```

3. **Run Azure CLI Commands:**

   **Example: Create a Resource Group:**
   ```yaml
   steps:
     - script: |
         az group create --name myResourceGroup --location eastus
       displayName: 'Create Resource Group'
   ```
   🏷️
   - **Sample Output:**
     ```json
     {
       "id": "/subscriptions/<your-subscription-id>/resourceGroups/myResourceGroup",
       "location": "eastus",
       "managedBy": null,
       "name": "myResourceGroup",
       "properties": {
         "provisioningState": "Succeeded"
       },
       "tags": {}
     }
     ```

---

By following these steps, you’ll be able to create a Service Principal, verify its login using Azure CLI in PowerShell, configure it in Azure DevOps, and use it to manage Azure resources securely and efficiently. 🌟

Feel free to customize the examples and details based on your specific needs and environment!
