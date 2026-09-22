# 100 Days of DevOps — Day 31/100

## Introduction to Containers and Docker

Today we officially start the Docker section of the roadmap.

After completing Git & GitHub through Day 30, we now move into one of the most important technologies in modern DevOps: containers.

## 1. What problem do containers solve?

A common problem is:

> "It works on my machine!"

An application can work in one environment but fail in another because of differences in runtime versions, libraries, dependencies, configuration, or operating environments.

Containers help package an application together with the dependencies it needs to run consistently.

```text
             Container
┌───────────────────────────────┐
│        Application            │
│        Dependencies           │
│        Configuration          │
└───────────────────────────────┘
              │
              ▼
       Runs consistently
```

## 2. What is a Container?

A container is an isolated environment used to run an application and its dependencies.

```text
Container
│
├── Application
├── Libraries
├── Runtime
├── Configuration
└── Dependencies
```

Containers use the host operating system's kernel rather than carrying an entire guest operating system like a traditional virtual machine.

## 3. What is Docker?

Docker is a platform used to build, package, distribute, and run applications as containers.

```text
Application Code
       │
       ▼
 Dockerfile
       │
       ▼
 Docker Image
       │
       ▼
 Container
       │
       ▼
 Running Application
```

## 4. Docker Image vs Container

### Docker Image

An image is a read-only template used to create containers.

Examples:

```text
ubuntu
nginx
python
node
mysql
```

### Container

A container is a running instance of an image.

```text
Docker Image
     │
     ├──────► Container 1
     ├──────► Container 2
     └──────► Container 3
```

One image can create multiple containers.

## 5. Virtual Machine vs Container

A traditional VM:

```text
┌─────────────────────────┐
│ Application             │
├─────────────────────────┤
│ Libraries               │
├─────────────────────────┤
│ Guest Operating System  │
├─────────────────────────┤
│ Hypervisor              │
├─────────────────────────┤
│ Host OS / Hardware      │
└─────────────────────────┘
```

A container:

```text
┌─────────────────────────┐
│ Application             │
├─────────────────────────┤
│ Dependencies            │
├─────────────────────────┤
│ Container Runtime       │
├─────────────────────────┤
│ Host OS Kernel          │
├─────────────────────────┤
│ Hardware                │
└─────────────────────────┘
```

Containers are generally lightweight and fast to start, making them useful for CI/CD, microservices, and cloud environments.

Containers and VMs are not mutually exclusive. Both are useful and can also be used together.

## 6. Docker Architecture

```text
                 Docker CLI
                     │
                     ▼
              Docker Engine
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
      Images                Containers
          │                     │
          │                     ▼
          │               Application
          │
          ▼
     Docker Registry
```

The Docker CLI is what you interact with from your terminal. Docker Engine manages Docker objects such as images and containers. A registry stores and distributes images.

## 7. Install Docker

For a MacBook Air M1, install Docker Desktop for Mac with Apple Silicon:

https://www.docker.com/products/docker-desktop/

After installation:

```bash
docker --version
```

## 8. Hands-On Lab

### Step 1 — Check Docker

```bash
docker --version
docker info
```

Docker should return version and environment information when Docker Desktop is running.

### Step 2 — Run your first container

```bash
docker run hello-world
```

Docker will check for the image, pull it if necessary, create the container, run it, and display output.

You should see:

```text
Hello from Docker!
```

### Step 3 — List Docker images

```bash
docker images
```

Expected:

```text
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    xxxxxxxx       ...           ...
```

### Step 4 — List containers

```bash
docker ps
docker ps -a
```

`docker ps` shows running containers. `docker ps -a` shows all containers, including stopped containers.

### Step 5 — Run an Nginx container

```bash
docker run -d -p 8080:80 nginx
```

This maps:

```text
Host Port 8080 → Container Port 80
```

### Step 6 — Check the container

```bash
docker ps
```

Expected:

```text
CONTAINER ID   IMAGE   STATUS         PORTS
xxxxxxxx       nginx   Up ...         0.0.0.0:8080->80/tcp
```

### Step 7 — Open Nginx

Open:

```text
http://localhost:8080
```

You should see the Nginx welcome page.

```text
Browser
   │
   │ localhost:8080
   ▼
Mac Host
   │
   ▼
Container :80
   │
   ▼
Nginx
```

### Step 8 — View logs

```bash
docker ps
docker logs <container_id>
```

### Step 9 — Stop the container

```bash
docker stop <container_id>
docker ps
docker ps -a
```

### Step 10 — Remove the container

```bash
docker rm <container_id>
docker ps -a
```

## 9. Commands You Learned

| Command | Purpose |
|---|---|
| `docker --version` | Check Docker version |
| `docker info` | Show Docker environment information |
| `docker run` | Create and start a container |
| `docker images` | List images |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker logs` | View container logs |
| `docker stop` | Stop a container |
| `docker rm` | Remove a container |

## 10. Troubleshooting

### Docker command not found

```text
docker: command not found
```

Make sure Docker Desktop is installed and running.

### Cannot connect to Docker daemon

Start Docker Desktop, wait until it is ready, then run:

```bash
docker info
```

### Port 8080 already in use

Use another host port:

```bash
docker run -d -p 8081:80 nginx
```

Then open:

```text
http://localhost:8081
```

### Container immediately exits

Check:

```bash
docker ps -a
docker logs <container_id>
```

## 11. What to Capture for LinkedIn

Capture a clean terminal screenshot showing:

```bash
docker images
docker ps
docker ps -a
```

Also capture:

```text
http://localhost:8080
```

showing the Nginx page.

## 12. Day 31 Takeaway

```text
Docker
  │
  ├── Image
  │      └── Template
  │
  └── Container
         └── Running instance
```

Fundamental workflow:

```text
Code
 ↓
Docker Image
 ↓
Container
 ↓
Application
```

Focus on understanding the relationship between an image and a container rather than memorizing every command.

## Day 31/100 — LinkedIn Post

Today I started the Docker section of my DevOps journey.

Before Docker, one common problem developers face is:

> "It works on my machine."

The application may work perfectly in one environment but fail somewhere else because of differences in runtime versions, dependencies, libraries, configuration, or operating environments.

This is where containers become useful.

A container packages an application together with the dependencies it needs to run.

The basic idea is:

**Application → Docker Image → Container → Running Application**

Today I also learned the difference between an image and a container.

**Docker Image = template**

**Container = running instance of that template**

For hands-on practice, I:

- Installed/verified Docker
- Ran my first `hello-world` container
- Pulled the Nginx image
- Started an Nginx container
- Mapped port `8080 → 80`
- Checked running and stopped containers
- Viewed container logs
- Stopped and removed a container

Containers aren't just about packaging applications. They become especially useful when combined with CI/CD, cloud platforms, Kubernetes, and infrastructure automation.

**Day 31 complete. 69 days to go.**

#100DaysOfDevOps #DevOps #Docker #Containers #Cloud #CI #CD #DevOpsLearning #DockerLearning #LearningInPublic

## Day 32 Preview

**Docker Images and Containers**

Next we'll go deeper into Docker images, image layers, containers, and the container lifecycle.
