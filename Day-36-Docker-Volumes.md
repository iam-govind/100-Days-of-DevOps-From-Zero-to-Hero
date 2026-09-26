# Day 36/100 — Docker Volumes

## 100 Days of DevOps — From Zero to Hero

Welcome to **Day 36** of the **100 Days of DevOps — From Zero to Hero** journey.

Yesterday we learned how to build Docker images and run containers.

Today we will answer an important question:

> **What happens to our data when a Docker container is deleted?**

The answer leads us to one of the most important Docker concepts:

# Docker Volumes

---

# 1. Today's Topic

**Day 36 — Docker Volumes**

Today we will learn:

- Why container data can disappear
- What persistent data means
- What Docker volumes are
- How to create a volume
- How to attach a volume to a container
- How to store data inside a volume
- How data survives container deletion
- What bind mounts are
- Volumes vs bind mounts
- How to inspect and manage volumes
- A practical hands-on lab
- Troubleshooting common volume problems

---

# 2. The Problem: Containers Are Ephemeral

A Docker container is designed to be replaceable.

For example:

```text
Container
   │
   ├── Application
   ├── Temporary data
   └── Files
```

If we remove the container:

```bash
docker rm my-container
```

the writable layer belonging to that container is removed.

That means data stored only inside the container's writable layer should not be treated as persistent application data.

The typical DevOps pattern is:

```text
Container
   │
   │
   ▼
Persistent Storage
```

Docker provides several ways to manage this storage.

---

# 3. Container Storage Without a Volume

Let's see the problem first.

Run:

```bash
docker run -it --name test-container ubuntu:24.04 bash
```

Inside the container:

```bash
echo "Hello Docker Storage" > /data.txt
```

Check:

```bash
cat /data.txt
```

Expected:

```text
Hello Docker Storage
```

Exit:

```bash
exit
```

Now remove the container:

```bash
docker rm test-container
```

Create another container:

```bash
docker run -it --name test-container ubuntu:24.04 bash
```

Try:

```bash
cat /data.txt
```

You will get something similar to:

```text
cat: /data.txt: No such file or directory
```

Why?

Because the file existed inside the previous container.

The previous container was removed.

Therefore, the data was removed with it.

---

# 4. Persistent Storage

If application data needs to survive container deletion, we should store it outside the container's writable layer.

Conceptually:

```text
Without Volume

Container
   │
   └── Data

Remove Container
   │
   ▼
Data Lost
```

With a volume:

```text
Container
   │
   │
   ▼
Docker Volume
   │
   │
   ▼
Persistent Data
```

Now:

```text
Remove Container
       │
       ▼
Volume remains
       │
       ▼
Data remains
```

This is the main purpose of Docker volumes.

---

# 5. What Is a Docker Volume?

A Docker volume is storage managed by Docker that exists independently of a particular container.

A volume has its own lifecycle.

For example:

```text
Docker Host
   │
   └── Docker Volume
          │
          ▼
       Container
```

The container can be deleted while the volume remains.

We can then attach the same volume to another container.

---

# 6. Create a Docker Volume

Create a volume:

```bash
docker volume create my-volume
```

Expected:

```text
my-volume
```

List volumes:

```bash
docker volume ls
```

You should see something similar to:

```text
DRIVER    VOLUME NAME
local     my-volume
```

---

# 7. Inspect a Docker Volume

Run:

```bash
docker volume inspect my-volume
```

Docker will display information such as:

- Volume name
- Driver
- Mount point
- Scope

Example structure:

```json
[
    {
        "Name": "my-volume",
        "Driver": "local",
        "Scope": "local"
    }
]
```

The exact output can vary depending on your Docker environment.

---

# 8. Mount a Volume into a Container

Now let's attach the volume to a container.

Run:

```bash
docker run -it --name volume-container -v my-volume:/data ubuntu:24.04 bash
```

Let's understand this:

```text
-v my-volume:/data
      │        │
      │        └── Path inside container
      │
      └────────── Docker volume
```

Inside the container, `/data` is connected to `my-volume`.

---

# 9. Create Data Inside the Volume

Inside the container:

```bash
echo "Hello from Docker Volume" > /data/message.txt
```

Check:

```bash
cat /data/message.txt
```

Expected:

```text
Hello from Docker Volume
```

Now exit:

```bash
exit
```

---

# 10. Remove the Container

Remove the container:

```bash
docker rm volume-container
```

The container is gone.

But the volume should still exist.

Check:

