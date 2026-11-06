
# Week 5 – DevOps Integration Project
## End-to-End Application Deployment: Linux → Docker → Docker Compose → Kubernetes → AWS Architecture

---

# 1. Project Overview

## 1.1 Introduction

This project integrates the DevOps concepts and practical tasks completed during Days 30–35 of the DevOps Winter Arc Challenge.

The objective is to take a simple web application and understand how it moves through the complete DevOps lifecycle, starting from application source code and Linux configuration, followed by containerization, multi-container orchestration, Kubernetes deployment, and finally a proposed AWS cloud architecture.

The project demonstrates how an application can be:

- Developed and organized using application source files.
- Hosted and tested in a Linux environment.
- Containerized using Docker.
- Managed with Docker Compose.
- Deployed and operated using Kubernetes with Minikube.
- Designed for a secure and highly available AWS architecture.
- Monitored, troubleshot, recovered, and documented.

This is an integration project that brings together the work performed separately during Days 30–35.

## 1.2 Problem the Application Solves

The application is a lightweight static web application that provides a browser-accessible web interface.

It contains:

- `index.html` – the main application page.
- `health.html` – a simple endpoint/file used to verify whether the web server is serving application content.

The application is intentionally simple so that the focus remains on understanding the DevOps deployment lifecycle rather than developing complex business functionality.

The main problem addressed by this project is understanding how the same application can be packaged, deployed, exposed, monitored, and maintained across different infrastructure environments.

Instead of manually configuring and running the application every time, the project uses:

- Linux for the operating environment.
- Docker for consistent application packaging.
- Docker Compose for multi-service management.
- Kubernetes for deployment orchestration and scaling.
- AWS architecture for a proposed production-style cloud deployment.

## 1.3 Project Objectives

1. Understand Linux application and service management.
2. Create a Dockerfile for the application.
3. Build and run a Docker image.
4. Manage application and supporting services using Docker Compose.
5. Configure persistent database storage using Docker volumes.
6. Deploy the application using Kubernetes.
7. Configure Deployment, Service, ConfigMap, and health probes.
8. Perform scaling, rollout, and rollback operations.
9. Design a secure AWS architecture using ALB, EKS, and RDS PostgreSQL.
10. Understand IAM least privilege, Security Groups, and observability.
11. Troubleshoot issues across the complete deployment process.

---

# 2. Project Architecture and Folder Structure

## 2.1 Complete DevOps Architecture

The application follows this deployment journey:

```text
             APPLICATION SOURCE CODE
                       |
                       v
                 LINUX (WSL)
              Application Testing
              Processes and Ports
                       |
                       v
                    DOCKER
              Dockerfile Creation
              Image Build and Run
                       |
                       v
                DOCKER COMPOSE
              Application Service
              PostgreSQL Service
              Networking and Volumes
                       |
                       v
                  KUBERNETES
                    MINIKUBE
              Deployment and Pods
              ConfigMap and Service
              Health Checks and Scaling
                       |
                       v
                PROPOSED AWS DESIGN
                       |
             +---------+---------+
             |                   |
             v                   v
          HTTPS ALB          CLOUDWATCH
             |
             v
          AMAZON EKS
       Application Workloads
             |
             v
        AMAZON RDS
         PostgreSQL
```

**Important:** The AWS architecture is a design-only proposal. The local application is deployed and tested using Minikube. The AWS architecture has not been deployed.

## 2.2 Project Folder Structure

```text
Week5-Integration-Project/
│
├── README.md
│
├── app/
│   ├── health.html
│   └── index.html
│
├── architecture.png
│
├── aws-design/
│   ├── Architecture.png
│   ├── IAMLeast-PrivilegeDesign.png
│   ├── Network&SecurityGroupMatrix.png
│   ├── ObservabilityPlan.png
│   └── README.md
│
├── docker/
│   ├── Dockerfile
│   └── README.md
│
├── docker-compose/
│   ├── README.md
│   └── compose.yaml
│
├── docs/
│   └── revision-tracker.md
│
├── kubernetes/
│   ├── README.md
│   ├── configmap.yaml
│   ├── deployment.yaml
│   └── service.yaml
│
├── linux/
│   └── notes/
│       ├── README.md
│       ├── incident-report.txt
│       ├── permission-test.txt
│       ├── service-network-report.txt
│       └── system-report.txt
│
└── screenshots/
```

## 2.3 Purpose of Each Directory

| Directory/File | Purpose |
|---|---|
| `app/` | Contains application HTML files |
| `architecture.png` | Overall project architecture |
| `aws-design/` | AWS architecture and security design diagrams |
| `docker/` | Dockerfile and Docker documentation |
| `docker-compose/` | Compose configuration and documentation |
| `docs/` | Project revision and progress notes |
| `kubernetes/` | Kubernetes YAML manifests |
| `linux/notes/` | Linux reports and incident notes |
| `screenshots/` | Actual project execution evidence |
| `README.md` | Main integration project documentation |

---

# 3. Application Source Code

## 3.1 Application Files

The application source code is located inside:

```text
app/
├── index.html
└── health.html
```

The application is a static website served by Nginx.

The browser requests the main page, and Nginx returns the HTML content.

The health file provides a simple URL that Kubernetes can request to verify that the web server is responding.

## 3.2 Main Application – index.html

Location:

```text
app/index.html
```

Example implementation:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>DevOps Winter Arc Application</title>

    <style>
        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: #f4f7fb;
            color: #1e293b;
        }

        header {
            background: #172554;
            color: white;
            padding: 30px;
            text-align: center;
        }

        main {
            max-width: 850px;
            margin: 40px auto;
            padding: 30px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
        }

        .status {
            color: #15803d;
            font-weight: bold;
        }

        footer {
            text-align: center;
            padding: 20px;
            color: #64748b;
        }
    </style>
</head>

<body>

<header>
    <h1>DevOps Winter Arc</h1>
    <p>Week 5 – Integration Project</p>
</header>

<main>
    <h2>Application Successfully Deployed</h2>

    <p>
        This application demonstrates an end-to-end
        DevOps deployment workflow.
    </p>

    <h3>Technology Stack</h3>

    <ul>
        <li>Linux – Application environment</li>
        <li>Docker – Containerization</li>
        <li>Docker Compose – Multi-container management</li>
        <li>Kubernetes – Container orchestration</li>
        <li>AWS – Proposed cloud architecture</li>
    </ul>

    <p class="status">
        Application Status: Running Successfully
    </p>
</main>

<footer>
    DevOps Winter Arc – Days 30 to 36
</footer>

