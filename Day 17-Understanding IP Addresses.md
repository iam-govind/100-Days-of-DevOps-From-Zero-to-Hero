Day 17/100 — Understanding IP Addresses

Welcome to Day 17 of your restarted 100 Days of DevOps — From Zero to Hero journey.

Yesterday we started Networking Fundamentals. Today we go one level deeper into one of the most important networking concepts:

IP addresses

You'll encounter IP addresses everywhere in DevOps—Linux servers, Azure VMs, Docker containers, Kubernetes services, load balancers, databases, and DNS.

🎯 Today's Learning Objectives

By the end of Day 17, you should understand:

What an IP address is
IPv4 structure
Network portion vs host portion
IPv4 octets
Subnet masks
CIDR notation
What /24, /16, and /8 mean
Basic subnet concepts
How to inspect IP addresses on Linux
1. What Is an IP Address?

An IP address is a network address used to identify an interface/device on an IP network.

For example:

192.168.1.10

Think of it like an address:

House Address → identifies a house
IP Address    → identifies a network interface

When one machine wants to communicate with another, the destination IP helps determine where the traffic should go.

2. IPv4 Addresses

The most familiar type is IPv4.

Example:

192.168.1.10

IPv4 addresses contain 32 bits.

They're normally written as four decimal numbers called octets:

192 . 168 . 1 . 10
 │     │    │    │
 ▼     ▼    ▼    ▼
 8     8    8    8 bits

Therefore:

8 + 8 + 8 + 8 = 32 bits

Each octet can have a value from:

0 → 255

So:

192.168.1.10

is a valid IPv4 address.

3. Network Portion vs Host Portion

An IP address doesn't work alone.

We also need to know which part identifies the network and which part identifies the host.

For example:

192.168.1.10/24

With /24:

Network portion        Host portion
----------------       ------------
192.168.1              .10

Conceptually:

192.168.1.10
└─────────┘└─┘
 Network   Host

The /24 tells us how many bits belong to the network portion.

4. What Does /24 Mean?

CIDR notation is commonly used to describe networks.

For example:

192.168.1.0/24

/24 means:

24 network bits
8 host bits

Because IPv4 has 32 bits:

32 - 24 = 8

So there are 8 bits available for hosts.

That gives:

2^8 = 256

total addresses in the address space.

Traditionally, in a basic IPv4 subnet, the first address is the network address and the last is the broadcast address, leaving:

256 - 2 = 254

usable host addresses.

5. Common CIDR Notations

You'll frequently encounter:

CIDR	Network Bits	Host Bits
/8	8	24
/16	16	16
/24	24	8
/25	25	7
/26	26	6
/27	27	5
/28	28	4

For now, focus primarily on:

/8
/16
/24

You'll use CIDR notation heavily when we reach Azure Virtual Networks.

6. Example: /24

Consider:

192.168.10.0/24

The network is:

192.168.10.0

The traditional broadcast address is:

192.168.10.255

Typical usable host addresses:

192.168.10.1
      ↓
192.168.10.254

So:

Network
192.168.10.0

Hosts
192.168.10.1 → 192.168.10.254

Broadcast
192.168.10.255
7. Private IP Addresses

Some IPv4 ranges are reserved for private networks.

The major private ranges are:

10.0.0.0/8
172.16.0.0/12
192.168.0.0/16

Examples:

10.0.0.5
172.16.10.20
192.168.1.50

These are commonly used inside:

Home networks
Corporate networks
Cloud networks
Virtual machines
Containers

We'll study Public IP vs Private IP tomorrow on Day 18.

🧪 Hands-On Lab

Let's inspect the IP configuration of your Linux machine.

Step 1 — Display Network Interfaces

Run:

ip addr

or:

ip a

Look for:

inet 192.168.x.x/24

or possibly:

inet 10.x.x.x/24

depending on your environment.

Step 2 — Get a Cleaner IP Output

Run:

hostname -I

Example:

192.168.1.20
Step 3 — Identify the Interface

Run:

ip -br addr

Example:

lo       UNKNOWN  127.0.0.1/8
eth0     UP       192.168.1.20/24

This is an easy-to-read view of your network interfaces.

8. Understand Your CIDR

Suppose you see:

192.168.1.20/24

The /24 tells you that the machine belongs to the:

192.168.1.0/24

network.

You can verify the route with:

ip route

You may see something similar to:

192.168.1.0/24 dev eth0
default via 192.168.1.1 dev eth0

