# Day 54/100 — Azure DNS and Private DNS Zones

**100 Days of DevOps — From Zero to Hero**  
**Goal:** Understand Azure DNS and configure private name resolution for Azure VMs.

## 1. What is DNS?

The Domain Name System (DNS) translates hostnames into IP addresses. For example, `app.lab.internal` can resolve to a private IP such as `10.0.2.4`.

| Term | Meaning |
|---|---|
| A record | Maps a hostname to an IPv4 address |
| CNAME | Maps one hostname to another hostname |
| Public DNS zone | Contains records intended for public DNS resolution |
| Private DNS zone | Resolves names within linked private networks |
| Azure-provided DNS | Default Azure VNet recursive resolver: `168.63.129.16` |

Azure DNS public zones host public records; Azure Private DNS resolves internal names for linked VNets. A zone alone does not make a domain publicly resolvable; domain delegation is also required. DNS resolution and network connectivity are separate.

## 2. Lab architecture

Example values—replace with your own resources:

- Private DNS zone: `lab.internal`
- DNS record: `app.lab.internal`
- Example private IP: `10.0.2.4`
- VNet: `dev-vnet`
- Link name: `dev-vnet-link`

**Safety note:** The commands below create resources. Confirm your subscription and names first, use a dedicated lab resource group where possible, and check applicable pricing. Do not modify production DNS zones for practice.

## 3. Hands-on lab

### Step 1 — Sign in and select a subscription

```bash
az login
az account list --output table
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
az account show --output table
```

### Step 2 — Set variables and verify your VNet

```bash
RG="<YOUR_RESOURCE_GROUP>"
VNET="<YOUR_VNET_NAME>"
DNS_RG="$RG"
ZONE="lab.internal"
LINK="dev-vnet-link"

az group show --name "$RG" --output table
az network vnet show --resource-group "$RG" --name "$VNET" --output table
```

If the zone belongs in a different resource group, change `DNS_RG`.

### Step 3 — Create the private DNS zone

```bash
az network private-dns zone create \
  --resource-group "$DNS_RG" \
  --name "$ZONE"

az network private-dns zone show \
  --resource-group "$DNS_RG" \
  --name "$ZONE" \
  --output table
```

### Step 4 — Link the zone to the VNet

```bash
VNET_ID=$(az network vnet show \
  --resource-group "$RG" \
  --name "$VNET" \
  --query id --output tsv)

az network private-dns link vnet create \
  --resource-group "$DNS_RG" \
  --zone-name "$ZONE" \
  --name "$LINK" \
  --virtual-network "$VNET_ID" \
  --registration-enabled false
```

Verify the link:

```bash
az network private-dns link vnet list \
  --resource-group "$DNS_RG" \
  --zone-name "$ZONE" \
  --output table
```

Auto-registration is optional. This lab uses a manually created record, so `--registration-enabled false` is appropriate.

### Step 5 — Add an A record

Replace the example IP with the real private IP of your intended VM or lab endpoint.

```bash
VM_PRIVATE_IP="10.0.2.4"

az network private-dns record-set a add-record \
  --resource-group "$DNS_RG" \
  --zone-name "$ZONE" \
  --record-set-name "app" \
  --ipv4-address "$VM_PRIVATE_IP"

az network private-dns record-set a list \
  --resource-group "$DNS_RG" \
  --zone-name "$ZONE" \
  --output table
```

The hostname is now `app.lab.internal`.

### Step 6 — Test from a Linux VM in the linked VNet

```bash
nslookup app.lab.internal
getent hosts app.lab.internal
```

If installed:

```bash
dig app.lab.internal +short
```

Expected result: the configured private IP, for example `10.0.2.4`.

To test HTTP, the record must point to a reachable server, a web server must be running, and network rules must allow traffic:

```bash
curl -I http://app.lab.internal
```

DNS resolving successfully does not guarantee that the application is reachable.

## 4. Read-only discovery commands

