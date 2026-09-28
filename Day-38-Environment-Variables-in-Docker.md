# Day 38 — Environment Variables in Docker

## 100 Days of DevOps — From Zero to Hero

**Day:** 38/100  
**Topic:** Environment Variables in Docker  
**Category:** Docker

---

## 1. What Problem Are We Solving?

Imagine an application that needs configuration such as:

- Application environment
- Database hostname
- Database port
- API endpoint
- Application username
- Feature flags

We do not want to hard-code environment-specific values into the application.

A better approach is to keep configuration outside the application and provide it at runtime.

That is where **environment variables** come in.

```text
Application
     |
     v
Environment Variables
     |
     +---- APP_ENV
     +---- DATABASE_HOST
     +---- DATABASE_PORT
     +---- API_URL
```

---

## 2. What is an Environment Variable?

An environment variable is a key-value pair used to provide configuration information to a process or application.

Example:

```text
APP_ENV=production
```

Here:

```text
APP_ENV     → variable name
production  → variable value
```

---

## 3. Why Are Environment Variables Important in DevOps?

DevOps environments commonly have multiple stages:

```text
Development
      ↓
Testing
      ↓
Staging
      ↓
Production
```

The application code can remain the same while configuration changes.

For example:

### Development

```text
APP_ENV=development
DATABASE_HOST=dev-db
```

### Production

```text
APP_ENV=production
DATABASE_HOST=prod-db
```

The same application image can be reused across environments.

---

## 4. Environment Variables in Docker

Docker allows us to pass environment variables into containers.

The simplest method is:

```bash
docker run -e APP_ENV=production nginx
```

The general format is:

```text
-e VARIABLE=VALUE
```

---

## 5. Basic Docker Environment Variable Example

Run:

```bash
docker run --rm -e APP_ENV=development alpine env
```

Look for:

```text
APP_ENV=development
```

To display only the variable:

```bash
docker run --rm -e APP_ENV=development alpine sh -c 'echo $APP_ENV'
```

Expected:

```text
development
```

---

## 6. Passing Multiple Environment Variables

You can use multiple `-e` options:

```bash
docker run --rm \
  -e APP_ENV=development \
  -e APP_VERSION=1.0 \
  -e APP_PORT=8080 \
  alpine \
  sh -c 'echo "Environment: $APP_ENV"; echo "Version: $APP_VERSION"; echo "Port: $APP_PORT"'
```

Expected:

```text
Environment: development
Version: 1.0
Port: 8080
```

---

# 7. Hands-On Lab — Python Application

Create a directory:

```bash
mkdir docker-day38
cd docker-day38
```

Create `app.py`:

```python
import os

app_name = os.getenv("APP_NAME", "DefaultApp")
app_env = os.getenv("APP_ENV", "development")
app_version = os.getenv("APP_VERSION", "1.0")

print(f"Application: {app_name}")
print(f"Environment: {app_env}")
print(f"Version: {app_version}")
```

The important function is:

```python
os.getenv()
```

It reads an environment variable from the container environment.

---

## 8. Create the Dockerfile

Create `Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Project structure:

```text
docker-day38/
├── Dockerfile
└── app.py
```

---

## 9. Build the Image

```bash
docker build -t docker-day38-app .
```

Verify:

```bash
docker images
```

---

## 10. Run Without Environment Variables

```bash
docker run --rm docker-day38-app
```

Expected:

```text
Application: DefaultApp
Environment: development
Version: 1.0
```

These are the default values from `os.getenv()`.

---

## 11. Run With Environment Variables

```bash
docker run --rm \
  -e APP_NAME=DevOpsLearning \
  -e APP_ENV=production \
  -e APP_VERSION=2.0 \
  docker-day38-app
```

Expected:

```text
Application: DevOpsLearning
Environment: production
Version: 2.0
```

Notice that the image did not need to be rebuilt.

We changed only the runtime configuration.

---

## 12. Same Image, Different Environments

### Development

```bash
docker run --rm \
  -e APP_NAME=DevOpsLearning \
  -e APP_ENV=development \
  -e APP_VERSION=2.0 \
  docker-day38-app
```

### Production

```bash
docker run --rm \
  -e APP_NAME=DevOpsLearning \
  -e APP_ENV=production \
  -e APP_VERSION=2.0 \
  docker-day38-app
