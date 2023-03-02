# Setting Up Kubernetes Cluster

<img src="https://user-images.githubusercontent.com/6856382/222053379-340efb3f-99be-4ca9-abb1-1f284fe1a645.png">

## Instructions (From Kubernetes Essential Course)

### Installing Kubenetes

1. Disable Swap

**Kubernetes Control Plane**
```
sudo swapoff -a
```

2. Add Kubernetes Repository and install kubernetes (from kubernetes essential notes)

**All Kubernetes Nodes**
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
## Installing Docker


1. Login to each individual nodes, and type in the following command line codes (above doesnt work)

**All Kubernetes Nodes**
```
# UPDATE apt PACKAGE INDEX AND INSTALL PACKAGES TO ALLOW apt TO USE REPO OVER HTTPS
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release

# ADD THE DOCKER'S OFFICIAL GPG KEY:
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# SETUP REPO
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# RELOAD THE APT SOURCE LIST:
sudo apt-get update

# INSTALL DOCKER:
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu

# PREVENT UPDATE
sudo apt-mark hold docker-ce
```

## Setting up K8s Cluster

1. Start and enable kubelet

**Kubernetes Control Plane**
```
sudo systemctl start kubelet 
sudo systemctl enable kubelet
```

2. Create k8 config file

***To All Nodes** 
```
# set value `net.bridge.bridge-nf-call-iptables=1`, and make it apply even after computer restarts
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

# Make configuration to apply immediately
sudo sysctl -p
```

#