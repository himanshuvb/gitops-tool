🚀 gitops-tool

This project demonstrates a simple GitOps workflow using Argo CD on Kubernetes.
An NGINX application is deployed automatically from a Git repository hosted on GitHub.

📦 Prerequisites
Kubernetes cluster (Minikube / VM / Cloud)
kubectl configured
Internet access
⚙️ Install Argo CD
1. Create namespace
kubectl create ns argocd
2. Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml --server-side
🔗 How Argo CD Connects to Your Deployment

There is no direct connection between your Deployment and Argo CD.

👉 The connection works like this:

You store Kubernetes YAMLs in GitHub
Argo CD reads that repo
It applies everything inside the defined path
📁 Repository Structure
gitops-tool/
│── deployment.yaml
│── service.yaml
│── application.yaml
📌 Argo CD Application (Important Part)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-custom-app
  namespace: argocd 
spec:
  project: default
  source:
    repoURL: 'https://github.com/himanshuvb/gitops-tool.git'
    targetRevision: main
    
    # 👇 THIS IS IMPORTANT
    path: '.'

  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
🧠 What path: '.' Means
. = current/root directory of the repo
Argo CD will read all YAML files from the root
In this case:
deployment.yaml
service.yaml

👉 If your files were inside a folder:

gitops-tool/
│── k8s/
    │── deployment.yaml
    │── service.yaml

Then you would use:

path: 'k8s'
🌐 Kubernetes Resources (Deployed by Argo CD)
Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-webserver
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
Service
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
▶️ Apply Argo CD Application
kubectl apply -f application.yaml
🔄 Full Flow
Push YAML files to GitHub
Argo CD reads repo (repoURL)
Looks inside path: '.'
Finds Deployment + Service
Applies them to Kubernetes
Keeps them in sync automatically
✅ Verify
kubectl get pods
kubectl get svc
💡 Key Takeaway
path: '.' → means read YAMLs from repo root
Argo CD deploys whatever exists in that path
Git = source of truth
Argo CD = automation
Kubernetes = runtime