</body>
</html>
```

## 3.3 Health Check File – health.html

Location:

```text
app/health.html
```

Example:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Application Health</title>
</head>

<body>
    <h1>Application Health Check</h1>
    <p>Web server is responding successfully.</p>
    <p>Status: Healthy</p>
</body>
</html>
```

The health check URL will be:

```text
http://localhost:8080/health.html
```

Because this is a static file, an HTTP 200 response proves that Nginx can serve the file. It does not prove that every application dependency, such as a database, is healthy.

## 3.4 Test the Application Source

Open the project directory:

```bash
cd ~/Week5-Integration-Project
```

Check the application files:

```bash
ls -la app/
```

Check the main page:

```bash
cat app/index.html
```

Check the health page:

```bash
cat app/health.html
```

At this stage, the application source code is ready for Linux and Docker configuration.

---

# 4. Linux Environment – Day 31

## 4.1 Objective

Prepare the Linux environment and understand how application processes, ports, services, permissions, and logs are managed.

The project uses WSL Ubuntu as the Linux environment.

## 4.2 Check Linux System Information

Check operating system:

```bash
cat /etc/os-release
```

Check current user:

```bash
whoami
```

Check user permissions:

```bash
id
```

Check hostname:

```bash
hostname
```

Check kernel:

```bash
uname -a
```

Check uptime:

```bash
uptime
```

Check CPU information:

```bash
lscpu
```

Check memory:

```bash
free -h
```

Check disk usage:

```bash
df -h
```

These commands help identify the Linux environment and available system resources.

## 4.3 Create the Project Directory

If the project does not already exist:

```bash
mkdir -p ~/Week5-Integration-Project
```

Navigate into it:

```bash
cd ~/Week5-Integration-Project
```

Check location:

```bash
pwd
```

Create the project directories:

```bash
mkdir -p app
mkdir -p docker
mkdir -p docker-compose
mkdir -p kubernetes
mkdir -p linux/notes
mkdir -p docs
mkdir -p aws-design
mkdir -p screenshots
```

Do not recreate or overwrite existing project files unnecessarily.

## 4.4 Linux File and Permission Management

Create a test file:

```bash
touch linux/notes/permission-test.txt
```

Check permissions:

```bash
ls -l linux/notes/permission-test.txt
```

Set permissions:

```bash
chmod 644 linux/notes/permission-test.txt
```

Check again:

```bash
stat linux/notes/permission-test.txt
```

Permission `644` means:

- Owner: read and write.
- Group: read only.
- Others: read only.

## 4.5 Linux Networking Checks

Check listening ports:

```bash
ss -tulpn
```

Check a specific port:

```bash
ss -tulpn | grep ':8080'
```

Test local HTTP access:

```bash
curl -I http://localhost:8080
```

The port will become active after the application container is started.

## 4.6 Linux Process and Service Management

Check Nginx process if installed:

```bash
pgrep -a nginx
```

Check Nginx service:

```bash
sudo service nginx status
```

Start Nginx if you are using the host installation:

```bash
sudo service nginx start
```

Check system logs:

```bash
sudo journalctl -n 50 --no-pager
```

In WSL, systemd may not be enabled or used for every service. If `systemctl` does not work, use the supported service command or inspect processes directly.

## 4.7 Linux Reports

The Linux notes directory contains:

```text
linux/notes/
├── README.md
├── incident-report.txt
├── permission-test.txt
├── service-network-report.txt
└── system-report.txt
```

These files record system information, file permissions, service status, network checks, and troubleshooting observations.

**Day 31 Learning:** Linux provides the operating environment for applications. Understanding processes, permissions, ports, services, and logs is essential before containerization.

---

# 5. Docker – Day 32

## 5.1 Objective

Package the static web application into a Docker image and run it inside a container.

Docker provides a consistent application environment that can run across different systems.

## 5.2 Dockerfile

Location:

```text
docker/Dockerfile
```

The Dockerfile defines how the application image is built.

For this project, use the Nginx unprivileged image.

```dockerfile
FROM nginxinc/nginx-unprivileged:alpine

WORKDIR /usr/share/nginx/html

COPY app/ .

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
```

### Dockerfile Explanation

| Instruction | Purpose |
|---|---|
| `FROM` | Selects the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies application files into the image |
| `EXPOSE` | Documents the container listening port |
| `CMD` | Defines the container startup command |

The unprivileged Nginx image runs Nginx without requiring root privileges.

The application files are copied into Nginx's web directory.

Nginx listens on port 8080 in this image.

## 5.3 Create .dockerignore

Create a `.dockerignore` file in the project root:

```text
.git
.gitignore
.env
.env.*
*.log
.DS_Store
screenshots/
docs/
```

This prevents unnecessary files and local configuration secrets from being included in the Docker build context.

## 5.4 Build the Docker Image

Navigate to the project root:

```bash
cd ~/Week5-Integration-Project
```

Build the image:

```bash
docker build -f docker/Dockerfile -t winter-arc-app:day32 .
```

Explanation:

- `docker build` builds an image.
- `-f docker/Dockerfile` selects the Dockerfile.
- `-t winter-arc-app:day32` assigns the image name and tag.
- `.` provides the project root as the build context.

## 5.5 Verify the Docker Image

```bash
docker images
```

Expected image:

```text
winter-arc-app   day32
```

Inspect image details:

```bash
docker image inspect winter-arc-app:day32
```

Inspect image layers:

```bash
docker image history winter-arc-app:day32
```

## 5.6 Run the Docker Container

```bash
docker run -d \
  --name winter-arc-app \
  -p 8080:8080 \
  winter-arc-app:day32
```

Explanation:

- `-d` runs the container in the background.
- `--name` assigns a container name.
- `-p 8080:8080` maps host port 8080 to container port 8080.
- The last value specifies the Docker image.

## 5.7 Check Container Status

```bash
docker ps
```

Check all containers:

```bash
docker ps -a
```

Check port mapping:

```bash
docker port winter-arc-app
```

Inspect container configuration:

```bash
docker inspect winter-arc-app
```

## 5.8 Test the Application

Main page:

```bash
curl -i http://localhost:8080
```

Health page:

```bash
curl -i http://localhost:8080/health.html
```

Open in browser:

```text
http://localhost:8080
```

```text
http://localhost:8080/health.html
```

The expected result is an HTTP success response and the corresponding HTML content.

## 5.9 Inspect Container Logs

```bash
docker logs winter-arc-app
```

Show the last 50 lines:

```bash
docker logs --tail=50 winter-arc-app
```

## 5.10 Stop and Remove the Test Container

Stop:

```bash
docker stop winter-arc-app
```

