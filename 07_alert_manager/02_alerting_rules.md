# Alerting Rules

- `alerting rules` allow you to define alert conditions based on Prometheus expression language and send notifications about firing alerts to an external service.
- `alerting rules` is very similar to recording rules


## Alerting Rules - Syntax

```
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
```

- `for` is the amount of time the alert needs to be active before it fires
- `severe` determines how severe the alert is going to be 
- `annotation` is like the summary of the alert


## Instruction 

1. download `linuxacademy/content-kubernetes-prometheus-env` from github

**Kubernetes Control Plane**
```
git clone git@github.com:linuxacademy/content-kubernetes-prometheus-env.git
```

2. go to `alertmanager` folder from the project root folder

**Kubernetes Control Plane**
```
cd content-kubernetes-prometheus-env
cd alertmanager
```

3. Apply `prometheus-rules.yml` to kubernetes

**kubernetes Control Plane**
```
kubectl apply -f prometheus-rules.yml
```

