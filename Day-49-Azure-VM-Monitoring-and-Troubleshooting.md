# Day 49/100 — Azure VM Monitoring and Troubleshooting

## 🎯 Learning Objectives

By the end of this lesson, you will be able to:

- Explain why Azure VM monitoring matters.
- Check VM power and provisioning status using Azure CLI.
- View Azure Monitor metrics and Activity Logs.
- Understand Boot Diagnostics and basic VM health checks.
- Troubleshoot common SSH and web-server connectivity problems.
- Capture useful evidence for a troubleshooting report and LinkedIn post.

---

## 1. What Is Azure VM Monitoring?

Monitoring is the process of collecting and reviewing information about a system's health, performance, and activity.

For an Azure Virtual Machine, monitoring can help answer questions such as:

- Is the VM running?
- Is CPU usage unusually high?
- Is memory or disk capacity becoming a problem?
- When did a VM restart or stop?
- Did a configuration change happen before an incident?
- Why can users not reach the application?

### Key Azure monitoring tools

| Tool | What it helps you investigate |
|---|---|
| Azure Monitor Metrics | CPU usage, network activity, and other supported metrics |
| Activity Log | Subscription-level and resource-management events, including operations on a VM |
| Boot Diagnostics | Startup screenshots and serial-console information for supported scenarios |
| VM Insights | Enhanced performance and dependency insights when configured |
| Network Watcher | Network connectivity and troubleshooting tools |
| Azure CLI | Command-line checks and repeatable investigation steps |
| Nginx/system logs | Application and operating-system service issues inside the VM |

**Important:** Azure platform metrics are not the same as guest operating-system metrics. For example, memory usage inside Linux may require additional monitoring configuration or an agent; do not assume every metric is available automatically.

---

## 2. Understand the Monitoring Flow

```text
User reports a problem
        |
        v
Check VM state and Azure Activity Log
        |
        v
Review Azure Monitor metrics
        |
        v
Check boot diagnostics and network rules
        |
        v
Connect to VM (if possible)
        |
        v
Inspect OS and application services
        |
        v
Identify root cause, fix, and verify
```

A good troubleshooting approach checks one layer at a time rather than changing many settings at once.

---

## 3. Prerequisites

- An Azure subscription and Azure CLI.
- Permission to read the VM and its monitoring data.
- An existing test VM. You can use the VM created in Day 48 if it still exists.
- If investigating the web-server lab, Nginx should be installed on the VM.

The examples below use:

```bash
RESOURCE_GROUP="rg-day48-app-deployment"
VM_NAME="webvm-day48"
```

Replace these values with your actual resource group and VM name. If you deleted the Day 48 lab, use an existing VM that you are authorized to inspect. These monitoring commands are primarily read-only.

---

## 4. Step 1 — Sign In and Select the Correct Subscription

```bash
az login
az account show --output table
```

If required, select your subscription:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

Confirm the active subscription:

```bash
az account show --query "{Name:name,Id:id}" --output table
```

**Expected result:** Azure CLI displays the subscription you intend to investigate.

---

## 5. Step 2 — Check VM Status

Check the VM's power state and provisioning details:

```bash
az vm get-instance-view \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --output table
```

You can also retrieve the main status fields:

```bash
az vm get-instance-view \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query "{VM:name,Statuses:instanceView.statuses[].displayStatus}" \
  --output json
```

**What to look for:**

- `PowerState/running` — the VM is powered on.
- `PowerState/stopped` — the VM is stopped but may still be allocated.
- `PowerState/deallocated` — compute has been deallocated.
- Provisioning errors or unexpected status messages.

A VM showing as running does **not** guarantee that SSH, Nginx, or the application is healthy.

If the command says the resource was not found, verify the resource group, VM name, and active subscription.

---

## 6. Step 3 — Review Azure Monitor Metrics

Metrics help you understand performance over time.

### In the Azure portal

1. Open **Virtual machines** and select your VM.
2. Under **Monitoring**, select **Metrics**.
3. Choose a time range, such as the last 30 minutes or 24 hours.
4. Select a metric such as **Percentage CPU**.
5. Adjust the aggregation and time granularity as needed.
6. Add another chart for a supported network metric if useful.

### What should you investigate?

- **High CPU:** Could indicate a busy workload, an inefficient process, or a traffic spike.
- **Unexpected network activity:** Could be normal traffic or a sign of unusual workload behavior.
- **Metric spikes:** Compare their timestamps with application issues and Activity Log events.
- **No data:** Confirm the time range, selected resource, metric availability, and permissions.

### Optional: Query CPU metrics using Azure CLI

Get the VM resource ID:

```bash
VM_ID=$(az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query id \
  --output tsv)

echo "$VM_ID"
```

List available metric definitions:

```bash
az monitor metrics list-definitions \
  --resource "$VM_ID" \
  --output table
```

Query CPU percentage over the previous hour on macOS:

```bash
START_TIME=$(date -u -v-1H '+%Y-%m-%dT%H:%M:%SZ')
END_TIME=$(date -u '+%Y-%m-%dT%H:%M:%SZ')

az monitor metrics list \
  --resource "$VM_ID" \
  --metric "Percentage CPU" \
  --interval PT5M \
  --aggregation Average \
  --start-time "$START_TIME" \
  --end-time "$END_TIME" \
  --output table
```

**Linux note:** If you run this on Linux, use this start-time command instead:

```bash
START_TIME=$(date -u -d '1 hour ago' '+%Y-%m-%dT%H:%M:%SZ')
```

Metric names and availability can vary by resource type. If the CLI reports that a metric is unavailable, inspect the metric definitions and use a supported metric. If there is no data, try a longer time range.

**Expected result:** If data is available, Azure returns timestamped metric values. A newly created or idle VM may show low CPU usage.

---

## 7. Step 4 — Inspect the Azure Activity Log

The Activity Log records Azure control-plane events, such as VM start, stop, restart, configuration changes, and failed management operations.

### In the Azure portal

1. Open the VM or its resource group.
2. Select **Activity log**.
3. Set a suitable time range.
4. Filter by status, operation, or event severity.
5. Open an event to inspect its details and correlation information.

### Using Azure CLI

List recent events for the VM:

```bash
az monitor activity-log list \
  --resource-id "$VM_ID" \
  --offset 24h \
  --max-events 50 \
  --output table
```

To inspect failed events in the subscription over the last day:

```bash
az monitor activity-log list \
  --status Failed \
  --offset 24h \
  --max-events 50 \
  --output table
```

The second command is subscription-level and may return failures unrelated to this VM.

**What to investigate:**

- Was the VM restarted or stopped around the time the issue began?
- Did an operation fail?
- Who or what initiated the operation, if that information is available?
- Is there an error message or correlation ID that can help further investigation?

**Important:** Activity Log records Azure management operations. It does not automatically contain every command executed inside Linux or every application error.

---

## 8. Step 5 — Check Boot Diagnostics

Boot Diagnostics can help investigate a VM that fails to start correctly or becomes stuck during boot.

### In the Azure portal

1. Open the VM.
2. Under **Help**, find **Boot diagnostics** (the menu name may vary).
3. Review the boot screenshot and serial log if available.
4. Look for boot errors, kernel messages, or a stalled startup sequence.

Boot Diagnostics availability and details depend on the VM configuration. Serial-console access may require additional permissions and configuration.

If the VM is running but SSH fails, Boot Diagnostics is only one clue. You should also check network rules, SSH configuration, and authentication.

---

## 9. Step 6 — Troubleshoot SSH Connectivity

If SSH to the VM fails, work through these checks in order.

### A. Confirm the VM is running

```bash
az vm get-instance-view \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --output table
```

### B. Get the public IP

```bash
az vm show \
  --show-details \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query publicIps \
  --output tsv
```

### C. Check the NSG

In the Azure portal, open the VM's **Networking** page and inspect inbound security rules. Confirm TCP port 22 is allowed from your current trusted public IP (`/32`) or through your approved access method.

Do not open SSH to the entire internet as a troubleshooting shortcut.

### D. Test the SSH connection

From your local Mac:

```bash
ssh -v azureuser@<PUBLIC_IP>
```

Replace the username and public IP with your actual values. The `-v` option provides diagnostic details. Do not share logs publicly without checking them for sensitive information.

Common causes include an incorrect username or key, a changed public IP, a restrictive NSG, a stopped VM, or SSH service issues inside the guest OS.

---

## 10. Step 7 — Troubleshoot Nginx and the Web Application

If the VM is running but the website does not load, connect over SSH if possible and run these commands **inside the VM**.

Check Nginx:

```bash
sudo systemctl status nginx --no-pager
```

Check whether a process is listening on port 80:

```bash
sudo ss -lntp | grep ':80'
```

Test the local web server:

```bash
curl -I http://localhost
```

Review recent service logs:

```bash
sudo journalctl -u nginx --no-pager -n 50
```

Validate Nginx configuration:

```bash
sudo nginx -t
```

If the service is stopped and you are authorized to start it:

```bash
sudo systemctl start nginx
```

Then test again:

```bash
curl -I http://localhost
```

**How to interpret the results:**

- If localhost works but the public website fails, investigate the NSG, public IP, and guest firewall.
- If localhost fails, focus first on Nginx, its configuration, and the application files.
- If Nginx is running but the page is wrong, inspect the web files and Nginx configuration.

---

## 11. Troubleshooting Reference

