# 🚀 DevOps Winter Arc — Integration Project

## 📌 Project Name - Winter Arc Integration Project

**Deploy, Monitor & Recover a Containerized Application**

A hands-on DevOps integration project connecting the skills learned during Days 1–29 of the DevOps Winter Arc Challenge.

The project demonstrates how to deploy the same application across Linux, Docker, Docker Compose and Kubernetes, understand its AWS architecture, and perform monitoring, troubleshooting and recovery.

---

## 🎯 Project Purpose

The main objective of this project is to combine individual DevOps concepts into one practical end-to-end workflow.

Instead of learning each technology separately, this project demonstrates how different DevOps tools work together in a real deployment environment.

### Key objectives

- Deploy and manage a web application on a Linux host.
- Containerize the application using Docker.
- Manage application services using Docker Compose.
- Deploy and manage the application using Kubernetes.
- Understand cloud infrastructure and security using AWS architecture.
- Monitor application health and troubleshoot failures.
- Practise application recovery and deployment verification.
- Maintain project documentation and evidence using Git and GitHub.

**Final goal:** Build a documented DevOps project that demonstrates the ability to deploy, monitor, troubleshoot and recover an application across different environments.

---

## 💻 Chosen Application

### Winter Arc Web App

| Component | Technology |
|---|---|
| Application | Static website |
| Frontend | HTML and CSS |
| Web Server | Nginx |
| Containerization | Docker |
| Orchestration | Docker Compose and Kubernetes |
| Local Kubernetes | kind |
| Cloud Architecture | AWS |

### Application functionality

The application will contain two main endpoints:

| Endpoint | Purpose |
|---|---|
| `/` | Main application homepage |
| `/health.html` | Application health verification |

### Why did I choose a static Nginx application?

- Nginx is already familiar from previous Docker and Linux tasks.
- HTML and CSS keep application development simple.
- It allows the focus to remain on DevOps concepts.
- The same application can be deployed across Linux, Docker, Compose and Kubernetes.
- Health checks and failure recovery can be demonstrated without unnecessary application complexity.

The application will remain consistent throughout the integration project.

---

## 🏗️ Project Architecture

The project consists of three major stages:

1. Local Linux host and Docker deployment.
2. Local Kubernetes deployment using kind.
3. AWS cloud architecture design.

![Winter Arc Integration Architecture](docs/architecture.png)


### 1. Linux Host + Docker — Local Environment

The application initially runs on a local Linux environment.

```text
       User / Browser
              |
              v
          Linux Host
              |
              v
       Docker Container
              |
              v
         Nginx Server
              |
              v
       Web Application
          HTML / CSS
```

**Request flow:**

1. A user sends an HTTP request.
2. The request reaches the Linux host.
3. Docker forwards the mapped host port to the container.
4. Nginx receives the request on port 80.
5. Nginx serves the static HTML application.

**Environment:** Local Linux / WSL2 / VM.

### 2. Local Kubernetes — kind

The same application is deployed inside a local Kubernetes cluster using kind.

```text
        User / Browser
               |
               v
       Local kind Cluster
               |
               v
            Service
               |
               v
          Deployment
               |
               v
        +------+------+ 
        |      |      |
        v      v      v
       Pod    Pod    Pod
        |      |      |
        +------+------+ 
               |
               v
         Nginx Application
```

**Key components:**

- **kind:** Runs a local Kubernetes cluster using containers.
- **Deployment:** Manages the desired number of application replicas.
- **Pods:** Run the Nginx application containers.
- **Service:** Provides stable network access to the application Pods.

**Environment:** Local Kubernetes cluster running through kind.

### 3. AWS Architecture — Conceptual Design Only

The final stage introduces how the application could be deployed in AWS.

```text
                Internet
                    |
                    v
              AWS Region
                    |
                    v
                   VPC
                    |
          +---------+----------+
          |                    |
          v                    |
     Public Subnet             |
          |                    |
          v                    |
     Public ALB                 |
          |                    |
          v                    |
     Private Subnet             |
          |                    |
          v                    |
      Application Tier          |
       EC2 / EKS                |
          |                    |
          v                    |
      Private Database          |
          RDS                   |
                               |
```

### AWS components

| Component | Purpose |
|---|---|
| AWS Region | Geographic location for AWS resources |
| VPC | Isolated virtual network |
| Public Subnets | Host the internet-facing load balancer |
| Application Load Balancer | Distributes incoming application traffic |
| Private Subnets | Host application and database resources |
| EC2 / EKS | Runs the application workload |
| RDS | Managed relational database |

**Important:** This is a conceptual architecture design. No AWS infrastructure will be deployed during the initial planning stage. Actual AWS deployment will only happen if explicitly approved.

---

## 🛠️ Prerequisites

The following tools and technologies are required throughout the project.

### 1. Operating System

- Ubuntu Linux / WSL2
- Linux terminal access
- Basic Linux command-line knowledge

### 2. Linux Tools

```bash
sudo apt update
sudo apt install -y git curl wget nano tree
```

Used for:
- File and directory management.
- Process and service management.
- Networking and connectivity testing.
- Log inspection and troubleshooting.

