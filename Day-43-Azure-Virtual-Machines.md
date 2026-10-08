# Day 43/100 — Azure Virtual Machines — Creating and Connecting to a Linux VM

## 🎯 Day 43 Goal

Today we move from managing Azure resources to actually running a server in the cloud.

By the end of this lab, you will be able to:

- Understand Azure Virtual Machines
- Understand the main components of an Azure VM
- Create an Ubuntu Linux VM using Azure CLI
- Generate and use SSH keys
- Find the VM's public IP address
- Connect to the VM using SSH
- Verify the Linux environment
- Install and verify Nginx
- Check VM status
- Start and deallocate a VM
- Troubleshoot common VM/SSH problems
- Clean up Azure resources safely

---

## 1. What is an Azure Virtual Machine?

An **Azure Virtual Machine (VM)** is a virtual server running inside Microsoft Azure.

Instead of buying a physical server, we can create a virtual server in Azure and use it for:

- Web servers
- Application servers
- Development environments
- Testing
- Databases
- CI/CD workloads
- DevOps infrastructure

```text
                 Azure
                   |
                   v
            Virtual Machine
                   |
       +-----------+-----------+
       |           |           |
      CPU         RAM         Disk
       |
       v
    Ubuntu Linux
```

Think of an Azure VM as a computer that exists inside Microsoft's cloud infrastructure.

---

## 2. Azure VM Architecture

A VM is connected to several other Azure resources.

```text
                         Internet
                            |
                            v
                       Public IP
                            |
                            v
                  Network Interface
                            |
                            v
                      Azure VM
                   +-------------+
                   |   Ubuntu    |
                   |   Linux     |
                   +-------------+
                     |    |    |
                     v    v    v
                    CPU  RAM  Disk
```

A typical Azure VM environment contains:

```text
VM
 |
 +--- OS Disk
 |
 +--- Network Interface
 |
 +--- Virtual Network
 |
 +--- Subnet
 |
 +--- Public IP
 |
 +--- Network Security Group
```

Understanding these relationships is important because DevOps engineers usually manage infrastructure as a collection of interconnected resources rather than treating a VM as an isolated server.

---

## 3. Linux vs Windows Azure VMs

Azure supports multiple operating systems, including:

- Ubuntu
- Debian
- Red Hat Enterprise Linux
- SUSE Linux
- Windows Server

For DevOps learning, Linux is especially important because many DevOps tools and production workloads run on Linux.

In this lab, we will use:

**Ubuntu 22.04**

---

## 4. What is SSH?

**SSH (Secure Shell)** allows you to securely connect to a remote Linux server.

```text
Your Mac
   |
   | SSH
   v
Azure Linux VM
```

Example:

```bash
ssh azureuser@<VM-IP>
```

Instead of sitting physically in front of the server, you can manage it remotely from your terminal.

---

## 5. SSH Key Authentication

Azure can generate an SSH key pair for you when creating the VM.

```text
SSH Key Pair
     |
     +----------------+
     |                |
     v                v
Private Key       Public Key
     |                |
Your Mac          Azure VM
     |                |
 KEEP SECRET       Server
```

### Important

Never share your private SSH key.

The private key stays on your computer.

The public key is placed on the server.

---

## 6. Azure VM Architecture for This Lab

```text
                 Azure Subscription
                        |
                        v
               rg-100days-devops
                        |
                        v
                 Azure Virtual VM
                        |
                 Ubuntu Linux
                        |
                    Public IP
                        |
                        v
                    SSH :22
                        |
                        v
                   Your Mac
```

---

## 7. Prerequisites

You need:

- Azure account
- Azure subscription
- Azure CLI
- Mac/Linux/Windows terminal
- Internet connection
- SSH client

Check Azure CLI:

```bash
az version
```

Login:

```bash
az login
```

Check your subscription:

```bash
az account show --output table
```

List subscriptions:

```bash
az account list --output table
```

Select the correct subscription if needed:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

Verify:

```bash
az account show --output table
```

---

## 8. Set Lab Variables

```bash
RESOURCE_GROUP="rg-100days-devops"
VM_NAME="vm-100days-devops"
LOCATION="eastus"
ADMIN_USER="azureuser"
```

Verify:

```bash
echo $RESOURCE_GROUP
echo $VM_NAME
echo $LOCATION
echo $ADMIN_USER
```

Expected output:

```text
rg-100days-devops
vm-100days-devops
eastus
azureuser
```

---

## 9. Check or Create the Resource Group

Check whether the resource group exists:

```bash
az group exists   --name "$RESOURCE_GROUP"
```

If it doesn't exist, create it:

```bash
az group create   --name "$RESOURCE_GROUP"   --location "$LOCATION"
```

Verify:

```bash
az group show   --name "$RESOURCE_GROUP"   --output table
```

---

## 10. Create the Azure Linux VM

```bash
az vm create   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"   --image Ubuntu2204   --admin-username "$ADMIN_USER"   --generate-ssh-keys
```

