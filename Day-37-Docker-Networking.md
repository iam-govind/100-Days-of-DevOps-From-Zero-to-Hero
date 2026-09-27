# Day 37 — Docker Networking

## 100 Days of DevOps — From Zero to Hero

**Day:** 37/100  
**Topic:** Docker Networking  
**Category:** Docker

---

## 1. What Problem Are We Solving?

Imagine you have two containers:

```text
+------------------+       +------------------+
|   Web Container  |       | Database Container|
|                  |       |                  |
|     Nginx        |       |      MySQL       |
+------------------+       +------------------+
```

The application running in the web container needs to communicate with the database container.

But how?

Containers are isolated environments. Docker networking provides the mechanism that allows containers to communicate with:

- Other containers
- The host machine
- External networks
- The internet

The basic idea is:

```text
Container A
     |
     | Docker Network
     |
     v
Container B
```

---

# 2. What is Docker Networking?

Docker networking allows containers to communicate with each other and with external systems.

Docker provides networking features such as:

- Container-to-container communication
- Container-to-host communication
- Port publishing
- DNS-based service discovery
- Network isolation

A simplified architecture:

```text
                    Host Machine
                         |
                  Docker Network
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Web Container          App Container
             |                       |
             +-----------+-----------+
                         |
                         v
                  Database Container
```

---

# 3. Why Do We Need Docker Networking?

Consider a typical application:

```text
User
 |
 v
Web Server
 |
 v
Application
 |
 v
Database
```

Each component could run inside its own container:

```text
Browser
   |
   v
+---------+
|   Web   |
|Container|
+---------+
    |
    v
+---------+
|  App    |
|Container|
+---------+
    |
    v
+---------+
|Database |
|Container|
+---------+
```

Docker networking allows these containers to communicate without putting everything into a single container.

This gives us better:

- Isolation
- Scalability
- Manageability
- Service separation

---

# 4. Docker Network Drivers

Docker supports several network drivers.

The most commonly encountered ones are:

| Driver | Purpose |
|---|---|
| `bridge` | Common networking for containers on a single Docker host |
| `host` | Container shares the host's network stack |
| `none` | Disables networking |
| `overlay` | Networking across multiple Docker hosts, commonly with Swarm |
| `macvlan` | Gives containers network interfaces that appear directly on the physical network |

For today's beginner lab, we will focus mainly on:

**bridge networks and custom bridge networks.**

---

# 5. The Default Bridge Network

Docker normally creates a default network called:

```text
bridge
```

Check your Docker networks:

```bash
docker network ls
```

Typical output:

```text
NETWORK ID     NAME      DRIVER    SCOPE
xxxxxxxxxxxx   bridge    bridge    local
xxxxxxxxxxxx   host      host      local
xxxxxxxxxxxx   none      null      local
```

The exact IDs can be different on your machine.

---

# 6. Inspect the Default Network

Run:

```bash
docker network inspect bridge
```

This shows information about the network, including:

- Network ID
- Driver
- Subnet
- Gateway
- Connected containers
- Configuration

Docker creates a virtual network environment on the host.

---

# 7. What is a Bridge Network?

A bridge network allows containers on the same Docker host to communicate.

Conceptually:

```text
              Docker Host
                   |
            +--------------+
            | Bridge       |
            | Network      |
            +--------------+
              /          \
             /            \
            v              v
      Container A     Container B
```

For example:

```text
Web Container  <---->  App Container
```

---

# 8. Port Publishing

There is an important difference between:

**Container networking**

and

**Publishing a container port to the host.**

Suppose Nginx listens on port 80 inside a container.

If we run:

```bash
docker run -d --name web nginx
```

Nginx may be listening on:

```text
Container:80
```

But this does not automatically mean that you can access it through:

```text
Host:80
```

To publish the port:

```bash
docker run -d --name web -p 8080:80 nginx
```

The format is:

```text
-p HOST_PORT:CONTAINER_PORT
```

So:

```text
-p 8080:80
```

means:

```text
Host Port 8080
       |
       v
Container Port 80
```

---

# 9. Port Publishing Diagram

```text
Browser
   |
   | http://localhost:8080
   v
Docker Host
   |
   | Port 8080
   v
Container
   |
   | Port 80
   v
Nginx
```

This is how we can access a containerized web application from the host.

---

# 10. Hands-On Lab 1 — Run Nginx with a Published Port

## Step 1 — Pull Nginx

```bash
docker pull nginx
```

---

## Step 2 — Run Nginx

```bash
docker run -d --name web-server -p 8080:80 nginx
```

Expected output:

```text
<container-id>
```

---

## Step 3 — Check the Container

```bash
docker ps
```

You should see something similar to:

```text
CONTAINER ID   IMAGE   COMMAND                  PORTS
xxxxxxxxxxxx   nginx   "/docker-entrypoint..."  0.0.0.0:8080->80/tcp
```

---

## Step 4 — Test the Web Server

Open your browser:

```text
http://localhost:8080
```

You should see the Nginx welcome page.

You can also test with:

