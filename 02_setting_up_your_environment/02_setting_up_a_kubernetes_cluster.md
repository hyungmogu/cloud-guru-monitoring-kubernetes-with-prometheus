# Setting Up Kubernetes Cluster

<img src="https://user-images.githubusercontent.com/6856382/222053379-340efb3f-99be-4ca9-abb1-1f284fe1a645.png">

## Instructions

### Installing Kubenetes

1. Disable Swap

**Kubernetes Control Plane**
```
sudo swapoff -a
```

2. Add Kubernetes Repository and install kubernetes (from kubernetes essential notes)

**Kubernetes Control Plane**
```
# Add Kubernetes Repository GPG key
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Add Kubernetes Repository
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

# Reload the apt sources list
sudo apt-get update

# Install packages
sudo apt-get install -y kubelet=1.15.7-00 kubeadm=1.15.7-00 kubectl=1.15.7-00

# Hold auto upgrading of kubeadm, kubectl, kubelet
sudo apt-mark hold kubelet kubeadm kubectl

# Check version
kubeadm version
```

