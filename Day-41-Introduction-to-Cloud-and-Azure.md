# Day 41 — Introduction to Cloud Computing & Microsoft Azure

## 100 Days of DevOps — From Zero to Hero

**Day:** 41/100  
**Topic:** Introduction to Cloud Computing & Microsoft Azure  
**Category:** Azure Cloud

---

# 1. Today's Goal

The Docker fundamentals section is complete.

Today we start the **Cloud + Azure** section.

We will learn:

- What cloud computing is
- Why organizations use cloud platforms
- IaaS, PaaS and SaaS
- Public, private and hybrid cloud
- What Microsoft Azure is
- Azure regions
- Availability zones
- Azure subscriptions
- Resource groups
- Azure Portal
- Azure CLI
- How Azure fits into DevOps

---

# 2. What Problem Does Cloud Computing Solve?

Before cloud computing became mainstream, companies often had to purchase and maintain physical servers.

A typical setup looked like:

```text
                Company Data Center
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
     Servers        Storage        Networking
        |
        v
    Applications
```

The company had to manage:

- Physical servers
- Networking
- Storage
- Electricity
- Cooling
- Hardware failures
- Data center space
- Hardware upgrades

This could be expensive and slow.

---

# 3. What is Cloud Computing?

Cloud computing means using computing resources over the internet instead of owning and maintaining all the physical infrastructure yourself.

Examples include:

- Virtual machines
- Storage
- Databases
- Networks
- Containers
- Kubernetes
- Serverless services
- Monitoring
- AI services

Conceptually:

```text
              Internet
                  |
                  v
            Cloud Provider
                  |
       +----------+----------+
       |          |          |
       v          v          v
    Compute     Storage    Database
       |          |          |
       +----------+----------+
                  |
                  v
             Applications
```

---

# 4. Why Do Companies Use Cloud?

## Scalability

Need more computing power?

You can increase resources without physically purchasing another server.

```text
Small Traffic
     |
     v
  2 Servers

High Traffic
     |
     v
  10 Servers
```

## Flexibility

Cloud resources can be created, modified and removed quickly.

## Global Availability

Applications can be deployed closer to users around the world.

## Pay for Usage

Organizations can generally pay for the cloud resources they consume instead of purchasing all infrastructure upfront.

## Automation

Cloud resources can be managed using:

- CLI
- APIs
- Infrastructure as Code
- CI/CD pipelines

This is especially important for DevOps.

---

# 5. Major Cloud Providers

Some major cloud platforms are:

```text
Cloud Providers
      |
      +---- Microsoft Azure
      |
      +---- Amazon Web Services
      |
      +---- Google Cloud
```

Today we're focusing on:

## Microsoft Azure

Azure is Microsoft's cloud computing platform.

---

# 6. What is Microsoft Azure?

Azure provides cloud services for:

- Compute
- Storage
- Networking
- Databases
- Containers
- Kubernetes
- Security
- Monitoring
- AI
- DevOps

Instead of buying physical infrastructure, you can provision resources through Azure.

For example:

```text
Azure Portal
     |
     v
Create Virtual Machine
     |
     v
Choose OS
     |
     v
Choose CPU/RAM
     |
     v
Configure Network
     |
     v
Create VM
```

---

# 7. Azure and DevOps

Azure is highly relevant to DevOps because it provides services for both infrastructure and software delivery.

A simplified workflow:

```text
Developer
    |
    v
Git Repository
    |
    v
CI/CD Pipeline
    |
    v
Build
    |
    v
Test
    |
    v
Deploy
    |
    v
Azure
    |
    v
Application
```

---

# 8. Cloud Service Models

One of the most important cloud concepts is:

```text
IaaS
PaaS
SaaS
```

---

# 9. IaaS — Infrastructure as a Service

With IaaS, the cloud provider provides infrastructure such as:

- Virtual machines
- Networking
- Storage

You manage much of the operating system and application stack.

Example:

```text
Azure VM
```

Conceptually:

```text
You manage
     |
     +---- Application
     +---- Runtime
     +---- OS
     |
     v
Cloud Provider
     |
     +---- Physical Server
     +---- Networking
     +---- Storage
     +---- Data Center
```

---

# 10. PaaS — Platform as a Service

With PaaS, the cloud provider manages more of the underlying infrastructure.

You focus more on the application.

Example:

```text
Application
     |
     v
Azure App Service
     |
     v
Azure manages infrastructure
```

You don't need to manage the underlying server in the same way you would with an IaaS VM.

---

# 11. SaaS — Software as a Service

With SaaS, you use a complete software product.

Examples include:

- Email platforms
- Collaboration tools
- CRM applications

Conceptually:

```text
User
 |
 v
Software
 |
 v
Cloud Provider manages almost everything
```

---

# 12. IaaS vs PaaS vs SaaS

Think of it as a responsibility spectrum:

```text
More Control
     |
     v
    IaaS
     |
     v
    PaaS
     |
     v
    SaaS
     |
     v
Less Infrastructure Management
```

| Model | You Mainly Manage |
|---|---|
| IaaS | OS + application + configuration |
| PaaS | Application + configuration |
| SaaS | Mainly software usage/configuration |

