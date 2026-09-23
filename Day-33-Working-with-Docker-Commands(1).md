# 100 Days of DevOps — Day 33/100

## Working with Docker Commands

Today we focus on the Docker CLI and learn how to manage containers from the terminal.

### 1. Docker command structure

```bash
docker <command> <options> <arguments>
```

Examples:

```bash
docker ps
docker run -d --name web nginx
```

### 2. Check Docker

```bash
docker --version
docker version
docker info
```

- `docker --version` — Docker CLI version
- `docker version` — client/server version information
- `docker info` — Docker environment details

### 3. Work with images

List images:

```bash
docker images
docker image ls
```

Pull Nginx:

```bash
docker pull nginx
docker pull nginx:latest
```

Remove an image:

```bash
docker rmi nginx
```

Check containers first with:

```bash
docker ps -a
```

### 4. Run a container

```bash
docker run -d --name web-server -p 8080:80 nginx
```

Breakdown:

```text
docker run       → Create and start
-d               → Detached mode
--name           → Container name
-p 8080:80       → Port mapping
nginx            → Image
```

### 5. List containers

```bash
docker ps
docker ps -a
```

`docker ps` shows running containers. `docker ps -a` shows all containers.

### 6. Hands-on lab

Start Nginx:

```bash
docker run -d --name web-server -p 8080:80 nginx
```

Verify:

```bash
docker ps
```

Expected:

```text
CONTAINER ID   IMAGE   STATUS      PORTS
xxxxxxx        nginx   Up ...      0.0.0.0:8080->80/tcp
```

Test:

```bash
curl http://localhost:8080
```

Or open:

```text
http://localhost:8080
```

### 7. View logs

```bash
docker logs web-server
docker logs -f web-server
docker logs --tail 20 web-server
```

Press `Ctrl + C` to stop following logs.

### 8. Inspect a container

```bash
docker inspect web-server
```

Useful for checking:

- IP address
- Network
- Ports
- Mounts
- Environment
- State
- Configuration

### 9. Execute commands inside a container

```bash
docker exec -it web-server /bin/bash
```

If Bash is unavailable:

```bash
docker exec -it web-server /bin/sh
```

Inside:

```bash
pwd
ls
ls /usr/share/nginx/html
```

Exit:

```bash
exit
```

`docker exec` runs a command inside an already running container.

### 10. Container lifecycle

```text
             docker run
                  ↓
              Created
                  ↓
              Running
             /   |   \
            /    |    \
       stop     pause  restart
        ↓         ↓       ↓
     Stopped    Paused   Running
        |
        | start
        ↓
     Running
        |
        | rm
        ↓
     Removed
```

Stop:

```bash
docker stop web-server
```

Start:

```bash
docker start web-server
```

Restart:

```bash
docker restart web-server
```

Pause/unpause:

```bash
docker pause web-server
docker unpause web-server
```

### 11. Stop vs kill

Graceful stop:

```bash
docker stop web-server
```

Immediate kill:

```bash
docker kill web-server
```

Prefer `docker stop` when possible.

### 12. Remove a container

```bash
docker stop web-server
docker rm web-server
```

Force remove:

```bash
docker rm -f web-server
```

Use force removal carefully.

### 13. Container IDs and names

Every container has an ID and a name.

```text
CONTAINER ID   NAME
a1b2c3d4       web-server
```

Both can be used:

```bash
docker stop web-server
docker stop a1b2c3d4
```

### 14. View processes

```bash
docker top web-server
```

### 15. Check resource usage

```bash
docker stats
docker stats web-server
```

Useful metrics include CPU, memory, network I/O, and block I/O.

Press `Ctrl + C` to exit.

### 16. View port mappings

```bash
docker port web-server
```

Example:

```text
80/tcp -> 0.0.0.0:8080
```

Meaning:

```text
Host :8080
    ↓
Container :80
```

## 17. Full practical exercise

```bash
docker pull nginx
docker run -d --name devops-web -p 8080:80 nginx
docker ps
curl http://localhost:8080
docker logs devops-web
docker inspect devops-web
docker exec -it devops-web /bin/bash
```