```bash
docker volume ls
```

You should still see:

```text
DRIVER    VOLUME NAME
local     my-volume
```

This is the key difference.

---

# 11. Attach the Same Volume to Another Container

Create another container using the same volume:

```bash
docker run -it --name volume-container-2 -v my-volume:/data ubuntu:24.04 bash
```

Now check:

```bash
cat /data/message.txt
```

Expected:

```text
Hello from Docker Volume
```

🎉 The data survived the deletion of the first container.

This is persistent storage.

---

# 12. Docker Volume Architecture

The flow looks like this:

```text
                Docker Host
                     │
                     ▼
              ┌─────────────┐
              │ my-volume   │
              └──────┬──────┘
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
        Container 1      Container 2
             │                │
             ▼                ▼
          /data            /data
```

Both containers can access the same volume.

---

# 13. Why Are Volumes Important?

Volumes are useful for applications that need persistent data.

Examples:

- Databases
- Application uploads
- User-generated files
- Logs
- Shared application data
- Build caches
- Persistent application state

For example:

```text
MySQL Container
      │
      ▼
MySQL Volume
      │
      ▼
Database Data
```

If the MySQL container is replaced, the volume can preserve the database files.

---

# 14. Volumes vs Container Writable Layer

## Container Writable Layer

```text
Container
   │
   └── Writable Layer
          │
          └── Temporary data
```

If the container is removed:

```text
Container Removed
       ↓
Writable Layer Removed
       ↓
Data Lost
```

## Docker Volume

```text
Container
     │
     ▼
Docker Volume
     │
     ▼
Persistent Data
```

If the container is removed:

```text
Container Removed
       ↓
Volume Remains
       ↓
Data Remains
```

---

# 15. Named Volumes

The volume we created:

```bash
docker volume create my-volume
```

is called a **named volume**.

Named volumes are easy to identify:

```text
my-volume
database-data
app-data
jenkins-data
```

For example:

```bash
docker volume create database-data
```

List it:

```bash
docker volume ls
```

---

# 16. Anonymous Volumes

Docker can also create volumes without explicitly giving them a meaningful name.

For example, an image may contain a volume declaration.

You might see an automatically generated volume name.

Named volumes are generally easier to manage because you can identify them clearly.

---

# 17. Volume Mount Syntax

The common syntax is:

```bash
-v volume-name:/container/path
```

Example:

```bash
docker run -v my-volume:/data ubuntu:24.04
```

Meaning:

```text
my-volume
     │
     │ mounted to
     ▼
/data
```

Another example:

```bash
docker run -v database-data:/var/lib/mysql mysql
```

This connects the Docker volume to MySQL's data directory.

---

# 18. Using `--mount`

Docker also supports the more explicit `--mount` syntax.

Example:

```bash
docker run -it \
  --mount source=my-volume,target=/data \
  ubuntu:24.04 bash
```

Compare:

```bash
-v my-volume:/data
```

with:

```bash
--mount source=my-volume,target=/data
```

Both can mount the volume.

The `--mount` syntax is more verbose and explicit.

For learning Docker, understanding both is useful.

---

# 19. Create and Use a Volume with `--mount`

Create:

```bash
docker volume create demo-volume
```

Run:

```bash
docker run -it \
  --name demo-container \
  --mount source=demo-volume,target=/data \
  ubuntu:24.04 bash
```

Inside:

```bash
echo "Persistent Docker data" > /data/demo.txt
```

Check:

```bash
cat /data/demo.txt
```

Expected:

```text
Persistent Docker data
```

Exit:

```bash
exit
```

---

# 20. Reuse the Volume

Remove the container:

```bash
docker rm demo-container
```

Create another:

```bash
docker run -it \
  --name demo-container-2 \
  --mount source=demo-volume,target=/data \
  ubuntu:24.04 bash
```

Check:

```bash
cat /data/demo.txt
```

Expected:

```text
Persistent Docker data
```

This demonstrates volume persistence.

---

# 21. Bind Mounts

Docker also supports **bind mounts**.

A bind mount connects a directory from the Docker host to a directory inside the container.

Conceptually:

```text
Host Directory
      │
      │ bind mount
      ▼
Container Directory
```

For example:

```bash
docker run -it \
  -v "$(pwd):/workspace" \
  ubuntu:24.04 bash
```

This maps the current host directory to:

```text
/workspace
```

inside the container.

---

# 22. Volume vs Bind Mount