---

# 13. Public vs Private vs Hybrid Cloud

## Public Cloud

Infrastructure is provided by a cloud provider.

Examples:

```text
Azure
AWS
Google Cloud
```

## Private Cloud

Infrastructure is dedicated to a particular organization.

```text
Company
   |
   v
Private Cloud
```

## Hybrid Cloud

Combines on-premises infrastructure and public cloud.

```text
On-Premises
     |
     |
     +--------+
              |
              v
           Azure
```

---

# 14. Azure Regions

Azure resources are deployed into geographic locations called **regions**.

Conceptually:

```text
                  Azure
                    |
       +------------+------------+
       |            |            |
       v            v            v
     Region A     Region B     Region C
```

Choosing a region can affect:

- Latency
- Data residency
- Availability
- Compliance
- Cost

For example, an application serving users in India may benefit from a geographically closer Azure region, depending on its requirements.

---

# 15. Availability Zones

Some Azure regions contain multiple availability zones.

Conceptually:

```text
              Azure Region
                   |
        +----------+----------+
        |          |          |
        v          v          v
       AZ1        AZ2        AZ3
```

Availability zones are physically separate locations within a region designed to improve resilience.

---

# 16. Core Azure Services

You don't need to memorize every Azure service.

Start with the major categories:

```text
Azure
 |
 +---- Compute
 |
 +---- Storage
 |
 +---- Networking
 |
 +---- Database
 |
 +---- Containers
 |
 +---- Kubernetes
 |
 +---- Monitoring
 |
 +---- Security
 |
 +---- DevOps
```

Examples:

### Compute

- Azure Virtual Machines
- Azure App Service
- Azure Functions

### Storage

- Azure Blob Storage
- Azure Files
- Managed Disks

### Networking

- Azure Virtual Network
- Azure Load Balancer
- Azure Application Gateway

### Containers

- Azure Container Registry
- Azure Container Apps
- Azure Kubernetes Service

### DevOps

- Azure DevOps
- Azure Repos
- Azure Pipelines

---

# 17. Azure Portal

The Azure Portal is a web-based interface for managing Azure resources.

General workflow:

```text
Azure Portal
     |
     v
Subscription
     |
     v
Resource Group
     |
     v
Azure Resource
```

For example:

```text
Subscription
     |
     v
Resource Group
     |
     +---- Virtual Machine
     |
     +---- Network
     |
     +---- Storage
```

---

# 18. What is an Azure Subscription?

A subscription is an important boundary for Azure resources, billing and access control.

Conceptually:

```text
Microsoft Account
       |
       v
Azure Subscription
       |
       v
Resource Groups
       |
       v
Resources
```

A subscription can contain many resource groups and resources.

---

# 19. What is a Resource Group?

A resource group is a logical container for Azure resources.

Example:

```text
Resource Group: devops-lab
        |
        +---- VM
        |
        +---- Virtual Network
        |
        +---- Public IP
        |
        +---- Storage
```

Resource groups make it easier to organize and manage related resources.

---

# 20. Hands-On Lab — Explore Azure Portal

If you have access to an Azure subscription:

1. Open the Azure Portal.
2. Go to **Resource Groups**.
3. Select **Create**.
4. Choose your Azure subscription.
5. Create a resource group named:

```text
rg-100days-devops
```

6. Choose an appropriate region.
7. Create the resource group.

---

# 21. Verify the Resource Group

Open:

```text
Resource Groups
```

Find:

```text
rg-100days-devops
```

Initially, it may contain no resources.

Conceptually:

```text
rg-100days-devops
        |
        +---- Resources
              |
              +---- Currently empty
```

This resource group will be useful in upcoming Azure labs.

---

# 22. Optional Azure CLI Lab

If Azure CLI is installed, verify:

```bash
az version
```

Log in:

```bash
az login
```

List subscriptions:

```bash
az account list --output table
```

Set the desired subscription:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

Verify:

```bash
az account show --output table
```

---

# 23. Create a Resource Group Using Azure CLI

Instead of using the portal, you can create resources programmatically.

Example:

```bash
az group create   --name rg-100days-devops   --location eastus
```

Check:

```bash
az group show   --name rg-100days-devops   --output table
```

The exact region should be chosen according to your Azure subscription and lab requirements.

---

# 24. Portal vs CLI

Both approaches are useful.

### Azure Portal

Good for:

- Beginners
- Visual exploration
- Learning resource relationships
- Quickly checking configuration

### Azure CLI

Good for:

- Automation
- Scripts
- DevOps pipelines
- Repeatable deployments
- Infrastructure workflows

As a DevOps engineer, you should become comfortable with both.

---

# 25. Why CLI Matters in DevOps

Imagine creating the same infrastructure manually every time:

```text
Click
Click
Click
Click
Click
...
```

Automation changes that:

```text
Script
  |
  v
Azure CLI
  |
  v
Infrastructure
```

This is one of the foundations of Infrastructure as Code and automation.

Later we'll explore tools such as Terraform and Azure automation workflows.

---

# 26. Hands-On Challenge

Try completing the following independently.

### Task 1

Open Azure Portal.

