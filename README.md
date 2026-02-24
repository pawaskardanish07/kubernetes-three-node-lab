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

<img width="1205" height="90" alt="Swap Status" src="https://github.com/user-attachments/assets/b5af56d3-617f-43df-bcad-0e0823477a73" />

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

<img width="1880" height="488" alt="Screenshot 2026-02-23 133616" src="https://github.com/user-attachments/assets/cf945c93-94c1-4199-9c59-38465040cc9c" />


Now *Edit the config.toml and Change: SystemdCgroup = true*

<img width="1881" height="982" alt="Chnage Config file" src="https://github.com/user-attachments/assets/a426036b-5226-4089-9ad1-441d1b3d1562" />


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

<img width="1887" height="70" alt="Screenshot 2026-02-23 134849" src="https://github.com/user-attachments/assets/cb9b7ea3-0b58-43ed-9681-e2225c5a733b" />

*kubectl version --client status:*

<img width="1388" height="73" alt="Screenshot 2026-02-23 134739" src="https://github.com/user-attachments/assets/bce87c73-cc00-454e-b936-0f0e9a6059e7" />

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

<img width="1889" height="464" alt="Screenshot 2026-02-23 135633" src="https://github.com/user-attachments/assets/beb2d568-b486-4a84-a6fb-fe039c6cbf5f" />

---

### Step 5 — Install Network Plugin (Master)
*Commands run only Master server*
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl get pods -n kube-system
```

<img width="1121" height="252" alt="Screenshot 2026-02-23 140336" src="https://github.com/user-attachments/assets/6d1ba30a-c434-4d3e-a0d9-f860ab3016c2" />

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

<img width="1082" height="112" alt="image" src="https://github.com/user-attachments/assets/d29f6c7e-d284-4d5c-949e-df25ef80589b" />

---

### Step 7 — Deploy Application on Plane Server
*Commands run only Master server*
```bash
kubectl create deployment nginx-test --image=nginx --replicas=3
kubectl get pods -o wide
```

<img width="1889" height="110" alt="Screenshot 2026-02-23 144429" src="https://github.com/user-attachments/assets/baaed750-4c39-4f37-8a55-fd7c3b871ce7" />

---

### Step 8 — Expose Application on Plane Server
*Commands run only Master server*
```bash
kubectl expose deployment nginx-test --type=NodePort --port=80
kubectl get svc
```

*Node Port:*

<img width="1721" height="134" alt="Screenshot 2026-02-23 144730" src="https://github.com/user-attachments/assets/5e79507f-11d6-4063-8074-59a0ed1df17f" />

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


Thank you!









