# Day 39 — Docker Compose

## 100 Days of DevOps — From Zero to Hero

**Day:** 39/100  
**Topic:** Docker Compose  
**Category:** Docker

---

## 1. What Problem Are We Solving?

So far, we have been running Docker containers individually:

```bash
docker run -d --name web nginx
```

This works well for one container. But real applications usually contain multiple services:

```text
                Application
                    |
        +-----------+-----------+
        |                       |
        v                       v
   Web Container          Database Container
      Nginx                    MySQL
```

Managing every container, network, volume and environment variable manually quickly becomes difficult.

Docker Compose lets us define the complete multi-container application in one YAML file.

---

## 2. What is Docker Compose?

Docker Compose is a tool for defining and running multi-container applications using a YAML configuration file.

A Compose file can define:

- Services
- Images
- Ports
- Networks
- Volumes
- Environment variables
- Service configuration

The application can then be started with:

```bash
docker compose up
```

---

## 3. Docker Compose Architecture

```text
                 Docker Compose
                       |
             +---------+---------+
             |                   |
             v                   v
        Web Service         Database Service
          Nginx                  MySQL
             |                   |
             +---------+---------+
                       |
                  Docker Network
```

---

## 4. Why Do We Need Docker Compose?

Without Compose:

```text
Create Network
      ↓
Create Volume
      ↓
Start Database
      ↓
Start Application
      ↓
Start Web Server
      ↓
Configure Everything
```

With Compose:

```text
        compose.yaml
             |
             v
      docker compose up
             |
      +------+------+
      |             |
      v             v
    Web           Database
```

The application configuration lives in one place.

---

## 5. Your First Compose File

Create a directory:

```bash
mkdir docker-day39
cd docker-day39
```

Create `compose.yaml`:

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

Project structure:

```text
docker-day39/
└── compose.yaml
```

---

## 6. Start the Application

Run:

```bash
docker compose up
```

Open:

```text
http://localhost:8080
```

You should see the Nginx welcome page.

Run in the background with:

```bash
docker compose up -d
```

---

## 7. Check Compose Services

```bash
docker compose ps
```

You should see the `web` service running.

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

View only the web service:

```bash
docker compose logs web
```

---

## 8. Stop the Application

```bash
docker compose down
```

Start it again:

```bash
docker compose up -d
```

---

## 9. Understanding the Compose File

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

### `services`

Defines the containers/services in the application.

### `web`

The service name.

### `image`

The Docker image used by the service.

### `ports`

Publishes a host port to a container port:

```text
Host:8080
    ↓
Container:80
```

---

## 10. Hands-On Lab — Two Services

Let's create a small multi-container application using Nginx and Redis.

Update `compose.yaml`:

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"

  redis:
    image: redis:7-alpine
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

You should see both services running.

---

## 11. Automatic Compose Networking

Compose creates an application network for the services.

```text
             Docker Compose
                    |
          +---------+---------+
          |                   |
          v                   v
       Web Service       Redis Service
         Nginx              Redis
```

Services can communicate using service names.

For example, the Redis service is reachable using the hostname:

```text
redis
```

This builds on the Docker networking concepts from Day 37.

---

## 12. Test Redis

Run:

```bash
docker compose exec redis redis-cli ping
```

Expected:

```text
PONG
```

You can also open a shell:

```bash
docker compose exec redis sh
```

Then:

```bash
redis-cli ping
```

Exit:

```bash
exit
```

---

## 13. Compose Environment Variables

Compose can provide environment variables to services:

```yaml
services:
  web:
    image: nginx
    environment:
      APP_ENV: development
      APP_VERSION: "1.0"
    ports:
      - "8080:80"
```

This connects directly to Day 38.

---

## 14. Using an Environment File

Create `app.env`:

```text
APP_ENV=development
APP_VERSION=1.0
```

Use it in Compose:

```yaml
services:
  web:
    image: nginx
    env_file:
      - app.env
    ports:
      - "8080:80"
```

For real projects, do not commit files containing production secrets.

---

## 15. Compose Volumes

Compose can manage persistent volumes:

```yaml
services:
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

Architecture:

```text
Redis Container
      |
      v
redis-data
      |
      v
