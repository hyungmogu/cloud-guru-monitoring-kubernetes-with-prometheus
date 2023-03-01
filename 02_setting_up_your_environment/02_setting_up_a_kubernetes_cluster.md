# Setting Up Kubernetes Cluster

<img src="https://user-images.githubusercontent.com/6856382/222053379-340efb3f-99be-4ca9-abb1-1f284fe1a645.png">

## Instructions

### Installing Kubenetes

1. Disable Swap

**Kubernetes Control Plane**
```
sudo swapoff -a
```

2. edit file to permanently disable swap

**Kubernetes Control Plane**
```
sudo vim /etc/fstab
```

/etc/fstab
<img src="https://user-images.githubusercontent.com/6856382/222060817-256b186b-d097-4c52-a21e-87932379637e.png"/>