### What does this command do?

| Option | Meaning |
|---|---|
| `az vm create` | Creates an Azure VM |
| `--resource-group` | Resource group where the VM is created |
| `--name` | Name of the VM |
| `--image Ubuntu2204` | Uses Ubuntu 22.04 |
| `--admin-username` | Creates the Linux admin user |
| `--generate-ssh-keys` | Generates SSH keys if needed |

Azure will create the VM and supporting networking resources.

This may take a few minutes.

---

## 11. Check Your SSH Keys

```bash
ls -la ~/.ssh
```

You may see:

```text
id_rsa
id_rsa.pub
```

or:

```text
id_ed25519
id_ed25519.pub
```

The file without `.pub` is the private key.

The `.pub` file is the public key.

**Never share your private key.**

---

## 12. Find the VM Public IP

```bash
az vm show   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"   --show-details   --query publicIps   --output tsv
```

Example:

```text
20.XX.XX.XX
```

The actual IP will be different for your VM.

---

## 13. Store the Public IP in a Variable

```bash
VM_IP=$(az vm show   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"   --show-details   --query publicIps   --output tsv)
```

Check it:

```bash
echo $VM_IP
```

---

## 14. Connect to the VM Using SSH

```bash
ssh "$ADMIN_USER@$VM_IP"
```

You should see a prompt similar to:

```text
azureuser@vm-100days-devops:~$
```

Congratulations! 🎉

You are now connected to a Linux server running in Microsoft Azure.

---

## 15. Verify the Linux Server

Once connected:

```bash
hostname
```

```bash
cat /etc/os-release
```

```bash
uname -a
```

```bash
ip addr
```

```bash
df -h
```

```bash
free -h
```

```bash
echo "Hello from Azure VM"
```

---

## 16. Install Nginx

Update packages:

```bash
sudo apt update
```

Install Nginx:

```bash
sudo apt install nginx -y
```

Check the service:

```bash
sudo systemctl status nginx
```

You should see that Nginx is running.

---

## 17. Test Nginx from Inside the VM

```bash
curl http://localhost
```

You should receive HTML output from Nginx.

```text
Azure VM
   |
   v
Ubuntu
   |
   v
Nginx
   |
   v
HTTP Response
```

---

## 18. Exit the SSH Session

```bash
exit
```

You should return to your local Mac terminal.

---

## 19. Check VM Status

```bash
az vm get-instance-view   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"   --query instanceView.statuses   --output table
```

You should see status information such as:

```text
PowerState/running
```

---

## 20. Start the VM

If the VM is stopped/deallocated:

```bash
az vm start   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"
```

---

## 21. Deallocate the VM

For a temporary lab:

```bash
az vm deallocate   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"
```

Deallocation releases the compute allocation, although disks and other resources may remain.

Always understand the cost implications of the resources you create.

---

## 22. List Azure VMs

```bash
az vm list --output table
```

---

## 23. Delete the VM

If you only want to remove the VM:

```bash
az vm delete   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"
```

---

## 24. Delete the Entire Lab Resource Group

If this resource group contains only your lab resources:

```bash
az group delete   --name "$RESOURCE_GROUP"
```

### ⚠️ Important

Do **not** run this command if the resource group contains production or important resources.

---

## 25. Troubleshooting

### Problem 1 — `az: command not found`

Check:

```bash
az version
```

Install Azure CLI if necessary.

### Problem 2 — Not logged in

```bash
az login
```

Then:

```bash
az account show --output table
```

### Problem 3 — Wrong Azure subscription

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

### Problem 4 — SSH connection refused

Check VM status:

```bash
az vm get-instance-view   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"   --query instanceView.statuses   --output table
```

Start the VM if required:

```bash
az vm start   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"
```

### Problem 5 — Wrong IP address

```bash
az vm show   --resource-group "$RESOURCE_GROUP"   --name "$VM_NAME"   --show-details   --query publicIps   --output tsv
```

Then reconnect using the current IP.

### Problem 6 — SSH permission denied

Check:

- Correct username
- Correct private key
- VM was created with the expected SSH key
- Private key permissions

For example:

```bash
chmod 600 ~/.ssh/id_rsa
```

Use your actual key filename if different.

### Problem 7 — VM doesn't exist

```bash
az vm list --output table
```

Check the resource group:

```bash
az group show   --name "$RESOURCE_GROUP"   --output table
```

---

## 26. Day 43 Hands-On Challenge

Create:

```text
VM Name: vm-100days-devops
OS: Ubuntu
Admin User: azureuser
Resource Group: rg-100days-devops
```

Then:

1. Generate an SSH key.
2. Create the VM.
3. Find the public IP.
4. Connect using SSH.
5. Run:
   - `hostname`
   - `cat /etc/os-release`
   - `df -h`
   - `free -h`
6. Install Nginx.
7. Check Nginx status.
8. Run:
   ```bash
   curl http://localhost
   ```
