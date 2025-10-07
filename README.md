# K8s Setup Guide

[Universal Setup](#) [Master Node](#) [Worker Node](#) [Verification](#) [Troubleshooting](#) [Audit](#)

## Universal Node Setup

These steps must be performed on all nodes in your cluster, both master and worker nodes. This section covers installing the container runtime and essential Kubernetes components.

### 1\. Install Containerd Master & Worker

This script installs containerd, the container runtime Kubernetes will use to manage container lifecycles.

Copy

```
sudo apt-get update
sudo apt install apt-transport-https curl -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install containerd.io -y
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

### 2\. Configure Containerd for Systemd Master & Worker

Modify the containerd configuration to use the systemdCgroup driver, which is required by kubelet.

First, open the configuration file:

Copy

```
sudo vim /etc/containerd/config.toml
```

Find the line SystemdCgroup = false and change SystemdCgroup to true.

Then, restart the service:

Copy

```
sudo systemctl restart containerd
```

### 3\. Install Kubernetes Components Master & Worker

Install \`kubelet\`, \`kubeadm\`, and \`kubectl\` and hold their versions to prevent unintended upgrades.

Copy

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

### 4\. Final Prerequisites Master & Worker

Disable swap and ensure the br\_netfilter module is loaded for Kubernetes networking.

Disable swap:

Copy

```
sudo swapoff -a
# Also comment out the swap line in /etc/fstab to make it permanent
```

Enable kernel modules and settings:

Copy

```
sudo modprobe br_netfilter
sudo sysctl -w net.ipv4.ip_forward=1
```

## Master Node Setup

These commands are run only on the master node. This section initializes the cluster control plane and sets up networking.

### 1\. Initialize Cluster Master Only

Start the Kubernetes control plane. This command will output a kubeadm join command; save it to join worker nodes later.

Copy

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### 2\. Configure Kubeconfig & CNI Master Only

Configure kubectl for the root user and apply the Flannel CNI (Container Network Interface) for pod networking.

Copy

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## Worker Node Setup

This command is run only on worker (slave) nodes. It connects the worker to the master's control plane.

### 1\. Join the Cluster Worker Only

Use the kubeadm join command that was generated during the kubeadm init step on the master node.

Copy

```
# This is an example command. Use the one from YOUR master node.
kubeadm join 10.0.0.6:6443 --token 92v88z.22t0ypwhkly8ipc7 --discovery-token-ca-cert-hash sha256:913e4bbd...
```

## Cluster Verification

After joining all nodes, run these commands on the master node to verify the cluster's health and node status.

### 1\. Check Node Status Master Only

Verify that all nodes have joined the cluster and are in a Ready state.

Copy

```
kubectl get nodes
```

### 2\. Check System Pods Master Only

Ensure all pods in the kube-system namespace (including Flannel) are running correctly.

Copy

```
kubectl get pods --all-namespaces
```

## Troubleshooting

If you encounter networking issues or nodes are not becoming ready, these steps can help resolve common problems.

### 1\. Fix br\_netfilter Issue Master & Worker

If nodes cannot communicate, explicitly load the kernel module and configure sysctl for bridge-nf-call on all nodes.

Copy

```
sudo modprobe br_netfilter
echo 'br_netfilter' | sudo tee /etc/modules-load.d/br_netfilter.conf

sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF

sudo sysctl --system
```

### 2\. Verify Flannel Pods Master Only

Check the status of Flannel pods specifically to debug networking issues.

Copy

```
kubectl get pods -n kube-system -l app=flannel -o wide
kubectl get pods -n kube-flannel -o wide
```

## Cluster Audit with Kubeaudit

Use kubeaudit on the master node to scan your cluster for common security vulnerabilities and misconfigurations.

### 1\. Install Kubeaudit Master Only

Download and install the kubeaudit binary. Check the [official releases page](https://github.com/shopify/kubeaudit/releases) for the latest version.

Copy

```
# Example for v0.22.2
wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.2/kubeaudit_0.22.2_linux_amd64.tar.gz
tar -xvf kubeaudit_0.22.2_linux_amd64.tar.gz
sudo mv kubeaudit /usr/local/bin/
```

### 2\. Run Audit Master Only

Execute the audit on all cluster resources.

Copy

```
kubeaudit all
```

## Author & Contact

This guide was created to simplify Kubernetes setup. Feel free to connect for feedback or collaboration.

### Get in Touch

*   **Name:** Karan Negi
*   **Email:** [knegi2003@gmail.com](mailto:knegi2003@gmail.com)
*   **GitHub:** [https://github.com/Karan-Negi-12](https://github.com/Karan-Negi-12)
*   **LinkedIn:** [https://www.linkedin.com/in/karan-negi-62529022b/](https://linkedin.com/in/your-profile-handle)
*   **Guide Version:** v1.0.1 (October 2025)
