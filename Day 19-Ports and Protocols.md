Day 19/100 — Ports and Protocols

Welcome to Day 19/100 of your restarted 100 Days of DevOps — From Zero to Hero journey.

We’ve now covered:

Day 16: Networking Fundamentals
Day 17: IP Addresses
Day 18: Public vs Private IP
Day 19: Ports and Protocols

Today we'll connect the pieces:

IP Address + Port + Protocol
              ↓
       Application
       Communication

This is one of the most important networking concepts you'll need for Linux, Docker, Azure, Kubernetes, and CI/CD.

🎯 Today's Learning Objectives

By the end of today, you should understand:

What a network port is
What a protocol is
TCP vs UDP
Common ports
HTTP vs HTTPS
SSH
How applications listen on ports
How to identify listening ports on Linux
How to test ports and services
1. What Is a Port?

An IP address identifies a network interface.

A port identifies a particular service/application endpoint.

Think of it like this:

IP Address = Building address
Port       = Door number

For example:

192.168.1.20:22

means:

192.168.1.20 → destination IP
22           → destination port

Port 22 is commonly associated with SSH.

Another example:

192.168.1.20:80

Port 80 is commonly associated with HTTP.

2. Why Do We Need Ports?

Imagine a server running several services:

              Linux Server
           192.168.1.20
                 │
       ┌─────────┼─────────┐
       │         │         │
       ▼         ▼         ▼
      SSH       HTTP      App
      :22        :80      :8080

The IP gets the traffic to the machine.

The port helps direct it to the appropriate service.

3. Common Ports

Here are some ports you'll encounter frequently:

Port	Protocol/Service	Typical Purpose
22	SSH	Remote administration
53	DNS	Name resolution
80	HTTP	Web traffic
443	HTTPS	Secure web traffic
25	SMTP	Email transfer
3306	MySQL	Database
5432	PostgreSQL	Database
6379	Redis	In-memory database
8080	HTTP/app	Common application port

⚠️ A port number doesn't magically guarantee what application is using it. A service can often be configured to use a different port.

4. What Is a Protocol?

A protocol defines rules for communication between systems.

Some important protocols:

HTTP
HTTPS
SSH
TCP
UDP
DNS
ICMP

For example:

Browser
   │
 HTTPS
   │
   ▼
Web Server

The protocol determines how the communication works.

5. TCP vs UDP

Two important transport protocols are:

TCP

TCP is connection-oriented and provides mechanisms for reliable, ordered delivery.

A simplified model:

Client
  │
  │ Connection
  ▼
Server
  │
  │ Data
  ▼
Client

TCP is commonly used by:

HTTP/HTTPS
SSH
Many database connections
UDP

UDP is connectionless and has lower protocol overhead, but does not provide TCP's built-in guarantees for reliable, ordered delivery.

Common examples include:

DNS queries
Streaming/real-time applications
Some monitoring and gaming traffic

Think:

TCP → reliability-oriented
UDP → lightweight/low-overhead

Don't reduce the distinction to simply "TCP is fast/UDP is slow." The choice depends on the application's requirements.

6. HTTP and HTTPS
HTTP

HTTP commonly uses:

TCP port 80

Example:

http://example.com
HTTPS

HTTPS commonly uses:

TCP port 443

Example:

https://example.com

HTTPS adds encryption and authentication through TLS.

Simplified:

Client
  │
  │ HTTPS :443
  ▼
Web Server
7. SSH

SSH is one of the most important protocols for Linux and DevOps.

Default port:

TCP 22

Example:

ssh user@192.168.1.20

Conceptually:

Your Computer
      │
      │ SSH
      │ TCP :22
      ▼
Linux Server

You'll use this concept repeatedly when working with Linux servers and cloud VMs.

🧪 Hands-On Lab

Let's inspect your Linux machine.

Step 1 — List Listening TCP/UDP Ports

Run:

sudo ss -tuln

Example:

Netid State  Local Address:Port
tcp   LISTEN 0.0.0.0:22

This tells you that something is listening on TCP port 22.

Step 2 — Understand the Options

The command:

ss -tuln

means:

-t → TCP
-u → UDP
-l → Listening
-n → Numeric output

So you're essentially asking:

"Show me TCP/UDP ports currently listening on this machine."

Step 3 — Check SSH

Run:

sudo ss -tuln | grep ':22'

If SSH is listening, you'll see a line containing:

:22

You can also check the service:

sudo systemctl status ssh

Expected:

Active: active (running)
Step 4 — Check HTTP

If you've installed Nginx from the previous Linux labs, run:

sudo ss -tuln | grep ':80'

If Nginx is running, you may see:

:80

Then test it:

curl -I http://localhost

A successful response might look like:

HTTP/1.1 200 OK
Server: nginx
Step 5 — Test HTTPS

Run:

curl -I https://example.com

You should receive an HTTP response such as:

HTTP/2 200

or another valid HTTP status.

Now you've tested:

HTTPS
  ↓
Port 443
  ↓
Web Server
Step 6 — Identify Which Process Owns a Port

Run:

sudo ss -tulpn

The -p option displays process information when available.

You may see something similar to:

users:(("sshd",pid=1234,fd=3))

or:

users:(("nginx",pid=5678,fd=6))

