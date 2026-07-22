# Kubernetes (Kind) Lab Journey 🚀

Yeh repository meri Kubernetes learning journey ka ek step-by-step track hai, jahan maine Linux terminal par basic configurations se lekar Kubernetes Deployments aur Scaling tak ka safar taye kiya hai.

---

## 🛠️ Step 1: Fix Permissions & Group Setup
Shuruat mein literal `@USER` use karne se Linux error de raha tha, jise correct environment variable `$USER` se solve kiya gaya taake Docker bina `sudo` ke chal sake.
```bash
# Docker group mein user add karne ke liye
sudo usermod -aG docker \$USER && newgrp docker
```

---

## 🏗️ Step 2: Kind Cluster Setup (YAML Configuration)
**Kind (Kubernetes in Docker)** ko configuration file ke zariye up kiya, jis mein **1 Control-Plane** aur **2 Worker Nodes** setup kiye gaye.

`kind_config.yml` (Asli file ka naam typo free):
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
- role: worker
- role: worker
```

**Command to Create Cluster:**
```bash
kind create cluster --config kind_config.yml --name tws-kind-cluster
```

---

## 📁 Step 3: Working with Namespaces (NS)
Kubernetes resources ko isolate karne ke liye ek alag, saaf-suthra environment (Namespace) banana seekha.

`namespace.yml`:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tws-prod-env
```

**Commands:**
```bash
# Create Namespace using YAML
kubectl apply -f namespace.yaml

# Check All Namespaces
kubectl get ns

# Delete Namespace (Auto-deletes all pods inside it)
kubectl delete ns tws-prod-env
```

---

## 📦 Step 4: Deploying Pods (The Nginx Lab)
Nginx container ko ek specific namespace ke andar standalone pod ke roop mein chalaya.

`pod.yml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-prod-pod
  namespace: tws-prod-env
  labels:
    app: nginx-web
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

**Commands:**
```bash
# Apply Pod
kubectl apply -f pod.yml

# Get Pods in specific Namespace
kubectl get pods -n tws-prod-env

# Pod ke container ke andar terminal access (SSH alternative) karna:
kubectl exec -it nginx-prod-pod -n tws-prod-env -- /bin/bash
```

---

## 🚀 Step 5: Master Strategy - Deployments, Auto-Healing & Syntax Fixes
YAML files mein strict spacing aur structure seekha (jaise Deployment templates mein metadata ke andar `name:` allowed nahi hota). Deployment se hume **Auto-Healing** aur **High Availability** mili.

`deployment.yml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: tws-prod-env
  labels:
    app: nginx-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-web
  template:
    metadata:
      labels:
        app: nginx-web
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

**Commands & Status Checks:**
```bash
# Deploy the deployment
kubectl apply -f deployment.yml

# Check Overall Pods across ALL Namespaces
kubectl get pods -A

# Check Pods with Internal IPs and Node location
kubectl get pods -n tws-prod-env -o wide

# Troubleshooting & Detailed Logs/Events Check
kubectl describe pod <POD-NAME> -n tws-prod-env
```
* **Auto-Healing Test:** Jab maine `kubectl delete pod` se ek pod ko delete kiya, toh Deployment ne automatically ek naya pod khada kar diya!

---

## 📈 Step 6: Scaling the Application
Zaroorat ke mutabik application ka load handle karne ke liye pods ki sankhya (replicas) badhana ya kam karna seekha.

```bash
# Command Line Scaling (Tez tarika)
kubectl scale deployment nginx-deployment --replicas=5 -n tws-prod-env

# Ya phir deployment.yml mein `replicas: 5` badal kar run karein:
kubectl apply -f deployment.yml
```

---

## 🌍 Next Step (Future Lab)
- [ ] Application ko network layer de kar Service (NodePort/ClusterIP) ke zariye browser par expose karna.
