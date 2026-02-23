# kubernetes-three-node-lab
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
<img width="1205" height="90" alt="Swap Status" src="https://github.com/user-attachments/assets/b5af56d3-617f-43df-bcad-0e0823477a73" />
