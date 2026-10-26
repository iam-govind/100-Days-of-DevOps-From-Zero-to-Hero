# Day 55/100 — Azure Monitor, Log Analytics, and VM Troubleshooting

**100 Days of DevOps — From Zero to Hero**

## Goal
Monitor Azure VM metrics, query collected logs with KQL, and troubleshoot Linux VM and application problems systematically.

## 1. Key concepts

- **Azure Monitor Metrics:** Numerical measurements over time, such as CPU percentage.
- **Azure Monitor Logs:** Queryable records from configured sources.
- **Log Analytics workspace:** Stores and queries supported logs using Kusto Query Language (KQL).
- **Azure Monitor Agent (AMA):** Collects configured guest OS data.
- **Data Collection Rule (DCR):** Specifies which data to collect and where to send it.
- **Azure Monitor Alerts:** Evaluate metrics or logs against conditions and notify action groups.

Platform metrics may be available without a guest agent. Guest OS logs and additional guest performance data require suitable collection configuration.

## 2. Metrics vs. logs

| Metrics | Logs |
|---|---|
| Numerical measurements | Records and events |
| Detect patterns such as high CPU | Investigate why an event happened |
| Example: CPU reached 95% | Example: application logs show database timeouts |
| Metrics Explorer | Log Analytics / Logs |

A running VM does not necessarily mean its application is healthy. Combine infrastructure metrics with logs and application checks.

## 3. Hands-on lab: inspect a VM

The following initial commands are read-only. Use your existing lab VM if possible.

### Step 1 — Sign in and select your subscription

```bash
az login
az account list --output table
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
az account show --output table

RG="<YOUR_RESOURCE_GROUP>"
VM="<YOUR_VM_NAME>"
```

Check VM state:

```bash
az vm get-instance-view \
  --resource-group "$RG" \
  --name "$VM" \
  --output table
```

Inspect VM configuration and IPs:

```bash
az vm show \
  --resource-group "$RG" \
  --name "$VM" \
  --query "{Name:name,Location:location,Size:hardwareProfile.vmSize}" \
  --output table

az vm list-ip-addresses --resource-group "$RG" --name "$VM" --output table
```

### Step 2 — Inspect Azure platform metrics

In Azure Portal, open **Virtual machines → your VM → Monitoring → Metrics**. Select **Percentage CPU**, set the time range to the last hour, and look for spikes or sustained high CPU.

Or use Azure CLI:

```bash
RESOURCE_ID=$(az vm show \
  --resource-group "$RG" \
  --name "$VM" \
  --query id --output tsv)

az monitor metrics list-definitions --resource "$RESOURCE_ID" --output table

az monitor metrics list \
  --resource "$RESOURCE_ID" \
  --metric "Percentage CPU" \
  --interval PT5M \
  --output table
```

Expected result: timestamped CPU measurements if the metric is available and data exists.

### Step 3 — Inspect Linux

SSH to the VM, then run:

```bash
uptime
top
free -h
df -h
lsblk
vmstat 1 5
```

For Nginx:

```bash
sudo systemctl status nginx --no-pager
sudo ss -lntp
curl -I http://localhost
```

Apache may be called `apache2` on Ubuntu or `httpd` on other distributions.

## 4. Configure Log Analytics

1. In Azure Portal, create or select a **Log Analytics workspace**.
2. Open the VM's monitoring or Insights configuration.
3. Enable guest monitoring with **Azure Monitor Agent (AMA)** and a suitable **Data Collection Rule (DCR)**.
4. Configure the DCR to collect required Linux Syslog facilities/severities and guest performance data.
5. Confirm the destination workspace and wait for data ingestion.

**Important:** Creating a workspace alone does not automatically send every VM log to it. The agent, DCR, destination, permissions, and data sources must be configured. Collect only what you need to control cost.

## 5. KQL query examples

Open the workspace and select **Logs**.

### Recent heartbeats

```kusto
Heartbeat
| where TimeGenerated > ago(1h)
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| order by LastHeartbeat desc
```

This checks which computers recently reported heartbeat records. It does not prove application health.

### Linux Syslog records

```kusto
Syslog
| where TimeGenerated > ago(1h)
| project TimeGenerated, Computer, HostIP, Facility, SeverityLevel, SyslogMessage
| order by TimeGenerated desc
| take 50
```

