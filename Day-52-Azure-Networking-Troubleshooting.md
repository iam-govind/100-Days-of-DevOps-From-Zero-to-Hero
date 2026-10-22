# 100 Days of DevOps — Day 52/100
## Azure Networking Troubleshooting: NSGs, Routes, and Connectivity

**Goal:** Troubleshoot common Azure VM connectivity problems using Azure CLI, the Azure portal, and commands inside Linux VMs.  
**Environment:** Azure subscription, Azure CLI, macOS Terminal, and ideally two VMs in the same virtual network.

> **Safety note:** Read-only inspection commands are generally safe. Avoid changing NSG rules or route tables in production without approval. Never expose SSH or administrative ports to the entire internet unless a short-lived, approved lab specifically requires it.

---

## 1. Understand the network path

Example architecture:

- **VM1 — Web VM:** public IP for browser/SSH access, plus a private IP.
- **VM2 — Private VM:** private IP only.
- Both VMs are connected to a virtual network (VNet), potentially in separate subnets.

For a connection to succeed, check the path in order:

1. Correct destination IP and port.
2. A route exists in the required direction.
3. Network Security Group (NSG) rules permit the traffic.
4. The guest operating system's firewall permits the traffic.
5. The application is running and listening on the expected interface and port.

### Key components

| Component | What it does | Typical failure |
|---|---|---|
| Network Security Group (NSG) | Allows or denies traffic using priority-ordered rules | TCP/22 or TCP/80 is denied |
| Route table / effective routes | Chooses the next hop for traffic | Unexpected route or missing path |
| Guest OS firewall | Filters traffic inside Linux/Windows | `ufw` or firewalld blocks the port |
| Application/service | Listens for requests | Nginx/Apache is stopped or bound to localhost |
| Public IP / DNS | Provides an internet-facing address or name | Wrong IP or DNS record |

Azure evaluates NSG rules by priority; lower numbers are evaluated first. A matching rule can determine the result before a later rule is reached. NSGs are stateful, so return traffic for an allowed connection is normally tracked automatically. Verify the effective configuration on the actual NIC and subnet.

---

## 2. Lab preparation — Sign in and select the subscription

On your Mac, open Terminal:

```bash
az login
az account show --output table
az account list --output table
```

If you have multiple subscriptions, select the correct one:

```bash
az account set --subscription "<SUBSCRIPTION_ID_OR_NAME>"
az account show --output table
```

Set variables to match your resources. Replace every placeholder before running:

```bash
RG="<VM_RESOURCE_GROUP>"
VM1="<WEB_VM_NAME>"
VM2="<PRIVATE_VM_NAME>"
```

Check the VM details:

```bash
az vm show --resource-group "$RG" --name "$VM1" \
  --show-details \
  --query "{name:name,powerState:powerState,publicIps:publicIps,privateIps:privateIps}" \
  --output table

az vm show --resource-group "$RG" --name "$VM2" \
  --show-details \
  --query "{name:name,powerState:powerState,publicIps:publicIps,privateIps:privateIps}" \
  --output table
```

**Expected:** VM1 should show a public IP if configured. VM2 should have a private IP and no public IP if designed as a private-only VM.

If the VMs are in different resource groups, use a separate resource-group variable for each VM.

---

## 3. Lab A — Confirm the VNet, subnet, NIC, and NSGs

### Step 1: Identify the VM's network interface

```bash
az vm show -g "$RG" -n "$VM1" \
  --query "networkProfile.networkInterfaces[].id" -o tsv
```

Copy the returned NIC resource ID. Then inspect it:

```bash
NIC_ID=$(az vm show -g "$RG" -n "$VM1" \
  --query "networkProfile.networkInterfaces[0].id" -o tsv)

az network nic show --ids "$NIC_ID" \
  --query "{nic:name,privateIP:ipConfigurations[0].privateIPAddress,subnet:ipConfigurations[0].subnet.id,nsg:networkSecurityGroup.id}" \
  --output json
```

Repeat for VM2 by changing `"$VM1"` to `"$VM2"`.

### Step 2: Inspect effective NSG rules

```bash
az network nic list-effective-nsg --ids "$NIC_ID" --output json
```

This shows effective NSG rules where supported. If it errors, check the Azure CLI version and permissions, and inspect NSGs attached to both the NIC and subnet in the portal.

### Step 3: Inspect the subnet and route table

In the Azure portal, open **Virtual networks → your VNet → Subnets**. Select the relevant subnet and inspect:

- Network security group
- Route table
- Address range

