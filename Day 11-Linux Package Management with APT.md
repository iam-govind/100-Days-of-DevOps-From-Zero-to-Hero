Day 11/100 — Linux Package Management with APT

Welcome to Day 11/100 of your restarted 100 Days of DevOps — From Zero to Hero journey.

Today we continue from Linux processes and services and learn another essential server administration skill:

How do you install and manage software on a Linux server?

For Ubuntu/Debian-based systems, one of the most important tools is APT.

🎯 Today's Objective

By the end of Day 11, you should understand:

What a Linux package is
What a package manager does
What APT is
Repositories
apt update
apt upgrade
apt install
apt remove
apt purge
apt search
apt show
dpkg
Why package management matters in DevOps
🧠 What Is a Linux Package?

A package is a bundle containing software and the information required to install it on your system.

For example, instead of manually downloading and configuring every file needed for a tool, you can use:

sudo apt install nginx

APT takes care of much of the work.

Think of it like this:

Software Repository
        ↓
      APT
        ↓
    Package
        ↓
 Installed Software
🏪 What Is a Repository?

A repository is a location containing software packages that your Linux distribution can retrieve.

Your system knows which repositories to use through its package configuration.

So when you execute:

sudo apt install nginx

APT can:

Find package
     ↓
Resolve dependencies
     ↓
Download packages
     ↓
Install packages
     ↓
Configure software

This is much easier than manually installing every dependency.

🔄 apt update vs apt upgrade

This distinction is extremely important.

apt update
sudo apt update

Refreshes the local package information from configured repositories.

Think:

"Find out what packages and versions are available."

apt upgrade
sudo apt upgrade

Actually upgrades installed packages where updates are available.

Think:

"Install available updates."

So:

apt update
     ↓
Refresh package information
     ↓
apt upgrade
     ↓
Install available updates

apt update does not itself upgrade your installed packages.

💻 Hands-on Lab

We'll use a small web-server example because this is directly relevant to DevOps.

Step 1 — Create Today's Lab Directory
cd ~/devops-linux
mkdir -p day11
cd day11

Check:

pwd
Step 2 — Check Your Linux Distribution

Run:

cat /etc/os-release

On Ubuntu, you'll see information containing something similar to:

NAME="Ubuntu"
VERSION="..."
ID=ubuntu

You can also run:

uname -a

Remember:

Linux kernel ≠ Linux distribution

Ubuntu is a distribution that uses the Linux kernel.

Step 3 — Check APT Version

Run:

apt --version

You should see an APT version.

This confirms that APT is available.

🔄 Step 4 — Refresh Package Information

Run:

sudo apt update

You'll see APT contacting configured repositories.

The output may contain lines such as:

Hit:1 ...
Get:2 ...
Reading package lists... Done

The exact output will vary depending on your Ubuntu version and repositories.

Important

At this stage, you haven't necessarily installed anything.

You've refreshed the information about available packages.

🔍 Step 5 — Search for a Package

Let's search for Nginx:

apt search nginx

You'll get a list of packages related to Nginx.

You can also use:

apt-cache search nginx

For a specific package:

apt show nginx

This provides information such as:

Package name
Version
Architecture
Dependencies
Description
Package size
📦 Step 6 — Install Nginx

Now install the web server:

sudo apt install nginx

APT may ask:

Do you want to continue? [Y/n]

Enter:

Y

APT will:

Download package
      ↓
Install Nginx
      ↓
Configure it
      ↓
Start the service
🔍 Step 7 — Check the Installed Package

Run:

apt list --installed | grep nginx

You should see installed Nginx packages.

You can also:

dpkg -l | grep nginx

This introduces another important Linux command:

dpkg is the lower-level Debian package management tool.

A simplified relationship is:

APT
 ↓
Higher-level package management
 ↓
Dependencies + repositories

dpkg
 ↓
Lower-level package management
 ↓
.deb packages

You don't need to memorize the distinction yet.

🖥️ Step 8 — Check the Nginx Service

