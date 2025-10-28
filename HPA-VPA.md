# Kubernetes HPA and VPA Setup on Minikube or KIND



## 🧰 Before You Begin

Make sure you have:

- One virtual machine (for example, AWS EC2 `t2.medium`, 2 CPU, 4 GB RAM)
- **Minikube** or **KIND** cluster installed and running
- **kubectl** command-line tool configured

---

## ⚙️ Step 1: Enable Metrics Server

Metrics Server collects CPU / memory data used by HPA and VPA.

### ▶️ For Minikube
```bash
minikube addons enable metrics-server
```

### For KIND
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
edit the Metrics Server deployment:
```bash
kubectl -n kube-system edit deployment metrics-server
```
Add the following under container.args:
```bash
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,ExternalIP
```
Restart Metrics Server:
```bash
kubectl -n kube-system rollout restart deployment metrics-server
```
Check that it’s working:
```bash
kubectl get pods -n kube-system
kubectl top nodes
```
---

## Step 2: Create nginx Deployment and Service

We’ll deploy a simple nginx web server with CPU requests / limits.
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
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
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
Apply the deployment and service:
```bash
kubectl apply -f nginx-deployment.yaml
```
## Step 3: Create Horizontal Pod Autoscaler (HPA)

HPA will automatically add or remove pods depending on CPU utilization.
```bash
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 20
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 50
```
Apply the HPA:
```bash
kubectl apply -f nginx-hpa.yaml
```
---
## Step 4: Access the nginx Service

Forward the service port to access nginx from your browser:
```bash
kubectl port-forward svc/nginx-service 8081:80 --address 0.0.0.0 &
```
Open your browser:
```bash
http://<your-vm-public-ip>:8081
```
##Step 5: Test Autoscaling (Load Generation)

Start a BusyBox container to generate CPU load:
```bash
kubectl run -i --tty load-generator --image=busybox --restart=Never -- /bin/sh
```
Inside the container, run:
```bash
while true; do wget -q -O- http://65.2.39.125:80; done
```
In another terminal, watch the HPA activity:
```bash
kubectl get hpa -w
```
---


# Install Vertical Pod Autoscaler (VPA)

VPA changes pod resource limits automatically based on usage.

Clone the autoscaler repository:
```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
```bash
./hack/vpa-up.sh
```
Verify VPA pods:
```bash
kubectl get pods -n kube-system | grep vpa
```
Check cluster resource usage:
```bash
kubectl top pods
kubectl top nodes
```

### You have successfully set up HPA and VPA on your Minikube / KIND cluster!













