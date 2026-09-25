# Day 35 — Building and Running Docker Images

## 100 Days of DevOps — From Zero to Hero

Welcome to **Day 35** of my 100 Days of DevOps journey.

Today we will learn how to **build our own Docker image and run a container from it**.

---

## 📌 Today's Topic

**Day 35 — Building and Running Docker Images**

Before today, we learned:

- Day 31 — Introduction to Containers and Docker
- Day 32 — Docker Images and Containers
- Day 33 — Working with Docker Commands
- Day 34 — Writing Your First Dockerfile

Today, we connect those concepts together:

```text
Dockerfile
    │
    ▼
docker build
    │
    ▼
Docker Image
    │
    ▼
docker run
    │
    ▼
Docker Container
```

---

# 1. What Does `docker build` Do?

A Dockerfile contains instructions that describe how our image should be created.

For example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

When we run:

```bash
docker build -t my-python-app .
```

Docker reads the Dockerfile and creates an image.

```text
Dockerfile
     │
     │ docker build
     ▼
Docker Image
```

The `.` at the end is important.

It tells Docker:

> Use the current directory as the build context.

---

# 2. What Is a Build Context?

The **build context** is the set of files Docker can access while building the image.

For example:

```text
docker-day35/
├── Dockerfile
├── app.py
└── README.md
```

If we run:

```bash
docker build -t my-python-app .
```

the current directory becomes the build context.

Docker can access files inside this context.

For example:

```dockerfile
COPY app.py .
```

copies `app.py` from the build context into the image.

---

# 3. Docker Image Tags

A Docker image can have a name and a tag.

Example:

```bash
docker build -t my-python-app:v1 .
```

Here:

```text
my-python-app:v1
      │        │
      │        └── Tag
      │
      └────────── Image name
```

Another version:

```bash
docker build -t my-python-app:v2 .
```

Now we have two versions:

```text
my-python-app:v1
my-python-app:v2
```

Tags are extremely useful for versioning Docker images.

---

# 4. Build an Image

Let's create a simple Python application.

Create a directory:

```bash
mkdir docker-day35
cd docker-day35
```

Create `app.py`:

```bash
cat > app.py <<'EOF'
print("Hello from Docker Day 35!")
print("This application is running inside a Docker container.")
EOF
```

Create the Dockerfile:

```bash
cat > Dockerfile <<'EOF'
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
EOF
```

Your directory should look like:

```text
docker-day35/
├── Dockerfile
└── app.py
```

---

# 5. Build the Docker Image

Run:

```bash
docker build -t my-python-app:v1 .
```

You should see Docker processing the Dockerfile.

The final output should contain something similar to:

```text
Successfully tagged my-python-app:v1
```

Depending on your Docker version, the output may look different because modern Docker commonly uses BuildKit.

---

# 6. Verify the Image

Run:

```bash
docker images
```

You should see something similar to:

```text
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
my-python-app    v1        xxxxxxxxxxxx   seconds ago      ...
python           3.12-slim xxxxxxxxxxxx   ...              ...
```

Our image is now ready.

---

# 7. Run the Docker Image

Now create a container from our image:

```bash
docker run --name my-python-container my-python-app:v1
```

Expected output:

```text
Hello from Docker Day 35!
This application is running inside a Docker container.
```

The relationship is:

```text
Docker Image
     │
     │ docker run
     ▼
Docker Container
```

---

# 8. Check the Container

Run:

```bash
docker ps -a
```

You should see:

```text
CONTAINER ID   IMAGE               COMMAND          STATUS
xxxxxxxxxxxx   my-python-app:v1    "python app.py" Exited (0)
```

Why is the container showing `Exited`?

Because our Python program finished executing.

A container remains alive only while its main process is running.

For example:

```text
python app.py
     │
     ▼
Application starts
     │
     ▼
Application finishes
     │
     ▼
Container exits
```

This is expected behavior.

---

# 9. View Container Logs

Run:

```bash
docker logs my-python-container
```

Expected:

```text
Hello from Docker Day 35!
This application is running inside a Docker container.
```

---

# 10. Build Version 2