Persistent Data
```

This connects to Day 36 — Docker Volumes.

---

## 16. Compose Networks

You can define a custom network:

```yaml
services:
  web:
    image: nginx
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    networks:
      - app-network

networks:
  app-network:
```

Architecture:

```text
             app-network
            /           \
           /             \
          v               v
        Web             Redis
      Container        Container
```

---

## 17. Complete Compose Example

This combines concepts from Days 36, 37 and 38:

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    environment:
      APP_ENV: development
      APP_VERSION: "1.0"
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - app-network

volumes:
  redis-data:

networks:
  app-network:
```

This one file defines services, ports, environment variables, networking and persistent storage.

---

## 18. Useful Docker Compose Commands

Start:

```bash
docker compose up
```

Start in background:

```bash
docker compose up -d
```

Stop/remove application containers and network:

```bash
docker compose down
```

List services:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

Service logs:

```bash
docker compose logs web
```

Execute a command:

```bash
docker compose exec redis redis-cli ping
```

Pull images:

```bash
docker compose pull
```

Restart services:

```bash
docker compose restart
```

Validate configuration:

```bash
docker compose config
```

---

## 19. `docker compose down` vs `docker compose down -v`

Normal cleanup:

```bash
docker compose down
```

This removes the containers and application network, while named volumes are normally preserved.

To also remove Compose-managed volumes:

```bash
docker compose down -v
```

Be careful: removing volumes can delete persistent application data.

---

## 20. Hands-On Project

Build this setup:

```text
             Docker Compose
                    |
          +---------+---------+
          |                   |
          v                   v
        Nginx               Redis
        :8080                :6379
          |                   |
          +---------+---------+
                    |
               app-network
```

Create:

```text
docker-day39/
├── compose.yaml
└── app.env
```

Use this `compose.yaml`:

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    env_file:
      - app.env
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - app-network

volumes:
  redis-data:

networks:
  app-network:
```

Create `app.env`:

```text
APP_ENV=development
APP_VERSION=1.0
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Test Nginx:

```bash
curl http://localhost:8080
```

Test Redis:

```bash
docker compose exec redis redis-cli ping
```

Expected:

```text
PONG
```

View logs:

```bash
docker compose logs
```

Stop:

```bash
docker compose down
```

---

## 21. Troubleshooting

### Problem 1 — `docker compose` is not recognized

Check:

```bash
docker compose version
```

Make sure Docker and the Compose plugin are installed correctly.

### Problem 2 — Port 8080 is already in use

Change:

```yaml
ports:
  - "8081:80"
```

Then access:

```text
http://localhost:8081
```

### Problem 3 — Service is not running

Check:

```bash
docker compose ps
```

Then:

```bash
docker compose logs <service>
```

### Problem 4 — Configuration error

Validate:

```bash
docker compose config
```

### Problem 5 — Redis is not responding

Check:

```bash
docker compose logs redis
```

Then:

```bash
docker compose exec redis redis-cli ping
```

Expected:

```text
PONG
```

---

## 22. YAML Indentation

YAML depends on indentation.

Correct:

```yaml
services:
  web:
    image: nginx
```

Avoid:

```yaml
services:
web:
image: nginx
```

Use spaces consistently and avoid mixing tabs and spaces.

---

## 23. Real-World DevOps Usage

Docker Compose is useful for:

- Local development
- Testing
- Development environments
- Small multi-container applications
- Learning microservices
- Reproducing application environments

A developer can clone a project and potentially start the required services with:

```bash
docker compose up -d
```

---

## 24. Docker Compose in CI/CD

A simplified testing pipeline can look like:

```text
Git Push
   |
   v
CI Pipeline
   |
   v
Build Application
   |
   v
Start Dependencies
   |
   v
docker compose up
   |
   v
Run Tests
   |
   v
docker compose down
```

This gives the CI environment a predictable set of services for testing.

---

## 25. Docker Compose vs Docker Run

### Docker Run

Useful for an individual container:

```bash
docker run -d --name web -p 8080:80 nginx
```

### Docker Compose

Useful for related services defined together:

```bash
docker compose up -d
```

The main benefit is having the application's desired configuration in one declarative file.

---

## 26. Hands-On Challenge

Build this yourself without copying the complete project:

```text
Web
 |
 +---- Nginx
 |
 +---- Redis
```