Verify that both VMs are in the intended VNet and that peering, VPN, or other routing is configured if they are not in the same VNet.

---

## 4. Lab B — Test connectivity from the client

### Step 1: Get the VM IPs

```bash
az vm show -g "$RG" -n "$VM1" --show-details \
  --query "{public:publicIps,private:privateIps}" -o json

az vm show -g "$RG" -n "$VM2" --show-details \
  --query "{public:publicIps,private:privateIps}" -o json
```

### Step 2: Test the web port from your Mac

For a web VM with a public IP:

```bash
curl -I --connect-timeout 5 "http://<WEB_VM_PUBLIC_IP>"
```

If HTTPS is configured:

```bash
curl -Iv --connect-timeout 5 "https://<WEB_VM_PUBLIC_IP>"
```

**Interpret the result:**

- `HTTP/1.1 200`, `301`, or `302`: an HTTP service responded.
- `Connection refused`: the host was reached, but nothing accepted the connection on that port, or a firewall actively rejected it.
- Timeout: traffic may be dropped by an NSG, route, firewall, or upstream network; the service may also be unreachable.
- DNS error: check the hostname and DNS record before troubleshooting TCP.

Test SSH only if SSH is enabled and your source IP is permitted:

```bash
ssh -i "<PATH_TO_PRIVATE_KEY.pem>" <ADMIN_USERNAME>@<WEB_VM_PUBLIC_IP>
```

Never share the private key or paste it into a public repository.

### Step 3: Test a private VM from a machine with private network access

A private IP such as `10.x.x.x` is not ordinarily reachable directly from a Mac over the public internet. Use an approved path, such as:

- SSH from VM1 (a jump host), if policy allows it;
- Azure Bastion;
- a VPN/ExpressRoute-connected network.

From VM1, test VM2's private IP and intended port:

```bash
ping -c 4 <VM2_PRIVATE_IP>
nc -vz -w 5 <VM2_PRIVATE_IP> 22
```

ICMP ping may be blocked even when TCP works, so a failed ping alone does not prove that the VM is down. `nc` may not be installed; use an approved equivalent if needed.

---

## 5. Lab C — Check the Linux guest and application

Connect to the VM through your approved access method, then run:

### Step 1: Verify interface and routes

```bash
ip addr
ip route
```

Check that the VM has the expected private IP and a route for the destination network.

### Step 2: Check listening ports

```bash
sudo ss -lntp
```

For a web server, look for port `80` or `443`. For SSH, look for port `22`.

A service bound only to `127.0.0.1` may be accessible from the VM itself but not from another machine.

### Step 3: Check service status

For Nginx:

```bash
sudo systemctl status nginx --no-pager
curl -I http://localhost
```

For Apache on Ubuntu/Debian:

```bash
sudo systemctl status apache2 --no-pager
curl -I http://localhost
```

On some distributions, Apache's service name is `httpd`.

### Step 4: Check the Linux firewall

If `ufw` is installed:

```bash
sudo ufw status verbose
```

If the service is expected to accept web traffic, confirm the OS firewall permits the intended port. Change firewall rules only when you understand the security impact and have authorization.

---

## 6. Lab D — Inspect effective routes in Azure

Effective routes show the routes applied to a network interface, including system routes, custom routes, and routes learned through supported network connections.

In the Azure portal:

1. Open **Virtual machines** and select the VM.
2. Open **Networking** and select the network interface.
3. Look for **Effective routes** (portal wording may vary).
4. Inspect the destination prefix, next hop type, and next hop IP where shown.

You can inspect route tables using Azure CLI:

```bash
az network route-table list --resource-group "$RG" --output table
```

Then inspect a known route table:

```bash
az network route-table show -g "$RG" -n "<ROUTE_TABLE_NAME>" --output json

az network route-table route list -g "$RG" \
  --route-table-name "<ROUTE_TABLE_NAME>" --output table
```

Do not create or modify routes just to experiment in a production VNet. A bad route can interrupt SSH, application traffic, or access to other network services.

---

## 7. NSG troubleshooting workflow

When a port is blocked:

1. Identify the source IP, destination IP, protocol, and destination port.
2. Check both the subnet NSG and NIC NSG, if present.
3. Review inbound rules on the destination side and outbound rules on the source side.
4. Confirm rule priority, source/destination address ranges, protocol, and port ranges.
5. Inspect effective security rules and the actual connection test result.
6. Check the guest OS firewall and service listener.
7. Re-test from the same source after an approved change.

