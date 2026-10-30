# Day 58/100 — Azure Infrastructure as Code with Bicep

**100 Days of DevOps — From Zero to Hero**

## Goal
Understand Infrastructure as Code and deploy reusable Azure resources with Bicep.

## 1. What is Infrastructure as Code?

Infrastructure as Code (IaC) defines and manages infrastructure through configuration files rather than manually creating every resource.

Benefits include automation, consistency, repeatability, version control, collaboration, and fewer manual errors.

## 2. What is Azure Bicep?

Azure Bicep is Microsoft's declarative IaC language for Azure. It has a concise syntax and compiles to an ARM template for deployment through Azure Resource Manager.

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: 'mystorageaccount12345'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

## 3. Bicep vs. ARM vs. Terraform

| Feature | Bicep | ARM Templates | Terraform |
|---|---|---|---|
| Language | Bicep DSL | JSON | HCL |
| Ecosystem | Azure | Azure | Multi-provider |
| Readability | High | More verbose | High |
| State | Azure Resource Manager deployment state | Azure Resource Manager deployment state | Terraform state |
| Best suited for | Azure-native IaC | Azure-native deployments | Multi-provider workflows |

## 4. Bicep file building blocks

- `param`: deployment input values
- `var`: reusable values and expressions
- `resource`: Azure resources to create or manage
- `output`: values returned after deployment
- `module`: reusable infrastructure definitions

```bicep
param location string = resourceGroup().location
param storageName string

var storageSku = 'Standard_LRS'

resource storage 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageName
  location: location
  sku: {
    name: storageSku
  }
  kind: 'StorageV2'
}

output storageId string = storage.id
```

## 5. Hands-on lab — deploy a Storage Account

**Cost note:** This lab creates Azure resources and may incur charges. Check your subscription, region, resource name, and expected costs. Use a dedicated lab resource group.

### Step 1 — Check Azure CLI and install Bicep

```bash
az version
az bicep install
az bicep version
```

If already installed:

```bash
az bicep upgrade
```

### Step 2 — Sign in and select subscription

```bash
az login
az account list --output table
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
az account show --output table
```

### Step 3 — Create project folder

```bash
mkdir azure-bicep-lab
cd azure-bicep-lab
touch main.bicep
```

Open `main.bicep` in VS Code or another editor.

### Step 4 — Add the Bicep template

```bicep
@description('Azure region for the storage account')
param location string = resourceGroup().location

@description('Globally unique storage account name')
@minLength(3)
@maxLength(24)
param storageName string

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}

output storageAccountId string = storageAccount.id
output storageAccountName string = storageAccount.name
```

This defines a StorageV2 account with locally redundant storage (LRS), HTTPS-only traffic, TLS 1.2 minimum, and anonymous blob public access disabled.

Storage account names must be globally unique, lowercase letters and numbers only, 3–24 characters.

### Step 5 — Create a lab resource group

```bash
az group create \
  --name rg-bicep-lab \
  --location centralindia
```

Choose another supported region if appropriate.

### Step 6 — Build and validate

```bash
az bicep build --file main.bicep
```

Validate the deployment:

```bash
az deployment group validate \
  --resource-group rg-bicep-lab \
  --template-file main.bicep \
  --parameters storageName="<YOUR_UNIQUE_STORAGE_NAME>"
```

Expected result: validation succeeds if the template, name, permissions, subscription, and configuration are valid.

### Step 7 — Preview changes with what-if

```bash
az deployment group what-if \
  --resource-group rg-bicep-lab \
  --template-file main.bicep \
  --parameters storageName="<YOUR_UNIQUE_STORAGE_NAME>"
```

Review the proposed changes before deployment.

### Step 8 — Deploy

```bash
az deployment group create \
  --resource-group rg-bicep-lab \
  --template-file main.bicep \
  --parameters storageName="<YOUR_UNIQUE_STORAGE_NAME>"
```

Expected result: Azure creates the storage account and returns deployment outputs.

### Step 9 — Verify

```bash
az storage account list \
  --resource-group rg-bicep-lab \
  --output table

az storage account show \
  --resource-group rg-bicep-lab \
  --name "<YOUR_UNIQUE_STORAGE_NAME>" \
  --output table
```

