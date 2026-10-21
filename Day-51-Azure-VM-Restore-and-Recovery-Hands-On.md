# 100 Days of DevOps --- Day 51/100

## Azure VM Restore and Recovery Hands-On

**Date:** 17 October 2026\
**Level:** Beginner to intermediate\
**Environment:** Azure Portal, Azure CLI, macOS Terminal or Azure Cloud
Shell

> Day 49 covered monitoring and troubleshooting; Day 50 covered Azure VM
> backup basics. Today we inspect recovery points and plan a safe
> restore.

## 1. Goal

A backup is only useful if you can recover from it. Today you will: -
Verify backup jobs and recovery points. - Inspect protected items with
Azure CLI. - Review the Azure Portal restore workflow. - Plan a restore
to a separate test VM, not over a production VM. - Validate
infrastructure and application health after recovery.

**Cost and safety:** Restores can create billable VMs, disks, and
storage. Use only resources you are authorized to manage. Check cost,
permissions, and target settings before starting. Do not choose "Replace
existing disks" for this beginner exercise.

## 2. Restore options

  -----------------------------------------------------------------------
  Option                              Meaning
  ----------------------------------- -----------------------------------
  Create a new VM                     Creates a separate VM from a
                                      recovery point; generally the safer
                                      practice path

  Restore disks                       Recovers disks for a more
                                      customized VM recovery

  Replace existing disks              Replaces disks on an existing VM;
                                      high-impact and not for this lab

  File recovery                       Recovers selected files in
                                      supported scenarios
  -----------------------------------------------------------------------

Available choices depend on the VM, recovery point, vault configuration,
and supported scenario.

## 3. Inspect backup state with Azure CLI

### Step 1 --- Sign in and select the subscription

``` bash
az login
az account show --output table
az account set --subscription "<YOUR_SUBSCRIPTION_ID_OR_NAME>"
```

### Step 2 --- Set variables

Replace the placeholders with your values:

``` bash
VAULT_RG="<YOUR_VAULT_RESOURCE_GROUP>"
VAULT="<YOUR_RECOVERY_SERVICES_VAULT>"
VM_RG="<YOUR_VM_RESOURCE_GROUP>"
VM="<YOUR_VM_NAME>"
```

Check the vault and VM:

``` bash
az backup vault show --resource-group "$VAULT_RG" --name "$VAULT" --output table

az vm show --resource-group "$VM_RG" --name "$VM" \
  --query "{name:name,location:location,provisioningState:provisioningState}" \
  --output table
```

**Expected result:** Azure returns the vault and VM details.

### Step 3 --- Check backup jobs

``` bash
az backup job list \
  --resource-group "$VAULT_RG" \
  --vault-name "$VAULT" \
  --output table
```

Find the most recent backup job and verify it completed successfully. If
it is running or failed, inspect its details in the portal before
proceeding.

### Step 4 --- List protected items

``` bash
az backup item list \
  --resource-group "$VAULT_RG" \
  --vault-name "$VAULT" \
  --backup-management-type AzureIaaSVM \
  --output table
```

Identify the correct protected item. If names are similar, use the exact
identifiers returned by Azure instead of assuming a friendly VM name is
unique.

### Step 5 --- List recovery points

Use the exact container and item names returned by Azure:

``` bash
az backup recoverypoint list \
  --resource-group "$VAULT_RG" \
  --vault-name "$VAULT" \
  --backup-management-type AzureIaaSVM \
  --container-name "<EXACT_CONTAINER_NAME>" \
  --item-name "<EXACT_ITEM_NAME>" \
  --output table
```

If you do not know the identifiers, inspect the output of
`az backup container list` and `az backup item list`.

**Expected result:** One or more recovery points appear. Record the
chosen recovery point's name and timestamp.

> If no recovery point appears, stop and inspect the backup job, vault,
> protected item, and identifiers before trying to restore.

## 4. Review the restore workflow in Azure Portal

1.  Sign in to the Azure portal and open **Virtual machines**.
2.  Select the protected test VM and open **Backup**, or open the
    associated Recovery Services vault.
3.  Open the restore/recovery experience and select the protected VM.
4.  Choose the recovery point you identified above.
5.  Review the available restore options.

### Choose a safe target

For practice, choose **Create new virtual machine** when supported and
appropriate.

-   Give the new VM a distinct name.
-   Use an isolated test resource group.
-   Select an appropriate virtual network and subnet in a supported
    region.
-   Review staging/storage requirements and estimated charges.
-   Ensure the recovered VM will not accidentally receive production
    traffic.

**Do not select "Replace existing disks" for this exercise.**

Before starting, verify the recovery point, target name, resource group,
network, permissions, and expected cost. If you are not authorized or
ready to create resources, stop after reviewing the wizard and capture
the screen as a learning artifact.

### Monitor the restore

Use the vault's **Backup Jobs** or the VM's backup experience to follow
the job. If it fails, open the job details and record the failed subtask
and error message.

**Expected result:** When the job completes, Azure creates the target
resources for the selected restore path. Timing varies by workload and
recovery scenario.

## 5. Validate the recovered VM

A completed restore job does not prove the application is healthy.

For a newly created VM:

``` bash
TARGET_RG="<YOUR_TEST_TARGET_RESOURCE_GROUP>"
TARGET_VM="<YOUR_RESTORED_VM_NAME>"

az vm show \
  --resource-group "$TARGET_RG" \
  --name "$TARGET_VM" \
  --query "{name:name,location:location,provisioningState:provisioningState}" \
  --output table

az vm get-instance-view \
  --resource-group "$TARGET_RG" \
  --name "$TARGET_VM" \
  --query "instanceView.statuses[].displayStatus" \
  --output table
```

**Expected result:** You can confirm that the target VM exists and
inspect its provisioning and power-state information.

