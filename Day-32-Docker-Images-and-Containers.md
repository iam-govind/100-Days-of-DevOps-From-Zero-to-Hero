# 100 Days of DevOps — Day 32/100

## Docker Images and Containers

Yesterday we started Docker and learned the basic idea:

**Image → Container → Application**

Today we'll go one level deeper and understand what Docker images are, how containers are created from them, and how containers move through their lifecycle.

## 1. What is a Docker Image?

A Docker image is a read-only template used to create containers.

```text
Docker Image
     │
     ├── Application
     ├── Runtime
     ├── Libraries
     ├── Dependencies
     └── Configuration
```

Think of an image as a blueprint and a container as the instance created from that blueprint.

```text
Image
  │
  │ docker run
  ▼
Container
  │
  ▼
Running Application
```

## 2. One Image Can Create Multiple Containers

```text
             nginx Image
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
   Container 1 Container 2 Container 3
```

All can be created from the same image but remain separate container instances.

## 3. Images Are Made of Layers

Docker images are built using layers.

```text
┌──────────────────────────┐
│ Application Layer        │
├──────────────────────────┤
│ Dependency Layer         │
├──────────────────────────┤
│ Runtime Layer            │
├──────────────────────────┤
│ Base Image Layer         │
└──────────────────────────┘
```

This layered architecture helps Docker reuse existing layers, reduce storage, speed up image builds, and improve build caching.

We'll explore image layers in more detail when we learn Dockerfiles.

## 4. Pulling an Image

```bash
docker pull nginx
docker images
```

Expected:

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    xxxxxxxx       ...           ...
```

## 5. Understanding Image Tags

Notice:

```text
nginx:latest
```

Here:

```text
nginx   → Image name
latest  → Tag
```

You can specify another tag:

```bash
docker pull nginx:1.27
```

Available tags can change over time, so production environments should deliberately choose and manage image versions rather than blindly relying on `latest`.

## 6. Create a Container from an Image

```bash
docker run -d --name my-nginx nginx
```

Check:

```bash
docker ps
```

You should see the `my-nginx` container.

```text
nginx image
     │
     ▼
my-nginx container
```

## 7. Give the Container a Name

Using:

```bash
--name my-nginx
```

lets you use:

```bash
docker stop my-nginx
```

instead of remembering a long container ID.

# 8. Hands-On Lab

## Step 1 — Download Nginx

```bash
docker pull nginx
```

Docker downloads the required image layers.

## Step 2 — Check images

```bash
docker images
```

Expected:

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    xxxxxxxx       ...           ...
```

## Step 3 — Run the container

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

This means:

```text
docker run       → Create and start a container
-d               → Detached mode
--name my-nginx  → Container name
-p 8080:80       → Host port 8080 → Container port 80
nginx            → Image
```

## Step 4 — Verify

```bash
docker ps
```

Expected:

```text
CONTAINER ID   IMAGE   STATUS      PORTS
xxxxxxxx       nginx   Up ...      0.0.0.0:8080->80/tcp
```

## Step 5 — Test the application

Open:

```text
http://localhost:8080
```

Or:

```bash
curl http://localhost:8080
```

You should receive the Nginx welcome page/HTML response.

## Step 6 — Inspect the container

```bash
docker inspect my-nginx
```

This shows information such as:

- Container ID
- Image
- Network
- Ports
- Mounts
- Configuration
- Environment
- State

## Step 7 — Check logs

```bash
docker logs my-nginx
```

Follow logs:

```bash
docker logs -f my-nginx
```

Press `Ctrl + C` to stop following logs.

## Step 8 — Enter the container

Try:

```bash
docker exec -it my-nginx /bin/bash
```

If Bash is unavailable:

```bash
docker exec -it my-nginx /bin/sh
```

Inside the container:

```bash
ls
pwd
cat /etc/os-release
```

Exit:

```bash
exit
```

Important: `docker exec` runs a command inside an already running container.

## 9. Container Lifecycle

```text
          docker run
              │
              ▼
          Created
              │
              ▼
           Running
          /                /          docker stop    docker kill
       │             │
       ▼             ▼
    Stopped       Stopped
       │
       │ docker start
       └──────────────► Running
```

Restart:

```bash
docker restart my-nginx
```

## 10. Stop the Container

```bash
docker stop my-nginx
docker ps
docker ps -a
```

The container should appear as stopped/exited.

## 11. Start the Container Again

```bash
docker start my-nginx
docker ps
curl http://localhost:8080
```

## 12. Remove the Container

```bash
docker stop my-nginx
docker rm my-nginx
docker ps -a
```

The container should no longer appear.

## 13. Image vs Container

