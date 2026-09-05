Day 13/100 — SSH and Remote Server Access

Today we're learning one of the most important Linux skills for DevOps:

How to securely connect to a remote Linux server using SSH.

This becomes especially important when you start working with Azure VMs, cloud servers, CI/CD servers, and production infrastructure.

🎯 Today's Learning Objectives

By the end of today, you should understand:

What SSH is
How SSH works
SSH client vs SSH server
How to connect to a remote Linux machine
SSH username and IP address
Password-based authentication
SSH key-based authentication
Basic SSH troubleshooting
How to safely disconnect from a server
1. What is SSH?

SSH = Secure Shell

SSH is a protocol that allows you to securely access and manage another computer over a network.

For example:

Your Laptop
     │
     │ SSH
     ▼
Internet / Network
     │
     ▼
Linux Server
10.0.0.20

You can remotely execute commands on the Linux server as if you were sitting in front of it.

2. Why SSH Matters in DevOps

Imagine your application is running on an Azure VM.

You don't physically have access to that server.

Instead, you can connect using:

ssh username@server-ip

For example:

ssh ubuntu@20.10.30.40

Once connected:

Your Laptop
     │
     │ SSH
     ▼
Azure Linux VM
     │
     ├── Docker
     ├── Nginx
     ├── Jenkins
     └── Application

This is why SSH is fundamental to DevOps.

3. How SSH Works

The basic process looks like this:

SSH Client
   │
   │ 1. Connection request
   ▼
SSH Server
   │
   │ 2. Authentication
   ▼
User verified
   │
   ▼
Secure encrypted session
   │
   ▼
Execute commands remotely

By default, SSH uses TCP port 22.

We'll explore ports more deeply in the Networking section.

4. Check Whether SSH Is Available

If you're using Ubuntu/Linux, check:

ssh -V

You should see something similar to:

OpenSSH_9.x

The exact version may be different.

This confirms that the SSH client is installed.

🧪 Hands-On Lab

We'll perform a simple SSH connection between two machines.

You can use:

Two Linux machines, or
A Linux VM and your laptop, or
WSL/Linux VM on your computer

If you only have one Linux machine available, you can still practice many of today's commands.

Step 1 — Find Your IP Address

On Linux:

ip addr

or:

hostname -I

Example:

192.168.1.25

This is the IP address you'll use to connect to the machine from another computer on the same network.

Step 2 — Check the SSH Server

On Ubuntu, check:

sudo systemctl status ssh

You want to see:

Active: active (running)

If SSH isn't installed, install the OpenSSH server:

sudo apt update
sudo apt install openssh-server

Then:

sudo systemctl enable --now ssh

Check again:

sudo systemctl status ssh
Step 3 — Check SSH Port

Run:

sudo ss -tlnp | grep :22

You should see something similar to:

LISTEN ... :22 ...

This means the SSH server is listening for connections on port 22.

Step 4 — Connect Using SSH

From another Linux machine, Windows PowerShell, or Terminal:

ssh username@IP_ADDRESS

For example:

ssh govind@192.168.1.25

The first time you connect, you may see:

Are you sure you want to continue connecting (yes/no/[fingerprint])?

Type:

yes

Then enter the user's password.

If successful, you'll get a remote shell.

For example:

govind@server:~$

🎉 You're now controlling the remote Linux machine.

Step 5 — Confirm You're on the Remote Machine

Run:

hostname

Then:

whoami

And:

hostname -I

Example:

server01
govind
192.168.1.25

This is a great way to prove that your SSH connection worked.

Step 6 — Execute a Remote Command Without Opening a Shell

SSH can also execute a single command remotely.

For example:

ssh username@IP_ADDRESS "hostname"

Or:

ssh username@IP_ADDRESS "uptime"

Or:

ssh username@IP_ADDRESS "df -h"

This is extremely important for automation.

Instead of manually logging into hundreds of servers, automation tools can execute commands remotely.

Step 7 — Disconnect

When you're finished:

exit

You'll return to your local machine.

You can also press:

Ctrl + D
🔐 SSH Key-Based Authentication