```bash
curl http://localhost:8080
```

You should receive HTML output from Nginx.

---

# 11. Check the Network

Run:

```bash
docker inspect web-server
```

Look for the networking information.

You can also run:

```bash
docker network inspect bridge
```

You should see the `web-server` container connected to the default bridge network.

---

# 12. Hands-On Lab 2 — Create a Custom Network

Although the default bridge network works, custom networks are very useful.

Create one:

```bash
docker network create devops-network
```

Expected output:

```text
<network-id>
```

Verify:

```bash
docker network ls
```

You should see:

```text
devops-network
```

---

# 13. Inspect the Custom Network

Run:

```bash
docker network inspect devops-network
```

You will see details such as:

- Driver
- Subnet
- Gateway
- Containers
- Network configuration

---

# 14. Run Two Containers on the Custom Network

Run an Nginx container:

```bash
docker run -d \
  --name web \
  --network devops-network \
  nginx
```

Run an Alpine container:

```bash
docker run -it \
  --name client \
  --network devops-network \
  alpine sh
```

Now we have:

```text
       devops-network
             |
       +-----+-----+
       |           |
       v           v
      web        client
    Nginx        Alpine
```

---

# 15. Container-to-Container Communication

Inside the `client` container, install curl:

```bash
apk add --no-cache curl
```

Then run:

```bash
curl http://web
```

You should receive the Nginx HTML response.

This is an important Docker networking concept.

The container can reach another container using its **container name**:

```text
http://web
```

Instead of needing to manually remember the container IP address.

---

# 16. Docker DNS

Docker provides DNS-based service discovery on user-defined networks.

For example:

```text
web container
     |
     | DNS name: web
     v
Nginx container
```

When the client container runs:

```bash
curl http://web
```

Docker resolves:

```text
web
```

to the IP address of the appropriate container on the Docker network.

This is one reason custom Docker networks are extremely useful.

---

# 17. Why Container Names Matter

Suppose the web container gets an IP address:

```text
172.x.x.x
```

That IP can change when the container is recreated.

Hard-coding the IP is therefore not a good approach.

Instead:

```text
curl http://web
```

Docker handles the name resolution.

Conceptually:

```text
Application
     |
     | web
     v
Docker DNS
     |
     v
Container IP
```

---

# 18. Test the Network

Inside the `client` container:

```bash
curl http://web
```

If you receive Nginx HTML, container-to-container communication is working.

You can also try:

```bash
ping web
```

Depending on the Alpine image and installed utilities, `ping` may not be available by default.

If needed:

```bash
apk add --no-cache iputils
```

Then:

```bash
ping -c 3 web
```

---

# 19. Connect an Existing Container to a Network

You can connect an already-running container to a network.

Example:

```bash
docker network connect devops-network web-server
```

Inspect:

```bash
docker network inspect devops-network
```

You should now see the container connected.

---

# 20. Disconnect a Container

To disconnect:

```bash
docker network disconnect devops-network web-server
```

Verify:

```bash
docker network inspect devops-network
```

---

# 21. Run a Container with No Network

Docker also supports the `none` network.

Example:

```bash
docker run --rm --network none alpine ip addr
```

The container has no normal external network connectivity.

This can be useful when network access should be deliberately restricted.

---

# 22. Host Networking

Docker also has a `host` network mode.

Example:

```bash
docker run --rm --network host nginx
```

With host networking, the container shares the host's network stack.

This behaves differently from the normal bridge networking model.

Host networking is platform-dependent, so always understand how Docker implements it on your operating system before using it in production.

---

# 23. Common Docker Networking Commands

### List networks

```bash
docker network ls
```

### Inspect a network

```bash
docker network inspect devops-network
```

### Create a network

```bash
docker network create devops-network
```

### Remove a network

```bash
docker network rm devops-network
```

### Connect a container

```bash
docker network connect devops-network web
```

### Disconnect a container

```bash
docker network disconnect devops-network web
```

### Show container networking information

```bash
docker inspect web
```

---

# 24. Troubleshooting

## Problem 1 — Cannot access Nginx from the browser

Check:

```bash
docker ps
```

Look for:

```text
0.0.0.0:8080->80/tcp
```

If the port was not published, recreate the container:

```bash
docker rm -f web-server
docker run -d --name web-server -p 8080:80 nginx
```

---

## Problem 2 — `curl http://web` does not work

Make sure both containers are connected to the same custom network:

```bash
docker network inspect devops-network
```

You should see both:

```text
web
client
```

---

## Problem 3 — Container name cannot be resolved

Check the network:

```bash
docker inspect web
```

Make sure the container is connected to the expected user-defined network.

You can reconnect it:

```bash
docker network connect devops-network web
```

---

## Problem 4 — Port already in use

If you see an error that port `8080` is already in use, choose another host port:

```bash
docker run -d --name web-server-2 -p 8081:80 nginx
```

Then access:

```text
http://localhost:8081
```

---

# 25. Hands-On Challenge

Try building this yourself:

```text
              devops-network
                    |
          +---------+---------+
          |                   |
          v                   v
       web-app             client
        Nginx              Alpine
```

### Requirements

1. Create `devops-network`.
2. Start an Nginx container called `web-app`.
3. Start an Alpine container called `client`.
4. Put both containers on `devops-network`.
5. Enter the `client` container.
6. Install curl.
7. Run:

```bash
curl http://web-app
```

8. Confirm that the Nginx page is returned.

If you can complete this without looking at the previous commands, you have understood the core concept.

---

# 26. Real-World DevOps Example

A typical application may look like:

```text
                    Internet
                       |
                       v
                +-------------+
                | Load Balancer|
                +-------------+
                       |
                       v
                +-------------+
                | Web Container|
                +-------------+
                       |
                       v
                +-------------+
                | App Container|
                +-------------+
                       |
                       v
                +-------------+
                | DB Container |
                +-------------+
```

Each service can be separated while still communicating through appropriate networks.

In larger environments, Kubernetes and cloud networking provide more advanced networking capabilities.

---

# 27. Docker Networking vs Port Publishing

These concepts are often confused.

### Container-to-container communication

```text
Container A
     |
     v
Docker Network
     |
     v
Container B
```

### Host-to-container communication

```text
Host
 |
 | Published Port
 v
Container
```

For example:

```bash
-p 8080:80
```

means:

```text
Host:8080 → Container:80
```

It is not the same thing as container-to-container communication.

---

# 28. Key Takeaways

Today I learned:

- Docker networking allows containers to communicate.
- Docker provides several network drivers.
- `bridge` is a common Docker network.
- User-defined bridge networks are useful for application communication.
- Containers on the same user-defined network can communicate by container name.
- Docker provides DNS-based name resolution on user-defined networks.
- `-p HOST:CONTAINER` publishes a container port to the host.
- `docker network create` creates a custom network.
- `docker network inspect` helps troubleshoot networking.
- Container IP addresses can change, so applications should avoid depending on hard-coded container IPs.

The most important concept:

```text
Container Name
      |
      v
Docker DNS
      |
      v
Container IP
      |
      v
Target Container
```

---

# 29. Screenshot Guidance for LinkedIn

Capture these screenshots from your lab.

## Screenshot 1 — Docker Networks

Run:

```bash
docker network ls
```

Capture the output showing:

```text
bridge
host
none
devops-network
```

---

## Screenshot 2 — Custom Network

Run:

```bash
docker network inspect devops-network
```

Capture the section showing your connected containers.

---

## Screenshot 3 — Container-to-Container Communication

Inside the `client` container:

```bash
curl http://web
```

Capture the Nginx response.

This is the most useful screenshot because it demonstrates that two containers are communicating through the Docker network.

---

# 30. LinkedIn Post — Day 37/100

## Docker Networking

Today I learned something that initially looked simple but is actually an important part of working with containers:

**How do Docker containers communicate with each other?**

Imagine having:

```text
Web Container
      ↓
App Container
      ↓
Database Container
```

If these services are running in separate containers, they still need a way to communicate.

That's where Docker Networking comes in.

Today I practiced:

✅ Listing Docker networks

✅ Understanding the default bridge network

✅ Creating a custom Docker network

✅ Connecting multiple containers

✅ Communicating between containers

✅ Using container names for communication

✅ Understanding Docker DNS

✅ Publishing container ports

One of the most useful things I learned today was that containers on a user-defined Docker network can communicate using their container names.

For example:

```bash
curl http://web
```

Instead of depending on a hard-coded container IP.

The basic flow:

```text
Client Container
       |
       v
Docker Network
       |
       v
Docker DNS
       |
       v
Web Container
```

Another important distinction:

**Container-to-container communication is different from publishing a port to the host.**

For example:

```bash
-p 8080:80
```

means:

```text
Host:8080 → Container:80
```

Day 37 complete. 🚀

Next up:

**Day 38 — Environment Variables in Docker**

#100DaysOfDevOps #DevOps #Docker #DockerNetworking #Containers #Cloud #DevOpsJourney #LearningInPublic #DockerCompose #DevOpsEngineer

---

# 31. GitHub Commit

After saving this file in your repository:

```bash
git status
```

Add it:

```bash
git add Day-37-Docker-Networking.md
```

Commit:

```bash
git commit -m "Add Day 37 Docker Networking"
```

Push:

```bash
git push origin main
```

---

# 32. Day 38 Preview

Tomorrow:

## Day 38 — Environment Variables in Docker

We will learn:

- What environment variables are
- Why applications use them
- `docker run -e`
- `--env-file`
- Environment variables in containers
- Separating configuration from application code
- Practical configuration lab

Example:

```text
Application
     |
     v
Environment Variables
     |
     +---- DATABASE_HOST
     +---- DATABASE_PORT
     +---- APP_ENV
     +---- API_KEY
```

---

## Progress

**37/100 Days Completed 🚀**

```text
[███████████████████░░░░░░░░░░░] 37%
```

Keep learning. Keep practicing. Keep building.
