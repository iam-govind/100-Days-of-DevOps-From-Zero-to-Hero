# Day 42 — Azure Resource Groups, Subscriptions & Resource Management

## 🎯 Day 42 Goal

By the end of today, you should understand:

- What an Azure Resource is
- What an Azure Subscription is
- What a Resource Group is
- How Subscription → Resource Group → Resources are related
- Resource Group locations
- Azure Tags
- Resource lifecycle management
- How to manage Resource Groups using Azure Portal
- How to manage them using Azure CLI
- How Resource Group deletion affects resources

---

## 1. What Is an Azure Resource?

Almost everything you create in Azure is called a **resource**.

Examples:

- Virtual Machine
- Storage Account
- Virtual Network
- Public IP
- Network Security Group
- Azure Database
- App Service
- Load Balancer
- Key Vault

Think of a resource as an individual building block of your cloud infrastructure.

```text
                    Azure
                      |
        +-------------+-------------+
        |             |             |
       VM          Storage         Network
        |             |             |
     Resource      Resource       Resource
```

---

## 2. What Is an Azure Subscription?

An **Azure Subscription** is a logical and billing boundary for Azure resources.

It helps with:

- Billing
- Access control
- Resource organization
- Usage tracking
- Quotas and limits

```text
Microsoft Account / Organization
              |
              v
       Azure Subscription
              |
      +-------+-------+
      |               |
      v               v
 Resource Group A   Resource Group B
      |               |
   +--+--+         +--+--+
   |     |         |     |
  VM   VNet       VM   Storage
```

You can have multiple subscriptions depending on your organization and requirements.

---

## 3. What Is a Resource Group?

A **Resource Group (RG)** is a logical container for Azure resources.

```text
Resource Group
rg-100days-devops
        |
        +--- VM
        |
        +--- VNet
        |
        +--- NSG
        |
        +--- Public IP
        |
        +--- Storage Account
```

Resource Groups help you:

- Organize resources
- Manage permissions
- Monitor resources
- Apply tags
- Track deployments
- Delete related resources together

---

## 4. Subscription vs Resource Group vs Resource

This is important for Azure interviews.

| Level | Purpose |
|---|---|
| Subscription | Billing and management boundary |
| Resource Group | Logical container for resources |
| Resource | Actual Azure service |

Example:

```text
Azure Subscription
        |
        v
Dev Subscription
        |
        v
Resource Group
rg-devops-lab
        |
        +-------- VM
        |
        +-------- VNet
        |
        +-------- NSG
        |
        +-------- Public IP
```

A simple analogy:

- **Subscription = Building**
- **Resource Group = Room**
- **Resources = Objects inside the room**

---

## 5. Why Do We Need Resource Groups?

Without organization:

```text
VM
VM
Storage
VNet
VM
Database
NSG
Storage
VM
...
```

With Resource Groups:

```text
Production
   |
   +--- Web Application
   +--- Database
   +--- Network

Development
   |
   +--- Web Application
   +--- Database
   +--- Network

Monitoring
   |
   +--- Log Analytics
   +--- Alerts
```

This makes cloud infrastructure easier to manage.

---

## 6. Can Resources in One Resource Group Be in Different Regions?

**Yes.**

A Resource Group can contain resources deployed in different Azure regions, subject to the capabilities and constraints of those resource types.

For example:

```text
Resource Group
rg-company-app
        |
        +--- VM
        |    East US
        |
        +--- Storage
        |    Central US
        |
        +--- Another Resource
             West Europe
```

The **Resource Group itself has a location**. That location is used for Resource Group metadata.

It does **not** mean every resource inside the Resource Group must be deployed in that same region.

---

## 7. Resource Groups and Lifecycle

Resource Groups are useful when resources share a lifecycle.

```text
rg-devops-lab
       |
       +--- VM
       +--- VNet
       +--- NSG
       +--- Public IP
       +--- Storage
```

After finishing a temporary lab, you can delete the Resource Group.

Deleting a Resource Group generally deletes the resources contained within it.

