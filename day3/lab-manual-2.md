# Kubernetes Lab Manual

**Deploying and Accessing Multi-Service Applications Using Minikube**

## Lab Overview

In this lab, you will deploy **three microservices** on Kubernetes:

1. Categories service
2. Products service
3. Suppliers service

The services will communicate **internally using Kubernetes Services**, and the **Products service** will be exposed **externally using NodePort** so that it can be accessed using the **Minikube public IP**.

## Lab Objectives

By completing this lab, you will be able to:

1. Create Pods using Kubernetes Deployments
2. Create internal and external Kubernetes Services
3. Understand how Kubernetes service discovery works
4. Verify inter-service communication
5. Access a service using Minikube IP and NodePort

## Prerequisites

### System Requirements

- Minimum 8 GB RAM
- Internet connectivity

### Software Requirements

- Docker installed and running
- Minikube installed
- kubectl installed

## Lab 1: Start the Kubernetes Cluster

### Step 1: Start Minikube

```bash
minikube start
```

### Step 2: Verify Cluster Status

```bash
kubectl get nodes
```

Expected:

- One node in `Ready` state

## Lab 2: Create the Kubernetes Manifest File

### Step 1: Create a YAML File

Create a file named:

```bash
nw-apps.yaml
```

This single file will contain **all Deployments and Services** required for this lab.

## Lab 3: Categories Service Deployment and Service

### Categories Deployment YAML

Add the following content to `nw-apps.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nw-categories
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nw-categories
  template:
    metadata:
      labels:
        app: nw-categories
    spec:
      containers:
        - name: nw-categories
          image: learnwithvinod/nw-categories
          ports:
            - containerPort: 8080
```

### Explanation

- **Deployment** ensures the Categories pod is always running
- `replicas: 1` means one pod will be created
- `labels` and `selector` connect the Deployment to its Pods
- The container listens on port **8080**

### Categories Service YAML

Append the following:

```yamlapiVersion: v1
kind: Service
metadata:
  name: nw-categories-service
spec:
  selector:
    app: nw-categories
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

### Explanation

- This is a **ClusterIP** service (default type)
- It exposes Categories **only inside the cluster**
- Other services can access it using:

  ```
  http://nw-categories-service:8080
  ```

## Lab 4: Suppliers Service Deployment and Service

### Suppliers Deployment YAML

```yamlapiVersion: apps/v1
kind: Deployment
metadata:
  name: nw-suppliers
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nw-suppliers
  template:
    metadata:
      labels:
        app: nw-suppliers
    spec:
      containers:
        - name: nw-suppliers
          image: learnwithvinod/nw-suppliers
          ports:
            - containerPort: 8080
```

### Explanation

- Similar to Categories deployment
- Runs the Suppliers microservice on port **8080**

### Suppliers Service YAML

```yamlapiVersion: v1
kind: Service
metadata:
  name: nw-suppliers-service
spec:
  selector:
    app: nw-suppliers
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

### Explanation

- Internal service only
- Accessible inside the cluster using:

  ```
  http://nw-suppliers-service:8080
  ```

## Lab 5: Products Service Deployment (Dependent Service)

### Products Deployment YAML

```yamlapiVersion: apps/v1
kind: Deployment
metadata:
  name: nw-products
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nw-products
  template:
    metadata:
      labels:
        app: nw-products
    spec:
      containers:
        - name: nw-products
          image: learnwithvinod/nw-products
          ports:
            - containerPort: 8080
          env:
            - name: CATEGORY_SERVICE_HOST
              value: nw-categories-service
            - name: CATEGORY_SERVICE_PORT
              value: '8080'
            - name: SUPPLIER_SERVICE_HOST
              value: nw-suppliers-service
            - name: SUPPLIER_SERVICE_PORT
              value: '8080'
```

### Explanation

- Products service depends on **Categories** and **Suppliers**
- Service names are injected as environment variables
- Kubernetes DNS resolves service names automatically
- No IP addresses are hardcoded

## Lab 6: Products Service (NodePort)

### Products Service YAML

```yamlapiVersion: v1
kind: Service
metadata:
  name: nw-products-service
spec:
  selector:
    app: nw-products
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30000
  type: NodePort
```

### Explanation

- This service is exposed **outside the cluster**
- `NodePort` opens port **30000** on the Minikube node
- External users can access Products service using:

  ```
  http://<MINIKUBE-IP>:30000
  ```

## Lab 7: Deploy All Resources

### Step 1: Apply the YAML File

```bash
kubectl apply -f nw-apps.yaml
```

### Step 2: Verify Deployments

```bash
kubectl get deployments
```

Expected:

- nw-categories
- nw-suppliers
- nw-products

### Step 3: Verify Pods

```bash
kubectl get pods
```

Expected:

- One pod per deployment
- Status should be `Running`

## Lab 8: Verify Services

```bash
kubectl get services
```

Expected:

- Internal services:

  - nw-categories-service
  - nw-suppliers-service

- External service:

  - nw-products-service (NodePort)

## Lab 9: Access Products Service Using Minikube IP

### Step 1: Get Minikube IP

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

### Step 2: Access the Application

```text
http://<MINIKUBE-IP>:30000
```

Expected:

- Products service responds successfully
- Data from Categories and Suppliers is included

## Lab 10: Verification and Troubleshooting

### Check Pod Logs

```bash
kubectl logs <nw-products-pod-name>
```

### Check Service Endpoints

```bash
kubectl get endpoints nw-products-service
```

## Lab 11: Cleanup (Optional)

```bash
kubectl delete -f nw-apps.yaml
minikube stop
```

## Acceptance Criteria (Self-Assessment)

You should now be able to:

- Explain the role of each Deployment
- Explain why some services are ClusterIP and one is NodePort
- Explain how service discovery works using service names
- Successfully access Products service externally
- Troubleshoot common connectivity issues