Remove:

```bash
docker rm winter-arc-app
```

The image remains available unless you explicitly remove it.

**Day 32 Learning:** Docker packages the application and its runtime environment into an image. A container is a running instance of that image.

---

# 6. Docker Compose – Day 33

## 6.1 Objective

Use Docker Compose to define and run multiple services together.

This project uses:

1. Application service – Nginx static website.
2. Database service – PostgreSQL.

Important: The current HTML application is static and does not contain database connection code. Therefore, PostgreSQL is a separate supporting service in this Compose lab. Its presence does not mean the website is integrated with the database.

## 6.2 Compose Architecture

```text
              DOCKER COMPOSE
                    |
          +---------+---------+
          |                   |
          v                   v
     APP SERVICE          DB SERVICE
        NGINX              POSTGRES
       Port 8080            Port 5432
          |                   |
          |                   v
          |             Named Volume
          |              db_data
          |
          v
       Browser
    localhost:8080

   Shared Compose Network
```

## 6.3 Create the Compose File

Location:

```text
docker-compose/compose.yaml
```

Use the following configuration:

```yaml
services:

  app:
    build:
      context: ..
      dockerfile: docker/Dockerfile

    image: winter-arc-app:day33

    container_name: winter-arc-app-compose

    ports:
      - "8080:8080"

    depends_on:
      db:
        condition: service_healthy

    networks:
      - app-network

  db:
    image: postgres:16-alpine

    container_name: winter-arc-postgres

    restart: unless-stopped

    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

    volumes:
      - db_data:/var/lib/postgresql/data

    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U "$$POSTGRES_USER" -d "$$POSTGRES_DB"

      interval: 10s
      timeout: 5s
      retries: 5

    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db_data:
```

### Compose File Explanation

**app service**

- Builds the application image using the Dockerfile.
- Exposes port 8080.
- Connects to the shared Compose network.
- Waits for the database health check to pass before Compose starts the app service.

The `depends_on` health condition controls startup ordering. It does not make the static website communicate with PostgreSQL.

**db service**

- Uses PostgreSQL 16 Alpine.
- Creates a database and database user using environment variables.
- Uses a named volume for persistent database storage.
- Uses `pg_isready` to check database readiness.

**network**

Both services share the same Docker Compose network.

Containers can discover one another by service name.

For example, a database-aware application would use `db` as its database hostname, not `localhost`.

**volume**

`db_data` stores PostgreSQL data outside the container's writable layer.

## 6.4 Create the .env File

Create a local `.env` file in the project root:

```env
POSTGRES_DB=winter_arc_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=CHANGE_THIS_TO_A_LOCAL_STRONG_PASSWORD
```

Do not commit this file to Git.

Create or update the root `.gitignore`:

```gitignore
.env
.env.*
!.env.example

node_modules/
*.log
```

For sharing the project, create `.env.example` with placeholder values only:

```env
POSTGRES_DB=winter_arc_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=replace_me_locally
```

## 6.5 Validate Compose Configuration

From the project root:

```bash
docker compose -f docker-compose/compose.yaml config
```

This checks whether the YAML configuration and environment-variable substitutions are valid.

## 6.6 Start the Compose Services

```bash
docker compose -f docker-compose/compose.yaml up -d --build
```

Explanation:

- `up` creates and starts the services.
- `-d` runs them in the background.
- `--build` builds the application image before starting.

## 6.7 Check Service Status

```bash
docker compose -f docker-compose/compose.yaml ps
```

Expected services:

```text
winter-arc-app-compose
winter-arc-postgres
```

Check logs:

```bash
docker compose -f docker-compose/compose.yaml logs --tail=50
```

Check only application logs:

```bash
docker compose -f docker-compose/compose.yaml logs app
```

Check database logs:

```bash
docker compose -f docker-compose/compose.yaml logs db
```

## 6.8 Test the Application

```bash
curl -i http://localhost:8080
```

```bash
curl -i http://localhost:8080/health.html
```

The static website should respond through the browser.

## 6.9 Verify PostgreSQL

Check the database service:

```bash
docker compose -f docker-compose/compose.yaml exec db \
  pg_isready -U "$POSTGRES_USER" -d "$POSTGRES_DB"
```

If the shell variables are not available in your terminal, use the actual local database name and username from `.env`:

```bash
docker compose -f docker-compose/compose.yaml exec db \
  pg_isready -U app_user -d winter_arc_db
```

Open the PostgreSQL shell:

```bash
docker compose -f docker-compose/compose.yaml exec db \
  psql -U app_user -d winter_arc_db
```

Inside PostgreSQL:

```sql
CREATE TABLE IF NOT EXISTS day33_test (
    id SERIAL PRIMARY KEY,
    message VARCHAR(100)
);

INSERT INTO day33_test (message)
VALUES ('Docker Compose persistence test');

SELECT * FROM day33_test;
```

Exit:

```sql
\q
```

## 6.10 Test Database Persistence

Restart the database container:

```bash
docker compose -f docker-compose/compose.yaml restart db
```

Wait for it to become healthy:

```bash
docker compose -f docker-compose/compose.yaml ps
```

Query the data again:

```bash
docker compose -f docker-compose/compose.yaml exec db \
  psql -U app_user -d winter_arc_db \
  -c "SELECT * FROM day33_test;"
```

The previously inserted record should still exist.

This demonstrates named-volume persistence.

## 6.11 Inspect Compose Networking

List networks:

```bash
docker network ls
```

Inspect the Compose network:

```bash
docker network inspect week5-integration-project_app-network
```

The actual network name may differ depending on your Compose project name.

## 6.12 Stop Compose Services Safely

```bash
docker compose -f docker-compose/compose.yaml down
```

This removes the Compose containers and network but preserves the named volume by default.

Do not use:

```bash
docker compose -f docker-compose/compose.yaml down -v
```

unless you intentionally want to delete the PostgreSQL volume and its data.

**Day 33 Learning:** Docker Compose manages multiple containers, shared networking, configuration, health checks, and persistent storage.

---

# 7. Kubernetes – Day 34

## 7.1 Objective

Deploy the same Docker application into a local Kubernetes cluster using Minikube.

Kubernetes manages application Pods and maintains the desired state of the application.

The deployment includes:

- Namespace.
- Deployment.
- Pods.
- Service.
- ConfigMap.
- Readiness probe.
- Liveness probe.
- Scaling and rollback.

## 7.2 Kubernetes Architecture

