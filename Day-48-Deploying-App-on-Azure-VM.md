# Day 48/100 — Deploying an Application on an Azure Virtual Machine

## 🎯 Learning Objectives

By the end of this lesson, you will be able to create an Ubuntu VM in Azure, connect over SSH, install Nginx, deploy a sample web page, configure HTTP access, test the application, and troubleshoot common problems.

## 1. What Is Application Deployment?

Application deployment means putting an application on a server and making it available to users. In this lab, we will deploy a simple HTML page using Nginx on an Ubuntu VM.

### Minimal architecture diagram

```text
Browser
   |
   | HTTP :80
   v
Azure Public IP
   |
   v
Network Security Group
   |
   v
Ubuntu VM --> Nginx --> index.html
```

A public IP alone does not guarantee that a site is reachable. The network rules, VM firewall, web server, and application must all work together.

## 2. Prerequisites and Cost Notes

- An Azure subscription with permission to create resources.
- Azure CLI installed and an SSH client (macOS Terminal is fine).
- A browser for testing.

**Cost and security:** A VM, disk, and public IP can incur charges. Use a small VM size supported by your subscription, and delete the lab resource group when finished. Never publish private SSH keys or credentials.

## 3. Sign In and Select a Subscription

```bash
az login
az account show --output table
```

If needed, select the intended subscription:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
az account show --query "{Name:name,Id:id}" --output table
```

## 4. Define Lab Variables and Create a Resource Group

```bash
RESOURCE_GROUP="rg-day48-app-deployment"
LOCATION="eastus"
VM_NAME="webvm-day48"
ADMIN_USER="azureuser"

az group create \
  --name "$RESOURCE_GROUP" \
  --location "$LOCATION"
```

If you already have a suitable lab resource group, you can use it instead. Choose a region supported by your subscription.

## 5. Create an Ubuntu VM

```bash
az vm create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --location "$LOCATION" \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username "$ADMIN_USER" \
  --generate-ssh-keys \
  --nsg-rule SSH
```

`Standard_B1s` is an example size; availability depends on region and subscription. If it is unavailable, choose a small supported size.

**Security note:** The `--nsg-rule SSH` shortcut may create a rule that is too broad for production. Immediately restrict inbound TCP port 22 in the VM's Network Security Group to your trusted public IP (`/32`). Do not leave SSH open to the entire internet for production systems.

Get the public IP:

```bash
PUBLIC_IP=$(az vm show \
  --show-details \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query publicIps \
  --output tsv)

echo "$PUBLIC_IP"
```

Expected: Azure returns VM details and a public IP address.

## 6. Restrict SSH and Connect to the VM

In the Azure portal:

1. Open the VM and select **Networking**.
2. Open the associated Network Security Group.
3. Edit the SSH inbound rule for TCP port 22.
4. Set the source to your trusted public IP with `/32`, not `Any` or `Internet`.

To check your current public IP from your terminal:

```bash
curl -s https://api.ipify.org
echo
```

Use your actual IP followed by `/32`. The example address `203.0.113.10/32` is documentation-only.

Connect from your Mac:

```bash
ssh "$ADMIN_USER@$PUBLIC_IP"
```

If you need a specific key, use its actual path:

```bash
ssh -i ~/.ssh/id_rsa "$ADMIN_USER@$PUBLIC_IP"
```

Never upload your private key to GitHub.

Expected: Your terminal opens a shell on the Ubuntu VM. If SSH times out, check the VM state, public IP, NSG rule, and local network. If authentication fails, check the username and key.

## 7. Install Nginx

Run these commands **inside the VM**, not on your Mac:

```bash
sudo apt-get update
sudo apt-get install -y nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
```

Test Nginx locally:

```bash
curl -I http://localhost
```

Expected: Nginx is active and the response includes an HTTP status such as `200 OK`.

## 8. Deploy a Sample HTML Page

```bash
sudo tee /var/www/html/index.html > /dev/null <<'EOF'
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Day 48 - Azure VM Deployment</title>
</head>
<body>
  <h1>My Day 48 Azure Deployment</h1>
  <p>This page is running on an Ubuntu VM in Azure.</p>
