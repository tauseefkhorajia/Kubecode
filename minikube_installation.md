# Minikube Installation Guide (Ubuntu on AWS EC2)

This guide explains how to install Minikube on an Ubuntu-based AWS EC2 instance step-by-step.
Minikube helps you create a local Kubernetes cluster for learning, development, and testing.

## Prerequisites

Before you start, make sure you have:

 * Ubuntu EC2 instance


## Step 1: Update Your System

Keep your system packages up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```
---
## Step 2: Install Required Packages

Install the basic utilities required for installation:
```bash
sudo apt install -y curl wget apt-transport-https ca-certificates
```
---
## Step 3: Install Docker

Minikube uses Docker to create a local Kubernetes cluster.
Install and enable Docker:
```bash
sudo apt install -y docker.io
```
```bash
sudo systemctl enable --now docker
```
Add your user to the Docker group (so you can use Docker without sudo):
```bash
sudo usermod -aG docker $USER && newgrp docker
```
---
## Step 4: Install Minikube

Download the latest version of Minikube:
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
Make it executable and move it into your path:
```bash
chmod +x minikube
sudo mv minikube /usr/local/bin/
```


Verify installation:
```bash
minikube version
```
---
## Step 5: Install kubectl

kubectl is the command-line tool to interact with your Kubernetes cluster.

Download and install it using:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
Make it executable and move it into your path:
```bash
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
Check version:
```bash
kubectl version --client
```
---
## Step 6: Start Minikube

Start your Kubernetes cluster using Docker as the driver:
```bash
minikube start --driver=docker
```
This will download necessary components and set up a single-node cluster.

---
## Step 7: Verify the Cluster
```bash
minikube status
```

## View the Kubernetes nodes:
```bash
kubectl get nodes
```
If everything is correct, you should see your node in the “Ready” state.
