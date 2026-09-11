Day 20/100 — TCP, UDP & OSI Model

Welcome to Day 20 of your restarted 100 Days of DevOps — From Zero to Hero journey.

Your Networking section so far:

Day 16: Networking Fundamentals
Day 17: Understanding IP Addresses
Day 18: Public IP vs Private IP
Day 19: Ports and Protocols
Day 20: TCP, UDP & OSI Model

Today we're going one level deeper into how network communication actually works.

🎯 Today's Learning Objectives

By the end of Day 20, you should understand:

What TCP is
What UDP is
TCP vs UDP
What the OSI model is
The 7 OSI layers
How data moves through the layers
Where IP, TCP, UDP, HTTP, HTTPS and Ethernet fit
Basic Linux commands for troubleshooting network connections
Why the OSI model is useful for DevOps
1. Why Do We Need Layers?

Imagine opening a website:

https://example.com

A lot happens behind the scenes.

Your computer needs to:

Understand the application request
Establish communication
Find the destination IP
Send packets
Put them onto the local network
Physically transmit the data

Trying to understand all of this as one giant process would be complicated.

So networking is organized into layers.

This is where the OSI Model comes in.

2. What Is the OSI Model?

OSI = Open Systems Interconnection

The OSI model divides network communication into 7 conceptual layers.

┌─────────────────────────┐
│ 7. Application          │
├─────────────────────────┤
│ 6. Presentation         │
├─────────────────────────┤
│ 5. Session              │
├─────────────────────────┤
│ 4. Transport            │
├─────────────────────────┤
│ 3. Network              │
├─────────────────────────┤
│ 2. Data Link            │
├─────────────────────────┤
│ 1. Physical             │
└─────────────────────────┘

You don't need to memorize the model perfectly today.

The important thing is understanding what each layer represents.

3. The Seven OSI Layers
Layer 7 — Application

This is closest to the applications users interact with.

Examples:

HTTP
HTTPS
DNS
SSH
SMTP

Example:

Browser
   ↓
HTTPS
Layer 6 — Presentation

This layer is concerned conceptually with things such as:

Data representation
Encoding
Encryption/decryption
Compression

Examples often associated with this layer include:

TLS/SSL
Data encoding
Compression

In modern networking, these responsibilities don't always map cleanly to a single OSI layer, but the conceptual model is useful.

Layer 5 — Session

This layer deals conceptually with establishing, managing, and terminating communication sessions.

For example:

Start session
     ↓
Maintain session
     ↓
End session

In modern TCP/IP implementations, session responsibilities are often handled across multiple layers rather than by a distinct "Session Layer" protocol.

Layer 4 — Transport

This is where:

TCP
UDP

fit into the OSI model.

The Transport layer deals with end-to-end communication between applications.

It also uses ports.

For example:

Server: 192.168.1.20
Port: 443
Protocol: TCP
4. TCP

TCP = Transmission Control Protocol

TCP is connection-oriented and provides mechanisms for:

Reliable delivery
Ordered data
Retransmission
Flow control
Congestion control

A simplified TCP communication flow:

Client                    Server

   │                        │
   │──── Connection ───────►│
   │                        │
   │──── Data ─────────────►│
   │                        │
   │◄──── Acknowledgment ───│
   │                        │
5. TCP Three-Way Handshake

Before normal TCP data transfer, a connection is established using a three-way handshake.

Conceptually:

Client                    Server
  │                         │
  │        SYN              │
  │────────────────────────►│
  │                         │
  │       SYN + ACK         │
  │◄────────────────────────│
  │                         │
  │        ACK              │
  │────────────────────────►│
  │                         │
  │     Connection Ready    │

The three messages are:

SYN
SYN-ACK
ACK

You don't need to memorize packet internals yet.

Just remember:

TCP establishes a connection before transferring normal application data.

6. UDP

UDP = User Datagram Protocol

UDP is connectionless.

It doesn't provide TCP's built-in mechanisms for:

Reliable delivery
Ordering
Retransmission

That makes UDP lightweight and useful where low overhead or application-controlled reliability is desirable.

Examples include:

DNS
Real-time communication
Streaming-related traffic
Gaming

Conceptually:

Client
  │
  │ Datagram
  ├──────────────────► Server
  │
  │ Datagram
  ├──────────────────► Server

There is no TCP-style connection establishment first.

7. TCP vs UDP
Feature	TCP	UDP
Connection-oriented	✅	❌
Connectionless	❌	✅
Reliable delivery mechanism	✅	❌
Ordered delivery	✅	❌
Retransmission	✅	❌
Lower overhead	❌	✅
Uses ports	✅	✅
Simple mental model:
TCP → reliability + ordering

UDP → lightweight + application-controlled behavior
8. Layer 3 — Network

The Network layer is responsible for logical addressing and routing.

The major protocol you should associate with this layer is:

IP

For example:

192.168.1.20

Routers use IP addressing to determine where packets should go.

Conceptually:

Computer
192.168.1.10
     │
     ▼
 Router
     │
     ▼
Network
     │
     ▼
