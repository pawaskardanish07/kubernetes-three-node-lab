# Kubernetes Three-Node Lab Cluster

## 🎯 Learning Objective
This repository captures my **hands-on lab practice** in building a Kubernetes cluster from scratch.  
The goal was to understand how to prepare servers, install Kubernetes components, configure networking, and deploy applications accessible via IP and hostname.

## 🖥️ Lab Environment
I performed this setup in my personal lab using three Ubuntu 22.04 servers:

- **Server 1 (Master)**  
  Hostname: k8s-test1  
  IP: 192.168.10.1  

- **Server 2 (Worker Node)**  
  Hostname: k8s-test2  
  IP: 192.168.10.20  

- **Server 3 (Worker Node)**  
  Hostname: k8s-test3  
  IP: 192.168.10.30  

Networking was provided by **Calico**, and the application deployed was **Nginx**.

## ⚙️ Lab Exercises
 
### Lab Exercise 1 — Preparing All Servers

**Commands run on all nodes:**
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

**Verification Command:**

```bash
free -h
```

**Output (snippet):**

<img width="989" height="92" alt="image" src="https://github.com/user-attachments/assets/e241f406-1e73-43e1-a134-7a754ee76671" />


**Explaination:**

The absence of a Swap line confirms swap is disabled on all nodes.
This is required for Kubernetes scheduling stability and ensures pods run reliably.

---

### Lab Exercise 2 — Install Container Runtime
**Commands run on all nodes:**
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

**Verification Command:**
```bash
systemctl status containerd
```

**Output (snippet):**

<img width="1847" height="489" alt="image" src="https://github.com/user-attachments/assets/7ba5ec8f-e9e4-4137-bb9a-1e0cb9036e4a" />

**Explaination:**

Containerd was installed and configured to use systemd as the cgroup driver, ensuring compatibility with kubelet.

Now *Edit the config.toml and Change: SystemdCgroup = true*

**Command:**
```bash
sudo nano config.taml
```

<img width="1235" height="983" alt="image" src="https://github.com/user-attachments/assets/1bdebf58-a985-4bfc-bada-4233ff798148" />



After save this config file restart & enable containerd


```bash
sudo systemctl restart containerd
```
```bash
sudo systemctl enable containerd
```

---

### Lab Exercise 3 — Install Kubernetes Packages
**Commands run on all nodes:**
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

**Verification Command:**
```bash
kubectl version --client
```

**Output (snippet)**

<img width="734" height="66" alt="image" src="https://github.com/user-attachments/assets/3c9afc0e-0275-4625-9281-10b6badc86a1" />

**Explanation:**
Kubernetes components were installed and held at version 1.28 to prevent accidental upgrades.

---

###Lab Exercise 4 — Initialize Control Plane (Master Only)
**Command:**
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

**Post-Init Setup:**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Verification Command:**
```bash
kubectl get nodes
```

**Output (snippet):**

<img width="854" height="285" alt="image" src="https://github.com/user-attachments/assets/b1b6a707-b31b-4fb7-b152-30578e756411" />

**Explanation:**

Control plane initialized with pod CIDR 192.168.0.0/16. kubeconfig configured for kubectl access.

---

