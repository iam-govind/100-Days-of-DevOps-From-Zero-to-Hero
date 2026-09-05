Day 14/100 — Linux Networking Commands

Today we move from SSH and remote access into the networking commands you’ll use constantly when troubleshooting Linux servers.

When an application is not reachable, a DevOps engineer needs to answer questions like:

Does the server have an IP address?
Can I reach another machine?
Is a port listening?
Can the server reach the internet?
Is DNS working?
Can I access a website from the server?

Linux provides simple commands to investigate all of these.

🎯 Today's Learning Objectives

By the end of Day 14, you should understand:

ip
ping
ss
curl
wget
traceroute
nslookup
Basic network troubleshooting
1. Understanding Linux Networking

A simplified view:

Your Linux Server
       │
       ├── Network Interface
       │       ↓
       │    IP Address
       │
       ├── Gateway
       │       ↓
       │    Internet
       │
       └── DNS
               ↓
          Domain → IP

Each command helps investigate a different part of this path.

2. ip — Inspect Network Configuration

Start with:

ip addr

or the shorter version:

ip a

You'll see interfaces such as:

lo
eth0
ens33
enp0s3

Look for a line similar to:

inet 192.168.1.20/24

That's an IP address assigned to the interface.

Find only your IP addresses
hostname -I

Example:

192.168.1.20
3. Check Your Routing Table

Run:

ip route

You may see:

default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link

The important part is:

default via 192.168.1.1

This is your default gateway.

It is generally where traffic destined for networks outside your local network is sent.

🧪 Hands-On Lab

We'll perform a small network troubleshooting exercise.

Step 1 — Check Your IP
ip addr

Then:

hostname -I
Expected result

You should see one or more IP addresses.

Step 2 — Check Your Default Gateway
ip route

Look for:

default via <gateway-ip>

For example:

default via 192.168.1.1 dev eth0
4. ping — Test Connectivity

Try:

ping -c 4 8.8.8.8

This sends four packets to Google's public DNS server.

You may see:

64 bytes from 8.8.8.8: icmp_seq=1 ...

At the end:

4 packets transmitted, 4 received, 0% packet loss

That indicates successful connectivity to that destination.

Why -c 4?

Without it:

ping 8.8.8.8

may continue indefinitely.

-c 4 tells Linux to send four packets and stop.

5. Test DNS Separately

Now try:

ping -c 4 google.com

If this works, you're testing both:

DNS resolution
      +
Network connectivity

This gives us a useful troubleshooting comparison.

For example:

ping 8.8.8.8     → works
ping google.com  → fails

could indicate a DNS problem rather than a basic network connectivity problem.

6. ss — Check Listening Ports

One of the most useful commands for DevOps troubleshooting is:

ss -tuln

Meaning:

-t → TCP
-u → UDP
-l → Listening
-n → Numeric addresses/ports

You might see:

LISTEN 0 128 0.0.0.0:22

This tells you something is listening on TCP port 22, commonly SSH.

If you installed Nginx during the previous package-management lab, you may also see:

0.0.0.0:80

Port 80 is commonly used for HTTP.

Check a Specific Port

For SSH:

ss -tuln | grep :22

For HTTP:

ss -tuln | grep :80

This is extremely useful when troubleshooting:

"The application is running, but I can't connect to it."

You can first ask:

Is anything actually listening on the expected port?

7. curl — Test HTTP Services

Run:

curl -I https://example.com

You should receive HTTP headers similar to:

HTTP/2 200
content-type: text/html

You can also retrieve the page:

curl https://example.com

This is particularly useful on servers where there is no graphical browser.

Local test

If Nginx is installed:

curl -I http://localhost

You should get an HTTP response.

8. wget — Download Files

Try:

wget https://example.com

wget is commonly used to download files from the command line.

For example:

wget https://example.com/index.html

You can then check:

ls
9. nslookup — DNS Troubleshooting

Try:

nslookup google.com

You'll see information similar to:

Name: google.com
Address: ...

This allows you to check whether a domain name can be resolved to an IP address.

If ping 8.8.8.8 works but:

nslookup google.com

fails, you should investigate DNS configuration.

10. traceroute — See the Network Path

Install it if necessary:

sudo apt update
sudo apt install traceroute

Then:

traceroute google.com

You'll see multiple hops between your machine and the destination.

Conceptually:

Your Server
    ↓
Gateway
    ↓