```bash
# List private DNS zones
az network private-dns zone list --output table

# Inspect a zone
az network private-dns zone show \
  --resource-group "<DNS_RESOURCE_GROUP>" \
  --name "<PRIVATE_ZONE_NAME>" --output json

# List VNet links
az network private-dns link vnet list \
  --resource-group "<DNS_RESOURCE_GROUP>" \
  --zone-name "<PRIVATE_ZONE_NAME>" --output table

# List A records
az network private-dns record-set a list \
  --resource-group "<DNS_RESOURCE_GROUP>" \
  --zone-name "<PRIVATE_ZONE_NAME>" --output table

# Inspect VNet DNS settings
az network vnet show \
  --resource-group "<VNET_RESOURCE_GROUP>" \
  --name "<VNET_NAME>" \
  --query "dhcpOptions.dnsServers" --output json
```

## 5. Troubleshooting

| Symptom | Possible cause | Check |
|---|---|---|
| `NXDOMAIN` | Missing record, typo, or zone not visible to resolver | Record name and VNet link |
| Wrong IP | Incorrect or stale A record | Inspect record set |
| Works from one VM only | Different DNS configuration or VNet link | Compare resolvers and links |
| DNS timeout | Resolver unreachable or DNS traffic blocked | Resolver, firewall, NSG, routes |
| Name resolves but SSH/HTTP fails | Network or service issue | Route, NSG, port, service status |

Linux checks:

```bash
cat /etc/resolv.conf
resolvectl status       # if systemd-resolved is available
getent hosts app.lab.internal
ping -c 3 <PRIVATE_IP>  # ICMP may be blocked
curl -Iv http://app.lab.internal
```

A common mistake is linking the private zone to one VNet but testing from a VM in another. Check the link and verify which DNS server the VM uses. If the VNet uses custom DNS, configure forwarding for the private zone. VNet peering alone does not automatically configure every DNS-resolution path.

## 6. Production best practices

- Use private DNS for internal services that should not be exposed through public DNS.
- Link zones only to VNets that need them.
- Keep records aligned with actual resource IPs.
- Configure and test forwarding when using custom DNS servers.
- Plan DNS resolution for hub-and-spoke and hybrid networks.
- Test DNS, routing, security rules, and the application port separately.

## 7. Screenshot checklist

1. Azure Portal showing the `lab.internal` zone.
2. The VNet link.
3. The `app` A record and private IP.
4. `nslookup app.lab.internal` output.
5. A successful `curl` response, if a web server is configured.

## 8. LinkedIn post

🚀 **Day 54/100 — Azure DNS and Private DNS Zones**

Have you ever tried connecting to an Azure VM by hostname, only to discover that the name doesn't resolve?

That’s where DNS troubleshooting becomes an essential cloud and DevOps skill.

🔹 **What I learned today:**
- DNS translates hostnames into IP addresses.
- Azure DNS hosts public DNS zones.
- Azure Private DNS resolves internal hostnames within linked virtual networks.
- A private DNS zone needs the right VNet link and DNS records.
- DNS resolution and network connectivity are two different things.

🛠️ **Hands-on practice:**
1. Created a private DNS zone.
2. Linked it to an Azure Virtual Network.
3. Added an A record for an internal hostname.
4. Tested name resolution from Linux using `nslookup` and `getent`.
5. Reviewed common causes of `NXDOMAIN`, incorrect IPs, and DNS timeouts.

💡 **Real-world lesson:** A hostname can resolve correctly while the application remains unreachable. Check DNS, routing, security rules, and the service itself separately.

Cloud troubleshooting is about following the evidence layer by layer—not guessing.

I’m continuing my **100 Days of DevOps — From Zero to Hero** journey with practical labs and real-world concepts.

Follow along for the next lesson!

#100DaysOfDevOps #Azure #MicrosoftAzure #DevOps #CloudComputing #DNS #Networking #LearningInPublic

## 9. Suggested LinkedIn visual

Create a simple diagram:

`Client VM → Azure DNS / Private DNS Zone → Private IP → Internal application VM`

Label the VNet link and show that the destination VM does not need a public IP for private name resolution.

## 10. Next lesson

**Day 55/100 — Azure Monitor, Log Analytics, and VM Troubleshooting**

We'll explore metrics, logs, queries, alerts, and a practical workflow for diagnosing Azure VM and application issues.

## Official Microsoft documentation

- [Troubleshoot Azure DNS](https://learn.microsoft.com/en-us/azure/dns/dns-troubleshoot)
- [Virtual network links for Azure Private DNS zones](https://learn.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links)
- [Troubleshoot private endpoint DNS resolution](https://learn.microsoft.com/en-us/troubleshoot/azure/private-link/troubleshoot-private-endpoint-dns-resolution)
