
# Day 32/90 — Docker Image & Container Revision Lab

**Challenge:** DevOps Winter Arc
**Day:** 32
**Environment:** WSL Ubuntu  
**Topics:** Dockerfile, Image Build, Container Run, Port Mapping, Logs, Health Checks, Non-root User, Troubleshooting

---

## 1. Lab Overview

The objective of Day 32 is to package the same integration-project application selected on Day 30 into a Docker image, run it as a container, inspect its behavior, and practice troubleshooting.

This lab focuses on:

- Creating a Dockerfile.
- Creating a `.dockerignore` file.
- Building a Docker image.
- Running a Docker container.
- Mapping host ports to container ports.
- Checking application availability using curl or a browser.
- Inspecting containers and viewing logs.
- Reviewing non-root execution.
- Checking that secrets are not copied into the image.
- Performing a controlled break/fix troubleshooting exercise.

All tasks are performed locally using Docker.

---

## 2. Environment Preparation

### Step 1: Open WSL Ubuntu

Check your Linux environment:

```bash
whoami
pwd
uname -a
```

### Step 2: Check Docker installation

```bash
docker --version
docker compose version
docker info
```

These commands check:

- Docker CLI version.
- Docker Compose availability.
- Docker engine status.

If Docker Desktop is being used on Windows:

1. Open Docker Desktop.
2. Navigate to Settings.
3. Open Resources → WSL Integration.
4. Enable integration for Ubuntu.
5. Apply changes and restart Docker Desktop if required.
6. Run `docker info` again.

### Step 3: Locate the Day 30 application

```bash
cd ~
ls -la
```

Navigate to your existing project:

```bash
cd ~/DevOps-Winter-Arc
ls -la
```

Enter your actual Day 30 application directory:

```bash
cd /path/to/your/day30-app
pwd
ls -la
```

Identify your application type:

- `package.json` indicates a Node.js application.
- `index.html` and static assets may indicate a static website.
- Other manifest files may indicate a different application runtime.

Continue using the same application selected on Day 30.

---

## 3. Task A — Create a Dockerfile

A Dockerfile contains instructions that Docker uses to build an image.

It defines:

- Base image.
- Working directory.
- Application files.
- Dependencies.
- Exposed port.
- Startup command.

### Option A: Dockerfile for a Static Website

If your application is a static website, create:

```bash
nano Dockerfile
```

Add:

```dockerfile
FROM nginxinc/nginx-unprivileged:alpine

WORKDIR /usr/share/nginx/html

COPY . .

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
```

Save using `Ctrl + O`, press `Enter`, and then `Ctrl + X`.

### Option B: Dockerfile for a Node.js Application

If your application is built with Node.js, create:

```bash
nano Dockerfile
```

Add:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Important points:

- Use a Node.js version supported by your application.
- `npm ci` requires a lockfile such as `package-lock.json`.
- The application must listen on `0.0.0.0` inside the container.
- Adjust the port and startup command if your application uses different settings.

### Step 4: Create `.dockerignore`

A `.dockerignore` file prevents unnecessary and sensitive files from entering the Docker build context.

Create it:

```bash
nano .dockerignore
```

Add:

```text
.git
.gitignore
node_modules
.env
.env.*
*.log
npm-debug.log
.DS_Store
coverage
```

Save the file.

Explanation:

| Entry | Purpose |
|---|---|
| `.git` | Excludes Git history |
| `node_modules` | Excludes local dependencies |
| `.env` | Excludes environment variables and possible secrets |
| `.env.*` | Excludes environment-specific files |
| `*.log` | Excludes local log files |
| `coverage` | Excludes test coverage output |

### Step 5: Verify Docker files

```bash
ls -la
cat Dockerfile
cat .dockerignore
```

Both files should exist in your application directory.

---

## 4. Task B — Build and Run the Docker Image

### Step 6: Build the Docker image

Run:

```bash
docker build -t winter-arc-app:day32 .
```

Command explanation:

- `docker build` — builds a Docker image.
- `-t` — assigns an image name and tag.
- `winter-arc-app` — image name.
- `day32` — image tag.
- `.` — current directory is the build context.

Verify the image:

```bash
docker images
```

You should find:

```text
winter-arc-app    day32
```

### Step 7: Check existing containers

```bash
docker ps -a
```

If an old lab container with the same name exists, remove it:

```bash
docker rm -f winter-arc-app
```

### Step 8: Run the container

For the static website:

```bash
docker run -d \
  --name winter-arc-app \
  -p 8080:8080 \
  winter-arc-app:day32
```

For the Node.js application:

```bash
docker run -d \
  --name winter-arc-app \
  -p 8080:3000 \
  winter-arc-app:day32
```

Explanation:

- `docker run` — creates and starts a container.
- `-d` — runs in detached mode.
- `--name` — assigns a container name.
- `-p` — maps host and container ports.
- `winter-arc-app:day32` — image used to create the container.

### Understanding Port Mapping

Example:

