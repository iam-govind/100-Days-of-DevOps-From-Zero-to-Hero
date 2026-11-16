# Day 61/100 — Azure Bicep CI/CD with Azure DevOps Pipelines

## Goal
Automate Azure Bicep deployments using Azure DevOps YAML pipelines: build templates, validate them, preview changes with What-If, and deploy to Azure.

## 1. Prerequisites
- Azure subscription and Azure DevOps organization/project.
- Azure Repos Git repository containing your Day 59 Bicep project.
- Permission to create an Azure Resource Manager service connection.
- Existing Azure resource group for the lab.

Suggested structure:

```text
azure-bicep-devops/
├── main.bicep
├── dev.bicepparam
├── modules/storage.bicep
└── azure-pipelines.yml
```

Check local tools:

```bash
az version
az bicep version
```

## 2. Connect Azure DevOps to Azure

1. Open your Azure DevOps project.
2. Select **Project Settings → Service connections**.
3. Select **New service connection → Azure Resource Manager**.
4. Choose workload identity federation if available.
5. Select the Azure subscription and scope access to the lab resource group where supported.
6. Name the connection `sc-azure-bicep-dev` and save.

Prefer workload identity federation over long-lived client secrets. Grant only the permissions needed for deployment.

## 3. Create the pipeline

In Azure DevOps, open **Pipelines → New Pipeline → Azure Repos Git**, select your repository, then select the YAML file `azure-pipelines.yml`.

## 4. Create `azure-pipelines.yml`

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: ubuntu-latest

variables:
  azureServiceConnection: 'sc-azure-bicep-dev'
  resourceGroup: 'rg-bicep-modules'
  location: 'centralindia'

stages:
- stage: Build
  displayName: 'Build Bicep Templates'
  jobs:
  - job: BuildBicep
    steps:
    - checkout: self
    - script: |
        az bicep install
        az bicep version
      displayName: 'Install Bicep'
    - script: az bicep build --file main.bicep
      displayName: 'Build Bicep Template'

- stage: Validate
  displayName: 'Validate Infrastructure'
  dependsOn: Build
  jobs:
  - job: ValidateBicep
    steps:
    - checkout: self
    - task: AzureCLI@2
      displayName: 'Validate and preview deployment'
      inputs:
        azureSubscription: '$(azureServiceConnection)'
        scriptType: bash
        scriptLocation: inlineScript
        inlineScript: |
          az deployment group validate             --resource-group "$(resourceGroup)"             --template-file main.bicep             --parameters dev.bicepparam

          az deployment group what-if             --resource-group "$(resourceGroup)"             --template-file main.bicep             --parameters dev.bicepparam

- stage: Deploy
  displayName: 'Deploy Infrastructure'
  dependsOn: Validate
  condition: succeeded()
  jobs:
  - deployment: DeployBicep
    displayName: 'Deploy to Azure Development'
    environment: 'azure-dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
          - task: AzureCLI@2
            displayName: 'Deploy Bicep template'
            inputs:
              azureSubscription: '$(azureServiceConnection)'
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                az deployment group create                   --name "ado-bicep-$(Build.BuildId)"                   --resource-group "$(resourceGroup)"                   --template-file main.bicep                   --parameters dev.bicepparam
```

The resource group must already exist. The service connection identity needs permission to deploy resources and create/read deployment records at that scope.

## 5. Understand the stages

| Stage | Responsibility |
|---|---|
| Build | Compiles the Bicep template |
| Validate | Checks the template against Azure |
| What-If | Previews proposed changes |
| Deploy | Applies infrastructure changes |

`AzureCLI@2` executes Azure CLI commands using the configured service connection.

## 6. Configure deployment approvals

The deployment job references an Azure DevOps Environment named `azure-dev`.

1. Open **Pipelines → Environments**.
2. Create `azure-dev`.
3. Open **Approvals and checks**.
4. Add an approval if a manual review is required.
5. For production, create a separate environment with stricter approval rules.

The environment must exist, and approval checks must be configured separately from the YAML pipeline.

## 7. Run the pipeline

Commit and push:

```bash
git add .
git commit -m "Add Azure DevOps Bicep pipeline"
git push origin main
```

Open **Pipelines**, select your pipeline, and inspect the Build, Validate, and Deploy stages.

## 8. Verify deployment

```bash
az storage account list   --resource-group rg-bicep-modules   --output table

az deployment group list   --resource-group rg-bicep-modules   --output table
```

Inspect a deployment's outputs:

```bash
az deployment group show   --resource-group rg-bicep-modules   --name "<DEPLOYMENT_NAME>"   --query properties.outputs
```

Replace the placeholder with the actual deployment name.

## 9. Troubleshooting

| Problem | What to check |
|---|---|
| Service connection authorization failed | Identity and Azure RBAC |
| Resource group not found | Subscription and resource group name |
| Bicep build failed | Template syntax and module paths |
| Parameter validation failed | Parameter names and types |
| Pipeline cannot access repository | Repository permissions |
| Approval is pending | Environment approvals and checks |
| Storage name conflict | Use a globally unique name |

## 10. Production best practices

- Use workload identity federation.
- Assign least-privilege RBAC permissions.
- Separate development, testing, and production environments.
- Add approval gates before production deployment.
- Run build, validation, and What-If before deployment.
- Keep environment configuration separate.
- Protect the main branch with pull request policies.
- Keep pipeline YAML and Bicep templates in Git.
- Review deployment logs and retain them for auditing.

## 11. Day 61 practical challenge

1. Import your Day 59 Bicep project into Azure Repos.
2. Create an Azure Resource Manager service connection.
3. Add `azure-pipelines.yml`.
4. Run the Build stage.
5. Review Validate and What-If output.
6. Deploy to your Azure development resource group.
7. Configure an approval check on the deployment environment.
8. Capture screenshots of the successful pipeline.

## 12. Learning questions

1. What is an Azure DevOps Pipeline?
2. What is the purpose of an Azure Resource Manager service connection?
3. What is the difference between Build and Deploy stages?
4. Why run What-If before deployment?
5. How do Azure DevOps Environments help control production deployments?
6. Why is workload identity federation preferred over long-lived secrets?

**Key takeaway:** Bicep defines the infrastructure, while Azure DevOps Pipelines automates how that infrastructure is built, validated, reviewed, and deployed.

## Official resources

- [Azure Pipelines](https://learn.microsoft.com/azure/devops/pipelines/)
- [Azure Resource Manager service connections](https://learn.microsoft.com/azure/devops/pipelines/library/connect-to-azure)
- [Azure DevOps Environments and approvals](https://learn.microsoft.com/azure/devops/pipelines/process/approvals)
- [Azure Bicep](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)

**Cost reminder:** Azure resources may incur charges. Check your subscription and remove lab resources when finished, after confirming you no longer need their data. Deleting a resource group permanently deletes its resources.
