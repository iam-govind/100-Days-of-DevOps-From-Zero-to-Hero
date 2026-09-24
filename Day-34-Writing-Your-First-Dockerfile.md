

Day-34-Writing-Your-First-Dockerfile.md

# 100 Days of DevOps — Day 34/100

## Writing Your First Dockerfile

We have now completed:

- Day 31 — Introduction to Containers and Docker
- Day 32 — Docker Images and Containers
- Day 33 — Working with Docker Commands

Today is an important step.

Until now, we've been using existing Docker images such as Nginx.

Today, we'll learn how to **build our own Docker image using a Dockerfile**.

---

## 1. What is a Dockerfile?

A **Dockerfile** is a text file containing instructions that Docker uses to build an image.

The basic workflow is:

```text
Application Code
      ↓
  Dockerfile
      ↓
 docker build
      ↓
 Docker Image
      ↓
  Container

Example project:

my-app/
│
├── Dockerfile
└── app.py

The Dockerfile describes how the application should be packaged.

2. Why Do We Need Dockerfiles?

Instead of manually creating a container every time an application needs to be deployed, we define the environment as code:

Dockerfile
    │
    ├── Base image
    ├── Application files
    ├── Dependencies
    ├── Configuration
    └── Startup command

Then anyone can build the same image:

docker build

This makes application packaging:

Repeatable
Consistent
Version controlled
Automatable
Suitable for CI/CD
3. Dockerfile Example

A simple Dockerfile:

FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]

Let's understand each instruction.

4. FROM
FROM python:3.12-slim

FROM specifies the base image.

Examples:

FROM ubuntu
FROM nginx
FROM python:3.12-slim

Think of it as:

"Start building my image from this existing image."

5. WORKDIR
WORKDIR /app

This sets the working directory inside the image/container.

Docker uses this directory for subsequent instructions and when the container starts.

6. COPY
COPY app.py .

This copies a file from your local build context into the image.

Local machine
     │
     │ COPY
     ▼
Docker image
7. CMD
CMD ["python", "app.py"]

This specifies the default command to run when the container starts.

Container starts
       ↓
python app.py
       ↓
Application runs
8. EXPOSE

You may also see:

EXPOSE 8080

This documents the port the application is expected to use.

Important:

EXPOSE does not publish the port to your host by itself.

You still need something like:

docker run -p 8080:8080 my-app
9. Our First Docker Project

Let's create a simple Python application.

Directory structure:

docker-day34/
│
├── Dockerfile
└── app.py
10. Hands-On Lab
Step 1 — Create a project directory

Run:

mkdir docker-day34
cd docker-day34
Step 2 — Create app.py

Run:

touch app.py

Add:

print("Hello from my first Docker application!")

Or create it directly:

cat > app.py <<'EOF'
print("Hello from my first Docker application!")
EOF

Verify:

cat app.py

Expected:

print("Hello from my first Docker application!")
Step 3 — Create the Dockerfile

Create:

touch Dockerfile

Add:

FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]

Or create it directly:

cat > Dockerfile <<'EOF'
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
EOF

Check it:

cat Dockerfile
11. Understand the Build Process

Your directory should now look like:

docker-day34/
│
├── Dockerfile
└── app.py

Docker reads:

Dockerfile
    │
    ├── FROM
    ├── WORKDIR
    ├── COPY
    └── CMD

and creates:

Docker Image
12. Build the Image

Run:

docker build -t my-python-app .

Breakdown:

docker build

Build an image.

-t my-python-app

Give the image a name/tag.

.

Use the current directory as the build context.

13. Expected Build Result

The exact output varies by Docker version, but the important result is that the build completes successfully.

The resulting image should be tagged as:

my-python-app:latest
14. Verify the Image

Run:

docker images

You should see something similar to:

REPOSITORY       TAG       IMAGE ID       CREATED
my-python-app    latest    xxxxxxxx       ...
15. Run Your Own Image

Now run:

docker run --name my-python-container my-python-app

Expected output:

Hello from my first Docker application!

🎉 You just built and ran your own Docker image.

16. Check the Container

Run:

docker ps -a

You should see:

my-python-container    my-python-app    Exited (0)

Why Exited?

Because the Python program prints one message and finishes.

There is no long-running process.

17. Check Container Logs

Run:

docker logs my-python-container

Expected:

Hello from my first Docker application!
18. Modify the Application

Change app.py:

print("Hello from Docker!")
print("This is Day 34 of my 100 Days of DevOps journey.")

Check:

cat app.py

Now rebuild:

docker build -t my-python-app:v2 .

Check:

docker images

You should now have:

my-python-app   latest
my-python-app   v2

This demonstrates the importance of image tags.

19. Run Version 2

Run:

docker run --name my-python-container-v2 my-python-app:v2

Expected:

Hello from Docker!
This is Day 34 of my 100 Days of DevOps journey.
20. Dockerfile Instructions We Learned
Instruction	Purpose
FROM	Select base image
WORKDIR	Set working directory
COPY	Copy files into image
CMD	Default startup command
EXPOSE	Document intended port
21. Dockerfile vs Docker Image vs Container

This distinction is very important.

Dockerfile
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker run
    ▼
Container
    │
    ▼
Application

Think about it like:

Dockerfile = Recipe
Docker Image = Prepared Package
Container = Running Instance
22. What is Build Context?

When you run:

docker build -t my-python-app .

the final . means:

Use the current directory as the build context.

Docker can access files inside that build context for instructions such as:

COPY

Example:

docker-day34/
│
├── Dockerfile
├── app.py
└── config.txt

Docker can copy files from the build context.

23. .dockerignore

Just like Git has .gitignore, Docker has:

.dockerignore

It prevents unnecessary files from being sent to the Docker build context.

Create:

touch .dockerignore

Example:

.git
.gitignore
README.md
*.log
.env

This is important because you should avoid sending unnecessary or sensitive files into the build context.

24. Check the Project

Your directory can now look like:

docker-day34/
│
├── Dockerfile
├── .dockerignore
└── app.py

Check:

ls -la
25. Troubleshooting
Dockerfile not found

If you see:

failed to read dockerfile

Check:

ls

Make sure the file is named exactly:

Dockerfile

Not:

Dockerfile.txt
Build context problem

Make sure you're running:

docker build -t my-python-app .

from the directory containing the Dockerfile.

Image doesn't exist

Check:

docker images

If the image is missing, rebuild:

docker build -t my-python-app .
Container exits immediately

For our application, this is expected because:

print(...)

finishes immediately.

Check:

docker ps -a

Then:

docker logs my-python-container
Changes aren't appearing

If you modify app.py, rebuild the image:

docker build -t my-python-app:v2 .

Then run the new image:

docker run --name my-python-container-v2 my-python-app:v2

Remember:

Changing local source code does not automatically change an already-built Docker image.

26. What to Capture for LinkedIn

Today's best screenshot is your terminal showing:

docker build -t my-python-app .
docker images
docker run --name my-python-container my-python-app

The most important output to capture is:

Hello from my first Docker application!

A second useful screenshot is:

docker images

showing:

my-python-app   latest
my-python-app   v2

This demonstrates:

Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Container
    ↓
Application
27. Day 34 Takeaway

Today you moved from:

Using Docker Images

to:

Building Docker Images

The most important workflow is:

Application Code
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

Remember:

Dockerfile = instructions

Image = packaged application

Container = running instance

This is the foundation for containerizing real applications.

Day 34/100 — LinkedIn Post

Today I moved from using Docker images to actually building my own Docker image.

This was an important step in my Docker learning journey.

The basic workflow I learned today is:

Application Code → Dockerfile → Docker Image → Container

I created a simple Python application and wrote my first Dockerfile using:

FROM
WORKDIR
COPY
CMD

Then I built the image:

docker build -t my-python-app .

And ran it:

docker run --name my-python-container my-python-app

The application successfully ran inside my own Docker container.

I also learned about:

Docker build context
Image tags
.dockerignore
Dockerfile instructions
Container logs
Rebuilding images after code changes

One concept that really clicked today:

Dockerfile = Recipe

Docker Image = Prepared Package

Container = Running Instance

This is where Docker starts becoming much more practical for DevOps.

The next step is to get more comfortable with building and running Docker images and understanding what happens during the build process.

Day 34 complete. 66 days to go.

#100DaysOfDevOps #DevOps #Docker #Dockerfile #Containers #DockerLearning #Cloud #CI #CD #DevOpsJourney #LearningInPublic

Day 35 Preview

Building and Running Docker Images

Tomorrow we'll go deeper into:

Dockerfile
    ↓
docker build
    ↓
Image
    ↓
Image Tags
    ↓
docker run
    ↓
Container

We'll work with image versions and make the build/run process more production-oriented.

please give me this in .md format

Absolutely. Which lesson would you like in .md format — Day 34 (Dockerfile) or Day 35 (Building and Running Docker Images)?

yes

Sure — I’ll take that as Day 35: Building and Running Docker Images. Here is the complete GitHub-ready Markdown.

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
1. What Does docker build Do?

A Dockerfile contains instructions that describe how our image should be created.

For example:

FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]

When we run:

docker build -t my-python-app .

Docker reads the Dockerfile and creates an image.

Dockerfile
     │
     │ docker build
     ▼
Docker Image

The . at the end is important.

It tells Docker:

Use the current directory as the build context.

2. What Is a Build Context?

The build context is the set of files Docker can access while building the image.

For example:

docker-day35/
├── Dockerfile
├── app.py
└── README.md

If we run:

docker build -t my-python-app .

the current directory becomes the build context.

Docker can access files inside this context.

For example:

COPY app.py .

copies app.py from the build context into the image.

3. Docker Image Tags

A Docker image can have a name and a tag.

Example:

docker build -t my-python-app:v1 .

Here:

my-python-app:v1
      │        │
      │        └── Tag
      │
      └────────── Image name

Another version:

docker build -t my-python-app:v2 .

Now we have two versions:

my-python-app:v1
my-python-app:v2

Tags are extremely useful for versioning Docker images.

4. Build an Image

Let's create a simple Python application.

Create a directory:

mkdir docker-day35
cd docker-day35

Create app.py:

cat > app.py <<'EOF'
print("Hello from Docker Day 35!")
print("This application is running inside a Docker container.")
EOF

Create the Dockerfile:

cat > Dockerfile <<'EOF'
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
EOF

Your directory should look like:

docker-day35/
├── Dockerfile
└── app.py
5. Build the Docker Image

Run:

docker build -t my-python-app:v1 .

You should see Docker processing the Dockerfile.

The final output should contain something similar to:

Successfully tagged my-python-app:v1

Depending on your Docker version, the output may look different because modern Docker commonly uses BuildKit.

6. Verify the Image

Run:

docker images

You should see something similar to:

REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
my-python-app    v1        xxxxxxxxxxxx   seconds ago      ...
python           3.12-slim xxxxxxxxxxxx   ...              ...

Our image is now ready.

7. Run the Docker Image

Now create a container from our image:

docker run --name my-python-container my-python-app:v1

Expected output:

Hello from Docker Day 35!
This application is running inside a Docker container.

The relationship is:

Docker Image
     │
     │ docker run
     ▼
Docker Container
8. Check the Container

Run:

docker ps -a

You should see:

CONTAINER ID   IMAGE               COMMAND          STATUS
xxxxxxxxxxxx   my-python-app:v1    "python app.py" Exited (0)

Why is the container showing Exited?

Because our Python program finished executing.

A container remains alive only while its main process is running.

For example:

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

This is expected behavior.

9. View Container Logs

Run:

docker logs my-python-container

Expected:

Hello from Docker Day 35!
This application is running inside a Docker container.
10. Build Version 2

Let's modify the application.

Edit app.py:

cat > app.py <<'EOF'
print("Hello from Docker Day 35!")
print("This is version 2 of my application.")
EOF

Now build another image:

docker build -t my-python-app:v2 .

Check the images:

docker images

You should now have:

my-python-app:v1
my-python-app:v2
11. Run Version 2

Run:

docker run --name my-python-container-v2 my-python-app:v2

Expected:

Hello from Docker Day 35!
This is version 2 of my application.

Now we have two versions of our application image:

my-python-app:v1
        │
        └── Container 1

my-python-app:v2
        │
        └── Container 2
12. Image Versioning

In real DevOps environments, image tags are commonly used to identify application versions.

For example:

myapp:1.0.0
myapp:1.1.0
myapp:1.2.0

Or:

myapp:dev
myapp:test
myapp:staging
myapp:production

A CI/CD pipeline may build an image like:

myapp:1.4.2

and push it to a container registry.

Examples of container registries include:

Docker Hub
GitHub Container Registry
Azure Container Registry
Amazon Elastic Container Registry
Google Artifact Registry
13. Docker Build Workflow

A typical Docker workflow looks like this:

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

This is one of the foundations of container-based DevOps.

14. Understanding Docker Build Layers

Docker images are built in layers.

For example:

FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]

Conceptually:

┌─────────────────────────────┐
│ CMD ["python", "app.py"]    │
├─────────────────────────────┤
│ COPY app.py .               │
├─────────────────────────────┤
│ WORKDIR /app                │
├─────────────────────────────┤
│ Python 3.12 Slim            │
└─────────────────────────────┘

Docker can reuse unchanged layers during subsequent builds.

This is one reason Docker builds can become much faster after the first build.

15. Inspect the Image

Run:

docker image inspect my-python-app:v1

This displays detailed information about the image.

You can also check the image history:

docker history my-python-app:v1

This is useful for understanding how the image was built.

16. Compare Image Versions

Run:

docker images my-python-app

You may see:

REPOSITORY       TAG       IMAGE ID       CREATED
my-python-app    v1        abc123         ...
my-python-app    v2        def456         ...

Each tag points to an image.

17. Clean Up Containers

List containers:

docker ps -a

Remove the containers:

docker rm my-python-container
docker rm my-python-container-v2

Verify:

docker ps -a
18. Remove Images

List images:

docker images

Remove version 1:

docker rmi my-python-app:v1

Remove version 2:

docker rmi my-python-app:v2

Verify:

docker images

Do not remove an image if containers still depend on it unless you intentionally want to clean them up.

19. Hands-On Challenge

Now try this yourself without copying the previous example exactly.

Task

Create a Docker image for a Python application.

Requirements

Create:

docker-day35-challenge/
├── Dockerfile
└── app.py

Your app.py should print:

Welcome to my DevOps journey!
Day 35 - Docker Image Build

Create a Dockerfile using:

FROM python:3.12-slim

Build the image with:

docker build -t devops-day35:v1 .

Run it:

docker run --name devops-day35-container devops-day35:v1

Check:

docker images

and:

docker ps -a

Finally check:

docker logs devops-day35-container
20. Expected Result

You should see:

Welcome to my DevOps journey!
Day 35 - Docker Image Build

And:

docker images

should show:

devops-day35    v1
21. Troubleshooting
Problem 1: Docker command not found

Run:

docker --version

If Docker is not available, make sure Docker Desktop is installed and running.

Problem 2: Dockerfile not found

If you see an error related to the Dockerfile, check:

ls

You should have:

Dockerfile
app.py

Then run:

docker build -t my-python-app:v1 .

Make sure you are inside the correct directory.

Problem 3: Image doesn't appear

Run:

docker images

If the image isn't listed, check the output of:

docker build -t my-python-app:v1 .

Look for errors during the build.

Problem 4: Container exits immediately

Run:

docker ps -a

Then:

docker logs my-python-container

For a short-lived Python script, an Exited (0) status is normal.

Problem 5: COPY app.py . fails

Check:

ls

Make sure app.py exists in the build context.

Also make sure you are running:

docker build -t my-python-app:v1 .

from the directory containing app.py.

22. Important Commands Learned Today
Command	Purpose
docker build	Build an image
docker images	List images
docker run	Create and run a container
docker logs	View container logs
docker history	View image layers/history
docker image inspect	Inspect image details
docker ps -a	List all containers
docker rm	Remove containers
docker rmi	Remove images
23. Key Takeaways

Today we learned:

What docker build does
What a Docker build context is
How to build an image from a Dockerfile
How to tag Docker images
How to run a container from an image
How image versions work
How Docker image layers work
How to inspect images
How to troubleshoot Docker builds
Why image versioning matters in DevOps

The important relationship to remember is:

Dockerfile
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker run
    ▼
Docker Container
24. LinkedIn Post — Day 35/100
Building and Running My First Docker Images 🚀

Day 35 of my 100 Days of DevOps — From Zero to Hero journey.

Yesterday I learned how to write a Dockerfile.

Today I went one step further:

I built my own Docker image and ran a container from it.

The workflow is simple:

Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Docker Container

One thing that became much clearer today was the difference between an image and a container.

An image is the packaged application.

A container is the running instance created from that image.

I also practiced:

✅ Building images with docker build

✅ Tagging images with versions

✅ Running containers

✅ Checking images with docker images

✅ Checking containers with docker ps -a

✅ Viewing logs with docker logs

✅ Understanding Docker build layers

I also created two versions of the same application:

my-python-app:v1
my-python-app:v2

This made the idea of container image versioning much more practical.

The next step is to understand something equally important:

Where does data go when a container is removed?

That's where Docker Volumes come in.

One more day. One more step toward understanding real-world DevOps.

#100DaysOfDevOps #DevOps #Docker #Containers #Cloud #DevOpsJourney #LearningInPublic #Dockerfile #GitHub #DevOpsEngineer

25. What to Capture for LinkedIn

Take screenshots of:

Screenshot 1 — Docker Build
docker build -t my-python-app:v1 .
Screenshot 2 — Docker Images
docker images
Screenshot 3 — Running the Container
docker run --name my-python-container my-python-app:v1
Screenshot 4 — Container Logs
docker logs my-python-container

A good LinkedIn carousel can show:

Dockerfile
   ↓
Build
   ↓
Image
   ↓
Run
   ↓
Container
26. Day 36 Preview

Tomorrow:

Docker Volumes

We will learn:

Why containers are ephemeral
What happens to data when a container is deleted
Docker volumes
Bind mounts
Creating volumes
Mounting volumes into containers
Sharing data between containers
Practical hands-on lab

See you on Day 36/100 🚀
