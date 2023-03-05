# Node Exporter

<img src="https://user-images.githubusercontent.com/6856382/222939962-19f24cec-375c-45f0-842f-e5319d92fea4.png">

1. `node exporter` is a way of exposing hardware and OS metrics to prometheus
2. `node exporter` is scraped by Prometheus through endpoint
3. `node exporter` returns to prometheus:
    1. CPU Metrics
    2. Memory
    3. Disk Space
4. `node exporter` requires to make sure we are not running as root


## Instruction - Setting Up Node Exporter

1. Add user `prometheus`

**Kubernetes Control Plane**
```
adduser prometheus
```

