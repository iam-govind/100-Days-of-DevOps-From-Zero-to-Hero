# Day 45/100 — Public IP, Private IP & Network Security Groups (NSGs)

## Goal
Continue from Day 44 (Azure Virtual Networks). Learn how public and private IPs work, how NSGs filter traffic, and how to create safer inbound rules with Azure CLI.

## 1. Public IP vs. private IP

- **Public IP:** Can enable internet communication when routing and security rules permit it.
- **Private IP:** Used for communication inside private network environments, such as an Azure VNet.
- A public IP does not mean all ports are open. NSGs, routes, host firewalls, and the service listening on the destination port also affect connectivity.

```text
Internet
   |
Public IP
   |
 Web VM
Private IP
   |
Azure Virtual Network
   |
Database VM (private IP)
```

A VM can have a private IP without a public IP. Private-only VMs may be accessed through approved private connectivity, such as a VPN or Azure Bastion.

## 2. What is an NSG?

A **Network Security Group (NSG)** contains rules that allow or deny network traffic. It can be associated with a subnet or a network interface (NIC).

```text
Internet -> Public IP -> NSG Rules -> NIC -> Azure VM
```

Rules evaluate source and destination addresses, ports, protocol, direction, action, and priority.

## 3. Inbound vs. outbound

- **Inbound:** Traffic coming into a resource or subnet.
- **Outbound:** Traffic leaving a resource or subnet.

| Service | Protocol | Port | Typical use |
|---|---|---:|---|
| SSH | TCP | 22 | Remote Linux administration |
| HTTP | TCP | 80 | Web traffic |
| HTTPS | TCP | 443 | Encrypted web traffic |
| RDP | TCP | 3389 | Remote Windows administration |

## 4. NSG priorities and safety

NSG priorities range from 100 to 4096. A **lower number has higher priority**. Azure also creates default rules, including a default deny for other inbound traffic after the default allows.

For SSH, prefer allowing your own public IPv4 address using `/32` rather than opening port 22 to everyone.

```text
SSH (22)
Source: Your IPv4 address/32
Action: Allow
```

When subnet-level and NIC-level NSGs both apply, traffic must be allowed by both applicable security layers.

> **Safety note:** This lab associates the new NSG with the separate Day 44 `web-subnet`, not your existing VM's subnet. This avoids accidentally changing access to your Day 43 VM.

## 5. Hands-on lab — Create an NSG

### Step 1 — Sign in

```bash
az login
az account show --output table
```

If needed:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

### Step 2 — Set variables

```bash
RESOURCE_GROUP="rg-100days-devops"
LOCATION="eastus"
VNET_NAME="vnet-day44-devops"
WEB_SUBNET="web-subnet"
NSG_NAME="nsg-day45-web"
```

Verify the resource group, VNet and subnet:

```bash
az group show --name "$RESOURCE_GROUP" --output table

az network vnet show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VNET_NAME" \
  --output table

az network vnet subnet show \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$WEB_SUBNET" \
  --output table
```

If the VNet or subnet is missing, complete the Day 44 lab first.

### Step 3 — Find your current public IP

On your Mac/Linux terminal:

```bash
MY_PUBLIC_IP=$(curl -s https://api.ipify.org)
echo "$MY_PUBLIC_IP"
```

If this is blank or unexpected, obtain your public IPv4 address through a trusted method and set it manually:

```bash
MY_PUBLIC_IP="YOUR_PUBLIC_IPV4_ADDRESS"
```

Use a single IPv4 address without `/32` in the variable. The rule below appends `/32`. Your public IP may change when you change networks.

### Step 4 — Create the NSG

```bash
az network nsg create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$NSG_NAME" \
  --location "$LOCATION"
```

Verify:

```bash
az network nsg show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$NSG_NAME" \
  --output table
```

### Step 5 — Allow SSH only from your IP

Check the variable before proceeding:

```bash
echo "$MY_PUBLIC_IP"
```

Then create the rule:

```bash
az network nsg rule create \
  --resource-group "$RESOURCE_GROUP" \
  --nsg-name "$NSG_NAME" \
  --name "Allow-SSH-From-My-IP" \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "$MY_PUBLIC_IP/32" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 22
```

This allows SSH from that one source IPv4 address to resources protected by this NSG, subject to other applicable network controls.

### Step 6 — Allow HTTP

```bash
az network nsg rule create \
  --resource-group "$RESOURCE_GROUP" \
  --nsg-name "$NSG_NAME" \
  --name "Allow-HTTP-From-Internet" \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes Internet \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 80
```

This permits HTTP traffic from the Azure `Internet` service tag. It does not install a web server, create a public IP, or guarantee a website is reachable.

### Step 7 — List rules

```bash
az network nsg rule list \
  --resource-group "$RESOURCE_GROUP" \
  --nsg-name "$NSG_NAME" \
  --output table
```

You should see the custom SSH and HTTP rules along with the NSG's default rules.

### Step 8 — Associate the NSG with the Day 44 web subnet

```bash
az network vnet subnet update \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$WEB_SUBNET" \
  --network-security-group "$NSG_NAME"
```

Verify:

```bash
az network vnet subnet show \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$WEB_SUBNET" \
  --query "{subnet:name,networkSecurityGroup:networkSecurityGroup.id}" \
  --output json
```

This is a separate lab subnet, so it should not change your Day 43 VM's network configuration.

## 6. Inspect the Day 43 VM's IPs

If `vm-100days-devops` still exists, get its public IP:

```bash
az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "vm-100days-devops" \
  --show-details \
  --query publicIps \
  --output tsv
```

Get its NIC ID and private IP:

```bash
NIC_ID=$(az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "vm-100days-devops" \
  --query "networkProfile.networkInterfaces[0].id" \
  --output tsv)

az network nic show \
  --ids "$NIC_ID" \
  --query "ipConfigurations[0].privateIPAddress" \
  --output tsv
```

If the VM was deleted, these inspection commands will fail because the resource no longer exists; the NSG lab can still be completed.

## 7. Expected results

| Rule | Direction | Source | Protocol/port | Action |
|---|---|---|---|---|
| `Allow-SSH-From-My-IP` | Inbound | Your IPv4 `/32` | TCP 22 | Allow |
| `Allow-HTTP-From-Internet` | Inbound | Internet service tag | TCP 80 | Allow |

```text
          Your IP/32              Internet
              |                       |
            TCP 22                  TCP 80
              |                       |
              +-----------+-----------+
                          |
                          v
                      NSG Rules
                          |
                          v
                    Day 44 web-subnet
                     10.10.1.0/24
                          |
                          v
                     Future Web VM
```

The diagram shows the intended rules. Since the lab subnet has no VM, it does not demonstrate a live SSH or HTTP connection.

## 8. Troubleshooting

### Resource not found

```bash
az group list --output table
az network vnet list --output table
```

Check the subscription and resource names.

### NSG already exists

```bash
az network nsg list --output table
```

Inspect the existing NSG before modifying or deleting anything.

### Rule creation fails

```bash
az network nsg rule list \
  --resource-group "$RESOURCE_GROUP" \
  --nsg-name "$NSG_NAME" \
  --output table
```

Check for duplicate names, invalid IP prefixes, and conflicting priorities.

### SSH still doesn't work
Check that the VM is running, the IP is correct, the rule is associated with the relevant subnet/NIC, the source IP matches, no other applicable NSG blocks traffic, the host firewall permits SSH, and the SSH service is running.

### HTTP doesn't work
Check that a web server is installed and running, the VM is reachable, port 80 is allowed by all applicable security layers, and the server is listening on the expected interface.

### Public IP changed
Check the current IP and update the SSH source rule after verifying it. Don't widen the rule to `Any` just to make it work.