| Feature | Docker Volume | Bind Mount |
|---|---|---|
| Managed by Docker | Yes | No |
| Host path required | No | Yes |
| Easy to move between containers | Yes | Yes |
| Good for persistent app data | Yes | Yes |
| Good for local development | Yes | Very useful |
| Docker manages storage location | Yes | No |
| Useful for sharing source code | Less common | Very common |

Simple rule:

```text
Volume
  ↓
Docker-managed persistent data

Bind Mount
  ↓
Host directory ↔ Container directory
```

---

# 23. Practical Bind Mount Example

Create a directory on your host:

```bash
mkdir docker-data
```

Create a file:

```bash
echo "Hello from my Mac" > docker-data/message.txt
```

Run:

```bash
docker run -it \
  --name bind-container \
  -v "$(pwd)/docker-data:/data" \
  ubuntu:24.04 bash
```

Inside the container:

```bash
cat /data/message.txt
```

Expected:

```text
Hello from my Mac
```

Now create a file from inside the container:

```bash
echo "Created inside container" > /data/container.txt
```

Exit:

```bash
exit
```

On your host:

```bash
ls docker-data
```

You should see:

```text
container.txt
message.txt
```

This demonstrates two-way access through the bind mount.

---

# 24. Docker Volume Commands

List volumes:

```bash
docker volume ls
```

Create a volume:

```bash
docker volume create my-volume
```

Inspect a volume:

```bash
docker volume inspect my-volume
```

Remove a volume:

```bash
docker volume rm my-volume
```

Remove unused volumes:

```bash
docker volume prune
```

Be careful with:

```bash
docker volume prune
```

It removes unused local volumes.

Always review what Docker is going to remove before confirming.

---

# 25. Hands-On Challenge

Now build your own persistent storage lab.

## Goal

Create a Docker volume called:

```text
devops-data
```

### Step 1 — Create the volume

```bash
docker volume create devops-data
```

### Step 2 — Run a container

```bash
docker run -it \
  --name devops-volume-test \
  -v devops-data:/data \
  ubuntu:24.04 bash
```

### Step 3 — Create a file

Inside the container:

```bash
echo "Day 36 - Docker Volumes" > /data/day36.txt
```

### Step 4 — Verify

```bash
cat /data/day36.txt
```

Expected:

```text
Day 36 - Docker Volumes
```

### Step 5 — Exit

```bash
exit
```

### Step 6 — Remove the container

```bash
docker rm devops-volume-test
```

### Step 7 — Create another container

```bash
docker run -it \
  --name devops-volume-test-2 \
  -v devops-data:/data \
  ubuntu:24.04 bash
```

### Step 8 — Verify persistence

```bash
cat /data/day36.txt
```

Expected:

```text
Day 36 - Docker Volumes
```

If you see the file, your volume persistence test worked successfully. 🎉

---

# 26. Expected Result

Your workflow should look like:

```text
Create Volume
     │
     ▼
devops-data
     │
     ▼
Container 1
     │
     ├── Create day36.txt
     │
     ▼
Remove Container 1
     │
     ▼
Volume still exists
     │
     ▼
Container 2
     │
     ▼
Read day36.txt
```

Expected output:

```text
Day 36 - Docker Volumes
```

---

# 27. Troubleshooting

## Problem 1 — Volume doesn't exist

Run:

```bash
docker volume ls
```

If it doesn't exist:

```bash
docker volume create devops-data
```

---

## Problem 2 — File doesn't exist

Check that the volume was mounted correctly:

```bash
docker inspect devops-volume-test-2
```

Look for the `Mounts` section.

You should see information similar to:

```text
Source: .../devops-data
Destination: /data
```

---

## Problem 3 — Permission denied

If you get a permission error, check the permissions of the directory or file involved.

For bind mounts, remember that the host filesystem permissions affect what the container can access.

---

## Problem 4 — Volume cannot be removed

If Docker says the volume is in use, check:

```bash
docker ps -a
```

Find the container using the volume.

Remove the container first:

```bash
docker rm <container-name>
```

Then remove the volume:

```bash
docker volume rm devops-data
```

---

## Problem 5 — Bind mount path doesn't work

Check your current directory:

```bash
pwd
```

Then:

```bash
ls
```

Make sure the source directory exists.

For example:

```bash
mkdir docker-data
```

Then use:

```bash
-v "$(pwd)/docker-data:/data"
```

---

# 28. Important Commands Learned Today