```text
Delete Resource Group
        |
        v
+-----------------------+
| VM                    |
| VNet                  |
| NSG                   |
| Public IP             |
| Storage               |
+-----------------------+
        |
        v
      Deleted
```

> ⚠️ **Warning:** Resource Group deletion is destructive. Don't delete production resources just to clean up a lab.

---

## 8. Azure Resource Tags

Tags are key-value metadata attached to Azure resources.

Example:

```text
Environment = Dev
Project     = 100DaysDevOps
Owner       = DevOps
CostCenter  = Learning
```

Useful tags include:

```text
Environment = Dev
Project     = 100DaysDevOps
Purpose     = Learning
```

Tags help with:

- Cost tracking
- Resource organization
- Automation
- Reporting
- Ownership
- Environment identification

---

## 9. Naming Conventions

Good naming is important in real DevOps environments.

Instead of:

```text
vm1
vm2
storage123
network1
```

Use meaningful names:

```text
rg-devops-dev
rg-devops-prod

vm-web-dev
vm-web-prod

vnet-devops-dev
vnet-devops-prod
```

A common pattern is:

```text
<resource-type>-<application>-<environment>
```

Example:

```text
rg-devops-dev
vm-web-dev
vnet-web-dev
```

> Note: Azure resource types have different naming restrictions. Always check the naming rules for the specific service.

---

# 🧪 10. Azure Portal Hands-On Lab

We'll use the Resource Group created in Day 41:

```text
rg-100days-devops
```

### Step 1 — Open Azure Portal

Go to:

https://portal.azure.com/

Sign in to your Azure account.

### Step 2 — Open Resource Groups

Search for:

```text
Resource groups
```

Open **Resource groups**.

You should see:

```text
rg-100days-devops
```

### Step 3 — Open the Resource Group

Click:

```text
rg-100days-devops
```

Explore:

- Overview
- Activity log
- Access control (IAM)
- Deployments
- Resources
- Tags
- Automation
- Locks

The exact menu can vary slightly as Azure Portal changes over time.

---

## 11. Add Tags Through Azure Portal

Open:

```text
rg-100days-devops
```

Find:

```text
Tags
```

Add:

```text
Environment = Dev
Project = 100DaysDevOps
Purpose = Learning
```

Then save.

---

# 💻 12. Azure CLI

First check Azure CLI:

```bash
az version
```

---

## 13. Login

```bash
az login
```

A browser window may open.

Then check your account:

```bash
az account show --output table
```

---

## 14. List Subscriptions

```bash
az account list --output table
```

Example:

```text
Name                  CloudName    SubscriptionId
--------------------  -----------  ----------------
Azure Subscription    AzureCloud   xxxxxxxx-xxxx
```

---

## 15. Select a Subscription

If you have multiple subscriptions:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

Verify:

```bash
az account show --output table
```

---

## 16. List Resource Groups

```bash
az group list --output table
```

Example:

```text
Name                 Location
-------------------  --------
rg-100days-devops    eastus
```

---

## 17. Show Resource Group Details

```bash
az group show   --name rg-100days-devops   --output table
```

For complete JSON:

```bash
az group show   --name rg-100days-devops
```

---

## 18. Check Whether a Resource Group Exists

```bash
az group exists   --name rg-100days-devops
```

Possible output:

```text
true
```

or:

```text
false
```

This is useful in automation scripts.

---

## 19. Update Resource Group Tags

```bash
az group update   --name rg-100days-devops   --tags Environment=Dev Project=100DaysDevOps Purpose=Learning
```

Verify:

```bash
az group show   --name rg-100days-devops   --query tags
```

Expected output:

```json
{
  "Environment": "Dev",
  "Project": "100DaysDevOps",
  "Purpose": "Learning"
}
```

---

## 20. List Resources Inside a Resource Group

```bash
az resource list   --resource-group rg-100days-devops   --output table
```

If the Resource Group contains no resources, an empty result is fine.

---

## 21. Create a Resource Group Using CLI

If it doesn't already exist:

```bash
az group create   --name rg-100days-devops   --location eastus
```

