# Day 56/100 — Azure VM Backup and Disaster Recovery

**100 Days of DevOps — From Zero to Hero**  
**Level:** Beginner to production  
**Goal:** Understand Azure VM backups, recovery points, backup policies, restore operations, and disaster recovery planning.

## 1. What is Azure VM Backup?

Azure VM Backup is a managed service that protects supported Azure virtual machines by creating recovery points. These can be used to restore a VM or recover supported data after failures.

Common scenarios include:
- Accidental VM deletion or configuration changes.
- Application failures after a deployment.
- Data corruption or accidental file deletion.
- Recovery after an infrastructure incident.

## 2. Important concepts

| Concept | Meaning |
|---|---|
| Azure Backup | Managed backup service for supported workloads, including Azure VMs |
| Recovery Services vault | Resource used to manage recovery points for supported workloads |
| Recovery point | Saved backup state used for a supported recovery operation |
| Backup policy | Defines backup schedule and retention |
| RPO | Recovery Point Objective: tolerable data loss measured in time |
| RTO | Recovery Time Objective: target time to restore service |

Example: An RPO of one hour aims to limit data loss to roughly the last hour. An RTO of four hours means the recovery plan targets restoring service within four hours. Actual outcomes depend on the design and a tested recovery process.

## 3. Backup vs. disaster recovery

| Backup | Disaster recovery |
|---|---|
| Protects data and VM state | Restores service availability after a major disruption |
| Restores from a recovery point | May fail over to a replicated recovery environment or region |
| Example: recover after accidental deletion | Example: recover after a regional outage |
| Azure Backup | Azure Site Recovery for supported replication and failover scenarios |

Backups and disaster recovery complement each other. A backup vault alone does not provide automatic regional failover.

## 4. Hands-on lab — inspect an Azure VM

The first commands are read-only. Use your existing lab VM if possible.

### Step 1 — Sign in and identify your VM

```bash
az login
az account list --output table
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"

RG="<YOUR_RESOURCE_GROUP>"
VM="<YOUR_VM_NAME>"

az vm get-instance-view \\
  --resource-group "$RG" \\
  --name "$VM" \\
  --output table

az vm show \\
  --resource-group "$RG" \\
  --name "$VM" \\
  --query id \\
  --output tsv
```

Confirm the correct subscription and VM before making changes.

### Step 2 — Check whether backup is already enabled

In Azure Portal:

1. Search for **Backup center**.
2. Open **Backup instances**.
3. Filter by subscription and resource type.
4. Find your VM and review its protection status.

If it is already protected, review the policy and last backup status instead of enabling duplicate protection.

### Step 3 — Create or select a Recovery Services vault

In Azure Portal:

1. Search for **Recovery Services vaults**.
2. Create a vault in an appropriate region or select an existing vault.
3. Review redundancy, security, soft-delete, and retention settings.

**Cost and safety note:** Vault configuration, redundancy, retention, protected instance size, and storage usage can affect cost. Do not change an existing production vault's settings without reviewing the impact.

### Step 4 — Enable VM backup

From the Recovery Services vault:

1. Select **Backup**.
2. Choose **Azure** as the workload location and **Virtual machine** as the workload type.
3. Select an appropriate backup policy.
4. Choose your VM and enable backup.
5. Review the schedule and retention period before applying the configuration.

### Step 5 — Run the first backup

1. Open the vault's **Backup items** section.
2. Select **Azure Virtual Machine**.
3. Open the VM's backup item.
4. Choose **Backup now** if available.
5. Set the requested retention date and start the job.
6. Monitor **Backup jobs** until the job completes.

Expected result: the backup job completes successfully and a recovery point becomes available. The first backup can take time depending on VM size, disk size, data, and configuration.

## 5. Verify using Azure CLI

Check which backup commands are available in your installed CLI version:

```bash
az backup vault list --output table
az backup item list --help
```

Then identify the correct vault and resource group:

```bash
VAULT="<YOUR_RECOVERY_SERVICES_VAULT>"

az backup item list \\
  --resource-group "$RG" \\
  --vault-name "$VAULT" \\
  --backup-management-type AzureIaasVM \\
  --output table
```

