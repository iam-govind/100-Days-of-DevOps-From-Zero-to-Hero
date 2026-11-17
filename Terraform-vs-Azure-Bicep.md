# Day 62/100 — Terraform vs Azure Bicep

**Track:** Infrastructure as Code (IaC)  
**Goal:** Compare Terraform and Azure Bicep, then deploy a storage account with each.

## 1. Introduction

Terraform and Azure Bicep both let you define infrastructure as code.

- **Terraform:** HashiCorp's declarative IaC tool, using HCL and providers. It supports Azure, AWS, Google Cloud, and many other platforms.
- **Azure Bicep:** Microsoft's declarative language for Azure. Bicep compiles to ARM JSON templates and deploys through Azure Resource Manager.

## 2. Workflows

### Terraform

```text
Write .tf files
     |
terraform init
     |
terraform validate
     |
terraform plan
     |
terraform apply
     |
Resources + updated state
```

### Bicep

```text
Write .bicep file
     |
Build / validate
     |
What-If preview
     |
Deploy through ARM
     |
Azure resources
```

## 3. Comparison

| Feature | Terraform | Azure Bicep |
|---|---|---|
| Language | HCL | Bicep |
| Cloud scope | Multi-cloud | Azure-focused |
| Deployment mechanism | Terraform core + providers | Azure Resource Manager |
| Preview | `terraform plan` | `what-if` |
| Apply/deploy | `terraform apply` | ARM deployment |
| State | Terraform state file | No Terraform-style state file |
| Modules | Yes | Yes |
| Azure integration | AzureRM provider | Native ARM integration |
| Ecosystem | Broad provider ecosystem | Azure ecosystem |

### Terraform state — important concept

Terraform uses `terraform.tfstate` to map configuration to real infrastructure and track managed resources.

```text
Terraform configuration
          |
          v
     State file
          |
          v
  Actual infrastructure
```

For team and production use, store state in a secure remote backend such as Azure Storage. Use access controls and locking to avoid concurrent changes. State may contain sensitive values: never commit state files, credentials, or sensitive variable files to Git.

Bicep does not need a separate Terraform-style state file. ARM handles deployment operations and keeps deployment history. This does not remove the need for governance and drift management.

## 4. Lab A — Deploy with Terraform

### Prerequisites

- Azure subscription and permission to create a resource group and storage account.
- Azure CLI and Terraform installed.
- A globally unique storage account name (3–24 lowercase letters and numbers).

Check tools and sign in:

```bash
az version
terraform version
az login
az account list --output table
az account set --subscription "$AZURE_SUBSCRIPTION_ID"
export ARM_SUBSCRIPTION_ID="$AZURE_SUBSCRIPTION_ID"
```

Set `AZURE_SUBSCRIPTION_ID` to your subscription ID in your shell. Do not commit it or other secrets.

Create a folder:

```bash
mkdir terraform-azure-lab
cd terraform-azure-lab
```

Create `main.tf`:

```hcl
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

variable "subscription_id" {
  type        = string
  description = "Azure subscription ID"
}

variable "location" {
  type    = string
  default = "centralindia"
}

variable "resource_group_name" {
  type    = string
  default = "rg-terraform-lab"
}

variable "storage_account_name" {
  type        = string
  description = "Globally unique lowercase storage account name"
}

resource "azurerm_resource_group" "lab" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_storage_account" "lab" {
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.lab.name
  location                 = azurerm_resource_group.lab.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  min_tls_version          = "TLS1_2"
  allow_nested_items_to_be_public = false
}
```

Create a local `terraform.tfvars` file with your own values:

```hcl
subscription_id      = "YOUR-SUBSCRIPTION-ID"
storage_account_name = "sttfday62unique123"
```

Replace the storage name with a globally unique name. Keep `terraform.tfvars` out of Git. Add this `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
*.tfplan
crash.log
```

Run the workflow:

```bash
terraform init
terraform fmt -recursive
terraform validate
terraform plan -out=tfplan
terraform apply tfplan
```

Review the plan before applying. Verify:

```bash
az group show --name rg-terraform-lab --output table

az storage account show \
  --name sttfday62unique123 \
  --resource-group rg-terraform-lab \
  --output table
```

Use your actual storage account name in the verification command.

### Clean up Terraform lab

Only after confirming the resource group contains no data or resources you need:

```bash
terraform plan -destroy
terraform destroy
```

Review the destroy plan before confirming. Destroying a storage account can permanently delete its data.

## 5. Lab B — Deploy with Bicep

Use a different resource group and storage account name so the two labs do not conflict.

Create a folder and `main.bicep`:

```bash
mkdir bicep-azure-lab
cd bicep-azure-lab
```

```bicep
param location string = 'centralindia'
param storageAccountName string = 'stbicepday62unique123'

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageAccountName
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
  properties: {
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}

output storageAccountId string = storageAccount.id
```

Replace the name with a globally unique lowercase name.

