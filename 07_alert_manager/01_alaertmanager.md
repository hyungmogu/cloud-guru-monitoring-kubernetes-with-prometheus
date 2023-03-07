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