# DevOps: Zero to Hero — A Developer's Roadmap

> A practical, beginner-friendly guide written for a developer who already knows **React.js, Next.js, and Nest.js** and wants to learn **DevOps** from scratch.
>
> Every lesson includes a real example. Concepts are explained in plain language first, then shown in code. Tools change, but the *principles* here are timeless — this guide focuses on both.

---

## Table of Contents

1. [What DevOps Actually Is](#1-what-devops-actually-is)
2. [The Mental Model: How Code Reaches Users](#2-the-mental-model-how-code-reaches-users)
3. [Linux & The Command Line](#3-linux--the-command-line)
4. [Networking Basics](#4-networking-basics)
5. [Git & Version Control (Beyond the Basics)](#5-git--version-control-beyond-the-basics)
6. [Containers with Docker](#6-containers-with-docker)
7. [Docker Compose: Multi-Container Apps](#7-docker-compose-multi-container-apps)
8. [CI/CD Pipelines](#8-cicd-pipelines)
9. [Cloud Fundamentals](#9-cloud-fundamentals)
10. [Reverse Proxies & Web Servers (Nginx)](#10-reverse-proxies--web-servers-nginx)
11. [Infrastructure as Code (IaC)](#11-infrastructure-as-code-iac)
12. [Container Orchestration with Kubernetes](#12-container-orchestration-with-kubernetes)
13. [Monitoring, Logging & Observability](#13-monitoring-logging--observability)
14. [Security Basics (DevSecOps)](#14-security-basics-devsecops)
15. [Secrets & Configuration Management](#15-secrets--configuration-management)
16. [Putting It All Together: A Full Project](#16-putting-it-all-together-a-full-project)
17. [The 6-Month Study Plan](#17-the-6-month-study-plan)
18. [Future-Proofing Your DevOps Skills](#18-future-proofing-your-devops-skills)

---

## 1. What DevOps Actually Is

**DevOps** is not a tool or a job title — it's a *culture and set of practices* that shortens the gap between writing code (**Dev**) and running it reliably in production (**Ops**).

Before DevOps, developers wrote code and "threw it over the wall" to an operations team who deployed it. This caused the classic "it works on my machine" problem. DevOps says: the people who build software should also be responsible for shipping and running it, supported by automation.

The core goals:

- **Automate everything repetitive** (building, testing, deploying).
- **Ship small changes frequently** instead of big risky releases.
- **Measure everything** so you know when something breaks.
- **Recover fast** when things go wrong.

**Analogy:** Think of a restaurant. The chef (developer) shouldn't have to also build the oven, manage electricity, and deliver food across the city manually every time. DevOps builds the *kitchen systems and delivery pipeline* so the chef can focus on cooking, and food reliably reaches customers hot and on time.

> **Future-proof principle:** Tools come and go (Jenkins → GitHub Actions → whatever's next), but the *goal* — automate the path from code to user — never changes. Always learn the "why" before the "how."

---

## 2. The Mental Model: How Code Reaches Users

Before diving into tools, internalize this flow. Every DevOps tool fits somewhere in this chain:

```
You write code
      │
      ▼
   Git (version control)  ──►  push to GitHub/GitLab
      │
      ▼
   CI Pipeline  ──►  installs deps, runs tests, builds the app
      │
      ▼
   Build artifact  ──►  a Docker image (a packaged version of your app)
      │
      ▼
   Registry  ──►  stored image (Docker Hub, GitHub Container Registry)
      │
      ▼
   CD Pipeline  ──►  deploys the image to a server / cloud / Kubernetes
      │
      ▼
   Server (Linux + Docker)  ──►  runs your container
      │
      ▼
   Reverse Proxy (Nginx)  ──►  routes traffic, handles HTTPS
      │
      ▼
   Users  ◄──  monitored by Prometheus/Grafana, logs collected
```

Keep this diagram in your head. Whenever you learn a new tool, ask: *"Which box is this?"*

---

## 3. Linux & The Command Line

99% of servers run Linux. You **must** be comfortable in a terminal. As a Node developer you already use a terminal, so this is about going deeper.

### Key concepts

- **Everything is a file** (even devices and processes).
- **Permissions** control who can read/write/execute.
- **Processes** are running programs; you can start, stop, and inspect them.
- **The shell** (bash/zsh) is how you talk to the OS.

### Essential commands

```bash
# Navigation
pwd                 # where am I?
ls -lah             # list files (long, all, human-readable sizes)
cd /var/log         # change directory

# Files
cat app.log         # print a file
less app.log        # scroll through a big file (q to quit)
tail -f app.log     # follow a log in real time (CRUCIAL for debugging servers)
grep "ERROR" app.log # search for text

# Permissions
chmod +x deploy.sh  # make a script executable
chown user:group f  # change ownership

# Processes
ps aux | grep node  # find running node processes
kill -9 <PID>       # force-kill a process
htop                # interactive process viewer

# System info
df -h               # disk space
free -h             # memory usage
```

### Example: Investigating why a Node app crashed

Imagine your Nest.js API stopped responding on a server. Here's a real workflow:

```bash
# 1. Is the process even running?
ps aux | grep node
# (no node process found — it crashed)

# 2. Check the logs
tail -n 50 /var/app/logs/error.log
# Shows: "FATAL ERROR: JavaScript heap out of memory"

# 3. Check memory
free -h
# Shows almost no free RAM

# Diagnosis: out-of-memory crash. Fix: add swap or increase server RAM.
```

> **Practice tip:** Install WSL2 (Windows) or use the built-in terminal (Mac/Linux) and live in it for a week. Force yourself to do file operations via the command line instead of a GUI.

---

## 4. Networking Basics

Servers talk over networks. You don't need to be a network engineer, but you must understand the fundamentals.

### Key concepts

| Term | Plain explanation |
|------|-------------------|
| **IP address** | The "home address" of a machine (e.g. `192.168.1.10`). |
| **Port** | A "door" on that machine. Your Nest app might listen on port `3000`. |
| **DNS** | The phone book that turns `myapp.com` into an IP address. |
| **HTTP/HTTPS** | The protocol browsers use. HTTPS = encrypted (port 443), HTTP = plain (port 80). |
| **Firewall** | Rules deciding which ports/IPs are allowed in or out. |
| **localhost / 127.0.0.1** | "This same machine." |

### Example: Understanding `0.0.0.0` vs `localhost`

A super common bug: you containerize your Nest app and it works locally but not in Docker.

```typescript
// ❌ This binds only to localhost — unreachable from outside a container
await app.listen(3000, '127.0.0.1');

// ✅ This binds to all network interfaces — reachable inside Docker/cloud
await app.listen(3000, '0.0.0.0');
```

`127.0.0.1` means "only accept connections from *inside* this machine." Inside a container, that blocks the outside world. `0.0.0.0` means "accept on all interfaces."

### Useful networking commands

```bash
curl -I https://myapp.com    # check if a site responds (headers only)
ping google.com              # test basic connectivity
nslookup myapp.com           # what IP does this domain resolve to?
netstat -tulpn               # what's listening on which ports?
ss -tulpn                    # modern replacement for netstat
```

---

## 5. Git & Version Control (Beyond the Basics)

You already use Git. DevOps requires *fluency*, especially around branching strategies and automation triggers.

### Branching strategy example (Trunk-Based + Feature Branches)

```bash
# Start a feature
git checkout -b feature/user-auth

# ... make changes, commit small and often ...
git add .
git commit -m "feat(auth): add JWT login endpoint"

# Keep up to date with main
git fetch origin
git rebase origin/main

# Push and open a Pull Request
git push origin feature/user-auth
```

### Why this matters for DevOps

CI/CD pipelines are **triggered by Git events**. A push to `main` might trigger a production deploy. A PR might trigger tests. So your Git workflow *is* your deployment workflow.

### Example: A `.gitignore` for a Node DevOps project

```gitignore
node_modules/
dist/
.env
.env.local
*.log
.DS_Store
coverage/
# Never commit secrets or build output
```

> **Future-proof principle:** Never commit secrets (`.env`, API keys, certificates) to Git. This rule will never change. Use secret managers (covered in Section 15).

---

## 6. Containers with Docker

This is the heart of modern DevOps. **Master this section.**

### The problem Docker solves

"It works on my machine" happens because your machine has Node 20, specific env vars, and certain system libraries. The server might differ. **Docker packages your app *with* its entire environment** into a single portable unit called an **image**. Run that image anywhere and it behaves identically.

### Key vocabulary

| Term | Meaning |
|------|---------|
| **Image** | A blueprint — a frozen snapshot of your app + its environment. |
| **Container** | A running instance of an image (like an object instantiated from a class). |
| **Dockerfile** | The recipe that builds an image. |
| **Registry** | Where images are stored/shared (Docker Hub, GHCR). |
| **Volume** | Persistent storage that outlives a container. |

**Analogy:** An image is a *class*, a container is an *object* (instance). You can run many containers from one image.

### Example 1: Dockerizing a Nest.js app (multi-stage build)

A **multi-stage build** keeps the final image small by separating the build environment from the runtime environment.

```dockerfile
# ---- Stage 1: Build ----
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files first (Docker caches this layer if they don't change)
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build

# ---- Stage 2: Production ----
FROM node:20-alpine AS production
WORKDIR /app

# Only copy what we need to RUN the app, not build it
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist

# Run as a non-root user for security
USER node

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

**Why multi-stage?** The `builder` stage has dev dependencies and source code (large). The `production` stage only has compiled output and production deps (small, secure). The final image only includes Stage 2.

### Example 2: Dockerizing a Next.js app

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
# Next.js "standalone" output (set output: 'standalone' in next.config.js)
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
USER node
EXPOSE 3000
CMD ["node", "server.js"]
```

### Essential Docker commands

```bash
# Build an image and tag it
docker build -t my-nest-api:1.0 .

# Run a container (-d = detached, -p = map ports host:container)
docker run -d -p 3000:3000 --name api my-nest-api:1.0

# See running containers
docker ps

# View logs (the -f follows in real time)
docker logs -f api

# Get a shell INSIDE a running container (debugging gold)
docker exec -it api sh

# Stop and remove
docker stop api && docker rm api

# Clean up unused stuff (frees disk space)
docker system prune -a
```

### The `.dockerignore` file (don't skip this!)

```dockerignore
node_modules
dist
.git
.env
*.log
Dockerfile
.dockerignore
```

This prevents copying junk into your image, making builds faster and images smaller.

### Example: Debugging a container that won't start

```bash
docker run -d -p 3000:3000 --name api my-nest-api:1.0
docker ps        # container not there?
docker ps -a     # shows it exited
docker logs api  # "Error: Cannot find module 'dist/main.js'"
# Fix: the build step failed or CMD path is wrong.
```

---

## 7. Docker Compose: Multi-Container Apps

Real apps aren't just one container. A typical app = **API + database + cache + frontend**. **Docker Compose** defines and runs them together with one command.

### Example: A full Nest.js + Postgres + Redis stack

```yaml
# docker-compose.yml
services:
  api:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/myapp
      REDIS_URL: redis://cache:6379
    depends_on:
      - db
      - cache
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data   # persists data
    ports:
      - "5432:5432"

  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

### The magic: service names as hostnames

Notice `DATABASE_URL` uses `db` as the host, not an IP. **Compose creates an internal network where each service is reachable by its name.** Your API connects to `db:5432` and `cache:6379`. No IP addresses needed.

### Commands

```bash
docker compose up -d        # start everything in background
docker compose ps           # status of all services
docker compose logs -f api  # follow logs of one service
docker compose down         # stop and remove everything
docker compose down -v      # ...and delete volumes (wipes the DB!)
docker compose up -d --build # rebuild images and restart
```

> **Real-world use:** Compose is perfect for local development environments and small single-server deployments. For large-scale production, you graduate to Kubernetes (Section 12).

---

## 8. CI/CD Pipelines

- **CI (Continuous Integration):** Every code push automatically builds and tests your code, catching bugs early.
- **CD (Continuous Delivery/Deployment):** After tests pass, automatically deploy to staging or production.

We'll use **GitHub Actions** because it's free, popular, and lives next to your code.

### How it works

A YAML file in `.github/workflows/` defines *jobs* triggered by *events* (push, PR, schedule). Each job runs *steps* on a fresh virtual machine.

### Example: CI pipeline for a Nest.js app

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

### Example: CD pipeline — build Docker image & push to registry

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest

  deploy:
    needs: build-and-push   # only runs if build succeeds
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull ghcr.io/${{ github.repository }}:latest
            docker stop api || true
            docker rm api || true
            docker run -d -p 3000:3000 --name api ghcr.io/${{ github.repository }}:latest
```

### The mental model of a pipeline

```
Push to main
   → Job 1: build & test (fail fast if broken)
   → Job 2: build Docker image, push to registry
   → Job 3: SSH into server, pull new image, restart container
```

> **Future-proof principle:** The *pattern* (trigger → test → build → deploy) is universal across GitHub Actions, GitLab CI, CircleCI, Jenkins, etc. Learn the pattern, and switching tools becomes trivial.

---

## 9. Cloud Fundamentals

The "cloud" is just someone else's computers you rent. The big three are **AWS**, **Google Cloud (GCP)**, and **Azure**. Start with one — **AWS** has the largest market share and most jobs.

### Core service categories (learn these concepts, not just one provider)

| Concept | AWS name | What it does |
|---------|----------|--------------|
| **Virtual server** | EC2 | A rented Linux machine you control. |
| **Object storage** | S3 | Store files/images/backups cheaply. |
| **Managed database** | RDS | Postgres/MySQL that AWS maintains for you. |
| **Container hosting** | ECS / EKS | Run Docker containers / Kubernetes. |
| **Serverless functions** | Lambda | Run code without managing servers. |
| **DNS** | Route 53 | Manage domains. |
| **Networking** | VPC | Your private network in the cloud. |

### Example: Deploying your Dockerized app to a cloud server (EC2)

```bash
# 1. SSH into your rented server
ssh -i my-key.pem ubuntu@<server-ip>

# 2. Install Docker
curl -fsSL https://get.docker.com | sh

# 3. Log in to your registry and pull your image
docker login ghcr.io
docker pull ghcr.io/myuser/my-nest-api:latest

# 4. Run it
docker run -d -p 80:3000 --name api ghcr.io/myuser/my-nest-api:latest

# Your app is now live at http://<server-ip>
```

### Where to start cheaply

Before AWS's complexity, beginners often start with simpler platforms that abstract the hard parts:

- **Railway, Render, Fly.io** — push code or a Dockerfile, they deploy it.
- **DigitalOcean / Hetzner** — simple, cheap virtual servers (great for practicing Linux + Docker).
- **Vercel** — ideal for Next.js (you likely know this already).

> **Recommendation:** Practice on a $5/month DigitalOcean or Hetzner VPS. Real Linux, real Docker, real money pressure to not waste resources — the best teacher.

---

## 10. Reverse Proxies & Web Servers (Nginx)

A **reverse proxy** sits in front of your app and routes incoming traffic. **Nginx** is the most common.

### Why you need one

Your Nest app runs on port 3000. But users expect `https://api.myapp.com` on port 443. Nginx:

1. Listens on ports 80/443 (standard web ports).
2. Forwards requests to your app on port 3000.
3. Handles **HTTPS/SSL certificates**.
4. Can serve multiple apps on one server (routing by domain).
5. Load-balances across multiple app instances.

### Example: Nginx config for a Nest.js API with HTTPS

```nginx
# /etc/nginx/sites-available/api.myapp.com

server {
    listen 80;
    server_name api.myapp.com;
    # Redirect all HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name api.myapp.com;

    ssl_certificate     /etc/letsencrypt/live/api.myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.myapp.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;   # forward to your app
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Free HTTPS with Let's Encrypt (Certbot)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d api.myapp.com
# Certbot auto-edits your Nginx config and auto-renews certificates. Free HTTPS!
```

### Example: Routing frontend + backend on one server

```nginx
server {
    listen 443 ssl;
    server_name myapp.com;

    # API requests go to Nest.js
    location /api/ {
        proxy_pass http://localhost:3000/;
    }

    # Everything else goes to Next.js
    location / {
        proxy_pass http://localhost:3001/;
    }
}
```

---

## 11. Infrastructure as Code (IaC)

Instead of manually clicking around a cloud console to create servers, **IaC lets you define your infrastructure in code files.** This makes it repeatable, version-controlled, and reviewable. **Terraform** is the industry standard.

### Why it matters

Imagine setting up a server manually, then needing an identical one. With IaC you just run the code again. Need to destroy everything? One command. Your infrastructure becomes as manageable as your application code.

### Example: Provisioning a server with Terraform

```hcl
# main.tf
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
    }
  }
}

provider "digitalocean" {
  token = var.do_token
}

# Define a server (called a "droplet" on DigitalOcean)
resource "digitalocean_droplet" "web" {
  image  = "ubuntu-24-04-x64"
  name   = "my-app-server"
  region = "sgp1"          # Singapore — close to Bangladesh
  size   = "s-1vcpu-1gb"   # cheapest tier

  tags = ["production", "api"]
}

# Output the server's IP after creation
output "server_ip" {
  value = digitalocean_droplet.web.ipv4_address
}
```

### Terraform workflow

```bash
terraform init      # download providers
terraform plan      # preview what WILL change (always review this!)
terraform apply     # actually create the infrastructure
terraform destroy   # tear it all down
```

> **The big idea:** Your entire infrastructure lives in Git. Code review for servers. Roll back infrastructure like you roll back code. This concept is here to stay.

---

## 12. Container Orchestration with Kubernetes

When you have *many* containers across *many* servers, you need an orchestrator. **Kubernetes (K8s)** automatically handles scheduling, scaling, self-healing, and networking of containers.

> ⚠️ **Beginner advice:** Kubernetes is advanced. Don't start here. Master Docker + Compose first, deploy real apps on a single server, *then* learn K8s when you actually feel the pain it solves (managing dozens of containers). Many successful companies never need it.

### What Kubernetes gives you

- **Self-healing:** A container crashes? K8s restarts it automatically.
- **Auto-scaling:** Traffic spikes? K8s spins up more copies.
- **Rolling updates:** Deploy new versions with zero downtime.
- **Load balancing:** Distributes traffic across copies.

### Key concepts

| Term | Meaning |
|------|---------|
| **Pod** | The smallest unit — wraps one (or few) containers. |
| **Deployment** | Declares "I want 3 copies of this app running." |
| **Service** | A stable network endpoint for a set of pods. |
| **Ingress** | Routes external traffic to services (like Nginx). |
| **Node** | A worker machine in the cluster. |
| **Cluster** | The whole group of nodes managed together. |

### Example: Deploying a Nest.js app to Kubernetes

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nest-api
spec:
  replicas: 3                 # run 3 copies for reliability
  selector:
    matchLabels:
      app: nest-api
  template:
    metadata:
      labels:
        app: nest-api
    spec:
      containers:
        - name: nest-api
          image: ghcr.io/myuser/my-nest-api:latest
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
---
# service.yaml — exposes the deployment internally
apiVersion: v1
kind: Service
metadata:
  name: nest-api-service
spec:
  selector:
    app: nest-api
  ports:
    - port: 80
      targetPort: 3000
```

```bash
kubectl apply -f deployment.yaml     # create/update resources
kubectl get pods                     # see your running pods
kubectl logs <pod-name>              # view logs
kubectl scale deployment nest-api --replicas=5   # scale up
```

The beauty: you *declare* the desired state ("3 replicas"), and Kubernetes constantly works to make reality match. If a pod dies, it makes a new one without you doing anything.

---

## 13. Monitoring, Logging & Observability

You can't fix what you can't see. **Observability** = understanding your system's internal state from its outputs. The three pillars:

1. **Metrics** — numbers over time (CPU %, requests/sec, error rate).
2. **Logs** — timestamped text records of events.
3. **Traces** — following one request through your whole system.

### The standard open-source stack

- **Prometheus** — collects and stores metrics.
- **Grafana** — beautiful dashboards to visualize metrics.
- **Loki** — collects logs.
- **Alertmanager** — sends alerts (Slack, email) when thresholds breach.

### Example: Exposing metrics from a Nest.js app

```typescript
// Install: npm install prom-client
import { register, Counter, Histogram } from 'prom-client';

// Count total HTTP requests
const httpRequests = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status'],
});

// Measure request duration
const httpDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Request duration in seconds',
  labelNames: ['method', 'route'],
});

// Expose a /metrics endpoint that Prometheus scrapes
@Controller('metrics')
export class MetricsController {
  @Get()
  async getMetrics(@Res() res: Response) {
    res.set('Content-Type', register.contentType);
    res.send(await register.metrics());
  }
}
```

### Example: Prometheus scrape config

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nest-api'
    static_configs:
      - targets: ['api:3000']   # scrape /metrics from your app
```

### Building your own simple monitor (since you mentioned interest!)

A basic health-check monitor in Node:

```typescript
// simple-monitor.ts — pings services and alerts on failure
import axios from 'axios';

const services = [
  { name: 'API', url: 'http://localhost:3000/health' },
  { name: 'Frontend', url: 'http://localhost:3001' },
];

async function checkServices() {
  for (const service of services) {
    try {
      const start = Date.now();
      await axios.get(service.url, { timeout: 5000 });
      const ms = Date.now() - start;
      console.log(`✅ ${service.name} UP (${ms}ms)`);
    } catch (err) {
      console.error(`🔴 ${service.name} DOWN`);
      // Here you'd send a Slack/email/Telegram alert
      await sendAlert(`${service.name} is down!`);
    }
  }
}

async function sendAlert(message: string) {
  // Example: post to a Slack webhook
  await axios.post(process.env.SLACK_WEBHOOK!, { text: message });
}

setInterval(checkServices, 30_000); // check every 30 seconds
```

### Health-check endpoint (every service should have one)

```typescript
@Controller('health')
export class HealthController {
  @Get()
  check() {
    return { status: 'ok', timestamp: new Date().toISOString() };
  }
}
```

> **Golden rule:** Monitor *symptoms users feel* (slow responses, errors) more than raw resources (CPU). A server at 90% CPU but serving fast responses is fine; a server at 20% CPU returning 500 errors is not.

---

## 14. Security Basics (DevSecOps)

Security is everyone's job. You don't need to be a hacker, but follow these baseline practices.

### Essential server hardening

```bash
# 1. Keep the system updated
sudo apt update && sudo apt upgrade -y

# 2. Set up a firewall (allow only what you need)
sudo ufw allow 22    # SSH
sudo ufw allow 80    # HTTP
sudo ufw allow 443   # HTTPS
sudo ufw enable

# 3. Disable password SSH login — use keys only (edit /etc/ssh/sshd_config)
#    PasswordAuthentication no
#    PermitRootLogin no

# 4. Install fail2ban to block brute-force attempts
sudo apt install fail2ban
```

### Container security checklist

- ✅ Run containers as a **non-root user** (`USER node` in Dockerfile).
- ✅ Use **specific image tags** (`node:20.11-alpine`), not `latest`, for reproducibility.
- ✅ Use **minimal base images** (`alpine` over full `node`) — smaller attack surface.
- ✅ **Scan images for vulnerabilities**:

```bash
# Trivy scans an image for known CVEs
trivy image my-nest-api:1.0
```

### Dependency scanning in CI

```yaml
# Add to your GitHub Actions workflow
- name: Audit dependencies
  run: npm audit --audit-level=high
```

> **Future-proof principle:** "Least privilege" — give every user, service, and container only the minimum access it needs. This principle predates DevOps and will outlive every tool.

---

## 15. Secrets & Configuration Management

**Never hardcode secrets.** API keys, DB passwords, and tokens must live outside your code.

### The hierarchy of secret handling (worst to best)

```
❌ Hardcoded in source code (committed to Git) — NEVER
⚠️  .env file (fine for local dev, gitignored)
✅ CI/CD secrets (GitHub Actions Secrets)
✅ Cloud secret managers (AWS Secrets Manager, Vault)
```

### Example: Local development with `.env`

```bash
# .env (this file is in .gitignore!)
DATABASE_URL=postgres://localhost:5432/myapp
JWT_SECRET=super-secret-change-me
REDIS_URL=redis://localhost:6379
```

```typescript
// Nest.js loads these via ConfigModule
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true })],
})
export class AppModule {}

// Access anywhere:
const dbUrl = this.configService.get<string>('DATABASE_URL');
```

### Example: Secrets in GitHub Actions

Store secrets in **Repo → Settings → Secrets and variables → Actions**, then reference them:

```yaml
steps:
  - name: Deploy
    env:
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    run: ./deploy.sh
```

The secret value is **encrypted** and never appears in logs.

### Provide a template for teammates

```bash
# .env.example  (THIS one IS committed — no real values)
DATABASE_URL=
JWT_SECRET=
REDIS_URL=
```

> **Golden rule:** If a secret ever gets committed to Git, consider it *compromised forever* (Git history keeps it). Rotate it immediately.

---

## 16. Putting It All Together: A Full Project

Build this end-to-end project to cement everything. This is your **portfolio piece**.

### The goal

Deploy a Next.js frontend + Nest.js API + Postgres, fully automated, monitored, with HTTPS.

### Step-by-step

```
1. CODE
   └─ Nest.js API with a /health endpoint
   └─ Next.js frontend that calls the API

2. CONTAINERIZE
   └─ Write a multi-stage Dockerfile for each
   └─ Write docker-compose.yml (api + frontend + postgres)
   └─ Test locally: docker compose up

3. VERSION CONTROL
   └─ Push to GitHub
   └─ Add .gitignore and .dockerignore

4. CI PIPELINE (.github/workflows/ci.yml)
   └─ On PR: lint, test, build

5. PROVISION INFRASTRUCTURE
   └─ Terraform script to create a DigitalOcean/Hetzner server
   └─ terraform apply

6. CONFIGURE SERVER
   └─ Install Docker
   └─ Set up Nginx as reverse proxy
   └─ Get HTTPS via Certbot
   └─ Harden with ufw + fail2ban

7. CD PIPELINE (.github/workflows/deploy.yml)
   └─ On push to main: build image → push to GHCR → SSH deploy

8. MONITORING
   └─ Add /metrics to the API
   └─ Run Prometheus + Grafana (via docker compose)
   └─ Build a Grafana dashboard
   └─ Set an alert: notify Slack if API is down

9. DOCUMENT
   └─ Write a great README explaining the architecture
```

### Architecture diagram

```
                    ┌─────────────────────────────────────┐
   Developer        │           Cloud Server (Linux)        │
      │             │                                       │
   git push         │   ┌─────────┐                         │
      │             │   │  Nginx  │ :443 (HTTPS)            │
      ▼             │   └────┬────┘                         │
  GitHub Repo       │        │ routes                       │
      │             │   ┌────┴─────┬──────────┐             │
  GitHub Actions    │   ▼          ▼          ▼             │
   (CI/CD)          │ Next.js   Nest.js    Postgres         │
      │             │ :3001     :3000       :5432           │
      ▼             │              │                        │
  Build image       │         /metrics                      │
      │             │              ▼                        │
   Push to GHCR ────┼──►  Prometheus → Grafana → Slack alert│
      │             │                                       │
   SSH deploy ──────┘                                       │
                    └─────────────────────────────────────┘
                                    ▲
                                    │ HTTPS
                                  Users
```

When you can build this from scratch and explain every arrow, **you are no longer a beginner.**

---

## 17. The 6-Month Study Plan

A realistic, paced roadmap. Spend ~1 hour/day.

### Month 1 — Foundations
- Linux command line (do everything in the terminal).
- Networking basics (IP, ports, DNS, HTTP).
- Git fluency (branching, rebasing, PR workflows).
- **Milestone:** Comfortably navigate and debug a Linux server.

### Month 2 — Docker
- Containerize your Nest.js and Next.js apps.
- Multi-stage builds, `.dockerignore`, volumes.
- Docker Compose for multi-service stacks.
- **Milestone:** Run your full app stack with one `docker compose up`.

### Month 3 — CI/CD + Cloud
- GitHub Actions: build/test pipeline.
- Rent a cheap VPS; deploy your container manually.
- Set up Nginx + HTTPS.
- **Milestone:** Your app is live on the internet with HTTPS.

### Month 4 — Automation & IaC
- Full CD pipeline (auto-deploy on push to main).
- Terraform to provision the server.
- Server hardening + secrets management.
- **Milestone:** Push to main → app auto-deploys, zero manual steps.

### Month 5 — Observability
- Prometheus + Grafana + health checks.
- Build your own monitoring/alerting (your interest area!).
- Centralized logging.
- **Milestone:** A dashboard showing your app's health + a working alert.

### Month 6 — Scaling & Advanced
- Kubernetes basics (local cluster with `kind` or `minikube`).
- Deploy your app to K8s.
- Explore managed cloud services (RDS, ECS).
- **Milestone:** Your app running on Kubernetes with auto-scaling.

---

## 18. Future-Proofing Your DevOps Skills

Tools will change. These principles won't:

1. **Learn concepts, not just tools.** Understand *what* a reverse proxy does, and you can use Nginx, Caddy, Traefik, or whatever's next.

2. **Automation over manual work.** If you do something twice, script it. This mindset is the essence of DevOps.

3. **Everything as code.** Infrastructure, configs, pipelines — all in version control, all reviewable.

4. **Immutable infrastructure.** Don't patch running servers; rebuild and replace them. Containers embody this.

5. **Observability is non-negotiable.** You can't operate what you can't see.

6. **Security by default.** Least privilege, encrypt secrets, scan dependencies — bake it in, don't bolt it on.

7. **Fail fast, recover faster.** Small frequent changes + good monitoring + easy rollbacks = resilient systems.

8. **Read the docs.** Official documentation is your most reliable, current source. Tutorials age; docs are maintained.

### What's emerging (keep an eye on)

- **Platform Engineering** — building internal platforms so developers self-serve infrastructure.
- **GitOps** — Git as the single source of truth for deployments (tools like ArgoCD, Flux).
- **AI-assisted ops** — using AI for incident analysis, log summarization, and automation.
- **WebAssembly (Wasm)** — a potential lighter alternative to containers.
- **eBPF** — deep observability and security at the Linux kernel level.

But remember: don't chase shiny tools. Master the fundamentals in this guide first. A strong foundation makes every new tool easy to pick up.

---

## Quick Reference: Your DevOps Toolbox

| Layer | Beginner Tool | Why |
|-------|---------------|-----|
| OS | Linux (Ubuntu) | Industry standard for servers |
| Version Control | Git + GitHub | Triggers your whole pipeline |
| Containers | Docker | Package once, run anywhere |
| Local orchestration | Docker Compose | Multi-service dev environments |
| CI/CD | GitHub Actions | Free, lives with your code |
| Cloud | DigitalOcean → AWS | Start simple, grow into complexity |
| Reverse Proxy | Nginx + Certbot | Routing + free HTTPS |
| IaC | Terraform | Infrastructure in version control |
| Orchestration | Kubernetes | When you outgrow a single server |
| Monitoring | Prometheus + Grafana | See everything |
| Secrets | .env → GitHub Secrets → Vault | Never hardcode |

---

> **Final word:** DevOps is learned by *doing*, not just reading. After each section, build something. Break it. Fix it. That struggle is where real understanding forms. You already think like a builder as a developer — now you're learning to ship and run what you build. Welcome to DevOps. 🚀

---

*Last conceptual review: 2026. Tools evolve — always cross-check version-specific commands with official documentation.*