### Lab Exercise 5 — Install Calico Network Plugin
**Commands*
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl get pods -n kube-system
```

**Verification Command:**
```bash
kubectl get pods -n kube-system
```

**Output (snippet):**

<img width="940" height="246" alt="image" src="https://github.com/user-attachments/assets/01d4e6fe-545c-4c8c-93a3-2748686355df" />


**Explanation:**

Calico was deployed to provide pod networking. All pods transitioned to Running state.

---

### Lab Exercise 6 — Join Worker Nodes
**On Master server**

```bash
kubeadm token create --print-join-command
```

**On Worker 1 & Worker 2:**

```bash
sudo kubeadm join 10.10.3.68:6443 --token <token> \ --discovery-token-ca-cert-hash sha256:<hash>
```

**Verification Command (Master):**
```bash
kubectl get nodes
```

**Output (snippet):**

<img width="681" height="110" alt="image" src="https://github.com/user-attachments/assets/fab71cf4-95d3-4e16-b767-5401d40f9206" />


**Explanation:**

Both worker nodes joined successfully, cluster shows all nodes in Ready state.

---

### Lab Exercise 7 — Deploy Nginx Application (Master Only)
**Command:**
```bash
kubectl create deployment nginx-test --image=nginx --replicas=3
kubectl get pods -o wide
```

**Verification Command:**
```bash
kubectl get pods -o wide
```

**Output (snippet):**

<img width="1295" height="94" alt="image" src="https://github.com/user-attachments/assets/00039f3d-6a0a-4bf7-be40-c86de707d74c" />


**Explanation:**

Nginx deployment created with 3 replicas, pods distributed across worker nodes.

---

### Lab Exercise 8 — Expose Application (Master Only)
**Command**
```bash
kubectl expose deployment nginx-test --type=NodePort --port=80
```

**Verification Command:**
```bash
kubectl get svc
```

**Output (snippet):**

<img width="945" height="126" alt="image" src="https://github.com/user-attachments/assets/79880faf-9236-49d7-bf53-a25e989ad75f" />


**Access:**

    http://192.168.10.1:32188/ (192.168.10.1 in Bing)

    http://192.168.10.20:32188/ (192.168.10.20 in Bing)

    http://192.168.10.30:32188/ (192.168.10.30 in Bing)
    

<img width="1599" height="844" alt="image" src="https://github.com/user-attachments/assets/0cb78fe2-7881-4dbf-b73f-361b0f252eb1" />


**Explanation:**

Service exposed via NodePort, accessible on all nodes.

---

### Lab Exercise 9 — Ingress (Optional Enhancement)

**Step 1 — Check Ingress Controller Pods (before deployment):**
```bash
kubectl get pods -n ingress-nginx
```

**Output (snippet):**

<img width="939" height="68" alt="image" src="https://github.com/user-attachments/assets/82a1418a-d8f5-430e-86e0-0e25f27249b8" />

**Explanation:**

Initially, no ingress controller pods are running in the ingress-nginx namespace.

**Step 2 — Deploy Ingress Controller:**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

**Output (snippet):**

<img width="1443" height="375" alt="image" src="https://github.com/user-attachments/assets/c00c5429-b51f-4a1a-af75-54a66c25aa04" />

**Explanation:**  

The NGINX ingress controller was deployed successfully. Pods in the ingress-nginx namespace transitioned to Running state.

**Step 3 — Create Ingress Resource (Master Node):**

**Edit Ignress.yaml config file**

```bash
sudo nano ingress.yaml
```

**Ingress YAML Configuration:**

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

**Apply Resource:**
```bash
kubectl apply -f ingress.yaml
```

**Verification Command:**
```bash
kubectl get ingress
```

**Output (snippet):**

<img width="798" height="132" alt="image" src="https://github.com/user-attachments/assets/7109c2d9-c231-459c-9b05-1d08fe57b292" />

**Explanation:**

Ingress resource created successfully, routing traffic for hostname nginx.local to the nginx-test service on port 80.

**Step 4 — Configure Host File (Windows or Client Machine):**

Add all server IPs with ingress hostname nginx.local in the local host file.

<img width="1137" height="546" alt="image" src="https://github.com/user-attachments/assets/29de6a6d-2419-4012-a01a-9ebda130826a" />

**Explanation:**

Since no DNS server was configured, the hostname was mapped manually in the host file to resolve nginx.local to cluster node IPs.

**Step 5 — Access Application via Hostname:**

Open browser and access:

    http://nginx.local/
    
<img width="1596" height="843" alt="image" src="https://github.com/user-attachments/assets/a8266596-cd7b-4936-8184-c48305f329ec" />

**Explanation:**

Nginx application is now accessible via hostname nginx.local through the ingress controller, demonstrating production-style hostname routing.

---

## ✅ Learning Outcomes
- Built a Kubernetes cluster from scratch in my lab environment
- Configured Calico networking for pod communication
- Deployed and scaled Nginx across worker nodes
- Exposed applications via NodePort for IP-based access
- Configured Ingress for hostname-based routing

---

## 🚀 Future Work
- Add TLS/HTTPS via Ingress for secure traffic
- Deploy multiple applications and route via different paths/hosts
- Automate cluster setup with Ansible or Terraform
- Integrate CI/CD pipelines for automated deployments