Then validate: - Expected disks are attached. - Network, subnet, NSG,
and IP configuration are correct. - You can access the VM through the
intended management path. - Required services are running. -
Application/system logs do not show blocking errors. - Data and
configuration from the chosen recovery point are present. - The VM is
isolated from production until validation is complete.

A recovery point represents a prior state; changes made after that point
might not be present.

## 6. Troubleshooting

  -----------------------------------------------------------------------
  Problem                             What to check
  ----------------------------------- -----------------------------------
  No recovery points                  Confirm the backup job completed;
                                      verify the vault and protected item
                                      identifiers.

  Protected item not found            Check the active subscription,
                                      vault resource group, and exact CLI
                                      output.

  Restore option unavailable          Review the recovery scenario, vault
                                      configuration, and current
                                      Microsoft support documentation.

  Permission denied                   Check Azure RBAC permissions on the
                                      vault and target resources.

  Name conflict                       Use a unique target VM name or
                                      isolated resource group.

  Wrong/missing network               Verify region, VNet, subnet, NSG,
                                      and IP configuration.

  Restore job fails                   Inspect the job and failed subtask
                                      details; follow the specific error
                                      message.

  VM starts but app fails             Check service status, logs, DNS,
                                      firewall/NSG rules, configuration,
                                      and dependencies.

  Unexpected cost                     Review the restored VM, disks, and
                                      staging resources; clean up only
                                      resources you own and no longer
                                      need.
  -----------------------------------------------------------------------

## 7. Production recovery checklist

-   [ ] Intended recovery point selected.
-   [ ] Restore job completed successfully.
-   [ ] Target infrastructure is healthy.
-   [ ] Network and access controls are correct.
-   [ ] Application and dependencies work.
-   [ ] Data consistency is checked.
-   [ ] Monitoring and alerting are enabled.
-   [ ] Service owner accepts the recovered service.
-   [ ] Actual RPO/RTO outcomes are recorded.
-   [ ] Temporary resources are tracked for cleanup.

Follow organizational incident and disaster-recovery procedures for
production. Do not delete recovery points or source disks as improvised
cleanup.

## 8. Day 51 challenge

1.  Find an authorized test VM and verify a completed backup job.
2.  List its recovery points and record one timestamp.
3.  Open the restore wizard and identify the available options.
4.  Document the target resource group, VM name, network, and validation
    steps.
5.  If authorized and budgeted, restore to a separate test VM;
    otherwise, stop before starting the restore.
6.  Write down how you would prove the application is healthy after
    recovery.

### Reflection questions

1.  Why is a new VM safer for practice than replacing existing disks?
2.  What could be wrong even if the restored VM is running?
3.  How does the selected recovery point affect the latest data
    available?
4.  What evidence would show that recovery met the team's RTO?

## 9. LinkedIn screenshot checklist

Capture 2--3 screenshots: 1. Backup page showing the protected VM and
backup status. 2. Recovery-point list with the selected timestamp. 3.
Restore wizard showing the **new VM** target, or a completed restore job
if you actually performed it.

Before posting, hide subscription/tenant IDs, public IPs, sensitive
resource names, and organization-specific details. Never expose
credentials, private keys, access tokens, or secrets. Do not claim a
restore was completed if you only reviewed the wizard.

## 10. LinkedIn post --- Day 51/100

**100 Days of DevOps --- Day 51/100 🚀**

**Azure VM Restore and Recovery Hands-On**

Backups matter---but recovery is where we prove whether our backup
strategy can help when something goes wrong.

Today, I explored the Azure VM recovery workflow and practised checking
backup jobs, protected items, and recovery points before planning a
restore.

🛠️ **What I learned:**

✅ Verified backup status with Azure CLI\
✅ Inspected protected items and recovery points\
✅ Reviewed the Azure Portal restore workflow\
✅ Learned why restoring to a separate test VM is safer for practice\
✅ Identified infrastructure and application checks to run after
recovery\
✅ Connected restore validation with RPO and RTO

💡 **My biggest takeaway:**

**A VM that starts is not necessarily an application that has
recovered.**

A reliable recovery process checks the recovery point, restore job,
infrastructure, networking, application dependencies, and data---not
just the VM's power state.

🎯 **Question for the DevOps community:**

After restoring a VM, what is the first validation you perform before
allowing users back onto the application?

Continuing my **100 Days of DevOps** journey---learning by practising,
validating, and documenting.

#100DaysOfDevOps #Azure #AzureBackup #DisasterRecovery #CloudComputing
#DevOps #SiteReliabilityEngineering #LearningInPublic

## 11. GitHub update

Save this file as `Day-51-Azure-VM-Restore-and-Recovery-Hands-On.md`.

From your repository root:

``` bash
git add Day-51-Azure-VM-Restore-and-Recovery-Hands-On.md
git commit -m "Add Day 51 Azure VM restore and recovery lesson"
git push
```

If lessons are stored in a subfolder, adjust the `git add` path.

## 12. Next lesson

**Day 52 --- Azure Networking Troubleshooting: NSGs, Routes, and
Connectivity**

We will investigate how network security groups, routes, and
connectivity checks affect communication with Azure resources.

## Official Microsoft references

-   [Restore VMs by using the Azure
    portal](https://learn.microsoft.com/en-us/azure/backup/backup-azure-arm-restore-vms)
-   [Manage Azure VM recovery
    points](https://learn.microsoft.com/en-us/azure/backup/manage-recovery-points)
-   [Azure Backup restore CLI
    reference](https://learn.microsoft.com/en-us/cli/azure/backup/restore)
-   [Back up an Azure VM from the VM
    settings](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vms-first-look-arm)

Check current Microsoft documentation for supported options and
prerequisites before restoring a production workload.