```text
                USER / BROWSER
                       |
                       v
                 localhost:8080
                       |
                 Port Forwarding
                       |
                       v
                CLUSTERIP SERVICE
                 revision-app
                       |
                       v
                DEPLOYMENT
                 revision-app
                       |
             +---------+---------+
             |         |         |
             v         v         v
           POD 1     POD 2     POD 3
             |         |         |
             +---------+---------+
                       |
                       v
                 NGINX CONTAINER
                    PORT 8080
                       |
                       v
                 index.html
                 health.html
```

## 7.3 Check Minikube and kubectl

Check Minikube:

```bash
minikube version
```

Check kubectl:

```bash
kubectl version --client
```

Check Minikube status:

```bash
minikube status
```

Start Minikube if required:

```bash
minikube start
```

Check current context:

```bash
kubectl config current-context
```

Switch to Minikube if needed:

```bash
kubectl config use-context minikube
```

Check nodes:

```bash
kubectl get nodes -o wide
```

The Minikube node should show `Ready`.

## 7.4 Create a Namespace

A namespace logically separates resources belonging to this project.

```bash
kubectl create namespace day34-revision
```

Verify:

```bash
kubectl get namespaces
```

## 7.5 Build and Load the Application Image

Kubernetes running inside Minikube needs access to the application image.

From the project root:

```bash
docker build -f docker/Dockerfile -t winter-arc-app:day32 .
```

Load the image into Minikube:

```bash
minikube image load winter-arc-app:day32
```

Verify:

```bash
minikube image ls
```

The Deployment must use the exact same image name and tag.

For a local image, use:

```yaml
imagePullPolicy: IfNotPresent
```

This avoids an unnecessary remote image pull when the image is already present locally.

## 7.6 Create Kubernetes ConfigMap

Location:

```text
kubernetes/configmap.yaml
```

Code:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: revision-app-config
  namespace: day34-revision

data:
  APP_ENV: "revision"
```

### ConfigMap Explanation

A ConfigMap stores non-sensitive configuration separately from the container image.

Here it provides:

```text
APP_ENV=revision
```

The environment variable is available inside the Nginx container. The static HTML pages do not automatically read or display this variable.

## 7.7 Create Kubernetes Deployment

Location:

```text
kubernetes/deployment.yaml
```

Code:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: revision-app
  namespace: day34-revision

spec:
  replicas: 2

  selector:
    matchLabels:
      app: revision-app

  template:
    metadata:
      labels:
        app: revision-app

    spec:
      containers:
        - name: revision-app

          image: winter-arc-app:day32

          imagePullPolicy: IfNotPresent

          ports:
            - containerPort: 8080

          envFrom:
            - configMapRef:
                name: revision-app-config

          readinessProbe:
            httpGet:
              path: /health.html
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5

          livenessProbe:
            httpGet:
              path: /health.html
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10
```

### Deployment Explanation

**apiVersion**

Specifies the Kubernetes API version.

**kind**

Defines the resource type as Deployment.

**metadata**

Contains the Deployment name and namespace.

**replicas**

Requests two application Pods.

**selector**

Identifies which Pods are managed by this Deployment.

**template**

Defines the Pod configuration.

**image**

Uses the Docker image built during Day 32.

**imagePullPolicy**

Uses the local image when available.

**containerPort**

Documents the application container listening port.

**envFrom**

Loads environment variables from the ConfigMap.

**readinessProbe**

Checks whether the application is ready to receive traffic.

**livenessProbe**

Checks whether the application is responding and should remain running.

The `/health.html` path exists in the application's static files, so it is suitable for this basic web-server availability check.

## 7.8 Create Kubernetes Service

Location:

```text
kubernetes/service.yaml
```

Code:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: revision-app
  namespace: day34-revision

spec:
  type: ClusterIP

  selector:
    app: revision-app

  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### Service Explanation

**type: ClusterIP**

Exposes the application internally within the Kubernetes cluster.

**selector**

Selects Pods carrying:

```yaml
app: revision-app
```

**port: 80**

The Service port.

**targetPort: 8080**

The container application port.

The Service provides a stable endpoint even when individual Pods are replaced.

## 7.9 Apply Kubernetes YAML Files

Navigate to the project root:

```bash
cd ~/Week5-Integration-Project
```

Apply the ConfigMap:

```bash
kubectl apply -f kubernetes/configmap.yaml
```

Apply the Deployment:

```bash
kubectl apply -f kubernetes/deployment.yaml
```

Apply the Service:

```bash
kubectl apply -f kubernetes/service.yaml
```

Alternatively, apply all manifests:

```bash
kubectl apply -f kubernetes/
```

## 7.10 Check Kubernetes Resources

Check Deployments:

```bash
kubectl -n day34-revision get deployments
```

Check Pods:

```bash
kubectl -n day34-revision get pods
```

Check Services:

```bash
kubectl -n day34-revision get services
```

Check all resources:

```bash
kubectl -n day34-revision get all
```

Check rollout status:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

Check Service endpoints:

```bash
kubectl -n day34-revision get endpoints revision-app
```

The Service should have endpoints corresponding to ready application Pods.

## 7.11 Verify ConfigMap

```bash
kubectl -n day34-revision get configmap
```

Check the environment variable inside a Pod:

```bash
kubectl -n day34-revision exec deployment/revision-app -- printenv APP_ENV
```

Expected:

```text
revision
```

## 7.12 Access the Kubernetes Application

Run:

```bash
kubectl -n day34-revision port-forward service/revision-app 8080:80
```

Keep this terminal open.

Open another terminal:

```bash
curl -i http://localhost:8080
```

Check the health page:

```bash
curl -i http://localhost:8080/health.html
```

Open in a browser:

```text
http://localhost:8080
```

```text
http://localhost:8080/health.html
```

Press Ctrl+C to stop port forwarding.

## 7.13 Scale the Kubernetes Deployment

Check the original replica count:

```bash
kubectl -n day34-revision get deployment revision-app
```

Scale to three replicas:

```bash
kubectl -n day34-revision scale deployment/revision-app --replicas=3
```

Check Pods:

```bash
kubectl -n day34-revision get pods
```

Check Deployment:

```bash
kubectl -n day34-revision get deployment revision-app
```

## 7.14 Rollout History and Rollback

View rollout history:

```bash
kubectl -n day34-revision rollout history deployment/revision-app
```

Create a harmless Pod-template update:

```bash
kubectl -n day34-revision patch deployment revision-app \
  -p '{"spec":{"template":{"metadata":{"annotations":{"revision":"update-1"}}}}}'
```

Check rollout:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

Roll back:

```bash
kubectl -n day34-revision rollout undo deployment/revision-app
```

Check status:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

