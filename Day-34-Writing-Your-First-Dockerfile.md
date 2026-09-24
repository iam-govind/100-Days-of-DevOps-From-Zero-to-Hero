# Day 34 — Writing Your First Dockerfile

## 100 Days of DevOps — From Zero to Hero

Welcome to **Day 34** of my 100 Days of DevOps journey.

Today we move from **using existing Docker images** to **creating our own Docker image**.

---

# 📌 Today's Topic

**Day 34 — Writing Your First Dockerfile**

So far we have learned:

- Day 31 — Introduction to Containers and Docker
- Day 32 — Docker Images and Containers
- Day 33 — Working with Docker Commands

Today we will learn how to create a **Dockerfile** and use it to build our own Docker image.

The basic workflow is:

```text
Application Code
       │
       ▼
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

# 1. What Is a Dockerfile?

A **Dockerfile** is a text file containing instructions that Docker uses to build an image.

For example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Each instruction tells Docker how the image should be created.

Think of a Dockerfile as a **recipe for creating a Docker image**.

```text
Dockerfile = Recipe
Docker Image = Prepared Package
Docker Container = Running Application
```

---

# 2. Why Do We Need Dockerfiles?

Imagine you have an application that requires:

- Python
- Application code
- Specific dependencies
- Environment configuration
- A specific startup command

Instead of manually configuring everything on every server, we can describe the setup in a Dockerfile.

Then Docker can build the same image consistently.

```text
Without Dockerfile:

Developer Machine
       ↓
Manual Setup
       ↓
Test Server
       ↓
More Manual Setup
       ↓
Production Server
       ↓
Different Environment Problems
```

With Docker:

```text
Dockerfile
    ↓
Docker Image
    ↓
Same Image
    ↓
Dev / Test / Production
```

This helps reduce:

> "It works on my machine!"

---

# 3. Basic Dockerfile Instructions

Some of the most important Dockerfile instructions are:

| Instruction | Purpose |
|---|---|
| `FROM` | Defines the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies files into the image |
| `RUN` | Executes commands while building |
| `CMD` | Defines the default command |
| `ENTRYPOINT` | Defines the main executable |
| `EXPOSE` | Documents the application port |
| `ENV` | Defines environment variables |

Today we will focus on the most important beginner-level instructions.

---

# 4. FROM

`FROM` defines the base image.

Example:

```dockerfile
FROM python:3.12-slim
```

This tells Docker:

> Start with the Python 3.12 slim image.

Another example:

```dockerfile
FROM nginx:alpine
```

Or:

```dockerfile
FROM ubuntu:24.04
```

Every normal Dockerfile starts with a `FROM` instruction.

---

# 5. WORKDIR

`WORKDIR` sets the working directory inside the image.

Example:

```dockerfile
WORKDIR /app
```

After this instruction, commands will operate from:

```text
/app
```

Instead of manually doing:

```bash
cd /app
```

Docker handles it for us.

---

# 6. COPY

`COPY` copies files from the build context into the Docker image.

Example:

```dockerfile
COPY app.py .
```

This means:

```text
Local Machine
     │
     │ app.py
     ▼
Docker Image
     │
     ▼
/app/app.py
```

Because we already used:

```dockerfile
WORKDIR /app
```

the file will be copied into:

```text
/app/app.py
```

---

# 7. RUN

`RUN` executes a command while building the Docker image.

Example:

```dockerfile
RUN pip install flask
```

Another example:

```dockerfile
RUN apt-get update
```

The important point is:

> `RUN` happens during image build time.

For example:

```text
docker build
      │
      ▼
RUN command
      │
      ▼
New image layer
```

---

# 8. CMD

`CMD` defines the default command that runs when a container starts.

Example:

```dockerfile
CMD ["python", "app.py"]
```

When we run:

```bash
docker run my-python-app
```

Docker starts:

```bash
python app.py
```

Important:

> `CMD` runs when the container starts, not when the image is built.

---

# 9. EXPOSE

`EXPOSE` documents the port that the application is expected to use.

Example:

```dockerfile
EXPOSE 8080
```

For an application listening on port 8080, we might document it with:

```dockerfile
EXPOSE 8080
```

Important:

`EXPOSE` does **not** publish the port to your host machine.

To publish a port, use:

```bash
docker run -p 8080:8080 my-app
```

---

# 10. Our First Dockerfile

Let's create a simple Python application.

Create a project directory:

```bash
mkdir docker-day34
cd docker-day34
```

Create the application:

```bash
cat > app.py <<'EOF'
print("Hello from Docker!")
print("This is Day 34 of my 100 Days of DevOps journey.")
EOF
```

Now create the Dockerfile:

```bash
cat > Dockerfile <<'EOF'
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
EOF
```

Our project now looks like:

```text
docker-day34/
├── Dockerfile
└── app.py
```

---

# 11. Understand the Dockerfile

Our Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Let's understand it step by step.

### Step 1

```dockerfile
FROM python:3.12-slim
```

Start with Python.

### Step 2

```dockerfile
WORKDIR /app
```

Create/use `/app` as the working directory.

### Step 3

```dockerfile
COPY app.py .
```

Copy our application into `/app`.

### Step 4

```dockerfile
CMD ["python", "app.py"]
```

Run the Python application when the container starts.

---

# 12. Build the Docker Image

Now build the image:

```bash
docker build -t my-python-app .
```

Explanation:

```text
docker build
     │
     ├── -t my-python-app
     │       └── Image name
     │
     └── .
         └── Build context
