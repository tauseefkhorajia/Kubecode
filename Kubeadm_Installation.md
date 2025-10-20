# Kubeadm Installation Guide (Ubuntu on AWS EC2)

This guide will help you install Kubernetes on Ubuntu EC2 instances using kubeadm.
It includes steps for both master and worker nodes.

## Prerequisites:
Ubuntu
take minimum t2.medium

# Execute on Both Master and Worker

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
### CRI-DockerD Installation
```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.17/cri-dockerd_0.3.17.3-0.ubuntu-jammy_amd64.deb

sudo apt install -y ./cri-dockerd_0.3.17.3-0.ubuntu-jammy_amd64.deb

sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now cri-docker.service
sudo systemctl enable cri-docker.socket
```
---
## step 2: Prerequisites for Kubernetes

Turn off swap:
```bash
 sudo swapoff -a
```
Load kernel modules for Kubernetes networking:
```bash
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/k8s.conf
sudo modprobe overlay
sudo modprobe br_netfilter
```
Set system networking parameters (bridge traffic and IP forwarding):
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
lsmod | grep br_netfilter
lsmod | grep overlay
```
---
## step 3: Install Kubernetes
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Install Kubernetes components:
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```
Hold versions to prevent unintended upgrades:
```bash
sudo apt-mark hold kubelet kubeadm kubectl
```
Enable kubelet service:
```bash
sudo systemctl enable --now kubelet
```

# Execute on only Master
start master node:
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
```
Setup kubectl for your user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Install Flannel network plugin:
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
Get worker join command:
```bash
kubeadm token create --print-join-command
```
# Execute on only worker node
```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash <HASH> --cri-socket=unix:///var/run/cri-dockerd.sock
```
