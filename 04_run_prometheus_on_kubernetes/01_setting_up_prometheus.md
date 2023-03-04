# Setting Up Prometheus

<img src="https://user-images.githubusercontent.com/6856382/222880850-496c0fdc-3d1f-4ca9-9801-c13075bdef1b.png">

## Instructions

1. install `git` on `kubernetes master`

**Kubernetes Master**
```
sudo apt-get install git
```

or in (centOS)

```
yum install -y git
```

2. download already-proviuded kubernetes prometheus env from git repo
- Requires addition of ssh-key. Resource found [here](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux)

**Kubernetes Control Plane**
```
git clone git@github.com:linuxacademy/content-kubernetes-prometheus-env.git
```

3. go to the clone repo folder

**Kubernetes Control Plane**
```
cd content-kubernetes-prometheus-env/
```

4. Create the kubernetes namespace where prometheus will be living in

**Kubernetes Control Plane**
```
kubectl apply -f namespaces.yml
```

3. setup prometheus confirguation map
- `configuration map` is an API object used to store non-confidental data in key-value pairs ([source](https://kubernetes.io/docs/concepts/configuration/configmap/))
- `configuration map` is a really convinent way of being able to go and decouple your configuration artifacts from your image content

**Kubernetes control plane**
```

```

#