**Day 34 Learning:** Kubernetes manages application workloads using Deployments, Pods, Services, configuration, probes, scaling, and rollbacks.

---

# 8. AWS Architecture – Day 35

## 8.1 Objective

Translate the local application architecture into a proposed secure, highly available, observable, and cost-aware AWS architecture.

## IMPORTANT: AWS DESIGN ONLY – NOT DEPLOYED

The AWS architecture in this project is a design proposal.

The application has been practised locally using Docker, Compose, and Minikube.

Amazon EKS, ALB, RDS, NAT Gateway, and other AWS resources have not been launched as part of this design exercise.

No AWS deployment is claimed.

## 8.2 Selected Architecture – Option B

The selected architecture is:

**Application Load Balancer + Amazon EKS + Amazon RDS PostgreSQL**

This architecture was selected because the application was already containerized and deployed using local Kubernetes during Days 30–34.

EKS provides a managed Kubernetes control plane for running the same containerized application in AWS.

## 8.3 AWS Architecture Diagram

![AWS Application Architecture](Week5-Integration-Project/aws-design/Architecture.png)

**File:** `Week5-Integration-Project/aws-design/Architecture.png`

## 8.4 Architecture Components

| Component | AWS Service | Purpose |
|---|---|---|
| DNS | Route 53 | Optional custom domain resolution |
| Load Balancer | Application Load Balancer | Receives HTTPS requests |
| Container Orchestration | Amazon EKS | Manages Kubernetes workloads |
| Database | Amazon RDS PostgreSQL | Stores application data |
| Networking | Amazon VPC | Provides isolated network infrastructure |
| Monitoring | CloudWatch | Collects metrics, logs, and alarms |
| Identity | IAM | Controls AWS permissions |
| Outbound access | NAT Gateway | Provides private-resource internet egress when required |

## 8.5 AWS Request Flow

```text
             USERS
               |
               v
          ROUTE 53
           Optional
               |
               v
        HTTPS ALB :443
        PUBLIC SUBNETS
               |
               v
       EKS APP WORKLOADS
       PRIVATE SUBNETS
            :8080
               |
               v
       RDS POSTGRESQL
       PRIVATE SUBNETS
            :5432
```

The ALB terminates TLS and forwards application traffic to the application tier.

The application tier communicates with PostgreSQL only through the approved database port.

The database does not accept direct public traffic.

## 8.6 Network and Security Group Matrix

![Network and Security Group Matrix](Week5-Integration-Project/aws-design/Network&SecurityGroupMatrix.png)

**File:** `Week5-Integration-Project/aws-design/Network&SecurityGroupMatrix.png`

### Proposed Security Group Rules

| Source | Destination | Protocol / Port | Purpose |
|---|---|---|---|
| Internet | ALB SG | TCP 443 | HTTPS access |
| Internet | ALB SG | TCP 80 | Optional HTTPS redirect |
| ALB SG | App SG | TCP 8080 | Application traffic |
| App SG | DB SG | TCP 5432 | PostgreSQL connection |
| Admin | Approved resources | Session Manager | Controlled administration |

The Security Groups should allow only the traffic required by the application.

The database must not have direct public ingress.

For EKS, these rules need to be implemented using the appropriate AWS load balancer, node, and Pod network controls. Security Group references are not automatically equivalent to Kubernetes Service selectors.

## 8.7 IAM Least-Privilege Design

![IAM Least Privilege Design](Week5-Integration-Project/aws-design/IAMLeast-PrivilegeDesign.png)

**File:** `Week5-Integration-Project/aws-design/IAMLeast-PrivilegeDesign.png`

The architecture uses three main identity roles.

| Actor | Example Role | Purpose |
|---|---|---|
| Deployment pipeline | devops-deploy-role | Deploy approved application versions |
| Application workload | app-workload-role | Access required AWS resources |
| Human operator | devops-operator-role | Authorized infrastructure operations |

### Deployment Role

Uses an approved CI/CD identity and OIDC trust relationship.

It should receive only the permissions needed to deploy the application.

### Application Workload Role

Uses EKS workload identity, such as IRSA where configured.

It should access only the required AWS resources, such as a specific S3 bucket or CloudWatch log group.

### Human Operator Role

Provides authorized operational access.

MFA should be used where applicable.

AWS credentials should never be hard-coded into application code, Docker images, or repositories.

## 8.8 Observability Plan

![Observability Plan](Week5-Integration-Project/aws-design/ObservabilityPlan.png)

**File:** `Week5-Integration-Project/aws-design/ObservabilityPlan.png`

CloudWatch is the proposed monitoring service.

Six signals are included:

| Signal | What it monitors |
|---|---|
| ALB 5xx and target health | Availability and server-side errors |
| Application error rate and latency | Application performance |
| CPU and workload utilization | Compute capacity |
| Database health and connections | PostgreSQL performance |
| Deployment failures | Application rollout problems |
| System and resource health | Pod restarts, node status, and memory |

Alerts should have defined thresholds and clear first-response procedures.

Log retention should be configured according to operational and compliance requirements.

Never store secrets or sensitive user information in logs.

## 8.9 Cost-Aware Architecture Decision

Two architecture options were considered.

### Option A – ALB + EC2 Auto Scaling + RDS

Advantages:

- Simpler infrastructure.
- Easier for a small application.
- No EKS control-plane charge.
- Familiar EC2 administration.

Disadvantages:

- Requires operating system maintenance.
- Requires instance management.
- Less container-native orchestration.

### Option B – ALB + EKS + RDS PostgreSQL

Advantages:

- Container-native architecture.
- Kubernetes deployment management.
- Rolling updates and rollbacks.
- Workload scaling.
- Better alignment with Days 30–34.
- Suitable for future application growth.

Disadvantages:

- Higher operational complexity.
- EKS control-plane cost.
- Worker-node compute cost.
- NAT Gateway and networking costs where applicable.
- Additional monitoring and maintenance.

### Why Option B Was Selected

Option B is selected because the application was already developed using Docker and deployed with Kubernetes in Minikube.

EKS is the natural continuation of that deployment model.

It allows the application to use Kubernetes concepts in a managed AWS environment.

However, Option B is not automatically the cheapest option.

For a small static website with limited traffic, EC2 Auto Scaling may be simpler and more economical.

The selection is based on container orchestration learning, scalability, and future growth rather than minimum cost.

### AWS Cost Drivers

| Resource | Possible cost |
|---|---|
| ALB | Hourly and processed-data charges |
| EKS | Cluster control-plane charge |
| EC2 worker nodes | Compute charges |
| NAT Gateway | Hourly and data-processing charges |
| RDS PostgreSQL | Database instance and storage |
| EBS | Persistent storage |
| Data Transfer | Network transfer |
| CloudWatch | Metrics, logs, and alarms |