9. Exit the VM.
10. Check VM status from Azure CLI.
11. Deallocate the VM when finished.

---

## 27. Expected Result

```text
                    Azure
                      |
                      v
             Resource Group
             rg-100days-devops
                      |
                      v
               Azure VM
             vm-100days-devops
                      |
                      v
                Ubuntu Linux
                      |
                      v
                    Nginx
                      ^
                      |
                    SSH
                      |
                   Your Mac
```

---

## 28. Screenshots to Capture for LinkedIn

### Screenshot 1 — Azure VM

Show:

- VM name
- Running status
- Ubuntu
- Azure Portal

### Screenshot 2 — SSH Terminal

Show:

```text
azureuser@vm-100days-devops:~$
```

### Screenshot 3 — Linux Verification

Show:

```bash
hostname
cat /etc/os-release
df -h
```

### Screenshot 4 — Nginx

Show:

```bash
sudo systemctl status nginx
```

### Strong visual

```text
        Your Mac
           |
          SSH
           |
           v
      Azure VM
           |
        Ubuntu
           |
         Nginx
```

---

## 29. Key Takeaways

Today you learned:

- Azure Virtual Machines
- VM architecture
- Linux VMs
- Ubuntu
- SSH
- SSH key pairs
- Public IP addresses
- Network interfaces
- Virtual networks
- Subnets
- Network Security Groups
- Creating VMs with Azure CLI
- Connecting to a cloud server
- Installing Nginx
- Checking VM status
- Starting a VM
- Deallocating a VM
- Deleting Azure resources
- Cloud cost awareness

The important DevOps mindset is:

> Infrastructure is not just a server. It is a collection of connected resources that must be created, configured, secured, monitored and eventually cleaned up.

---

## 30. Real-World DevOps Example

A simplified workflow:

```text
Developer
    |
    v
Infrastructure Code
    |
    v
Azure
    |
    v
Virtual Machine
    |
    v
Application
```

Later, Terraform will help automate this.

A more mature workflow looks like:

```text
Git
 |
 v
Terraform
 |
 v
Azure Infrastructure
 |
 v
VM / Kubernetes
 |
 v
Application
 |
 v
CI/CD
```

---

## 31. GitHub Update

After adding this file:

```bash
git status
```

```bash
git add Day-43-Azure-Virtual-Machines.md
```

```bash
git commit -m "Add Day 43 Azure Virtual Machines"
```

```bash
git push origin main
```

Verify:

```bash
git log --oneline -1
```

---

# 32. LinkedIn Post — Day 43/100

## Day 43/100 — I Created an Azure VM and Connected to It Using SSH 🚀

Today I moved from simply managing Azure resources to actually running a server in the cloud.

The goal was simple:

**Create a Linux VM → connect to it → manage it from the terminal.**

The architecture looked like:

```text
My Mac
   |
   | SSH
   v
Public IP
   |
   v
Azure VM
   |
   v
Ubuntu Linux
   |
   v
Nginx
```

Today I practiced:

✅ Creating an Ubuntu VM using Azure CLI

✅ Generating SSH keys

✅ Finding the VM's public IP

✅ Connecting using SSH

✅ Checking the Linux environment

✅ Installing Nginx

✅ Checking the Nginx service

✅ Managing the VM using Azure CLI

One command that made the whole experience real:

```bash
ssh azureuser@<VM-IP>
```

And suddenly I was inside a server running somewhere in Azure.

I also learned that an Azure VM isn't just a single component.

Behind it are resources such as:

```text
VM
 |
 +--- OS Disk
 +--- Network Interface
 +--- Virtual Network
 +--- Subnet
 +--- Public IP
 +--- NSG
```

This is where cloud infrastructure starts becoming much more interesting.

Another important lesson:

**Cloud resources cost money.**

So after completing a lab, it's important to understand when to stop, deallocate or delete resources.

Today wasn't just about creating a VM.

It was about understanding how a DevOps engineer interacts with infrastructure running in the cloud.

**Day 43 complete. 🚀**

Next up:

**Day 44 — Azure Virtual Networks**

Now that I have a VM, the next question is:

**How does it communicate with other resources?**

#100DaysOfDevOps #DevOps #Azure #MicrosoftAzure #AzureVM #CloudComputing #Linux #SSH #AzureCLI #LearningInPublic #DevOpsJourney

---

## 33. Day 44 Preview

### Day 44 — Azure Virtual Networks

Next we will learn:

- What Azure VNet is
- Subnets
- Private communication
- Public vs private networking
- VNet architecture
- How Azure VMs communicate
- Basic VNet commands

We will start moving from:

```text
One VM
```

to:

```text
VM
 |
 v
Virtual Network
 |
 +--- Subnet
 |
 +--- Other Resources
```

---

# 🚀 Progress

**43 / 100 Days Completed**

```text
█████████████████████░░░░░░░░░░░░░░░░░░░ 43%
```

**Day 43 complete — Azure Virtual Machines.**
