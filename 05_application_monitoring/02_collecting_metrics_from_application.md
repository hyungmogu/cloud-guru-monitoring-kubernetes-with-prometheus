# Collecting Metrics from Application

- This lesson covers how to 
    1. deploy application to kubernetes
    2. have prometheus gather metrics from it

<img src="https://user-images.githubusercontent.com/6856382/222991465-1afca741-7fbc-4d69-953b-79055ddef5ca.png">

## Requirements

1. Docker hub account (to upload docker containers)

## Instruction

1. Download app from github `linuxacademy/content-kubernetes-prometheus-app`

**Kubernetes Control Plane (or local)**
```
git clone git@github.com:linuxacademy/content-kubernetes-prometheus-app.git
cd content-kubernetes-prometheus-app
```

2. Ensure the following annotation is included in `deployment.yml`
- `service discovery` requires it to find and collect data

**~/content-kubernetes-prometheus-app/deployment.yml**
```
...
      annotations:
            prometheus.io/scrape: "true"
            prometheus.io/path: 'swagger-stats/metrics'
            prometheus.io/port: "3000"
```

- `prometheus.io/port: "3000"` is the internal port in kubernetes
- `prometheus.io/port: "3000"` is NOT servers port (8001) used to access endpoints in browser

**~/content-kubernetes-prometheus-app/deployment.yml (Full File)**
```
apiVersion: v1
kind: Service
metadata:
  name: comicbox-service
spec:
  selector:
    app: comicbox
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 8001
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: comicbox
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: comicbox
      annotations:
            prometheus.io/scrape: "true"
            prometheus.io/path: 'swagger-stats/metrics'
            prometheus.io/port: "3000"
    spec:
      containers:
        - name: comicbox
          image: rivethead42/comicbox
          ports:
            - containerPort: 3000
```

3. Create a Docker image containing the application

**Syntax**
```
docker built -t <docker_hub_username>/<app_name> .
```

**Kubernetes Control Plane (or Local), Example**
```
docker built -t rivethead42/comicbox . 
```

4. Login to docker hub

**Kubernetes Control Plane (or Local)**
```
docker login
```

#