| Symptom | Possible cause | First checks |
|---|---|---|
| VM is stopped | Manual stop or automation | VM instance view and Activity Log |
| SSH times out | Network rule, route, or IP issue | VM state, public IP, NSG, approved access path |
| SSH says `Permission denied` | Wrong username or key | SSH command, key path, VM access configuration |
| Website times out | HTTP blocked or server unreachable | NSG port 80, guest firewall, public IP |
| Connection refused | No service listening on the port | Nginx status and listening ports |
| High CPU | Workload spike or inefficient process | CPU metrics and guest processes |
| No metric data | Wrong time range or unsupported metric | Metric definitions, resource, permissions |
| VM fails during startup | Boot or OS issue | Boot Diagnostics and serial logs |

A symptom is not proof of a root cause. Gather evidence before changing configuration.

---

## 12. Production Best Practices

- Define alerts for important metrics and availability signals.
- Use Azure Monitor alerts and action groups appropriate to the team's incident process.
- Configure guest-level monitoring when memory, filesystem, or process metrics are required.
- Send relevant logs to a centralized destination when required by your operational and retention policies.
- Review Activity Log events during incident investigations.
- Restrict SSH and other management ports to trusted access paths.
- Document the incident timeline, evidence, root cause, fix, and verification.
- Test recovery procedures and keep appropriate backups.
- Avoid restarting or resizing a production VM before collecting useful evidence and understanding the impact.

---

## 13. Hands-On Challenge

Using a test VM you are authorized to inspect:

1. Check its power and provisioning status.
2. Find its resource ID.
3. Review CPU metrics for the last hour.
4. Inspect Activity Log events from the last 24 hours.
5. Open Boot Diagnostics and identify what information it provides.
6. If the VM hosts Nginx, check the service status and test `http://localhost`.
7. Write down one symptom, the evidence collected, the likely cause, and the verification step.

Do not deliberately stop services or change production infrastructure to create a failure. Use a disposable lab for experiments.

### Reflection Questions

1. What is the difference between Azure Monitor Metrics and Activity Log?
2. Why can a VM be running while the application is unavailable?
3. If `curl http://localhost` succeeds but the public website fails, which layers would you investigate?
4. Why might memory usage not appear in the default platform metrics?
5. What evidence should you collect before restarting a VM?

---

## 14. Screenshots to Capture for LinkedIn

Capture a few useful, non-sensitive screenshots:

- Azure portal showing the VM's running status.
- Azure Monitor CPU metric chart with the selected time range visible.
- Activity Log showing recent VM events.
- Boot Diagnostics page (if available).
- Terminal output showing `systemctl status nginx` and `curl -I http://localhost`.

Hide subscription IDs, tenant IDs, email addresses, public IPs when sensitive, and any credentials or private keys.

---

## 15. LinkedIn Post — Day 49/100

**100 Days of DevOps — Day 49/100 🚀**

📊 **Azure VM Monitoring and Troubleshooting**

A virtual machine showing **Running** does not necessarily mean the application hosted on it is healthy.

Today, I explored how to investigate Azure VM issues using monitoring data and practical troubleshooting commands.

🛠️ **What I practiced:**

✅ Checked VM power and provisioning status using Azure CLI.  
✅ Explored Azure Monitor metrics such as CPU usage.  
✅ Reviewed Activity Log events to understand resource operations.  
✅ Learned how Boot Diagnostics can help investigate startup issues.  
✅ Checked Nginx status, listening ports, and service logs.  
✅ Reviewed a step-by-step approach to SSH and website connectivity problems.

💡 **My biggest takeaway:**

Good troubleshooting starts with evidence—not guesswork.

Check the VM, metrics, activity history, network rules, operating system, and application layer one step at a time. This makes it easier to narrow down the root cause without making unnecessary changes.

**Question for the DevOps community:** When an application becomes unreachable on a cloud VM, what do you check first—VM health, network rules, or application logs?

Continuing my 100 Days of DevOps journey, one practical lesson at a time!

#100DaysOfDevOps #Azure #AzureMonitor #CloudComputing #Linux #Troubleshooting #DevOps #LearningInPublic

---

## 16. Upload This Lesson to GitHub

Save this file as:

`Day-49-Azure-VM-Monitoring-and-Troubleshooting.md`

From the root of your local repository, run:

```bash
git add Day-49-Azure-VM-Monitoring-and-Troubleshooting.md
git commit -m "Add Day 49 Azure VM monitoring and troubleshooting"
git push origin main
```

Confirm that the file appears in your GitHub repository.

---

## Coming Next: Day 50/100

**Azure VM Backup and Recovery Basics**

Next, we will explore Azure Backup concepts, recovery points, backup configuration, restore options, and practical steps for protecting important VM workloads.

Keep learning, keep building, and keep documenting your DevOps journey! 🚀
