# 100 Days of DevOps — Day 53/100
## Azure Load Balancer vs. Application Gateway: Traffic Distribution and Troubleshooting

**Goal:** Understand the difference between Azure Load Balancer and Application Gateway, learn the main building blocks, and practise troubleshooting backend health.  
**Environment:** Azure subscription and Azure CLI. You can complete the inspection steps even if you do not deploy new resources.  
**Estimated time:** 45–60 minutes.

> **Cost and safety:** These services can incur charges. Prefer existing lab resources, review Azure pricing before deploying anything, and remove lab resources when finished. Do not modify production listeners, probes, backend pools, or rules without approval.

---

## 1. Why do we need load balancing?

Imagine one web server receives all requests. If traffic increases, it may become slow. If it fails, users may lose access to the application.

A load balancer distributes traffic across backend instances. With appropriate health checks, it can avoid sending new traffic to unhealthy instances.

```text
Users
  |
  v
Frontend IP / DNS
  |
  v
Load-balancing service
  |
  +----------+----------+
  |          |          |
  v          v          v
 Web VM 1  Web VM 2  Web VM 3
```

Load balancing does not automatically make an application highly available. You also need healthy backend instances, appropriate network rules, application resilience, and a deployment strategy.

---

## 2. Azure Load Balancer vs. Application Gateway

| Feature | Azure Load Balancer | Azure Application Gateway |
|---|---|---|
| OSI layer | Layer 4 | Layer 7 |
| Main traffic | TCP and UDP | HTTP and HTTPS web traffic |
| Understands URLs/HTTP headers | No | Yes |
| URL-path routing | No | Yes |
| Host-based routing | No | Yes |
| Health probes | Yes | Yes |
| Web Application Firewall (WAF) | No built-in WAF | Available with WAF SKU |
| Common use | Network-level traffic distribution | Web apps, TLS termination, HTTP routing |

### Azure Load Balancer

Use it when you need Layer 4 distribution for TCP or UDP traffic. Common building blocks include:

- **Frontend IP:** The address clients connect to.
- **Backend pool:** The virtual machines or supported resources receiving traffic.
- **Health probe:** Checks whether a backend is healthy.
- **Load-balancing rule:** Maps a frontend IP and port to a backend pool and port.
- **Inbound NAT rule:** Maps a frontend port to a specific backend instance, when needed.

Azure Load Balancer can be public-facing or internal.

### Azure Application Gateway

Use it when you need Layer 7 features for HTTP/HTTPS applications. Its main building blocks include:

- **Frontend IP:** The address clients connect to.
- **Listener:** Accepts requests on a protocol, port, and optional hostname.
- **Backend pool:** The web servers or supported targets receiving requests.
- **Backend HTTP settings:** Configure backend protocol, port, timeout, and related behavior.
- **Health probe:** Checks whether a backend is ready to receive traffic.
- **Routing rule:** Connects a listener to backend settings and a backend pool.
- **WAF (optional):** Helps protect web applications from common web attacks.

Application Gateway supports host-based routing, URL-path routing, and TLS termination. Exact features depend on the selected SKU and configuration.

### Simple decision guide

- Choose **Azure Load Balancer** for TCP/UDP distribution at Layer 4.
- Choose **Application Gateway** for HTTP/HTTPS routing based on hostnames or URL paths, and for its web application features.
- Some architectures use both, or combine them with other Azure networking services, when each layer has a clear purpose.

---

## 3. Lab A — Sign in and discover existing resources

Run these commands from macOS Terminal.

### Step 1: Sign in and verify the subscription

```bash
az login
az account show --output table
az account list --output table
```

If needed, select your intended subscription:

```bash
az account set --subscription "<SUBSCRIPTION_ID_OR_NAME>"
az account show --output table
```

### Step 2: List existing Load Balancers

```bash
az network lb list --output table
```

To inspect one specific Load Balancer:

```bash
RG="<RESOURCE_GROUP>"
LB="<LOAD_BALANCER_NAME>"

az network lb show \\
  --resource-group "$RG" \\
  --name "$LB" \\
  --output json
```

List its rules and health probes:

```bash
az network lb rule list \\
  --resource-group "$RG" \\
  --lb-name "$LB" \\
  --output table

az network lb probe list \\
  --resource-group "$RG" \\
  --lb-name "$LB" \\
  --output table
```

