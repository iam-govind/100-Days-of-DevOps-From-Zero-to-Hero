Day 15/100 — Linux Troubleshooting Basics

Today we complete the Linux section of the 100 Days of DevOps journey.

So far, you've learned Linux fundamentals, users and permissions, processes and services, package management, SSH, and networking.

Today we'll bring those pieces together and learn how to troubleshoot a Linux server systematically.

🎯 Today's Goal

By the end of Day 15, you should know how to investigate:

CPU problems
Memory problems
Disk-space problems
Service failures
Network problems
High system load
Running processes
System logs

The biggest lesson today is:

Don't guess when troubleshooting. Collect evidence first.

1. A Simple Linux Troubleshooting Method

When something goes wrong, follow a structured process:

       Problem
          │
          ▼
   Check system status
          │
          ▼
   Check CPU / Memory
          │
          ▼
      Check Disk
          │
          ▼
 Check Processes/Services
          │
          ▼
    Check Networking
          │
          ▼
      Check Logs
          │
          ▼
    Identify Root Cause

This approach becomes extremely valuable later with Docker, Kubernetes, Azure, and CI/CD.

2. Check System Uptime and Load

Start with:

uptime

Example:

10:15:30 up 2 days, 4:12, 2 users, load average: 0.20, 0.15, 0.10

The load averages represent system load over approximately:

1 minute
5 minutes
15 minutes

You don't need to memorize the interpretation yet. The important thing is to recognize when the system is under unusually high load.

3. Check CPU and Processes

Run:

top

You'll get a live view of:

CPU usage
Memory usage
Running processes
Process IDs
System load

Press:

q

to exit.

If available, you can also use:

htop

Install it with:

sudo apt update
sudo apt install htop

Then:

htop
4. Find Resource-Hungry Processes

Run:

ps aux --sort=-%cpu | head

This shows processes consuming the most CPU.

For memory:

ps aux --sort=-%mem | head

This is useful when someone reports:

"The server is very slow."

Instead of guessing, you can identify which processes are consuming resources.

5. Check Memory

Run:

free -h

Example:

              total   used   free
Mem:           7.7G   2.1G   3.2G
Swap:          2.0G   0.0G   2.0G

The -h means human-readable.

Without it:

free

may show values in bytes or less-friendly units.

6. Check Disk Space

A common server problem is:

Disk is full.

Start with:

df -h

Example:

Filesystem      Size  Used Avail Use%
/dev/sda2        50G   32G   16G  67%

Pay attention to:

Use%

If a filesystem reaches 100%, applications can start failing.

7. Find Large Directories

If disk usage is high, investigate further.

Run:

sudo du -sh /var/*

You might discover:

/var/log
/var/cache
/var/lib

are consuming significant space.

You can drill down:

sudo du -sh /var/log/*

This helps identify where the disk space is being consumed.

8. Check Services

Suppose your Nginx web server isn't working.

Check:

sudo systemctl status nginx

If it isn't running:

sudo systemctl start nginx

To check whether it starts automatically after reboot:

sudo systemctl is-enabled nginx

This connects directly with what you learned earlier about Linux services.

9. Check Service Logs

If a service fails, don't stop at:

systemctl status nginx

Look at the logs:

sudo journalctl -u nginx

For recent logs:

sudo journalctl -u nginx --since "10 minutes ago"

For the most recent entries:

sudo journalctl -u nginx -n 50

Logs often contain the actual reason for failure.

10. Check Networking

If a server cannot reach something, use the commands from Day 14.

Check IP:

ip addr

Check routes:

ip route

Test connectivity:

ping -c 4 8.8.8.8

Check DNS:

nslookup google.com

Check listening ports:

ss -tuln

Test HTTP:

curl -I https://example.com
🧪 Hands-On Troubleshooting Lab

Let's simulate a real DevOps troubleshooting workflow.

Step 1 — Get an Overall System Picture

Run:

hostname
uptime
free -h
df -h

You're collecting the basic health information of the machine.

Step 2 — Check CPU Usage

Run:

top

Look at:

Load average
CPU usage
Memory usage
Running processes

Exit with:

q
Step 3 — Find the Top CPU Consumers
ps aux --sort=-%cpu | head -10

Expected result:

A list of processes sorted by CPU consumption.

Step 4 — Find the Top Memory Consumers
ps aux --sort=-%mem | head -10

Expected result:

A list of processes sorted by memory usage.

Step 5 — Check Disk Usage
df -h

Then:

sudo du -sh /var/log/*

This is a particularly useful command for server troubleshooting because logs can grow significantly over time.

Step 6 — Check Running Services

Run:

systemctl --type=service --state=running

This gives you an overview of currently running services.

Step 7 — Check SSH

Since SSH is essential for remote administration:

sudo systemctl status ssh

Then:

ss -tuln | grep :22

You should see SSH listening on port 22 if the service is configured normally.

Step 8 — Check Recent System Errors

Run:

sudo journalctl -p err -b

This filters the journal for error-priority messages from the current boot.

If there are no results, that's okay.

It can simply mean there are no errors at that priority level.

🔥 A Real-World Troubleshooting Scenario

Imagine someone tells you:

"The application on my Linux server is not responding."

Don't immediately restart everything.

Instead:

1. Is the server alive?
uptime
2. Is the server under resource pressure?
free -h
df -h
3. Is the application process running?
ps aux | grep application-name
4. Is its service running?
systemctl status application-name
5. Is the expected port listening?
ss -tuln
6. Can the service respond locally?
curl http://localhost:PORT
7. Are there errors in the logs?
journalctl -u application-name
8. Is there a network problem?
ip route
ping -c 4 DESTINATION

This gives you a repeatable troubleshooting methodology.

🛠️ Common Problems
df -h shows 100%

Investigate:

sudo du -sh /var/*

Then:

sudo du -sh /var/log/*

Don't blindly delete files from system directories.

A service won't start

Check:

systemctl status SERVICE_NAME

Then:

journalctl -u SERVICE_NAME

Also check configuration if the application supports a configuration test.

For Nginx:

sudo nginx -t
Server feels slow

Start with:

uptime
free -h
top

Then identify resource-heavy processes:

ps aux --sort=-%cpu | head

and:

ps aux --sort=-%mem | head
Application isn't reachable

Check in this order:

ip addr
ip route
ss -tuln
curl http://localhost:PORT

Then investigate firewall/network rules.

📸 What to Capture for LinkedIn

Today's strongest screenshot would be a single terminal screenshot showing your troubleshooting toolkit:

uptime
free -h
df -h
ss -tuln
systemctl --type=service --state=running

If possible, capture a second screenshot showing:

ps aux --sort=-%cpu | head
journalctl -p err -b

Before posting, hide anything sensitive such as:

Public IP addresses
Hostnames you don't want exposed
Usernames
Internal infrastructure information
🧠 Day 15 Key Takeaways

The commands are useful, but the bigger lesson is the troubleshooting mindset:

Observe
  ↓
Measure
  ↓
Check resources
  ↓
Check services
  ↓
Check network
  ↓
Check logs
  ↓
Find root cause
  ↓
Fix
  ↓
Verify

Avoid:

Problem
  ↓
Random command
  ↓
Restart everything
  ↓
Hope it works 😅

Good DevOps troubleshooting is about evidence-driven diagnosis.

💼 LinkedIn Post — Day 15/100

Day 15/100 — Linux Troubleshooting Basics 🔧🐧

A server can be running and still have a serious problem.

An application might be slow.

A service might have stopped.

The disk might be full.

A port might not be listening.

Or the application could be perfectly fine while the actual problem is somewhere in the network.

So the real question isn't:

"What command should I run?"

It's:

"How do I systematically find the root cause?"

Today I practiced the basics of Linux troubleshooting.

Some of the commands I worked with:

🔹 uptime — system uptime and load
🔹 top — CPU, memory and processes
🔹 free -h — memory usage
🔹 df -h — disk usage
🔹 ps — identify resource-heavy processes
🔹 systemctl — check services
🔹 ss — check listening ports
🔹 ip — inspect networking
🔹 curl — test services
🔹 journalctl — investigate logs

The troubleshooting flow I'm taking away is:

Observe
   ↓
Measure
   ↓
Check resources
   ↓
Check services
   ↓
Check networking
   ↓
Check logs
   ↓
Identify the root cause
   ↓
Fix + verify

The biggest lesson today:

Don't guess. Collect evidence.

This is one of the habits I want to carry forward as I move from Linux into networking, Git, Docker, cloud, Kubernetes, and CI/CD.

15 days completed. 🚀

Day 15/100 ✅

Next up: Networking Fundamentals 🌐

#100DaysOfDevOps #DevOps #Linux #LinuxAdministration #Troubleshooting #Networking #Cloud #Automation #DevOpsJourney #LearningInPublic

🔜 Tomorrow — Day 16/100

We officially start the Networking section.

Day 16 — Networking Fundamentals 🌐

We'll learn the foundations behind:

Network → Devices → IP → Packets → Protocols → Ports → Services

This will give you the networking foundation you'll need before moving into Docker, Azure, Kubernetes,
