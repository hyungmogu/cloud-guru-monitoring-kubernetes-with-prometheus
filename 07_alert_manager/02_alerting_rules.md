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
- `expr` is the criteria that must be met for the alert to fire
- `for` is the amount of time the alert needs to be active before it fires
- `severity` determines how severe the alert is going to be 
  - e.g. is it just a warning, or do we need to page and wake someone up to debug the problem?
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

3. Apply `prometheus-rules-config-map.yml` to kubernetes

**kubernetes Control Plane**
```
kubectl apply -f prometheus-rules-config-map.yml
```

**content-kubernetes-prometheus-env/alertmanager/prometheus-rules-config-map.yml**
```
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: prometheus-rules-conf
  namespace: monitoring
data:
  kubernetes_alerts.yml: |
    groups:
      - name: kubernetes_alerts
        rules:
        - alert: DeploymentGenerationOff
          expr: kube_deployment_status_observed_generation != kube_deployment_metadata_generation
          for: 5m
          labels:
            severity: warning
          annotations:
            description: Deployment generation does not match expected generation {{ $labels.namespace }}/{{ $labels.deployment }}
            summary: Deployment is outdated
        - alert: DeploymentReplicasNotUpdated
          expr: ((kube_deployment_status_replicas_updated != kube_deployment_spec_replicas)
            or (kube_deployment_status_replicas_available != kube_deployment_spec_replicas))
            unless (kube_deployment_spec_paused == 1)
          for: 5m
          labels:
            severity: warning
          annotations:
            description: Replicas are not updated and available for deployment {{ $labels.namespace }}/{{ $labels.deployment }}
            summary: Deployment replicas are outdated
        - alert: PodzFrequentlyRestarting
          expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
          for: 10m
          labels:
            severity: warning
          annotations:
            description: Pod {{ $labels.namespace }}/{{ $labels.pod }} was restarted {{ $value }} times within the last hour
            summary: Pod is restarting frequently
        - alert: KubeNodeNotReady
          expr: kube_node_status_condition{condition="Ready",status="true"} == 0
          for: 1h
          labels:
            severity: warning
          annotations:
            description: The Kubelet on {{ $labels.node }} has not checked in with the API,
              or has set itself to NotReady, for more than an hour
            summary: Node status is NotReady
        - alert: KubeManyNodezNotReady
          expr: count(kube_node_status_condition{condition="Ready",status="true"} == 0)
            > 1 and (count(kube_node_status_condition{condition="Ready",status="true"} ==
            0) / count(kube_node_status_condition{condition="Ready",status="true"})) > 0.2
          for: 1m
          labels:
            severity: critical
          annotations:
            description: '{{ $value }}% of Kubernetes nodes are not ready'
        - alert: APIHighLatency
          expr: apiserver_latency_seconds:quantile{quantile="0.99",subresource!="log",verb!~"^(?:WATCH|WATCHLIST|PROXY|CONNECT)$"} > 4
          for: 10m
          labels:
            severity: critical
          annotations:
            description: the API server has a 99th percentile latency of {{ $value }} seconds for {{ $labels.verb }} {{ $labels.resource }}
        - alert: APIServerErrorsHigh
          expr: rate(apiserver_request_count{code=~"^(?:5..)$"}[5m]) / rate(apiserver_request_count[5m]) * 100 > 5
          for: 10m
          labels:
            severity: critical
          annotations:
            description: API server returns errors for {{ $value }}% of requests
        - alert: KubernetesAPIServerDown
          expr: up{job="kubernetes-apiservers"} == 0
          for: 10m
          labels:
            severity: critical
          annotations:
            summary: Apiserver {{ $labels.instance }} is down!
        - alert: KubernetesAPIServersGone
          expr:  absent(up{job="kubernetes-apiservers"})
          for: 10m
          labels:
            severity: critical
          annotations:
            summary: No Kubernetes apiservers are reporting!
            description: Werner Heisenberg says - OMG Where are my apiserverz?
  prometheus_alerts.yml: |
    groups:
    - name: prometheus_alerts
      rules:
      - alert: PrometheusConfigReloadFailed
        expr: prometheus_config_last_reload_successful == 0
        for: 10m
        labels:
          severity: warning
        annotations:
          description: Reloading Prometheus configuration has failed on {{$labels.instance}}.
      - alert: PrometheusNotConnectedToAlertmanagers
        expr: prometheus_notifications_alertmanagers_discovered < 1
        for: 1m
        labels:
          severity: warning
        annotations:
          description: Prometheus {{ $labels.instance}} is not connected to any Alertmanagers
  node_alerts.yml: |
    groups:
    - name: node_alerts
      rules:
      - alert: HighNodeCPU
        expr: instance:node_cpu:avg_rate5m > 80
        for: 10s
        labels:
          severity: warning
        annotations:
          summary: High Node CPU of {{ humanize $value}}% for 1 hour
      - alert: DiskWillFillIn4Hours
        expr: predict_linear(node_filesystem_free_bytes{mountpoint="/"}[1h], 4*3600) < 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Disk on {{ $labels.instance }} will fill in approximately 4 hours.
      - alert: KubernetesServiceDown
        expr: up{job="kubernetes-service-endpoints"} == 0
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: Pod {{ $labels.instance }} is down!
      - alert: KubernetesServicesGone
        expr:  absent(up{job="kubernetes-service-endpoints"})
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: No Kubernetes services are reporting!
          description: Werner Heisenberg says - OMG Where are my servicez?
      - alert: CriticalServiceDown
        expr: node_systemd_unit_state{state="active"} != 1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: Service {{ $labels.name }} failed to start.
          description: Service {{ $labels.instance }} failed to (re)start service {{ $labels.name }}.
  redis_alerts.yml: |
    groups:
    - name: redis_alerts
      rules:
      - alert: RedisCacheMissesHigh
        expr: redis_keyspace_hits_total / (redis_keyspace_hits_total + redis_keyspace_misses_total) > 0.8
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: Redis Server {{ $labels.instance }} Cache Misses are high.
      - alert: RedisRejectedConnectionsHigh
        expr: redis_connected_clients{} > 100
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Redis instance {{ $labels.addr }} may be hitting maxclient limit."
          description: "The Redis instance at {{ $labels.addr }} had {{ $value }} rejected connections during the last 10m and may be hitting the maxclient limit."
      - alert: RedisServerDown
        expr: redis_up{app="media-redis"} == 0
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: Redis Server {{ $labels.instance }} is down!
      - alert: RedisServerGone
        expr:  absent(redis_up{app="media-redis"})
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: No Redis servers are reporting!
          description: Werner Heisenberg says - there is no uncertainty about the Redis server being gone.
  kubernetes_rules.yml: |
    groups:
      - name: kubernetes_rules
        rules:
        - record: apiserver_latency_seconds:quantile
          expr: histogram_quantile(0.99, rate(apiserver_request_latencies_bucket[5m])) / 1e+06
          labels:
            quantile: "0.99"
        - record: apiserver_latency_seconds:quantile
          expr: histogram_quantile(0.9, rate(apiserver_request_latencies_bucket[5m])) / 1e+06
          labels:
            quantile: "0.9"
        - record: apiserver_latency_seconds:quantile
          expr: histogram_quantile(0.5, rate(apiserver_request_latencies_bucket[5m])) / 1e+06
          labels:
            quantile: "0.5"
  node_rules.yml: |
    groups:
    - name: node_rules
      rules:
        - record: instance:node_cpu:avg_rate5m
          expr: 100 - avg(irate(node_cpu_seconds_total{job="node-exporter", mode="idle"}[5m])) by (instance) * 100
        - record: instance:node_memory_usage:percentage
          expr: ((sum(node_memory_MemTotal_bytes) - sum(node_memory_MemFree_bytes) - sum(node_memory_Buffers_bytes) - sum(node_memory_Cached_bytes)) / sum(node_memory_MemTotal_bytes)) * 100
        - record: instance:root:node_filesystem_usage:percentage
          expr: (node_filesystem_size_bytes{mountpoint="/rootfs"} - node_filesystem_free_bytes{mountpoint="/rootfs"}) /node_filesystem_size_bytes{mountpoint="/rootfs"} * 100
  redis_rules.yml: |
    groups:
    - name: redis_rules
      rules:
      - record: redis:command_call_duration_seconds_count:rate2m
        expr: sum(irate(redis_command_call_duration_seconds_count[2m])) by (cmd, environment)
      - record: redis:total_requests:rate2m
        expr: rate(redis_commands_processed_total[2m])

```

- all the information about alert written in config-map.yml file is include in UI

<img src="https://user-images.githubusercontent.com/6856382/223890380-79ba51f1-abd7-48b3-b42c-1f57db01c44a.png"/>


#