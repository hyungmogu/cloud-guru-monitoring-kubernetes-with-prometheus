# Alert Manager

<img src="https://user-images.githubusercontent.com/6856382/223328522-61ff583d-9cf3-4b50-8704-586bdb4d763d.png">

- In this lesson, we are going to send alert messages to slack

## Instruction

1. download github repository `linuxacademy/content-kubernetes-prometheus-env`

**Kubernetes Control Plane**
```
gie clone git@github.com:linuxacademy/content-kubernetes-prometheus-env.git
```

2. go to project root folder

**Kubernetes Control Plane**
```
cd content-kubernetes-prometheus-env
```

## Instruction - Setting up Slack

1. Create a slack account and log into slack dashboard

2. Select `create a channel`

<img src="https://user-images.githubusercontent.com/6856382/223332441-603cd855-1cc6-45a6-9b58-6273c1b514ba.png">

3. Name the channel `alert` and select `create channel`

<img src="https://user-images.githubusercontent.com/6856382/223333439-af4480e7-179d-4b59-a684-e046b3eee80d.png">

## Instruction - Setting up custom prometheus app

1. Go to `https://api.slack.com/` and select `create an app`

<img src="https://user-images.githubusercontent.com/6856382/223336003-27562070-4d25-4320-9235-4ee2c835b707.png">

2. Select `Create From Scratch`

<img src="https://user-images.githubusercontent.com/6856382/223336265-3b4aed56-60e7-45a7-a986-61f8cea61df5.png">

3. Set
    1. `App Name` as Prometheus
    2. `Development Slack Workspace` as the workspace where the app should be placed

<img src="https://user-images.githubusercontent.com/6856382/223336638-104e2fc2-0a85-481b-a817-0d8e06ac252d.png">

4. Activate incoming webhooks

<img src="https://user-images.githubusercontent.com/6856382/223336968-9b1b0d96-022a-431f-8b48-0543309892f5.png">

5. Scroll down and select `Add New Webhook to Workspace`

<img src="https://user-images.githubusercontent.com/6856382/223343360-99d93f17-36e0-4117-926e-468aa6fac381.png">

6. Select the channel created earlier under `Instruction - Setting up Slack` section

<img src="https://user-images.githubusercontent.com/6856382/223344156-93dc4b22-b6cc-4536-bcd8-b3a6911e219e.png">

7. copy webhook url and paste it to `api_url` in file `content-kubernetes-prometheus-env/alertmanager/alertmanager-configmap.yml`

<img src="https://user-images.githubusercontent.com/6856382/223345877-9c79a200-90a0-4e69-82a6-7d234501b05e.png">

**content-kubernetes-prometheus-env/alertmanager/alertmanager-configmap.yml**
<img src="https://user-images.githubusercontent.com/6856382/223346072-63447229-278e-4a9c-bf00-fb153523b637.png">

8. copy channel name and paste it to `channel` in file `content-kubernetes-prometheus-env/alertmanager/alertmanager-configmap.yml`

<img src="https://user-images.githubusercontent.com/6856382/223346576-2c032f37-f8e5-463c-af2d-09bfbf4ad062.png">

**content-kubernetes-prometheus-env/alertmanager/alertmanager-configmap.yml**
<img src="https://user-images.githubusercontent.com/6856382/223346898-b8366933-a7db-4763-b9ab-9d92d674b99d.png">


9. Copy username and paste it to `username` in file

<img src="https://user-images.githubusercontent.com/6856382/223347117-43206aaa-6e84-4a2c-acf7-9c802b24f3fb.png">

## Instruction - Applying alertmanager configuration to Kubernetes

1. apply file `alertmanager-configmap.yml` to kubernetes

**Kubernetes Control Plane**
```
cd content-kubernetes-prometheus-env/alertmanager
kubectl apply -f alertmanager-configmap.yml
```

2. apply deployment instruction from file `alertmanager-deployment.yml` to kubernetes

**Kubernetes Control Plane**
```
kubectl apply -f alertmanager-deployment.yml
```

