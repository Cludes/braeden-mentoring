---
layout: page
title: "Kubernetes Fundamentals"
week_number: "53"
release_date: "2027-04-20"
phase_label: "Phase 11"
phase_name: "Advanced DevOps"
phase_color: "#06b6d4"
permalink: /week-53/
---

> Week 53. Phase 11: Advanced DevOps.
> 
> Kubernetes is the industry standard for running containers at scale. If Docker is how you package an application, Kubernetes is how you run thousands of them reliably. Every major cloud provider has a managed Kubernetes service (EKS, GKE, AKS).
> 
> Install minikube (https://minikube.sigs.k8s.io) and kubectl before this session. Both are free.

---

# Kubernetes Fundamentals

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what Kubernetes is and why it exists
- Describe the core objects: Pod, Deployment, Service, Namespace
- Deploy and scale an application using kubectl
- Read and write basic Kubernetes manifest YAML
- Understand the control plane and worker node architecture

---

## What is Kubernetes?

Docker solves "how do I package and run one container?" Kubernetes solves "how do I run hundreds of containers reliably across many machines?"

Kubernetes (K8s) is an open source container orchestration platform originally built by Google. It handles:

- **Scheduling** - deciding which machine to run each container on
- **Self-healing** - restarting crashed containers automatically
- **Scaling** - adding or removing container replicas based on load
- **Networking** - connecting containers to each other and the outside world
- **Rolling updates** - deploying new versions without downtime

---

## Core Concepts

### Pod

The smallest deployable unit in Kubernetes. A Pod is one or more containers that share a network namespace and storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Pods are ephemeral. When a Pod dies it does not come back. Deployments manage Pods and ensure the right number are always running.

### Deployment

A Deployment manages a set of identical Pod replicas. It ensures the desired number are running and handles rolling updates.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Service

A Service gives a stable IP address and DNS name to a set of Pods. Without a Service, Pod IPs change every time a Pod restarts.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Service types:
- **ClusterIP** - internal only, accessible within the cluster
- **NodePort** - exposes the service on each node's IP at a static port
- **LoadBalancer** - provisions a cloud load balancer (AWS/GCP/Azure)

### Namespace

A way to divide a cluster into virtual sub-clusters. Different teams or environments (dev, staging, prod) typically get separate namespaces.

```bash
kubectl create namespace staging
kubectl get pods -n staging
```

---

## Architecture

```
Control Plane (manages the cluster)
├── API Server        <- everything talks to this
├── etcd              <- cluster state database
├── Scheduler         <- decides where to run pods
└── Controller Manager <- ensures desired state matches actual state

Worker Nodes (run your applications)
├── kubelet           <- agent that manages pods on this node
├── kube-proxy        <- handles networking
└── Container Runtime <- Docker or containerd
```

---

## Essential kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Pods
kubectl get pods
kubectl get pods -n kube-system          # system pods
kubectl describe pod my-pod              # detailed info + events
kubectl logs my-pod                      # container logs
kubectl exec -it my-pod -- /bin/bash     # shell into a running pod

# Deployments
kubectl get deployments
kubectl scale deployment nginx --replicas=5
kubectl rollout status deployment nginx
kubectl rollout undo deployment nginx    # roll back

# Apply and delete
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl delete pod my-pod

# Services
kubectl get services
kubectl port-forward service/nginx 8080:80   # local port forwarding
```

---

## Writing a Complete Application Manifest

A single YAML file can contain multiple resources separated by `---`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: app
        image: your-app:1.0
        ports:
        - containerPort: 5000
        env:
        - name: ENV
          value: production
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: production
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 5000
  type: LoadBalancer
```

---

## ConfigMaps and Secrets

Do not hardcode configuration in images. Use ConfigMaps for non-sensitive config and Secrets for sensitive values.

```bash
# Create a ConfigMap
kubectl create configmap app-config --from-literal=ENV=production --from-literal=LOG_LEVEL=info

# Create a Secret
kubectl create secret generic db-creds --from-literal=password=mysecret
```

Reference them in a Pod spec:

```yaml
env:
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: LOG_LEVEL
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-creds
      key: password
```

---

## Local Development with minikube

minikube runs a single-node Kubernetes cluster on your laptop.

```bash
minikube start
minikube status
minikube dashboard          # open the web UI
minikube service my-service # open a NodePort service in browser
eval $(minikube docker-env) # use minikube's Docker daemon
minikube stop
minikube delete
```

---

## Key Things to Remember

- A Pod is not a container - it wraps one or more containers
- Never manage Pods directly - use a Deployment
- Services are how Pods talk to each other and the outside world
- `kubectl apply` is idempotent - running it twice is safe
- Resource limits prevent one bad Pod from consuming all node resources

---

## Activity: Deploy an Application to Kubernetes

1. Start a local cluster: minikube start
1. Create a Deployment for nginx: kubectl create deployment nginx --image=nginx
1. Expose it as a Service: kubectl expose deployment nginx --port=80 --type=NodePort
1. Access it in your browser: minikube service nginx
1. Scale it to 3 replicas: kubectl scale deployment nginx --replicas=3 -- verify with kubectl get pods
1. Write a manifest YAML for a Deployment of your recon tool Docker image from Week 41. Apply it: kubectl apply -f recon-deployment.yaml
1. You know you are done when your recon tool is running as a Kubernetes Deployment and you can see the pods with kubectl get pods

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between a Pod, a Deployment, and a Service in Kubernetes?
2. What does kubectl apply -f do?
3. What is a Namespace in Kubernetes and why would you use one?
4. What is the difference between a NodePort and a ClusterIP service?