Verify:

```bash
az group show   --name rg-100days-devops   --output table
```

---

# 🔄 22. Important DevOps Concept — Desired State

Eventually, we don't want to manually create everything through the Azure Portal.

Instead:

```text
Git Repository
      |
      v
Infrastructure Code
      |
      v
CI/CD Pipeline
      |
      v
Azure
      |
      +--- Resource Group
      +--- Network
      +--- VM
      +--- Storage
```

This is where tools such as:

- Terraform
- Bicep
- ARM templates
- Azure CLI
- CI/CD pipelines

become important.

You'll learn Terraform later in this 100-day journey.

---

# ♻️ 23. Resource Management Lifecycle

A typical Azure resource lifecycle looks like:

```text
       Create
          |
          v
      Configure
          |
          v
       Deploy
          |
          v
      Monitor
          |
          v
       Update
          |
          v
      Maintain
          |
          v
       Delete
```

DevOps engineers need to understand the entire lifecycle.

---

# 🧪 Day 42 Hands-On Challenge

Try completing this without copying every command from above.

### Task 1 — Find Your Azure Subscription

```bash
az account show --output table
```

### Task 2 — List All Resource Groups

```bash
az group list --output table
```

### Task 3 — Create the Resource Group

Create:

```text
rg-100days-devops
```

if it doesn't already exist.

```bash
az group create   --name rg-100days-devops   --location eastus
```

### Task 4 — Add Tags

```bash
az group update   --name rg-100days-devops   --tags Environment=Dev Project=100DaysDevOps Purpose=Learning
```

### Task 5 — Verify Tags

```bash
az group show   --name rg-100days-devops   --query tags
```

### Task 6 — List Resources

```bash
az resource list   --resource-group rg-100days-devops   --output table
```

---

# 🔍 Expected Result

You should have:

```text
Azure Subscription
        |
        v
rg-100days-devops
        |
        +--- Environment = Dev
        +--- Project = 100DaysDevOps
        +--- Purpose = Learning
```

If you created resources during previous Azure exercises, they may also appear inside the Resource Group.

---

# ⚠️ Important Cost Warning

For today's lab, **you do not need to create a VM or other potentially billable resource**.

Creating a Resource Group and managing its metadata is enough to practice today's concepts.

If you later create resources such as:

```text
VM
Disk
Public IP
Load Balancer
Database
```

check the pricing and free-tier eligibility first.

> **Deleting a Resource Group deletes the resources inside it.**

---

# 🛠️ Troubleshooting

## Problem 1 — `az: command not found`

Check:

```bash
az version
```

If Azure CLI isn't installed, install it using Microsoft's current installation instructions.

## Problem 2 — Not Logged In

```bash
az login
```

Then:

```bash
az account show
```

## Problem 3 — Wrong Subscription

```bash
az account list --output table
```

Then:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

## Problem 4 — Resource Group Doesn't Exist

```bash
az group exists   --name rg-100days-devops
```

If it returns:

```text
false
```

create it:

```bash
az group create   --name rg-100days-devops   --location eastus
```

## Problem 5 — Authorization Error

You may not have sufficient permissions for the subscription/resource group.

Check:

```bash
az account show
```

For organizational Azure accounts, access may need to be granted by an administrator.

---

# 📸 Screenshot for LinkedIn

Capture these:

### Screenshot 1 — Azure Portal

```text
Resource Groups
       ↓
rg-100days-devops
```

### Screenshot 2 — Resource Group List

```bash
az group list --output table
```

### Screenshot 3 — Tags

```bash
az group show   --name rg-100days-devops   --query tags
```

A simple visual:

```text
Azure Portal          Terminal
     |                   |
     v                   v
Resource Group      Azure CLI
     |                   |
     +--------+----------+
              |
              v
       Azure Management
```

---

# 📌 Key Takeaways

Today you learned:

- Azure Subscription
- Azure Resource Group
- Azure Resources
- Relationship between them
- Resource Group locations
- Resource lifecycle
- Resource tagging
- Naming conventions
- Azure Portal management
- Azure CLI management
- Resource listing
- Subscription selection
- Why Resource Groups matter in DevOps

