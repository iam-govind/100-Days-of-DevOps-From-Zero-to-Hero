# Day 60/100 — Azure Bicep with CI/CD: Automating Infrastructure Deployments

## Goal
Connect reusable Bicep templates to GitHub Actions. Pull requests build the template; changes pushed to `main` can authenticate to Azure, validate, preview, and deploy infrastructure.

## 1. Prerequisites
- Azure subscription and permission to configure an identity and Azure RBAC.
- GitHub repository with the Day 59 Bicep files.
- An existing Azure resource group.
- Azure CLI and Bicep.

```bash
az version
az bicep version
# If needed:
az bicep install
```

## 2. Repository structure

```text
azure-bicep-cicd/
├── main.bicep
├── dev.bicepparam
├── modules/
│   └── storage.bicep
└── .github/
    └── workflows/
        └── bicep-deploy.yml
```

Reuse the parent template, child module, and parameter file from Day 59. Ensure the storage account name is globally unique.

## 3. Configure Azure authentication with OIDC

Prefer OpenID Connect (OIDC) workload identity federation over a long-lived client secret.

1. Create an app registration or use an appropriate managed identity in Microsoft Entra ID.
2. Configure a federated credential for your GitHub repository and `main` branch.
3. Ensure issuer, subject, and audience match the GitHub Actions identity token.
4. Assign the identity the minimum Azure RBAC permissions needed, scoped to the lab resource group. Contributor at resource-group scope is a common lab starting point; production permissions should be narrower where practical.
5. Add these GitHub Actions repository variables:

| Variable | Value |
|---|---|
| `AZURE_CLIENT_ID` | Application or managed identity client ID |
| `AZURE_TENANT_ID` | Microsoft Entra tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Azure subscription ID |

The workflow uses a GitHub environment named `dev`. If the federated credential trusts an environment subject instead of a branch subject, configure it to match that environment. Never commit passwords, client secrets, or access keys.

Guide: https://learn.microsoft.com/azure/developer/github/connect-from-azure

## 4. Create the GitHub Actions workflow

Create `.github/workflows/bicep-deploy.yml`:

```yaml
name: Azure Bicep CI-CD

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

env:
  RESOURCE_GROUP: rg-bicep-modules
  LOCATION: centralindia

jobs:
  build:
    name: Build Bicep
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install Bicep
        run: |
          az bicep install
          az bicep version

      - name: Build Bicep template
        run: az bicep build --file main.bicep

  deploy:
    name: Validate and deploy to Azure
    needs: build
    if: github.event_name != 'pull_request'
    runs-on: ubuntu-latest
    environment: dev
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Azure login using OIDC
        uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

      - name: Install Bicep
        run: az bicep install

      - name: Validate deployment
        run: |
          az deployment group validate             --resource-group "$RESOURCE_GROUP"             --template-file main.bicep             --parameters dev.bicepparam

      - name: Preview changes
        run: |
          az deployment group what-if             --resource-group "$RESOURCE_GROUP"             --template-file main.bicep             --parameters dev.bicepparam

      - name: Deploy infrastructure
        run: |
          az deployment group create             --name "github-actions-${{ github.run_number }}"             --resource-group "$RESOURCE_GROUP"             --template-file main.bicep             --parameters dev.bicepparam
```

The resource group must already exist. The Azure identity needs permission to deploy resources and create/read deployment records at that scope.

## 5. Configure GitHub variables and environment

In your repository:
1. Open **Settings → Secrets and variables → Actions**.
2. Under **Variables**, add the three Azure variables.
3. Under **Environments**, create an environment named `dev`.

For production, use a separate environment, identity, resource group, parameter file, and approval rules. Match the federated credential to the branch or environment subject you choose.

## 6. Run the pipeline

```bash
git add .
git commit -m "Add Azure Bicep CI CD pipeline"
git push origin main
```

In GitHub, open **Actions → Azure Bicep CI-CD** and inspect the build and deployment job logs.

## 7. Test pull requests

```bash
git checkout -b feature/update-bicep
git add .
git commit -m "Update Bicep template"
git push origin feature/update-bicep
```

Open a pull request against `main`. The build job should run; deployment should be skipped because the workflow does not deploy from pull requests. This prevents untrusted pull request code from receiving deployment permissions.

## 8. Pipeline stages

| Stage | Purpose |
|---|---|
| Checkout | Gets repository files |
| Install Bicep | Makes Bicep CLI available |
| Build | Compiles the template |
| Azure login | Authenticates through OIDC |
| Validate | Checks the deployment |
| What-If | Previews proposed changes |
| Deploy | Applies the template to Azure |

## 9. Production best practices

- Prefer OIDC over long-lived client secrets.
- Use least-privilege Azure RBAC scoped to the required resource group.
- Protect `main` with pull request reviews.
- Use GitHub Environments and approval gates for production.
- Run build, validation, and What-If before deployment.
- Use separate parameters and identities for each environment.
- Pin third-party GitHub Actions to reviewed commit SHAs in production.
- Review deployment output and retain logs for auditing.
- Never deploy production infrastructure from an unreviewed pull request.

## 10. Troubleshooting

| Error | What to check |
|---|---|
| Azure login failed | OIDC issuer, subject, and audience |
| Authorization failed | Azure RBAC role and scope |
| Resource group not found | Group exists in selected subscription |
| Bicep build failed | Syntax and module paths |
| Parameter validation failed | Parameter names, types, and values |
| Storage name conflict | Use a globally unique name |
| Workflow not triggered | YAML path, branch, and event configuration |

## 11. Day 60 practical challenge

1. Create the GitHub Actions workflow.
2. Configure Azure OIDC authentication.
3. Push Bicep code to GitHub.
4. Verify the build job.
5. Review What-If output.
6. Deploy to the Azure development resource group.
7. Open a pull request and verify deployment is skipped.

Capture screenshots of the workflow run and Azure deployment for your GitHub notes.

## 12. Official learning resources

- [GitHub Actions documentation](https://docs.github.com/actions)
- [Azure Login GitHub Action](https://github.com/Azure/login)
- [Configure OIDC with Azure](https://learn.microsoft.com/azure/developer/github/connect-from-azure)
- [Azure Bicep documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)

**Key takeaway:** GitHub Actions helps build, validate, review, and deploy Azure infrastructure through a repeatable CI/CD pipeline.

**Cost reminder:** Azure resources may incur charges. Check your subscription and clean up lab resources when finished, after confirming you do not need their data. Deleting a resource group permanently deletes its resources.