## 6. Reuse the template across environments

Create `dev.bicepparam`:

```bicep
using './main.bicep'

param storageName = 'stdevexample123'
param location = 'centralindia'
```

Create `prod.bicepparam`:

```bicep
using './main.bicep'

param storageName = 'stprodexample123'
param location = 'centralindia'
```

Deploy to existing resource groups:

```bash
az deployment group create \
  --resource-group rg-dev \
  --template-file main.bicep \
  --parameters dev.bicepparam

az deployment group create \
  --resource-group rg-prod \
  --template-file main.bicep \
  --parameters prod.bicepparam
```

Ensure those resource groups exist and each storage account name is globally unique.

## 7. Integrate Bicep with Git

Suggested structure:

```text
azure-bicep-lab/
├── main.bicep
├── dev.bicepparam
├── prod.bicepparam
└── README.md
```

```bash
git init
git add .
git commit -m "Add Azure Bicep storage account template"
```

Configure a Git remote and push to your repository as appropriate. Use pull requests and review what-if results before production deployments. Never commit secrets or credentials into Bicep files or Git.

## 8. Common errors and troubleshooting

| Error | Possible reason | Solution |
|---|---|---|
| Invalid template | Incorrect Bicep syntax | Run `az bicep build` |
| Storage account name unavailable | Name already used globally | Choose another unique name |
| Authorization failed | Insufficient permissions | Check Azure RBAC |
| Location unavailable | Region or service restriction | Select a supported region |
| Validation failed | Incorrect configuration | Review validation output |
| Existing resource differs | Resource already exists with other settings | Inspect resource and what-if output |

Useful commands:

```bash
az bicep version
az bicep build --file main.bicep
az deployment group list --resource-group rg-bicep-lab --output table
```

## 9. Production best practices

1. Store Bicep files in Git.
2. Use parameters for environment-specific configuration.
3. Split larger infrastructure into reusable modules.
4. Validate and run what-if before deploying.
5. Use Azure RBAC and least-privilege access.
6. Avoid storing passwords, keys, and secrets in source code.
7. Use deployment pipelines with approvals for production.
8. Review resource costs before deployment.
9. Keep templates and API versions maintained.
10. Document infrastructure changes.

## 10. Screenshot checklist

1. `main.bicep` open in VS Code.
2. Successful `az bicep build` output.
3. Successful deployment validation.
4. What-if deployment preview.
5. Storage account visible in Azure Portal.
6. Azure CLI deployment output.

## 11. LinkedIn post — ready to publish

🚀 **Day 58/100 — Azure Infrastructure as Code with Bicep**

Imagine creating the same Azure infrastructure manually across Development, Testing, and Production.

It takes time, introduces inconsistencies, and makes changes difficult to track.

This is where Infrastructure as Code (IaC) makes a difference.

🔹 **What I learned today:**
- IaC lets us define infrastructure through code.
- Azure Bicep provides a clean, declarative way to deploy Azure resources.
- Parameters make templates reusable across environments.
- Azure Resource Manager handles the deployment.
- Git helps version and review infrastructure changes.

🛠️ **Hands-on practice:**
1. Installed and checked the Bicep CLI.
2. Created a Bicep template for an Azure Storage Account.
3. Built and validated the template.
4. Previewed changes using `what-if`.
5. Deployed and verified the resource using Azure CLI.

💡 **Key takeaway:** Infrastructure should be repeatable, reviewable, and version-controlled—not dependent on manual portal clicks.

Moving toward production-ready DevOps, one practical lesson at a time!

Continuing my **100 Days of DevOps — From Zero to Hero** journey. 🚀

How are you managing infrastructure in your projects—manually, Bicep, or Terraform?

#100DaysOfDevOps #Azure #Bicep #InfrastructureAsCode #IaC #DevOps #CloudComputing #Automation #LearningInPublic

## 12. Next lesson

**Day 59/100 — Azure Bicep Modules, Parameters, and Reusable Templates**

We'll organize infrastructure into modules, share common configurations, and build templates that scale across environments.

## Official Microsoft documentation

- [What is Bicep?](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview)
- [Install Bicep tools](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install)
- [Bicep parameters](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/parameters)
- [Bicep modules](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules)
- [What-if deployment operation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-what-if)
