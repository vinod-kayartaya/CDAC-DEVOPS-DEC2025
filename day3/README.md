# Docker and Containerization

## 1. What Is Containerization?

Containerization is a way of packaging an application along with everything it needs to run:

- Application code
- Runtime (for example, Java, Python, Node.js)
- Libraries and dependencies
- Configuration files

All of this is bundled into a **container**, which runs consistently across different environments such as:

- Developer laptop
- Test server
- Production server

### Why containerization matters

- Eliminates “it works on my machine” problems
- Makes application deployment faster
- Improves portability and scalability

## 2. Containers vs Virtual Machines

### Virtual Machines (VMs)

- Each VM includes a full operating system
- Heavy on memory and disk usage
- Slower startup time

### Containers

- Share the host operating system kernel
- Lightweight and fast
- Start in seconds (or milliseconds)

**Key difference:**
VMs virtualize hardware, while containers virtualize the operating system.

## 3. What Is Docker?

Docker is a platform that helps you:

- Build container images
- Run containers
- Share container images using registries like Docker Hub

### Core Docker components

- **Docker Engine** – Runs containers
- **Docker Image** – Blueprint for a container
- **Docker Container** – A running instance of an image
- **Dockerfile** – Instructions to build an image
- **Docker Hub** – Public image registry

## 4. Your First Docker Command

### Create a container

```bash
docker run hello-world
docker run learnwithvinod/whois
docker run -d -i -t busybox ping -c 100 vinod.co
```

### Check running containers

```bash
docker ps
```

### Check all containers (including stopped)

```bash
docker ps -a
```

## 5. Understanding Docker Images and Containers

### Image

- Read-only template
- Example: `nginx`, `python`, `mysql`

### Container

- Running instance of an image
- You can start, stop, delete containers

Example:

```bash
docker run -d --name my-nginx nginx
```

Check status:

```bash
docker ps
```

Stop container:

```bash
docker stop my-nginx
```

Remove container:

```bash
docker rm my-nginx
```

## 6. Running a Web Server Using Docker (Practical Example)

### Step 1: Run Nginx container

```bash
docker run -d -p 8080:80 nginx
```

### Step 2: Open browser

Visit:

```
http://localhost:8080
```

You should see the Nginx welcome page.

### Explanation

- `-d` → run in detached mode
- `-p 8080:80` → map host port 8080 to container port 80
- `nginx` → image name

## 7. Creating Your Own Docker Image (Dockerfile)

Let us containerize a simple Python application.

### Step 1: Create a project folder

```bash
mkdir python-docker-demo
cd python-docker-demo
```

### Step 2: Create `app.py`

```python
print("Hello from Docker container")
```

### Step 3: Create `Dockerfile`

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

### Step 4: Build the image

```bash
docker build -t my-python-app .
```

### Step 5: Run the container

```bash
docker run my-python-app
```

Output:

```
Hello from Docker container
```

## 8. Docker Container Lifecycle

