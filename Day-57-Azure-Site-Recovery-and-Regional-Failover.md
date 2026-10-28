# Day 57/100 — Azure Site Recovery and Regional Failover

**100 Days of DevOps — From Zero to Hero**

## Goal
Understand Azure Site Recovery (ASR), replication, recovery points, test failover, recovery plans, and safe disaster recovery drills.

## 1. What is Azure Site Recovery?
Azure Site Recovery helps maintain business continuity by replicating supported workloads to a recovery location. During an outage, you can fail over to that location and later plan a failback.

- **Replication:** Copies VM changes to a recovery location.
- **Recovery point:** A selected state used to recover a replicated VM.
- **Test failover:** A drill that validates recovery without switching off production.
- **Failover:** Switches the workload to the recovery environment.
- **Failback:** Returns the workload to the original environment after required recovery and replication steps.

Azure Backup focuses on data protection and recovery points; Site Recovery focuses on replication and service recovery. Many production systems need both.

## 2. RPO and RTO
- **RPO (Recovery Point Objective):** Maximum tolerable data loss, expressed as time.
- **RTO (Recovery Time Objective):** Target time to restore service.

These are targets, not guarantees. Actual RPO depends on replication and available recovery points. Actual RTO depends on VM startup, networking, application dependencies, and operational steps.

## 3. Hands-on lab — inspect your source VM
Initial commands are read-only. Replication and failover change Azure resources and may incur charges.

```bash
az login
az account list --output table
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"

RG="<YOUR_RESOURCE_GROUP>"
VM="<YOUR_VM_NAME>"

az vm show \
  --resource-group "$RG" \
  --name "$VM" \
  --query "{Name:name,Location:location,Id:id}" \
  --output json
```

Record the source region. Select a supported recovery region that meets your business requirements and is not exposed to the same likely disaster.

```bash
az account list-locations --query "[].name" --output table
TARGET_REGION="<YOUR_RECOVERY_REGION>"

az vm list-skus \
  --location "$TARGET_REGION" \
  --resource-type virtualMachines \
  --output table
```

This is only an initial check; confirm the exact source/target compatibility, VM size, disk, and feature support in Microsoft documentation.

## 4. Enable replication in Azure Portal
1. Search for **Recovery Services vaults**.
2. Select a suitable vault or create one in the appropriate recovery region.
3. Open **Site Recovery**.
4. Choose the option to enable replication for Azure virtual machines.
5. Review source and target regions, subscription, target resource group, network, storage, and replication policy.
6. Start replication only after verifying the configuration.

Before enabling replication, check permissions, supported regions, target VNet/subnet, quotas and VM capacity, cost, application dependencies, DNS, IP addressing, and security rules.

The vault holds Site Recovery configuration information; replicated VM data is handled through the replication architecture, not simply stored as ordinary backup data in the vault.

## 5. Monitor replication health
In the vault, open **Replicated items** and select your VM. Review replication health/status, available recovery points, configuration errors, and the last successful replication information.

Do not run a recovery drill until replication is healthy and a suitable recovery point is available.

## 6. Run a test failover
A test failover validates whether a replicated VM can start in the recovery environment without switching off the production workload. When correctly configured, it should not interrupt ongoing replication.

1. Select the replicated item or a configured recovery plan.
2. Choose **Test failover** and select a suitable recovery point.
3. Select an isolated test VNet/subnet where possible; avoid IP conflicts and unintended production connectivity.
4. Start the test failover.
5. Verify that the test VM boots.
6. Validate application health, ports, dependencies, and expected data.
7. Record recovery time and issues.
8. Choose **Cleanup test failover** and confirm cleanup after testing.

Expected result: the test VM starts, checks pass, and temporary test resources are cleaned up. Never run a production failover just to practise.

## 7. Recovery plans
Recovery plans coordinate failover steps and the order in which related machines start. They can include scripts and manual actions.

Example three-tier application:
1. **Group 1:** Database VM
2. **Group 2:** Application/API VM
3. **Group 3:** Web frontend VM

In Azure Portal, open **Recovery plans** in Site Recovery and create or configure a plan for protected VMs. Add machines in dependency order.