**Security best practice:** Restrict SSH (TCP/22) to your known public IP or use Azure Bastion. Avoid broad `0.0.0.0/0` SSH rules. For a web server, expose only the ports required by the application. Use least privilege and document any temporary rule with an owner and expiry.

---

## 8. Common errors and fixes

| Symptom | Likely causes | Checks |
|---|---|---|
| SSH times out | NSG drop, route issue, wrong IP, source network restrictions | Verify public IP, inbound TCP/22, effective routes, Bastion/VPN path |
| SSH says `Connection refused` | SSH service stopped, wrong port, OS firewall rejection | Check `systemctl status ssh`/`sshd`, listening port, firewall |
| Browser cannot open website | Web service stopped, wrong port, NSG rule, OS firewall | `curl -I localhost`, `ss -lntp`, inbound TCP/80 or 443 |
| VM1 cannot reach VM2 | Wrong private IP, different VNet without route, NSG rule, OS firewall | Verify VNet/subnet, routes, NSGs, and test TCP from VM1 |
| Ping fails but SSH works | ICMP blocked | Use the actual application port test |
| `az` command fails | Not logged in, wrong subscription, insufficient permissions | `az login`, `az account show`, verify RBAC |
| Effective route/NSG command fails | Wrong NIC ID, unsupported CLI version, permissions | Check NIC ID, `az version`, and portal view |

---

## 9. Optional: Azure Network Watcher connection troubleshooting

Azure Network Watcher provides tools such as **Connection troubleshoot** and **IP flow verify**, subject to region, resource, and permission support.

In the portal, open **Network Watcher → Connection troubleshoot** (or the current equivalent), select the source and destination, and test the destination port. For an NSG flow issue, use **IP flow verify** when available to determine whether traffic is allowed or denied and which rule is responsible.

Use these tools to gather evidence before changing rules. Exact options depend on the region and resource configuration.

---

## 10. Day 52 practical checklist

- [ ] Confirmed the active Azure subscription.
- [ ] Checked public and private IPs for both VMs.
- [ ] Identified each VM's NIC and subnet.
- [ ] Inspected NSGs and effective security rules.
- [ ] Checked the effective route path.
- [ ] Tested the actual TCP port rather than relying only on ping.
- [ ] Checked the guest firewall and service listener.
- [ ] Made no broad or unapproved production security changes.

## 11. What to capture for LinkedIn

Capture genuine output from your own lab:

1. Azure VM networking page showing public/private IP configuration.
2. NSG inbound rules showing the intended, restricted ports.
3. Effective routes or connection-troubleshoot results.
4. Terminal output from `curl -I`, `nc -vz`, or `ss -lntp` showing the test and its result.

Never show private keys, passwords, tokens, or sensitive internal network details from production. If you did not perform a test, don't present a screenshot as proof that it succeeded.

---

## 12. LinkedIn post — Day 52/100

**100 Days of DevOps — Day 52/100 🚀**

Today I practised **Azure Networking Troubleshooting: NSGs, Routes, and Connectivity**.

A VM can be running and an application can be installed—yet users may still be unable to connect. Troubleshooting means checking the entire path, not just restarting the server.

In this lab, I focused on:
- Checking Azure VM public and private IPs.
- Identifying network interfaces, subnets, and NSGs.
- Understanding how NSG rules and route tables affect traffic.
- Testing real ports with `curl` and `nc`.
- Checking Linux routes, firewall rules, and listening services.
- Using effective routes and Network Watcher tools to narrow down failures.

**Key takeaway:** A failed ping doesn't always mean a VM is down. Test the actual application port and verify each layer of the network path.

In production, I would gather evidence first, make the smallest approved change, and re-test from the same source.

**Question:** When a VM is unreachable, what do you check first—the NSG, the route table, or the guest OS firewall?

Learning one practical DevOps concept at a time. 🚀

#100DaysOfDevOps #Azure #AzureNetworking #CloudComputing #DevOps #NetworkSecurity #Linux #LearningInPublic

---

## 13. Tomorrow — Day 53/100

**Next topic: Azure Load Balancer and Application Gateway — When to Use Each**

We'll compare Layer 4 and Layer 7 load balancing, health probes, backend pools, routing, and common troubleshooting scenarios.

## Official references

- Azure Network Security Groups: https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview
- Effective routes: https://learn.microsoft.com/en-us/azure/virtual-network/diagnose-network-routing-problem
- Azure Network Watcher: https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-overview
- Azure CLI VM commands: https://learn.microsoft.com/en-us/cli/azure/vm
