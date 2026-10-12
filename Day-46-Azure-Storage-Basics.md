# 100 Days of DevOps — Day 46/100
## Azure Storage Basics: Storage Accounts, Containers and Blobs

### Today's goals
- Understand storage accounts, containers, and blobs.
- Create an Azure Storage account using Azure CLI.
- Upload, list, and download files with Azure Blob Storage.
- Apply basic security and cost-management practices.

## 1. Why cloud storage?
Applications need to store user uploads, logs, backups, and build artifacts. Keeping everything on a VM's local disk can complicate scaling and recovery. Azure Storage provides cloud storage services separately from compute resources.

## 2. Key concepts
- **Storage account:** Top-level Azure resource that provides access to storage services and controls settings such as region, redundancy, and security.
- **Container:** Logical grouping of blobs inside Blob Storage.
- **Blob:** An individual object, such as an image, log, backup, or build artifact.

```text
Azure Subscription
└── Resource Group
    └── Storage Account
        └── Blob Container
            ├── application.log
            ├── backup.tar.gz
            └── release.zip
```

A resource group organizes Azure resources; a blob container organizes blobs inside a storage account.

## 3. Common Azure storage services

| Service | Typical use |
|---|---|
| Blob Storage | Images, logs, backups, artifacts, unstructured data |
| Azure Files | Managed file shares using supported protocols such as SMB or NFS |
| Queue Storage | Messages for asynchronous processing |
| Table Storage | NoSQL key/attribute-based data |
| Managed disks | Persistent disks attached to Azure VMs |

This lab focuses on **Azure Blob Storage**.

## 4. Hands-on lab

### Step 1 — Sign in and select a subscription

```bash
az login
az account show --output table
az account set --subscription "<SUBSCRIPTION_ID>"
```

Run the subscription command only if needed. Verify the active subscription before creating resources.

### Step 2 — Define lab variables

```bash
RESOURCE_GROUP="rg-100days-devops"
LOCATION="eastus"
CONTAINER_NAME="devops-artifacts"
az group create --name "$RESOURCE_GROUP" --location "$LOCATION"
```

If your existing resource group is in another region, update `LOCATION`. Confirm this is the intended lab resource group.

### Step 3 — Generate a unique storage account name

Storage account names must be globally unique, lowercase, 3–24 characters, and contain only letters and numbers.

```bash
SUFFIX=$(date +%s | tail -c 7)
STORAGE_ACCOUNT="stdevops${SUFFIX}"
echo "$STORAGE_ACCOUNT"
```

If Azure says the name is taken, generate a new suffix and retry.

### Step 4 — Create the storage account

```bash
az storage account create \
  --name "$STORAGE_ACCOUNT" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --sku Standard_LRS \
  --kind StorageV2 \
  --allow-blob-public-access false
```

- `Standard_LRS`: standard locally redundant storage.
- `StorageV2`: general-purpose version 2 account.
- `--allow-blob-public-access false`: disables anonymous public access to blob containers.

Verify the account:

```bash
az storage account show \
  --name "$STORAGE_ACCOUNT" \
  --resource-group "$RESOURCE_GROUP" \
  --query "{name:name,location:location,sku:sku.name,kind:kind}" \
  --output table
```

Expected: account name, region, SKU, and account kind.

### Step 5 — Create sample files

```bash
mkdir -p ~/day46-storage-lab
cd ~/day46-storage-lab

echo "Day 46: Azure Blob Storage hands-on lab" > devops-notes.txt
echo "Environment=lab" > app-config.txt
ls -l
cat devops-notes.txt
```

### Step 6 — Create a private container

```bash
az storage container create \
  --name "$CONTAINER_NAME" \
  --account-name "$STORAGE_ACCOUNT" \
  --auth-mode login
```

The response should indicate that the container was created.

**Permissions:** Your signed-in identity needs a suitable data-plane role, such as **Storage Blob Data Contributor**, to create containers and upload/download blobs using Microsoft Entra authentication. Permission to create the storage account does not automatically grant blob-data access.

### Step 7 — Upload files

```bash
az storage blob upload \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "devops-notes.txt" \
  --file "devops-notes.txt" \
  --auth-mode login

az storage blob upload \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "app-config.txt" \
  --file "app-config.txt" \
  --auth-mode login
```

The CLI should report successful uploads.

### Step 8 — List blobs

```bash
az storage blob list \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --auth-mode login \
  --query "[].{Name:name,Size:properties.contentLength}" \
  --output table
```

Expected: a table showing `devops-notes.txt` and `app-config.txt`, with their sizes.

### Step 9 — Download and verify a blob