| Docker Image | Docker Container |
|---|---|
| Template | Running/created instance |
| Read-only image filesystem | Writable container layer |
| Used to create containers | Created from an image |
| Can be stored in a registry | Runs on Docker Engine |
| Example: `nginx` | Example: `my-nginx` |

Mental model:

```text
             IMAGE
               │
        docker run
               │
               ▼
           CONTAINER
               │
        ┌──────┴──────┐
        ▼             ▼
     Running       Stopped
```

## 14. Mini Challenge

Run two Nginx containers from the same image.

Container 1:

```text
my-nginx-1
```

Port:

```text
8080
```

Container 2:

```text
my-nginx-2
```

Port:

```text
8081
```

Commands:

```bash
docker run -d --name my-nginx-1 -p 8080:80 nginx
docker run -d --name my-nginx-2 -p 8081:80 nginx
```

Check:

```bash
docker ps
```

Test:

```bash
curl http://localhost:8080
curl http://localhost:8081
```

Concept:

```text
             nginx:latest
                  │
          ┌───────┴───────┐
          ▼               ▼
     my-nginx-1       my-nginx-2
       :8080             :8081
```

## 15. Important Commands

| Command | Purpose |
|---|---|
| `docker pull` | Download an image |
| `docker images` | List images |
| `docker run` | Create/start a container |
| `docker ps` | Show running containers |
| `docker ps -a` | Show all containers |
| `docker inspect` | Inspect container configuration |
| `docker logs` | View logs |
| `docker exec` | Execute a command inside a container |
| `docker stop` | Stop a container |
| `docker start` | Start a stopped container |
| `docker restart` | Restart a container |
| `docker rm` | Remove a container |

## 16. Troubleshooting

### Port already in use

Check:

```bash
docker ps
lsof -i :8080
```

Choose another host port:

```bash
docker run -d --name my-nginx -p 8082:80 nginx
```

### Container name already exists

Check:

```bash
docker ps -a
```

Remove the old container:

```bash
docker rm my-nginx
```

Or choose another name.

### Container exits unexpectedly

Check:

```bash
docker ps -a
docker logs <container-name>
```

### Can't access localhost

Check:

```bash
docker ps
```

Make sure the port mapping exists:

```text
0.0.0.0:8080->80/tcp
```

Then:

```bash
curl http://localhost:8080
```

## 17. What to Capture for LinkedIn

Capture a clean terminal screenshot showing:

```bash
docker images
docker ps
```

A stronger screenshot shows two containers running from the same image:

```text
CONTAINER       IMAGE   PORT
my-nginx-1      nginx   8080->80
my-nginx-2      nginx   8081->80
```

You can also capture:

```text
http://localhost:8080
http://localhost:8081
```

This visually demonstrates:

**One image → Multiple containers**

## 18. Day 32 Takeaway

```text
                Docker Image
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     Container 1           Container 2
       :8080                  :8081
          │                     │
          ▼                     ▼
       Nginx                  Nginx
```

Remember:

> **Image = blueprint**

> **Container = instance created from the blueprint**

Images are made from reusable layers, which is one reason Docker can build and distribute applications efficiently.

# Day 32/100 — LinkedIn Post

Yesterday I learned the basics of containers.

Today I went one level deeper into **Docker Images and Containers**.

One concept that became clear to me:

**An image is not the container.**

Think of it like this:

**Docker Image = Blueprint**

**Docker Container = Instance created from that blueprint**

Today I practiced:

- Pulling Docker images
- Understanding image tags
- Running containers
- Naming containers
- Mapping host and container ports
- Inspecting containers
- Checking container logs
- Executing commands inside containers
- Stopping and starting containers
- Removing containers
- Running multiple containers from the same image

I also learned about **Docker image layers**.

The layered architecture is interesting because Docker can reuse existing layers instead of rebuilding everything from scratch.

The hands-on exercise that helped me understand it best was running:

**One Nginx image → Two containers**

```text
nginx image
    │
    ├── my-nginx-1 → :8080
    │
    └── my-nginx-2 → :8081
```

This is where Docker started feeling less like "just another tool" and more like an important building block for modern DevOps.

Next, I'll start working with Docker commands more deeply and understand how to manage containers from the command line.

**Day 32 complete. 68 days to go.**

#100DaysOfDevOps #DevOps #Docker #Containers #DockerLearning #Cloud #CI #CD #DevOpsJourney #LearningInPublic

## Day 33 Preview

**Working with Docker Commands**

Next we'll build stronger command-line skills for:

```text
Create
  ↓
Run
  ↓
Inspect
  ↓
Logs
  ↓
Exec
  ↓
Stop
  ↓
Start
  ↓
Remove
```

The goal will be to become comfortable managing containers from the terminal before we start writing our first Dockerfile.
