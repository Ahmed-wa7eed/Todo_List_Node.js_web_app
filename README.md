# Todo_List_Node.js_web_app

## 🚀 Overview
This project is a **Todo List Node.js web app** that is containerized, deployed with Docker and Kubernetes, and uses CI/CD via **GitHub Actions** and **ArgoCD**.

---

## 📄 Initial Setup and Assumptions

### MongoDB Atlas Setup:
- Created an account on **MongoDB Atlas**.
- Created a **free cluster**.
- Added my **IP address** to the access list.
- Created a **database user** with proper permissions.
- Obtained a **connection URI** and used it in the `.env` file.

### DockerHub Setup:
- Created a **DockerHub account**.
- Created a repository named **fortstack**.
- Generated **DockerHub credentials** and added them to **GitHub Secrets**.

### GitHub Setup:
- Created a **GitHub repository** for the project.
- Pushed all **application code**, `Dockerfile`, **GitHub Actions workflow**, and **Kubernetes manifests**.

### Kind Kubernetes Cluster:
- Used **Kind** to create a local Kubernetes cluster.
- Verified cluster with `kubectl get nodes` and `kubectl cluster-info`.

### ArgoCD Installation:
- Installed **ArgoCD** in the Kind cluster.
- Accessed ArgoCD dashboard via **port-forwarding**.

### VM Configuration using Ansible:
- Created a **Linux VM** (locally or on cloud).
- Used **Ansible** to install Docker on the VM.

---

## ✅ Part 1: Clone and Setup Project

### Clone Repo:
```bash
git clone https://github.com/Ahmed-wa7eed/Fort_Stack-task.git

```
### Install Dependencies:
```
cd Fort_Stack-task
npm install
```
### Create .env file:
```
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/todo?retryWrites=true&w=majority&appName=YourClusterName
```
### Test app locally:
```
npm start
Visit: http://localhost:4000
```
## 🧰 Ansible Playbook for Docker Installation
```
- name: Configuring Docker
  hosts: ip

  tasks:

    - name: Adding Docker repository
      yum_repository:
        name: Docker
        description: Docker Repo
        baseurl: https://download.docker.com/linux/centos/9/x86_64/stable
        gpgcheck: yes
        gpgkey: https://download.docker.com/linux/centos/gpg

    - name: Installing Docker package
      package:
        name: docker-ce
        state: latest

    - name: Starting docker service
      service:
        name: docker
        state: started
        enabled: yes

    - name: Installing pip3 for Python 3
      package:
        name: python3-pip
        state: present

    - name: Installing Docker SDK for python3
      command: pip3 install docker

    - name: Add user 'ahmed' to docker group
      user:
        name: ahmed
        groups: docker
        append: yes
```
![image alt](https://github.com/Ahmed-wa7eed/Fort_Stack-task/blob/master/Screenshot%202025-07-31%20132826.png?raw=true)

## 🐳 Dockerize the Application
```
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 4000
CMD ["npm", "start"]
```
### Build Image Locally:
```
docker build -t yourdockerhub/fortstack:latest .
```
## ⚙️ GitHub Actions (CI) to push image automatic
```
name: CI Workflow
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Image
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: yourdockerhub/fortstack:latest
```
## 🚀 Kubernetes (CD with ArgoCD)
### Install ArgoCD on Kind
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### Access ArgoCD Dashboard:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
Visit: https://localhost:8080
```
![image alt](https://github.com/Ahmed-wa7eed/Fort_Stack-task/blob/master/argo%20image.png?raw=true)
### Get initial password:
```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode && echo
```
### Deploy App via ArgoCD:
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: fortstack-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<your-username>/Fort_Stack-task.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```
###  Apply: 
```
kubectl apply -f k8s/argocd-app.yaml -n argocd
```
![image alt](https://github.com/Ahmed-wa7eed/Fort_Stack-task/blob/master/cd%20image.png?raw=true)





