# 🚀 Hello World Kubernetes Deployment + GitOps with Argo CD

This guide covers the full process of deploying a simple Hello World app using NGINX in a local Kubernetes cluster (kind) and optionally managing it with Argo CD for GitOps.

---

## 📦 Project Structure

```
hello-world-k8s/
│
├── app/
│   └── index.html
│
├── docker/
│   └── Dockerfile
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
└── README.md
```

---

## ⚙️ 1. Prerequisites

- Docker
- kind (Kubernetes IN Docker)
- kubectl
- Argo CD (optional, for GitOps)

---

## 🐳 2. Docker Image Creation

### Dockerfile (inside `docker/` folder)
```Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

### Build Docker Image
```bash
docker build -t hello-world-nginx docker/
```

---

## ⛴️ 3. Create Kubernetes Cluster with Kind

### Create a Cluster
```bash
kind create cluster --name hello-world-cluster
```

### Verify
```bash
kubectl get nodes
```

---

## 📦 4. Load Docker Image into Kind Cluster

```bash
kind load docker-image hello-world-nginx --name hello-world-cluster
```

---

## 📄 5. Kubernetes Deployment and Service

### Deployment YAML (`k8s/deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world-container
        image: hello-world-nginx
        ports:
        - containerPort: 80
```

### Service YAML (`k8s/service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
spec:
  type: NodePort
  selector:
    app: hello-world
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

---

## 🚀 6. Deploy to Kubernetes

Apply the YAMLs:
```bash
kubectl apply -f k8s/
```

Check pods and services:
```bash
kubectl get pods
kubectl get svc
```

---

## 🌐 7. Access the Hello World App

### Option 1: NodePort

Find NodePort:
```bash
kubectl get svc
```
Access the app:
```
http://localhost:30080
```

> ⚠️ If `curl http://localhost:30080` fails, proceed to option 2.

---

### Option 2: Port Forwarding

Use port-forward if NodePort access doesn't work:
```bash
kubectl port-forward svc/hello-world-service 8081:80
```
Then access:
```
http://localhost:8081
```

---

## 🚀 8. (Optional) Install and Configure Argo CD

### Install Argo CD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

### Expose Argo CD Server

Patch the service to NodePort:
```bash
kubectl patch svc argocd-server -n argocd --type merge -p '{"spec": {"type": "NodePort"}}'
```

Get Argo CD Server details:
```bash
kubectl get svc -n argocd
```
Find the NodePort (example: `30496`).

Access the Argo CD UI:
```
http://localhost:<NodePort>
```

---

### Get Argo CD Admin Password

Retrieve the initial admin password:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

Login:
- **Username**: `admin`
- **Password**: (password from above)

---

## ⚙️ 9. (Optional) Setup GitOps with Argo CD

Create a GitHub repository for your YAMLs (`deployment.yaml`, `service.yaml`).

Inside Argo CD UI:
- Create a new **Application**
- Set source repo to your GitHub URL
- Set destination to your cluster
- Sync it.

---

# ✅ Final Summary

You have:
- Deployed a simple app inside Kubernetes
- (Optionally) Setup GitOps with Argo CD
- Fully running on your local machine!

---

## 🛠️ Troubleshooting

| Problem | Solution |
| :------ | :------- |
| `curl localhost:30080` not working | Use `kubectl port-forward` |
| Argo CD login fails | Use decoded initial password |
| ImagePullBackOff error | Ensure image is loaded to kind |
| Ports already in use | Change `port-forward` ports |

---

# 🎉 Congratulations!

You successfully completed the assignment! 🚀

