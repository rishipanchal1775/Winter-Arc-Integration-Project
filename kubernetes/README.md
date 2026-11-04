
# Day 34 – Kubernetes Deployment & Operations Lab

## Lab Overview

The goal of Day 34 is to deploy the application used in Days 30–33 into a local Kubernetes cluster and practise basic Kubernetes deployment and operations.

In this lab, we will:

- Verify that the local Kubernetes cluster is running.
- Create a dedicated Kubernetes namespace.
- Make the application Docker image available to Minikube.
- Create a Kubernetes Deployment and Service.
- Configure application settings using a ConfigMap.
- Configure health probes where supported by the application.
- Access the application using port forwarding.
- Scale the Deployment.
- Perform a rollout and rollback.
- Troubleshoot common Kubernetes issues.

**Environment:** Minikube running locally with kubectl.

**Important:** This lab uses a local Minikube cluster. No AWS resources are required.

---

## Environment Setup

Check whether Minikube is installed:

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

If Minikube is stopped, start it:

```bash
minikube start
```

Check the current Kubernetes context:

```bash
kubectl config current-context
```

If the current context is not Minikube, switch to it:

```bash
kubectl config use-context minikube
```

Check the cluster nodes:

```bash
kubectl get nodes -o wide
```

The node should show `Ready` status.

---

## Task 1 – Confirm Cluster and Create Namespace

A namespace provides logical separation for Kubernetes resources.

Create the Day 34 namespace:

```bash
kubectl create namespace day34-revision
```

Verify that it was created:

```bash
kubectl get namespaces
```

Check the specific namespace:

```bash
kubectl get namespace day34-revision
```

All resources created for this lab will be placed inside the `day34-revision` namespace.

---

## Task 2 – Make the Docker Image Available to Minikube

Kubernetes needs access to the application image before it can start application containers.

Check whether the Day 32 image exists locally:

```bash
docker images
```

Look for the image:

```text
winter-arc-app:day32
```

If the image does not exist, build it again from the directory containing the Day 32 Dockerfile:

```bash
docker build -t winter-arc-app:day32 .
```

Load the image into Minikube:

```bash
minikube image load winter-arc-app:day32
```

Verify the image:

```bash
minikube image ls
```

In the Kubernetes Deployment, use:

```yaml
image: winter-arc-app:day32
imagePullPolicy: IfNotPresent
```

`IfNotPresent` tells Kubernetes to use the image already available locally when possible instead of unnecessarily trying to pull it from a remote registry.

**Important:** Use the exact image name and tag created during Day 32. If your image name is different, update the Kubernetes manifest accordingly.

---

## Task 3 – Create Kubernetes Deployment and Service

### 1. Create the Kubernetes directory

From your project directory:

```bash
mkdir -p k8s
cd k8s
```

### 2. Create the Deployment

Create a file named `deployment.yaml`.

The Deployment manages application Pods and maintains the desired number of replicas.

**For a static Nginx application using container port 8080:**

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
```

**For a Node.js application using container port 3000:**

Use the same Deployment structure, but change the container port to:

```yaml
ports:
  - containerPort: 3000
```

The container port must match the port on which your application actually listens.

### 3. Create the Service

Create a file named `service.yaml`.

A Service provides a stable network endpoint for accessing the application Pods.

**For static Nginx on port 8080:**

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

**For Node.js on port 3000:**

Use:

```yaml
targetPort: 3000
```

The Service listens on port 80 and forwards traffic to the application's container port.

### 4. Apply the Kubernetes manifests

From the directory containing the YAML files:

```bash
kubectl apply -f .
```

Check the Deployment, Pods, and Service:

```bash
kubectl -n day34-revision get deployments,pods,services
```

Check Deployment rollout status:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

You should see two application Pods running if the Deployment is healthy.

---

## Task 4 – Configure Runtime Settings and Health Probes

### 1. Create a ConfigMap

A ConfigMap stores non-sensitive application configuration separately from the container image.

Create `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: revision-app-config
  namespace: day34-revision
data:
  APP_ENV: "revision"
```

Apply it:

```bash
kubectl apply -f configmap.yaml
```

### 2. Add ConfigMap settings to the Deployment

Inside the container definition in `deployment.yaml`, add:

```yaml
envFrom:
  - configMapRef:
      name: revision-app-config