During a test, verify:
- The application VM can reach the database.
- DNS resolves to the intended recovery addresses.
- NSGs and routes are correct.
- Secrets, certificates, and configuration are available.
- The web frontend serves requests successfully.

A VM starting successfully does not prove that the full application recovered.

## 8. Failover vs. failback
- **Planned failover:** Used for planned events when the source is available and the supported workflow can synchronize changes.
- **Unplanned failover:** Used when the primary environment is unavailable; the selected recovery point affects potential data loss.
- **Failback:** Returns the workload to the original environment after required reprotection, replication, and validation.

The exact process depends on the replication scenario. Do not assume every failback is a single click or has zero data loss.

## 9. Troubleshooting

| Symptom | What to investigate |
|---|---|
| Replication won't start | Source/target support, permissions, network, prerequisites, error details |
| Replication health degraded | Replication status, agent/component health, data churn, target resources |
| Test VM won't boot | Recovery point, boot diagnostics, OS configuration, target settings |
| VM boots but app is unavailable | DNS, IP changes, NSGs, routes, service health, dependencies |
| Recovery exceeds target RTO | VM capacity, startup order, scripts, manual steps |
| Test resources remain | Finish cleanup test failover and verify resources |

## 10. Production best practices
- Define and approve RPO/RTO with application owners.
- Monitor replication health and configure alerts.
- Ensure target-region quota and capacity are sufficient.
- Plan networking, DNS, identity, secrets, and dependencies.
- Keep required agents and components updated.
- Use isolated networks for recovery drills.
- Test recovery plans regularly, and record results and remediation.
- Keep Azure Backup and Site Recovery aligned but treat them as distinct capabilities.

## 11. Screenshot checklist
Capture from your own lab, hiding sensitive IDs:
1. Replicated item and replication health.
2. Recovery point information.
3. Recovery plan with startup groups.
4. Test failover job completion.
5. Recovered test VM and application validation.
6. Cleanup confirmation.

Do not claim a successful test unless you actually performed it.

## 12. LinkedIn post — ready to publish

🚀 **Day 57/100 — Azure Site Recovery and Regional Failover**

What happens if your primary Azure region becomes unavailable? How do you bring your application back online without improvising during an incident?

Today I explored Azure Site Recovery (ASR), which helps replicate supported workloads to a recovery location and coordinate failover.

🔹 **What I learned:**
- Replication maintains a recovery copy of supported workloads.
- Recovery points determine the state used during recovery.
- Test failover helps validate disaster recovery without intentionally disrupting production.
- Recovery plans coordinate the startup order of dependent VMs.
- RPO and RTO define business recovery expectations.

🛠️ **Hands-on learning:**
1. Inspected an Azure VM and its region.
2. Reviewed Site Recovery replication setup.
3. Explored replication health and recovery points.
4. Learned the safe test failover and cleanup workflow.
5. Reviewed application dependencies and recovery plan sequencing.

💡 **Key takeaway:** A disaster recovery plan isn't proven by configuration alone. Test the full application recovery process—including networking, DNS, dependencies, and cleanup.

Building resilience is about preparation, validation, and repeatable recovery—not just having a backup.

Continuing my **100 Days of DevOps — From Zero to Hero** journey, one practical lesson at a time. 🚀

Does your team run regular disaster recovery drills?

#100DaysOfDevOps #Azure #AzureSiteRecovery #DisasterRecovery #DevOps #CloudComputing #BusinessContinuity #SRE #LearningInPublic

## 13. Next lesson
**Day 58/100 — Azure Infrastructure as Code with Bicep**

We'll explore declarative infrastructure, parameters, deployments, and reusable templates for Azure resources.

## Official Microsoft documentation
- [Azure Site Recovery overview](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)
- [Replicate Azure VMs to another region](https://learn.microsoft.com/en-us/azure/site-recovery/azure-to-azure-tutorial-enable-replication)
- [Run a test failover](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-test-failover-to-azure)
- [Recovery plan overview](https://learn.microsoft.com/en-us/azure/site-recovery/recovery-plan-overview)
- [Site Recovery reliability recommendations](https://learn.microsoft.com/en-us/azure/reliability/reliability-site-recovery)