Let's modify the application.

Edit `app.py`:

```bash
cat > app.py <<'EOF'
print("Hello from Docker Day 35!")
print("This is version 2 of my application.")
EOF
```

Now build another image:

```bash
docker build -t my-python-app:v2 .
```

Check the images:

```bash
docker images
```

You should now have:

```text
my-python-app:v1
my-python-app:v2
```

---

# 11. Run Version 2

Run:

```bash
docker run --name my-python-container-v2 my-python-app:v2
```

Expected:

```text
Hello from Docker Day 35!
This is version 2 of my application.
```

Now we have two versions of our application image:

```text
my-python-app:v1
        │
        └── Container 1

my-python-app:v2
        │
        └── Container 2
```

---

# 12. Image Versioning

In real DevOps environments, image tags are commonly used to identify application versions.

For example:

```text
myapp:1.0.0
myapp:1.1.0
myapp:1.2.0
```

Or:

```text
myapp:dev
myapp:test
myapp:staging
myapp:production
```

A CI/CD pipeline may build an image like:

```text
myapp:1.4.2
```

and push it to a container registry.

Examples of container registries include:

- Docker Hub
- GitHub Container Registry
- Azure Container Registry
- Amazon Elastic Container Registry
- Google Artifact Registry

---

# 13. Docker Build Workflow

A typical Docker workflow looks like this:

```text
Developer
    │
    ▼
Application Code
    │
    ▼
Dockerfile
    │
    ▼
docker build
    │
    ▼
Docker Image
    │
    ▼
Image Tag
    │
    ▼
Container Registry
    │
    ▼
docker pull
    │
    ▼
Docker Container
```

This is one of the foundations of container-based DevOps.

---

# 14. Understanding Docker Build Layers

Docker images are built in layers.

For example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Conceptually:

```text
┌─────────────────────────────┐
│ CMD ["python", "app.py"]    │
├─────────────────────────────┤
│ COPY app.py .               │
├─────────────────────────────┤
│ WORKDIR /app                │
├─────────────────────────────┤
│ Python 3.12 Slim            │
└─────────────────────────────┘
```

Docker can reuse unchanged layers during subsequent builds.

This is one reason Docker builds can become much faster after the first build.

---

# 15. Inspect the Image

Run:

```bash
docker image inspect my-python-app:v1
```

This displays detailed information about the image.

You can also check the image history:

```bash
docker history my-python-app:v1
```

This is useful for understanding how the image was built.

---

# 16. Compare Image Versions

Run:

```bash
docker images my-python-app
```

You may see:

```text
REPOSITORY       TAG       IMAGE ID       CREATED
my-python-app    v1        abc123         ...
my-python-app    v2        def456         ...
```

Each tag points to an image.

---

# 17. Clean Up Containers

List containers:

```bash
docker ps -a
```

Remove the containers:

```bash
docker rm my-python-container
docker rm my-python-container-v2
```

Verify:

```bash
docker ps -a
```

---

# 18. Remove Images

List images:

```bash
docker images
```

Remove version 1:

```bash
docker rmi my-python-app:v1
```

Remove version 2:

```bash
docker rmi my-python-app:v2
```

Verify:

```bash
docker images
```

> Do not remove an image if containers still depend on it unless you intentionally want to clean them up.

---

# 19. Hands-On Challenge

Now try this yourself without copying the previous example exactly.

## Task

Create a Docker image for a Python application.

### Requirements

Create:

```text
docker-day35-challenge/
├── Dockerfile
└── app.py
```

Your `app.py` should print:

```text
Welcome to my DevOps journey!
Day 35 - Docker Image Build
```

Create a Dockerfile using:

```dockerfile
FROM python:3.12-slim
```

Build the image with:

```bash
docker build -t devops-day35:v1 .
```

Run it:

```bash
docker run --name devops-day35-container devops-day35:v1
```

Check:

```bash
docker images
```

and:

```bash
docker ps -a
```

Finally check:

```bash
docker logs devops-day35-container
```

---

# 20. Expected Result

You should see:

```text
Welcome to my DevOps journey!
Day 35 - Docker Image Build
```

