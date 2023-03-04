# Setting Up Prometheus

<img src="https://user-images.githubusercontent.com/6856382/222880850-496c0fdc-3d1f-4ca9-9801-c13075bdef1b.png">

## Instructions - Setting up Prometheus Cluster

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

## Instructions - Setting up Prometheus Configuration Map

1. setup prometheus confirguation map
- `configuration map` is an API object used to store non-confidental data in key-value pairs ([source](https://kubernetes.io/docs/concepts/configuration/configmap/))
- `configuration map` is a really convinent way of being able to go and decouple your configuration artifacts from your image content
- `configuration map` requires `<KUBERNETES_IP>` to IP **private ip** [IMPORTANT]

**Kubernetes control plane**
```
kubectl apply -f prometheus-config-map.yml
```

**/cloud-guru-monitoring-kubernetes-with-prometheus/prometheus/prometheus-config-map.yml**
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

2. Verify the config map is working

**Kubernetes Control Plane**
```
kubectl get configmaps  -n monitoring
```

```
NAME                     DATA   AGE
prometheus-server-conf   1      2m1s
```

**Kubernetes Control Plane**
```
kubectl describe configmaps  -n monitoring
```

```
Name:         prometheus-server-conf
Namespace:    monitoring
Labels:       name=prometheus-server-conf
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","data":{"prometheus.yml":"global:\n  scrape_interval: 5s\n  evaluation_interval: 5s\n\nscrape_configs:\n  - job_name: '...

Data
====
prometheus.yml:
----
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
    - targets: ['172.31.108.239:9100', '172.31.101.1:9100']

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
Events:  <none>
```

## Instructions - Setting up Prometheus Pod

**Kubernetes Control Plane**
```
kubectl apply -f prometheus-deployment.yml
```

**/cloud-guru-monitoring-kubernetes-with-prometheus/prometheus/prometheus-deployment.yml**
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
          emptyDir: {
```