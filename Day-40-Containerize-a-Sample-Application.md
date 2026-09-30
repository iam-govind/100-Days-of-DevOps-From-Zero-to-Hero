# Day 40 — Containerize a Sample Application

## 100 Days of DevOps — From Zero to Hero

**Day:** 40/100  
**Topic:** Containerize a Sample Application  
**Category:** Docker

---

# 1. Today's Goal

Today is an important Docker milestone.

During Days 31–39, we learned Docker concepts one by one:

```text
Day 31 → Containers
Day 32 → Images
Day 33 → Docker Commands
Day 34 → Dockerfile
Day 35 → Building Images
Day 36 → Volumes
Day 37 → Networking
Day 38 → Environment Variables
Day 39 → Docker Compose
```

Today we combine those concepts into one practical project.

We will:

- Create a Python application
- Write a Dockerfile
- Build a Docker image
- Run a Docker container
- Publish a port
- Use environment variables
- Use a Docker volume
- Use Docker Compose
- Create a Docker network
- Test the complete application

---

# 2. What Does "Containerize an Application" Mean?

Containerizing an application means packaging the application and the required runtime into a Docker image so that it can run consistently in different environments.

Without containers:

```text
Developer Machine
       |
       +---- Python
       +---- Dependencies
       +---- Application
       +---- Configuration
```

With Docker:

```text
              Docker Image
                   |
          +--------+--------+
          |                 |
          v                 v
    Application          Runtime
          |
          v
       Container
```

The goal is:

> Build once and run consistently wherever Docker is available.

---

# 3. Today's Application

We will build a simple Python HTTP application.

The architecture will be:

```text
                  Browser
                     |
                     | localhost:8080
                     v
             +----------------+
             | Docker         |
             | Container      |
             |                |
             | Python App     |
             +----------------+
                    |
                    +---- Environment Variables
                    |
                    +---- Persistent Volume
```

---

# 4. Prerequisites

Check Docker:

```bash
docker --version
```

Check Docker Compose:

```bash
docker compose version
```

Verify Docker is running:

```bash
docker info
```

---

# 5. Create the Project

```bash
mkdir docker-day40
cd docker-day40
```

Create:

```text
app.py
```

Project:

```text
docker-day40/
└── app.py
```

---

# 6. Create the Python Application

Add this to `app.py`:

```python
import os
from http.server import BaseHTTPRequestHandler, HTTPServer

APP_NAME = os.getenv("APP_NAME", "Docker App")
APP_ENV = os.getenv("APP_ENV", "development")
APP_VERSION = os.getenv("APP_VERSION", "1.0")


class Handler(BaseHTTPRequestHandler):

    def do_GET(self):
        response = f"""
        <html>
        <head>
            <title>{APP_NAME}</title>
        </head>
        <body>
            <h1>{APP_NAME}</h1>
            <p>Environment: {APP_ENV}</p>
            <p>Version: {APP_VERSION}</p>
            <p>Running inside Docker 🚀</p>
        </body>
        </html>
        """

        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(response.encode())


server = HTTPServer(("0.0.0.0", 8080), Handler)

print(f"{APP_NAME} started on port 8080")

server.serve_forever()
```

The application reads configuration using `os.getenv()`.

---

# 7. Create the Dockerfile

Create:

```text
Dockerfile
```

Add:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

EXPOSE 8080