CLI options and returned fields may vary by version. If a command isn't recognized, use the Azure Portal or consult the installed CLI command reference before proceeding.

## 6. Restore and disaster recovery planning

A backup is only useful if you can recover from it.

A safe recovery drill:

1. Confirm a successful recovery point exists.
2. Review the available restore options.
3. Restore to a separate test VM or restore disks into a test environment.
4. Verify the operating system boots and application data is usable.
5. Test the application and document the recovery duration.
6. Clean up test resources after validation.

**Never perform a destructive restore against a production VM just to practice.** A restore may overwrite data, create additional resources, or interrupt service depending on the chosen method.

For regional disaster recovery, review Azure Site Recovery, supported replication scenarios, network configuration, failover procedures, and failback requirements.

## 7. Common problems

| Problem | What to check |
|---|---|
| Backup job fails | Open job details and inspect the error code and recommended action |
| No recovery point appears | Confirm the first backup completed successfully |
| Backup cost is higher than expected | Review protected instance size, storage, redundancy, and retention |
| Restore takes longer than expected | Check disk size, target configuration, resource availability, and restore method |
| Restored VM doesn't serve traffic | Check boot status, IP configuration, routes, NSGs, and application services |
| Regional outage affects workload | Verify the recovery plan covers the required region and dependencies |

## 8. Production best practices

- Define RPO and RTO before choosing backup schedules.
- Apply backup policies consistently across production VMs.
- Protect vaults with suitable access controls and security settings.
- Enable supported soft-delete and other appropriate protections.
- Monitor backup failures and configure notifications.
- Align retention with business and compliance requirements.
- Test restores regularly; a successful backup job alone does not prove recovery will work.
- Document application dependencies, DNS changes, network rules, and recovery ownership.
- Treat backup and disaster recovery as separate but coordinated resilience capabilities.

## 9. Screenshot checklist

Capture these from your own lab, hiding subscription IDs and sensitive details:

1. Recovery Services vault overview.
2. Backup policy showing schedule and retention.
3. Backup job showing successful completion.
4. Backup item showing the latest recovery point.
5. A test restore or recovery drill result, if performed.

Do not claim a successful restore unless you have actually tested one.

## 10. LinkedIn post — ready to publish

🚀 **Day 56/100 — Azure VM Backup and Disaster Recovery**

What would happen if your production VM was accidentally deleted, data became corrupted, or an infrastructure failure interrupted your application?

That’s why backup and disaster recovery are essential parts of cloud reliability.

🔹 **What I learned today:**
- Azure Backup protects supported Azure VMs using recovery points.
- Recovery Services vaults help manage backup data and recovery operations.
- Backup policies define schedules and retention.
- RPO describes tolerable data loss; RTO describes the target time to restore service.
- Azure Backup and Azure Site Recovery solve related but different problems.

🛠️ **Hands-on practice:**
1. Inspected an Azure VM using Azure CLI.
2. Reviewed backup protection in Azure Backup Center.
3. Explored Recovery Services vaults and backup policies.
4. Learned how to monitor backup jobs and recovery points.
5. Reviewed a safe restore drill and disaster recovery planning.

💡 **Key takeaway:** A successful backup is only part of the job. You must also test recovery, understand RPO and RTO, and document how the application will return to service.

**Backup protects your data. Tested recovery builds confidence.**

Continuing my **100 Days of DevOps — From Zero to Hero** journey, one hands-on lesson at a time. 🚀

How often does your team test its backups?

#100DaysOfDevOps #Azure #AzureBackup #DisasterRecovery #DevOps #CloudComputing #BusinessContinuity #SRE #LearningInPublic

## 11. Next lesson

**Day 57/100 — Azure Site Recovery and Regional Failover**

We'll explore replication, recovery plans, test failover, and the difference between backup-based recovery and regional disaster recovery.

## Official Microsoft documentation

- [Azure VM backup overview](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vms-introduction)
- [Back up an Azure VM in the portal](https://learn.microsoft.com/en-us/azure/backup/backup-azure-arm-vms)
- [Restore Azure VM data](https://learn.microsoft.com/en-us/azure/backup/backup-azure-arm-restore-vms)
- [Azure Site Recovery overview](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)