Free Tier eligibility and service pricing must be checked before any deployment.

Use:

https://calculator.aws/

**Day 35 Learning:** AWS architecture requires balancing security, availability, scalability, observability, operational complexity, and cost.

---

# 9. Complete Application Deployment Flow

This section explains how the same application moves through each layer.

## Step 1 – Application Source

Files:

```text
app/
├── index.html
└── health.html
```

The source code contains the website and its health-check page.

## Step 2 – Linux

The project is organized and inspected in WSL Ubuntu.

Linux commands help verify:

- System information.
- Permissions.
- Processes.
- Listening ports.
- Services.
- Logs.

## Step 3 – Docker

The Dockerfile defines the Nginx runtime and copies application files into the image.

Build:

```bash
docker build -f docker/Dockerfile -t winter-arc-app:day32 .
```

Run:

```bash
docker run -d \
  --name winter-arc-app \
  -p 8080:8080 \
  winter-arc-app:day32
```

Verify:

```bash
docker ps
curl -i http://localhost:8080
```

## Step 4 – Docker Compose

Compose builds and starts the application and PostgreSQL supporting service.

Validate:

```bash
docker compose -f docker-compose/compose.yaml config
```

Start:

```bash
docker compose -f docker-compose/compose.yaml up -d --build
```

Verify:

```bash
docker compose -f docker-compose/compose.yaml ps
```

Check logs:

```bash
docker compose -f docker-compose/compose.yaml logs
```

Verify PostgreSQL persistence using the named volume.

## Step 5 – Kubernetes

Build and load the image into Minikube:

```bash
docker build -f docker/Dockerfile -t winter-arc-app:day32 .
```

```bash
minikube image load winter-arc-app:day32
```

Apply the manifests:

```bash
kubectl apply -f kubernetes/
```

Check resources:

```bash
kubectl -n day34-revision get all
```

Check rollout:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

Access:

```bash
kubectl -n day34-revision port-forward service/revision-app 8080:80
```

Verify:

```bash
curl -i http://localhost:8080
```

## Step 6 – Proposed AWS Design

The local Kubernetes workload is mapped to the proposed AWS architecture:

| Local concept | Proposed AWS equivalent |
|---|---|
| Minikube cluster | Amazon EKS |
| Kubernetes Deployment | EKS-managed Kubernetes Deployment |
| Kubernetes Pods | Application containers running in EKS |
| Kubernetes Service | Kubernetes Service / AWS load-balancer integration as appropriate |
| Local application access | HTTPS Application Load Balancer |
| Local PostgreSQL service | Amazon RDS PostgreSQL |
| Local configuration | Kubernetes ConfigMap and appropriate AWS configuration services |
| Local logs | CloudWatch Logs |
| Local workload identity | IAM workload role |

This is an architectural mapping, not proof that the local cluster was migrated to AWS.

---

# 10. Troubleshooting Guide – Linux to AWS

This section combines the common troubleshooting scenarios encountered throughout Days 30–35.

| Layer | Problem | Possible Cause | Diagnostic Command | Resolution |
|---|---|---|---|---|
| Linux | Permission denied | Incorrect file permissions | `ls -l FILE` | Use appropriate `chmod` or ownership correction |
| Linux | Service not running | Service stopped | `sudo service nginx status` | Start the service |
| Linux | Port already in use | Another process occupies the port | `ss -tulpn` | Identify the process and use an available port |
| Linux | Connection refused | No process listening | `curl -i URL` | Start the service and verify the listening port |
| Linux | Disk full | Insufficient free storage | `df -h` | Identify and safely remove unnecessary files |
| Docker | Image not found | Image not built or wrong tag | `docker images` | Build the correct image |
| Docker | Container exits | Invalid command or application error | `docker logs CONTAINER_NAME` | Correct the startup configuration |
| Docker | Container unreachable | Incorrect port mapping | `docker port CONTAINER_NAME` | Correct host-to-container port mapping |
| Docker | Build failure | Dockerfile or build context error | `docker build -f docker/Dockerfile .` | Verify Dockerfile and copied file paths |
| Docker | Image too large | Unnecessary files in image | `docker image history IMAGE` | Use a suitable base image and `.dockerignore` |
| Compose | YAML configuration error | Invalid indentation or configuration | `docker compose config` | Correct the YAML |
| Compose | App cannot reach database | Wrong hostname or network | `docker compose ps` | Use the Compose service name as the database hostname |
| Compose | Database not ready | Database initialization delay | `docker compose logs db` | Inspect health check and wait for readiness |
| Compose | Data disappeared | Data stored only in container layer or volume removed | `docker volume ls` | Use a named volume and avoid `down -v` |
| Compose | Port conflict | Host port already occupied | `ss -tulpn` | Change host port or stop conflicting service |
| Kubernetes | ImagePullBackOff | Image unavailable or incorrect tag | `kubectl describe pod POD_NAME` | Load image into Minikube and verify image name |
| Kubernetes | CrashLoopBackOff | Container startup failure | `kubectl logs POD_NAME` | Inspect logs and correct application configuration |
| Kubernetes | Pending Pod | Insufficient cluster resources or scheduling issue | `kubectl describe pod POD_NAME` | Inspect events and resource requests |
| Kubernetes | Service unreachable | Incorrect Service selector | `kubectl get pods --show-labels` | Match Service selector with Pod labels |
| Kubernetes | No endpoints | No ready Pods selected | `kubectl get endpoints SERVICE_NAME` | Fix Pod readiness or Service selector |
| Kubernetes | Probe failure | Incorrect health path or port | `kubectl describe pod POD_NAME` | Verify `/health.html` and port 8080 |
| Kubernetes | Rollout stuck | New Pods fail to become ready | `kubectl rollout status deployment/NAME` | Inspect Pod events and logs |
| Kubernetes | ConfigMap variable missing | Incorrect ConfigMap reference | `kubectl get configmap` | Verify name, namespace, and Deployment reference |
| AWS Design | ALB target unhealthy | Wrong health path, port, or network access | Review target-group health design | Verify health endpoint and SG rules |
| AWS Design | App cannot reach database | Incorrect database SG or routing | Review SG and subnet design | Allow only app-to-DB traffic on required port |
| AWS Design | IAM AccessDenied | Missing or overly restricted permission | Review IAM policy and trust relationship | Add only the required scoped permission |
| AWS Design | EKS workload lacks AWS access | Workload identity not configured | Review EKS workload role design | Configure approved IAM workload identity |
| AWS Design | High AWS cost | Unnecessary NAT, ALB, compute, or database resources | Review AWS billing and pricing | Right-size and remove unused approved resources |
| AWS Design | No monitoring visibility | Metrics or logging not configured | Review CloudWatch design | Configure required metrics, logs, and alarms |

