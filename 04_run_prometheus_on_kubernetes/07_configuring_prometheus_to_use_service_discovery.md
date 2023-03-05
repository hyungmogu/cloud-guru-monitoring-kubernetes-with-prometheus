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
        - job_name: `kubernetes-cadvisor`
```

