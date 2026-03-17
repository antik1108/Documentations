# 🐳 Docker & Containerisation — Practical Notes
> **Arch Linux Edition** | Install → Run → Understand → Remember

---

## Table of Contents
- [1. The Problem Docker Solves](#1-the-problem-docker-solves)
- [2. Key Concepts](#2-key-concepts)
- [3. Installing Docker on Arch Linux](#3-installing-docker-on-arch-linux)
- [4. Pull and Run a Node.js Container](#4-pull-and-run-a-nodejs-container)
- [5. Running a Web Server & Accessing localhost](#5-running-a-web-server--accessing-localhost)
- [6. Docker Commands — Complete Reference](#6-docker-commands--complete-reference)
- [7. Common Workflows & Patterns](#7-common-workflows--patterns)
- [8. Common Mistakes & Fixes](#8-common-mistakes--fixes)
- [9. Quick-Fire Cheatsheet](#9-quick-fire-cheatsheet)

---

## 1. The Problem Docker Solves

Your code works on your machine, but breaks on someone else's. Here's why:

| Problem | Example |
|---|---|
| **Different OS** | Windows vs macOS vs Linux behave differently |
| **Missing dependencies** | The other person doesn't have the right libraries |
| **Wrong runtime version** | Python 3.11 vs 3.8 · Node 18 vs Node 14 |
| **Env variable mismatch** | Secrets / paths configured differently |
| **Painful onboarding** | "Install this, then downgrade that, then..." 😵 |

> 💡 **Docker's answer:** Package the app + its entire environment into a container. Ship the container. It runs the same everywhere.

---

## 2. Key Concepts

### Image vs Container

| | Docker Image | Docker Container |
|---|---|---|
| **Analogy** | Blueprint / Recipe | The actual house / cooked meal |
| **State** | Read-only, immutable | Running, has its own writable layer |
| **Created with** | `docker build` / `docker pull` | `docker run <image>` |
| **Lives on disk?** | Yes — stored locally | Yes — while running or stopped |
| **Multiple instances?** | One image | Many containers from one image |

### VM vs Docker

| Feature | Virtual Machine | Docker |
|---|---|---|
| **OS** | Each VM has its own full guest OS | Containers share the host OS kernel |
| **Size** | Large (GBs) | Small (MBs) |
| **Startup time** | Slow (minutes) | Fast (seconds or less) |
| **Resource usage** | High (CPU, RAM, storage) | Low and efficient |
| **Performance** | Slower due to full OS overhead | Near-native performance |
| **Isolation** | Strong (hardware-level) | Process-level |
| **Portability** | Less portable | Highly portable |

---

## 3. Installing Docker on Arch Linux

### Step 1 — Install the packages

```bash
# Install Docker engine + CLI
sudo pacman -S docker

# (Optional but recommended) Install Docker Compose
sudo pacman -S docker-compose
```

### Step 2 — Start & enable the Docker daemon

The daemon (`dockerd`) is the background service that manages containers. Start it once, enable it for auto-start on boot.

```bash
# Start Docker service right now
sudo systemctl start docker

# Enable it to start automatically on every boot
sudo systemctl enable docker

# Check it's running
sudo systemctl status docker
```

> ✅ **What to look for:** In the status output you should see `active (running)` in green.
> If it says `failed`, run `journalctl -xe` to see the error.

### Step 3 — Add your user to the docker group

Without this, every docker command needs `sudo`.

```bash
# Add yourself to the docker group
sudo usermod -aG docker $USER

# IMPORTANT: Log out and log back in for the group to take effect
# Or run this in your current terminal session:
newgrp docker
```

### Step 4 — Verify the installation

```bash
# Check Docker version
docker --version

# Run the hello-world test container
docker run hello-world

# Expected output (last line):
# Hello from Docker!
```

> 🎯 **Milestone:** If you see `Hello from Docker!` — Docker is installed and working correctly.

---

## 4. Pull and Run a Node.js Container

### Pull the Node image

```bash
# Pull the official Node.js image (latest version)
docker pull node

# Pull a specific version (recommended for production)
docker pull node:20
docker pull node:20-alpine       # Smaller, Alpine Linux based (~50 MB vs ~350 MB)

# Verify it downloaded
docker images
```

### Why did `docker run node` exit immediately?

A container runs **one main program**. When that program ends, the container stops.
The Node image starts the Node REPL — but without a terminal attached, there's nothing to do, so Node exits instantly.

> 📌 **Mental model:** Container lifecycle = program lifecycle. Container alive = program running. Program exits = container stops.

### Run Node interactively

```bash
# -i  = keep stdin open (interactive)
# -t  = allocate a terminal (pseudo-TTY)
docker run -it node

# You'll see the Node.js REPL:
# Welcome to Node.js v20.x.x
# >

# Try it:
# > console.log('Hello from Docker!')
# > 2 + 2
# > .exit    ← to quit
```

### Run a one-liner Node script

```bash
# Run a one-liner script
docker run node node -e "console.log('Hello from container!')"

# Run with a name so it's easy to reference later
docker run --name my-node node node -e "console.log('Named container!')"
```

---

## 5. Running a Web Server & Accessing localhost

### The port isolation problem

By default, containers are isolated — their ports are **NOT** accessible from your machine.
A Node app on port 3000 inside a container ≠ `localhost:3000` on your host.

> 🔑 **Key concept:** You must map a container port to a host port using `-p host_port:container_port`

### Quickest example — Nginx web server

```bash
# -p 8080:80 means:
#   host port 8080  →  container port 80
# -d means run in detached (background) mode
docker run -d -p 8080:80 --name my-nginx nginx

# Now open your browser: http://localhost:8080
# You should see the Nginx welcome page!
```

### Full workflow — Node.js Express app on localhost

#### Step 1 — Create your app files

```bash
mkdir my-app && cd my-app

# Create app.js
cat > app.js << 'EOF'
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello from Docker! 🐳'));
app.listen(3000, () => console.log('Server running on port 3000'));
EOF

# Create package.json
cat > package.json << 'EOF'
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": { "express": "^4.18.0" }
}
EOF
```

#### Step 2 — Create a Dockerfile

```dockerfile
# Start from the official Node image
FROM node:20-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package files first (Docker layer caching)
COPY package.json .

# Install dependencies
RUN npm install

# Copy the rest of the app
COPY . .

# Tell Docker which port the app listens on (documentation only)
EXPOSE 3000

# The command to run when the container starts
CMD ["node", "app.js"]
```

#### Step 3 — Build the image

```bash
# -t  = tag/name the image
# .   = build context (current folder)
docker build -t my-node-app .

# Verify the image was created
docker images
```

#### Step 4 — Run it & access on localhost

```bash
# Map host port 3000 to container port 3000
docker run -d -p 3000:3000 --name my-app-container my-node-app

# Check it's running
docker ps

# Test in terminal
curl http://localhost:3000

# Or open browser: http://localhost:3000
# Output: Hello from Docker! 🐳
```

> 🧠 **Port mapping format:** `-p HOST:CONTAINER` — Think of it as: "traffic arriving at host port X should go to container port Y"

---

## 6. Docker Commands — Complete Reference

### Image Commands

| Command | What it does | Example |
|---|---|---|
| `docker pull <image>` | Download image from registry | `docker pull node:20` |
| `docker images` | List all local images | `docker images` |
| `docker rmi <image>` | Remove an image | `docker rmi node:20` |
| `docker build -t name .` | Build image from Dockerfile | `docker build -t my-app .` |
| `docker push <image>` | Upload image to registry | `docker push myuser/my-app` |
| `docker image prune` | Remove unused/dangling images | `docker image prune` |
| `docker inspect <image>` | Show detailed image info (JSON) | `docker inspect nginx` |
| `docker history <image>` | Show image layers history | `docker history node` |

### Container Lifecycle Commands

| Command | What it does | Example |
|---|---|---|
| `docker run <image>` | Create + start a container | `docker run node` |
| `docker run -it <image>` | Run with interactive terminal | `docker run -it node` |
| `docker run -d <image>` | Run in detached (background) mode | `docker run -d nginx` |
| `docker run -p H:C <image>` | Map host:container ports | `docker run -p 8080:80 nginx` |
| `docker run --name n <image>` | Give container a name | `docker run --name web nginx` |
| `docker run --rm <image>` | Auto-remove container when it stops | `docker run --rm node` |
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers (incl. stopped) | `docker ps -a` |
| `docker stop <id/name>` | Gracefully stop a container | `docker stop my-app` |
| `docker start <id/name>` | Start a stopped container | `docker start my-app` |
| `docker restart <id/name>` | Restart a container | `docker restart my-app` |
| `docker rm <id/name>` | Remove a stopped container | `docker rm my-app` |
| `docker rm -f <id/name>` | Force remove (even if running) | `docker rm -f my-app` |
| `docker logs <id/name>` | View container stdout/stderr logs | `docker logs my-app` |
| `docker logs -f <id/name>` | Follow logs in real time | `docker logs -f my-app` |
| `docker exec -it <id> bash` | Open shell inside running container | `docker exec -it my-app bash` |
| `docker exec -it <id> sh` | Open sh (for Alpine images) | `docker exec -it my-app sh` |

### Useful Flags for `docker run`

| Flag | What it does | Example |
|---|---|---|
| `-d` | Detached mode (run in background) | `docker run -d nginx` |
| `-it` | Interactive + TTY terminal | `docker run -it node` |
| `-p H:C` | Port mapping host:container | `docker run -p 3000:3000 myapp` |
| `--name` | Assign a name to the container | `docker run --name web nginx` |
| `--rm` | Remove container after it stops | `docker run --rm node` |
| `-e KEY=VAL` | Set environment variable | `docker run -e NODE_ENV=prod myapp` |
| `-v /host:/cont` | Bind mount a volume | `docker run -v $(pwd):/app node` |
| `--network` | Connect to a Docker network | `docker run --network mynet app` |

### System & Cleanup Commands

| Command | What it does | Example |
|---|---|---|
| `docker system df` | Show disk usage by Docker | `docker system df` |
| `docker system prune` | Remove all unused data | `docker system prune` |
| `docker system prune -a` | Remove everything not in use | `docker system prune -a` |
| `docker container prune` | Remove all stopped containers | `docker container prune` |
| `docker image prune -a` | Remove all unused images | `docker image prune -a` |
| `docker volume ls` | List all volumes | `docker volume ls` |
| `docker volume prune` | Remove unused volumes | `docker volume prune` |
| `docker info` | Show Docker system info | `docker info` |
| `docker version` | Show client + server versions | `docker version` |

---

## 7. Common Workflows & Patterns

### Workflow 1 — Debug a running container

```bash
# Get the container ID or name
docker ps

# Jump inside with a shell
docker exec -it <container_name> bash
# or for Alpine-based images:
docker exec -it <container_name> sh

# Look around inside
ls
cat /etc/os-release
env               # see environment variables
exit              # leave the container (it keeps running)
```

### Workflow 2 — Hot-reload with volume mount (dev mode)

Instead of rebuilding the image on every code change, mount your local folder into the container.

```bash
# $(pwd) = your current directory on the host
# /app    = the path inside the container
docker run -it -p 3000:3000 -v $(pwd):/app -w /app node node app.js

# Now edit app.js on your machine → changes reflect immediately in the container
```

### Workflow 3 — Stop & clean up everything

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all stopped containers
docker rm $(docker ps -aq)

# Remove a specific image
docker rmi my-node-app

# Nuclear option — clean EVERYTHING unused
docker system prune -a
```

### Workflow 4 — Check what's happening inside a container

```bash
# View real-time logs
docker logs -f my-app

# See resource usage (CPU, memory, network)
docker stats

# See processes running inside the container
docker top my-app

# Inspect container details (IP, mounts, env vars, etc.)
docker inspect my-app
```

---

## 8. Common Mistakes & Fixes

| ❌ Mistake / Error | ✅ Fix |
|---|---|
| `Got permission denied` | Add user to docker group: `sudo usermod -aG docker $USER` then `newgrp docker` |
| Container exits immediately | The main program has nothing to do. Use `-it` for interactive programs, or make sure your app starts a server. |
| `Port already in use` | Change the host port: `-p 8081:80` instead of `-p 8080:80` |
| Can't access localhost | You forgot to map ports! Always use `-p` when running web apps. |
| `Image not found` | Typo in the image name, or you need to `docker pull` it first. |
| Container keeps restarting | Your app is crashing on start. Check: `docker logs <name>` to see the error. |
| Changes not reflected | You edited code but didn't rebuild the image. Run `docker build` again, or use volume mounts for dev. |

---

## 9. Quick-Fire Cheatsheet

> 🚀 **Full workflow to run a Node app on localhost:3000:**
> `docker pull node:20` → `docker build -t myapp .` → `docker run -d -p 3000:3000 --name myapp myapp` → `curl localhost:3000`

```bash
docker pull <image>               # Download image
docker build -t <name> .          # Build image from Dockerfile
docker run <image>                # Run container
docker run -it <image>            # Run interactively
docker run -d -p 3000:3000 <img>  # Run detached, map port 3000
docker run --name abc <image>     # Run with a name
docker ps                         # List running containers
docker ps -a                      # List ALL containers
docker stop <name>                # Stop container
docker rm <name>                  # Remove container
docker rmi <image>                # Remove image
docker logs -f <name>             # Follow live logs
docker exec -it <name> bash       # Get shell inside container
docker images                     # List local images
docker system prune -a            # Nuke everything unused
```

---

*End of Notes · C01: System & Web Essentials · Docker & Containerisation*
