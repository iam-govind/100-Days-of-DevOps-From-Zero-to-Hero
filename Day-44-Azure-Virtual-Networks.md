# Day 44/100 — Azure Virtual Networks (VNet)

## Goal
Continue from Day 43: Azure Virtual Machines. Learn how Azure resources communicate through VNets and subnets, then create and inspect a VNet using Azure CLI.

## 1. What is a VNet?
An Azure Virtual Network is a logically isolated network in Azure. It lets Azure resources communicate when routing and security rules permit it.

```text
             Azure VNet
            10.10.0.0/16
                  |
          +-------+-------+
          |               |
          v               v
      Web Subnet       App Subnet
     10.10.1.0/24     10.10.2.0/24
          |               |
          v               v
        Web VM          App VM
```

## 2. Key terms

| Term | Meaning |
|---|---|
| VNet | Logically isolated Azure network |
| Subnet | Smaller IP range inside a VNet |
| Private IP | Address used inside private networks |
| Public IP | Address for internet communication when routing/security allow it |
| NIC | Network interface connecting a VM to a subnet |
| NSG | Filters network traffic using security rules |
| CIDR | IP range notation, e.g. `10.10.0.0/16` |

A VM connects through its network interface:

```text
Azure VM -> NIC -> Subnet -> VNet
```

## 3. Public IP vs private IP

```text
                 Internet
                    |
                 Public IP
                    |
                  Web VM
               Private IP
                    |
                    v
               Azure VNet
                    |
                    v
                Database VM
                Private IP
```

A public IP alone does not guarantee connectivity. NSGs, routing, host firewalls, and the service listening on the port also matter.

## 4. What is a subnet?
A subnet divides a VNet's address space into smaller ranges.

```text
VNet: 10.10.0.0/16
 |
 +-- Web Subnet: 10.10.1.0/24
 +-- App Subnet: 10.10.2.0/24
 +-- Database Subnet: 10.10.3.0/24
```

Azure reserves five IPv4 addresses in each subnet.

## 5. Hands-on lab — Create a VNet

### Step 1: Sign in and check subscription

```bash
az login
az account show --output table
```

If needed:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

### Step 2: Set variables

```bash
RESOURCE_GROUP="rg-100days-devops"
LOCATION="eastus"
VNET_NAME="vnet-day44-devops"
WEB_SUBNET="web-subnet"
APP_SUBNET="app-subnet"
```

Check the resource group:

```bash
az group show --name "$RESOURCE_GROUP" --output table
```

If it doesn't exist, create it:

```bash
az group create --name "$RESOURCE_GROUP" --location "$LOCATION"
```

### Step 3: Create the VNet and first subnet

```bash
az network vnet create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VNET_NAME" \
  --location "$LOCATION" \
  --address-prefixes 10.10.0.0/16 \
  --subnet-name "$WEB_SUBNET" \
  --subnet-prefixes 10.10.1.0/24
```

### Step 4: Create the second subnet

```bash
az network vnet subnet create \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$APP_SUBNET" \
  --address-prefixes 10.10.2.0/24
```

### Step 5: Verify the VNet

```bash
az network vnet show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VNET_NAME" \
  --output table
```

Inspect the address space:

```bash
az network vnet show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VNET_NAME" \
  --query addressSpace.addressPrefixes \
  --output json
```

Expected:

```json
[
  "10.10.0.0/16"
]
```

### Step 6: List subnets

```bash
az network vnet subnet list \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --output table
```

Expected ranges:

| Subnet | Range |
|---|---|
| `web-subnet` | `10.10.1.0/24` |
| `app-subnet` | `10.10.2.0/24` |

### Step 7: Inspect the Day 43 VM's network (if it still exists)

```bash
az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "vm-100days-devops" \
  --query "networkProfile.networkInterfaces[].id" \
  --output tsv
```

To inspect the NIC's private IP and subnet:

```bash
NIC_ID=$(az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "vm-100days-devops" \
  --query "networkProfile.networkInterfaces[0].id" \
  --output tsv)

az network nic show \
  --ids "$NIC_ID" \
  --query "{privateIP:ipConfigurations[0].privateIPAddress,subnet:ipConfigurations[0].subnet.id}" \
  --output json
```

