Day 18/100 — Public IP vs Private IP

Good morning! 🌅

Today we continue your restarted 100 Days of DevOps — From Zero to Hero journey with Day 18.

Yesterday we learned how IP addresses work. Today we're answering a very practical question:

What's the difference between a Private IP and a Public IP?

This concept becomes extremely important when you start working with Azure VMs, Virtual Networks, Docker, Kubernetes, load balancers, and cloud infrastructure.

🎯 Today's Learning Objectives

By the end of today, you should understand:

What a private IP address is
What a public IP address is
Common private IP ranges
Why private IPs aren't directly routable across the public Internet
What NAT does
How your laptop accesses the Internet using a private IP
How cloud servers use private and public IPs
How to identify your local IP and public IP
1. Private IP Address

A private IP address is used inside a private network.

Examples:

192.168.1.10
10.0.0.5
172.16.10.20

Private IPs are commonly used for:

Home networks
Office networks
Internal servers
Cloud virtual networks
Databases
Internal applications

For example:

          Home Network
               │
       ┌───────┴───────┐
       │               │
    Laptop           Phone
192.168.1.10      192.168.1.20

These devices can communicate with each other using their private IP addresses.

2. Private IPv4 Ranges

The three major private IPv4 ranges are:

Range	CIDR
10.0.0.0 – 10.255.255.255	10.0.0.0/8
172.16.0.0 – 172.31.255.255	172.16.0.0/12
192.168.0.0 – 192.168.255.255	192.168.0.0/16

Examples:

10.10.10.10
172.16.5.20
192.168.1.100

These addresses are intended for private networking.

3. Public IP Address

A public IP address is an address used for communication over the public Internet.

For example, a website might be reachable through a public IP.

Conceptually:

Your Computer
     │
Private IP
     │
     ▼
Router / NAT
     │
Public IP
     │
     ▼
Internet
     │
     ▼
Web Server

Your home devices generally don't each need their own public IPv4 address.

Instead, your router performs NAT.

4. What Is NAT?

NAT = Network Address Translation

NAT allows private addresses to communicate with external networks by translating between private and public addressing.

For example:

Laptop
192.168.1.10
     │
     ▼
Home Router
     │
     │ NAT
     ▼
Public IP
     │
     ▼
Internet

From the Internet, the connection appears to come from the public-facing address of the router/network rather than directly from 192.168.1.10.

5. Why Do We Need Private IPs?

Imagine an organization with:

10,000 servers

They don't necessarily need 10,000 public IPv4 addresses.

Instead, they can build a private network:

10.0.0.0/8

and use private addresses internally:

10.0.1.10
10.0.1.11
10.0.2.10
10.0.2.11
...

Only systems that need direct Internet-facing connectivity need appropriate public addressing or another supported Internet access mechanism.

This is one reason private networking is fundamental to cloud architecture.

6. Public vs Private — Simple Comparison
Feature	Private IP	Public IP
Used inside private networks	✅	Not primarily
Routable across public Internet	❌	✅
Common in home networks	✅	Usually router/network level
Common in cloud internal networks	✅	✅ when Internet-facing
Example	192.168.1.10	Publicly assigned address
Internet-facing	Usually no	Yes
🧪 Hands-On Lab

Let's investigate the networking on your own machine.

Step 1 — Find Your Private IP

Run:

hostname -I

You may see:

192.168.1.25

or:

10.0.0.15

This is likely a private IP if it falls into one of the RFC 1918 ranges.

You can also use:

ip -br addr
Step 2 — Find Your Default Gateway

Run:

ip route

You might see:

default via 192.168.1.1 dev eth0

Your machine sends traffic destined for other networks toward the default gateway.

Step 3 — Find Your Public IP

One simple way is:

curl -4 ifconfig.me

You may receive an address such as:

203.x.x.x

Don't post your actual public IP publicly.

Another option:

curl -4 https://icanhazip.com
Step 4 — Compare Them

You may now have something like:

Private IP:
192.168.1.25

Public IP:
203.x.x.x

Conceptually:

Linux Machine
192.168.1.25
      │
      ▼
Router / NAT
      │
      ▼
Public IP
203.x.x.x
      │
      ▼
Internet

This is one of the most useful practical demonstrations of today's lesson.

Step 5 — Check Your DNS

Run:

nslookup google.com

This demonstrates another important networking concept:

Domain Name
     ↓
DNS
     ↓
IP Address

We'll explore DNS more deeply later in the Networking section.

Step 6 — Test Internet Connectivity

Run:

ping -c 4 8.8.8.8

Then:

curl -I https://example.com

You are testing two different things:

ping
 ↓
Basic IP connectivity

curl
 ↓