> **A Resource Group is not the resource itself. It is a logical management container for Azure resources.**

---

# 💼 Real-World DevOps Example

Imagine a company running an e-commerce application.

```text
Production Subscription
          |
          v
   rg-ecommerce-prod
          |
    +-----+-----+-----+------+
    |           |     |      |
    v           v     v      v
   VM          VNet   NSG   Storage
```

Tags:

```text
Environment = Production
Application = Ecommerce
Owner       = DevOps
CostCenter  = 1001
```

This makes the environment easier to manage, automate, monitor and audit.

---

# 💻 GitHub Update

After completing today's lab:

```bash
git status
```

```bash
git add .
```

```bash
git commit -m "Day 42 - Azure Resource Groups and Resource Management"
```

```bash
git push origin main
```

Progress:

```text
Day 41 ✅
Day 42 ✅
Day 43 ⏭️
```

---

# 🔥 Day 42 LinkedIn Post

Today I went one level deeper into Azure as part of my **100 Days of DevOps** journey.

Yesterday I started my Azure journey.

Today I focused on an important question:

**How do we organize and manage Azure resources?**

That's where **Subscriptions and Resource Groups** come in.

I learned the relationship between:

```text
Azure Subscription
        |
        v
Resource Group
        |
   +----+----+
   |    |    |
   v    v    v
  VM   VNet Storage
```

A simple way I understand it now:

**Subscription → management/billing boundary**

**Resource Group → logical container**

**Resource → actual Azure service**

Today I practiced:

✅ Listing Azure subscriptions

✅ Selecting a subscription using Azure CLI

✅ Creating and inspecting Resource Groups

✅ Listing resources inside a Resource Group

✅ Adding tags

✅ Checking Resource Group properties

✅ Understanding Resource Group lifecycle

One concept that stood out:

A Resource Group can contain resources deployed in different Azure regions, while the Resource Group itself has a location for its metadata.

I also learned why tags are important:

```text
Environment = Dev
Project     = 100DaysDevOps
Purpose     = Learning
```

These small pieces of metadata become useful for organization, automation and cost management.

I practiced everything using Azure CLI:

```bash
az group list --output table
```

and:

```bash
az group show   --name rg-100days-devops   --query tags
```

Another important lesson:

**Deleting a Resource Group can delete the resources inside it.**

So cloud resource management isn't just about creating things.

It's also about understanding their **lifecycle, ownership, organization and cleanup**.

Day 42 complete. 🚀

Next:

**Day 43 — Creating and Connecting to Azure Virtual Machines**

#100DaysOfDevOps #DevOps #Azure #MicrosoftAzure #CloudComputing #AzureCLI #CloudEngineer #DevOpsJourney #LearningInPublic #AzureDevOps

---

# 🔜 Day 43 Preview

## Creating and Connecting to Azure Virtual Machines

We'll create an Azure VM and connect to it using **SSH**, bringing together the Azure, Linux and SSH concepts learned earlier in the journey.

---

## 📊 Progress

**42/100 Days Completed**

```text
DevOps Fundamentals  ████████████████████  5/5
Linux                ████████████████████ 10/10
Networking           ████████████████████  5/5
Git & GitHub         ████████████████████ 10/10
Docker               ████████████████████ 10/10
Azure Cloud          ████░░░░░░░░░░░░░░░░  2/10
CI/CD                ░░░░░░░░░░░░░░░░░░░░  0/10
Kubernetes           ░░░░░░░░░░░░░░░░░░░░  0/10
Terraform            ░░░░░░░░░░░░░░░░░░░░  0/10
Ansible              ░░░░░░░░░░░░░░░░░░░░  0/5
Monitoring           ░░░░░░░░░░░░░░░░░░░░  0/5
DevSecOps            ░░░░░░░░░░░░░░░░░░░░  0/5
Final Project        ░░░░░░░░░░░░░░░░░░░░  0/5

Total                ████████░░░░░░░░░░░░ 42/100
```