```bash
-p 8080:3000
```

This means:

- Host port: `8080`
- Container port: `3000`

Traffic arriving at host port 8080 is forwarded to container port 3000.

### Step 9: Verify the running container

```bash
docker ps
```

Check port mapping:

```bash
docker port winter-arc-app
```

Check the application:

```bash
curl -I http://localhost:8080
```

Check application content:

```bash
curl http://localhost:8080
```

A successful application response will normally return:

```text
HTTP/1.1 200 OK
```

Open the application in your Windows browser:

```text
http://localhost:8080
```

### Step 10: Check Docker logs

```bash
docker logs winter-arc-app
```

Follow logs continuously:

```bash
docker logs -f winter-arc-app
```

Press `Ctrl + C` to stop following logs. This does not stop the container.

---

## 5. Task C — Container Behavior and Basic Hardening

### Step 11: Inspect the container

```bash
docker inspect winter-arc-app
```

Check its status:

```bash
docker inspect -f '{{.State.Status}}' winter-arc-app
```

Expected output:

```text
running
```

Inspect published ports:

```bash
docker inspect -f '{{json .NetworkSettings.Ports}}' winter-arc-app
```

### Step 12: Check the runtime user

```bash
docker exec winter-arc-app id
```

Explanation:

- `uid=0` means the process is running as root.
- A non-zero UID generally indicates a non-root user.

The static website example uses an unprivileged Nginx image.

For Node.js, you can use its built-in non-root user by modifying the Dockerfile:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY --chown=node:node . .

USER node

EXPOSE 3000

CMD ["npm", "start"]
```

This assumes your application works with the permissions of the `node` user.

### Step 13: Review image history

```bash
docker image history winter-arc-app:day32
```

This displays the image's build layers and instructions.

Review:

- Base image.
- Files copied into the image.
- Build commands.
- Unnecessary files and layers.

### Step 14: Check that secrets were not copied

Review `.dockerignore`:

```bash
cat .dockerignore
```

Search the image filesystem for environment files:

```bash
docker run --rm --entrypoint sh winter-arc-app:day32 -c 'find / -name ".env*" 2>/dev/null'
```

Review any results carefully.

Avoid copying credentials, passwords, API keys, or `.env` files into the Docker image.

### Health Check Concept

A Docker health check periodically tests whether an application is functioning as expected.

Only add a `HEALTHCHECK` when it accurately tests application health.

For example, if your application provides a genuine health endpoint, the health check can test that endpoint.

Do not add a health check that merely verifies whether a process exists and claim that it verifies application health.

---

## 6. Task D — Safe Break/Fix Troubleshooting Drill

The purpose of this task is to create one controlled issue, investigate it, identify the root cause, fix it, and verify recovery.

### Step 15: Confirm the application is healthy

```bash
docker ps
curl -I http://localhost:8080
```

Confirm that the application responds before introducing the issue.

### Step 16: Stop and remove the original container

```bash
docker stop winter-arc-app
docker rm winter-arc-app
```

The Docker image remains available.

### Step 17: Create a container with an incorrect host port

For the static website:

```bash
docker run -d \
  --name winter-arc-app \
  -p 8081:8080 \
  winter-arc-app:day32
```

For Node.js:

```bash
docker run -d \
  --name winter-arc-app \
  -p 8081:3000 \
  winter-arc-app:day32
```

The application is now published on host port 8081 instead of 8080.

### Step 18: Observe the problem

Try the original port:

```bash
curl -I http://localhost:8080
```

It should fail because the container is no longer published on host port 8080.

Try the new port:

```bash
curl -I http://localhost:8081
```

This should normally return a successful HTTP response.

### Step 19: Investigate the issue

Check container status:

```bash
docker ps -a
```

Check logs:

```bash
docker logs winter-arc-app
```

Inspect configuration:

```bash
docker inspect winter-arc-app
```

Check port mapping:

```bash
docker port winter-arc-app
```

### Root Cause

The container was running correctly, but the host port was incorrectly mapped.

The application was available through port 8081 instead of the expected port 8080.

### Step 20: Fix the issue

Stop and remove the incorrectly configured container:

```bash
docker stop winter-arc-app
docker rm winter-arc-app
```

Recreate it with the correct mapping.

For the static website:

```bash
docker run -d \
  --name winter-arc-app \
  -p 8080:8080 \
  winter-arc-app:day32
```

For Node.js:

```bash
docker run -d \
  --name winter-arc-app \
  -p 8080:3000 \
  winter-arc-app:day32
