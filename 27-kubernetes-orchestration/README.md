# ☸️ Chapter 27: Kubernetes & Orchestration

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-27%20of%2034-blue?style=for-the-badge" alt="Chapter 27">
</p>

---

## 📑 Table of Contents

- [What is Kubernetes?](#what-is-kubernetes)
- [Architecture](#architecture)
- [Setting Up (minikube)](#setting-up-minikube)
- [Core Concepts](#core-concepts)
- [kubectl — The CLI](#kubectl--the-cli)
- [Deployments](#deployments)
- [Services](#services)
- [ConfigMaps & Secrets](#configmaps--secrets)
- [Persistent Storage](#persistent-storage)
- [Helm — Package Manager](#helm--package-manager)
- [Practice Exercises](#-practice-exercises)

---

## What is Kubernetes?

Kubernetes (K8s) automates container deployment, scaling, and management.

---

## Architecture

```
┌────────────────────────────────────────────────────┐
│                  CONTROL PLANE                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────┐│
│  │ API      │ │ etcd     │ │ Controller Manager    ││
│  │ Server   │ │ (state)  │ │ + Scheduler           ││
│  └──────────┘ └──────────┘ └──────────────────────┘│
└────────────────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
┌───────▼──────┐ ┌────▼───────┐ ┌───▼──────────┐
│   Node 1     │ │   Node 2   │ │   Node 3     │
│ ┌──────────┐ │ │ ┌────────┐ │ │ ┌──────────┐ │
│ │ kubelet  │ │ │ │kubelet │ │ │ │ kubelet  │ │
│ │ kube-    │ │ │ │kube-   │ │ │ │ kube-    │ │
│ │ proxy    │ │ │ │proxy   │ │ │ │ proxy    │ │
│ ├──────────┤ │ │ ├────────┤ │ │ ├──────────┤ │
│ │ Pod  Pod │ │ │ │Pod Pod │ │ │ │ Pod  Pod │ │
│ └──────────┘ │ │ └────────┘ │ │ └──────────┘ │
└──────────────┘ └────────────┘ └──────────────┘
```

---

## Setting Up (minikube)

```bash
# Install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl

# Start cluster
minikube start
kubectl cluster-info
kubectl get nodes
```

---

## Core Concepts

| Resource | Description |
|----------|-------------|
| **Pod** | Smallest unit — one or more containers |
| **Deployment** | Manages pod replicas and rollouts |
| **Service** | Stable networking endpoint for pods |
| **ConfigMap** | External configuration data |
| **Secret** | Sensitive data (base64 encoded) |
| **Namespace** | Virtual cluster isolation |
| **Ingress** | HTTP routing / load balancing |
| **PV/PVC** | Persistent storage |

---

## kubectl — The CLI

```bash
# Get resources
kubectl get pods                      # List pods
kubectl get pods -o wide              # With more info
kubectl get services                  # List services
kubectl get all                       # Everything
kubectl get all -n kube-system        # In a namespace

# Describe (detailed info)
kubectl describe pod <name>
kubectl describe service <name>

# Logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>            # Follow
kubectl logs <pod-name> -c <container>  # Specific container

# Exec into pod
kubectl exec -it <pod-name> -- bash

# Apply manifest
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml

# Shortcuts
kubectl get po                        # pods
kubectl get svc                       # services
kubectl get deploy                    # deployments
kubectl get ns                        # namespaces
```

---

## Deployments

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 5
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl rollout status deployment/web-app
kubectl scale deployment web-app --replicas=5
kubectl rollout undo deployment/web-app       # Rollback
```

---

## Services

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP                # Internal only
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-external
spec:
  type: NodePort                 # Accessible externally
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080              # Access via node:30080
---
apiVersion: v1
kind: Service
metadata:
  name: web-lb
spec:
  type: LoadBalancer             # Cloud load balancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

---

## ConfigMaps & Secrets

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  DB_HOST: postgres-service
  LOG_LEVEL: info
---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  DB_PASSWORD: supersecret
  API_KEY: my-api-key-123
```

```yaml
# Use in deployment
spec:
  containers:
  - name: app
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secrets
```

---

## Persistent Storage

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
# Use in pod
spec:
  containers:
  - name: db
    volumeMounts:
    - mountPath: /var/lib/postgresql/data
      name: db-data
  volumes:
  - name: db-data
    persistentVolumeClaim:
      claimName: db-storage
```

---

## Helm — Package Manager

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search
helm search repo nginx

# Install
helm install my-nginx bitnami/nginx
helm install my-db bitnami/postgresql --set auth.postgresPassword=secret

# List releases
helm list

# Upgrade
helm upgrade my-nginx bitnami/nginx --set replicaCount=3

# Uninstall
helm uninstall my-nginx
```

---

## 🏋️ Practice Exercises

1. **Setup**: Install minikube and start a local cluster
2. **Deploy**: Create a deployment with 3 Nginx replicas
3. **Service**: Expose the deployment with a NodePort service
4. **Scale**: Scale up to 5 replicas and back to 2
5. **ConfigMap**: Create a ConfigMap and use it in a pod
6. **Logs**: View logs from a specific pod
7. **Rollout**: Deploy a new image version and then rollback
8. **Helm**: Install a chart from the Bitnami repository

---

<p align="center">
  <a href="../26-docker-containers/README.md">← Previous: Docker & Containers</a> · <a href="../README.md">🏠 Home</a> · <a href="../28-advanced-filesystems/README.md">Next: Advanced Filesystems →</a>
</p>