### Task 2

Create:

```text
rg-100days-devops
```

### Task 3

Find the resource group in the portal.

### Task 4

If Azure CLI is available:

```bash
az login
```

### Task 5

List subscriptions:

```bash
az account list --output table
```

### Task 6

Check the resource group:

```bash
az group show   --name rg-100days-devops   --output table
```

### Task 7

Write down the difference between:

```text
Subscription
Resource Group
Resource
```

If you can explain these three concepts clearly, you've completed today's core objective.

---

# 27. Troubleshooting

## Azure CLI not found

If:

```bash
az version
```

doesn't work, Azure CLI may not be installed.

You can continue today's lab using the Azure Portal.

---

## `az login` fails

Check:

```bash
az account show
```

If authentication has expired, run:

```bash
az login
```

again.

---

## Wrong subscription selected

List subscriptions:

```bash
az account list --output table
```

Then:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

Verify:

```bash
az account show --output table
```

---

## Resource group already exists

That's usually fine.

Check it:

```bash
az group show   --name rg-100days-devops   --output table
```

---

## Region unavailable

Not every Azure service is available in every region.

If a chosen region isn't available for a resource, select another suitable region supported by your subscription.

---

# 28. Screenshot Guidance for LinkedIn

## Screenshot 1 — Azure Portal

Show:

```text
rg-100days-devops
```

in the Azure Portal.

## Screenshot 2 — Azure CLI

Run:

```bash
az account show --output table
```

Capture the subscription information.

## Screenshot 3 — Resource Group CLI

Run:

```bash
az group show   --name rg-100days-devops   --output table
```

Capture the output.

### Best LinkedIn screenshot

The Azure Portal showing:

```text
rg-100days-devops
```

is the best visual for today's post because it clearly demonstrates the beginning of the Azure hands-on journey.

---

# 29. Key Takeaways

Today I learned:

- What cloud computing is
- Why organizations use cloud platforms
- What Microsoft Azure is
- IaaS
- PaaS
- SaaS
- Public cloud
- Private cloud
- Hybrid cloud
- Azure regions
- Availability zones
- Azure subscriptions
- Resource groups
- Azure Portal
- Azure CLI
- Why automation matters in DevOps

The most important mental model:

```text
                 Azure
                   |
              Subscription
                   |
              Resource Group
                   |
        +----------+----------+
        |          |          |
        v          v          v
       VM       Network     Storage
```

---

# 30. LinkedIn Post — Day 41/100

☁️ **Day 41/100 — Starting My Azure Journey**

The Docker fundamentals section of my **100 Days of DevOps** journey is complete.

Today I started learning **Cloud Computing and Microsoft Azure**.

Before jumping directly into Azure services, I wanted to understand the bigger picture.

One question I focused on:

**Why do organizations use the cloud?**

Traditionally, companies had to purchase and maintain physical infrastructure:

```text
Servers
Storage
Networking
Data Center
Cooling
Hardware
```

Cloud changed this model.

Instead of owning all the infrastructure, organizations can provision computing resources when they need them.

Today I learned about:

✅ Cloud Computing

✅ IaaS

✅ PaaS

✅ SaaS

✅ Public vs Private vs Hybrid Cloud

✅ Azure Regions

✅ Availability Zones

✅ Azure Subscriptions

✅ Resource Groups

✅ Azure Portal

✅ Azure CLI

One concept I found particularly important for DevOps:

**Automation.**

Creating infrastructure manually is useful for learning, but DevOps is about making infrastructure and deployments repeatable.

For example:

```text
Script
  |
  v
Azure CLI
  |
  v
Infrastructure
```

I also created my first resource group for this learning journey:

```text
rg-100days-devops
```

The Azure journey has officially started. 🚀☁️

Next up:

**Day 42 — Azure Resource Groups, Subscriptions & Resource Management**

#100DaysOfDevOps #DevOps #Azure #MicrosoftAzure #CloudComputing #Cloud #AzureDevOps #LearningInPublic #DevOpsJourney #CloudEngineer

---

# 31. GitHub Commit

Save this file as:

```text
Day-41-Introduction-to-Cloud-and-Azure.md
```

Check:

```bash
git status
```

Add:

```bash
git add Day-41-Introduction-to-Cloud-and-Azure.md
```

Commit:

```bash
git commit -m "Add Day 41 Introduction to Cloud and Azure"
```

Push:

```bash
git push origin main
```

---

# 32. Day 42 Preview

## Day 42 — Azure Resource Groups, Subscriptions & Resource Management

Tomorrow we'll go deeper into Azure resource organization.

We'll explore:

```text
Azure
 |
 +---- Subscription
          |
          +---- Resource Group
                    |
          +---------+---------+
          |         |         |
          v         v         v
         VM      Network    Storage
```

We'll also practice managing resources through:

- Azure Portal
- Azure CLI
- Resource groups
- Resource tags
- Basic resource lifecycle management

---

# 33. Progress

**41/100 Days Completed 🚀**

```text
[████████████████████░░░░░░░░░░] 41%
```

**Docker → Complete ✅**

**Azure → Started ☁️**

Keep learning. Keep practicing. Keep building.
