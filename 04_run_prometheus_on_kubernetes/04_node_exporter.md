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

**All Nodes**
```
adduser prometheus
```

2. Download Node Exporter

**All Nodes**
```
cd /home/prometheus
curl -LO "https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz"
tar -xvzf node_exporter-0.16.0.linux-amd64.tar.gz
mv node_exporter-0.16.0.linux-amd64 node_exporter
cd node_exporter
chown prometheus:prometheus node_exporter
```

3. Create a file that allows node exporter to start as a daemon (background task)

**All Nodes**
```
vim /etc/systemd/system/node_exporter.service
```

**/etc/systemd/system/node_exporter.service**
```
[Unit]
Description=Node Exporter

[Service]
User=prometheus
ExecStart=/home/prometheus/node_exporter/node_exporter

[Install]
WantedBy=default.target
```

4. Reload daemon

**All Nodes**
```
systemctl daemon-reload
```

5. Enable node exporter service

**All Nodes**
```
systemctl enable node_exporter.service
```

6. Start node exporter service

**All Nodes**
```
systemctl start node_exporter.service
```

7. Check node_exporter.service is running properly

**All Nodes**
```
systemctl status node_exporter.service
```


#