Server
192.168.1.20
9. Layer 2 — Data Link

The Data Link layer handles communication over a local network.

Important concepts include:

Ethernet
MAC addresses
Frames
Switches

For example:

IP Address
    ↓
MAC Address
    ↓
Ethernet Frame

Switches primarily operate at this layer.

10. Layer 1 — Physical

This is the physical transmission of bits.

Examples:

Ethernet cable
Fiber
Radio/Wi-Fi signals
Network hardware

At this level, we're dealing with actual transmission of signals/bits.

11. OSI Model — Easy Reference

Here's the version I recommend remembering:

Layer	Name	Examples / Concepts
7	Application	HTTP, HTTPS, DNS, SSH
6	Presentation	Encryption, encoding, compression
5	Session	Session management
4	Transport	TCP, UDP, Ports
3	Network	IP, Routing
2	Data Link	Ethernet, MAC, Switches
1	Physical	Cables, Fiber, Radio
🧠 Easy Way to Remember the Layers

From Layer 7 down:

Application
Presentation
Session
Transport
Network
Data Link
Physical

A commonly used mnemonic is:

All People Seem To Need Data Processing

12. TCP/IP Model

In real DevOps work, you'll more frequently encounter the TCP/IP model rather than strictly thinking in seven OSI layers.

A simplified TCP/IP model is:

┌──────────────────────────┐
│ Application              │
├──────────────────────────┤
│ Transport                │
├──────────────────────────┤
│ Internet                 │
├──────────────────────────┤
│ Link / Network Access    │
└──────────────────────────┘

Rough mapping:

OSI                    TCP/IP

Application ────────┐
Presentation         ├── Application
Session ────────────┘

Transport ───────────── Transport

Network ────────────── Internet

Data Link ───────────┐
Physical ────────────┴── Link

The OSI model is excellent for learning and troubleshooting, while the TCP/IP model more closely reflects how modern Internet networking is commonly described.

🌐 13. What Happens When You Open a Website?

Suppose you visit:

https://example.com

A simplified flow is:

Browser
   │
   ▼
HTTPS
   │
   ▼
TCP :443
   │
   ▼
IP
   │
   ▼
Ethernet / Wi-Fi
   │
   ▼
Internet
   │
   ▼
Web Server

You can map this to the layers:

Application → HTTPS
Transport   → TCP :443
Network     → IP
Data Link   → Ethernet/Wi-Fi
Physical    → Cable/Radio

This is the mental model you want to build.

🧪 Hands-On Practical Lab

Let's use Linux to investigate these concepts.

Step 1 — Check Your IP
ip -br addr

You may see:

eth0    UP    192.168.1.20/24

This gives us the Layer 3 addressing information.

Step 2 — Check Routing
ip route

Example:

default via 192.168.1.1 dev eth0

This tells you the default route used to reach other networks.

Step 3 — Check Listening TCP/UDP Ports
sudo ss -tuln

Remember:

-t → TCP
-u → UDP
-l → Listening
-n → Numeric
Step 4 — Check Processes Behind Ports

Run:

sudo ss -tulpn

This adds process information where available.

For example:

users:(("sshd",pid=1234,fd=3))
Step 5 — Test TCP/HTTPS

Run:

curl -I https://example.com

This tests application-level HTTP communication over HTTPS.

Conceptually:

HTTPS
 ↓
TCP
 ↓
IP
 ↓
Network
Step 6 — Test DNS

Run:

nslookup google.com

Or:

dig google.com

If dig isn't installed:

sudo apt install dnsutils

DNS is a great example of how application-level protocols rely on lower-level networking.

Step 7 — Inspect the Network Path

Try:

traceroute example.com

If it isn't installed:

sudo apt install traceroute

Then:

traceroute example.com

Depending on your environment, some hops may not respond.

You can also try:

tracepath example.com
🔥 Optional: See TCP Connections

Run:

ss -tan

This shows TCP sockets.

You may see states such as:

ESTAB
TIME-WAIT
LISTEN

For example:

State   Local Address    Peer Address
ESTAB   192.168.1.20     ...

This gives you a real look at TCP connections on your machine.

🧪 Day 20 Challenge

Try answering these without looking back:

Question 1

Which layer uses IP?

Answer:

Layer 3 — Network
Question 2

Which layer uses TCP and UDP?

Answer:

Layer 4 — Transport
Question 3

Which layer deals with MAC addresses and Ethernet?

Answer:

Layer 2 — Data Link
Question 4

Which protocol commonly uses port 22?

Answer:

SSH
Question 5

Which protocol commonly uses port 443?

Answer:

HTTPS
🔥 Real-World Troubleshooting Example

Imagine:

"The website isn't working."

Instead of guessing, think layer by layer.

Layer 7 — Application

Does the application respond?

curl -I https://example.com
Layer 4 — Transport

Is the expected port listening?

sudo ss -tulpn | grep ':443'
Layer 3 — Network

Do we have an IP and route?

ip addr
ip route
Layer 2

Is the network interface operational?

ip link
Layer 1

Is there a physical/network connection?

