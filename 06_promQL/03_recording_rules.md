# Recording Rules

- `recording rule` is a step before venturing into Prometheus's alerting manager
- `recording rule` can be accessed in `rule` menu item under `status`
- `recording rule` is a set of customized metrics stores in a file
- `recording rule` relieves user from having to write customized metrics on every setup 

<img src="https://user-images.githubusercontent.com/6856382/223038794-156806c5-f501-4558-8330-1e255861ff14.png">


## Syntax - Recording Rules

- `recording rules` has 
  1. `record` keyword 
    - `record` is the name of the time series to output to
    - `record` has type string
  2. `expr` keyword
    - `expr` is the PromQL expression to evaluate. 
    - `expr` evaluates expression at the current time
    - `expr` has type string
  
  3. `labels` keyword
    - `labels` add or overwrite before storing the default
    - `labels` is an array whose element is in format `<label_name>:<label_value>`


**Examples (~/content-kubernetes-prometheus-env/readrules/prometheus-read-rules.yml)**
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-read-rules-conf
  labels:
    name: prometheus-read-rules-conf
  namespace: monitoring
data:
  node_rules.yml: |-
    groups:
    - name: node_rules
      interval: 10s
      rules:
        - record: instance:node_cpu:avg_rate5m
          expr: 100 - avg(irate(node_cpu_seconds_total{job="node-exporter", mode="idle"}[5m])) by (instance) * 100
        - record: instance:node_memory_usage:percentage
          expr: ((sum(node_memory_MemTotal_bytes) - sum(node_memory_MemFree_bytes) - sum(node_memory_Buffers_bytes) - sum(node_memory_Cached_bytes)) / sum(node_memory_MemTotal_bytes)) * 100
        - record: instance:root:node_filesystem_usage:percentage
          expr: (node_filesystem_size_bytes{mountpoint="/rootfs"} - node_filesystem_free_bytes{mountpoint="/rootfs"}) /node_filesystem_size_bytes{mountpoint="/rootfs"} * 100
```

## Instruction - Adding Recording Rules

1. Download files from github repo

**Kubernetes Control Plane**
```
git clone git@github.com:linuxacademy/content-kubernetes-prometheus-env.git
```

2. Add rules files in `prometheus config map` file, and replace `KUBERNETES_IP` with **private ip** of operating server instances

```
    ...
    rule_files:
    - rules/*_rule.yml
```


**~/content-kubernetes-prometheus-env/prometheus/prometheus-config-map.yml**
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-server-conf
  labels:
    name: prometheus-server-conf
  namespace: monitoring
data:
  prometheus.yml: |-
    global:
      scrape_interval: 5s
      evaluation_interval: 5s

    rule_files:
    - rules/*_rule.yml

    scrape_configs:
      - job_name: 'node-exporter'
        static_configs:
        - targets: ['<KUBERNETES_IP>:9100', '<KUBERNETES_IP>:9100']

      - job_name: 'kubernetes-apiservers'

        kubernetes_sd_configs:
        - role: endpoints
        scheme: https

        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https

      - job_name: 'kubernetes-nodes'

        scheme: https

        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

        kubernetes_sd_configs:
        - role: node

        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics


      - job_name: 'kubernetes-pods'

        kubernetes_sd_configs:
        - role: pod

        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name

      - job_name: 'kubernetes-cadvisor'

        scheme: https

        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

        kubernetes_sd_configs:
        - role: node

        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor

      - job_name: 'kubernetes-service-endpoints'

        kubernetes_sd_configs:
        - role: endpoints

        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
          action: replace
          target_label: __scheme__
          regex: (https?)
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          target_label: __address__
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: kubernetes_name
```

3. Add read rules volume in `prometheus deployment` file

```
    ...
    spec:
      containers:
        - name: prometheus
          ...
          volumeMounts:
            - name: prometheus-read-rules-volume
              mountPath: /etc/prometheus/rules
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prometheus-server-conf
        - name: prometheus-storage-volume
          emptyDir: {}
        - name: prometheus-read-rules-volume
          configMap:
            defaultMode: 420
            name: prometheus-read-rules-conf

    
```

**~/content-kubernetes-prometheus-env/prometheus/prometheus-deployment.yml**
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:v2.2.1
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus/"
            - "--web.enable-lifecycle"
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus/
            - name: prometheus-storage-volume
              mountPath: /prometheus/
            - name: prometheus-read-rules-volume
              mountPath: /etc/prometheus/rules
        - name: watch
          image: weaveworks/watch:master-5b2a6e5
          imagePullPolicy: IfNotPresent
          args: ["-v", "-t", "-p=/etc/prometheus", "-p=/var/prometheus", "curl", "-X", "POST", "--fail", "-o", "-", "-sS", "http://localhost:9090/-/reload"]
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prometheus-server-conf

        - name: prometheus-storage-volume
          emptyDir: {}
```

2. go into `readrules` directory

**Kubernetes Control Plane**
```
cd content-kubernetes-prometheus-env
cd readrules
```

3. update prometheus config map

**Kubernetes Control Plane**
```
kubectl apply -f prometheus-config-map.yml
```

4. update prometheus read rules

**Kubernetes Control Plane**
```
kubectl apply -f prometheus-read-rules-map.yml
```

5. update prometheus deployment

**Kubernetes Control Plane**
```
kubectl apply -f prometheus-deployment.yml
```

6. Validate that changes are applied successfully

**Kubernetes Control Plane**
```
kubectl get pods -n monitoring
```

<img src="https://user-images.githubusercontent.com/6856382/223143305-7b0030a7-5e30-4c68-b944-1cb4702d6276.png">

#