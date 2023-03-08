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

#