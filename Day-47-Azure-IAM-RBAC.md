# Day 47/100 — Azure Identity and Access Management (IAM) and RBAC

## 🎯 Objective

Learn how Azure manages identities and permissions using Microsoft Entra ID and Azure Role-Based Access Control (RBAC).

By the end of this lesson, you will understand Azure IAM, explore role assignments using Azure CLI, and apply the principle of least privilege.

---

## 1. What Is Azure IAM?

Identity and Access Management (IAM) determines:

- **Who** can access a resource.
- **What** actions they can perform.
- **Where** they can perform those actions.

For example, a developer might be allowed to view an Azure virtual machine but not delete it.

### Microsoft Entra ID vs Azure RBAC

| Component | Purpose |
|---|---|
| Microsoft Entra ID | Manages identities, users, groups, and authentication |
| Azure RBAC | Controls which actions identities can perform on Azure resources |
| Managed Identity | Allows supported Azure resources to authenticate without storing credentials in application code |

### Authentication vs Authorization

- **Authentication:** Who are you?
- **Authorization:** What are you allowed to do?

Example: Signing in to Azure is authentication. Having permission to restart a virtual machine is authorization.

---

## 2. Understanding Azure RBAC

Azure RBAC grants permissions through a role assignment.

A role assignment has three main components:

1. **Security principal:** The user, group, service principal, or managed identity receiving access.
2. **Role definition:** The permissions being granted.
3. **Scope:** The resources where those permissions apply.

### Common Azure Built-in Roles

| Role | Purpose |
|---|---|
| Owner | Manage resources and assign Azure RBAC roles |
| Contributor | Create and manage resources but cannot grant Azure RBAC access |
| Reader | View resources without making changes |
| User Access Administrator | Manage access to Azure resources |
| Virtual Machine Contributor | Manage virtual machines within the role's permitted scope |
| Storage Blob Data Reader | Read blob data |
| Storage Blob Data Contributor | Read, write, and delete blob data |

**Important:** Azure resource-management permissions and data permissions are different. For example, Contributor does not automatically grant permission to read the contents of a storage blob.

---

## 3. Understanding Azure RBAC Scope

Azure RBAC assignments can be made at different levels.

```text
Management Group
      |
      v
Subscription
      |
      v
Resource Group
      |
      v
Individual Resource
```

A role assignment at a parent scope can be inherited by child resources.

**Best practice:** Grant only the permissions required, at the narrowest practical scope.

For example, if a developer only needs to view resources in one resource group, assign Reader at that resource-group scope instead of across the entire subscription.

---

## 4. Hands-On Lab: Explore Azure IAM Using CLI

### Prerequisites

- An Azure subscription.
- Azure CLI installed.
- Permission to view the subscription and role assignments.
- An existing resource group, such as `rg-100days-devops`.

### Step 1: Sign In to Azure

```bash
az login
```

Verify your active subscription:

```bash
az account show --output table
```

If you have multiple subscriptions, select the correct one:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

### Step 2: Inspect Your Resource Group

Set the resource group variable:

```bash
RESOURCE_GROUP="rg-100days-devops"
```

Check whether it exists:

```bash
az group show \
  --name "$RESOURCE_GROUP" \
  --query "{Name:name,Location:location,ID:id}" \
  --output table
```

If the resource group does not exist, replace its name with an existing resource group you are authorized to inspect.

### Step 3: Find the Resource Group Scope

```bash
SUBSCRIPTION_ID=$(az account show --query id --output tsv)

RG_SCOPE="/subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}"

echo "$RG_SCOPE"
```

The output should resemble:

```text
/subscriptions/<subscription-id>/resourceGroups/rg-100days-devops
```

### Step 4: List Role Assignments

View role assignments at the resource-group scope:

```bash
az role assignment list \
  --scope "$RG_SCOPE" \
  --all \
  --output table
```

To include assignments inherited from parent scopes:

```bash
az role assignment list \
  --scope "$RG_SCOPE" \
  --include-inherited \
  --all \
  --query "[].{Principal:principalName,Role:roleDefinitionName,Scope:scope}" \
  --output table
```

Review the results and identify:

- Who has access?
- Which roles are assigned?
- At which scope are the roles assigned?

The output depends on your subscription and permissions.

### Step 5: Inspect Your Own Role Assignments

Retrieve the object ID of the signed-in user:

```bash
USER_OBJECT_ID=$(az ad signed-in-user show --query id --output tsv)
```

List role assignments for that user:

```bash
az role assignment list \
  --assignee "$USER_OBJECT_ID" \
  --all \
  --output table
```

If the command fails, your account might lack permission to read directory information. Ask an administrator to verify the required permissions.

### Step 6: Inspect Built-in Role Definitions

Review the Reader role:

```bash
az role definition list \
  --name "Reader" \
  --query "[].{Role:roleName,Description:description}" \
  --output table
```

Review the Contributor role:

```bash
az role definition list \
  --name "Contributor" \
  --query "[].{Role:roleName,Description:description}" \
  --output table
```

Compare the roles and understand why read-only access is safer for users who only need to inspect resources.

---

## 5. Optional Lab: Assign and Remove a Reader Role

**Caution:** This exercise changes permissions. Perform it only in an environment where you are authorized to administer access. Use a test identity, not a production user.

Set the object ID of an authorized test user:

```bash
TEST_USER_OBJECT_ID="<TEST_USER_OBJECT_ID>"
```

Assign Reader at the resource-group scope:

```bash
az role assignment create \
  --assignee "$TEST_USER_OBJECT_ID" \
  --role "Reader" \
  --scope "$RG_SCOPE"
```

