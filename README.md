# Deploying Resources to Multiple Kubernetes Clusters Using Argo CD (Hub-and-Spoke Model)

This guide demonstrates how to deploy resources to multiple AWS EKS clusters using Argo CD in a Hub-and-Spoke model.

## What is the Hub-and-Spoke Model?
The Hub-and-Spoke model is a centralized approach to managing resources across multiple Kubernetes clusters:
- The **Hub** is a central Argo CD instance that manages configurations and deployments across clusters.
- The **Spokes** are individual Kubernetes clusters where applications and workloads are deployed.

This model provides centralized control while enabling distributed deployments, making it ideal for multi-cluster setups.

---

## Step 1: Create EKS Clusters
Use `eksctl` to create the Hub and Spoke clusters.
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
---
## Step 3 : Run Argo CD in HTTP Mode
Edit the Argo CD configuration to enable HTTP mode.
```bash
kubectl edit  configmap  argocd-cmd-params-cm -n argocd

# Add this in the file
data:
  server.instance: "true"

```
---
## Step 4 : Change Argo CD Service Type to NodePort
Change the Argo CD service type for external access.

```bash
kubectl edit svc argocd-server -n argocd

# Modify the type to NodePort:
spec:
  type: NodePort
```
---
## Step 5 : Expose Argo CD UI
To access the Argo CD UI, perform the following:
  * Obtain the public IP of the EC2 instance running your Hub cluster.
  * Open port 30812 in the security group associated with your EC2 instance.
  * Access the Argo CD UI in your browser at <your-public-ip>:30812.
  * Retrieve the Argo CD admin password.
```bash

# get the password and login to argocd
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

```
---

## 6 . Add Clusters to Argo CD
Log in to the Argo CD server and register the Spoke clusters.

```bash
# Login to your hub cluster with username and password
argocd login <your ec2 instance public ip>:30812

# Add spoke-cluster1 
argocd cluster add <context of the spoke-cluster1> --server <your ec2 hub instance public ip>:30812

# Add spoke-cluster2
argocd cluster add <context of the spoke-cluster2> --server <your ec2 hub instance public ip>:30812

```
![Screenshot 2024-12-21 155115](https://github.com/user-attachments/assets/753906f5-47f9-407e-a3cc-760b9365ed91)

---

## 7. Create a application in ArgoCD UI
* Log in to the Argo CD UI using the admin credentials.
* Navigate to Applications and create an application for each Spoke cluster.
* Provide the repository details, paths, and target clusters for each application.
* Save the applications. Argo CD will monitor the manifests and deploy the resources automatically.

---