</body>
</html>
EOF
```

Test the page:

```bash
curl -I http://localhost
curl http://localhost
```

Expected: The response includes `My Day 48 Azure Deployment`.

## 9. Allow HTTP Traffic on Port 80

In the Azure portal:

1. Open your VM, then **Networking**.
2. Open the associated Network Security Group.
3. Add an inbound rule for TCP port `80`.
4. For a public demo, set source to Internet; use a narrower source for internal applications.
5. Choose **Allow**, set a priority that does not conflict with existing rules, and name it `Allow-HTTP-Day48`.

Review existing rules before saving. If Ubuntu's firewall is enabled, check it inside the VM:

```bash
sudo ufw status
```

If it is active and HTTP is blocked, use the approved firewall configuration. For example:

```bash
sudo ufw allow 80/tcp
```

Do not change firewall settings remotely without ensuring you retain a safe administration path.

## 10. Test the Application

From your local computer, open this URL with your actual public IP:

```text
http://<YOUR_PUBLIC_IP>
```

You should see **My Day 48 Azure Deployment**.

You can also test from your local terminal:

```bash
curl -I "http://$PUBLIC_IP"
```

Expected: An HTTP response such as `200 OK`.

## 11. Troubleshooting

### Website times out

Check in this order:
1. VM is running.
2. Public IP is correct.
3. NSG allows inbound TCP port 80.
4. Ubuntu firewall permits port 80.
5. Nginx is running.

Useful commands inside the VM:

```bash
sudo systemctl status nginx --no-pager
sudo ss -lntp | grep ':80'
curl -I http://localhost
sudo ufw status
```

### Connection refused

The VM may be reachable, but no service is accepting the connection. Check and restart Nginx:

```bash
sudo systemctl restart nginx
sudo systemctl status nginx --no-pager
```

### SSH `Permission denied (publickey)`

Verify the admin username and private key. If needed, secure the key file:

```bash
chmod 600 ~/.ssh/id_rsa
```

Use the actual key path, and never share the private key.

### Default Nginx page appears

Check the page content and configuration:

```bash
cat /var/www/html/index.html
sudo nginx -t
sudo journalctl -u nginx --no-pager -n 50
```

### VM size or quota error

Choose a VM size available in your region and subscription, and review your quota in the Azure portal.

## 12. Production Considerations

This is a learning lab using a public IP and HTTP, not a production architecture. For production:

- Prefer HTTPS with a valid TLS certificate.
- Restrict inbound traffic to required ports and trusted sources.
- Avoid exposing SSH to the whole internet; consider Azure Bastion or an approved secure access method.
- Keep the OS patched and monitor logs, metrics, and alerts.
- Use backups, recovery procedures, and appropriate secret management.
- Follow your organization's security and change-management requirements.

## 13. Cleanup — Avoid Unwanted Charges

**Warning:** The following command deletes every resource in the named resource group. Confirm it contains only lab resources you intend to remove.

```bash
az group delete \
  --name "$RESOURCE_GROUP" \
  --yes \
  --no-wait
```

Deletion may take time to complete.

## 14. Hands-On Challenge

1. Create an Ubuntu VM and connect over SSH.
2. Install Nginx and verify its status.
3. Deploy your own HTML page.
4. Allow HTTP through the NSG.
5. Test the page in a browser and with `curl`.
6. In a safe lab, stop Nginx, observe the failure, and start it again.
7. Delete the lab resources when finished.

### Reflection Questions

- Why does a public IP alone not make a website reachable?
- What is the difference between an NSG and the Ubuntu firewall?
- Why should SSH access be restricted?
- Why is HTTPS important for real applications?
- What would you add before calling this deployment production-ready?

## 15. What to Capture for LinkedIn

Capture screenshots of:
- The VM shown as running in Azure.
- `sudo systemctl status nginx` showing Nginx is active.
- Your deployed page in the browser.
- `curl -I http://localhost` showing a successful response.
- The NSG inbound rule for HTTP.

Hide subscription details, private IP information where sensitive, credentials, and all SSH keys.

## 16. LinkedIn Post — Day 48/100

**100 Days of DevOps — Day 48/100 🚀**

☁️ **Deploying an Application on an Azure Virtual Machine**

Cloud concepts become much more meaningful when you build something and make it work.

Today, I deployed a simple web page on an Ubuntu VM in Azure using Nginx.

🛠️ **What I practiced:**

✅ Created an Ubuntu VM using Azure CLI.  
✅ Connected to the VM over SSH.  
✅ Installed and started Nginx.  
✅ Deployed a custom HTML page.  
✅ Configured an Azure Network Security Group to allow HTTP traffic.  
✅ Verified the application using a browser and `curl`.

💡 **My key takeaway:**

A running VM is only one part of a successful deployment. The application service, network rules, operating-system firewall, and security configuration must all work together.

🔐 In production, I would also prioritize HTTPS, least-privilege network access, monitoring, updates, and secure administration.

**Question for the DevOps community:** What do you usually configure first after creating a cloud VM?

Continuing my journey through 100 Days of DevOps — learning by building, testing, and troubleshooting.

#100DaysOfDevOps #Azure #AzureVM #Linux #Nginx #CloudComputing #DevOps #LearningInPublic

## 17. Upload This Lesson to GitHub

Save this file as `Day-48-Deploying-App-on-Azure-VM.md` in your 100 Days of DevOps repository.

```bash
git add Day-48-Deploying-App-on-Azure-VM.md
git commit -m "Add Day 48 Azure VM application deployment"
git push origin main
```

Confirm that the file appears in your GitHub repository.

## Coming Next: Day 49/100

**Azure VM Monitoring and Troubleshooting**

Next, we will explore VM metrics, boot diagnostics, activity logs, connectivity problems, and a practical troubleshooting workflow.

Keep learning, keep building, and keep documenting your DevOps journey! 🚀