## 10.1 General Troubleshooting Method

When the application becomes unavailable, follow this order:

```text
1. Check the user URL
          |
          v
2. Check Linux process and listening port
          |
          v
3. Check Docker image and container
          |
          v
4. Check Compose services and networking
          |
          v
5. Check Kubernetes Deployment and Pods
          |
          v
6. Check Kubernetes Service and endpoints
          |
          v
7. Check application and dependency logs
          |
          v
8. Check AWS network, IAM, and monitoring design
```

Do not randomly restart or delete resources.

First identify the failing layer, inspect the relevant logs and status, correct the cause, and verify recovery.

---

# 11. Incident Management and Recovery

## 11.1 Controlled Kubernetes Failure

A safe failure scenario is scaling the test Deployment to zero replicas.

Check original replicas:

```bash
kubectl -n day34-revision get deployment revision-app
```

Record the original count.

Scale to zero:

```bash
kubectl -n day34-revision scale deployment/revision-app --replicas=0
```

Check Pods and Service:

```bash
kubectl -n day34-revision get pods,svc
```

Check endpoints:

```bash
kubectl -n day34-revision get endpoints revision-app
```

The Service has no ready application Pods to serve traffic.

## 11.2 Recovery

Restore the original replica count.

For example, if the original value was three:

```bash
kubectl -n day34-revision scale deployment/revision-app --replicas=3
```

Check rollout:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

Verify Pods:

```bash
kubectl -n day34-revision get pods
```

Test the application again:

```bash
curl -i http://localhost:8080
```

## 11.3 Incident Report Format

Record:

| Item | Description |
|---|---|
| Incident | Application unavailable |
| Change | Deployment scaled to zero |
| Symptom | No application Pods serving traffic |
| Root cause | Desired replica count set to zero |
| Diagnosis | Deployment status and Service endpoints |
| Recovery | Restore original replica count |
| Verification | Ready Pods and successful HTTP response |
| Prevention | Restrict access to deployment scaling operations and verify desired replicas |

Only report an incident as completed after actually performing the test.

---

# 12. Security Best Practices

1. Do not store AWS credentials in source code.
2. Never commit `.env` files containing secrets.
3. Use `.dockerignore` to exclude sensitive and unnecessary files.
4. Use non-root containers where supported.
5. Keep application and database workloads in private network boundaries.
6. Expose only the required application ports.
7. Allow database connections only from approved application sources.
8. Use IAM roles instead of hard-coded AWS access keys.
9. Use MFA for human AWS access where applicable.
10. Avoid exposing SSH port 22 broadly.
11. Do not log passwords, access tokens, or sensitive information.
12. Do not delete persistent database volumes during troubleshooting.

---

# 13. Final Project Verification

Before considering the integration project complete, verify the following.

## Application

- [ ] `app/index.html` exists.
- [ ] `app/health.html` exists.
- [ ] Main page returns the expected response.
- [ ] Health page returns HTTP 200.

## Linux

- [ ] Linux system information checked.
- [ ] File permissions understood.
- [ ] Processes and ports inspected.
- [ ] Relevant service and logs inspected.

## Docker

- [ ] Dockerfile created.
- [ ] Docker image built.
- [ ] Container starts successfully.
- [ ] Port mapping verified.
- [ ] Application accessible.

## Docker Compose

- [ ] Compose YAML validated.
- [ ] Application and database services started.
- [ ] Service health checked.
- [ ] Compose networking understood.
- [ ] PostgreSQL persistence verified.
- [ ] Database volume preserved.

## Kubernetes

- [ ] Minikube running.
- [ ] Docker image loaded into Minikube.
- [ ] Namespace created.
- [ ] ConfigMap applied.
- [ ] Deployment applied.
- [ ] Service applied.
- [ ] Pods Ready.
- [ ] Service endpoints verified.
- [ ] Application accessible.
- [ ] Scaling tested.
- [ ] Rollout and rollback understood.

## AWS Design

- [ ] Architecture diagram included.
- [ ] Security Group matrix included.
- [ ] IAM diagram included.
- [ ] Observability diagram included.
- [ ] Option B justified.
- [ ] Cost drivers explained.
- [ ] AWS deployment clearly marked as design-only.

## Integration and Recovery

- [ ] Healthy baseline recorded.
- [ ] Linux, Docker, Compose, and Kubernetes validation performed.
- [ ] One controlled failure reproduced.
- [ ] Root cause identified.
- [ ] Service recovered.
- [ ] Final application response verified.
- [ ] README updated.
- [ ] Demo prepared.

---

# 14. Interview Concepts and Questions

## A. Linux Concepts

### 1. What is Linux?

Linux is an open-source operating system widely used for servers, cloud infrastructure, and DevOps environments.

### 2. What is a process?

A process is a running instance of a program.

### 3. What is a port?

A port is a logical communication endpoint used by network applications.

### 4. What does `ss -tulpn` do?

It displays listening TCP/UDP sockets and associated process information where permissions allow.

### 5. What is the difference between `chmod` and `chown`?

`chmod` changes file permissions, while `chown` changes file ownership.

### 6. How do you troubleshoot a Linux service?

Check service status, running processes, listening ports, and logs.

---

## B. Docker Concepts

### 7. What is Docker?

Docker is a platform used to package and run applications in containers.

### 8. What is a Docker image?

A Docker image is a packaged, read-only template used to create containers.

### 9. What is a container?

A container is a running instance of an image.

### 10. What is a Dockerfile?

A Dockerfile contains instructions used to build a Docker image.

### 11. What is the difference between CMD and ENTRYPOINT?

`CMD` provides a default command or arguments. `ENTRYPOINT` defines the main executable that runs when the container starts.

### 12. What is EXPOSE?

`EXPOSE` documents the port the containerized application is expected to use. It does not publish the port to the host.

### 13. What is port mapping?

Port mapping connects a host port to a container port.

Example:

```bash
docker run -p 8080:8080 IMAGE
```

### 14. Why use a non-root container?

It reduces the impact of a container compromise by avoiding unnecessary root privileges.

---

## C. Docker Compose Concepts

### 15. What is Docker Compose?

Docker Compose defines and manages multi-container applications using a YAML file.

### 16. What is a Compose service?

A service defines how a container should be built or run.

