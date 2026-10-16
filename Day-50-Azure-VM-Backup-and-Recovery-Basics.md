# 100 Days of DevOps --- Day 50/100

## Azure VM Backup and Recovery Basics

**Date:** 16 October 2026\
**Level:** Beginner to intermediate\
**Environment:** Azure Portal + Azure CLI on macOS Terminal or Azure
Cloud Shell

> **Continuity:** Day 49 covered Azure VM monitoring and
> troubleshooting. Day 51 will continue with a safe, hands-on restore
> workflow.

## 1. Why backup matters

Monitoring helps us detect a problem. Backup helps us recover from
accidental deletion, a failed change, corruption, or a VM outage.

Two key terms:

-   **RPO (Recovery Point Objective):** The maximum amount of data loss
    the business can tolerate, expressed as time.
-   **RTO (Recovery Time Objective):** The target time to restore the
    service after an outage.

A backup is useful only when it is recent enough, available, and
recoverable. A successful backup job does not prove the application can
be fully restored; recovery should be tested.

## 2. Azure Backup concepts

-   **Azure Backup:** Managed Azure backup service.
-   **Recovery Services vault:** A management and storage container for
    backup data and recovery points for supported workloads, including
    Azure VMs.
-   **Backup policy:** Defines schedule and retention.
-   **Recovery point:** A point-in-time version used to recover data.
-   **Backup job:** The operation that creates or manages a backup.

### Workflow

``` text
Azure VM
   ↓
Recovery Services vault + backup policy
   ↓
Scheduled or on-demand backup job
   ↓
Recovery point
   ↓
Select recovery point when recovery is needed
   ├── Restore files
   ├── Restore disks
   └── Recover/create a VM (supported scenarios)
```

## 3. Before you begin: safety and cost

1.  Use only a VM and subscription you are authorized to manage.
2.  Check the VM's resource group and region.
3.  Azure Backup may incur charges for protected instances, backup
    storage, snapshots, and related resources. Check current pricing and
    any trial-credit limits before enabling it.
4.  Do not delete a vault or recovery points without understanding the
    impact on recovery.
5.  For production, follow your organization's approval, retention, and
    security requirements.

The commands below use placeholders. Replace them with your own values.
Never post credentials, private keys, tokens, or sensitive subscription
details publicly.

## 4. Hands-on lab: protect an existing VM

### Step 1 --- Sign in and confirm the subscription

``` bash
az login
az account show --output table
az account list --output table
```

If needed, select the intended subscription:

``` bash
az account set --subscription "<YOUR_SUBSCRIPTION_ID_OR_NAME>"
az account show --query "{subscription:name, id:id, tenant:tenantId}" --output table
```

**Expected result:** The active subscription is the one where your test
VM lives.

### Step 2 --- Set variables and inspect the VM

Replace the resource group and VM name:

``` bash
RG="<YOUR_VM_RESOURCE_GROUP>"
VM="<YOUR_VM_NAME>"

az vm show --resource-group "$RG" --name "$VM" \
  --query "{name:name, resourceGroup:resourceGroup, location:location, provisioningState:provisioningState}" \
  --output table

LOCATION=$(az vm show --resource-group "$RG" --name "$VM" --query location --output tsv)
echo "$LOCATION"
```

**Expected result:** You see the VM's name, resource group, location,
and provisioning state.

### Step 3 --- Create a Recovery Services vault

Choose a valid, unique name. Change the example if it already exists.

``` bash
VAULT="rsvdevops50"

az backup vault create \
  --resource-group "$RG" \
  --name "$VAULT" \
  --location "$LOCATION"

az backup vault show \
  --resource-group "$RG" \
  --name "$VAULT" \
  --output table
```

**Expected result:** Azure returns the vault details in the VM's region.

### Step 4 --- Review the backup policy

``` bash
az backup policy list \
  --resource-group "$RG" \
  --vault-name "$VAULT" \
  --output table
```

Look for a policy such as `DefaultPolicy`. Review the actual schedule
and retention; do not assume defaults are suitable for production.

### Step 5 --- Enable protection

If `DefaultPolicy` exists and is appropriate for this lab:

``` bash
az backup protection enable-for-vm \
  --resource-group "$RG" \
  --vault-name "$VAULT" \
  --vm "$VM" \
  --policy-name "DefaultPolicy"
```

**Expected result:** Azure accepts the protection request. Configuration
and the first backup may take time. Acceptance of this command alone
does not prove a completed recovery point exists.

