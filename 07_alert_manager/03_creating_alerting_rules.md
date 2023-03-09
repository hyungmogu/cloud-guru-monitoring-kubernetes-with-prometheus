# Creating Alerting Rules

## Setting up the Environment

1. Bootstraping is a trouble-free way to setting up the environment.

**Kubernetes Control Plane**
```
sudo su -
cd prometheus
sh bootstrap.sh
```

## Creating a ConfigMap that will be used to manage the alerting rules

1. Edit `Prometheus-rules-config-map.yml` and add redis alerting rules
- More information about `alerting rules`, and the use of `$labels` can be found [here](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- More information about `functions` can be found [here](https://prometheus.io/docs/prometheus/latest/querying/functions/)


**~/prometheus/prometheus-rules-config-map.yml**
```
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: prometheus-rules-conf
  namespace: monitoring
data:
  redis_rules.yml: |
    groups:
    - name: redis_rules
      rules:
      - record: redis:command_call_duration_seconds_count:rate2m
        expr: sum(irate(redis_command_call_duration_seconds_count[2m])) by (cmd, environment)
      - record: redis:total_requests:rate2m
        expr: rate(redis_commands_processed_total[2m])
  redis_alerts.yml: |
    groups:
    - name: redis_alerts
      rules:
      - alert: RedisServerGone
        expr: absent(redis_up{app="media-redis"})
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: No Redis servers are reporting!
      - alert: RedisServerDown
        expr: redis_up{app="media-redis"} == 0
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: Redis Server {{ $labels.instance }} is down!
```

2. Apply changes to kubernetes cluster

**Kubernetes Control Plane**
```
kubectl apply -f prometheus-rules-config-map.yml
```

## Update Prometheus pod by deleting it

1. list all the pods under the namespace `monitoring`

**Kubernetes Control Plane**
```
kubectl get pods -n monitoring
```

```
NAME                                    READY     STATUS    RESTARTS   AGE
alertmanager-7bf787dfd8-5qvs5           1/1       Running   0          28m
kube-state-metrics-8548c449db-2fcnz     1/1       Running   0          28m
prometheus-deployment-8d4db8f98-gllk6   1/1       Running   0          28m
```

2. Delete prometheus pod

**Kubernetes Control Plane**
```
kubectl delete pods prometheus-deployment-8d4db8f98-gllk6 -n monitoring
```

3. Check that the new version of prometheus deployment pod is up

**Kubernetes Control Plane**
```
kubectl get pods -n monitoring
```

```
NAME                                    READY     STATUS    RESTARTS   AGE
alertmanager-7bf787dfd8-5qvs5           1/1       Running   0          31m
kube-state-metrics-8548c449db-2fcnz     1/1       Running   0          31m
prometheus-deployment-8d4db8f98-p2dr5   1/1       Running   0          18s
```

## Notes

1. `configMap` is an api object used to store non-confidential data
2. `configMap` is consumed by pod as 
    1. environmental variables
    2. command-line arguements
    3. configuration files in a volume