```

This makes the ConfigMap values available as environment variables inside the container.

Apply the updated Deployment:

```bash
kubectl apply -f deployment.yaml
```

Verify the ConfigMap:

```bash
kubectl -n day34-revision get configmap
```

Check the environment variable inside a running Pod:

```bash
kubectl -n day34-revision exec deployment/revision-app -- printenv APP_ENV
```

Expected output:

```text
revision
```

### 3. Configure health probes

Kubernetes uses probes to monitor container health.

- **Readiness probe:** Determines whether a container is ready to receive traffic.
- **Liveness probe:** Determines whether a container should be restarted.

If your application has a real `/health` HTTP endpoint, add the following under the container definition. Use the correct application port.

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```

For Node.js, replace port `8080` with `3000`.

**Important:** Do not configure an HTTP probe for `/health` unless your application really serves that endpoint. If it does not, use an appropriate TCP probe or explain why an HTTP probe is unsuitable.

For example, a TCP probe checks whether the application port accepts connections:

```yaml
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```

Apply the updated Deployment:

```bash
kubectl apply -f deployment.yaml
```

Check the Pods:

```bash
kubectl -n day34-revision get pods
```

---

## Task 5 – Access the Application Using Port Forwarding

The Service is configured as `ClusterIP`, which means it is accessible inside the Kubernetes cluster.

Use port forwarding to access it from your local machine.

Run:

```bash
kubectl -n day34-revision port-forward service/revision-app 8080:80
```

Keep this terminal running.

Open another terminal and test the application:

```bash
curl -i http://localhost:8080
```

You can also open the following address in your browser:

```text
http://localhost:8080
```

You should see the application response.

To stop port forwarding, press:

```text
Ctrl + C
```

---

## Task 6 – Scaling, Rollout, and Recovery

### 1. Scale the Deployment

Initially, the Deployment has two replicas.

Scale it to three replicas:

```bash
kubectl -n day34-revision scale deployment/revision-app --replicas=3
```

Check the Pods:

```bash
kubectl -n day34-revision get pods
```

Check the Deployment:

```bash
kubectl -n day34-revision get deployment revision-app
```

The desired replica count should now be three.

### 2. Check Rollout History

View the Deployment rollout history:

```bash
kubectl -n day34-revision rollout history deployment/revision-app
```

This displays the available Deployment revisions.

### 3. Perform a Safe Rollout Update

For a safe local rollout demonstration, update a harmless Pod-template annotation:

```bash
kubectl -n day34-revision patch deployment revision-app -p '{"spec":{"template":{"metadata":{"annotations":{"day34-revision":"update-1"}}}}}'
```

Check the rollout:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

View the rollout history again:

```bash
kubectl -n day34-revision rollout history deployment/revision-app
```

The Pod-template change creates a new Deployment revision.

### 4. Roll Back the Deployment

Undo the latest rollout:

```bash
kubectl -n day34-revision rollout undo deployment/revision-app
```

Check the rollback:

```bash
kubectl -n day34-revision rollout status deployment/revision-app
```

Verify the Pods:

```bash
kubectl -n day34-revision get pods
```

The Deployment should return to its previous Pod-template revision.

---

## Troubleshooting Reference

### 1. ImagePullBackOff

**Problem:** Kubernetes cannot find or pull the application image.

Check the Pod:

```bash
kubectl -n day34-revision get pods
```

Describe the Pod:

```bash
kubectl -n day34-revision describe pod POD_NAME
```

Check the local Minikube images:

```bash
minikube image ls
```

Load the image again if necessary:

```bash
minikube image load winter-arc-app:day32
```

Check that the image name, tag, and `imagePullPolicy` match.

### 2. CrashLoopBackOff

**Problem:** The container repeatedly starts and exits.

Check the logs:

```bash
kubectl -n day34-revision logs deployment/revision-app
```

Describe the Pod:

```bash
kubectl -n day34-revision describe pod POD_NAME
```

Check the application start command, configuration, and container port.

### 3. Service Not Reachable

Check the Service:

```bash
kubectl -n day34-revision get services
```

Check the Pods and labels:

```bash
kubectl -n day34-revision get pods --show-labels
```

Check the Service details:

```bash
kubectl -n day34-revision describe service revision-app
```

Make sure the Service selector matches the Pod labels and that `targetPort` matches the application container port.

### 4. Probe Failures

Check Pod details:

```bash
kubectl -n day34-revision describe pod POD_NAME
```

Check application logs:

```bash
kubectl -n day34-revision logs POD_NAME
```

Verify that the configured health endpoint exists and that the probe uses the correct port.

### 5. Rollout Stuck

Check Deployment status:

```bash
kubectl -n day34-revision describe deployment revision-app
```

Check Pod status:

```bash
kubectl -n day34-revision get pods
```

Check recent events:

```bash
kubectl -n day34-revision get events --sort-by=.lastTimestamp
```

Inspect the error, correct the issue, and check rollout status again.

---

## Useful Kubernetes Commands

| Command | Purpose |
|---|---|
| `kubectl get nodes` | List cluster nodes |
| `kubectl get namespaces` | List namespaces |
| `kubectl get pods` | List Pods |
| `kubectl get deployments` | List Deployments |
| `kubectl get services` | List Services |
| `kubectl describe pod POD_NAME` | Inspect a Pod |
| `kubectl logs POD_NAME` | View container logs |
| `kubectl exec -it POD_NAME -- sh` | Open a shell inside a Pod |
| `kubectl apply -f .` | Apply YAML manifests |
| `kubectl delete -f .` | Delete resources described by manifests |
| `kubectl rollout status deployment/NAME` | Check rollout progress |
| `kubectl rollout history deployment/NAME` | View rollout history |
| `kubectl rollout undo deployment/NAME` | Roll back a Deployment |
| `kubectl scale deployment/NAME --replicas=3` | Change replica count |

Remember to add `-n day34-revision` to commands when working with resources in this lab namespace.

---

## Interview Questions

**1. What is Kubernetes?**

Kubernetes is a container orchestration platform used to deploy, manage, scale, and monitor containerized applications.

**2. What is a Pod?**

A Pod is the smallest deployable unit in Kubernetes. It can contain one or more containers.

**3. What is a Deployment?**

A Deployment manages ReplicaSets and maintains the desired number of application Pods.

**4. What is a Kubernetes Service?**

A Service provides a stable network endpoint for accessing a group of Pods.

**5. What is ClusterIP?**

ClusterIP is the default Service type. It exposes an application through an internal cluster IP.

**6. What is a ConfigMap?**

A ConfigMap stores non-sensitive configuration separately from the application container image.

**7. What is a readiness probe?**

A readiness probe checks whether a container is ready to receive traffic.

**8. What is a liveness probe?**

A liveness probe checks whether a container is healthy. Kubernetes can restart a container when its liveness probe repeatedly fails.

**9. What is the difference between readiness and liveness probes?**

Readiness controls whether a Pod receives traffic. Liveness helps Kubernetes determine whether a container needs to be restarted.

**10. What is scaling in Kubernetes?**

Scaling means increasing or decreasing the number of application replicas.

**11. What is a rollout?**

A rollout is the process of updating a Deployment and replacing Pods with the updated version.

**12. How do you roll back a Deployment?**

Use:

```bash
kubectl rollout undo deployment/DEPLOYMENT_NAME
```

**13. What is ImagePullBackOff?**

It means Kubernetes is unable to pull the requested container image.

**14. What is CrashLoopBackOff?**

It means a container repeatedly starts and exits, and Kubernetes is delaying subsequent restart attempts.

**15. Why use Minikube?**

Minikube provides a local Kubernetes environment for learning, testing, and practising Kubernetes deployments without requiring a cloud cluster.

---

## Key Learnings

After completing Day 34, I understand how to:

- Run and verify a local Kubernetes cluster using Minikube.
- Create and use Kubernetes namespaces.
- Load a local Docker image into Minikube.
- Deploy an application using Kubernetes Deployment YAML.
- Expose Pods through a ClusterIP Service.
- Configure application settings using ConfigMaps.
- Understand readiness and liveness probes.
- Access an application using port forwarding.
- Scale application replicas.
- Inspect Deployment rollout history.
- Perform rollout updates and rollbacks.
- Troubleshoot common Kubernetes deployment and networking issues.

**Day 34 completed: Kubernetes Deployment & Operations Lab.**