And:

```bash
docker images
```

should show:

```text
devops-day35    v1
```

---

# 21. Troubleshooting

## Problem 1: Docker command not found

Run:

```bash
docker --version
```

If Docker is not available, make sure Docker Desktop is installed and running.

---

## Problem 2: Dockerfile not found

If you see an error related to the Dockerfile, check:

```bash
ls
```

You should have:

```text
Dockerfile
app.py
```

Then run:

```bash
docker build -t my-python-app:v1 .
```

Make sure you are inside the correct directory.

---

## Problem 3: Image doesn't appear

Run:

```bash
docker images
```

If the image isn't listed, check the output of:

```bash
docker build -t my-python-app:v1 .
```

Look for errors during the build.

---

## Problem 4: Container exits immediately

Run:

```bash
docker ps -a
```

Then:

```bash
docker logs my-python-container
```

For a short-lived Python script, an `Exited (0)` status is normal.

---

## Problem 5: `COPY app.py .` fails

Check:

```bash
ls
```

Make sure `app.py` exists in the build context.

Also make sure you are running:

```bash
docker build -t my-python-app:v1 .
```

from the directory containing `app.py`.

---

# 22. Important Commands Learned Today

| Command | Purpose |
|---|---|
| `docker build` | Build an image |
| `docker images` | List images |
| `docker run` | Create and run a container |
| `docker logs` | View container logs |
| `docker history` | View image layers/history |
| `docker image inspect` | Inspect image details |
| `docker ps -a` | List all containers |
| `docker rm` | Remove containers |
| `docker rmi` | Remove images |

---

# 23. Key Takeaways

Today we learned:

- What `docker build` does
- What a Docker build context is
- How to build an image from a Dockerfile
- How to tag Docker images
- How to run a container from an image
- How image versions work
- How Docker image layers work
- How to inspect images
- How to troubleshoot Docker builds
- Why image versioning matters in DevOps

The important relationship to remember is:

```text
Dockerfile
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker run
    ▼
Docker Container
```

---

# 24. LinkedIn Post — Day 35/100

## Building and Running My First Docker Images 🚀

Day 35 of my **100 Days of DevOps — From Zero to Hero** journey.

Yesterday I learned how to write a Dockerfile.

Today I went one step further:

**I built my own Docker image and ran a container from it.**

The workflow is simple:

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Docker Container
```

One thing that became much clearer today was the difference between an image and a container.

An image is the packaged application.

A container is the running instance created from that image.

I also practiced:

✅ Building images with `docker build`

✅ Tagging images with versions

✅ Running containers

✅ Checking images with `docker images`

✅ Checking containers with `docker ps -a`

✅ Viewing logs with `docker logs`

✅ Understanding Docker build layers

I also created two versions of the same application:

```text
my-python-app:v1
my-python-app:v2
```

This made the idea of container image versioning much more practical.

The next step is to understand something equally important:

**Where does data go when a container is removed?**

That's where Docker Volumes come in.

One more day. One more step toward understanding real-world DevOps.

#100DaysOfDevOps #DevOps #Docker #Containers #Cloud #DevOpsJourney #LearningInPublic #Dockerfile #GitHub #DevOpsEngineer

---

# 25. What to Capture for LinkedIn

Take screenshots of:

### Screenshot 1 — Docker Build

```bash
docker build -t my-python-app:v1 .
```

### Screenshot 2 — Docker Images

```bash
docker images
```

### Screenshot 3 — Running the Container

```bash
docker run --name my-python-container my-python-app:v1
```

### Screenshot 4 — Container Logs

```bash
docker logs my-python-container
```

A good LinkedIn carousel can show:

```text
Dockerfile
   ↓
Build
   ↓
Image
   ↓
Run
   ↓
Container
```

---

# 26. Day 36 Preview

Tomorrow:

## Docker Volumes

We will learn:

- Why containers are ephemeral
- What happens to data when a container is deleted
- Docker volumes
- Bind mounts
- Creating volumes
- Mounting volumes into containers
- Sharing data between containers
- Practical hands-on lab

See you on **Day 36/100** 🚀
