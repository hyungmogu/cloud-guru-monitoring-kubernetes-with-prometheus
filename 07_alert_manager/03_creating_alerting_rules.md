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
- More information about alerting rules, and the use of `$labels` can be found [here](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)


**~/prometheus/prometheus-rules-config-map.yml**
```

```


## Notes

1. `configMap` is an api object used to store non-confidential data
2. `configMap` is consumed by pod as 
    1. environmental variables
    2. command-line arguements
    3. configuration files in a volume

