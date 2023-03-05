# Configuring Prometheus to Use Service Discovery

1. once logging in, elevate permission to `root`

**Kubernetes Control Plane**
```
sudo su -
```

2. steup environment by running bootstrap shell program

**Kubernetes Control Plane**
```
cd /root/prometheus
sh ./bootstrap.sh
```

3. check to make sure all the pods are setup correctly

**Kubernetes Control Plane**
```
kubectl get pods -n monitoring
```

```
NAME                                     READY     STATUS    RESTARTS   AGE
kube-state-metrics-8548c449db-hzsm2      1/1       Running   0          46s      
prometheus-deployment-84697b66db-qgksz   1/1       Running   0          46s
```

4. edit `prometheus-config-map.yml` to create two service discovery targets
    - `kubernetes-apiservers` gets all the metrics from the API servers
    - `cadvisor` is an open-source agent integrated into the kubelet binary that monitors resource usage and analyzes the performance of containers
    - more information can be found [here](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)

**~/prometheus/prometheus-config-map.yml (before)**
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
```

**~/prometheus/prometheus-config-map.yml (after)**
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
            
        - job_name: `kubernetes-cadvisor`

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
```

- In the code above
    - `tls_config` means transport layer security configuration
    - `tls_config` is for connection to prometheus instance
    - `relabel_configs` is a tool to dynamically rewrite the label set of a target before it gets scraped
    - `relabel_configs` has 
        1. `source_labels` 
            - `source_labels` selects values from existing labels
        2. `action:labelmap`
            - `action:labelmap` matches regex against all source label names, not just thoese specified in `source_labels`.
            - `action:labelmap` copies the values of the matching labels to label names given by `replacement` with match group references `(${1}, ${2}, ...)` in `replacement` substituted by their value

5. Apply the changes to the prometheus configuration map

**kubernetes control plane**
```
kubectl apply -f prometheus-config-map.yml
```

6. Reload the prometheus pod by deleting it
- a. show all the pods under `monitoring` name space

**kubernetes control plane**
```
kubectl get pods -n monitoring
```

```
NAME                                     READY     STATUS    RESTARTS   AGE      
kube-state-metrics-8548c449db-hzsm2      1/1       Running   0          35m      
prometheus-deployment-84697b66db-qgksz   1/1       Running   0          35m 
```

- b. delete all the pods in here to make changes to take effect

**kubernetes control plane**
```
kubectl delete pods <pod_name> -n monitoring
```

7. open up a new brwoser and nagivate to the expression browser

```
http://<PUBLIC_IP>:8080
```

8. CLick status and select target from the dropdown. make sure two targets exists

<img src="https://user-images.githubusercontent.com/6856382/222984542-96f7dc0c-f404-4e59-b29b-c1a41f16c928.png">