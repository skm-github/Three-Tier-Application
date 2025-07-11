# Three-Tier Covid bed slot Booking Application - CI/CD with Azure DevOps, ACR, ArgoCD, and AKS

This project demonstrates a complete CI/CD workflow for a three-tier application built using:

* Frontend: HTML/CSS/JS
* Backend: Python Flask
* Database: MySQL

The infrastructure is hosted on **Azure Kubernetes Service (AKS)** with CI via **Azure DevOps** and CD via **ArgoCD**.

---

## 🧱 Architecture Overview

![Copy of Watch for changes](https://github.com/user-attachments/assets/c09a8bc7-2780-4257-bc96-d9eff02def7b)


---

## 🛠 CI/CD Workflow Summary

1. **Code is pushed to Azure Git Repo**
2. **Azure Pipeline** (self-hosted agent) builds and pushes the image to ACR
3. Shell script updates K8s manifests with the new image tag
4. ArgoCD watches for changes and syncs manifests to AKS
5. Application is exposed via **NodePort** (no LoadBalancer used)

---

## 🧩 Prerequisites

* Azure DevOps project
* Azure Kubernetes Service (AKS) cluster
* Azure Container Registry (ACR)
* ArgoCD installed in AKS
* GitHub repo: [https://github.com/skm-github/Three-Tier-Application](https://github.com/akshayachar03/three-tier-slot-booking-app)

---

## 📦 Step-by-Step Setup

### 1. 📂 Import GitHub Repo to Azure DevOps

* Go to Azure DevOps > Repos
* Click `Import` and provide the GitHub repo URL

### 2. 🐳 Create Azure Container Registry (ACR)

```bash
az acr create --name covidcicd --resource-group myRG --sku Basic
```

### 3. ☸️ Create AKS Cluster and Connect to ACR

```bash
az aks create \
  --resource-group myRG \
  --name myAKS \
  --node-count 1 \
  --enable-addons monitoring \
  --generate-ssh-keys \
  --attach-acr covidcicd
```

### 4. 🔐 Create Image Pull Secret (Optional if AKS is linked to ACR)

```bash
kubectl create secret docker-registry acr-auth \
  --docker-server=covidcicd.azurecr.io \
  --docker-username=<acr-username> \
  --docker-password=<acr-password> \
  --namespace default
```

---

## 🐙 Set Up ArgoCD (CD)

### 1. Install ArgoCD in AKS

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Patch ArgoCD Server for External Access (NodePort)

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc -n argocd
```

### 3. Access ArgoCD UI

To get the external IP of the AKS agent node (VMSS):

```bash
kubectl get nodes -o wide
```

Visit: `http://<AKS-VMSS-Public-IP>:Port`

Get initial password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

### 4. Prepare Kubernetes Manifests

In your GitHub/Azure DevOps repo, create a folder named `k8s-manifests/` and add the following files:

* `backend-deployment.yaml`: for Flask app deployment
* (Optionally) `mysql-deployment.yaml` and `mysql-secret.yaml`

Ensure these files reference the correct container image and labels.

### 5. Connect ArgoCD to Azure DevOps Repo and Deploy via UI

1. Open the ArgoCD UI in browser.
2. Login using the `admin` credentials.
3. Click on **"New App"**.
4. Fill the form:

   * **Application Name**: `three-tier-app`
   * **Project**: `default`
   * **Sync Policy**: `Automatic`
   * **Repository URL**: Azure DevOps Git repo (requires Personal Access Token for private repo)
   * **Revision**: `HEAD`
   * **Path**: `k8s-manifests`
   * **Cluster URL**: `https://kubernetes.default.svc`
   * **Namespace**: `default`
5. Click **Create**.

ArgoCD will now monitor your repo and sync the Kubernetes manifests into the AKS cluster.

---

To get the external IP of the AKS agent node (VMSS):

```bash
kubectl get nodes -o wide
```

Then from your browser:

```
http://<AKS-VMSS-Public-IP>:Port
```

---

## ✅ Done!

You now have:

* Git-pushed CI triggering image builds
* Updated Kubernetes manifests in your repo
* ArgoCD syncing those manifests into AKS
* App served to users using NodePort access 🎉

---

## 📎 Repo Link

[https://github.com/skm-github/Three-Tier-Application](https://github.com/skm-github/Three-Tier-Application)