| Command | Purpose |
|---|---|
| `docker volume create` | Create a volume |
| `docker volume ls` | List volumes |
| `docker volume inspect` | Inspect a volume |
| `docker volume rm` | Remove a volume |
| `docker volume prune` | Remove unused volumes |
| `docker run -v` | Mount a volume or bind mount |
| `docker run --mount` | Explicitly configure a mount |
| `docker inspect` | Inspect container configuration |

---

# 29. Real-World DevOps Example

Imagine we have a database container:

```text
             Docker Host
                  │
                  ▼
          ┌───────────────┐
          │ DB Volume     │
          │ database-data │
          └───────┬───────┘
                  │
                  ▼
          ┌───────────────┐
          │ MySQL         │
          │ Container     │
          └───────────────┘
```

If the MySQL container crashes:

```text
MySQL Container
      │
      X
   Deleted
```

The volume can remain:

```text
database-data
      │
      ▼
Persistent Database Data
```

A replacement container can mount the same volume.

This is a fundamental pattern for stateful containerized applications.

---

# 30. Important Production Consideration

Volumes provide persistence, but a volume is **not automatically a backup**.

For production systems, you also need to think about:

- Backups
- Restore procedures
- Disaster recovery
- Storage availability
- Access permissions
- Encryption
- Monitoring
- Data retention

A persistent volume protects data from container replacement, but it does not protect against every type of failure.

---

# 31. Key Takeaways

Today we learned:

- Containers have a writable layer
- Container-local data should not be treated as persistent
- Docker volumes provide persistent storage
- Named volumes are managed by Docker
- A volume can survive container deletion
- The same volume can be mounted into multiple containers
- `-v` can be used to mount storage
- `--mount` provides an explicit mount syntax
- Bind mounts connect host directories to containers
- Volumes and bind mounts have different use cases
- Docker volume management commands
- Why persistent storage is important for databases and applications
- Why volumes are not a replacement for backups

The most important concept:

```text
Container
    │
    │ mount
    ▼
Docker Volume
    │
    ▼
Persistent Data
```

---

# 32. LinkedIn Post — Day 36/100

## Docker Containers Are Temporary. What About the Data? 🐳

Day 36 of my **100 Days of DevOps — From Zero to Hero** journey.

One question I wanted to understand today was:

**What happens to data when a Docker container is deleted?**

The answer is important.

Data stored only inside a container's writable layer should not be treated as persistent.

That's where Docker Volumes come in.

Today I practiced:

✅ Creating Docker volumes

✅ Mounting volumes into containers

✅ Writing data into a volume

✅ Removing the container

✅ Creating a new container

✅ Verifying that the data was still there

I also learned the difference between:

**Docker Volume**

and

**Bind Mount**

The basic concept:

```text
Container
    ↓
Docker Volume
    ↓
Persistent Data
```

This is especially important when working with stateful applications such as databases.

One thing I learned today:

**Container lifecycle and data lifecycle don't have to be the same.**

A container can be replaced while the data it uses remains available through persistent storage.

Day 36 complete. 🚀

Next up: **Docker Networking.**

#100DaysOfDevOps #DevOps #Docker #DockerVolumes #Containers #Cloud #DevOpsJourney #LearningInPublic #DockerNetworking #DevOpsEngineer

---

# 33. What to Capture for LinkedIn

Capture screenshots of:

### Screenshot 1 — Create Volume

```bash
docker volume create devops-data
```

### Screenshot 2 — List Volumes

```bash
docker volume ls
```

### Screenshot 3 — Container With Volume

```bash
docker run -it \
  --name devops-volume-test \
  -v devops-data:/data \
  ubuntu:24.04 bash
```

### Screenshot 4 — Data Inside Volume

```bash
cat /data/day36.txt
```

### Screenshot 5 — Data After Container Replacement

Show the second container reading:

```bash
cat /data/day36.txt
```

This is a great screenshot because it demonstrates the main lesson:

```text
Container 1
    ↓
Create Data
    ↓
Delete Container
    ↓
Container 2
    ↓
Data Still Exists
```

---

# 34. Day 37 Preview

Tomorrow:

## Day 37 — Docker Networking

We will learn:

- What Docker networking is
- Container networking
- Bridge networks
- Creating custom networks
- Connecting containers
- Container-to-container communication
- Port publishing
- DNS between containers
- Hands-on Docker networking lab

The next important concept:

```text
Container A
     │
     │ Network
     ▼
Container B
```

See you on **Day 37/100** 🚀