### Recent severe Syslog messages

```kusto
Syslog
| where TimeGenerated > ago(1h)
| where SeverityLevel in ("err", "crit", "alert", "emerg")
| project TimeGenerated, Computer, ProcessName, SeverityLevel, SyslogMessage
| order by TimeGenerated desc
```

### Discover collected performance counters

```kusto
Perf
| where TimeGenerated > ago(1h)
| summarize Samples = count() by Computer, ObjectName, CounterName
| order by Samples desc
```

These queries require the relevant tables to exist and contain data. If a table is missing or empty, check AMA health, the DCR, selected data sources, workspace destination, permissions, and ingestion delay.

## 6. Troubleshooting challenge: website is down

Follow this sequence:

1. Check Azure VM power state and boot diagnostics.
2. Review CPU metrics and Linux system load.
3. Check memory with `free -h`.
4. Check filesystem capacity with `df -h`.
5. Check the service with `systemctl status nginx`.
6. Check listening ports with `sudo ss -lntp`.
7. Test locally using `curl -I http://localhost`.
8. Check NSGs, routes, and required ports.
9. Inspect system and application logs around the failure time.
10. If logs are missing, verify AMA, the DCR, and workspace destination.

Do not restart services or change firewall rules before collecting evidence unless your incident procedure requires immediate action.

## 7. Create a CPU alert

In Azure Portal:

1. Open the VM and select **Alerts**.
2. Create an alert rule with the VM as scope.
3. Select the `Percentage CPU` metric.
4. Choose a threshold and evaluation period suitable for your workload.
5. Attach an action group with an approved notification method.
6. Create the rule and test notification delivery.

Thresholds should reflect normal workload behavior. Tune them to avoid alert fatigue.

## 8. Production best practices

- Establish baselines for CPU, memory, disk, network, and application response time.
- Collect only the logs you need; ingestion volume affects cost.
- Define alert ownership and notification routes.
- Verify data ingestion and alert delivery.
- Use least-privilege access to workspaces and monitoring configuration.
- Correlate metrics, logs, deployments, and application changes during incidents.
- Document root causes and remediation steps.

## 9. Screenshot checklist

1. Azure VM CPU metrics.
2. VM instance view showing its state.
3. Linux terminal output from `free -h` and `df -h`.
4. Log Analytics editor showing a successful Heartbeat or Syslog query.
5. An alert rule configuration, with sensitive details hidden.

## 10. LinkedIn post — ready to publish

🚀 **Day 55/100 — Azure Monitor, Log Analytics, and VM Troubleshooting**

What would you do if your Azure VM was running, but your application was down?

Instead of guessing, today I explored how to investigate issues using metrics, logs, and Linux troubleshooting commands.

🔹 **What I learned:**
- Azure Monitor helps collect and analyze telemetry.
- Metrics reveal patterns such as high CPU usage.
- Log Analytics lets us query collected logs using Kusto Query Language (KQL).
- Azure Monitor Agent and Data Collection Rules configure guest data collection.
- Alerts can notify the team when defined conditions are met.

🛠️ **Hands-on practice:**
1. Checked Azure VM state using Azure CLI.
2. Inspected CPU metrics.
3. Investigated Linux memory, disk, and service health.
4. Practiced KQL queries for heartbeat and Syslog data.
5. Reviewed a systematic workflow for troubleshooting a website outage.

💡 **Key takeaway:** A running VM does not guarantee a healthy application. Combine metrics, logs, service checks, and network troubleshooting to find the root cause.

Monitoring is not just about collecting data—it's about turning signals into useful action.

Continuing my **100 Days of DevOps — From Zero to Hero** journey, one hands-on lesson at a time. 🚀

#100DaysOfDevOps #Azure #AzureMonitor #LogAnalytics #DevOps #CloudComputing #KQL #Linux #Observability #LearningInPublic

## 11. Next lesson

**Day 56/100 — Azure VM Backup and Disaster Recovery**

We'll explore recovery points, backup policies, restore options, and the importance of testing recovery procedures.

## Official Microsoft documentation

- [Azure Monitor overview](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)
- [Azure Monitor Agent overview](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/agents-overview)
- [Log Analytics tutorial](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-tutorial)
- [KQL overview](https://learn.microsoft.com/en-us/kusto/query/)
- [Create metric alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-create-metric-alert-rule)
