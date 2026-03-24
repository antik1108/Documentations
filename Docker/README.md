# 🐳 Docker — Complete Practical Notes
> From zero to production | Beginner → Advanced | Every command explained

---

## 📋 Table of Contents

| # | Section | Level |
|---|---------|-------|
| 1 | [What is Docker & Why It Exists](#1-what-is-docker--why-it-exists) | 🟢 Beginner |
| 2 | [Core Concepts — Must Know](#2-core-concepts--must-know) | 🟢 Beginner |
| 3 | [Install Docker on Arch Linux](#3-install-docker-on-arch-linux) | 🟢 Beginner |
| 4 | [Docker Images — Pull, List, Remove](#4-docker-images--pull-list-remove) | 🟢 Beginner |
| 5 | [Docker Containers — Run, Stop, Remove](#5-docker-containers--run-stop-remove) | 🟢 Beginner |
| 6 | [docker run — Every Flag Explained](#6-docker-run--every-flag-explained) | 🟡 Intermediate |
| 7 | [Ports — Accessing Your App on localhost](#7-ports--accessing-your-app-on-localhost) | 🟡 Intermediate |
| 8 | [Volumes — Persistent Data & File Sharing](#8-volumes--persistent-data--file-sharing) | 🟡 Intermediate |
| 9 | [Dockerfile — Build Your Own Image](#9-dockerfile--build-your-own-image) | 🟡 Intermediate |
| 10 | [Dockerfile Keywords — Every One Explained](#10-dockerfile-keywords--every-one-explained) | 🟡 Intermediate |
| 11 | [docker build — Every Flag Explained](#11-docker-build--every-flag-explained) | 🟡 Intermediate |
| 12 | [Docker Networking](#12-docker-networking) | 🔴 Advanced |
| 13 | [Docker Compose](#13-docker-compose) | 🔴 Advanced |
| 14 | [Full Command Reference Tables](#14-full-command-reference-tables) | 📖 Reference |
| 15 | [Troubleshooting — Errors & Fixes](#15-troubleshooting--errors--fixes) | 🛠 Troubleshoot |
| 16 | [Quick-Fire Cheatsheet](#16-quick-fire-cheatsheet) | ⚡ Cheatsheet |

---

## 1. What is Docker & Why It Exists

### The Problem

Your code works on your machine but breaks on someone else's. The reason:

| Problem | Real Example |
|---------|-------------|
| **Different OS** | Code works on Windows, fails on Linux |
| **Missing libraries** | Your machine has `libssl`, theirs doesn't |
| **Wrong runtime version** | You use Node 20, server has Node 14 |
| **Different env variables** | `DB_URL` is set on your machine, not theirs |
| **Painful onboarding** | "Install this, then downgrade that, then pray" 😵 |

### The Solution — Containerisation

Docker packages your app **plus its entire environment** into one portable unit called a **container**.
Ship the container. It runs the same everywhere — your laptop, your teammate's Mac, the Linux server, AWS, anywhere.

```
Without Docker:  Code + "it works on my machine" 🤞
With Docker:     Code + OS layer + runtime + libraries + config = Container 📦
```

### What Docker is NOT

- ❌ Not a Virtual Machine (much lighter — explained below)
- ❌ Not just for deployment (great for local dev too)
- ❌ Not magic — it still uses your host OS kernel

### VM vs Docker — Side by Side

```
Virtual Machine                     Docker Container
┌─────────────────────┐             ┌─────────────────────┐
│  App                │             │  App                │
│  Bins / Libs        │             │  Bins / Libs        │
│  Guest OS (full)    │             ├─────────────────────┤
├─────────────────────┤             │  Docker Daemon      │
│  Hypervisor         │             │  Host OS            │
│  Host OS            │             │  Infrastructure     │
│  Infrastructure     │             └─────────────────────┘
└─────────────────────┘
```

| Feature | Virtual Machine | Docker Container |
|---------|----------------|-----------------|
| OS | Each VM has its own full OS | All containers share the host OS kernel |
| Size | GBs | MBs |
| Startup | Minutes | Seconds (or milliseconds) |
| Resource use | High (CPU + RAM + disk) | Very low |
| Performance | Slower (OS overhead) | Near-native |
| Isolation | Strong (hardware-level) | Process-level |
| Portability | Low | Very high |
| Use case | Run different OS, legacy apps | Microservices, dev, CI/CD, cloud |

---

## 2. Core Concepts — Must Know

Understanding these 4 things explains everything else.

### 2.1 Docker Image

An **image** is a **read-only blueprint/template** for creating containers.
It contains: OS layer + runtime + libraries + your app code + config.

Think of it like a **class** in programming — you define it once, then create many instances from it.

```
Image = Recipe / Blueprint / Class definition
        (read-only, stored on disk, shareable)
```

### 2.2 Docker Container

A **container** is a **running instance of an image**.
It's the actual live process — isolated, has its own filesystem, network, and process space.

```
Container = A running copy of the image
            (has a thin writable layer on top of the image)
```

One image → many containers. Like one class → many objects.

### 2.3 Docker Registry

A **registry** is a storage server for Docker images.

- **Docker Hub** (`hub.docker.com`) — the default public registry
- **GitHub Container Registry** (`ghcr.io`) — GitHub's registry
- **AWS ECR, GCP GCR** — cloud provider registries
- **Self-hosted** — you can run your own

When you run `docker pull node`, Docker fetches the `node` image from Docker Hub.

### 2.4 Docker Daemon

The **daemon** (`dockerd`) is the background service that does all the work:
building images, running containers, managing networks and volumes.
The `docker` CLI just sends commands to the daemon.

```
You type:  docker run nginx
           ↓
CLI        →    Daemon (dockerd)    →    Container starts
```

### Image vs Container — Quick Comparison

| | Docker Image | Docker Container |
|--|-------------|-----------------|
| Analogy | Recipe / Blueprint | The cooked meal / Running app |
| State | Read-only, immutable | Running, has writable layer |
| Created by | `docker build` or `docker pull` | `docker run <image>` |
| Stored | On disk | In memory + thin disk layer |
| Can make multiples? | One image | Many containers from one image |

### Container Lifecycle

```
                 docker run
Image ──────────────────────→ Created
                                  │
                             docker start
                                  │
                                  ↓
                              Running ←──── docker restart
                                  │
                             docker stop
                                  │
                                  ↓
                              Stopped
                                  │
                              docker rm
                                  │
                                  ↓
                              Deleted
```

---

## 3. Install Docker on Arch Linux

### Step 1 — Install packages

```bash
# Install Docker engine + CLI
sudo pacman -S docker

# Install Docker Compose (for multi-container apps — covered later)
sudo pacman -S docker-compose
```

### Step 2 — Start the Docker daemon

The daemon must be running before any `docker` command works.

```bash
# Start Docker right now
sudo systemctl start docker

# Make it auto-start on every boot
sudo systemctl enable docker

# Start AND enable in one command
sudo systemctl enable --now docker

# Check the status
sudo systemctl status docker
```

**What to look for in `status` output:**
```
● docker.service - Docker Application Container Engine
     Active: active (running)   ← this is what you want
```

If it shows `failed` → run `journalctl -xe | grep docker` to see the error.

### Step 3 — Add yourself to the docker group

Without this, every `docker` command needs `sudo`. Fix it once:

```bash
# Add your user to the docker group
sudo usermod -aG docker $USER

# Apply the group in the current terminal (no re-login needed)
newgrp docker

# Verify you're in the group
groups
# → should include: docker
```

> ⚠️ For the group change to stick permanently, **log out and log back in**.

### Step 4 — Verify everything works

```bash
# Check the installed version
docker --version
# Docker version 25.x.x, build xxxxxxx

# Check daemon is reachable
docker info

# Run the official test container
docker run hello-world
```

**Expected output from hello-world:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

> ✅ If you see that message — Docker is fully installed and working.

### Useful daemon management commands

```bash
sudo systemctl start docker      # start
sudo systemctl stop docker       # stop
sudo systemctl restart docker    # restart
sudo systemctl status docker     # check status
sudo systemctl enable docker     # auto-start on boot
sudo systemctl disable docker    # remove auto-start
journalctl -u docker -f          # follow daemon logs live
```

---

## 4. Docker Images — Pull, List, Remove

### What is a Docker image tag?

```
node : 20 - alpine
 │     │      │
 │     │      └── variant (alpine = small, slim = medium, no tag = full)
 │     └── version number
 └── image name
```

Common tags:
- `node:20` — Node 20, full Debian base (~350 MB)
- `node:20-alpine` — Node 20, Alpine Linux base (~50 MB) ← use this for production
- `node:20-slim` — Node 20, minimal Debian (~150 MB)
- `node:latest` — latest stable (avoid in production — unpredictable)
- `node:lts` — long-term support version

### Pulling images

```bash
# Pull the latest version of an image
docker pull nginx

# Pull a specific version
docker pull node:20

# Pull a specific variant
docker pull node:20-alpine

# Pull from a specific registry (not Docker Hub)
docker pull ghcr.io/owner/repo:tag
docker pull public.ecr.aws/nginx/nginx:latest

# Pull multiple images
docker pull node:20 && docker pull mongo:7 && docker pull redis:7-alpine
```

### Listing images

```bash
# List all locally stored images
docker images

# Same command, longer form
docker image ls

# Show all images including intermediate layers
docker images -a

# Filter by name
docker images node

# Show only image IDs
docker images -q

# Show images with their full SHA digest
docker images --digests

# Format the output
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

**Output columns explained:**
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
node         20        a1b2c3d4e5f6   2 weeks ago    350MB
│            │         │              │              │
│            │         │              │              └── compressed size on disk
│            │         │              └── when image was built (not pulled)
│            │         └── first 12 chars of the image's SHA256 hash
│            └── version/variant tag
└── image name (repository)
```

### Removing images

```bash
# Remove a specific image
docker rmi node:20

# Remove by image ID
docker rmi a1b2c3d4e5f6

# Remove multiple images at once
docker rmi node:20 nginx mongo

# Force remove (even if a container is using it)
docker rmi -f node:20

# Remove all unused images (not used by any container)
docker image prune

# Remove ALL images not used by a running container
docker image prune -a

# Remove all images (nuclear option)
docker rmi $(docker images -q)
```

### Inspecting images

```bash
# Show full metadata of an image (JSON output)
docker inspect node:20

# See the layers that make up an image + their sizes
docker history node:20

# See history without truncation
docker history --no-trunc node:20

# Show image size
docker images node:20 --format "{{.Size}}"
```

### Saving and loading images (offline transfer)

```bash
# Save an image to a .tar file
docker save node:20 -o node20.tar

# Save multiple images to one file
docker save node:20 nginx -o images.tar

# Load an image from a .tar file
docker load -i node20.tar

# Share an image without a registry
# Machine A:
docker save my-app -o my-app.tar
scp my-app.tar user@server:/tmp/

# Machine B:
docker load -i /tmp/my-app.tar
docker run my-app
```

---

## 5. Docker Containers — Run, Stop, Remove

### The most important thing to understand

> 📌 **A container runs ONE main process. When that process ends, the container stops.**
> Container alive = process running. Process exits = container stops.

This is why `docker run node` exits immediately — the Node REPL starts with no terminal, finds nothing to do, and exits.

### Creating and running containers

```bash
# Basic run (creates + starts container)
docker run nginx

# Give it a name (easier to reference than random IDs)
docker run --name my-nginx nginx

# Run in detached mode (background — your terminal stays free)
docker run -d nginx

# Run interactively (attach terminal)
docker run -it node

# Run, then auto-delete when it stops (great for testing)
docker run --rm node node -e "console.log('hello')"
```

### Listing containers

```bash
# List currently RUNNING containers
docker ps

# List ALL containers (running + stopped)
docker ps -a

# Show only container IDs
docker ps -q

# Show IDs of all containers (including stopped)
docker ps -aq

# Custom format
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
```

**Output columns explained:**
```
CONTAINER ID   IMAGE    COMMAND       CREATED        STATUS        PORTS           NAMES
a1b2c3d4e5f6   nginx    "/docker…"   5 mins ago     Up 5 minutes  80/tcp          my-nginx
│              │        │            │              │              │               │
│              │        │            │              │              │               └── --name value
│              │        │            │              │              └── exposed ports
│              │        │            │              └── Up = running, Exited = stopped
│              │        │            └── when container was created
│              │        └── command the container is running
│              └── image it was created from
└── first 12 chars of container's SHA256 ID
```

### Stopping and starting containers

```bash
# Gracefully stop a container (sends SIGTERM, waits 10s, then SIGKILL)
docker stop my-nginx

# Stop with custom timeout (seconds)
docker stop -t 30 my-nginx

# Force kill immediately (SIGKILL — use only if stop hangs)
docker kill my-nginx

# Start a stopped container (keeps all its settings)
docker start my-nginx

# Start and attach to it
docker start -a my-nginx

# Restart a running container
docker restart my-nginx

# Restart with a delay
docker restart -t 5 my-nginx

# Stop ALL running containers
docker stop $(docker ps -q)
```

### Removing containers

```bash
# Remove a stopped container
docker rm my-nginx

# Force remove a running container (stop + remove)
docker rm -f my-nginx

# Remove multiple containers
docker rm container1 container2 container3

# Remove all stopped containers
docker container prune

# Remove all containers (running + stopped) — CAREFUL
docker rm -f $(docker ps -aq)
```

### Viewing logs

```bash
# Print all logs so far
docker logs my-app

# Follow logs in real time (like tail -f)
docker logs -f my-app

# Show last 50 lines
docker logs --tail 50 my-app

# Show logs with timestamps
docker logs -t my-app

# Combine: follow + last 20 lines + timestamps
docker logs -f --tail 20 -t my-app

# Show logs since a time
docker logs --since 10m my-app     # last 10 minutes
docker logs --since 2024-01-01 my-app
```

### Executing commands inside a running container

```bash
# Open an interactive bash shell (Debian/Ubuntu based images)
docker exec -it my-app bash

# Open sh shell (Alpine based images — no bash)
docker exec -it my-app sh

# Run a single command and exit
docker exec my-app ls -la /app
docker exec my-app cat /etc/os-release
docker exec my-app env

# Run as a specific user
docker exec -it -u root my-app bash

# Run as the container's default user
docker exec -it my-app whoami
```

### Inspecting a container

```bash
# Full JSON metadata (IP, mounts, env vars, health status, etc.)
docker inspect my-app

# Extract specific fields with --format
docker inspect --format '{{.NetworkSettings.IPAddress}}' my-app
docker inspect --format '{{.State.Status}}' my-app
docker inspect --format '{{json .Config.Env}}' my-app

# See running processes inside the container
docker top my-app

# See live CPU, memory, network, disk stats
docker stats

# Stats for a specific container (no refresh — snapshot)
docker stats my-app --no-stream

# Copy files between host and container
docker cp my-app:/app/logs/error.log ./error.log   # container → host
docker cp ./config.json my-app:/app/config.json    # host → container
```

---

## 6. docker run — Every Flag Explained

This is the most complex command. Every flag broken down:

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### Naming and identity

```bash
--name my-app          # give the container a name (instead of random "happy_einstein")
--hostname myserver    # set the container's internal hostname
```

### Detach vs interactive

```bash
-d                     # detached — run in background, print container ID
-i                     # interactive — keep STDIN open
-t                     # tty — allocate a pseudo-terminal
-it                    # almost always used together — interactive terminal

# When to use what:
docker run -d nginx              # web server — runs in background
docker run -it node              # REPL — needs interactive terminal
docker run -it ubuntu bash       # shell session — needs interactive terminal
docker run --rm node -e "..."    # one-off script — no interaction needed
```

### Ports

```bash
-p 8080:80             # map host port 8080 to container port 80
-p 127.0.0.1:8080:80   # bind only on localhost (more secure)
-p 8080:80/udp         # UDP port mapping
-P                     # auto-map ALL EXPOSEd ports to random host ports
```

### Volumes (data persistence)

```bash
-v /host/path:/container/path       # bind mount — link host dir to container dir
-v myvolume:/container/path         # named volume — Docker manages storage
-v /container/path                  # anonymous volume
--mount type=bind,src=/host,dst=/c  # newer syntax, more explicit
--mount type=volume,src=vol,dst=/c  # named volume (newer syntax)

# Read-only mount (container cannot write to it)
-v /host/config:/app/config:ro
```

### Environment variables

```bash
-e NODE_ENV=production             # set a single env variable
-e PORT=3000 -e DB_URL=postgres:// # set multiple
--env-file .env                    # load from a .env file
```

### Resource limits

```bash
--memory 512m          # max RAM (also: 1g, 256m)
--memory-swap 1g       # max RAM + swap combined
--cpus 1.5             # max 1.5 CPU cores
--cpu-shares 512       # relative CPU weight (default 1024)
```

### Restart policies

```bash
--restart no           # never restart (default)
--restart always       # always restart (even on docker daemon restart)
--restart on-failure   # restart only if exit code != 0
--restart on-failure:3 # restart on failure, max 3 times
--restart unless-stopped  # like always, but not if manually stopped
```

### Networking

```bash
--network bridge       # default — containers can communicate
--network host         # share host's network stack (no isolation)
--network none         # no network at all
--network my-network   # connect to a custom network
--add-host db:1.2.3.4  # add entry to /etc/hosts
```

### Other useful flags

```bash
--rm                   # auto-remove container when it exits (great for testing)
-w /app                # set working directory inside container
-u 1000:1000           # run as specific user:group
--privileged           # give container full host capabilities (dangerous)
--read-only            # make container's filesystem read-only
--entrypoint bash      # override the ENTRYPOINT in the Dockerfile
```

### Real examples combining flags

```bash
# Production web server
docker run -d \
  --name my-api \
  --restart unless-stopped \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e DB_URL=postgres://user:pass@db/mydb \
  --memory 512m \
  --cpus 1 \
  my-api:1.0

# Development with live reload
docker run -it \
  --name dev \
  --rm \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  -e NODE_ENV=development \
  node:20-alpine \
  node app.js

# One-off debugging container (deleted after exit)
docker run --rm -it \
  -v $(pwd):/app \
  -w /app \
  node:20-alpine \
  sh
```

---

## 7. Ports — Accessing Your App on localhost

### Why ports don't work by default

Containers are **network-isolated** by default. A port open inside a container is invisible to your host machine unless you explicitly map it.

```
Without -p:
  Your browser → localhost:3000  ✗  (nothing there)
  Container    → port 3000       ✓  (app running, but unreachable)

With -p 3000:3000:
  Your browser → localhost:3000  →  Container port 3000  ✓
```

### Port mapping syntax

```bash
-p HOST_PORT:CONTAINER_PORT

# Examples:
-p 3000:3000     # host 3000 → container 3000
-p 8080:80       # host 8080 → container 80  (access at localhost:8080)
-p 5000:3000     # host 5000 → container 3000 (different ports)
-p 80:80         # host 80   → container 80   (standard HTTP)
```

### Common port mappings to know

```bash
docker run -p 3000:3000 node-app       # Node.js default
docker run -p 8080:80 nginx            # Nginx (map 8080 on host to 80 in container)
docker run -p 5432:5432 postgres       # PostgreSQL
docker run -p 3306:3306 mysql          # MySQL
docker run -p 6379:6379 redis          # Redis
docker run -p 27017:27017 mongo        # MongoDB
docker run -p 8080:8080 jenkins        # Jenkins
```

### Check what ports a container is using

```bash
# See port mappings of a running container
docker port my-app

# See in docker ps output
docker ps
# PORTS column: 0.0.0.0:3000->3000/tcp

# Inspect in detail
docker inspect --format '{{json .NetworkSettings.Ports}}' my-app
```

### Multiple port mappings

```bash
# Expose multiple ports (e.g., HTTP + HTTPS)
docker run -d \
  -p 80:80 \
  -p 443:443 \
  --name my-nginx nginx
```

### Auto-publish all ports

```bash
# Capital -P maps ALL EXPOSEd ports to random available host ports
docker run -d -P nginx

# See which random port was assigned
docker ps
# PORTS: 0.0.0.0:49153->80/tcp  ← random port 49153 assigned
```

---

## 8. Volumes — Persistent Data & File Sharing

### Why volumes exist

Containers are **ephemeral** — when you remove a container, all data inside it is gone.
If your app writes to a database, uploads files, or generates logs — you need volumes.

```
Without volume:  Container removed → Data gone forever  💀
With volume:     Container removed → Data stays on host ✅
```

### 3 types of volumes

```bash
# Type 1: Bind Mount — link a host directory to container
-v /absolute/host/path:/container/path
-v $(pwd):/app              # current directory into /app
-v $(pwd)/data:/app/data    # subfolder

# Type 2: Named Volume — Docker manages the storage location
-v mydata:/container/path
-v postgres-data:/var/lib/postgresql/data

# Type 3: Anonymous Volume — temporary, hard to find later
-v /container/path          # no name = anonymous
```

### Bind mounts — development use

```bash
# Mount your code into a container (live editing without rebuild)
docker run -it \
  -v $(pwd):/app \      # your code → /app in container
  -w /app \             # set working dir to /app
  -p 3000:3000 \
  node:20-alpine \
  node app.js

# Now: edit any file on your machine → container sees it instantly

# Read-only mount (config files, secrets)
docker run -v $(pwd)/config:/app/config:ro my-app
```

### Named volumes — production use

```bash
# Create a named volume
docker volume create mydata

# Use it in a container
docker run -v mydata:/app/data my-app

# Data persists even if you remove the container
docker rm my-app
docker run -v mydata:/app/data my-app   # data is still there

# List all volumes
docker volume ls

# Inspect a volume (see where Docker stores it on disk)
docker volume inspect mydata
# "Mountpoint": "/var/lib/docker/volumes/mydata/_data"

# Remove a volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

### Database persistence example

```bash
# Run PostgreSQL with persistent data
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# The database survives container restarts and removal
docker stop postgres
docker rm postgres
docker run -d --name postgres -v postgres-data:/var/lib/postgresql/data postgres:15
# Your data is still there
```

---

## 9. Dockerfile — Build Your Own Image

### What is a Dockerfile?

A `Dockerfile` is a **plain text recipe** that tells Docker exactly how to build your image.
Docker reads it line by line, top to bottom, at build time.

```
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
```

> 📌 The file **must be named exactly `Dockerfile`** — capital D, no extension.

### How layers and caching work

Every instruction creates a **read-only layer**. Docker caches each layer.
If a layer hasn't changed, Docker reuses the cached version — making subsequent builds **much faster**.

```
Layer 6  ← CMD ["node", "app.js"]      ← changes rarely
Layer 5  ← COPY . .                     ← changes every time you edit code
Layer 4  ← RUN npm install              ← only re-runs when package.json changes
Layer 3  ← COPY package.json ./         ← only re-runs when package.json changes
Layer 2  ← WORKDIR /app
Layer 1  ← FROM node:20-alpine          ← never changes (unless you change the tag)
```

> 💡 **The golden rule:** Put things that change LEAST near the top. Things that change MOST near the bottom.
> This maximises cache hits and makes rebuilds fast.

### Minimal working Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]
```

### Optimised Dockerfile (with layer caching)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json package-lock.json ./   # copy ONLY this first
RUN npm install                           # cache this layer
COPY . .                                  # then copy the rest
EXPOSE 3000
CMD ["node", "app.js"]
```

**Why this is better:**
- `npm install` only re-runs when `package.json` changes
- Editing `app.js` does NOT trigger `npm install` again
- Builds are dramatically faster in development

### Building the image

```bash
# Basic build (uses Dockerfile in current directory)
docker build -t my-app .

# Build with a specific tag/version
docker build -t my-app:1.0 .

# Build from a different directory
docker build -t my-app /path/to/project

# Build using a differently-named Dockerfile
docker build -f Dockerfile.prod -t my-app .

# Build without using the cache
docker build --no-cache -t my-app .

# Build and see verbose output
docker build --progress=plain -t my-app .

# Pass build arguments
docker build --build-arg NODE_ENV=production -t my-app .
```

### `.dockerignore` — exclude files from the build

Create `.dockerignore` in the same folder as your Dockerfile.
Works exactly like `.gitignore` — Docker will NOT copy these files into the image.

```
# .dockerignore
node_modules/        ← ALWAYS exclude — installed fresh by RUN npm install
.git/                ← git history wastes space
.env                 ← NEVER put secrets in an image
*.log
.DS_Store
README.md
docker-compose.yml
coverage/
dist/
.nyc_output/
```

```bash
# See how big the build context is (smaller = faster)
docker build -t my-app . 2>&1 | head -1
# Sending build context to Docker daemon  1.234MB   ← should be small
```

> 🔐 **NEVER put passwords, API keys, or `.env` files in a Docker image.** Use `-e` at runtime.

---

## 10. Dockerfile Keywords — Every One Explained

### Quick reference card

```dockerfile
FROM node:20-alpine     # 1. base image   (always FIRST)
ARG VERSION=1.0         # 2. build-time variable
LABEL maintainer="you"  # 3. metadata
WORKDIR /app            # 4. working directory
COPY package.json .     # 5. copy files from host
ADD archive.tar.gz /app # 6. copy + auto-extract / download URL
RUN npm install         # 7. run command at BUILD time
ENV NODE_ENV=production # 8. env variable (persists at runtime too)
EXPOSE 3000             # 9. document the port
VOLUME ["/app/data"]    # 10. declare persistent data dir
USER appuser            # 11. switch to non-root user
HEALTHCHECK CMD curl …  # 12. health monitoring
ENTRYPOINT ["node"]     # 13. fixed executable
CMD ["app.js"]          # 14. default args (overridable)
```

---

### `FROM` — Set the base image

**Every Dockerfile must start with `FROM`.** It's the starting OS + runtime layer.

```dockerfile
# Syntax
FROM <image>:<tag>
FROM <image>:<tag> AS <stage>   # multi-stage builds (advanced)

# Pick the right base:
FROM ubuntu:22.04          # bare Ubuntu — you install everything yourself
FROM node:20               # Node.js on Debian (~350MB)
FROM node:20-slim          # Node.js on minimal Debian (~150MB)
FROM node:20-alpine        # Node.js on Alpine Linux (~50MB) ← best for production
FROM python:3.11-slim      # Python 3.11 minimal
FROM scratch               # completely empty (for Go/Rust compiled binaries)
```

**Choosing between variants:**

| Variant | Size | Shell | Package mgr | Use for |
|---------|------|-------|-------------|---------|
| `node:20` | ~350MB | bash | apt | Local dev, easy debugging |
| `node:20-slim` | ~150MB | bash | apt | When you need apt but want smaller |
| `node:20-alpine` | ~50MB | sh | apk | Production — smallest, most secure |

```bash
# Check image sizes after pulling
docker images node
```

---

### `ARG` — Build-time variable

Defines a variable that can be passed in at build time. **Does NOT persist into the running container.**

```dockerfile
# With a default value
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}-alpine

# Without a default (must be passed with --build-arg)
ARG APP_ENV
RUN echo "Building for: $APP_ENV"

# ARG after FROM (build-time only, not in container)
ARG BUILD_DATE
LABEL build-date=${BUILD_DATE}
```

```bash
# Pass values at build time
docker build --build-arg NODE_VERSION=18 -t my-app .
docker build --build-arg APP_ENV=staging -t my-app .

# Use defaults (no flag needed)
docker build -t my-app .

# Multiple build args
docker build \
  --build-arg NODE_VERSION=20 \
  --build-arg APP_ENV=production \
  -t my-app .
```

> ⚠️ **Scope rule:** `ARG` before `FROM` → only usable in `FROM` line.
> Declare it again after `FROM` if you need it in later instructions.

---

### `LABEL` — Add metadata

Adds key-value metadata to the image. Used for documentation, automation, and tooling.

```dockerfile
# Single label
LABEL version="1.0"

# Multiple labels (recommended — one instruction = one layer)
LABEL maintainer="your@email.com" \
      version="1.0.0" \
      description="Production Node API" \
      org.opencontainers.image.source="https://github.com/you/repo"
```

```bash
# Read labels
docker inspect my-app | grep -A 20 '"Labels"'

# Filter specific label
docker inspect --format '{{index .Config.Labels "version"}}' my-app
```

---

### `WORKDIR` — Set working directory

Sets the **current directory** for all subsequent `RUN`, `COPY`, `ADD`, `CMD`, and `ENTRYPOINT` instructions.
Creates the directory if it doesn't exist.

```dockerfile
WORKDIR /app

# All paths after this are relative to /app
COPY . .              # copies to /app/
RUN npm install       # runs inside /app/
CMD ["node", "app.js"] # runs node app.js from /app/
```

```bash
# Verify inside container
docker run -it my-app sh
pwd    # → /app
```

> 💡 Never do `RUN cd /app` — that `cd` only lasts for that single `RUN` step.
> `WORKDIR` persists for all subsequent instructions.

---

### `COPY` — Copy files from host to image

Copies files/directories from your machine (build context) into the image.

```dockerfile
# Copy a single file
COPY app.js /app/app.js

# Copy to current WORKDIR (use . for destination)
WORKDIR /app
COPY app.js .                  # copies to /app/app.js

# Copy entire directory
COPY src/ /app/src/

# Copy multiple files at once (destination must end with /)
COPY package.json package-lock.json /app/

# Wildcard
COPY *.json /app/

# The layer caching pattern — copy package files BEFORE code
COPY package.json package-lock.json ./   # Step 1
RUN npm install                           # Step 2 (cached if package.json unchanged)
COPY . .                                  # Step 3 (only this reruns when code changes)
```

**`COPY` vs `ADD`:**

| | `COPY` | `ADD` |
|--|--------|-------|
| Copy local files | ✅ | ✅ |
| Copy from URL | ❌ | ✅ |
| Auto-extract .tar | ❌ | ✅ |
| Predictable caching | ✅ | ❌ |
| **Recommendation** | **Always use this** | Only for tar/URL |

---

### `ADD` — Copy with extra powers

Like `COPY`, but can also download URLs and auto-extract tar archives.

```dockerfile
# Auto-extract a local tar archive
ADD app.tar.gz /app/

# Download from URL (avoid if possible — breaks layer cache)
ADD https://example.com/file.txt /app/file.txt

# For everything else, use COPY instead
```

> ⚠️ Prefer `COPY` for all regular file copying. Use `ADD` only when you specifically need tar extraction.

---

### `RUN` — Execute commands at build time

Runs a shell command inside the container **during the build**.
Used to install packages, run scripts, create directories, set permissions.

```dockerfile
# Shell form (runs through /bin/sh -c)
RUN npm install
RUN apt-get update && apt-get install -y curl

# Exec form (no shell — use when you don't need shell features)
RUN ["npm", "install"]
RUN ["apt-get", "install", "-y", "curl"]
```

**Shell form vs Exec form:**

| | Shell form | Exec form |
|--|------------|-----------|
| Syntax | `RUN cmd arg` | `RUN ["cmd", "arg"]` |
| Runs through | `/bin/sh -c` | Direct execution |
| Variable expansion | ✅ `$VAR` works | ❌ Use exec + sh |
| Signal handling | ❌ Poor | ✅ Good |
| Use when | Simple commands | Production CMD/ENTRYPOINT |

**Layer reduction — chain commands with `&&`:**

```dockerfile
# ❌ BAD — 3 separate layers, cache can go stale
RUN apt-get update
RUN apt-get install -y curl git
RUN rm -rf /var/lib/apt/lists/*

# ✅ GOOD — 1 layer, always consistent
RUN apt-get update \
    && apt-get install -y curl git \
    && rm -rf /var/lib/apt/lists/*

# Alpine package installation
RUN apk add --no-cache curl git bash

# Node.js dependency installation
RUN npm ci --only=production   # cleaner than npm install for production
RUN npm cache clean --force     # clean npm cache after installing
```

---

### `ENV` — Set environment variables

Sets variables that are available **both at build time AND at runtime** inside the container.

```dockerfile
# Set individual variables
ENV NODE_ENV=production
ENV PORT=3000

# Set multiple in one layer (recommended)
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info \
    APP_NAME=myapi
```

```bash
# Override at runtime with -e (does not change the image)
docker run -e NODE_ENV=development my-app
docker run -e PORT=4000 -e LOG_LEVEL=debug my-app

# Load from .env file
docker run --env-file .env my-app

# See all env vars inside a running container
docker exec -it my-app env
docker exec -it my-app printenv NODE_ENV
```

**`ENV` vs `ARG` — comparison:**

| Feature | `ARG` | `ENV` |
|---------|-------|-------|
| Available during build | ✅ | ✅ |
| Available at runtime | ❌ | ✅ |
| Override at runtime | ❌ | ✅ with `-e` |
| Visible in `docker inspect` | ❌ | ✅ |
| Persists in final image | ❌ | ✅ |
| Good for secrets? | ❌ | ❌ (use Docker secrets) |

---

### `EXPOSE` — Document a port

Tells Docker (and humans reading the Dockerfile) which port the app listens on.
**This is purely informational** — it does NOT publish the port or make it accessible.

```dockerfile
EXPOSE 3000          # app listens on port 3000 (TCP by default)
EXPOSE 80
EXPOSE 443
EXPOSE 53/udp        # UDP port
EXPOSE 3000 8080     # multiple ports
```

```bash
# EXPOSE alone does NOT make port accessible
docker run my-app               # port 3000 NOT reachable from host

# You STILL need -p to publish
docker run -p 3000:3000 my-app  # NOW accessible at localhost:3000

# Capital -P publishes ALL EXPOSEd ports to random host ports
docker run -P my-app
docker ps  # see which random ports Docker chose
```

> 📌 Think of `EXPOSE` as documentation. The actual publishing is always `-p` in `docker run`.

---

### `VOLUME` — Declare a persistent data directory

Declares a path inside the container that should store persistent data.
If you don't mount anything there at `docker run`, Docker creates an anonymous volume automatically.

```dockerfile
VOLUME ["/app/uploads"]        # persists uploaded files
VOLUME ["/var/lib/postgresql"] # persists database
VOLUME ["/app/logs"]           # persists log files
```

```bash
# Use a named volume (recommended — easy to manage)
docker run -v mydata:/app/uploads my-app

# Use a host bind mount (for development)
docker run -v $(pwd)/uploads:/app/uploads my-app

# List all volumes
docker volume ls

# Clean up unused volumes
docker volume prune
```

---

### `HEALTHCHECK` — Monitor container health

Defines a command Docker runs periodically to verify the container is working correctly.
Results shown in `docker ps` as: `(healthy)`, `(unhealthy)`, or `(starting)`.

```dockerfile
# Full syntax with all options
HEALTHCHECK --interval=30s \    # check every 30 seconds
            --timeout=5s \      # fail if no response in 5 seconds
            --start-period=15s \ # grace period when container starts
            --retries=3 \       # mark unhealthy after 3 consecutive failures
  CMD curl -f http://localhost:3000/health || exit 1

# Simpler version
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1

# Use wget on Alpine (no curl by default)
HEALTHCHECK CMD wget -qO- http://localhost:3000/health || exit 1

# Check a file exists
HEALTHCHECK CMD test -f /app/ready.txt || exit 1

# Disable healthcheck inherited from base image
HEALTHCHECK NONE
```

```bash
# See health status
docker ps                           # (healthy) or (unhealthy) in STATUS column
docker inspect --format '{{.State.Health.Status}}' my-app
docker inspect --format '{{json .State.Health}}' my-app | python3 -m json.tool
```

**Add a /health route to your Node app:**

```javascript
// Express example
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok', uptime: process.uptime() });
});
```

---

### `USER` — Run as a non-root user

Sets the user that runs `RUN`, `CMD`, and `ENTRYPOINT` instructions.
Containers run as **root by default** — a serious security risk in production.

```dockerfile
# Create a non-root user (Alpine syntax)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Create a non-root user (Debian/Ubuntu syntax)
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

# Give ownership of app directory to the new user
RUN chown -R appuser:appgroup /app

# Switch to non-root user (place this BEFORE CMD/ENTRYPOINT)
USER appuser

CMD ["node", "app.js"]
```

```bash
# Verify
docker exec -it my-app whoami     # → appuser (not root!)
docker exec -it my-app id         # → uid=1000(appuser) gid=1000(appgroup)

# Override user at runtime if needed (for debugging)
docker exec -it -u root my-app sh
```

---

### `ENTRYPOINT` — Fixed executable

Defines the **main executable** of the container. Unlike `CMD`, it is NOT replaced by arguments passed to `docker run`.

```dockerfile
# Exec form (recommended)
ENTRYPOINT ["node"]
ENTRYPOINT ["python", "-u"]
ENTRYPOINT ["nginx", "-g", "daemon off;"]

# Shell form (avoid — makes signal handling difficult)
ENTRYPOINT node app.js
```

```bash
# With ENTRYPOINT ["node"] and CMD ["app.js"]:
docker run my-app              # → node app.js     (CMD used as default arg)
docker run my-app server.js    # → node server.js  (CMD overridden by arg)
docker run my-app --inspect    # → node --inspect  (flags passed to node)

# Only way to override ENTRYPOINT:
docker run --entrypoint bash my-app    # replace node with bash
docker run --entrypoint sh my-app      # replace with sh
```

---

### `CMD` — Default command/arguments

Provides the **default command (or arguments to ENTRYPOINT)** when the container starts.
This IS overridden by anything passed to `docker run`.

```dockerfile
# Exec form (recommended)
CMD ["node", "app.js"]
CMD ["npm", "start"]
CMD ["python", "server.py"]
CMD ["nginx", "-g", "daemon off;"]

# Shell form
CMD node app.js

# As arguments to ENTRYPOINT
ENTRYPOINT ["node"]
CMD ["app.js"]          # default file — can be overridden
```

```bash
# CMD is overridden by docker run arguments
docker run my-app                 # runs: node app.js  (default CMD)
docker run my-app other.js        # runs: node other.js  (CMD overridden)
docker run -it my-app sh          # runs: sh  (CMD overridden entirely)
```

**`CMD` vs `ENTRYPOINT` — the definitive comparison:**

| | `CMD` | `ENTRYPOINT` |
|--|-------|-------------|
| What it sets | Default command (easily replaced) | Fixed main executable |
| Override with `docker run args` | ✅ Yes, completely replaced | ❌ No, args are appended |
| Override with `--entrypoint` | N/A | ✅ Yes |
| Best for | General images, flexible containers | Single-purpose tool containers |
| Combined use | Provides default args to ENTRYPOINT | The program that always runs |

> ⚠️ If you have multiple `CMD` instructions, **only the last one runs**.

---

### Complete production Dockerfile — all keywords used

```dockerfile
# ── Stage: build arguments ───────────────────────────────────────────────────
ARG NODE_VERSION=20

# ── Base image ───────────────────────────────────────────────────────────────
FROM node:${NODE_VERSION}-alpine

# ── Image metadata ───────────────────────────────────────────────────────────
LABEL maintainer="you@email.com" \
      version="1.0.0" \
      description="Production Node.js REST API"

# ── Runtime environment variables ────────────────────────────────────────────
ENV NODE_ENV=production \
    PORT=3000

# ── Working directory ─────────────────────────────────────────────────────────
WORKDIR /app

# ── Install dependencies (cached layer) ──────────────────────────────────────
COPY package.json package-lock.json ./
RUN npm ci --only=production && npm cache clean --force

# ── Copy application source ───────────────────────────────────────────────────
COPY . .

# ── Security: non-root user ───────────────────────────────────────────────────
RUN addgroup -S appgroup && adduser -S appuser -G appgroup \
    && chown -R appuser:appgroup /app
USER appuser

# ── Persistent storage ────────────────────────────────────────────────────────
VOLUME ["/app/uploads"]

# ── Network port ──────────────────────────────────────────────────────────────
EXPOSE 3000

# ── Health check ──────────────────────────────────────────────────────────────
HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

# ── Container startup ─────────────────────────────────────────────────────────
ENTRYPOINT ["node"]
CMD ["app.js"]
```

---

## 11. docker build — Every Flag Explained

```bash
docker build [OPTIONS] PATH | URL | -
```

```bash
# Core flags
-t my-app                   # tag image as "my-app:latest"
-t my-app:1.0               # tag with version
-t my-app:1.0 -t my-app:latest  # multiple tags at once
-f Dockerfile.prod          # use a differently-named Dockerfile
--no-cache                  # ignore all cached layers (full rebuild)
--pull                      # always pull latest version of base image

# Build context
.                           # current directory (most common)
/path/to/project            # specific directory
-                           # read Dockerfile from stdin

# Build args
--build-arg KEY=VALUE       # pass ARG values defined in Dockerfile
--build-arg NODE_ENV=prod

# Output control
--progress=plain            # full output (no fancy progress bars) — great for debugging
--progress=auto             # default smart output
--quiet -q                  # suppress output, only print image ID

# Platform (for cross-platform builds)
--platform linux/amd64      # build for specific architecture
--platform linux/arm64      # build for ARM (e.g., Apple M1 / Raspberry Pi)
```

**Real build commands:**

```bash
# Development build
docker build -t my-app:dev .

# Production build (clean, versioned, with metadata)
docker build \
  --no-cache \
  --pull \
  --build-arg NODE_VERSION=20 \
  -t my-app:1.2.0 \
  -t my-app:latest \
  -f Dockerfile.prod \
  .

# Debug build that shows every layer
docker build --progress=plain --no-cache -t my-app . 2>&1 | less

# Build for a different CPU architecture
docker build --platform linux/amd64 -t my-app .
```

---

## 12. Docker Networking

### Default networks

Docker creates 3 networks automatically:

```bash
docker network ls
# NETWORK ID   NAME      DRIVER    SCOPE
# abc123       bridge    bridge    local   ← default for all containers
# def456       host      host      local   ← shares host network
# ghi789       none      null      local   ← no network
```

**Bridge (default):**
- All containers get a private IP (e.g., `172.17.0.x`)
- Containers can communicate by IP
- Can access internet through NAT
- Host must use `-p` to reach containers

**Host:**
- Container shares the host's network stack
- `localhost` inside container = `localhost` on your machine
- No port mapping needed
- Less isolation

**None:**
- No network access at all
- Maximum isolation

### Custom networks (recommended)

```bash
# Create a custom bridge network
docker network create my-network

# Create with custom settings
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --gateway 172.20.0.1 \
  my-network

# List all networks
docker network ls

# Inspect a network (see connected containers)
docker network inspect my-network

# Remove a network
docker network rm my-network

# Remove all unused networks
docker network prune
```

**Why custom networks are better:**

```bash
# With a custom network, containers can reach each other by NAME
# Start a database
docker run -d --name postgres --network my-network postgres:15

# Start an app — connects to postgres by name (not by IP!)
docker run -d --name my-api --network my-network my-api
# Inside my-api container: connect to "postgres:5432" — it just works
```

### Connecting containers

```bash
# Start containers on the same network
docker run -d --name db --network my-network postgres:15
docker run -d --name api --network my-network my-api

# Connect a running container to a network
docker network connect my-network my-api

# Disconnect from a network
docker network disconnect my-network my-api

# Connect to multiple networks
docker run -d --name proxy --network frontend --network backend nginx
```

---

## 13. Docker Compose

### What is Docker Compose?

Docker Compose lets you define and run **multi-container applications** in a single YAML file.
Instead of running 5 `docker run` commands, you write one `docker-compose.yml` and run `docker compose up`.

### `docker-compose.yml` structure

```yaml
version: "3.9"

services:
  # Each service is one container
  api:
    build: .                    # build from Dockerfile in current dir
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db
      - redis
    volumes:
      - ./uploads:/app/uploads
    restart: unless-stopped

  db:
    image: postgres:15          # use official image (no Dockerfile needed)
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres-data:    # named volume — Docker manages this
```

### Docker Compose commands

```bash
# Start all services (build if needed, detached)
docker compose up -d

# Start and FORCE rebuild images
docker compose up -d --build

# View logs of all services
docker compose logs

# Follow logs
docker compose logs -f

# Follow logs of one service
docker compose logs -f api

# List running services
docker compose ps

# Stop all services (containers remain)
docker compose stop

# Stop and REMOVE containers, networks (volumes kept)
docker compose down

# Stop and remove EVERYTHING including volumes
docker compose down -v

# Rebuild one specific service
docker compose build api

# Restart one service
docker compose restart api

# Run a one-off command in a service
docker compose exec api bash
docker compose exec api node scripts/seed.js
docker compose run --rm api node migrate.js

# Scale a service (run multiple instances)
docker compose up -d --scale api=3
```

---

## 14. Full Command Reference Tables

### Image commands

| Command | What it does |
|---------|-------------|
| `docker pull <img>` | Download image from registry |
| `docker pull <img>:<tag>` | Download specific version |
| `docker images` | List all local images |
| `docker images -a` | List all images including intermediates |
| `docker images -q` | List only image IDs |
| `docker rmi <img>` | Remove an image |
| `docker rmi -f <img>` | Force remove (even if container uses it) |
| `docker image prune` | Remove dangling images |
| `docker image prune -a` | Remove all unused images |
| `docker build -t name .` | Build image from Dockerfile |
| `docker build --no-cache -t name .` | Build ignoring cache |
| `docker build -f File -t name .` | Build using specific Dockerfile |
| `docker push <img>` | Upload image to registry |
| `docker inspect <img>` | Show full image metadata (JSON) |
| `docker history <img>` | Show image layers and sizes |
| `docker save <img> -o file.tar` | Export image to tar file |
| `docker load -i file.tar` | Import image from tar file |
| `docker tag <img> new-name:tag` | Create a tag/alias for an image |

### Container commands

| Command | What it does |
|---------|-------------|
| `docker run <img>` | Create + start a container |
| `docker run -d <img>` | Run in background |
| `docker run -it <img>` | Run with interactive terminal |
| `docker run --name n <img>` | Run with a name |
| `docker run --rm <img>` | Auto-delete when it stops |
| `docker run -p H:C <img>` | Map host:container port |
| `docker run -e K=V <img>` | Set environment variable |
| `docker run -v /h:/c <img>` | Mount volume |
| `docker run --network n <img>` | Connect to network |
| `docker run --restart always <img>` | Auto-restart policy |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker stop <c>` | Gracefully stop container |
| `docker kill <c>` | Force kill container |
| `docker start <c>` | Start stopped container |
| `docker restart <c>` | Restart container |
| `docker rm <c>` | Remove stopped container |
| `docker rm -f <c>` | Force remove (stop + delete) |
| `docker container prune` | Remove all stopped containers |
| `docker logs <c>` | View logs |
| `docker logs -f <c>` | Follow logs live |
| `docker logs --tail 50 <c>` | Last 50 log lines |
| `docker exec -it <c> bash` | Open shell inside container |
| `docker exec <c> <cmd>` | Run command in container |
| `docker inspect <c>` | Full container metadata |
| `docker top <c>` | Processes inside container |
| `docker stats` | Live resource usage |
| `docker stats <c> --no-stream` | Single resource snapshot |
| `docker cp <c>:/path ./path` | Copy file from container |
| `docker cp ./path <c>:/path` | Copy file to container |
| `docker rename old new` | Rename a container |
| `docker pause <c>` | Pause all processes in container |
| `docker unpause <c>` | Resume paused container |
| `docker wait <c>` | Block until container stops |

### Volume commands

| Command | What it does |
|---------|-------------|
| `docker volume create name` | Create a named volume |
| `docker volume ls` | List all volumes |
| `docker volume inspect name` | Show volume details |
| `docker volume rm name` | Remove a volume |
| `docker volume prune` | Remove all unused volumes |

### Network commands

| Command | What it does |
|---------|-------------|
| `docker network create name` | Create a network |
| `docker network ls` | List all networks |
| `docker network inspect name` | Show network details |
| `docker network connect net <c>` | Connect container to network |
| `docker network disconnect net <c>` | Disconnect container |
| `docker network rm name` | Remove a network |
| `docker network prune` | Remove unused networks |

### System commands

| Command | What it does |
|---------|-------------|
| `docker info` | Show Docker system information |
| `docker version` | Show client + server versions |
| `docker system df` | Show disk usage by Docker |
| `docker system prune` | Remove unused data |
| `docker system prune -a` | Remove all unused data (images too) |
| `docker system prune -a --volumes` | Remove everything including volumes |
| `docker login` | Log in to Docker Hub |
| `docker logout` | Log out |
| `docker search nginx` | Search Docker Hub |

---

## 15. Troubleshooting — Errors & Fixes

### Installation & daemon issues

| Error | Cause | Fix |
|-------|-------|-----|
| `Cannot connect to the Docker daemon` | Daemon not running | `sudo systemctl start docker` |
| `permission denied while trying to connect to the Docker daemon` | User not in docker group | `sudo usermod -aG docker $USER` then `newgrp docker` |
| `docker: command not found` | Docker not installed | `sudo pacman -S docker` |
| Daemon starts but crashes | Config issue or system resource limit | Check `journalctl -u docker -n 50` |

### Container issues

| Error | Cause | Fix |
|-------|-------|-----|
| Container exits immediately | Main process has nothing to do | Use `-it` for interactive; ensure app starts a server |
| `Error: No such container` | Wrong name/ID or container removed | Run `docker ps -a` to check all containers |
| Container keeps restarting | App crashes on start | Check `docker logs <name>` for crash reason |
| `Port already in use` | Another process uses that host port | Change host port: `-p 8081:80` or find and kill the process |
| Can't access app on localhost | Port not mapped | Add `-p host:container` to `docker run` |
| Container uses too much memory | No resource limits set | Add `--memory 512m` to `docker run` |
| `OCI runtime exec failed` | Bad command for that image | Alpine has `sh` not `bash` — use `docker exec -it c sh` |

### Image & build issues

| Error | Cause | Fix |
|-------|-------|-----|
| `Unable to find image locally` | Image not pulled, or typo | Run `docker pull <image>` first |
| `COPY failed: file not found` | File doesn't exist in build context | Check path, check `.dockerignore` |
| Build is very slow every time | Code copied before `npm install` | Put `COPY package.json` → `RUN npm install` → `COPY . .` |
| Image is huge (500MB+) | Using full base image + unnecessary files | Switch to `-alpine`, add `.dockerignore` |
| `RUN: command not found` | Package not installed in that image | Install it first with `RUN apk add` or `RUN apt-get install` |
| `FROM node:latest` breaks | `latest` tag changes over time | Pin a version: `FROM node:20-alpine` |
| Layer cache not working | Build args or context changed | Use `--no-cache` to debug; review instruction order |
| Secrets visible in `docker inspect` | Secrets put in `ENV` or build args | Use runtime `-e`, `.env` files, or Docker secrets |

### Network & port issues

| Error | Cause | Fix |
|-------|-------|-----|
| `bind: address already in use` | Host port occupied | `lsof -i :PORT` to find process, or use different host port |
| Containers can't talk to each other | Not on same network | Create custom network; add both with `--network` |
| `Connection refused` inside container | Wrong hostname | Use service/container name, not `localhost` |
| Can't reach internet from container | DNS or iptables issue | Check `docker run --network host` as a test |

### Volume & data issues

| Error | Cause | Fix |
|-------|-------|-----|
| Data gone after `docker rm` | No volume mounted | Add `-v name:/path` or `-v $(pwd)/data:/path` |
| `Permission denied` writing to volume | User mismatch | Add `RUN chown -R appuser /app/data` in Dockerfile or use `docker run -u` |
| Can't see files in mounted volume | Wrong host path | Use absolute path or `$(pwd)/subdir:/container/path` |
| Volume data corrupted | Two containers writing same volume | Use one writer; others read-only with `:ro` |

### Dockerfile-specific mistakes

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| `COPY . .` before `npm install` | Every code change rebuilds dependencies | `COPY package.json` → `npm install` → `COPY . .` |
| `RUN apt-get update` alone | Cached stale update list, install fails | Always chain: `RUN apt-get update && apt-get install -y pkg` |
| No `.dockerignore` | `node_modules` and `.env` copied into image | Create `.dockerignore` with `node_modules/` and `.env` |
| `EXPOSE` and expecting port to open | `EXPOSE` is documentation only | Use `-p` in `docker run` to actually publish |
| Multiple `CMD` instructions | Only last CMD runs — confusing | Remove all but the final one |
| Running as root | Security vulnerability | Add `USER` instruction before `CMD` |
| `FROM node:latest` | Breaks when `latest` changes | Pin: `FROM node:20-alpine` |
| Not cleaning package cache | Bloats image size | `&& rm -rf /var/lib/apt/lists/*` after `apt-get` |

### Debugging commands — your toolkit

```bash
# ── Build debugging ──────────────────────────────────────────────────────────

# See full build output, skip all cache
docker build --no-cache --progress=plain -t my-app . 2>&1 | less

# Stop at a specific layer to debug
# (comment out instructions below that layer in Dockerfile, then run)
docker run -it my-app sh

# ── Running container debugging ──────────────────────────────────────────────

# Open a shell in a running container
docker exec -it my-app bash         # Debian/Ubuntu
docker exec -it my-app sh           # Alpine

# Open a shell in a container that won't start
docker run -it --entrypoint sh my-app

# Check what user is running
docker exec -it my-app whoami
docker exec -it my-app id

# Check environment variables
docker exec -it my-app env
docker exec -it my-app printenv NODE_ENV

# Check files in working directory
docker exec -it my-app ls -la /app

# Check if a port is listening inside the container
docker exec -it my-app netstat -tlnp
docker exec -it my-app ss -tlnp

# ── Logs ─────────────────────────────────────────────────────────────────────

# Follow all logs
docker logs -f my-app

# Last 50 lines with timestamps
docker logs --tail 50 -t my-app

# Logs since last 5 minutes
docker logs --since 5m my-app

# ── Inspect ──────────────────────────────────────────────────────────────────

# Full JSON metadata
docker inspect my-app

# Just the IP address
docker inspect --format '{{.NetworkSettings.IPAddress}}' my-app

# Just the status
docker inspect --format '{{.State.Status}}' my-app

# Just the env vars
docker inspect --format '{{json .Config.Env}}' my-app

# ── System ───────────────────────────────────────────────────────────────────

# Check disk usage
docker system df

# Live resource usage for all containers
docker stats

# Processes inside a container
docker top my-app

# Check all image layers and sizes
docker history --no-trunc my-app
```

---

## 16. Quick-Fire Cheatsheet

### The 5 most used commands (90% of daily use)

```bash
docker build -t my-app .                  # build image
docker run -d -p 3000:3000 my-app         # run it
docker ps                                  # check it's running
docker logs -f my-app                     # see what it's doing
docker exec -it my-app sh                 # get inside it
```

### Full command cheatsheet

```bash
# ── Images ───────────────────────────────────────────────────────────────────
docker pull node:20-alpine                # download image
docker images                             # list local images
docker rmi node:20-alpine                 # remove image
docker image prune -a                     # remove all unused images
docker history my-app                     # see layers + sizes
docker save my-app -o my-app.tar          # export to file
docker load -i my-app.tar                 # import from file

# ── Build ────────────────────────────────────────────────────────────────────
docker build -t my-app .                  # build with default Dockerfile
docker build -t my-app:1.0 .              # build with version tag
docker build -f Dockerfile.prod -t x .   # use specific Dockerfile
docker build --no-cache -t my-app .       # ignore cache (full rebuild)
docker build --build-arg KEY=VAL -t x .  # pass build argument

# ── Run ──────────────────────────────────────────────────────────────────────
docker run my-app                         # basic run
docker run -d my-app                      # detached (background)
docker run -it my-app sh                  # interactive shell
docker run --rm my-app                    # auto-delete after exit
docker run --name web my-app              # with a name
docker run -p 3000:3000 my-app            # publish port
docker run -e NODE_ENV=prod my-app        # set env variable
docker run --env-file .env my-app         # env vars from file
docker run -v $(pwd):/app my-app          # mount current dir
docker run -v data:/app/data my-app       # named volume
docker run --network my-net my-app        # join network
docker run --restart unless-stopped my-app # auto-restart
docker run --memory 512m --cpus 1 my-app  # resource limits

# ── Container management ─────────────────────────────────────────────────────
docker ps                                 # running containers
docker ps -a                              # all containers
docker stop my-app                        # graceful stop
docker kill my-app                        # force stop
docker start my-app                       # start stopped container
docker restart my-app                     # restart
docker rm my-app                          # remove stopped container
docker rm -f my-app                       # force remove
docker container prune                    # remove all stopped

# ── Inspect & debug ──────────────────────────────────────────────────────────
docker logs -f my-app                     # follow logs
docker logs --tail 50 my-app              # last 50 lines
docker exec -it my-app bash               # bash shell
docker exec -it my-app sh                 # sh shell (Alpine)
docker exec my-app env                    # list env vars
docker inspect my-app                     # full metadata
docker top my-app                         # running processes
docker stats                              # live CPU/RAM usage

# ── Volumes ──────────────────────────────────────────────────────────────────
docker volume create mydata               # create named volume
docker volume ls                          # list volumes
docker volume inspect mydata              # volume details
docker volume rm mydata                   # remove volume
docker volume prune                       # remove unused volumes

# ── Networks ─────────────────────────────────────────────────────────────────
docker network create my-net              # create network
docker network ls                         # list networks
docker network inspect my-net             # network details
docker network connect my-net my-app      # connect container
docker network rm my-net                  # remove network

# ── System ───────────────────────────────────────────────────────────────────
docker system df                          # disk usage
docker system prune                       # remove unused data
docker system prune -a                    # remove ALL unused data
docker system prune -a --volumes          # + volumes too
docker version                            # client + server version
docker info                               # system info
```

### Dockerfile cheatsheet

```dockerfile
FROM node:20-alpine                       # base image (always first)
ARG VERSION=1.0                           # build-time variable
LABEL maintainer="you@email.com"          # metadata
WORKDIR /app                              # working directory
COPY package.json package-lock.json ./   # copy files (specific first)
COPY . .                                  # copy rest of code
ADD archive.tar.gz /app/                  # copy + extract tar
RUN npm ci --only=production              # run command at build time
ENV NODE_ENV=production PORT=3000         # env variable (runtime too)
EXPOSE 3000                               # document port (not publish)
VOLUME ["/app/data"]                      # persistent data directory
USER appuser                              # non-root user (security)
HEALTHCHECK --interval=30s CMD curl -f http://localhost:3000/health || exit 1
ENTRYPOINT ["node"]                       # fixed executable
CMD ["app.js"]                            # default args (overridable)
```

### Docker Compose cheatsheet

```bash
docker compose up -d                      # start all services
docker compose up -d --build              # start + rebuild
docker compose down                       # stop + remove containers
docker compose down -v                    # + remove volumes
docker compose ps                         # list services
docker compose logs -f                    # follow all logs
docker compose logs -f api                # follow one service
docker compose exec api bash              # shell in service
docker compose restart api                # restart one service
docker compose build api                  # rebuild one service
```

### Common patterns

```bash
# Stop and remove everything (clean slate)
docker stop $(docker ps -q) && docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Nuclear clean (removes everything Docker-related)
docker system prune -a --volumes

# Get container IP
docker inspect --format '{{.NetworkSettings.IPAddress}}' my-app

# Copy logs out of container
docker cp my-app:/app/logs ./logs

# Run a temporary Ubuntu container (deleted after exit)
docker run --rm -it ubuntu bash

# Run a temporary Alpine container
docker run --rm -it alpine sh
```

---

*End of Notes · C01: System & Web Essentials · Docker & Containerisation*
*Sections: Beginner 🟢 → Intermediate 🟡 → Advanced 🔴 → Reference 📖 → Troubleshoot 🛠 → Cheatsheet ⚡*