```

### Step 21: Verify recovery

```bash
docker ps
docker port winter-arc-app
curl -I http://localhost:8080
```

Open:

```text
http://localhost:8080
```

The application should now respond on the correct port.

### Break/Fix Incident Summary

| Stage | Description |
|---|---|
| Symptom | Application unavailable on expected host port |
| Evidence | Container status and published port mapping |
| Root Cause | Incorrect host port mapping |
| Fix | Recreated container with correct port mapping |
| Verification | Application responded successfully on port 8080 |

---

## 7. Troubleshooting Reference

### Problem 1: Container exits immediately

Check:

```bash
docker ps -a
docker logs winter-arc-app
docker inspect winter-arc-app
```

Review the Dockerfile's `CMD` and `ENTRYPOINT` instructions and check startup errors.

### Problem 2: Cannot reach the application

Check:

```bash
docker ps
docker port winter-arc-app
docker inspect winter-arc-app
curl -I http://localhost:8080
```

Verify:

- Correct port mapping.
- Correct application port.
- Application listens on `0.0.0.0`.
- Container is running.

### Problem 3: Docker image build fails

Check:

```bash
pwd
ls -la
cat Dockerfile
cat .dockerignore
docker build -t winter-arc-app:day32 .
```

Review Dockerfile path, dependency files, network availability, and build context.

### Problem 4: Permission denied

Check:

```bash
docker exec winter-arc-app id
docker inspect winter-arc-app
```

Review file ownership, runtime user, and file permissions.

Avoid running as root by default.

### Problem 5: Docker image is too large

Check:

```bash
docker images
docker image history winter-arc-app:day32
```

Review:

- Base image size.
- Unnecessary copied files.
- `.dockerignore`.
- Build layers.
- Whether a multi-stage build is appropriate.

---

## 8. Useful Docker Commands Reference

| Command | Purpose |
|---|---|
| `docker --version` | Check Docker version |
| `docker info` | Inspect Docker engine |
| `docker images` | List images |
| `docker build -t name:tag .` | Build an image |
| `docker run -d` | Run a container in detached mode |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker stop name` | Stop a container |
| `docker start name` | Start an existing stopped container |
| `docker rm name` | Remove a container |
| `docker rm -f name` | Force-remove a container |
| `docker logs name` | View container logs |
| `docker logs -f name` | Follow container logs |
| `docker inspect name` | Inspect container configuration |
| `docker exec name id` | Check runtime user |
| `docker port name` | Display port mapping |
| `docker image history name:tag` | View image history |
| `docker run --rm` | Automatically remove a container after it exits |

---

## 9. Interview Warm-up — Day 32

### Q1. What is the difference between a Docker image and a container?

**Answer:** A Docker image is a packaged template containing the application and its dependencies. A container is a running instance created from that image.

### Q2. What does `-p 8080:3000` mean?

**Answer:** Host port 8080 forwards traffic to container port 3000.

### Q3. Why should an application listen on `0.0.0.0` inside a container?

**Answer:** It must accept connections through the container network interface. Listening only on loopback can prevent access through the published Docker port.

### Q4. Why do we use `.dockerignore`?

**Answer:** It excludes unnecessary or sensitive files from the Docker build context.

### Q5. What is a Dockerfile?

**Answer:** A Dockerfile is a text file containing instructions used to build a Docker image.

### Q6. What does `FROM` do?

**Answer:** It specifies the base image used to build a new Docker image.

### Q7. What is the purpose of `WORKDIR`?

**Answer:** It sets the working directory for subsequent Dockerfile instructions.

### Q8. What is the difference between `CMD` and `ENTRYPOINT`?

**Answer:** Both define what runs when a container starts. `CMD` provides a default command or arguments, while `ENTRYPOINT` defines the main executable.

### Q9. What does `docker build -t winter-arc-app:day32 .` do?

**Answer:** It builds an image from the Dockerfile in the current directory and assigns it the name `winter-arc-app` and tag `day32`.

### Q10. What is the difference between `docker run` and `docker exec`?

**Answer:** `docker run` creates and starts a container. `docker exec` executes a command inside an already running container.

### Q11. How do you check Docker container logs?

**Answer:**

```bash
docker logs container-name
```

### Q12. How do you check stopped containers?

**Answer:**

```bash
docker ps -a
```

### Q13. Why should containers run as non-root users?

**Answer:** Running as a non-root user reduces the potential impact of a security compromise inside the container.

### Q14. What is a Docker health check?

**Answer:** A health check periodically tests whether an application is functioning as expected and reports its health status.

### Q15. How would you troubleshoot a container that is running but the application is not accessible?

**Answer:** I would check the container status, published port mapping, application binding address, application logs, and container configuration using `docker ps`, `docker port`, `docker logs`, and `docker inspect`.

---

## 10. Key Learnings

By completing Day 32, I practiced:

1. Writing Dockerfiles for an application.
2. Creating `.dockerignore` files.
3. Building and tagging Docker images.
4. Running containers in detached mode.
5. Understanding Docker port mapping.
6. Checking container status and logs.
7. Inspecting container configuration.
8. Reviewing non-root execution.
9. Excluding unnecessary and sensitive files.
10. Troubleshooting container and port issues.
11. Performing a controlled break/fix exercise.

**Day 32 Finish Line:**

Application Packaged → Image Built → Container Running → Ports and Logs Checked → One Issue Fixed.

**Learn → Build → Break → Fix → Explain**
