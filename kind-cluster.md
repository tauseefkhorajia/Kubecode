# KIND (Kubernetes IN Docker) Installation Guide (Ubuntu on AWS EC2)

## Prerequisites:
* Ubuntu
* take minimum t2.medium

## step 1: Install Docker
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo \"${UBUNTU_CODENAME:-$VERSION_CODENAME}\") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```
Add current user to Docker group:
```bash
sudo usermod -aG docker $USER && newgrp docker
```
---
## Step 2: Install kubectl

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
## step 3: Install kind
Download the latest version of KIND binary:
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-linux-amd64

chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

kind version
```
---
## Step 4: setup kind Cluster
Create a kind-config.yml file:
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.31.2
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
```
Create the cluster using the kind-config.yml  file:
```bash
kind create cluster --name=demo-kind-cluster --config=kind-config.yml
```
Verify the Cluster:
```bash
kubectl cluster-info --context demo-kind-cluster
```
List all nodes:
```bash
kubectl get nodes
```
You should see one control-plane node in the Ready state.