```

You should see Docker processing the Dockerfile.

At the end, you should see output indicating that the image was successfully built and tagged.

---

# 13. Check the Image

Run:

```bash
docker images
```

You should see something similar to:

```text
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
my-python-app    latest    xxxxxxxxxxxx   seconds ago      ...
python           3.12-slim xxxxxxxxxxxx   ...              ...
```

Our custom image now exists.

---

# 14. Run the Container

Run:

```bash
docker run --name my-python-container my-python-app
```

Expected output:

```text
Hello from Docker!
This is Day 34 of my 100 Days of DevOps journey.
```

The complete process was:

```text
app.py
   │
   ▼
Dockerfile
   │
   │ docker build
   ▼
Docker Image
   │
   │ docker run
   ▼
Docker Container
   │
   ▼
Python Application
```

---

# 15. Check the Container

Run:

```bash
docker ps -a
```

You should see:

```text
CONTAINER ID   IMAGE           COMMAND          STATUS
xxxxxxxxxxxx   my-python-app   "python app.py" Exited (0)
```

The container exits because the Python program finishes.

This is expected.

A container stays running while its main process is running.

---

# 16. Check Container Logs

Run:

```bash
docker logs my-python-container
```

Expected:

```text
Hello from Docker!
This is Day 34 of my 100 Days of DevOps journey.
```

---

# 17. Inspect the Image

We can inspect the image using:

```bash
docker image inspect my-python-app
```

This provides information such as:

- Image ID
- Architecture
- OS
- Environment
- Entrypoint
- Command
- Working directory
- Layers

---

# 18. View Image History

Run:

```bash
docker history my-python-app
```

This helps us understand the layers created during the image build.

Conceptually:

```text
┌───────────────────────────┐
│ CMD python app.py         │
├───────────────────────────┤
│ COPY app.py               │
├───────────────────────────┤
│ WORKDIR /app              │
├───────────────────────────┤
│ Python 3.12 Slim          │
└───────────────────────────┘
```

Docker images are built using layers.

---

# 19. Make a Change and Rebuild

Let's modify our application.

```bash
cat > app.py <<'EOF'
print("Hello from Docker!")
print("This is Day 34 - Version 2.")
EOF
```

Now rebuild:

```bash
docker build -t my-python-app:v2 .
```

Check:

```bash
docker images
```

You should now see:

```text
my-python-app    v2
my-python-app    latest
```

Run version 2:

```bash
docker run --name my-python-container-v2 my-python-app:v2
```

Expected:

```text
Hello from Docker!
This is Day 34 - Version 2.
```

---

# 20. Dockerfile and Image Versioning

We can use tags to identify image versions.

For example:

```text
my-python-app:v1
my-python-app:v2
my-python-app:v3
```

In real DevOps projects, you may see:

```text
myapp:1.0.0
myapp:1.1.0
myapp:2.0.0
```

This becomes especially important when Docker images are used in CI/CD pipelines.

---

# 21. Create a `.dockerignore`

Just like `.gitignore`, Docker supports:

```text
.dockerignore
```

It tells Docker which files should not be sent as part of the build context.

Create one:

```bash
cat > .dockerignore <<'EOF'
.git
.gitignore
README.md
*.log
.env
EOF
```

Example project:

```text
docker-day34/
├── Dockerfile
├── .dockerignore
└── app.py
```

This is important because we don't want to unnecessarily copy:

- Git files
- Logs
- Secrets
- Temporary files
- Documentation
- Local development files

---

# 22. Why `.dockerignore` Matters

Suppose your project contains:

```text
docker-day34/
├── .git/
├── node_modules/
├── logs/
├── .env
├── Dockerfile
└── app.py
```

Sending unnecessary files to Docker can:

- Increase build context size
- Slow down builds
- Increase image-building overhead
- Accidentally expose sensitive files

So:

```text
.dockerignore
      ↓
Smaller build context
      ↓
Cleaner builds
      ↓
