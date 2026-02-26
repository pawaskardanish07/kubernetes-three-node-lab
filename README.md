# Kubernetes Three-Node Lab Cluster
Hands-on Kubernetes cluster setup with one master and two worker nodes, Calico networking, and Nginx deployment accessible via IP and hostname
# Kubernetes Three-Node Lab Cluster
This repository documents the setup of a **Kubernetes cluster** with one control plane and two worker nodes on Ubuntu servers.  
It demonstrates container runtime installation, kubeadm initialization, Calico networking, and deployment of an Nginx application accessible via IP and hostname.
## 📂 Environment
There are Three Servers - Server 1 - Master Server, Server 2 - Worker Node, Server 3 - Worker Node
Server 1 (Master Server) Hostname: k8s-test1 IP: 192.168.10.1
Server 2 (Worker Server) Hostname: k8s-test2 IP: 192.168.10.20
Server 3 (Worker Server) Hostname: k8s-test3 IP: 192.168.10.30
## ⚙️ Setup Steps 
### Step 1 — Prepare All Servers
*Commands run on Master + Worker 1 + Worker 2:*
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```
*Swap Disabled Status:*

<img width="989" height="92" alt="image" src="https://github.com/user-attachments/assets/e241f406-1e73-43e1-a134-7a754ee76671" />


---

### Step 2 — Install Container Runtime
*Commands run on Master + Worker 1 + Worker 2:*
```bash
sudo apt update
```
```bash
sudo apt install -y containerd
```
```bash
sudo mkdir -p /etc/containerd
```
```bash
containerd config default | sudo tee /etc/containerd/config.toml
```
```bash
sudo systemctl status conatinerd
```

<img width="1847" height="489" alt="image" src="https://github.com/user-attachments/assets/7ba5ec8f-e9e4-4137-bb9a-1e0cb9036e4a" />



Now *Edit the config.toml and Change: SystemdCgroup = true*

<img width="1235" height="983" alt="image" src="https://github.com/user-attachments/assets/1bdebf58-a985-4bfc-bada-4233ff798148" />



After save this config file restart & enable containerd


```bash
sudo systemctl restart containerd
```
```bash
sudo systemctl enable containerd
```

---

### Step 3 — Install Kubernetes Packages
*Commands run on Master + Worker 1 + Worker 2:*
```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key \
| sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.asc] \
https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

*kubeadm version status:*

<img width="1885" height="67" alt="image" src="https://github.com/user-attachments/assets/739f5737-3991-4386-a56c-3959cc3e36f6" />


*kubectl version --client status:*

<img width="734" height="66" alt="image" src="https://github.com/user-attachments/assets/3c9afc0e-0275-4625-9281-10b6badc86a1" />


---

### Step 4 — Initialize Control Plane (Master)
*Commands run only Master server*
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

<img width="854" height="285" alt="image" src="https://github.com/user-attachments/assets/b1b6a707-b31b-4fb7-b152-30578e756411" />


---

### Step 5 — Install Network Plugin (Master)
*Commands run only Master server*
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl get pods -n kube-system
```

<img width="952" height="245" alt="image" src="https://github.com/user-attachments/assets/587d1f44-93b6-4a61-ac1a-b2a61b0289a1" />


---

### Step 6 — Join Worker Nodes
*get kubeadm joined command from Master server*

```bash
kubeadm token create --print-join-command
```

*After that run this command on Worker Server*

```bash
sudo kubeadm join 10.10.3.68:6443 --token <token> \ --discovery-token-ca-cert-hash sha256:<hash>
```

*After that verify on Master Server*
```bash
kubectl get nodes
```

<img width="690" height="110" alt="image" src="https://github.com/user-attachments/assets/87ad494b-4b8f-40fe-99aa-43bea946cacf" />


---

### Step 7 — Deploy Application on Plane Server
*Commands run only Master server*
```bash
kubectl create deployment nginx-test --image=nginx --replicas=3
kubectl get pods -o wide
```

<img width="956" height="202" alt="image" src="https://github.com/user-attachments/assets/919a0f27-48fd-4b5e-947b-cce1a70a1d76" />


---

### Step 8 — Expose Application on Plane Server
*Commands run only Master server*
```bash
kubectl expose deployment nginx-test --type=NodePort --port=80
kubectl get svc
```

*Node Port:*

<img width="945" height="126" alt="image" src="https://github.com/user-attachments/assets/79880faf-9236-49d7-bf53-a25e989ad75f" />


*URL Acesss:*
- http://192.168.1.10:32188/
- http://192.168.1.20:32188/
- http://192.168.1.30:32188/

<img width="1595" height="852" alt="Screenshot 2026-02-23 145010" src="https://github.com/user-attachments/assets/b79df422-0066-4d2c-9709-3ca63e6adb98" />

---

### Step 9 — Install Ingress Controller (Master Only) for URL access via hostname

*Before check controller pods*
```bash
kubectl get pods -n ingress-nginx
```

<img width="1176" height="66" alt="Screenshot 2026-02-23 150052" src="https://github.com/user-attachments/assets/abe54197-0590-4f7a-8ca3-2290325be79e" />

*After deploying Ingress controller*
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

<img width="1883" height="446" alt="Screenshot 2026-02-23 145943" src="https://github.com/user-attachments/assets/9983f282-baf5-47f4-86c8-20c79911697f" />

# Create Ingress Resource (Master Only)
*Edit Ignress.yaml config file*
```bash
sudo nano ingress.yaml
```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default
spec:
  rules:
  - host: nginx.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-test
            port:
              number: 80
```

*After save this file and apply ingress.yaml*
```bash
kubectl apply -f ingress.yaml
```

*Now check URL hostname*
```bash
kubectl get ingress
```

<img width="1046" height="134" alt="Screenshot 2026-02-23 150935" src="https://github.com/user-attachments/assets/e56b19d2-36b7-4c1c-8b12-463e75acaea0" />

*Now check without DNS server edit host file in windows or where you can try to access nginx*
*Add all server ip with ingress hostname*

<img width="1442" height="617" alt="Screenshot 2026-02-23 151142" src="https://github.com/user-attachments/assets/ccb44bb6-d333-4308-b1fa-5af363f332cc" />

*URL accessible via Hostname*

<img width="1594" height="848" alt="Screenshot 2026-02-23 151805" src="https://github.com/user-attachments/assets/5e3c62e7-00cd-4ba5-a36b-9f65921b67ca" />

---

## ✅ Outcome
- A fully functional Kubernetes cluster with networking and application deployment.
- Demonstrates NodePort and hostname access.
- Optional ingress routing for production-style traffic management.

---

## 🚀 Future Work
- Add TLS/HTTPS via Ingress.  
- Deploy multiple apps and route via different paths/hosts.  
- Automate setup with Ansible/Terraform.

---

## Capture Verification Outputs
*After each major step, run a verification command and screenshot it:*

    Step 1: free -h → shows swap disabled

    Step 2: systemctl status containerd → shows containerd running

    Step 3: kubectl version --client → shows kubectl installed

    Step 4: kubectl get nodes → shows master node

    Step 5: kubectl get pods -n kube-system → shows Calico pods

    Step 6: kubectl get nodes → shows all nodes Ready

    Step 7: kubectl get pods -o wide → shows nginx pods

    Step 8: Browser screenshot → Nginx accessible via IP/NodePort

    Step 9: Browser screenshot → Nginx accessible via hostname











