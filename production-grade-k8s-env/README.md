## Kubernetes Production Grade High-Availability (HA) Cluster Setup on AWS ##



1. AWS Infrastructure Setup 
Master Nodes (3 EC2 Instances)
Instance Type: t3.medium (2 vCPUs, 4 GB RAM)
Operating System: Ubuntu 24.04
Security Group: Allow SSH (port 22) and Kubernetes API (port 6443) access from the load balancer.

      b.  Worker Nodes (2 EC2 Instances)
Instance Type: t3.micro (1 vCPU, 1 GB RAM)
Operating System: Ubuntu 24.04
Security Group: Allow SSH (port 22) and communication with master nodes.

      c. Load Balancer
Use AWS Application Load Balancer (ALB) .
Listener: Configure ALB to listen on port 6443 and forward traffic to the master nodes.
Target Group: Register all three master nodes with port 6443.
Health Checks: Configure health checks on port 6443 to monitor the master nodes' API server.

2. Set Up Kubernetes Cluster on AWS
Install Kubernetes and Container Runtime
i. Disable Swap
```
bash
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
Ii. Install Container Runtime (containerd)
Iii. Install and configure containerd on all EC2 instances (masters and workers).
Iv. Enable IPv4 packet forwarding:

sudo sysctl net.ipv4.ip_forward=1

v. Configure containerd with SystemdCgroup = true and restart the service:

sudo systemctl restart containerd

vi. Install kubeadm and kubelet
vii. Install required packages and Kubernetes components:

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

viii. Add the Kubernetes apt repository and install kubeadm and kubelet:

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm
sudo apt-mark hold kubelet kubeadm


B. Bootstrap the Kubernetes Cluster
            i. Initialize the First Master Node (master1)

Ii. Execute the following command on master1 to initialize the cluster:

Copy code
kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS_NAME:6443" --upload-certs

Iii. Set up the kubeconfig file for kubectl on master1:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Join Additional Master Nodes (master2 and master3)

Iv. Run the generated kubeadm join command on master2 and master3 to join them to the control plane.
Join Worker Nodes (worker1 and worker2)

v. Run the generated kubeadm join command on both worker nodes.

C. Set Up Networking
i. Configure Container Network Interface (CNI) & install a CNI plugin for pod networking . We have used calico here.

kubectl apply -f <CNI_PLUGIN_YAML>

ii. Verify that all nodes are in the Ready state:

kubectl get nodes

D. Configure Kubernetes Management Tools on Load Balancer Node
i. Set Up kubeconfig on Load Balancer Node
ii. Copy the kubeconfig from master1 to the load balancer node.
iii. Install kubectl on the load balancer node:

snap install kubectl --classic


3. Best Practices for Production-Grade Kubernetes Cluster

A. Security
i. RBAC: Implement Role-Based Access Control for cluster management.
ii. Network Policies: Control pod-to-pod communication using network policies.
iii. Regular Updates: Keep Kubernetes and all components updated.

B. Monitoring 
i. Monitoring: Set up Prometheus and Grafana for monitoring cluster health.

C. Backup and Disaster Recovery
Create Scheduled / On-Demand Backup Strategies & Store backups securely in a remote, highly available storage solution (e.g., AWS S3, GCS) and test recovery procedures frequently to ensure quick and reliable data restoration in case of failure.

D. Resource Management
i. Quotas and Limits: Use Resource Quotas and Limit Ranges to manage resource consumption.




E. Update Strategy
Plan for regular updates to Kubernetes and related components to maintain security and stability.


F. Secret Management
Use a secure secret management solution like HashiCorp Vault.


G. Node Management
Implement Node Management practices while using tools like Flux or Jenkins X for version-controlled cluster management.


This document outlines the process of setting up a multi-master Kubernetes cluster on AWS, leveraging services like ALB for load balancing, EC2 for compute resources, and best practices for maintaining a production-grade environment.


Full Documentation is on the below link:- 
https://docs.google.com/document/d/1NkygRLZspPWHjJTZ7CVb7yYkRbUz2iH1n-g2z7RSBcw/edit?usp=sharing