Password authentication works, but DevOps environments commonly use SSH keys.

Instead of:

Username + Password

you use:

Private Key + Public Key

The basic architecture is:

Your Computer
─────────────────
Private Key 🔑
     │
     │ SSH
     ▼
Linux Server
─────────────────
Public Key 🔓
Important:

Never share your private key.

Your private key should remain on your computer.

Step 8 — Generate an SSH Key

On your local machine:

ssh-keygen

You can usually accept the default location by pressing Enter.

You'll get files similar to:

~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub

The important distinction:

id_ed25519       → PRIVATE KEY
id_ed25519.pub   → PUBLIC KEY
Step 9 — Copy the Public Key

If ssh-copy-id is available:

ssh-copy-id username@IP_ADDRESS

Then connect:

ssh username@IP_ADDRESS

If configured correctly, you can authenticate using your SSH key instead of entering the server password each time.

🔥 Why SSH Keys Matter in DevOps

SSH keys are commonly used for:

Linux server access
Git/GitHub authentication
Cloud VM access
Ansible
CI/CD systems
Automation scripts
Infrastructure management

You'll encounter them repeatedly later in this 100-day journey.

🛠️ Troubleshooting
❌ Connection refused

Check whether SSH is running:

sudo systemctl status ssh

Start it:

sudo systemctl start ssh
❌ Connection timed out

This usually indicates a networking or firewall problem.

Check:

ping SERVER_IP

Then:

sudo ss -tlnp | grep :22

For cloud servers, also check the cloud firewall/security rules.

❌ Permission denied

Check the username:

whoami

Also verify that you're using the correct authentication method.

❌ ssh: command not found

Install the SSH client:

sudo apt update
sudo apt install openssh-client
❌ SSH service isn't installed

Install:

sudo apt install openssh-server

Then:

sudo systemctl enable --now ssh
📸 What Screenshot Should You Capture?

For today's LinkedIn post, capture a terminal showing:

ssh username@IP_ADDRESS

followed by:

hostname
whoami
hostname -I

For example:

govind@server:~$ hostname
server01

govind@server:~$ whoami
govind

govind@server:~$ hostname -I
192.168.1.25

Do not expose your public IP, private key, password, or other sensitive credentials in your LinkedIn screenshot.

🧠 Day 13 Key Takeaways

Remember these five things:

SSH
│
├── Secure remote access
├── Default port → 22
├── Client → ssh
├── Server → openssh-server
└── Authentication → Password / SSH Keys

The most important command:

ssh username@server-ip
💼 LinkedIn Post — Day 13/100

Here's a natural, problem-first version for your series:

Day 13/100 — SSH and Remote Server Access 🔐

Imagine your application is running on a Linux server somewhere in the cloud.

You don't have a monitor connected to it.

You can't physically sit in front of it.

So how do you manage it?

SSH.

Today I learned how SSH allows us to securely connect to and manage a remote Linux server from our own machine.

I practiced:

🔹 Checking the SSH client
🔹 Finding the server IP address
🔹 Checking the SSH service
🔹 Connecting using ssh username@server-ip
🔹 Executing commands remotely
🔹 Checking the hostname and logged-in user
🔹 Disconnecting from the remote server
🔹 Understanding SSH key-based authentication

The basic command is simple:

ssh username@server-ip

But behind that simple command is something very important for DevOps — secure remote access and automation.

As I continue this journey, I'll be using SSH concepts again with cloud VMs, Ansible, CI/CD, Git, and infrastructure automation.

Today's small hands-on exercise helped connect the dots between Linux administration and real-world DevOps.

One more step forward in the journey. 🚀

Day 13/100 ✅

Tomorrow: Linux Networking Commands 🌐

#100DaysOfDevOps #DevOps #Linux #SSH #Cloud #LinuxAdministration #Automation #DevOpsJourney #LearningInPublic #100DaysChallenge

🔜 Tomorrow — Day 14/100

Linux Networking Commands

We'll work with commands such as:

ip
ping
ss
curl
wget
traceroute
nslookup

and start connecting what we've learned about Linux + SSH + networking into a much bigger DevOps picture.