```

Conceptually:

```text
             Same Docker Image
                    |
          +---------+---------+
          |                   |
          v                   v
    Development           Production
          |                   |
   Different Config    Different Config
```

---

## 13. `--env` Instead of `-e`

Docker also supports the longer form:

```bash
docker run --rm \
  --env APP_ENV=production \
  alpine \
  sh -c 'echo $APP_ENV'
```

This is equivalent to:

```bash
docker run --rm -e APP_ENV=production alpine sh -c 'echo $APP_ENV'
```

---

## 14. Using an Environment File

When there are many variables, using multiple `-e` options can become inconvenient.

Create:

```text
app.env
```

Add:

```text
APP_NAME=DevOpsApp
APP_ENV=production
APP_VERSION=2.0
```

Run:

```bash
docker run --rm --env-file app.env docker-day38-app
```

Expected:

```text
Application: DevOpsApp
Environment: production
Version: 2.0
```

---

## 15. Environment File Format

The usual format is:

```text
KEY=value
```

Example:

```text
APP_NAME=DevOpsApp
APP_ENV=production
APP_VERSION=2.0
API_URL=https://api.example.com
```

Keep the syntax simple and consistent.

---

## 16. Dockerfile `ENV`

Docker also supports the `ENV` instruction.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

ENV APP_ENV=development

COPY app.py .

CMD ["python", "app.py"]
```

This creates a default environment variable in the image.

You can override it at runtime:

```bash
docker run --rm -e APP_ENV=production docker-day38-app
```

---

## 17. `ENV` vs Runtime `-e`

### Dockerfile

```dockerfile
ENV APP_ENV=development
```

Defines a default in the image.

### Runtime

```bash
docker run -e APP_ENV=production image-name
```

Supplies the runtime value.

Conceptually:

```text
Dockerfile ENV
      |
      v
Default Configuration
      |
      | can be overridden
      v
 docker run -e
      |
      v
Runtime Configuration
```

---

## 18. Security: Do Not Treat Environment Variables as a Secret Vault

Environment variables are useful for configuration, but they are not automatically a secure secrets-management solution.

Avoid committing real production credentials such as:

```text
DATABASE_PASSWORD=supersecret
API_KEY=secret-value
```

to Git.

For production, use appropriate secret-management mechanisms such as:

- Cloud secret managers
- CI/CD secret stores
- Kubernetes Secrets
- Docker secrets where applicable
- Vault or similar systems

Important lesson:

**Configuration and secrets are related, but they are not the same thing.**

---

## 19. `.gitignore` for Local Environment Files

If `app.env` contains sensitive local values, add it to `.gitignore`:

```text
app.env
```

Then:

```bash
git status
```

A safe example file can be committed instead:

```text
app.env.example
```

Example:

```text
APP_NAME=DevOpsApp
APP_ENV=development
APP_VERSION=1.0
```

This documents the required variables without exposing real secrets.

---

## 20. Checking Environment Variables

For a running container:

```bash
docker exec <container-name> env
```

You can also inspect a container:

```bash
docker inspect <container-name>
```

---

## 21. Hands-On Challenge

Build a small application that reads:

```text
APP_NAME
APP_ENV
APP_VERSION
```

Requirements:

1. Create a Python application.
2. Read variables using `os.getenv()`.
3. Create a Dockerfile.
4. Build the image.
5. Run without variables.
6. Run with `-e`.
7. Create an `app.env` file.
8. Run with `--env-file`.
9. Run the same image with development configuration.
10. Run the same image with production configuration.

Goal:

```text
Same Image
    |
    +---- Development Configuration
    |
    +---- Production Configuration
```

---

## 22. Troubleshooting

### Problem 1 — Variable is empty

Check the variable name carefully:

```text
APP_ENV
```

is different from:

```text
APP_ENVIRONMENT
```

Test:

```bash
docker run --rm -e APP_ENV=production alpine sh -c 'echo $APP_ENV'
```

---

### Problem 2 — `env-file` not found

Check:

```bash
ls
```

Make sure `app.env` exists in the current directory.

Then:

```bash
docker run --rm --env-file ./app.env docker-day38-app
```

---

### Problem 3 — Changes to `app.env` are not visible

Environment variables are supplied when the container starts.

Start a new container after changing the file:

```bash
docker run --rm --env-file app.env docker-day38-app
```

---

### Problem 4 — Secret accidentally committed to Git

Rotate/revoke the exposed credential immediately, then remove the secret from Git history using an appropriate history-cleaning process.