ISP
    ↓
Internet Routers
    ↓
Destination

⚠️ Some networks block or limit traceroute responses, so incomplete output does not automatically mean your network is broken.

🔥 Quick Command Cheat Sheet
Command	Purpose
ip addr	Show network interfaces/IP addresses
hostname -I	Show local IP addresses
ip route	Show routing table
ping	Test connectivity
ss -tuln	Show listening ports
curl	Test HTTP/HTTPS services
wget	Download files
nslookup	Test DNS resolution
traceroute	Trace network path
🧪 Mini Troubleshooting Challenge

Imagine you have a web application running on:

192.168.1.50:8080

But you cannot access it.

Here's a logical troubleshooting sequence:

1. Check your network
ip addr
2. Check the route
ip route
3. Check whether the server responds
ping -c 4 192.168.1.50
4. Check whether port 8080 is reachable/listening

On the server:

ss -tuln | grep :8080
5. Test the application
curl http://192.168.1.50:8080
6. If using a hostname, test DNS
nslookup your-domain.com

This is the beginning of a very important DevOps habit:

Don't guess. Troubleshoot layer by layer.

🛠️ Common Problems
ping: command not found

Install:

sudo apt install iputils-ping
nslookup: command not found

Install:

sudo apt install dnsutils
traceroute: command not found

Install:

sudo apt install traceroute
curl cannot connect

Check whether the destination service is running:

systemctl status nginx

Then check listening ports:

ss -tuln

Also verify the server firewall and, later in the course, cloud network security rules.

ping doesn't work

Don't immediately conclude that the server is down.

ICMP traffic can be blocked by firewalls or network policies.

Instead, test the actual service:

curl http://SERVER_IP:PORT
📸 What to Capture for LinkedIn

For today's post, capture one clean terminal screenshot containing:

hostname -I
ip route
ss -tuln
ping -c 4 8.8.8.8
nslookup google.com

A particularly good screenshot would show:

IP address
     ↓
Default gateway
     ↓
Listening ports
     ↓
Successful connectivity
     ↓
Successful DNS resolution

⚠️ Before posting, hide any public IP addresses, usernames, hostnames, or other infrastructure details you don't want to expose.

💡 Day 14 Key Takeaway

Today isn't really about memorizing commands.

It's about learning how to think when a server has a networking problem.

A useful mental model is:

Do I have an IP?
      ↓
Do I have a route?
      ↓
Can I reach the destination?
      ↓
Can DNS resolve the name?
      ↓
Is the required port listening?
      ↓
Does the application respond?

That troubleshooting mindset will become increasingly important as we move into cloud, containers, Kubernetes, and CI/CD.

💼 LinkedIn Post — Day 14/100

Day 14/100 — Linux Networking Commands 🌐

One of the biggest challenges when working with servers is that an application can be "running" and still be completely unreachable.

So when something doesn't work, where do you start?

Today I focused on the basics of Linux networking troubleshooting.

I practiced:

🔹 ip addr — checking network interfaces and IP addresses
🔹 ip route — checking the routing table
🔹 ping — testing connectivity
🔹 ss — checking listening ports
🔹 curl — testing HTTP services
🔹 wget — downloading files
🔹 nslookup — troubleshooting DNS
🔹 traceroute — understanding the network path

One thing I found particularly useful was thinking about troubleshooting step by step instead of immediately assuming the application is broken.

For example:

Do I have an IP?
       ↓
Do I have a route?
       ↓
Can I reach the server?
       ↓
Does DNS work?
       ↓
Is the port listening?
       ↓
Does the application respond?

This simple approach can save a lot of time when troubleshooting real servers.

I'm starting to see how the Linux fundamentals I've been learning are connecting together:

Linux → SSH → Networking → DevOps

And this is exactly the kind of foundation I want to build before moving into Docker, cloud, Kubernetes, and CI/CD.

Day 14/100 completed. 🚀

Tomorrow: Linux Troubleshooting Basics 🔧

#100DaysOfDevOps #DevOps #Linux #Networking #LinuxNetworking #Cloud #Automation #DevOpsJourney #LearningInPublic #100DaysChallenge

🔜 Tomorrow — Day 15/100

We'll wrap up the Linux section with Linux Troubleshooting Basics.

We'll bring together everything you've learned so far:

commands + processes + services + packages + SSH + networking

and use them to approach common server problems systematically.