If the VM and vault are in different resource groups, the vault resource
group is used for the vault scope, and you may need to pass the VM's
full resource ID rather than its name. Check the official CLI guide
before adapting the command.

### Step 6 --- Verify backup items and jobs

``` bash
az backup item list \
  --resource-group "$RG" \
  --vault-name "$VAULT" \
  --backup-management-type AzureIaaSVM \
  --output table

az backup job list \
  --resource-group "$RG" \
  --vault-name "$VAULT" \
  --output table
```

**Expected result:** Identify the protected VM and inspect the jobs.
Wait until the relevant backup job reports `Completed`.

### Step 7 --- Run an on-demand backup (optional)

**Portal method:**

1.  Open the Azure portal and select **Virtual machines**.
2.  Select your VM, then open **Backup**.
3.  Select **Backup now**.
4.  Choose the retention date and start the job.
5.  Open the backup jobs view and wait for completion.

**Azure CLI option:** The command below follows Microsoft's quickstart
simple-name example. Depending on how your protected item is registered,
use the exact container and item identifiers returned by Azure.

``` bash
az backup protection backup-now \
  --resource-group "$RG" \
  --vault-name "$VAULT" \
  --container-name "$VM" \
  --item-name "$VM" \
  --backup-management-type AzureIaaSVM \
  --retain-until 16-11-2026
```

If Azure says the container or item cannot be found, do not guess. List
containers and backup items, use the exact identifiers returned, or run
the backup from the portal.

### Step 8 --- List recovery points

After a backup job completes:

``` bash
az backup recoverypoint list \
  --resource-group "$RG" \
  --vault-name "$VAULT" \
  --backup-management-type AzureIaaSVM \
  --container-name "$VM" \
  --item-name "$VM" \
  --output table
```

**Expected result:** At least one recovery point appears after the
initial backup finishes. If none appears, inspect the job status and
verify the protected-item identifiers.

> **Key check:** A vault existing is not enough. Confirm a completed
> backup job and a visible recovery point.

## 5. Understand restore choices

  -----------------------------------------------------------------------
  Incident                            Typical approach
  ----------------------------------- -----------------------------------
  One file was deleted                Use the supported file-recovery
                                      workflow for the chosen recovery
                                      point

  A disk needs recovery               Restore the disk to a target
                                      resource group, then attach/use it
                                      as appropriate

  A VM must be recovered              Use a supported VM restore workflow

  Application/database data needs     Follow application- or
  recovery                            database-specific recovery
                                      procedures too
  -----------------------------------------------------------------------

For a beginner lab, do not replace a production VM or overwrite disks.
Review the restore wizard first, or restore into an isolated test
resource group after confirming permissions, networking, naming, and
cost.

### Portal restore walkthrough (review only)

1.  Open the VM's **Backup** page or its Recovery Services vault.
2.  Locate the protected item and recovery points.
3.  Select the appropriate supported option, such as **Restore VM** or
    **Restore disks**.
4.  Choose a recovery point and review the target settings.
5.  Confirm destination resource group, network, storage, and naming.
6.  Review impact and cost; start only when authorized.
7.  Monitor the restore job and validate the recovered resource before
    directing traffic to it.

A restored VM may still need network, identity, access, monitoring, and
application configuration before it is production-ready.

## 6. Troubleshooting

  -----------------------------------------------------------------------
  Problem                             What to check
  ----------------------------------- -----------------------------------
  Vault creation fails                Subscription, resource group,
                                      region, naming, permissions, quota

  `DefaultPolicy` not found           Run `az backup policy list` and
                                      choose a policy that exists in the
                                      vault

  VM not found                        Recheck `$RG`, `$VM`, active
                                      subscription, and VM resource group

  Backup job remains `InProgress`     Wait, inspect job details, and
                                      review Azure Backup alerts

  No recovery points                  Confirm initial backup completed
                                      and query the correct protected
                                      item

  Container/item not found            List containers and items; use
                                      exact identifiers from Azure

  Permission denied                   Check RBAC access to the VM, vault,
                                      and restore target

  Restore fails                       Check target-region support,
                                      permissions, naming conflicts, and
                                      job error details
  -----------------------------------------------------------------------

Useful checks:

``` bash
az account show --output table
az vm show --resource-group "$RG" --name "$VM" --output table
az backup vault list --output table
az backup policy list --resource-group "$RG" --vault-name "$VAULT" --output table
az backup job list --resource-group "$RG" --vault-name "$VAULT" --output table
```