**Expected:** If a Load Balancer exists, these commands show its configuration, rules, and probes. An empty list may mean there is no Load Balancer in the selected subscription.

### Step 3: List existing Application Gateways

```bash
az network application-gateway list --output table
```

Inspect a specific Application Gateway:

```bash
AGW="<APPLICATION_GATEWAY_NAME>"

az network application-gateway show \\
  --resource-group "$RG" \\
  --name "$AGW" \\
  --output json
```

Check backend health:

```bash
az network application-gateway show-backend-health \\
  --resource-group "$RG" \\
  --name "$AGW" \\
  --output json
```

**Expected:** The backend-health output reports the status of configured backend targets and may include reasons for an unhealthy state.

If resources are in different resource groups, set `RG` appropriately. You need sufficient Azure permissions to inspect these resources.

---

## 4. Lab B — Troubleshoot an unhealthy backend

Assume your frontend is reachable, but requests fail or some backends show as unhealthy.

### Step 1: Confirm backend health

For Application Gateway:

```bash
az network application-gateway show-backend-health \\
  --resource-group "$RG" \\
  --name "$AGW" \\
  --output json
```

Review:
- Which backend target is unhealthy?
- What protocol and port are being used?
- Is the health probe using the expected path and hostname?
- Does the response match the configured success criteria?

For Azure Load Balancer, inspect its configured probes and rules:

```bash
az network lb probe list -g "$RG" --lb-name "$LB" -o json
az network lb rule list -g "$RG" --lb-name "$LB" -o json
```

The CLI configuration alone may not show every live data-plane health detail. Use the Azure portal's monitoring and health views where available.

### Step 2: Verify the backend service

Connect to a backend VM using an approved method and run:

```bash
sudo systemctl status nginx --no-pager
sudo ss -lntp
curl -I http://localhost
```

For Apache on Ubuntu/Debian:

```bash
sudo systemctl status apache2 --no-pager
curl -I http://localhost
```

The service name and port depend on your application.

**Interpretation:**
- If the local request fails, investigate the service, application logs, configuration, and OS firewall.
- If the local request works but the probe fails, investigate network access, probe port/path, host header, protocol, and response status.
- If only one backend is unhealthy, compare its configuration with a healthy backend.

### Step 3: Check network security

Verify that:
- The backend can receive traffic from the relevant Azure service or subnet.
- The NSG and guest firewall allow the configured backend port.
- The route path is correct.
- The service listens on an interface reachable by the load-balancing service, not only `127.0.0.1`.
- For Application Gateway, backend TLS certificates, hostname settings, and probe configuration are correct when HTTPS is used.

Use the least-permissive network rule that meets the architecture. Do not open backend ports to the entire internet just to make a probe pass.

---

## 5. Lab C — Test the frontend

From a client that can reach the frontend:

```bash
curl -I --connect-timeout 5 "http://<FRONTEND_IP_OR_DNS>"
```

For HTTPS:

```bash
curl -Iv --connect-timeout 5 "https://<FRONTEND_DNS_NAME>"
```

Use the configured hostname when the listener or TLS certificate expects it.

### What the result can tell you

- **HTTP 200:** A web server responded successfully.
- **HTTP 301/302:** A redirect was returned; inspect the `Location` header.
- **HTTP 403:** The request may be denied by the application, WAF, or access policy.
- **HTTP 502/503:** The gateway or application may not be able to serve the request; check backend health and logs.
- **Connection timeout:** Check frontend IP/DNS, NSGs, routes, listeners, and client connectivity.
- **TLS/certificate error:** Check certificate validity, hostname, listener configuration, and TLS settings.

An HTTP response alone does not prove every backend is healthy. Correlate frontend tests with backend health and application logs.

---

## 6. Health probes: what to check

A health probe determines whether a backend should receive traffic. Common mistakes include:

1. Probe uses the wrong port.
2. HTTP probe path does not exist.
3. Application returns a status code outside the configured success range.
4. Backend only listens on localhost.
5. NSG or OS firewall blocks probe traffic.
6. Application startup takes longer than the probe configuration allows.
7. HTTPS hostname or certificate configuration is incorrect for the gateway.

**Best practice:** Use a purpose-built health endpoint where possible. Keep it lightweight and ensure it checks the dependencies needed to serve requests without making the probe itself fragile.

---

## 7. Troubleshooting checklist

When the frontend is unavailable or backends are unhealthy, follow this order:

1. Confirm the correct frontend IP or DNS name.
2. Confirm the Load Balancer rule or Application Gateway listener and routing rule.
3. Check backend pool membership and health.
4. Check probe protocol, port, path, hostname, and expected response.
5. Confirm NSG, route, and guest firewall rules.
6. Check whether the service is running and listening on the expected port.
7. Inspect application logs and gateway diagnostics.
8. Re-test after an approved change from the same client path.

| Symptom | Likely areas to inspect |
|---|---|
| Frontend times out | Frontend IP/DNS, listener/rule, NSG, route, client path |
| Application Gateway returns 502 | Backend health, backend port, probe, TLS/hostname, NSG |
| Some requests fail intermittently | One unhealthy backend, uneven application behavior, capacity, session assumptions |
| HTTP 403 | Application authorization, WAF policy, listener/routing configuration |
| HTTPS certificate warning | Certificate, hostname, expiry, TLS configuration |
| Load Balancer sends no traffic to a backend | Probe health, backend membership, rule and port configuration |

---

## 8. Production best practices

- Deploy backend instances across appropriate fault domains or availability zones where supported and suitable.
- Configure health probes to reflect whether a backend can serve requests.
- Monitor frontend availability, backend health, latency, and error rates.
- Use TLS appropriately and manage certificates securely.
- Use WAF where the threat model and requirements call for it; tune policies carefully.
- Avoid exposing backend servers directly to the internet when the architecture does not require it.
- Test failover and backend removal in a non-production environment.
- Document rollback steps and validate changes after deployment.
- Review current Azure pricing and SKU capabilities before deploying resources.

---

## 9. Day 53 practical checklist

- [ ] Compared Layer 4 Load Balancer with Layer 7 Application Gateway.
- [ ] Listed existing Azure resources using Azure CLI.
- [ ] Inspected rules and health probes where resources were available.
- [ ] Checked Application Gateway backend health if applicable.
- [ ] Tested the frontend with `curl`.
- [ ] Checked backend service status, listening ports, and network rules.
- [ ] Avoided unapproved production changes and unnecessary resource costs.

## 10. What to capture for LinkedIn

Capture real results from your own environment:

1. A comparison diagram or notes showing Layer 4 versus Layer 7.
2. Azure CLI output listing a Load Balancer or Application Gateway.
3. Backend health output, if you have an Application Gateway.
4. A terminal screenshot showing the `curl` test and HTTP response.
5. A portal screenshot of the relevant health probe or routing rule.

Hide sensitive IPs, hostnames, internal topology, secrets, and certificate private keys. Do not claim a test passed unless you ran it.

---

## 11. LinkedIn post — Day 53/100

**100 Days of DevOps — Day 53/100 🚀**

Today I explored **Azure Load Balancer vs. Application Gateway**.

Both distribute traffic, but they solve different problems.

In this hands-on session, I focused on:
- Understanding Layer 4 versus Layer 7 load balancing.
- Learning frontend IPs, backend pools, rules, listeners, and health probes.
- Inspecting Azure resources using Azure CLI.
- Checking backend health and service listeners.
- Testing frontend responses with `curl`.
- Troubleshooting common issues such as unhealthy backends and HTTP 502/503 responses.

**Key takeaway:** A frontend can be reachable while the backend is unhealthy. Check the probe configuration, network path, and application service before changing rules.

**Question:** For a web application that needs URL-path routing and WAF protection, which Azure service would you choose—and why?

Learning one practical DevOps concept at a time. 🚀

#100DaysOfDevOps #Azure #AzureLoadBalancer #ApplicationGateway #CloudComputing #DevOps #Networking #LearningInPublic

---

## 12. Tomorrow — Day 54/100

**Next topic: Azure DNS and Private DNS Zones**

We'll explore name resolution, public versus private DNS, linking private DNS zones to VNets, and troubleshooting DNS resolution between Azure VMs.

## Official documentation

- Azure Load Balancer: https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-overview
- Azure Application Gateway: https://learn.microsoft.com/en-us/azure/application-gateway/overview
- Application Gateway backend health: https://learn.microsoft.com/en-us/azure/application-gateway/application-gateway-backend-health
- Azure CLI Load Balancer commands: https://learn.microsoft.com/en-us/cli/azure/network/lb
- Azure CLI Application Gateway commands: https://learn.microsoft.com/en-us/cli/azure/network/application-gateway