This connects directly to Day 10, where we learned about services.

Run:

systemctl status nginx

Look for:

Active: active (running)

You can get a simpler result:

systemctl is-active nginx

Expected:

active

🎉 You've now connected two Linux concepts:

APT
 ↓
Install Nginx
 ↓
systemd
 ↓
Nginx service
 ↓
Running process
🌐 Step 9 — Test the Web Server

Run:

curl http://localhost

You should receive HTML output from Nginx.

You may see something beginning with:

<!DOCTYPE html>
<html>
...

This means your web server is responding locally.

You can also check:

curl -I http://localhost

You should see an HTTP response containing something similar to:

HTTP/1.1 200 OK
🔎 Step 10 — Find Where Nginx Is Installed

Run:

which nginx

You may get:

/usr/sbin/nginx

You can also inspect the package files:

dpkg -L nginx

This shows files installed by the package.

Because this can produce a lot of output, try:

dpkg -L nginx | less

Press:

q

to exit.

📊 Step 11 — Check the Installed Version

Run:

nginx -v

You should see the installed Nginx version.

This is useful when troubleshooting compatibility issues.

For example:

Application expects version X
Server has version Y

Knowing exactly what is installed matters in infrastructure work.

🛑 Step 12 — Stop Nginx

Let's connect today's lesson to yesterday's lesson.

Run:

sudo systemctl stop nginx

Check:

systemctl is-active nginx

Expected:

inactive

Now test:

curl -I http://localhost

The request should fail because Nginx isn't running.

▶️ Step 13 — Start Nginx Again

Run:

sudo systemctl start nginx

Check:

systemctl is-active nginx

Expected:

active

Test again:

curl -I http://localhost

You should receive an HTTP response again.

🔄 Step 14 — Remove Nginx

Once you've finished the lab, you can remove it:

sudo apt remove nginx

APT may ask for confirmation.

Enter:

Y

Check:

apt list --installed 2>/dev/null | grep nginx

Depending on dependencies and package state, you may still see related Nginx packages.

🧹 Step 15 — Understand remove vs purge

This distinction is worth knowing.

Remove
sudo apt remove nginx

Removes the package but may leave configuration files behind.

Purge
sudo apt purge nginx

Removes the package and its system-level configuration files associated with that package.

You can then clean unused dependencies with:

sudo apt autoremove

Be careful with autoremove on systems where you care about installed packages—it can remove packages APT considers no longer required.

🧪 Mini Challenge

Try completing these without looking back.

Challenge 1 — Find package information
apt show curl
Challenge 2 — Search for Git
apt search git
Challenge 3 — Check whether Git is installed
apt list --installed 2>/dev/null | grep '^git/'
Challenge 4 — Find the installed version

If Git is installed:

git --version
Challenge 5 — Find package files
dpkg -L curl | less
Challenge 6 — Check available updates
sudo apt update

Then inspect what could be upgraded:

apt list --upgradable
🧠 Important Commands Today
Command	Purpose
apt update	Refresh package information
apt upgrade	Upgrade installed packages
apt install	Install software
apt remove	Remove software
apt purge	Remove software + configuration
apt autoremove	Remove unused dependencies
apt search	Search for packages
apt show	Show package details
apt list --installed	List installed packages
apt list --upgradable	Show available upgrades
dpkg -l	List installed Debian packages
dpkg -L	List files installed by a package
which	Locate an executable
curl	Make HTTP/network requests
🔗 The DevOps Connection

Today's exercise may look like basic Linux administration.

But consider a real DevOps workflow.

You might provision a new server and need:

New Linux VM
     ↓
Update package information
     ↓
Install Git
     ↓
Install Docker
     ↓
Install monitoring agent
     ↓
Install application dependencies
     ↓
Configure services
     ↓
Start application

Later, when we reach:

Docker
Azure
CI/CD
Kubernetes
Terraform
Ansible

you'll repeatedly encounter the same fundamental concept:

Infrastructure needs software, and software needs to be installed, updated, configured, and maintained.