Sign in and create a separate resource group:

```bash
az login
az account set --subscription "$AZURE_SUBSCRIPTION_ID"

az group create \
  --name rg-bicep-lab \
  --location centralindia
```

Build and validate:

```bash
az bicep build --file main.bicep

az deployment group validate \
  --resource-group rg-bicep-lab \
  --template-file main.bicep
```

Preview before deploying:

```bash
az deployment group what-if \
  --resource-group rg-bicep-lab \
  --template-file main.bicep
```

Deploy:

```bash
az deployment group create \
  --name day62-bicep-deployment \
  --resource-group rg-bicep-lab \
  --template-file main.bicep
```

Verify:

```bash
az storage account show \
  --name stbicepday62unique123 \
  --resource-group rg-bicep-lab \
  --output table
```

Use the actual name you configured.

### Clean up Bicep lab

Inspect the resource group first. If it contains only disposable lab resources:

```bash
az group delete --name rg-bicep-lab --yes --no-wait
```

Deleting a resource group deletes everything inside it. Never run this against a shared or production resource group.

## 6. When should you choose Terraform?

Terraform is a strong fit when:

- You manage infrastructure across multiple clouds.
- You need a broad ecosystem of providers.
- You want a consistent IaC workflow across different platforms.
- Your team can operate and secure Terraform state and remote backends.

## 7. When should you choose Bicep?

Bicep is a strong fit when:

- Your infrastructure is primarily Azure.
- You want close integration with ARM and Azure-native deployment capabilities.
- Your team wants a concise alternative to ARM JSON templates.
- You prefer not to manage a separate Terraform-style state file.

**Both tools can coexist.** Define clear ownership boundaries and avoid independently managing the same resources with both tools unless you have deliberately designed that arrangement.

## 8. Troubleshooting

| Problem | Possible cause | What to check |
|---|---|---|
| Terraform authentication error | Wrong Azure context or missing identity configuration | `az account show`, `az login`, `ARM_SUBSCRIPTION_ID` |
| Provider initialization fails | Provider download/version issue | Network, provider constraint, rerun `terraform init` |
| Storage name unavailable | Name is not globally unique | Choose another lowercase name |
| Terraform plans replacement | Immutable property or configuration change | Inspect the plan before applying |
| Bicep validation fails | Syntax, API version, or property issue | Read the error location and resource schema |
| Authorization failure | Insufficient Azure RBAC | Check identity, role, and scope |

## 9. Production best practices

1. Store IaC in Git and review changes through pull requests.
2. Use remote Terraform state with access control and locking.
3. Never commit state files, credentials, or sensitive variable files.
4. Prefer managed identities or workload identity federation over long-lived secrets where supported.
5. Separate development, test, and production environments.
6. Review Terraform plans and Bicep What-If output before deployment.
7. Use least-privilege Azure RBAC.
8. Pin and review provider versions.
9. Add security, policy, and compliance checks to CI/CD pipelines.
10. Keep infrastructure ownership clear when multiple IaC tools are used.

## 10. Practical learning challenge

- [ ] Explain Terraform providers in your own words.
- [ ] Explain why Terraform needs a state file.
- [ ] Explain how Bicep uses ARM.
- [ ] Deploy a storage account using Terraform.
- [ ] Deploy a different storage account using Bicep.
- [ ] Compare `terraform plan` with Bicep What-If.
- [ ] Clean up both labs safely.
- [ ] Add the examples to your GitHub repo without committing state or local variable files.

### Interview questions

1. What is Infrastructure as Code?
2. What is the difference between Terraform and Azure Bicep?
3. What is a Terraform provider?
4. Why does Terraform maintain a state file?
5. What is a Terraform remote backend?
6. How does Bicep deploy resources to Azure?
7. What is the purpose of `terraform plan`?
8. What is the purpose of Bicep What-If?
9. Can Terraform and Bicep be used in one organization?
10. Which tool would you choose for a multi-cloud environment, and why?

## 11. Screenshot checklist

Capture these for your notes or LinkedIn post:

1. Terraform `init` and `validate` output.
2. Terraform plan showing resources to be created.
3. Terraform-created storage account in Azure Portal.
4. Bicep What-If output.
5. Bicep deployment success in Azure Portal.

Hide subscription IDs, tenant details, tokens, and other sensitive information before publishing.

## 12. Official documentation

- Terraform: https://developer.hashicorp.com/terraform/docs
- Terraform state: https://developer.hashicorp.com/terraform/language/state
- AzureRM remote backend: https://developer.hashicorp.com/terraform/language/backend/azurerm
- Azure Bicep overview: https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview
- Deploy Bicep with Azure CLI: https://learn.microsoft.com/azure/azure-resource-manager/bicep/deploy-cli



The best tool depends on your organization's cloud strategy, governance, team skills, and operational model.

**Next: Day 63/100 — Terraform Fundamentals: Providers, Resources, Variables, and Your First Azure Deployment.**