Now you can connect three concepts:

IP Address
192.168.1.20

Network
192.168.1.0/24

Gateway
192.168.1.1
9. Test Two Hosts on the Same Network

If you have another device on your local network, identify its private IP.

For example:

Laptop → 192.168.1.20
Phone  → 192.168.1.30

From Linux, try:

ping -c 4 192.168.1.30

If ICMP is allowed, you should receive responses.

This demonstrates two devices communicating using IP addresses.

10. Try a Different Network

Now test Google's public DNS:

ping -c 4 8.8.8.8

Notice the difference:

192.168.1.20

is a typical private address.

While:

8.8.8.8

is a public IP address.

We'll explore this distinction in detail tomorrow.

🔢 CIDR Practice

Try these examples yourself.

Example 1
10.0.0.0/8

Host bits:

32 - 8 = 24

Total address space:

2^24 = 16,777,216
Example 2
172.16.0.0/16

Host bits:

32 - 16 = 16

Total:

2^16 = 65,536
Example 3
192.168.1.0/24

Host bits:

32 - 24 = 8

Total:

2^8 = 256

Traditional usable host count:

256 - 2 = 254
🛠️ Troubleshooting
Problem: No IP address appears

Run:

ip link

Check whether your interface is:

UP

You can also run:

ip addr
Problem: hostname -I returns nothing useful

Check:

ip addr

Look for an interface other than:

lo

The lo interface is the local loopback interface.

Problem: Ping doesn't work

Check your route:

ip route

Then:

ping -c 4 8.8.8.8

Remember: a failed ping doesn't always prove that the destination is unreachable because ICMP may be blocked.

Problem: You see 127.0.0.1

That's the loopback address.

It refers back to your own machine.

127.0.0.1
    ↓
This computer

It isn't normally the address other machines use to reach your server.

📸 What to Capture for LinkedIn

For today's screenshot, run:

ip -br addr

and:

ip route

A good screenshot might show:

eth0    UP    192.168.1.20/24

192.168.1.0/24 dev eth0
default via 192.168.1.1 dev eth0

This visually demonstrates:

IP address + CIDR + network + gateway

You can also include:

ping -c 4 8.8.8.8

showing successful connectivity.

⚠️ Before posting, consider masking private/public IP information if it reveals details about your actual network.

🧠 Day 17 Key Takeaways

Remember these fundamentals:

IPv4
 ↓
32 bits
 ↓
4 octets
 ↓
0–255 per octet

And:

192.168.1.20/24
       │       │
       │       └── CIDR prefix
       │
       └────────── IP address

The /24 tells us:

24 network bits
8 host bits

Most importantly:

An IP address identifies an interface, while CIDR tells us how that address fits into a network.

💼 LinkedIn Post — Day 17/100

Day 17/100 — Understanding IP Addresses 📡

Yesterday I started learning networking fundamentals.

Today I went one level deeper into something that appears everywhere in DevOps:

IP addresses.

At first, an address like:

192.168.1.20/24

can look like just another number to memorize.

But understanding what it actually means makes networking much easier.

Today I learned:

🔹 IPv4 addresses use 32 bits
🔹 IPv4 is written as four octets
🔹 Each octet ranges from 0–255
🔹 IP addresses identify network interfaces
🔹 CIDR notation describes the network size
🔹 /24 means 24 network bits and 8 host bits
🔹 Private IP ranges are commonly used inside internal networks

I also practiced using Linux commands such as:

ip addr
ip -br addr
ip route
hostname -I
ping

One concept that really helped me:

192.168.1.20/24
      │       │
      │       └── Network prefix
      │
      └────────── IP address

And a simple mental model:

IP address → Where is the device?

Port → Which service do I want?

I'm starting to see how these networking fundamentals will connect later with:

Linux → Azure → Docker → Kubernetes → CI/CD

The goal isn't just to remember commands.

It's to understand what is happening when systems communicate.

Day 17/100 ✅

Tomorrow: Public IP vs Private IP 🌐

#100DaysOfDevOps #DevOps #Networking #Linux #IPAddress #IPv4 #CIDR #Cloud #Azure #DevOpsJourney #LearningInPublic

🔜 Day 18 — Public IP vs Private IP

Tomorrow we'll answer an important question:

Why can my Linux machine have a private IP while websites on the Internet have public IPs?

We'll explore:

Private IP addresses
Public IP addresses
NAT
Internet communication
Real-world cloud examples
Why Azure VMs often use both public and private networking concepts.
