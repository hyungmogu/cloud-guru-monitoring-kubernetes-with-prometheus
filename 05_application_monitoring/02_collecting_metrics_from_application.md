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

5. Publish image to Docker hub

**syntax**
```
docker push <docker_hub_username>/<app_name>
```

**Kubernetes Control Plane (or Local)**
```
docker push rivethead42/comicbox
```

6. In Kubernetes Control plane, apply kubernetes deployment sequence

**Kubernetes Control Plane**
```
kubectl apply -f deployment.yml
```

7. check to make sure pods are deployed 
- in this case, it's deployed to default namespace

**Kubernetes Control Plane**
```
kubectl get pods
```

```

```

8. check to make sure services are setup as intended

**Kubernetes Control Plane**
```
kubectl get services
```

```

```

9. Access swagger UI 

**Browser**
```
https://<PUBLIC_IP_ADDRESS>:8001/swagger-stats/ui
```

<img src="https://user-images.githubusercontent.com/6856382/222994359-b2d9ce06-fce7-4a55-8da7-77d14095b7b3.png">

## Importing Swagger-Stat data to Grafana

1. Access Grafana UI dashboard

**Browser**
```
https://<PUBLIC_IP_ADDRESS>:8000/dashboard
```

2. Import Swagger-Stat to Grafana Dashboard
- ID for Swagger-stat dashboard is `3091`

<img src="https://user-images.githubusercontent.com/6856382/222994510-0b09ebbb-1702-43a0-9bc7-1bd0f2de7b44.png">

<img src="https://user-images.githubusercontent.com/6856382/222995083-707f807d-2012-4ed1-ae50-b6e85ab85732.png">

3. Set name and `prometheus` as data source

<img src="https://user-images.githubusercontent.com/6856382/222995177-d0c8b091-84fe-4573-9828-e091632e1db7.png">

#