For a physical server, investigate the NIC, cable, switch, etc.

This gives you a structured troubleshooting approach.

🐳 DevOps Connection — Docker

Docker networking will build on these same concepts.

For example:

Host
 │
 │ Port 8080
 ▼
Docker Network
 │
 ▼
Container
 │
 │ Port 8080
 ▼
Application

You'll eventually need to understand:

Container IPs
Container ports
Host ports
Bridge networks
DNS between containers
☁️ DevOps Connection — Azure

Azure networking will use concepts such as:

Virtual Network
      ↓
Subnet
      ↓
Private IP
      ↓
Network Security Group
      ↓
VM
      ↓
Application Port

Understanding today's layers will make those concepts much easier.

☸️ DevOps Connection — Kubernetes

Kubernetes networking becomes even more interesting:

Client
   ↓
Ingress
   ↓
Service
   ↓
Pod
   ↓
Container

Underneath all of this are the same fundamental concepts:

IP
Ports
TCP/UDP
Routing
Network interfaces
🛠️ Troubleshooting Guide
traceroute: command not found

Install:

sudo apt update
sudo apt install traceroute

Then:

traceroute example.com
dig: command not found

Install:

sudo apt install dnsutils

Then:

dig google.com
ping works but curl doesn't

This is an important troubleshooting case.

It means basic IP connectivity may exist, but the application path could still have a problem.

Check:

curl -I https://example.com

and:

sudo ss -tulpn

Also investigate DNS, firewall rules, proxy settings, or TLS/application-level issues.

Port isn't listening

Run:

sudo ss -tulpn | grep ':PORT'

Then check the application/service.

📸 What Screenshot Should You Capture?

For LinkedIn, I'd recommend running:

ip -br addr

then:

ip route

then:

sudo ss -tulpn

and finally:

curl -I https://example.com

Your screenshot can demonstrate:

IP Address
     ↓
Routing
     ↓
TCP/UDP Ports
     ↓
HTTPS Response

This is better than posting a screenshot containing only one command.

Remember to hide sensitive IP addresses, usernames, hostnames, or infrastructure information before posting.

🧠 Day 20 — Key Takeaways

The most important model to remember:

Application
   ↓
TCP / UDP
   ↓
IP
   ↓
Ethernet / Wi-Fi
   ↓
Physical Network

Or using OSI:

7  Application     HTTP / HTTPS / DNS / SSH
6  Presentation    Encoding / Encryption
5  Session         Session management
4  Transport       TCP / UDP / Ports
3  Network         IP / Routing
2  Data Link       Ethernet / MAC
1  Physical        Cable / Fiber / Radio

And the most important troubleshooting principle:

When something doesn't work, think in layers instead of guessing.

💼 LinkedIn Post — Day 20/100

Day 20/100 — TCP, UDP & the OSI Model 🌐

Today I went deeper into networking by learning how network communication is organized into layers.

Until now, I had been looking at concepts such as:

IP → Port → Protocol

Today I started understanding what happens underneath.

The OSI model gives us seven conceptual layers:

7  Application
6  Presentation
5  Session
4  Transport
3  Network
2  Data Link
1  Physical

The layers I found most important for DevOps were:

Application → HTTP / HTTPS / DNS / SSH
Transport   → TCP / UDP / Ports
Network     → IP / Routing
Data Link   → Ethernet / MAC

I also explored the difference between TCP and UDP.

TCP focuses on connection-oriented, reliable and ordered delivery.

UDP is connectionless and lightweight, leaving reliability and ordering decisions to the application when needed.

I practiced with Linux commands such as:

ip -br addr
ip route
sudo ss -tulpn
curl -I https://example.com
nslookup google.com
traceroute example.com

One thing that really clicked for me today:

When an application isn't working, I shouldn't immediately start changing configurations.

I can troubleshoot layer by layer:

Application
     ↓
Transport
     ↓
Network
     ↓
Data Link
     ↓
Physical

This mindset will become extremely useful when I start working with:

Docker networking → Azure networking → Kubernetes networking

I'm realizing that learning DevOps isn't just about learning tools.

It's about understanding the infrastructure underneath those tools.

Day 20/100 ✅

Tomorrow: Git & GitHub — Introduction to Git and GitHub 🚀

#100DaysOfDevOps #DevOps #Networking #TCP #UDP #OSI #Linux #Git #GitHub #Cloud #Docker #Kubernetes #DevOpsJourney #LearningInPublic

🎉 Networking Section Complete

With Day 20, you've completed the Networking fundamentals portion of the roadmap:

Day 16 → Networking Fundamentals
Day 17 → Understanding IP Addresses
Day 18 → Public IP vs Private IP
Day 19 → Ports and Protocols
Day 20 → TCP, UDP & OSI Model
🔜 Day 21 — Introduction to Git and GitHub

Tomorrow we move into Git & GitHub.

We'll start from the basics:

Code
 ↓
Git
 ↓
Version History
 ↓
GitHub
 ↓
Collaboration

You'll also perform your first practical Git workflow rather than just learning the theory.