Verify the assignment:

```bash
az role assignment list \
  --assignee "$TEST_USER_OBJECT_ID" \
  --scope "$RG_SCOPE" \
  --all \
  --output table
```

Remove the assignment after the exercise:

```bash
az role assignment delete \
  --assignee "$TEST_USER_OBJECT_ID" \
  --role "Reader" \
  --scope "$RG_SCOPE"
```

Role changes can take time to become effective. Do not remove access that another person or workload still needs.

---

## 6. What Is a Managed Identity?

A managed identity allows a supported Azure service to authenticate to other services without storing passwords or client secrets in application code.

Example:

```text
Azure Virtual Machine
        |
        | Managed Identity
        v
Microsoft Entra ID
        |
        | Access Token
        v
Azure Storage
```

A managed identity still needs the appropriate permissions on the target resource. Having an identity alone does not grant access.

**Production best practice:** Prefer managed identities over hardcoded credentials whenever supported.

---

## 7. Common Problems and Troubleshooting

### AuthorizationFailed

**Possible cause:** Your identity lacks permission to perform the requested operation.

**What to check:**
- Your active subscription.
- Your role assignment.
- The scope where the role was assigned.
- Whether the operation requires additional permissions.

### Insufficient Directory Permissions

**Possible cause:** Your identity cannot read user or directory information.

**Solution:** Ask an administrator to verify the required Microsoft Entra ID permissions.

### Storage Access Denied

**Possible cause:** You have permission to manage the storage account but lack the necessary blob data permissions.

**Solution:** Check whether your identity has an appropriate role, such as Storage Blob Data Reader or Storage Blob Data Contributor, at the correct scope.

### Role Assignment Not Taking Effect

**Possible cause:** Permission changes have not propagated, or the assignment was made at a different scope.

**Solution:** Verify the assignment and allow time for propagation.

---

## 8. Production Best Practices

- Follow the principle of least privilege.
- Prefer Microsoft Entra ID groups for managing team access.
- Assign roles at the narrowest practical scope.
- Avoid unnecessary Owner or broad Contributor permissions.
- Use managed identities instead of hardcoded credentials where possible.
- Review role assignments regularly.
- Remove access when it is no longer required.
- Use separate identities for applications and human users.
- Follow organizational approval and change-management processes.

---

## 9. Hands-On Challenge

Answer these questions after completing the lab:

1. What is the difference between Microsoft Entra ID and Azure RBAC?
2. What are the three main parts of a role assignment?
3. What is the difference between Owner and Contributor?
4. Why is assigning Reader at the resource-group level often safer than assigning a broad role at subscription level?
5. Why does managing a storage account not automatically mean you can read its blob contents?
6. What problem does a managed identity solve?

Write your answers in your notes or add them to your GitHub repository.

---

## 10. Screenshots to Capture

For your learning documentation or LinkedIn post, capture:

- Azure CLI output showing the active subscription.
- The resource group's details.
- Role assignments at the resource-group scope.
- Reader and Contributor role definitions.
- The Azure portal's Access control (IAM) page.

**Security reminder:** Hide subscription IDs, tenant IDs, email addresses, object IDs, and other sensitive details before publishing screenshots.

---

## 11. Key Takeaways

Today, I learned:

- Microsoft Entra ID manages identities and authentication.
- Azure RBAC controls authorization to Azure resources.
- Role assignments combine a principal, role definition, and scope.
- Permissions can be inherited from parent scopes.
- Management-plane permissions differ from data-plane permissions.
- Managed identities help reduce credential-management risks.
- Least privilege is a core cloud-security principle.

---

## 12. LinkedIn Post — Day 47/100

**100 Days of DevOps — Day 47/100 🚀**

🔐 **Azure IAM and RBAC: Who should have access to your cloud resources?**

Creating Azure resources is only part of cloud engineering. Managing who can access those resources is equally important.

Today, I explored how Azure Identity and Access Management works.

**What I learned:**

✅ Microsoft Entra ID manages identities and authentication.

✅ Azure RBAC controls what users and applications can do.

✅ Roles such as Reader, Contributor, and Owner provide different levels of access.

✅ Scope determines where permissions apply.

✅ Managed identities help applications authenticate without hardcoded credentials.

**Hands-on practice:**

- Explored Azure role assignments using Azure CLI.
- Inspected built-in role definitions.
- Reviewed resource-group scope and inherited permissions.
- Learned the difference between Azure resource permissions and blob data permissions.

💡 **My key takeaway:** Just because someone needs access to Azure does not mean they need full administrative permissions.

Start with the minimum permissions required and expand only when necessary.

**Question:** How does your team manage Azure permissions—individual assignments, groups, or managed identities?

Follow along as I continue my 100 Days of DevOps journey!

#100DaysOfDevOps #Azure #AzureRBAC #MicrosoftEntraID #CloudSecurity #DevOps #IAM #LearningInPublic

---

## 13. Upload to GitHub

Save the file as:

`Day-47-Azure-IAM-RBAC.md`

Open Terminal in your local repository and run:

```bash
git add Day-47-Azure-IAM-RBAC.md
git commit -m "Add Day 47 Azure IAM and RBAC"
git push origin main
```

Check your GitHub repository to confirm that the file was uploaded successfully.

---

## Coming Next: Day 48/100

**Deploying an Application on an Azure Virtual Machine**

We will learn how to prepare an Azure VM, install a web server, deploy a sample application, configure network access, and verify that the application is reachable.

Keep learning, keep building, and keep documenting your DevOps journey! 🚀
