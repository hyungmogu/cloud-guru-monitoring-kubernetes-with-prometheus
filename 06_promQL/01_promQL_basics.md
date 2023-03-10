# PromQL Basics

1. PromQL has 4 data types
    1. Instant Vector
    2. Range Vector
    3. Scalar 
        - is a floating point numerical value
    4. String

2. Prometheus requires to-be-graphed data to be either scalar or instance vector

<img src="https://user-images.githubusercontent.com/6856382/223003532-49f0e80f-a784-43a3-824c-6d585f5640ac.png">

## Filtering result

- `PromQL` result can be filtered based on labels each data point is associated with

<img src="https://user-images.githubusercontent.com/6856382/223023113-763b1390-999d-48d7-a8e4-4a42f110050b.png">

- `PromQL` result can be filtered by multiple conditions using comma

<img src="https://user-images.githubusercontent.com/6856382/223024125-8891ef1e-29ab-4064-8b66-027d483e3bc8.png">

- `PromQL` result can be filtered using regexes
- `PromQL` result can be filtered using regexes by using =~ operator followed by regex expression

**Example**
```
node_cpu_seconds_total[job=~".*-exporter"]
```

<img src="https://user-images.githubusercontent.com/6856382/223025079-8577a966-9521-491d-ab02-62af0e61b25b.png">