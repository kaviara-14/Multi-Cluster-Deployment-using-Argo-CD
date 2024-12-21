# Deploying Resources to Multiple Kubernetes Clusters using Argo CD (Hub-and-Spoke Model)

This guide demonstrates how to deploy resources to multiple AWS EKS clusters using Argo CD in a Hub-and-Spoke model.

## What is the Hub-and-Spoke Model?
The Hub-and-Spoke model is a centralized approach to managing resources across multiple Kubernetes clusters:
- The **Hub** is a central Argo CD instance that manages configurations and deployments across clusters.
- The **Spokes** are individual Kubernetes clusters where applications and workloads are deployed.

This model provides centralized control while enabling distributed deployments, making it ideal for multi-cluster setups.

---

## Step 1: Create EKS Clusters

```bash
# Hub Cluster
eksctl create cluster --name hub-cluster --region <region> --nodes 3 --node-type t3.medium

# Spoke Clusters
eksctl create cluster --name spoke-cluster1 --region <region> --nodes 2 --node-type t3.medium
eksctl create cluster --name spoke-cluster2 --region <region> --nodes 2 --node-type t3.medium
```

---

## Step 2: Install Argo CD on the Hub Cluster

```bash
# Switch to Hub Cluster
kubectl config use-context < Hub Cluster >

# Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```
## Step 3 : Run Argo CD in HTTP Mode(Insecure)
```bash
kubectl edit  configmap  argocd-cmd-params-cm -n argocd

# Add this in the file
data:
  server.instance: "true"

```

## Step 4 : Change the type to NodePort

```bash
kubectl edit svc argocd-server -n argocd  # change the type to NodePort
```
## Step 5 : Expose Argo CD UI

Go to hub your ec2 instance, Copy your ec2 instance public Ip and open port for 30812 and paste it in the browser <your public ip>: 30812

```bash

# get the password and login to argocd
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

```

## 6 . Add Cluster

```bash
# Login to your hub cluster with username and password
argocd login <your ec2 instance public ip>:30812

# Add spoke-cluster1 
argocd cluster add <context of the spoke-cluster1> --server <your ec2 hub instance public ip>:30812

# Add spoke-cluster2
argocd cluster add <context of the spoke-cluster2> --server <your ec2 hub instance public ip>:30812

```

## Create a application in ArgoCD UI
* Create a application for spoke-cluster1 and spoke-cluster2 with all the repo details and path and click save. Now this will look for any changes happening in manifest file and it will create kubernetes for us.