Most importantly:

**Never commit real production credentials to Git.**

---

## 23. Real-World DevOps Example

The same application image can be deployed to several environments:

```text
                 Docker Image
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Dev          Staging     Production
          |           |           |
          v           v           v
      Dev Config   Stage Config  Prod Config
```

This is especially useful in CI/CD pipelines:

```text
Git Push
   |
   v
Build Image
   |
   v
Test Image
   |
   v
Deploy
   |
   +---- Development Config
   |
   +---- Production Config
```

---

## 24. Key Takeaways

Today I learned:

- Environment variables provide runtime configuration.
- Docker can pass variables using `-e` or `--env`.
- Multiple variables can be passed to a container.
- `--env-file` allows configuration to be supplied from a file.
- Dockerfile `ENV` can define default environment variables.
- Runtime variables can override defaults.
- The same Docker image can be used with different configurations.
- Application code should remain independent of environment-specific configuration.
- Environment files containing secrets should not be committed to Git.
- Production secrets should use proper secret-management mechanisms.

The most important concept:

```text
Same Application Image
          |
          +---- Development Configuration
          |
          +---- Staging Configuration
          |
          +---- Production Configuration
```

---

## 25. Screenshot Guidance for LinkedIn

### Screenshot 1 — Environment Variable

Run:

```bash
docker run --rm -e APP_ENV=production alpine sh -c 'echo $APP_ENV'
```

Capture:

```text
production
```

### Screenshot 2 — Same Image, Different Configuration

Development:

```bash
docker run --rm \
  -e APP_NAME=DevOpsLearning \
  -e APP_ENV=development \
  -e APP_VERSION=2.0 \
  docker-day38-app
```

Production:

```bash
docker run --rm \
  -e APP_NAME=DevOpsLearning \
  -e APP_ENV=production \
  -e APP_VERSION=2.0 \
  docker-day38-app
```

This demonstrates:

```text
Same Image
    ↓
Different Runtime Configuration
```

### Screenshot 3 — `env-file`

```bash
docker run --rm --env-file app.env docker-day38-app
```

Capture the application output.

---

## 26. LinkedIn Post — Day 38/100

Today I learned an important DevOps concept:

**How can we use the same Docker image in different environments without changing the application code?**

Imagine an application that needs:

```text
APP_ENV
DATABASE_HOST
DATABASE_PORT
API_URL
```

Development, staging and production will usually have different values.

Hard-coding these values into the application isn't a good approach.

That's where **environment variables** come in.

Today I practiced:

✅ Passing environment variables with `docker run -e`

✅ Passing multiple variables

✅ Reading variables inside a Python application

✅ Using `--env-file`

✅ Using Dockerfile `ENV`

✅ Overriding default values at runtime

The most interesting part was seeing the **same Docker image** behave differently based only on runtime configuration.

```text
Same Docker Image
       |
       +---- Development Config
       |
       +---- Production Config
```

No rebuild was required.

That made one DevOps principle very clear to me:

**Keep application code separate from environment-specific configuration.**

I also learned an important security lesson:

**Environment variables are not automatically a secure secrets-management solution.**

Production credentials should be handled using proper secret-management tools instead of being committed to Git.

Day 38 complete. 🚀

Next up:

**Day 39 — Docker Compose**

#100DaysOfDevOps #DevOps #Docker #DockerEnvironment #Containers #Cloud #DevOpsJourney #LearningInPublic #DockerCompose #DevOpsEngineer

---

## 27. GitHub Commit

After saving this file:

```bash
git status
```

Add it:

```bash
git add Day-38-Environment-Variables-in-Docker.md
```

Commit:

```bash
git commit -m "Add Day 38 Docker Environment Variables"
```

Push:

```bash
git push origin main
```

---

## 28. Day 39 Preview

Tomorrow:

# Day 39 — Docker Compose

We will learn how to manage multiple containers using a single YAML file.

Instead of running many commands manually:

```text
docker run ...
docker run ...
docker network ...
docker volume ...
```

Docker Compose lets us describe the application:

```text
docker-compose.yml
       |
       +---- Web
       |
       +---- App
       |
       +---- Database
```

Then start the application with:

```bash
docker compose up
```

---

## Progress

**38/100 Days Completed 🚀**

```text
[███████████████████░░░░░░░░░░░] 38%
```

Keep learning. Keep practicing. Keep building.