## 9. Day 45 hands-on challenge

1. Explain public IP vs. private IP.
2. Create `nsg-day45-web`.
3. Allow SSH from your own IPv4 `/32`.
4. Allow HTTP from the Internet service tag.
5. List the rules and explain their priorities.
6. Associate the NSG with Day 44's `web-subnet`.
7. Inspect the Day 43 VM's IPs if it still exists.
8. Explain why an NSG allow rule alone does not guarantee connectivity.

**Challenge question:** If a VM has a public IP but the applicable NSG denies inbound TCP port 22, will a direct SSH connection from the internet succeed?

**Expected answer:** No. The NSG blocks that traffic even though a public IP exists.

## 10. Screenshots to capture for LinkedIn

Capture real screenshots of:
1. The `nsg-day45-web` overview.
2. Inbound rules showing SSH restricted to your IP and HTTP on port 80.
3. The NSG association with `web-subnet`.
4. Public/private IP command output, if the Day 43 VM still exists.

Suggested diagram:

```text
Internet
   |
   v
Public IP
   |
   v
NSG: Allow required traffic only
   |
   v
Azure VM
```

Never expose private keys, credentials or other sensitive details in screenshots.

## 11. Key takeaways

- Public and private IPs serve different purposes.
- A public IP does not automatically open ports.
- NSGs filter traffic using rules.
- Inbound and outbound rules control different traffic directions.
- Lower priority numbers are evaluated first.
- Restrict SSH to trusted source addresses where practical.
- NSGs can be associated with subnets or NICs.
- Routing, NSGs, host firewalls, and applications all affect connectivity.

**DevOps lesson:** Secure infrastructure isn't just about making connections work. It's about allowing the right traffic from the right sources while keeping unnecessary access closed.

## 12. GitHub update

Save this file as `Day-45-Public-Private-IP-and-NSG.md` in your repository.

```bash
git status
git add Day-45-Public-Private-IP-and-NSG.md
git commit -m "Add Day 45 Public Private IP and NSG"
git push origin main
```

Verify:

```bash
git log --oneline -1
```

## 13. LinkedIn post — Day 45/100

### Day 45/100 — Public IP, Private IP & Network Security Groups in Azure 🔐

A VM can be running perfectly and still be unreachable.

Today I explored one of the reasons: **network access and security rules**.

As part of my 100 Days of DevOps journey, I practised working with Azure IP addresses and Network Security Groups (NSGs).

What I learned today:

✅ The difference between public and private IP addresses

✅ How VMs use private IPs inside a virtual network

✅ How NSGs control inbound and outbound traffic

✅ Why rule priorities matter

✅ How to create an NSG using Azure CLI

✅ How to restrict SSH access to a specific source IP

✅ How to allow HTTP traffic using an inbound rule

Here's the basic idea:

```text
Internet
   |
   v
Public IP
   |
   v
NSG Rules
   |
   v
Azure VM
```

One lesson stood out: **having a public IP doesn't mean every port should be open.**

For example, SSH access should be restricted to trusted source addresses whenever possible rather than casually exposing port 22 to the entire internet.

Cloud networking isn't only about making connections work. It's about making them work securely.

**Day 45 complete. 🚀**

Next: **Day 46 — Azure Storage Basics**

#100DaysOfDevOps #DevOps #Azure #MicrosoftAzure #AzureNetworking #NetworkSecurity #NSG #CloudSecurity #CloudComputing #LearningInPublic #DevOpsJourney

## 14. Day 46 preview — Azure Storage Basics

Tomorrow we'll explore Azure Storage accounts, Blob storage, containers and blobs, uploading/listing files, access basics, and common DevOps storage use cases.

```text
Application
    |
    v
Azure Storage Account
    |
    v
Blob Container
    |
    v
Files / Objects
```

---

## Progress

**Day 45 / 100 complete — Public IP, Private IP & Network Security Groups.**