Inside the container:

```bash
ls /usr/share/nginx/html
exit
```

Then:

```bash
docker top devops-web
docker stats devops-web
docker port devops-web
docker stop devops-web
docker start devops-web
docker restart devops-web
docker stop devops-web
docker rm devops-web
```

### Expected results

You should be able to:

- Pull an image
- Create and run a container
- Access Nginx through port 8080
- Read container logs
- Inspect configuration
- Execute commands inside the container
- View processes and resource usage
- Stop, start, restart, and remove a container

## 18. Command cheat sheet

| Command | Purpose |
|---|---|
| `docker --version` | Docker CLI version |
| `docker version` | Client/server version |
| `docker info` | Docker environment |
| `docker pull` | Download image |
| `docker images` | List images |
| `docker rmi` | Remove image |
| `docker run` | Create/start container |
| `docker ps` | Running containers |
| `docker ps -a` | All containers |
| `docker start` | Start stopped container |
| `docker stop` | Gracefully stop |
| `docker restart` | Restart |
| `docker kill` | Force stop |
| `docker rm` | Remove container |
| `docker rm -f` | Force remove |
| `docker logs` | View logs |
| `docker inspect` | Detailed configuration |
| `docker exec` | Run command inside container |
| `docker top` | Show processes |
| `docker stats` | Resource usage |
| `docker port` | Show port mappings |

## 19. Troubleshooting

### Container name already exists

```bash
docker ps -a
docker rm <container-name>
```

Or choose another name.

### Port already in use

```bash
docker ps
lsof -i :8080
```

Use another host port:

```bash
docker run -d --name devops-web -p 8081:80 nginx
```

### Container is not running

```bash
docker ps -a
docker logs <container-name>
```

### Cannot enter container

Make sure it is running:

```bash
docker ps
```

Then try:

```bash
docker exec -it <container-name> /bin/sh
```

Some minimal images do not contain Bash.

### Application is not responding

Check:

```bash
docker ps
docker port devops-web
curl http://localhost:8080
```

## 20. What to capture for LinkedIn

Capture a terminal screenshot showing:

```bash
docker ps
docker port devops-web
docker logs devops-web
```

A useful visual is:

```text
CONTAINER
devops-web
    |
    ├── Port: 8080 → 80
    ├── Status: Running
    └── Image: nginx
```

You can also capture:

```text
http://localhost:8080
```

showing the Nginx page.

## 21. Day 33 takeaway

The main lesson is the Docker container lifecycle:

```text
Pull Image
    ↓
Run Container
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
Restart
    ↓
Remove
```

Focus on these commands:

```bash
docker run
docker ps
docker logs
docker exec
docker inspect
docker stop
docker start
docker restart
docker rm
```

The goal is not to memorize every command. Understand what is happening to your container and which command helps you inspect or change its state.

# Day 33/100 — LinkedIn Post

Today was all about getting comfortable with the Docker CLI.

Learning Docker isn't just about knowing what a container is.

At some point, you need to actually manage those containers.

So today I practiced the complete container lifecycle:

**Create → Run → Inspect → Logs → Exec → Stop → Start → Restart → Remove**

Some of the commands I practiced:

```bash
docker run
docker ps
docker logs
docker inspect
docker exec
docker stop
docker start
docker restart
docker rm
```

I also explored a few commands that are useful when troubleshooting:

```bash
docker top
docker stats
docker port
```

One thing I'm realizing during this DevOps journey is that knowing the theory is only half the job.

The real learning starts when you open the terminal and actually break things, inspect them, fix them, and run them again.

Today's biggest takeaway:

**Don't just learn what a Docker command does. Understand when and why you would use it.**

That mindset is going to become even more important when we start building our own Docker images.

**Day 33 complete. 67 days to go.**

#100DaysOfDevOps #DevOps #Docker #Containers #DockerLearning #Cloud #CI #CD #DevOpsJourney #LearningInPublic

## Day 34 Preview

**Writing Your First Dockerfile**

Tomorrow we move from running existing images to building our own Docker image.

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
```

This is where Docker starts becoming much more powerful.
