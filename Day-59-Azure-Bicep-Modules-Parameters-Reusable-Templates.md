# Day 59/100 — Azure Bicep Modules, Parameters, and Reusable Templates

## The problem

Copying separate infrastructure templates for development, testing, and production causes duplication, configuration drift, and extra maintenance.

**Solution:** Bicep modules let you split infrastructure into reusable files. Parameters let you configure each environment without copying the deployment logic.

## 1. Key concepts

- `param`: accepts input values.
- `var`: defines reusable expressions.
- `module`: calls another Bicep file.
- `params`: passes values to a module.
- `output`: returns deployment information.
- `@secure()`: marks a sensitive parameter.

Example project:

```text
azure-bicep-modules/
├── main.bicep
├── dev.bicepparam
├── prod.bicepparam
└── modules/
    └── storage.bicep
```

## 2. Create a reusable storage module

Create `modules/storage.bicep`:

```bicep
param storageName string
param location string = resourceGroup().location

resource storage 'Microsoft.Storage/storageAccounts@2023-05-01' = {
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

output storageId string = storage.id
output storageAccountName string = storage.name
```

The module accepts a name and location, creates a StorageV2 account with HTTPS and TLS 1.2 enforced and public blob access disabled, then returns its ID and name.

## 3. Create the parent template

Create `main.bicep`:

```bicep
param location string = resourceGroup().location
param storageName string

module storageModule './modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageName: storageName
    location: location
  }
}

output storageAccountId string = storageModule.outputs.storageId
output deployedStorageName string = storageModule.outputs.storageAccountName
```

The parent calls the child module, passes its parameters, and reads its outputs.

## 4. Create environment parameter files

`dev.bicepparam`:

```bicep
using './main.bicep'

param storageName = 'devstg5926demo'
param location = 'centralindia'
```

`prod.bicepparam`:

```bicep
using './main.bicep'

param storageName = 'prodstg5926demo'
param location = 'centralindia'
```

These names are examples. Storage account names must be globally unique, 3–24 characters long, and contain only lowercase letters and numbers. Change them if already taken.

## 5. Deploy the lab

Sign in and select the correct subscription:

```bash
az login
az account show
az account list --output table
az account set --subscription "<SUBSCRIPTION_ID>"
```

Create a resource group (use a region available to your subscription):

```bash
az group create   --name rg-bicep-modules   --location centralindia
```

Build the parent template and referenced module:

```bash
az bicep build --file main.bicep
```

Validate:

```bash
az deployment group validate   --resource-group rg-bicep-modules   --template-file main.bicep   --parameters dev.bicepparam
```

Preview the changes:

```bash
az deployment group what-if   --resource-group rg-bicep-modules   --template-file main.bicep   --parameters dev.bicepparam
```

Review the proposed changes before deploying:

```bash
az deployment group create   --name dev-storage-deployment   --resource-group rg-bicep-modules   --template-file main.bicep   --parameters dev.bicepparam
```

Verify the storage account:

```bash
az storage account list   --resource-group rg-bicep-modules   --output table
```

Inspect deployment outputs:

```bash
az deployment group show   --resource-group rg-bicep-modules   --name dev-storage-deployment   --query properties.outputs
```

You can deploy the same template with `prod.bicepparam`. In a real production setup, use a separate production resource group and suitable access controls:

```bash
az deployment group create   --name prod-storage-deployment   --resource-group rg-bicep-modules   --template-file main.bicep   --parameters prod.bicepparam
```

## 6. Sensitive values

```bicep
@secure()
param adminPassword string
```

Never commit passwords, access keys, or other secrets in plain-text parameter files or Git. Use secure deployment mechanisms and appropriate Azure Key Vault integration.

## 7. Common troubleshooting

| Problem | What to check |
|---|---|
| Module file not found | Relative path from the parent file |
| Parameter mismatch | Names in parent and child files |
| Invalid parameter type | String, int, bool, or object type |
| Storage name unavailable | Choose a globally unique name |
| Permission denied | Azure RBAC permissions |
| Region not supported | Resource availability in that region |
| Output not found | Child module output name |

## 8. Production best practices

- Keep modules in Git with meaningful names and version history.
- Avoid hardcoding environment-specific values.
- Use separate parameter files for dev, test, and production.
- Protect secrets using secure parameter handling.
- Run build, validation, and what-if checks before deployment.
- Use code reviews and approval gates for production.
- Avoid over-modularizing; create modules where reuse and clarity benefit.

## 9. Screenshot checklist

1. Project folder structure in VS Code.
2. `modules/storage.bicep` code.
3. `main.bicep` calling the module.
4. Successful build and validation.
5. What-if output.
6. Successful deployment.
7. Storage account in Azure Portal.

## 10. Learning challenge

1. What is a Bicep module?
2. Why use modules instead of copy-pasting?
3. What is the difference between `param` and `output`?
4. How does a parent template communicate with a child module?
5. How can one template support dev and production?

**Key takeaway:** Bicep modules help move from individual deployment files to reusable, organized infrastructure architecture.

## Official resources

- [Bicep modules](https://learn.microsoft.com/azure/azure-resource-manager/bicep/modules)
- [Bicep parameters](https://learn.microsoft.com/azure/azure-resource-manager/bicep/parameters)
- [Bicep outputs](https://learn.microsoft.com/azure/azure-resource-manager/bicep/outputs)
- [Bicep parameter files](https://learn.microsoft.com/azure/azure-resource-manager/bicep/parameter-files)

**Cost note:** Azure Storage may incur charges. Review your subscription pricing. Clean up lab resources only after confirming they contain no data you need; deleting a resource group permanently deletes its resources.
