# SRE Kubernetes Assessment – Minikube, Services, and Ingress

## Overview

This project demonstrates how to deploy and expose two web applications (**NGINX** and **Apache HTTPD**) on a **Minikube Kubernetes cluster**.
Applications are exposed internally using **ClusterIP Services**, and external traffic is routed using an **NGINX Ingress Controller** with path-based routing.

---

## Objectives

* Set up a Minikube Kubernetes cluster
* Deploy two web applications (NGINX and Apache)
* Expose applications using Kubernetes Services
* Configure Ingress for HTTP routing
* Access multiple applications via a single endpoint

---

## Prerequisites

* Linux VM (VMware)
* Docker
* kubectl
* Minikube

---

## Project Structure

```text
k8s-nginx-apache/
├── nginx-deployment.yaml
├── httpd-deployment.yaml
├── ingress.yaml
└── README.md
```

---

## Step 1: Start Minikube

```bash
minikube start --driver=docker
```

Verify:

```bash
kubectl get nodes
```

---

## Step 2: Enable Ingress Controller

```bash
minikube addons enable ingress
```

Verify:

```bash
kubectl get pods -n ingress-nginx
```

Ensure `ingress-nginx-controller` is **Running**.

---

## Step 3: Application Deployments

### nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
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

Apply:

```bash
kubectl apply -f nginx-deployment.yaml
```

---

### httpd-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f httpd-deployment.yaml
```

Verify:

```bash
kubectl get pods
```

---

## Step 4: Create Services (Command Line)

### NGINX Service

```bash
kubectl expose deploy nginx-deployment \
  --name=nginx-service \
  --port=80 \
  --type=ClusterIP
```

### Apache Service

```bash
kubectl expose deploy httpd-deployment \
  --name=httpd-service \
  --port=80 \
  --type=ClusterIP
```

Verify:

```bash
kubectl get services
```

---

## Step 5: Ingress Configuration

### ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: <MINIKUBE-IP>.nip.io
    http:
      paths:
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
      - path: /httpd
        pathType: Prefix
        backend:
          service:
            name: httpd-service
            port:
              number: 80
```

Replace `<MINIKUBE-IP>` with your Minikube IP.

Get IP:

```bash
minikube ip
```

Apply Ingress:

```bash
kubectl apply -f ingress.yaml
```

Verify:

```bash
kubectl get ingress
kubectl describe ingress app-ingress
```

---

## Step 6: Access Applications

Example Minikube IP:

```text
192.168.49.2
```

### Open in Browser

* **NGINX**

```
http://192.168.49.2.nip.io/nginx
```

* **Apache**

```
http://192.168.49.2.nip.io/httpd
```

---

## Expected Output

* `/nginx` → NGINX welcome page
* `/httpd` → Apache HTTP Server page

Both applications are served **simultaneously** using a single Ingress endpoint.

---

## Validation Commands

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
kubectl get endpoints
```

---

## Cleanup (Optional)

```bash
kubectl delete ingress app-ingress
kubectl delete svc nginx-service httpd-service
kubectl delete deploy nginx-deployment httpd-deployment
minikube stop
```

---



---

## Author

This project is for academic and learning purposes only.