CMD ["python", "app.py"]
```

### Dockerfile instructions

`FROM` — selects the base image.

`WORKDIR` — sets the working directory.

`COPY` — copies the application into the image.

`EXPOSE` — documents the application's listening port.

`CMD` — starts the application.

---

# 8. Build the Docker Image

```bash
docker build -t docker-day40-app:v1 .
```

Check:

```bash
docker images
```

You should see:

```text
docker-day40-app    v1
```

---

# 9. Run the Container

```bash
docker run -d   --name docker-day40-container   -p 8080:8080   docker-day40-app:v1
```

Check:

```bash
docker ps
```

---

# 10. Test the Application

Open:

```text
http://localhost:8080
```

Or:

```bash
curl http://localhost:8080
```

You should receive HTML containing:

```text
Docker App
Environment: development
Version: 1.0
Running inside Docker
```

🎉 The application is now running inside Docker.

---

# 11. Check Container Logs

```bash
docker logs docker-day40-container
```

Expected:

```text
Docker App started on port 8080
```

---

# 12. Use Environment Variables

Remove the existing container:

```bash
docker rm -f docker-day40-container
```

Run it again with different configuration:

```bash
docker run -d   --name docker-day40-container   -p 8080:8080   -e APP_NAME=DevOpsApp   -e APP_ENV=production   -e APP_VERSION=2.0   docker-day40-app:v1
```

Test:

```bash
curl http://localhost:8080
```

You should now see:

```text
DevOpsApp
Environment: production
Version: 2.0
Running inside Docker
```

Important:

**We did not rebuild the image.**

Only the runtime configuration changed.

---

# 13. Why This Matters

The same Docker image can be used in different environments:

```text
              Same Docker Image
                     |
          +----------+----------+
          |          |          |
          v          v          v
        Dev       Staging     Production
          |          |          |
       Config     Config      Config
```

This is an important DevOps principle:

> Keep application code separate from environment-specific configuration.

---

# 14. Add a Docker Volume

Create a volume:

```bash
docker volume create day40-data
```

Verify:

```bash
docker volume ls
```

Remove the current container:

```bash
docker rm -f docker-day40-container
```

Run with the volume:

```bash
docker run -d   --name docker-day40-container   -p 8080:8080   -e APP_NAME=DevOpsApp   -e APP_ENV=production   -e APP_VERSION=2.0   -v day40-data:/data   docker-day40-app:v1
```

The relationship is:

```text
Container
    |
    v
/data
    |
    v
day40-data volume
```

---

# 15. Write Data to the Volume

```bash
docker exec docker-day40-container   sh -c "echo 'Docker Day 40' > /data/day40.txt"
```

Read it:

```bash
docker exec docker-day40-container   cat /data/day40.txt
```

Expected:

```text
Docker Day 40
```

---

# 16. Prove the Volume Persists

Remove the container:

```bash
docker rm -f docker-day40-container
```

Create a new container using the same volume:

```bash
docker run -d   --name docker-day40-container   -p 8080:8080   -v day40-data:/data   docker-day40-app:v1
```

Read the file:

```bash
docker exec docker-day40-container   cat /data/day40.txt
```

Expected:

```text
Docker Day 40
```

The data survived the original container being removed.

This connects directly to **Day 36 — Docker Volumes**.

---

# 17. Add Docker Compose

Create:

```text
compose.yaml
```

Add:

```yaml
services:
  app:
    build: .
    container_name: docker-day40-app

    ports:
      - "8080:8080"

    environment:
      APP_NAME: DevOpsApp
      APP_ENV: development
      APP_VERSION: "1.0"

    volumes:
      - app-data:/data

    networks:
      - devops-network

volumes:
  app-data:

networks:
  devops-network:
```

---

# 18. Final Project Structure

```text
docker-day40/
├── Dockerfile
├── app.py
└── compose.yaml
```

---

# 19. Start with Docker Compose

Remove the manually created container:

```bash
docker rm -f docker-day40-container
```

Start Compose:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

---

# 20. Test the Application

```bash
curl http://localhost:8080
```

Expected output contains:

```text
DevOpsApp
Environment: development
Version: 1.0
Running inside Docker
```

---

# 21. Check Compose Logs

```bash
docker compose logs
```

Or:

```bash
docker compose logs -f
```

Expected:

```text
DevOpsApp started on port 8080
```

Press `Ctrl+C` to stop following logs.

---

# 22. Test the Persistent Volume

Create a file:

```bash
docker compose exec app   sh -c "echo 'Persistent Docker data' > /data/test.txt"
```

Read it:

```bash
docker compose exec app   cat /data/test.txt
```

Expected:

```text
Persistent Docker data
```

Restart the application:

```bash
docker compose restart app
```

Read it again:

```bash
docker compose exec app   cat /data/test.txt
```

Expected:

```text
Persistent Docker data
```

---

# 23. Inspect the Docker Network

List networks:

```bash
docker network ls
```

You should see a Compose-created network.

You can inspect it with:

```bash
docker network inspect docker-day40_default
```

The exact network name can vary depending on your Compose project name.

---

# 24. Validate the Compose Configuration

```bash
docker compose config
```

This displays the resolved Compose configuration and helps identify YAML/configuration problems.

---

# 25. Final Architecture

```text
                       Browser
                          |
                          | :8080
                          v
                 +------------------+
                 | Docker Compose   |
                 +------------------+
                          |
                          v
                 +------------------+
                 | Python Container |
                 |                  |
                 | Python App       |
                 +------------------+
                    |            |
                    |            |
                    v            v
              Environment     app-data
               Variables        Volume
                    |
                    v
              APP_ENV=dev
              APP_VERSION=1.0
```

---

# 26. What We Combined

```text
Day 31 → Containers
Day 32 → Images
Day 33 → Docker Commands
Day 34 → Dockerfile
Day 35 → Building Images
Day 36 → Volumes
Day 37 → Networking
Day 38 → Environment Variables
Day 39 → Docker Compose
Day 40 → Complete Containerized Application
```

Today's project connects the individual Docker concepts into one workflow.

---

# 27. Troubleshooting

## Port 8080 is already in use

Check:

```bash
docker ps
```

Stop the container using the port:

```bash
docker stop <container-name>
```

Or change the Compose mapping:

```yaml
ports:
  - "8081:8080"
```

Then use:

```text
http://localhost:8081
```

---

## Container exits immediately

Check:

```bash
docker logs docker-day40-container
```

For Compose:

```bash
docker compose logs app
```

---

## Application is not reachable

Check:

```bash
docker compose ps
```

Then:

```bash
curl http://localhost:8080
```

---

## Build fails

Try:

```bash
docker compose build --no-cache
```

Then:

```bash
docker compose up -d
```

---

## Volume problem

Check:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect day40-data
```

For Compose, the volume may have a project-prefixed name.

---

## Compose configuration error

Run:

```bash
docker compose config
```

Check:

- YAML indentation
- Service names
- Port mappings
- Volume names
- Network names

---

# 28. Cleanup

Stop the Compose application:

```bash
docker compose down
```

To also remove Compose-managed volumes:

```bash
docker compose down -v
```

Be careful: `-v` removes Compose-managed volumes and their data.

Remove the manually created test volume:

```bash
docker volume rm day40-data
```

---

# 29. Important Commands

### Build

```bash
docker build -t docker-day40-app:v1 .
```

### Run

```bash
docker run -d   --name docker-day40-container   -p 8080:8080   docker-day40-app:v1
```

### List containers

```bash
docker ps
```

### Logs

```bash
docker logs docker-day40-container
```

### Create volume

```bash
docker volume create day40-data
```

### List volumes

```bash
docker volume ls
```

### Start Compose

```bash
docker compose up -d
```

### Compose services

```bash
docker compose ps
```

### Compose logs

```bash
docker compose logs
```

### Restart service

```bash
docker compose restart app
```

### Validate Compose

```bash
docker compose config
```

### Stop Compose

```bash
docker compose down
```

---

# 30. Hands-On Challenge

Try rebuilding the project without copying the complete solution.

Target:

```text
                 Docker Compose
                       |
                       v
               Python Application
                  /                           /                            v              v
        Environment        Persistent
         Variables           Volume
```

Requirements:

1. Create a Python HTTP application.
2. Read configuration using `os.getenv()`.
3. Create a Dockerfile.
4. Build an image.
5. Run the application on port 8080.
6. Pass environment variables.
7. Create a Docker volume.
8. Write a file into the volume.
9. Delete the container.
10. Recreate it.
11. Verify the file still exists.
12. Create a `compose.yaml`.
13. Configure the application with Compose.
14. Add a volume.
15. Add a network.
16. Start with:

```bash
docker compose up -d
```

17. Test:

```bash
curl http://localhost:8080
```

18. Check:

```bash
docker compose ps
```

19. Check:

```bash
docker compose logs
```

---

# 31. Screenshot Guidance for LinkedIn

## Screenshot 1 — Docker Image

```bash
docker images
```

Capture:

```text
docker-day40-app
v1
```

## Screenshot 2 — Running Container

```bash
docker ps
```

Show:

```text
0.0.0.0:8080->8080/tcp
```

## Screenshot 3 — Application Response

```bash
curl http://localhost:8080
```

Capture:

```text
DevOpsApp
Environment: development
Version: 1.0
Running inside Docker
```

## Screenshot 4 — Docker Compose

```bash
docker compose ps
```

Capture the application running through Compose.

## Screenshot 5 — Persistent Data

```bash
docker compose exec app cat /data/test.txt
```

Capture:

```text
Persistent Docker data
```

### Recommended LinkedIn screenshots

Use **Screenshot 3 + Screenshot 4**.

---

# 32. LinkedIn Post — Day 40/100

Today was an important milestone in my **100 Days of DevOps** journey. 🚀

For the last few days, I've been learning Docker one concept at a time.

Today I finally put those concepts together and **containerized a complete sample application**.

I started with a simple Python application and went through the complete flow:

```text
Application
    ↓
Dockerfile
    ↓
Docker Image
    ↓
Docker Container
```

Then I added the pieces needed for a more realistic setup:

✅ Environment variables

✅ Port mapping

✅ Persistent Docker volume

✅ Docker networking

✅ Docker Compose

The final setup looked like:

```text
              Docker Compose
                    |
                    v
             Python Application
                /                         /                          v              v
      Environment         Persistent
       Variables            Volume
```

One thing I really understood today:

**Containerization isn't just about putting an application inside a container.**

It's about creating a repeatable way to:

- Build the application
- Configure it
- Run it
- Store its data
- Connect it to other services
- Manage the complete environment

And the best part?

The same Docker image can be configured differently for different environments.

```text
Same Image
    |
    +---- Development
    |
    +---- Staging
    |
    +---- Production
```

Day 40 complete. 🎯

The Docker fundamentals section is now complete.

Next up: **Azure Cloud** ☁️

#100DaysOfDevOps #DevOps #Docker #DockerCompose #Containers #Cloud #DevOpsJourney #LearningInPublic #Containerization #DevOpsEngineer

---

# 33. GitHub Commit

Save this file as:

```text
Day-40-Containerize-a-Sample-Application.md
```

Check:

```bash
git status
```

Add:

```bash
git add Day-40-Containerize-a-Sample-Application.md
```

Commit:

```bash
git commit -m "Add Day 40 Containerize Sample Application"
```

Push:

```bash
git push origin main
```

---

# 34. Day 41 Preview

## Day 41 — Introduction to Cloud Computing and Azure

Tomorrow we start the Azure Cloud section.

We will learn:

- What cloud computing is
- Why organizations use cloud platforms
- IaaS, PaaS and SaaS
- Public, private and hybrid cloud
- What Microsoft Azure is
- Major Azure services
- Azure regions
- How Azure fits into DevOps

Basic idea:

```text
                  Azure Cloud
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Compute      Storage    Networking
          |           |           |
          +-----------+-----------+
                      |
                      v
                  Applications
```

---

# 35. Progress

**40/100 Days Completed 🚀**

```text
[████████████████████░░░░░░░░░░] 40%
```

**Docker section complete. Azure starts next. ☁️🚀**

Keep learning. Keep practicing. Keep building.
