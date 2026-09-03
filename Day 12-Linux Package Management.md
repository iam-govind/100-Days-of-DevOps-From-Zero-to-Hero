Day 12/100 — Linux Package Management

Series: 100 Days of DevOps — From Zero to Hero
Day 1: August 24, 2026
Today: Day 12 — September 3, 2026

🎯 Today's Topic
Linux Package Management

Today we'll understand how Linux installs, updates, removes, and manages software.

The main focus will be APT, the package manager commonly used on Ubuntu/Debian systems.

🧠 Why Do We Need Package Management?

Imagine you've created a new Linux server for your DevOps project.

Your application requires:

Git
Docker
Nginx
Python
Monitoring Agent

Without a package manager, you could end up manually:

Download software
       ↓
Find dependencies
       ↓
Install dependencies
       ↓
Configure software
       ↓
Install software
       ↓
Repeat...

That becomes difficult to manage.

A package manager simplifies this process:

             Package Repository
                    ↓
                   APT
                    ↓
          Dependencies Resolved
                    ↓
             Software Installed
                    ↓
               Service Ready

This is particularly important when we eventually start automating server provisioning.

📦 What Is a Package?

A package is a collection of files and metadata used to install software.

It can contain:

Application files
Configuration files
Dependencies
Documentation
Version information
Installation information

For example:

sudo apt install nginx

APT handles much of the work required to get Nginx installed.

🏪 What Is a Repository?

A repository is a source containing software packages.

Conceptually:

Linux Server
     ↓
Repository
     ↓
   APT
     ↓
 Package + Dependencies
     ↓
 Installed Software

Your Linux system has configured repositories from which APT can obtain packages.

⭐ Most Important Concept Today
apt update ≠ apt upgrade

This is something I want you to remember.

apt update
sudo apt update

This refreshes information about available packages.

Think:

"What software updates are available?"

apt upgrade
sudo apt upgrade

This actually upgrades installed packages where updates are available.

Think:

"Install those available updates."

So:

apt update
     ↓
Refresh package information
     ↓
apt upgrade
     ↓
Upgrade installed packages
💻 Hands-on Lab

We'll install Nginx, verify it, manage its service, test it, and then remove it.

This also connects today's lesson with the Linux Processes & Services topic from Day 11.

Step 1 — Create the Day 12 Lab
cd ~/devops-linux
mkdir -p day12
cd day12

Verify:

pwd

You should see something similar to:

/home/<your-user>/devops-linux/day12
Step 2 — Identify Your Linux Distribution

Run:

cat /etc/os-release

If you're using Ubuntu, you'll see something similar to:

NAME="Ubuntu"
ID=ubuntu

Also run:

uname -a

Remember:

Linux
  ↓
Kernel

Ubuntu
  ↓
Linux Distribution
Step 3 — Check APT

Run:

apt --version

You should get an APT version.

For example:

apt 2.x.x

The exact version depends on your Linux distribution.

🔄 Step 4 — Update Package Information

Run:

sudo apt update

You may see:

Hit:1 ...
Get:2 ...
Reading package lists... Done

The exact output will depend on your system and repositories.

Important

This command doesn't mean that all your installed software has been upgraded.

It refreshes the package information.

🔍 Step 5 — Search for Nginx

Run:

apt search nginx

You'll see packages related to Nginx.

Now inspect the package:

apt show nginx

Look for information such as:

Package
Version
Architecture
Depends
Description
📦 Step 6 — Install Nginx

Run:

sudo apt install nginx

If prompted:

Do you want to continue? [Y/n]

Enter:

Y

APT will handle the installation.

Conceptually:

Find package
      ↓
Resolve dependencies
      ↓
Download
      ↓
Install
      ↓
Configure
      ↓
Start service
🔎 Step 7 — Verify Installation

Run:

apt list --installed 2>/dev/null | grep nginx

You should see installed Nginx packages.

Another method:

dpkg -l | grep nginx
🧩 APT vs dpkg

You'll see both commands frequently.

A simplified understanding is:

APT
 │
 ├── Repositories
 ├── Dependencies
 ├── Install
 └── Upgrade

while:

dpkg
 │
 ├── Debian packages
 ├── Install information
 └── Package database

You don't need to master dpkg today.

Just remember:

APT is the higher-level package management tool you'll use frequently on Ubuntu/Debian.

🖥️ Step 8 — Check the Nginx Service

Now connect this lesson with yesterday's Linux service concepts.

Run:

systemctl status nginx

Look for:

Active: active (running)

Or use:

systemctl is-active nginx

Expected:

active

Your flow is now:

APT
 ↓
Install Nginx
 ↓
systemd
 ↓
Nginx Service
 ↓
Running Process
🌐 Step 9 — Test Nginx

Run:

curl http://localhost

You should receive HTML from the Nginx default page.

You can also just inspect the HTTP headers:

curl -I http://localhost

Look for something similar to:

HTTP/1.1 200 OK

🎉 Your Linux machine is now serving a web page.

🔍 Step 10 — Find the Nginx Executable

Run:

which nginx

You may get:

/usr/sbin/nginx

Now inspect the files installed by the package:

dpkg -L nginx | less

Press:

q

to exit.

🏷️ Step 11 — Check the Version

Run:

nginx -v

You'll see the installed Nginx version.

Knowing versions becomes important later when troubleshooting compatibility issues.

🛑 Step 12 — Stop Nginx

Run:

sudo systemctl stop nginx

Check:

systemctl is-active nginx

Expected:

inactive

Now test:

curl -I http://localhost

The request should fail because Nginx is no longer running.

▶️ Step 13 — Start Nginx Again

Run:

sudo systemctl start nginx

Check:

systemctl is-active nginx

Expected:

active

Test:

curl -I http://localhost

You should get an HTTP response again.

🔄 Step 14 — Restart Nginx

Run:

sudo systemctl restart nginx

Then:

systemctl is-active nginx

Expected:

active
🧹 Step 15 — Remove Nginx

Once you're finished with the lab:

sudo apt remove nginx

Confirm with:

Y

Then check:

apt list --installed 2>/dev/null | grep nginx
🧽 Remove vs Purge

This distinction is useful.

Remove
sudo apt remove nginx

Removes the package but may leave configuration files.

Purge
sudo apt purge nginx

Removes the package and its package-managed configuration.

You can also inspect unused dependencies:

sudo apt autoremove

⚠️ Always review what APT proposes before confirming autoremove.

🧪 Mini Challenge

Try these yourself.

1. Search for Git
apt search git
2. Show Git package information
apt show git
3. Check whether Git is installed
apt list --installed 2>/dev/null | grep '^git/'
4. Check Git version
git --version
5. Show available upgrades
apt list --upgradable
6. Find files belonging to curl
dpkg -L curl | less
🧠 Commands Learned Today
Command	Purpose
apt update	Refresh package information
apt upgrade	Upgrade installed packages
apt install	Install software
apt remove	Remove software
apt purge	Remove software + configuration
apt autoremove	Remove unused dependencies
apt search	Search packages
apt show	Show package information
apt list --installed	List installed packages
apt list --upgradable	Show available updates
dpkg -l	List Debian packages
dpkg -L	List package files
which	Locate an executable
curl	Make HTTP requests
🔧 Troubleshooting
❌ Unable to locate package

Run:

sudo apt update

Then:

apt search PACKAGE_NAME

Make sure the package name is correct.

❌ APT lock error

Check whether another APT process is running:

ps aux | grep -i apt

Don't immediately delete lock files.

❌ Nginx won't start

Check:

systemctl status nginx

Then:

sudo nginx -t

And inspect recent service logs:

journalctl -u nginx --no-pager -n 50
❌ Port 80 already in use

Run:

sudo ss -ltnp | grep ':80'

This will help identify what is listening on port 80.

You'll learn much more about ports and networking in the upcoming Networking section.

❌ systemctl doesn't work

If you're using WSL, a container, or a Linux environment without systemd, systemctl may not be available.

That's okay.

You can still practice:

apt
dpkg
curl
which

For service-management practice, a Linux VM with systemd is preferable.

📸 What Screenshot Should You Capture?

For LinkedIn, don't just screenshot the installation command.

Capture the complete story:

Screenshot 1
sudo apt install nginx
Screenshot 2
systemctl status nginx

showing:

Active: active (running)
Screenshot 3
curl -I http://localhost

showing:

HTTP/1.1 200 OK

Your visual story:

Linux Server
     ↓
    APT
     ↓
Install Nginx
     ↓
systemctl
     ↓
Service Running
     ↓
curl
     ↓
HTTP 200 OK
✍️ LinkedIn Post — Day 12/100

🚀 Day 12/100 — How Does Software Get Installed on a Linux Server?

Imagine creating a brand-new Linux server.

Your application needs:

Git.
Nginx.
Python.
Docker.
Monitoring tools.

Would you manually download every package, find its dependencies, configure everything, and repeat the process for every server?

That quickly becomes difficult to manage.

This is where Linux package management becomes important.

On Ubuntu/Debian systems, one of the key tools is:

apt

Today I learned how package management fits into the Linux and DevOps workflow.

I practiced:

🔹 apt update — Refresh package information
🔹 apt install — Install software
🔹 apt search — Search for packages
🔹 apt show — Inspect package details
🔹 apt remove — Remove software
🔹 apt purge — Remove software and package-managed configuration
🔹 dpkg — Work with Debian packages

For the hands-on lab, I installed Nginx:

sudo apt install nginx

Then I verified that the service was running:

systemctl is-active nginx

Finally, I tested the web server:

curl -I http://localhost

and received:

HTTP/1.1 200 OK

The workflow became:

Linux Server
     ↓
    APT
     ↓
Install Nginx
     ↓
Service Management
     ↓
Verify

One distinction I found especially important:

apt update does NOT mean "upgrade everything."

It refreshes the information about available packages.

Then:

apt upgrade

can install available updates.

The bigger DevOps connection is becoming clearer:

Install → Configure → Start → Verify → Automate

Later in this journey, tools such as Terraform and Ansible will help automate many of these tasks across infrastructure.

My biggest takeaway today:

DevOps engineers need to understand the operating system underneath the applications they're deploying.

The tools will change.

The fundamentals remain.

Day 12/100 completed ✅

Follow my profile for more practical DevOps insights, hands-on labs, and lessons from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #Linux #LinuxForDevOps #Ubuntu #APT #LinuxAdministration #Automation #CloudComputing #LearningInPublic

📁 GitHub

Save today's lesson as:

day12-linux-package-management.md

Recommended lab:

labs/
└── day12/
    └── README.md
🔜 Day 13/100 — SSH and Remote Server Access

Tomorrow we move into one of the most important skills for a DevOps engineer:

Connecting to and managing a remote Linux server.

We'll learn:

Your Laptop
     │
     │ SSH
     ↓
Remote Linux Server
     │
     ├── Commands
     ├── Files
     ├── Processes
     └── Services

You'll practice:

What SSH is
SSH client/server
ssh
SSH keys
ssh-keygen
Public vs private keys
Remote login
Basic SSH troubleshooting
Why SSH is fundamental to cloud and DevOps

Day 13 will be the bridge from "working on Linux" to "managing Linux servers remotely." 🚀