Better security
```

---

# 23. Hands-On Challenge

Now try building your own image without copying the exact example.

## Task

Create:

```text
docker-day34-challenge/
├── Dockerfile
└── app.py
```

Your `app.py` should print:

```text
Welcome to my DevOps journey!
Day 34 - My First Dockerfile
```

Your Dockerfile should:

1. Use Python 3.12 slim
2. Set `/app` as the working directory
3. Copy `app.py`
4. Run the Python application

Build:

```bash
docker build -t devops-day34:v1 .
```

Run:

```bash
docker run --name devops-day34-container devops-day34:v1
```

Check:

```bash
docker images
```

and:

```bash
docker ps -a
```

Finally:

```bash
docker logs devops-day34-container
```

---

# 24. Expected Result

You should get:

```text
Welcome to my DevOps journey!
Day 34 - My First Dockerfile
```

And:

```bash
docker images
```

should show:

```text
devops-day34    v1
```

---

# 25. Troubleshooting

## Problem 1: Dockerfile not found

Check:

```bash
ls
```

You should see:

```text
Dockerfile
app.py
```

Make sure you are inside the correct directory.

---

## Problem 2: Build fails

Run:

```bash
docker build -t my-python-app .
```

Carefully check the error message.

Common causes:

- Incorrect Dockerfile syntax
- Wrong file name
- Missing `app.py`
- Network problems while pulling the base image
- Incorrect build context

---

## Problem 3: `COPY app.py .` fails

Check:

```bash
ls -l
```

Make sure `app.py` exists.

Also make sure you are building from the directory containing `app.py`.

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

If the application is a short Python script, `Exited (0)` is normal.

---

## Problem 5: Image is not visible

Run:

```bash
docker images
```

If it isn't there, rebuild:

```bash
docker build -t my-python-app:v1 .
```

---

# 26. Important Commands Learned Today

| Command | Purpose |
|---|---|
| `docker build` | Build an image |
| `docker images` | List images |
| `docker run` | Create and run a container |
| `docker logs` | View container logs |
| `docker image inspect` | Inspect an image |
| `docker history` | View image layers/history |
| `docker ps -a` | List all containers |
| `docker rm` | Remove a container |
| `docker rmi` | Remove an image |

---

# 27. Key Takeaways

Today we learned:

- What a Dockerfile is
- Why Dockerfiles are useful
- How `FROM` works
- How `WORKDIR` works
- How `COPY` works
- How `RUN` works
- How `CMD` works
- What `EXPOSE` does
- How to create a Dockerfile
- How to build a Docker image
- How to run a container from our image
- How to inspect an image
- How to view image layers
- How to version Docker images
- Why `.dockerignore` is important
- How to troubleshoot basic Dockerfile problems

The most important workflow to remember:

```text
Application Code
       │
       ▼
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

# 28. LinkedIn Post — Day 34/100

## Writing My First Dockerfile 🚀

Day 34 of my **100 Days of DevOps — From Zero to Hero** journey.

Until now, I had been working mostly with existing Docker images.

Today I created one myself.

I wrote my first Dockerfile and used it to package a simple Python application.

The basic flow finally became clear:

```text
Application
    ↓
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Container
```

I practiced some important Dockerfile instructions:

✅ `FROM` — choose the base image

✅ `WORKDIR` — define the working directory

✅ `COPY` — copy application files

✅ `CMD` — define what runs when the container starts

I also learned why `.dockerignore` matters and how Docker images are built using layers.

The biggest takeaway for me today:

**A Dockerfile turns application setup into something repeatable.**

Instead of manually preparing an environment every time, we can describe the environment as code and build the same image consistently.

Next step:

**Building and running Docker images with different versions and understanding the build process more deeply.**

Day 34 complete. 🚀

#100DaysOfDevOps #DevOps #Docker #Dockerfile #Containers #Cloud #DevOpsJourney #LearningInPublic #GitHub #DevOpsEngineer

---

# 29. What to Capture for LinkedIn

Capture these screenshots from your terminal:

### Screenshot 1 — Dockerfile

Show:

```bash
cat Dockerfile
```

### Screenshot 2 — Build

Show:

```bash
docker build -t my-python-app:v1 .
```

### Screenshot 3 — Image

Show:

```bash
docker images
```

### Screenshot 4 — Run

Show:

```bash
docker run --name my-python-container my-python-app:v1
```

### Screenshot 5 — Logs

Show:

```bash
docker logs my-python-container
```

A simple visual story:

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

# 30. Day 35 Preview

Tomorrow:

## Building and Running Docker Images

We will go deeper into:

- Docker image builds
- Build context
- Image tags
- Image versioning
- Docker build layers
- Running different image versions
- Image inspection
- Practical build challenge
- Troubleshooting Docker builds

See you on **Day 35/100** 🚀
