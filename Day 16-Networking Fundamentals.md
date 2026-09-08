Day 16/100 — Networking Fundamentals

Welcome to Day 16 of your restarted 100 Days of DevOps — From Zero to Hero journey.

Today we officially begin the Networking section of your roadmap.

Networking is one of the most important foundations in DevOps. Later, when you work with Docker, Azure, Kubernetes, CI/CD, and cloud infrastructure, you'll constantly deal with networks, IPs, ports, DNS, and connectivity.

🎯 Today's Learning Objectives

By the end of Day 16, you should understand:

What a computer network is
LAN vs WAN
Client and server
Network devices
IP addresses at a basic level
Packets
Protocols
Ports
How a request travels from your computer to a server
Basic networking commands
1. What Is a Network?

A network is a group of devices that can communicate with each other.

For example, your home network might look like:

              Internet
                  │
                  ▼
             Wi-Fi Router
             /          \
            /            \
       Laptop            Phone
          │
          │
       Printer

All these devices can communicate through the network.

In a DevOps environment, the network might look like:

                 Internet
                     │
                     ▼
                Load Balancer
                     │
              ┌──────┴──────┐
              ▼             ▼
          Web Server     Web Server
              │             │
              └──────┬──────┘
                     ▼
                Application
                     │
                     ▼
                  Database
2. Why Networking Matters in DevOps

Consider a simple application:

User
 ↓
Browser
 ↓
Internet
 ↓
Load Balancer
 ↓
Application Server
 ↓
Database

If the application doesn't work, the problem could be anywhere.

For example:

DNS isn't resolving
Server has no IP
Firewall blocks traffic
Port isn't open
Application isn't listening
Route is incorrect
Database isn't reachable

That's why networking knowledge is essential for DevOps engineers.

3. LAN vs WAN
LAN — Local Area Network

A LAN connects devices in a relatively small area.

Examples:

Home network
Office network
School network
Data-center network

Example:

Laptop ──┐
Phone  ──┼── Router
Printer ─┘
WAN — Wide Area Network

A WAN connects networks over larger geographical areas.

The biggest example is the Internet.

Conceptually:

Office LAN
    │
    ▼
Internet
    │
    ▼
Cloud Network
    │
    ▼
Server
4. Client and Server

This is an important concept.

Client

The client requests something.

Examples:

Web browser
Mobile application
curl
SSH client
Server

The server provides a service.

Examples:

Web server
Database server
SSH server
DNS server

Example:

Client                         Server

Browser  ───── Request ──────► Web Server
         ◄──── Response ──────

When you type:

https://example.com

your browser acts as the client and communicates with a web server.

5. Network Devices

You don't need to master all networking hardware today, but understand these basics.

Switch

Connects devices within a network.

PC ──┐
PC ──┼── Switch
PC ──┘
Router

Connects different networks.

LAN
 │
 ▼
Router
 │
 ▼
Internet
Firewall

Controls which network traffic is allowed or blocked.

Internet
   │
   ▼
Firewall
   │
   ▼
Server

Firewalls become particularly important in Azure and Kubernetes.

6. What Is an IP Address?

An IP address identifies a device/interface on an IP network.

For example:

192.168.1.10

You can think of it like an address used for network communication.

We'll study IP addressing in much more detail on Day 17.

For now, remember:

Devices need network addresses so they can communicate.

7. What Is a Packet?

When you send data over a network, the data is broken into smaller pieces called packets.

Imagine sending:

"Hello Server"

Conceptually:

Original Data
     ↓
┌───────┬───────┬───────┐
│Packet1│Packet2│Packet3│
└───────┴───────┴───────┘
     ↓
     Network
     ↓
   Server
     ↓
Reassembled Data

Real networking is much more sophisticated, but this mental model is useful.

8. What Is a Protocol?

A protocol is a set of rules that defines how systems communicate.

Examples:

Protocol	Common Purpose
HTTP	Web communication
HTTPS	Secure web communication
SSH	Secure remote access
DNS	Domain-name resolution
TCP	Reliable transport
UDP	Connectionless transport
ICMP	Network diagnostic/control messages

We'll explore these throughout the Networking section.

9. What Is a Port?

An IP address identifies the machine/interface.

A port identifies a service/application endpoint on that machine.

Think:

IP Address
    +
Port
    ↓
Specific Service

For example:

192.168.1.10:22

could represent SSH.

And:

192.168.1.10:80

could represent HTTP.

Some commonly encountered ports:

Port	Typical Service
22	SSH
53	DNS
80	HTTP
443	HTTPS
3306	MySQL
5432	PostgreSQL
8080	Common application/web port

Don't worry about memorizing all of them today.

10. How a Web Request Works

Suppose you enter:

https://example.com

A simplified flow is:

Browser
   │
   │ DNS lookup
   ▼
DNS Server
   │
   │ IP address
   ▼
Browser
   │
   │ HTTPS request
   ▼
Internet
   │
   ▼
Web Server
   │
   │ Response
   ▼
Browser

Later we'll break this process down much more deeply.

🧪 Hands-On Practical Lab

Today we'll use your Linux machine to inspect its networking environment.

Step 1 — Check Your Network Interfaces

Run:

ip addr

or:

ip a

Look for something similar to:

2: eth0:
    inet 192.168.1.20/24

Your interface might instead be named:

ens33
enp0s3
wlan0

depending on your environment.

Step 2 — Display Your IP Address

Run:

hostname -I

Example:

192.168.1.20

Your address will probably be different.

Step 3 — Check Your Routing Table

Run:

ip route

Example:

default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0

The important part:

default via 192.168.1.1

is your default route/gateway.

Step 4 — Test Network Connectivity

Run:

ping -c 4 8.8.8.8

You may see:

4 packets transmitted, 4 received, 0% packet loss

This tells you that your machine can communicate with that destination using ICMP, assuming ICMP isn't blocked.

Step 5 — Test DNS

Run:

nslookup google.com

You should receive an IP address for the domain.

Then try:

ping -c 4 google.com

This combines:

DNS resolution
       +
Network connectivity
Step 6 — Check Listening Ports

Run:

sudo ss -tuln

If SSH is enabled, you may see:

LISTEN ... :22

If Nginx is running, you may see:

LISTEN ... :80

This gives you a practical connection between Day 13/14/15 and today's networking fundamentals.

Step 7 — Test HTTP

Run:

curl -I https://example.com

You should receive HTTP headers.

For example:

HTTP/2 200
content-type: text/html
🔍 Put Everything Together

Run:

hostname
hostname -I
ip route
ping -c 4 8.8.8.8
nslookup google.com
sudo ss -tuln
curl -I https://example.com

You're effectively performing a basic network health check.

🧠 Expected Results

You should be able to identify:

Your machine
Hostname
IP address
Your network
Default gateway
Connectivity
Ping response
DNS
Domain → IP
Services
Listening ports
Application communication
HTTP response
🛠️ Troubleshooting
ip command not found

On most modern Linux systems this should already be available.

On Ubuntu:

sudo apt update
sudo apt install iproute2
ping doesn't work

Try:

ping -c 4 8.8.8.8

If it fails, check:

ip addr
ip route

Remember that some networks block ICMP, so failed ping does not always mean the network is down.

nslookup doesn't exist

Install:

sudo apt install dnsutils

Then:

nslookup google.com
No ports appear in ss

That's possible if there aren't services currently listening.

Check SSH:

sudo systemctl status ssh
curl fails

Check basic connectivity first:

ping -c 4 8.8.8.8

Then DNS:

nslookup example.com

Then retry:

curl -I https://example.com

This is an example of layer-by-layer troubleshooting.

📸 What Screenshot Should You Capture?

For LinkedIn, I'd recommend a clean terminal screenshot containing:

hostname -I
ip route
ping -c 4 8.8.8.8
nslookup google.com
curl -I https://example.com

The screenshot should ideally demonstrate:

IP Address
    ↓
Gateway
    ↓
Connectivity
    ↓
DNS Resolution
    ↓
HTTP Response

Avoid exposing sensitive infrastructure information such as public IPs or internal server details.

💡 Day 16 Key Takeaway

Today is about building the mental model before diving deeper into IP addressing.

Remember:

Device
  ↓
Network Interface
  ↓
IP Address
  ↓
Network
  ↓
Router
  ↓
Internet
  ↓
Destination Server
  ↓
Port
  ↓
Application

And remember this distinction:

IP tells you where the machine is.
Port tells you which service you're trying to reach.

That concept will become extremely important when we get to Docker and Kubernetes.

💼 LinkedIn Post — Day 16/100

Day 16/100 — Networking Fundamentals 🌐

When an application isn't working, one of the first questions a DevOps engineer needs to answer is:

Is the application actually the problem?

It could be the network.

Maybe the server doesn't have the expected IP.

Maybe the route is wrong.

Maybe DNS isn't resolving.

Maybe the required port isn't listening.

Or maybe the application is working perfectly, but something between the client and server is blocking the traffic.

Today I started the Networking section of my 100 Days of DevOps journey.

I worked through the fundamentals:

🔹 LAN vs WAN
🔹 Client vs Server
🔹 Routers, switches and firewalls
🔹 IP addresses
🔹 Packets
🔹 Network protocols
🔹 Ports and services

I also practiced some basic Linux networking commands:

ip addr
ip route
ping
ss
nslookup
curl

One simple concept that stood out:

IP address tells us where we're connecting.
Port tells us which service we're trying to reach.

For example:

192.168.1.20:22

The IP identifies the destination machine, while port 22 is commonly used for SSH.

I'm beginning to see how the pieces of this journey are connecting:

Linux → SSH → Networking → Cloud → Containers → Kubernetes

The goal isn't just to memorize commands.

It's to understand what is happening underneath when two systems communicate.

Day 16/100 ✅

Tomorrow: Understanding IP Addresses 📡

#100DaysOfDevOps #DevOps #Networking #Linux #Cloud #IPAddresses #NetworkEngineering #Automation #DevOpsJourney #LearningInPublic

🔜 Day 17 — Understanding IP Addresses

Tomorrow we'll go deeper into IPv4 addresses, including:

192.168.1.10

We'll understand:

IPv4 structure
Network portion
Host portion
Subnets
CIDR notation
/24, /16, /8
Private IP ranges

This is a very important foundation before we move into Azure networking and Kubernetes networking.