Creating a separate VNet does **not** automatically move an existing VM into it.

## 6. VNet peering
VNet peering can connect VNets over the Azure backbone, subject to configuration and security rules. Address spaces should not overlap for typical peering setups.

```text
VNet A  -------- Peering --------  VNet B
  |                                  |
 VM A                                VM B
```

Peering does not automatically open ports. NSGs and host firewalls still apply.

## 7. Troubleshooting

**Resource group not found**
```bash
az group list --output table
```
Check the selected subscription.

**Address prefix overlaps**
Ensure each subnet range fits inside the VNet and doesn't overlap another subnet.

**VNet already exists**
```bash
az network vnet list --output table
```
Reuse it or choose a different name and non-overlapping range.

**Can't connect to a VM**
Check VM state, IP address, routing, NSG rules, host firewall, and whether the target service is listening.

**AuthorizationFailed**
Check that your Azure identity has permissions to manage networking resources.

## 8. Hands-on challenge
1. Create `vnet-day44-devops`.
2. Set address space to `10.10.0.0/16`.
3. Create `web-subnet` using `10.10.1.0/24`.
4. Create `app-subnet` using `10.10.2.0/24`.
5. List the VNet and subnets.
6. Inspect the Day 43 VM's NIC if the VM still exists.
7. Explain why creating a VNet doesn't automatically move an existing VM.

Expected architecture:

```text
vnet-day44-devops (10.10.0.0/16)
 |
 +-- web-subnet (10.10.1.0/24)
 |
 +-- app-subnet (10.10.2.0/24)
```

## 9. Screenshots for LinkedIn
Capture real screenshots showing:
1. Azure Portal with `vnet-day44-devops`.
2. Subnets `web-subnet` and `app-subnet`.
3. Terminal output from `az network vnet subnet list`.
4. Your network diagram.

```text
             Azure VNet
            10.10.0.0/16
                  |
          +-------+-------+
          |               |
          v               v
      Web Subnet       App Subnet
     10.10.1.0/24     10.10.2.0/24
```

## 10. Key takeaways
- VNets provide logical network isolation.
- Subnets divide an address space.
- VMs connect to subnets through NICs.
- Public IPs do not bypass NSG or firewall rules.
- VNet peering connects networks but doesn't automatically permit every type of traffic.

## 11. GitHub update

Save this file as `Day-44-Azure-Virtual-Networks.md` in your repository.

```bash
git status
git add Day-44-Azure-Virtual-Networks.md
git commit -m "Add Day 44 Azure Virtual Networks"
git push origin main
```

## 12. LinkedIn post — Day 44/100

**Day 44/100 — Understanding Azure Virtual Networks 🌐**

Yesterday I created an Azure VM and connected to it using SSH.

Today I explored the networking layer behind cloud infrastructure: **Azure Virtual Networks (VNets)**.

A VM needs more than compute and storage. It also needs a network that defines how it communicates with other resources.

Today I practised:

✅ Creating an Azure VNet using Azure CLI  
✅ Defining a VNet address space  
✅ Creating separate subnets  
✅ Understanding public IPs and private IPs  
✅ Inspecting a VM's network interface  
✅ Learning the basics of VNet peering

```text
             Azure VNet
            10.10.0.0/16
                  |
          +-------+-------+
          |               |
          v               v
      Web Subnet       App Subnet
     10.10.1.0/24     10.10.2.0/24
```

One important lesson: creating a VNet doesn't automatically move an existing VM into it, and having a public IP doesn't mean every connection is allowed. Routing and security rules matter too.

Understanding networking is a key step toward designing secure and reliable cloud infrastructure.

**Day 44 complete. 🚀**

Next: **Day 45 — Public IP, Private IP and Network Security Groups**

#100DaysOfDevOps #DevOps #Azure #MicrosoftAzure #AzureNetworking #VirtualNetwork #CloudComputing #LearningInPublic #DevOpsJourney

## 13. Day 45 preview
Next we'll explore public and private IP addresses in more detail, Network Security Groups, inbound/outbound rules, and common ports such as SSH (22) and HTTP (80).

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

**Progress: Day 44 / 100 complete — Azure Virtual Networks.**