```bash
mkdir -p downloaded

az storage blob download \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "devops-notes.txt" \
  --file "downloaded/devops-notes.txt" \
  --auth-mode login

cat downloaded/devops-notes.txt
```

Expected output:

```text
Day 46: Azure Blob Storage hands-on lab
```

You have created a storage account and container, uploaded blobs, listed them, and downloaded a file.

## 5. Troubleshooting

### Storage account name unavailable
Names are globally unique. Generate a different suffix and retry:

```bash
SUFFIX=$(date +%s | tail -c 7)
STORAGE_ACCOUNT="stdevops${SUFFIX}"
echo "$STORAGE_ACCOUNT"
```

### Authorization error
Your identity may lack blob-data permissions. Ask an Azure administrator to assign **Storage Blob Data Contributor** at an appropriate scope. Wait for the role assignment to propagate, then retry.

### Storage account not found
Check the active subscription and account list:

```bash
az account show --output table
az storage account list --resource-group "$RESOURCE_GROUP" --output table
```

### File not found
Check the working directory and file names:

```bash
pwd
ls -l
```

The `--file` path must point to a local file that exists.

## 6. DevOps use cases
- **Backups:** Store recovery data separately from compute resources.
- **CI/CD artifacts:** Store build outputs, packages, and release files.
- **Logs:** Store diagnostic exports for later investigation.
- **Application data:** Store images and documents independently of a VM.

## 7. Security and cost best practices
- Keep anonymous blob access disabled unless a reviewed use case requires it.
- Prefer Microsoft Entra ID and least-privilege role assignments over embedding account keys in scripts.
- Never commit storage keys, SAS tokens, or credentials to GitHub.
- Choose redundancy based on availability and recovery requirements.
- Consider lifecycle management for older data.
- Monitor storage capacity, transactions, and data-transfer costs. Storage may incur charges while idle.

### Optional cleanup
This permanently deletes the storage account and its data. Verify the name first, and do not delete the whole resource group if it contains resources from earlier lessons.

```bash
az storage account delete \
  --name "$STORAGE_ACCOUNT" \
  --resource-group "$RESOURCE_GROUP" \
  --yes
```

## 8. Day 46 challenge
- [ ] Create a storage account and private blob container.
- [ ] Upload three files.
- [ ] List blobs and inspect their sizes.
- [ ] Download one blob and verify its contents.
- [ ] Explain the difference between a storage account, container, and blob.
- [ ] Explain why management permissions do not automatically grant blob-data access.

**Bonus:** Research how an Azure DevOps pipeline or GitHub Actions workflow can upload an artifact to Blob Storage without hardcoding credentials.

## 9. LinkedIn screenshot checklist
1. Storage account details showing account name, region, and SKU.
2. The private `devops-artifacts` container.
3. Output of `az storage blob list`.
4. Downloaded file contents displayed with `cat`.

Hide subscription IDs and sensitive details before posting screenshots.

## 10. LinkedIn post — Day 46/100

**100 Days of DevOps — Day 46/100 🚀**

☁️ **Azure Storage Basics: Where should our application data live?**

Imagine deploying an application on an Azure Virtual Machine. Where would you store its backups, uploaded files, logs, and build artifacts?

Keeping everything on a VM's local disk can make scaling, recovery, and maintenance more complicated.

Today, I explored Azure Storage and learned how cloud storage can separate application data from compute resources.

💡 **What I learned:**

🔹 **Storage Account:** The Azure resource that provides access to storage services.

🔹 **Blob Container:** A logical grouping for blobs.

🔹 **Blob Storage:** A service for storing files such as logs, images, backups, and deployment artifacts.

🔹 **Access Control:** Private storage and least-privilege permissions help protect data.

🛠️ **Hands-on practice:**

✅ Created an Azure Storage account using Azure CLI.  
✅ Created a private blob container.  
✅ Uploaded files to Azure Blob Storage.  
✅ Listed blobs and inspected their sizes.  
✅ Downloaded a file and verified its contents.

📌 **My biggest takeaway:**

Cloud storage is not just about saving files. It is about making application data easier to manage, access securely, and use across different environments.

💬 **Question for the DevOps community:**

How do you manage build artifacts and backups in your projects — Azure Blob Storage, Amazon S3, or another solution?

I'd love to learn from your experience!

One more practical step in my journey to becoming a better DevOps engineer.

#100DaysOfDevOps #Azure #AzureStorage #CloudComputing #DevOps #CloudSecurity #CICD #LearningInPublic

---

## Coming up: Day 47/100

**Azure Identity and Access Management (IAM)** — learn about Microsoft Entra ID, Azure RBAC, roles, scopes, and granting users and applications only the permissions they need.

**Progress: Day 46/100**