Requirements:

1. Create `compose.yaml`.
2. Add an Nginx service.
3. Publish Nginx on port `8080`.
4. Add a Redis service.
5. Put both services on a custom network.
6. Add a named volume to Redis.
7. Add environment variables to the web service.
8. Start with:

```bash
docker compose up -d
```

9. Verify:

```bash
docker compose ps
```

10. Test Nginx:

```bash
curl http://localhost:8080
```

11. Test Redis:

```bash
docker compose exec redis redis-cli ping
```

12. View logs:

```bash
docker compose logs
```

13. Stop:

```bash
docker compose down
```

---

## 27. Key Takeaways

Today I learned:

- Docker Compose manages multi-container applications.
- Compose configuration is written in YAML.
- `docker compose up` starts services.
- `docker compose down` removes application containers and networks.
- `docker compose ps` shows service status.
- `docker compose logs` helps troubleshoot services.
- Compose can manage networks.
- Compose can manage volumes.
- Compose can provide environment variables.
- Services can communicate using service names.
- `docker compose config` helps validate configuration.

The most important concept:

```text
              compose.yaml
                   |
        +----------+----------+
        |          |          |
        v          v          v
      Web        App       Database
        |          |          |
        +----------+----------+
                   |
             Docker Network
                   |
                Volumes
```

---

## 28. Screenshot Guidance for LinkedIn

### Screenshot 1 — Compose Services Running

Run:

```bash
docker compose up -d
docker compose ps
```

Capture the terminal showing both `web` and `redis` running.

### Screenshot 2 — Redis Connectivity

Run:

```bash
docker compose exec redis redis-cli ping
```

Capture:

```text
PONG
```

### Screenshot 3 — Nginx

Run:

```bash
curl http://localhost:8080
```

Capture the Nginx response.

### Screenshot 4 — Compose Configuration

Run:

```bash
docker compose config
```

For LinkedIn, Screenshots 1 and 2 are enough to demonstrate the practical work.

---

## 29. LinkedIn Post — Day 39/100

Today I reached another important Docker concept:

**Docker Compose**

So far, I was running containers individually using `docker run`.

That works fine for one container.

But real applications usually have multiple services:

```text
Web
 ↓
Application
 ↓
Database
```

Managing each container, network, volume and configuration separately can quickly become difficult.

Docker Compose solves this by allowing us to define the application in a single YAML file.

Today I practiced:

✅ Creating a `compose.yaml`

✅ Running multiple services

✅ Starting services with `docker compose up`

✅ Running Compose in detached mode

✅ Checking services with `docker compose ps`

✅ Viewing logs

✅ Creating networks

✅ Using volumes

✅ Passing environment variables

✅ Communicating between services

My hands-on setup looked like:

```text
        Docker Compose
              |
       +------+------+
       |             |
       v             v
     Nginx          Redis
       |             |
       +------+------+
              |
         Docker Network
```

One command can now bring up the entire environment:

```bash
docker compose up -d
```

And when I'm finished:

```bash
docker compose down
```

What I like about Compose is that the application setup becomes **declarative**.

Instead of remembering a long list of Docker commands, the configuration lives in:

```text
compose.yaml
```

This makes local development and multi-container testing much easier to reproduce.

Day 39 complete. 🚀

Next up:

**Day 40 — Containerize a Sample Application**

#100DaysOfDevOps #DevOps #Docker #DockerCompose #Containers #Cloud #DevOpsJourney #LearningInPublic #DevOpsEngineer #DockerNetworking

---

## 30. GitHub Commit

```bash
git status
git add Day-39-Docker-Compose.md
git commit -m "Add Day 39 Docker Compose"
git push origin main
```

---

## 31. Day 40 Preview

### Day 40 — Containerize a Sample Application

We will combine the Docker concepts from Days 31–39 into a practical project.

We will:

- Create an application
- Write a Dockerfile
- Build an image
- Run a container
- Configure environment variables
- Use persistent storage
- Configure networking
- Use Docker Compose
- Test the complete application

The goal is to move from individual Docker concepts to a complete containerized application.

---

## Progress

**39/100 Days Completed 🚀**

```text
[███████████████████░░░░░░░░░░░] 39%
```

Keep learning. Keep practicing. Keep building.
