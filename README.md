# Kubernetes Cluster Setup Guide

---

This guide walks you through the **step-by-step setup of a Kubernetes cluster**. Each command below is preserved exactly as in the original instructions. Explanations have been added for clarity and learning purposes.

---

## Step 1: #this is self hosted cluster setup

#Script-01 (Apply on both master and slave)

## Step 2: --->

```bash
sudo apt-get update
```

```bash
sudo apt install apt-transport-https curl -y
```

```bash
sudo mkdir -p /etc/apt/keyrings
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt-get update -y
```

```bash
sudo apt-get install containerd.io -y
```

```bash
sudo mkdir -p /etc/containerd
```

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

## Step 3: --->

#after running first script open this file and made chanegs (on Both master and slave)

```bash
sudo vim /etc/containerd/config.toml
```

#made chnages (on Both master and slave)

## Step 4: Systemdcgroup = true

#Restart containerd (on Both master and slave)

```bash
sudo systemctl restart containerd
```

#Script-02 (Apply on both master and slave)

## Step 5: --->

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install -y kubelet kubeadm kubectl
```

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

```bash
sudo systemctl enable --now kubelet
```

## Step 6: --->

#Disable Swap (on Both master and slave)

```bash
sudo swapoff -a
```

#Check the swap it should be 0

## Step 7: Free -h

#if there are still swap on [commnet out the last swap] (on Both master and slave)

```bash
sudo vim /etc/fstab
```

#Enabel kernel modules (on Both master and slave)

```bash
sudo modprobe br_netfilter
```

#Apply setting in sysctl (on Both master and slave)

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

## Step 8: #master only

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16  [at the end of this command result there will be the kubeadm token which help slave machine to join the cluster]
```

## Step 9: #master only run one by one

```bash
mkdir -p $HOME/.kube
```

```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

```bash
kubectl get pods --all-namespaces
```

#Slave only [apply your token on the slave machine]

```bash
kubeadm join 10.0.0.6:6443 --token 92v88z.22t0ypwhkly8ipc7 --discovery-token-ca-cert-hash sha256:913e4bbdb7a46c00532e02e9024a82db57f77cbc3c8b5b040
```

## Step 10: #to verify the cluster

## Step 11: Kubeclt get nodes

#for kubeaduit [this will audit the cluster and scan if for the differnet kinf of vulnerability]

https://github.com/shopify/kubeaudit/releases --visit this copy link address as per your need

## Step 12: #in master to apply kubeaudit

```bash
wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.2/kubeaudit_0.22.2_linux_amd64.tar.gz
```

```bash
tar -xvf kubeaudit_0.22.2_linux_amd64.tar.gz
```

```bash
sudo mv kubeaudit /usr/local/bin/
```

## Step 13: Kubeaudit all

#If you face br_netfilter issue [master slave connectivity issue]

#Run the following on both master and worker nodes:

```bash
sudo modprobe br_netfilter
```

```bash
echo 'br_netfilter' | sudo tee /etc/modules-load.d/br_netfilter.conf
```

#Now enable the bridge network filtering [run this as a command]

```bash
sudo tee /etc/sysctl.d/k8s.conf <<EOF
```

## Step 14: Net.bridge.bridge-nf-call-iptables=1

## Step 15: Net.bridge.bridge-nf-call-ip6tables=1

## Step 16: Net.ipv4.ip_forward=1

## Step 17: Eof

```bash
sudo sysctl --system
```

## Step 18: #if will start working to verify

```bash
kubectl get pods -n kube-system -l app=flannel -o wide
```

```bash
kubectl get pods -n kube-flannel -o wide
```
