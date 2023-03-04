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


1. Login to each individual nodes, and type in the following command line codes 

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

2. Bootstrap the cluster

**kubernetes Control Plane**
```
# Initialize the cluster on the Kube Master server
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Setup kubeconfig for the local user on the kube master server
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

3. copy paste the following code provided after `sudo kubeadm init`

**Kubernetes Worker Nodes**
```
[[sudo kubeadm join $controller_ip:6443 --token $token --discovery-token-ca-cert-hash $hash]]
```

4. verify that the kubernetes cluster is working correctly

**Kubernetes Control Plane**
```
kubectl get nodes
```

```
NAME                           STATUS     ROLES    AGE     VERSION
92c60ee3641c.mylabserver.com   NotReady   master   3m22s   v1.15.7
92c60ee3642c.mylabserver.com   NotReady   <none>   57s     v1.15.7
```

## Configuring Networking with Flannel

1. turn on `net.bridge-bridge-nf-call-iptables`

**To All Nodes**
```
# set value `net.bridge.bridge-nf-call-iptables=1`, and make it apply even after computer restarts
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

# Make configuration to apply immediately
sudo sysctl -p
```

2. Install flannel

**Kubernentes Control Plane**
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel-old.yaml
```

3. Verify that all nodes now have status `Ready`

**Kubernetes Control Plane**
```
kubectl get nodes -w
```

```
NAME                           STATUS   ROLES    AGE     VERSION
92c60ee3641c.mylabserver.com   Ready    master   10m     v1.15.7
92c60ee3642c.mylabserver.com   Ready    <none>   8m18s   v1.15.7
```

4. Verify that flannel pods are up and running

**Kubernetes Control Plane**
```
kubectl get pods -n kube-system
```

```
NAME                                                   READY   STATUS              RESTARTS   AGE
coredns-5d4dd4b4db-qr55l                               0/1     ContainerCreating   0          11m
coredns-5d4dd4b4db-xzklt                               0/1     ContainerCreating   0          11m
etcd-92c60ee3641c.mylabserver.com                      1/1     Running             0          11m
kube-apiserver-92c60ee3641c.mylabserver.com            1/1     Running             0          11m
kube-controller-manager-92c60ee3641c.mylabserver.com   1/1     Running             0          11m
kube-flannel-ds-amd64-jmttb                            1/1     Running             0          98s
kube-flannel-ds-amd64-mjl6q                            1/1     Running             0          98s
kube-proxy-q5v4s                                       1/1     Running             0          9m41s
kube-proxy-vhkjz                                       1/1     Running             0          11m
kube-scheduler-92c60ee3641c.mylabserver.com            1/1     Running             0          10m
```

#