This is extremely useful when troubleshooting.

Instead of asking:

"What is using port 8080?"

you can investigate directly.

🔎 Step 7 — Search for a Specific Port

For example:

sudo ss -tulpn | grep ':8080'

If something is listening on port 8080, you'll see it.

If there is no output:

then nothing matching that port is currently listening.

🧪 Practical Challenge

Suppose a developer says:

"My application is running on port 8080, but I can't access it."

Use this troubleshooting sequence.

1. Is anything listening?
sudo ss -tulpn | grep ':8080'
2. Is the application process running?
ps aux | grep 8080
3. Can the server access the application locally?
curl http://localhost:8080
4. Check the service

If it's managed by systemd:

sudo systemctl status SERVICE_NAME
5. Check logs
sudo journalctl -u SERVICE_NAME -n 50
6. If local access works but remote access doesn't

Investigate:

Firewall
Cloud security rules
Network routing
Security groups/NSGs
Whether the application is bound only to 127.0.0.1

That last point is particularly important.

🔥 127.0.0.1 vs 0.0.0.0

Suppose an application listens on:

127.0.0.1:8080

It is accessible locally:

Same Server
     ↓
127.0.0.1:8080

but may not accept connections from other machines.

If it listens on:

0.0.0.0:8080

it is generally listening on all IPv4 interfaces, subject to firewall and application configuration.

This is a common source of confusion when deploying applications.

🧠 Important DevOps Connection

Consider a Docker container running:

Application
    │
    ▼
Container :8080

You might publish it to the host:

docker run -p 8080:8080 myapp

which conceptually creates:

Host :8080
     │
     ▼
Container :8080

We'll study this in detail when we reach Docker.

Similarly, in Kubernetes you'll encounter:

Pod Port
   ↓
Container Port
   ↓
Service Port
   ↓
Node/Ingress

So today's lesson is directly preparing you for those technologies.

🛠️ Troubleshooting Guide
Port isn't listening

Check:

sudo ss -tulpn | grep ':PORT'

Then determine whether the application/service is running.

Service is running but port isn't available

Check:

sudo systemctl status SERVICE_NAME

Then inspect logs:

sudo journalctl -u SERVICE_NAME -n 50

The service might have failed to bind to the port because:

Another process already uses it
Configuration is wrong
Permission problems
Application startup failure
Port works locally but not remotely

Test locally:

curl http://localhost:8080

If that works, investigate:

Firewall
   ↓
Cloud security rules
   ↓
Network routing
   ↓
Application bind address
curl says connection refused

Usually this means there is no service accepting the connection at that IP/port, though other network/security conditions can also affect the result.

Check:

sudo ss -tulpn | grep ':8080'
📸 What to Capture for LinkedIn

For today's practical screenshot, run:

sudo ss -tulpn

Then, if you have Nginx:

curl -I http://localhost

A strong screenshot could show:

:22    → SSH
:80    → HTTP

and:

HTTP/1.1 200 OK

For an additional troubleshooting screenshot:

sudo ss -tulpn | grep ':8080'

showing an application port if you have one available.

Don't expose sensitive server information, public IPs, usernames, or infrastructure details in your LinkedIn screenshot.

🧠 Day 19 Key Takeaways

The most important mental model today is:

IP Address
     +
Port
     +
Protocol
     ↓
Specific Communication

For example:

192.168.1.20:22
       │    │
       │    └── SSH port
       │
       └─────── Server IP

And:

example.com
     ↓
DNS
     ↓
IP Address
     ↓
TCP :443
     ↓
HTTPS
     ↓
Web Server

That's the beginning of understanding how real applications communicate.

💼 LinkedIn Post — Day 19/100

Day 19/100 — Ports and Protocols 🔌🌐

Today I learned an important networking concept that appears everywhere in DevOps:

An IP address alone isn't enough.

To communicate with a specific application, we also need to understand ports and protocols.

For example:

192.168.1.20:22

Here:

IP address → identifies the destination

Port 22 → identifies the service endpoint, commonly SSH

I also learned about commonly used ports:

22    → SSH
53    → DNS
80    → HTTP
443   → HTTPS
3306  → MySQL
5432  → PostgreSQL
8080  → Common application port

And the difference between two important transport protocols:

TCP → connection-oriented and reliability-focused

UDP → connectionless and lightweight, without TCP's built-in reliability guarantees

For the practical lab, I used Linux commands such as:

sudo ss -tuln
sudo ss -tulpn
curl -I http://localhost
curl -I https://example.com

One troubleshooting pattern I found especially useful:

Application not reachable
        ↓
Is the process running?
        ↓
Is the port listening?
        ↓
Can I access it locally?
        ↓
Is the firewall blocking it?
        ↓
Is the application bound correctly?

This also helped me understand why the same concepts will become important later with:

Docker port mapping → Azure networking → Kubernetes Services → CI/CD infrastructure

The more I learn networking, the more I realize that many "application problems" are actually communication problems.

Day 19/100 ✅

Tomorrow: TCP vs UDP & the OSI Model 🚀

#100DaysOfDevOps #DevOps #Networking #Linux #TCP #UDP #Ports #Protocols #Cloud #Docker #Kubernetes #DevOpsJourney #LearningInPublic