Package management is one of the building blocks underneath that process.

🔧 Troubleshooting
Unable to locate package

First run:

sudo apt update

Then try the installation again.

If it still fails, verify that the package name is correct:

apt search PACKAGE_NAME
Could not get lock

You may have another APT process running.

Check:

ps aux | grep -i apt

Don't immediately delete APT lock files.

First determine whether another package operation is actually running.

Nginx won't start

Check:

systemctl status nginx

Then inspect recent logs:

journalctl -u nginx --no-pager -n 50

You can also test its configuration:

sudo nginx -t

A successful configuration test should report that the configuration syntax is okay.

Port 80 already in use

Run:

sudo ss -ltnp | grep ':80'

This helps identify what is listening on port 80.

This is also a preview of our upcoming Linux networking lessons.

systemctl doesn't work

If you're using WSL, a container, or another environment without systemd enabled, systemctl may not work.

That's okay.

You can still practice:

apt
dpkg
curl
which

For the service portion, use a full Ubuntu VM or Linux system with systemd when available.

📸 What to Capture for LinkedIn

For today's post, I'd recommend a three-part terminal screenshot.

1️⃣ Package installation

Show:

sudo apt update
sudo apt install nginx
2️⃣ Service verification

Show:

systemctl status nginx

with:

Active: active (running)
3️⃣ Application verification

Show:

curl -I http://localhost

with:

HTTP/1.1 200 OK

Your story becomes:

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

That makes an excellent practical LinkedIn screenshot.

✍️ LinkedIn Post — Day 11/100

🚀 Day 11/100 — How Does Software Get Installed on a Linux Server?

Yesterday I learned how to investigate processes and services on Linux.

Today I took the next step:

How do we actually get software onto the server?

This is where Linux package management comes in.

On Ubuntu/Debian systems, one of the most important tools is:

apt

Today I practiced:

🔹 apt update — Refresh package information
🔹 apt install — Install software
🔹 apt search — Search for packages
🔹 apt show — Inspect package information
🔹 apt remove — Remove software
🔹 apt purge — Remove software and configuration
🔹 apt autoremove — Clean unused dependencies
🔹 dpkg — Work with Debian packages

For a hands-on exercise, I installed Nginx:

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

One distinction I found especially important:

sudo apt update

does not mean "upgrade everything."

It refreshes information about available packages.

Then:

sudo apt upgrade

can actually upgrade installed packages.

I also connected today's lesson with yesterday's:

Package management → Service management

Installing Nginx wasn't the end of the exercise.

I verified that the service was running:

systemctl is-active nginx

and then tested the web server:

curl -I http://localhost

Seeing an HTTP response from software I had just installed was a simple but useful demonstration of how Linux server administration works.

My biggest takeaway today:

A DevOps engineer doesn't just deploy applications. They also need to understand the operating system underneath those applications.

From installing software to managing services to troubleshooting failures, these fundamentals become the building blocks for automation later.

Day 11/100 completed ✅

Follow my profile for more practical DevOps insights, hands-on labs, and lessons from my 100 Days of DevOps — From Zero to Hero journey. 🚀

#100DaysOfDevOps #DevOps #Linux #LinuxForDevOps #Ubuntu #APT #LinuxAdministration #Automation #CloudComputing #LearningInPublic

📁 Recommended GitHub File

Save today's lesson as:

day11-linux-package-management.md

Lab folder:

labs/
└── day11/
    └── README.md
🔜 Day 12/100 — Linux Networking Fundamentals

Tomorrow we start Linux networking, which is a major foundation for DevOps.

We'll understand:

IP Address
    ↓
Network Interface
    ↓
Gateway
    ↓
DNS
    ↓
Internet

You'll practice commands such as:

ip
ping
ss
curl
nslookup
dig
traceroute

And we'll troubleshoot a simulated connectivity problem so you can start thinking like a DevOps engineer when an application says:

"It works on my machine, but I can't reach the server." 🚀
