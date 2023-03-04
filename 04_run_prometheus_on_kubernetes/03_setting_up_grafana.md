# Setting Up Grafana

<img src="https://user-images.githubusercontent.com/6856382/222930465-2e7e1d13-c5af-4aa7-9733-970ebd0000c3.png">

## Grafana deployment
- `Grafana deployment` uses `grafana/grafana:3.1.1` image

<img src="https://user-images.githubusercontent.com/6856382/222930501-999f0a85-db7c-4188-a806-3b29a00b780a.png">

- `Grafana deployment` requires `GF_SECURITY_ADMIN_PASSWORD` to be unique and difficult to access

<img src="https://user-images.githubusercontent.com/6856382/222930553-44991240-18b8-46f5-ab9e-b1e58fe31de2.png">


## Instruction - Setting up Grafana Deployment

1. Setup Grafana deployment

```
kubectl apply -f grafana-deployment.yml
```

2. verify that Grafana deployment is successful

```
kubectl get pods -n monitoring
```

<img src="https://user-images.githubusercontent.com/6856382/222930995-388a95c2-f359-4cc9-aea6-ba48694be106.png">


