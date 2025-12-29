# Kubernetes Self-Learning Lab Guide (Beginner Level)

## Lab Objectives

By the end of this lab, the student will be able to:

1. Install and run a local Kubernetes cluster
2. Understand Kubernetes architecture practically
3. Create and manage Pods
4. Create and manage Deployments
5. Expose applications using Services
6. Scale applications
7. Perform rolling updates
8. Verify and troubleshoot Kubernetes resources

## Prerequisites

### Hardware

- Minimum 8 GB RAM (16 GB recommended)
- Internet connectivity

### Software

- Docker installed and running
- Terminal access (Linux / macOS / Windows with WSL)

## Lab Environment Overview

- Local Kubernetes cluster using **Minikube**
- kubectl command-line tool
- nginx container image

## Lab 1: Installing Kubernetes Tooling

### Step 1: Install kubectl

#### Linux / macOS

```bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl version --client
```

Expected:

- kubectl client version is displayed

### Step 2: Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify:

```bash
minikube version
```

### Step 3: Start Kubernetes Cluster

```bash
minikube start
```

This command:

- Creates a VM
- Installs Kubernetes
- Configures kubectl

Verify cluster:

```bash
kubectl get nodes
```

Expected output:

- One node in `Ready` state

## Lab 2: Understanding Kubernetes Architecture (Observation)

![Image](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)

![Image](https://kubernetes.io/images/docs/components-of-kubernetes.svg)

### What to Observe

- One node acts as both control plane and worker
- Kubernetes components are running as pods

Run:

```bash
kubectl get pods -n kube-system
```

Expected:

- Pods like `kube-apiserver`, `etcd`, `coredns`

## Lab 3: Creating Your First Pod

### Step 1: Create Pod YAML

Create a file `nginx-pod.yaml`

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

### Step 2: Apply the Pod

```bash
kubectl apply -f nginx-pod.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

- Pod status is `Running`

### Step 3: Describe the Pod

```bash
kubectl describe pod nginx-pod
```

Observe:

- Pod IP
- Container details
- Events section

### Step 4: View Pod Logs

```bash
kubectl logs nginx-pod
```

### Step 5: Delete the Pod

```bash
kubectl delete pod nginx-pod
```

Observation:

- Pod is removed permanently
- Kubernetes does not recreate standalone pods

## Lab 4: Creating a Deployment (Recommended Way)

![Image](https://zesty.co/wp-content/uploads/2025/02/Replica-Set.png)

![Image](https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates2.svg)

### Step 1: Create Deployment YAML

Create `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
          ports:
            - containerPort: 80
```

### Step 2: Apply Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

Verify:

```bash
kubectl get deployments
kubectl get pods
```

Expected:

- Two pods created automatically

### Step 3: Delete One Pod Manually

```bash
kubectl delete pod <pod-name>
```

Verify:

```bash
kubectl get pods
```

Observation:

- Kubernetes recreates the deleted pod
- Demonstrates self-healing

## Lab 5: Exposing Application Using Service

![Image](https://assets.bytebytego.com/diagrams/0005-4-k8s-service-types.png)

![Image](https://i.sstatic.net/1lunW.png)

### Step 1: Create Service YAML

Create `nginx-service.yaml`

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

### Step 2: Apply Service

```bash
kubectl apply -f nginx-service.yaml
```

Verify:

```bash
kubectl get services
```

### Step 3: Access Application

```bash
minikube service nginx-service
```

Expected:

- Browser opens nginx welcome page

## Lab 6: Scaling the Application

### Step 1: Scale Deployment

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Verify:

```bash
kubectl get pods
```

Expected:

- Five running pods

### Step 2: Observe Load Balancing

Refresh browser multiple times:

- Requests are served by different pods

## Lab 7: Rolling Updates

### Step 1: Update Container Image

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25
```

### Step 2: Watch Rollout Status

```bash
kubectl rollout status deployment nginx-deployment
```

Verify:

```bash
kubectl get pods
```

Observation:

- Old pods terminate gradually
- New pods start without downtime

### Step 3: Rollback (Optional)

```bash
kubectl rollout undo deployment nginx-deployment
```

## Lab 8: Useful kubectl Commands

| Command          | Purpose             |
| ---------------- | ------------------- |
| kubectl get all  | View all resources  |
| kubectl describe | Detailed inspection |
| kubectl logs     | View logs           |
| kubectl exec     | Execute inside pod  |
| kubectl delete   | Remove resources    |

Example:

```bash
kubectl exec -it <pod-name> -- bash
```

## Lab 9: Cleanup Resources

```bash
kubectl delete deployment nginx-deployment
kubectl delete service nginx-service
```

Stop cluster:

```bash
minikube stop
```

## Acceptance Criteria (Self-Validation)

The student should be able to:

- Explain difference between Pod and Deployment
- Demonstrate self-healing by deleting a pod
- Scale replicas up and down
- Access application using Service
- Perform rolling updates
- Use kubectl confidently

## What to Learn Next

1. ConfigMaps and Secrets
2. Persistent Volumes
3. Ingress Controllers
4. Helm Package Manager
5. Kubernetes on Cloud Platforms