### 3. Git and GitHub

```bash
git --version
```

Used for:
- Version control.
- Tracking project changes.
- Creating commits.
- Pushing project files to GitHub.

### 4. Docker

```bash
docker --version
docker compose version
```

Used for:
- Building application images.
- Running containers.
- Managing container networking and storage.
- Running multi-container applications.

### 5. Kubernetes

```bash
kubectl version --client
kind version
```

Used for:
- Creating a local Kubernetes cluster.
- Deploying applications.
- Managing Pods and Deployments.
- Configuring Services and health probes.
- Testing scaling and rollback.

### 6. AWS Knowledge

No AWS deployment is required for the initial project.

Required conceptual knowledge:

- IAM
- VPC
- Public and private subnets
- EC2
- EKS
- Security Groups
- Application Load Balancer
- RDS

### Verify installed tools

```bash
git --version
docker --version
docker compose version
kubectl version --client
kind version
curl --version
```

---

## 📅 Week 5 — Seven-Day Project Roadmap

This integration project is divided into seven stages, from planning to final demonstration.

| Day | Topic | Tasks | Expected Deliverable |
|---|---|---|---|
| **Day 30** | Project Kickoff & Planning | Choose application, create GitHub repository, draw architecture, create revision tracker | Project structure and architecture |
| **Day 31** | Linux & Networking | Run application on Linux, manage permissions, inspect processes, services, ports and logs | Working Linux-hosted application |
| **Day 32** | Docker Integration | Write Dockerfile, build image, run container, configure health checks and non-root execution | Reusable Docker image |
| **Day 33** | Docker Compose | Configure application services, environment variables, health checks and persistent storage | Multi-container application |
| **Day 34** | Kubernetes Integration | Deploy application using kind, configure Deployment, Service, ConfigMaps, probes, scaling and rollback | Working Kubernetes deployment |
| **Day 35** | AWS Architecture & Security | Design IAM, VPC, ALB, private application tier and database architecture | AWS architecture documentation |
| **Day 36** | Break, Fix & Final Demo | Introduce failures, troubleshoot, recover application, verify deployment and document results | Completed integration project |

---

## 📂 Project Folder Structure

The main project repository will follow this structure:

```text
Winter-Arc-Integration-Project/
│
├── README.md
│
├── app/
│   ├── index.html
│   └── health.html
│
├── linux/
│   └── README.md
│
├── docker/
│   ├── Dockerfile
│   └── README.md
│
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── README.md
│
├── aws-design/
│   └── architecture.md
│
├── docs/
│   ├── architecture.md
│   └── revision-tracker.md
│
└── screenshots/
    ├── day30/
    ├── day31/
    ├── day32/
    ├── day33/
    ├── day34/
    ├── day35/
    └── day36/
```

Additional files such as Docker Compose configurations, ConfigMaps, Secrets examples and monitoring configurations will be added as the project progresses.

---

## 📊 Revision Tracker

During Day 30, the skills learned from Days 1–29 will be reviewed.

Each skill will be categorized into:

- **Confident:** Able to explain and perform independently.
- **Need Practice:** Understand the concept but need more hands-on experience.
- **Need Help:** Require additional explanation and practice.

Refer to:

`docs/revision-tracker.md`

The three weakest areas identified during the revision will receive additional attention throughout the integration week.

---

## 🔐 Security Considerations

Security practices will be followed throughout the project:

- Never commit passwords, private keys or access tokens.
- Use `.gitignore` to exclude sensitive files.
- Use environment variables for application configuration.
- Use Kubernetes Secrets for sensitive configuration, with appropriate access controls.
- Run Docker containers as non-root users where appropriate.
- Avoid exposing unnecessary container ports.
- Use private subnets for application and database infrastructure in the AWS design.
- Follow least-privilege principles for AWS IAM.
- Avoid deploying paid AWS resources without approval.

---

## 🎯 Final Project Deliverables

By the end of Day 36, the repository should contain:

1. A working static Nginx application.
2. Linux deployment documentation and troubleshooting evidence.
3. A reusable Dockerfile and built image.
4. Docker Compose configuration.
5. Kubernetes Deployment and Service manifests.
6. Kubernetes configuration and health-check evidence.
7. AWS architecture and security documentation.
8. Failure and recovery test evidence.
9. Updated project documentation.
10. GitHub repository containing the complete project.

---

## 🏁 Final Outcome

This project connects the DevOps skills learned throughout the Winter Arc challenge into one practical workflow.

**The complete journey:**

```text
Linux
  |
  v
Docker
  |
  v
Docker Compose
  |
  v
Kubernetes
  |
  v
AWS Architecture
  |
  v
Monitoring & Troubleshooting
  |
  v
Failure Recovery
  |
  v
Final DevOps Project
```

The goal is not just to run an application.

The goal is to understand how to deploy it, manage it, monitor it, troubleshoot it, secure it and recover it when something goes wrong.

**Deploy. Monitor. Troubleshoot. Recover.**

*DevOps Winter Arc — Week 5 Integration Project*
