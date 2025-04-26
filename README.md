# Hello World Kubernetes Deployment with Argo CD (Local Setup)

## Objective

Deploy a simple "Hello World" HTML page using NGINX on a local Kubernetes cluster (Kind) with optional GitOps integration using Argo CD.

## Project Structure

hello-world-k8s/│├── app/│   └── index.html         # Simple Hello World HTML│├── docker/│   └── Dockerfile         # Dockerfile to build the NGINX image│├── k8s/│   ├── deployment.yaml    # Kubernetes Deployment manifest│   └── service.yaml       # Kubernetes Service manifest (NodePort)│├── demo/│   └── video.mp4          # (Optional: Placeholder for a demo video)│└── README.md              # This file
## 1. Prerequisites

Before you begin, ensure you have the following tools installed:

* **Docker:** [Install Docker](https://docs.docker.com/get-docker/)
* **Kind (Kubernetes IN Docker):** [Install Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
* **kubectl:** [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* **(Optional) Argo CD CLI:** [Install Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/) (Needed if you want to manage Argo CD via CLI)

## 2. Build and Load Docker Image

This step builds the NGINX Docker image containing your `index.html` and loads it into your Kind cluster's node registry.

1.  **Build the Docker image:**
    Navigate to the project's root directory in your terminal.

    ```bash
    docker build -t hello-world-nginx:latest -f docker/Dockerfile app/
    ```
    * `-t hello-world-nginx:latest`: Tags the image as `hello-world-nginx` with the `latest` tag.
    * `-f docker/Dockerfile`: Specifies the path to the Dockerfile.
    * `app/`: Specifies the build context (where `index.html` is located).

2.  **Load the image into the Kind cluster:**
    *(Ensure your Kind cluster from Step 3 is running)*

    ```bash
    kind load docker-image hello-world-nginx:latest --name hello-world-cluster
    ```
    * `--name hello-world-cluster`: Specifies the name of your Kind cluster.

## 3. Local Kubernetes Cluster Setup (Kind)

Create a local Kubernetes cluster using Kind.

1.  **Create the cluster:**

    ```bash
    kind create cluster --name hello-world-cluster
    ```

2.  **Verify cluster nodes:** (Optional)

    ```bash
    kubectl get nodes
    ```
    You should see the control-plane node for your `hello-world-cluster`.

## 4. Deploy Kubernetes Manifests

Apply the Kubernetes Deployment and Service manifests to deploy the application onto your Kind cluster.

```bash
kubectl apply -f k8s/
This command will create:A Deployment named hello-world-deployment managing the NGINX pods.A Service named hello-world-service of type NodePort to expose the Deployment.5. Access the ApplicationThere are two primary ways to access the deployed "Hello World" application:Option 1: Using NodePortFind the NodePort: Get the details of the hello-world-service.kubectl get svc hello-world-service
Look for the port mapping under the PORT(S) column (e.g., 80:30001/TCP). The port after the colon (30001 in this example) is the NodePort.Access in Browser: Open your web browser and navigate to:http://localhost:<NodePort>
(Replace <NodePort> with the actual port number you found). You should see your "Hello World!" page.Option 2: Using Port-ForwardingIf accessing via NodePort doesn't work (e.g., due to firewall restrictions or Docker Desktop networking), use kubectl port-forward.Forward the port:kubectl port-forward svc/hello-world-service 8081:80
This command forwards local port 8081 to port 80 on the service inside the cluster. Keep this terminal window open.Access in Browser: Open your web browser and navigate to:http://localhost:8081
You should see your "Hello World!" page.6. Argo CD Installation & Setup (Optional GitOps)Follow these steps to install Argo CD into your cluster and configure it to manage the "Hello World" application (assuming your manifests are in a Git repository).Install Argo CD:# Create namespace for Argo CD
kubectl create namespace argocd

# Apply Argo CD manifests
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
Expose Argo CD Server UI (using NodePort):kubectl patch svc argocd-server -n argocd --type merge -p '{"spec": {"type": "NodePort"}}'
Find the Argo CD Server NodePort:kubectl get svc argocd-server -n argocd
Look for the port mapping for https/http (e.g., 443:30002/TCP). The port after the colon (30002 in this example) is the NodePort for the UI.Access Argo CD UI: Open your browser and navigate to:https://localhost:<ArgoCD NodePort>
(Replace <ArgoCD NodePort> with the actual port number). You might get a certificate warning; proceed if you trust the source.Get Initial Admin Password:kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode; echo
Copy the displayed password.Login to Argo CD:Username: adminPassword: (The password you retrieved in the previous step)Configure Argo CD Application:Inside the Argo CD UI, create a new application.Configure it to point to your Git repository where the k8s/ manifests are stored.Set the path to k8s/.Set the destination cluster URL to https://kubernetes.default.svc.Set the destination namespace to default (or wherever you want to deploy).Once created, Argo CD will automatically sync the manifests from your Git repository to the cluster, deploying or updating the "Hello World" application.CleanupTo remove the resources created:Delete Argo CD Application (if configured): Use the Argo CD UI or CLI.Uninstall Argo CD (if installed):kubectl delete -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
kubectl delete namespace argocd
Delete Kubernetes resources:kubectl delete -f k8s/
Delete the Kind cluster:kind delete cluster --name hello-world-cluster
Remove the Docker image (Optional):docker rmi hello-world-nginx:latest