### 17. How do Compose containers communicate?

Containers on the same Compose network can communicate using their service names.

### 18. What is a named volume?

A named volume stores persistent data outside the lifecycle of a container.

### 19. What is the difference between `docker compose stop` and `docker compose down`?

`stop` stops containers but leaves the containers and network configuration in place. `down` removes the Compose containers and network. Named volumes are preserved by default.

### 20. What does `docker compose down -v` do?

It also removes associated named volumes, which can delete persistent database data.

### 21. What is a health check in Compose?

A health check periodically verifies whether a container's service is responding according to a configured test.

---

## D. Kubernetes Concepts

### 22. What is Kubernetes?

Kubernetes is a container orchestration platform used to deploy, manage, scale, and recover containerized applications.

### 23. What is Minikube?

Minikube provides a local Kubernetes environment for learning and testing.

### 24. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes and can contain one or more containers.

### 25. What is a Deployment?

A Deployment manages ReplicaSets and maintains the desired number of Pods.

### 26. What is a Service?

A Service provides a stable network endpoint for accessing selected Pods.

### 27. What is ClusterIP?

ClusterIP exposes a Service internally within the Kubernetes cluster.

### 28. What is a ConfigMap?

A ConfigMap stores non-sensitive configuration separately from the container image.

### 29. What is a readiness probe?

A readiness probe checks whether a container is ready to receive traffic.

### 30. What is a liveness probe?

A liveness probe checks whether a container is healthy and should continue running.

### 31. What is scaling?

Scaling changes the number of application replicas.

### 32. What is rollout?

A rollout is the process of updating a Deployment and replacing old Pods with new ones.

### 33. How do you roll back a Deployment?

```bash
kubectl rollout undo deployment/DEPLOYMENT_NAME
```

### 34. What is ImagePullBackOff?

It means Kubernetes cannot obtain the requested container image.

### 35. What is CrashLoopBackOff?

It means a container repeatedly starts and exits, and Kubernetes is delaying subsequent restart attempts.

### 36. What is the difference between a Pod and a Deployment?

A Pod runs the application container. A Deployment manages the desired number of Pods and supports updates and rollbacks.

---

## E. AWS Concepts

### 37. What is Amazon VPC?

A VPC is a logically isolated virtual network for AWS resources.

### 38. What is the difference between public and private subnets?

Public subnets have a route to an Internet Gateway. Private subnets do not have direct internet routing through an Internet Gateway.

### 39. What is an Internet Gateway?

An Internet Gateway provides internet connectivity for resources in public subnets when routing and security rules allow it.

### 40. What is a NAT Gateway?

A NAT Gateway allows private resources to initiate outbound internet connections without permitting unsolicited inbound internet connections.

### 41. What is an Application Load Balancer?

An ALB distributes incoming application traffic across healthy targets and supports HTTP/HTTPS traffic.

### 42. What is Amazon EKS?

Amazon Elastic Kubernetes Service is a managed Kubernetes service provided by AWS.

### 43. What is Amazon RDS?

Amazon Relational Database Service is a managed relational database service.

### 44. What is Multi-AZ?

Multi-AZ is an availability configuration that distributes supported resources across Availability Zones to improve resilience.

### 45. What is IAM?

Identity and Access Management controls who can access AWS resources and what actions they can perform.

### 46. What is least privilege?

Least privilege means granting only the permissions needed to perform a particular task.

### 47. What is IRSA?

IAM Roles for Service Accounts allows EKS workloads to obtain AWS permissions through associated Kubernetes service accounts and IAM roles.

### 48. What is CloudWatch?

CloudWatch is an AWS monitoring service for metrics, logs, dashboards, and alarms.

### 49. What is the purpose of ALB health checks?

They identify healthy targets so the ALB can avoid routing normal traffic to unhealthy targets.

### 50. Why use HTTPS?

HTTPS encrypts communication between clients and the application endpoint using TLS.

### 51. Why should a database be private?

It reduces direct exposure and restricts database access to approved application sources.

### 52. What are the major costs of Option B?

ALB, EKS control plane, worker-node compute, NAT Gateway, RDS, storage, data transfer, and CloudWatch.

### 53. Is EKS always cheaper than EC2 Auto Scaling?

No. EKS offers Kubernetes orchestration but adds control-plane and operational costs. EC2 Auto Scaling can be simpler and cheaper for small applications.

---

## F. End-to-End DevOps Concepts

### 54. Explain the complete deployment process.

The application begins as source code in Linux. Docker packages it into an image. Docker Compose manages the application and supporting services. Kubernetes deploys and manages the application Pods. The AWS design maps the application to ALB, EKS, and RDS.

### 55. Why do we use containerization?

Containerization packages the application and its runtime environment consistently, reducing environment differences between systems.

### 56. What is the difference between Docker Compose and Kubernetes?

Compose defines and runs multi-container applications. Kubernetes provides orchestration, desired-state management, scaling, service discovery, and recovery.

### 57. How do you troubleshoot an unavailable application?

Trace the request path, inspect processes and ports, check containers and Pods, verify Service endpoints, inspect dependencies, and examine logs and events.

### 58. What is an incident report?

An incident report records the impact, symptoms, root cause, recovery action, verification, and prevention measures.

### 59. Why perform break/fix exercises?

They help DevOps engineers practise diagnosing failures, restoring service, and verifying recovery in a controlled environment.

### 60. What is the purpose of observability?

Observability provides information about system behavior through metrics, logs, and traces, helping teams detect and diagnose problems.

---

# 15. Final Conclusion

This Week 5 Integration Project combines the DevOps work completed during Days 30–35 into one application deployment lifecycle.

The project demonstrates the transition from application source code to Linux, Docker, Docker Compose, Kubernetes, and a proposed AWS architecture.

It also demonstrates the importance of configuration management, networking, persistence, security, monitoring, troubleshooting, and recovery.

The most important lesson is that DevOps is not just about deploying an application. It is about understanding how the application is built, packaged, deployed, monitored, secured, recovered, and improved throughout its lifecycle.

**Project:** Week 5 – DevOps Integration Project

**Technology Stack:**

- Linux (WSL Ubuntu)
- Docker
- Docker Compose
- Kubernetes
- Minikube
- AWS Architecture Design
- Amazon EKS (proposed)
- Application Load Balancer (proposed)
- Amazon RDS PostgreSQL (proposed)
- AWS IAM (proposed)
- Amazon CloudWatch (proposed)

**AWS Status:** DESIGN ONLY – NOT DEPLOYED.

**Final Outcome:** End-to-end understanding of the DevOps application deployment and operations lifecycle.