![Image](https://linuxhandbook.com/content/images/2021/05/docker-container-lifecycle-diagram.png)

![Image](https://miro.medium.com/1%2AuuZ-h5EH76LOtJ614z-qDA.png)

1. Image is built or pulled
2. Container is created
3. Container starts running
4. Container stops
5. Container is removed

Useful commands:

```bash
docker start <container-id>
docker stop <container-id>
docker restart <container-id>
docker rm <container-id>
```

## 9. Data Persistence with Volumes

By default, container data is temporary. Docker volumes help persist data.

### Example: Using a volume

```bash
docker run -d \
  -v mydata:/data \
  busybox \
  sh -c "echo Hello > /data/hello.txt && sleep 3600"
```

Check volume:

```bash
docker volume ls
```

## 10. Environment Variables in Containers

```bash
docker run -e APP_ENV=production busybox env
```

This is commonly used for:

- Database credentials
- Application configuration
- Environment-specific settings

## 11. Common Docker Commands Cheat Sheet

| Command         | Purpose                 |
| --------------- | ----------------------- |
| `docker images` | List images             |
| `docker ps`     | List running containers |
| `docker run`    | Run container           |
| `docker stop`   | Stop container          |
| `docker rm`     | Remove container        |
| `docker rmi`    | Remove image            |
| `docker logs`   | View logs               |

## 12. Real-World Use Cases

- Microservices-based applications
- CI/CD pipelines
- Local development environments
- Cloud-native deployments
- Application isolation

---

# Kubernetes

## What is Kubernetes?

Kubernetes (often called **K8s**) is an **open-source container orchestration platform** that helps you:

- Run containers at scale
- Automatically restart failed containers
- Distribute traffic (load balancing)
- Scale applications up or down
- Perform rolling updates without downtime

Think of Kubernetes as **an operating system for containers**.

## Why Do We Need Kubernetes?

You already know how to run containers using **Docker**.
But Docker alone becomes difficult when:

- You have **many containers**
- Containers **crash unexpectedly**
- You need **scaling**
- You want **zero-downtime deployments**
- Applications run on **multiple machines**

Kubernetes solves all these problems.

## Real-World Analogy

| Concept    | Real World          |
| ---------- | ------------------- |
| Container  | A shop              |
| Pod        | A shop unit         |
| Node       | A shopping mall     |
| Cluster    | City of malls       |
| Kubernetes | City administration |

## Kubernetes Architecture (High Level)

![Image](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)

### Control Plane (Brain)

Responsible for **decisions and management**.

| Component          | Purpose                         |
| ------------------ | ------------------------------- |
| API Server         | Entry point for all commands    |
| Scheduler          | Decides where pods run          |
| Controller Manager | Maintains desired state         |
| etcd               | Key-value store (cluster state) |

### Worker Nodes (Muscle)

Runs the actual applications.

| Component         | Purpose                     |
| ----------------- | --------------------------- |
| kubelet           | Talks to control plane      |
| Container Runtime | Runs containers             |
| kube-proxy        | Networking & load balancing |

## Core Kubernetes Objects

### Pod (Smallest Unit)

![Image](https://phoenixnap.com/kb/wp-content/uploads/2021/04/components-worker-node-kubernetes.png)

![Image](https://www.couchbase.com/blog/wp-content/uploads/2024/07/image1-2.png)

- A **Pod** contains **one or more containers**
- Containers in a pod share:

  - IP address
  - Network
  - Storage

Example:

```text
Pod
 ├── Container: web-app
 └── Container: log-agent
```

### Node

- A **Node** is a machine (VM or physical)
- Runs Pods
- Managed by Kubernetes

### Cluster

- A group of nodes
- Managed by a single Kubernetes control plane

## Kubernetes Manifest (YAML)

Kubernetes uses **YAML files** to describe desired state.

### Example: Pod Definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

### This file tells Kubernetes:

- WHAT to run
- HOW to run it
- WHERE it should live

## Deployments (Recommended Way)

![Image](https://zesty.co/wp-content/uploads/2025/02/Replica-Set.png)

A **Deployment**:

- Manages Pods
- Supports scaling
- Supports rolling updates
- Restarts failed pods

### Example Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

## Service (Accessing Pods)

Pods have **dynamic IPs**.
Services provide **stable access**.

### Service Types

| Type         | Use                    |
| ------------ | ---------------------- |
| ClusterIP    | Internal access        |
| NodePort     | External (via node IP) |
| LoadBalancer | Cloud environments     |

### Example Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

## Scaling Applications

```bash
kubectl scale deployment nginx-deploy --replicas=5
```

Kubernetes:

- Creates new pods
- Load balances traffic
- No downtime

## Self-Healing

If a pod crashes:

- Kubernetes **automatically restarts it**
- If a node fails:

  - Pods are moved to other nodes

No manual intervention needed.

## Rolling Updates

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.25
```

✔ Old pods are terminated gradually
✔ New pods come up
✔ Zero downtime

## Common kubectl Commands

| Command                        | Purpose          |
| ------------------------------ | ---------------- |
| `kubectl get pods`             | List pods        |
| `kubectl get nodes`            | List nodes       |
| `kubectl apply -f file.yaml`   | Apply config     |
| `kubectl describe pod podname` | Pod details      |
| `kubectl logs podname`         | View logs        |
| `kubectl delete -f file.yaml`  | Delete resources |

## Beginner Practice Path

### Step 1

- Install **Minikube** or **Kind**
- Start a local cluster

### Step 2

- Deploy a Pod
- Convert Pod → Deployment

### Step 3

- Expose using Service
- Access from browser

### Step 4

- Scale replicas
- Perform rolling update

## Key Takeaways

- Docker runs **containers**
- Kubernetes manages **containers at scale**
- Pods → Deployments → Services
- Kubernetes ensures:

  - High availability
  - Auto-scaling
  - Self-healing
  - Zero-downtime deployments