Application-level HTTP/HTTPS communication
☁️ Cloud Example

Imagine an Azure VM.

A simplified architecture might look like:

                 Internet
                    │
             Public IP
                    │
             ┌──────▼──────┐
             │ Azure VM    │
             │             │
             │ Private IP  │
             │ 10.0.1.10   │
             └─────────────┘

The VM can have:

Private IP
10.0.1.10

for internal communication.

It may also have a public IP association for Internet-facing connectivity, depending on the architecture.

Later, when we reach Day 42+ Azure networking, this concept will become much more concrete.

🔥 Real-World DevOps Example

Suppose you have:

Internet
   │
   ▼
Load Balancer
   │
   ├──────────┐
   ▼          ▼
Web Server  Web Server
10.0.1.10   10.0.1.11
   │          │
   └────┬─────┘
        ▼
    Database
    10.0.2.10

Notice something important:

The database doesn't necessarily need to be publicly accessible.

It can communicate through private networking.

This is a much safer architecture than exposing every server directly to the Internet.

🧠 Important Security Concept

A private IP address does not automatically make a system secure.

Security still depends on:

Firewalls
Network Security Groups
Access controls
Routing
Authentication
Application security
Network architecture

We'll explore these concepts later in Azure and DevSecOps.

🛠️ Troubleshooting
curl ifconfig.me doesn't work

Try:

curl -4 https://icanhazip.com

If neither works, test Internet connectivity:

ping -c 4 8.8.8.8
Your IP doesn't look like 192.168.x.x

That's completely fine.

It may be:

10.x.x.x

or:

172.16.x.x → 172.31.x.x

depending on your network.

You see 127.0.0.1

That's the loopback address.

It means:

This machine itself

It isn't your normal LAN address.

Public IP and private IP appear to be the same

This can happen in certain environments, such as some cloud, VPN, or directly assigned public-network configurations.

Don't assume every machine must have both a private and public IPv4 address.

📸 What Screenshot Should You Capture?

For today's LinkedIn post, create a terminal screenshot showing:

hostname -I
ip route
curl -4 ifconfig.me

You can structure the terminal output as:

Private IP:
192.168.1.25

Default Gateway:
192.168.1.1

Public IP:
[HIDDEN]

For LinkedIn, I strongly recommend masking your actual public IP.

A better screenshot can show:

Private IP → Router/NAT → Internet

without exposing sensitive network details.

🧠 Day 18 Key Takeaways

Remember:

Private IP
     │
     ▼
Internal Network
     │
     ▼
NAT / Router
     │
     ▼
Public IP
     │
     ▼
Internet

And the three private IPv4 ranges:

10.0.0.0/8
172.16.0.0/12
192.168.0.0/16

The most important concept:

Private IPs are primarily used for internal communication, while public IPs provide Internet-routable addressing.

And one more important DevOps lesson:

Not every server should be publicly accessible.

Keeping databases and internal services on private networks is a common architectural pattern.

💼 LinkedIn Post — Day 18/100

Day 18/100 — Public IP vs Private IP 🌐

When I first started looking at IP addresses, I wondered:

Why does my computer have an IP like 192.168.x.x while websites on the Internet use completely different addresses?

Today I learned the difference between private IPs and public IPs.

A private IP is used inside a private network.

Common private ranges include:

10.0.0.0/8
172.16.0.0/12
192.168.0.0/16

A public IP, on the other hand, is used for communication across the public Internet.

The interesting part is how these work together.

A typical home network looks something like:

Laptop
192.168.1.25
     ↓
Router / NAT
     ↓
Public IP
     ↓
Internet

I also practiced checking my network configuration using:

hostname -I
ip route
curl -4 ifconfig.me

One important concept I took away today:

Not every server needs to be publicly accessible.

For example, in a cloud environment you might have:

Internet
    ↓
Load Balancer
    ↓
Web Servers
    ↓
Application
    ↓
Database

The database can remain on a private network instead of being directly exposed to the Internet.

That's a simple concept, but it starts connecting networking with security and cloud architecture.

I'm looking forward to seeing how this works in Azure when I reach the cloud section of this journey.

Day 18/100 ✅

Tomorrow: Ports and Protocols 🔌

#100DaysOfDevOps #DevOps #Networking #IPAddress #PrivateIP #PublicIP #Cloud #Azure #CyberSecurity #DevOpsJourney #LearningInPublic

🔜 Tomorrow — Day 19/100
🔌 Ports and Protocols

We'll learn how:

IP Address
     +
Port
     +
Protocol
     ↓
Application Communication

works.

We'll explore TCP, UDP, HTTP, HTTPS, SSH, and why a server can be reachable by IP but still have a particular application inaccessible.