## 7. Production best practices

-   Agree RPO and RTO with application owners before choosing a policy.
-   Set retention to satisfy business, legal, and security requirements.
-   Monitor backup failures and alerts.
-   Restrict who can disable protection, delete recovery points, or
    remove a vault.
-   Evaluate vault redundancy and cross-region recovery when required;
    validate support and cost first.
-   Treat backups as sensitive data and control access.
-   Test restores periodically in an isolated environment.
-   Document dependencies, restore steps, validation checks, and
    escalation contacts.
-   Review charges and clean up only lab resources you own and no longer
    need.

## 8. Day 50 challenge

-   [ ] Identify an authorized test VM and confirm its resource group
    and region.
-   [ ] Create or identify a Recovery Services vault.
-   [ ] Review the backup policy.
-   [ ] Enable protection for the test VM.
-   [ ] Confirm the relevant backup job reaches `Completed`.
-   [ ] List and record at least one recovery point.
-   [ ] Explain RPO vs. RTO in your own words.
-   [ ] Review the restore workflow without overwriting production
    resources.

### Reflection questions

1.  Why is a `Completed` backup job more meaningful than simply seeing a
    vault?
2.  How would a 24-hour RPO affect a system that changes data every
    minute?
3.  Why should a restore be tested in an isolated environment first?
4.  What else may be required before a restored VM can serve production
    traffic?

## 9. LinkedIn screenshot checklist

Capture 2--3 screenshots:

1.  Recovery Services vault with its name and region.
2.  Backup status or a completed backup job.
3.  Recovery points listed for the VM.

A terminal screenshot of `az backup job list` or
`az backup recoverypoint list` is also useful.

Before posting, blur subscription IDs, tenant IDs, public IP addresses,
sensitive resource names, and organization-specific details. Never show
credentials, SSH private keys, tokens, or secret values.

## 10. LinkedIn post --- Day 50/100

**100 Days of DevOps --- Day 50/100 🚀**

**Azure VM Backup and Recovery Basics**

Deploying an application is only one part of operating it reliably. We
also need a plan for recovering when data is deleted, a change goes
wrong, or a virtual machine becomes unavailable.

Today, I explored how Azure Backup protects an Azure Virtual Machine and
how to verify that recovery points are available.

🛠️ **What I practiced:**

✅ Reviewed Recovery Services vaults and backup policies\
✅ Enabled backup protection for a test VM\
✅ Checked backup jobs using Azure CLI\
✅ Learned how to list available recovery points\
✅ Reviewed options for restoring files, disks, or a VM\
✅ Connected backup planning with RPO and RTO

💡 **My biggest takeaway:**

**A backup is not a recovery plan until you know how to restore it.**

Reliable operations require more than monitoring and alerts. We need
clear retention policies, controlled access, visible backup status, and
regular recovery testing.

🎯 **Question for the DevOps community:**

How often does your team test its backups by performing an actual
restore?

Continuing my **100 Days of DevOps** journey --- one practical lesson at
a time.

#100DaysOfDevOps #Azure #AzureBackup #CloudComputing #DevOps
#DisasterRecovery #BusinessContinuity #LearningInPublic

## 11. GitHub update

Save this lesson as `Day-50-Azure-VM-Backup-and-Recovery-Basics.md`.

From the repository root:

``` bash
git add Day-50-Azure-VM-Backup-and-Recovery-Basics.md
git commit -m "Add Day 50 Azure VM backup and recovery lesson"
git push
```

If your repository stores lessons in a subfolder, adjust the path in
`git add`.

## 12. Next lesson

**Day 51 --- Azure VM Restore and Recovery Hands-On**

We will build on today's backup by exploring recovery points and
practising a safe restore workflow, focusing on target resources,
validation, and avoiding accidental changes to production systems.

## Official Microsoft references

-   [Quickstart: Back up a VM with Azure
    CLI](https://learn.microsoft.com/en-us/azure/backup/quick-backup-vm-cli)
-   [Back up an Azure VM from the VM
    settings](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vms-first-look-arm)
-   [Tutorial: Restore a VM with Azure
    CLI](https://learn.microsoft.com/en-us/azure/backup/tutorial-restore-disk)
-   [Restore files to a VM with Azure
    Backup](https://learn.microsoft.com/en-us/azure/backup/tutorial-restore-files)

Check the official documentation for current prerequisites and supported
restore scenarios before applying these steps to production.
