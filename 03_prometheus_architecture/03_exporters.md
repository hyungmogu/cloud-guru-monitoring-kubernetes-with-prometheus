# Exporters

<img src="https://user-images.githubusercontent.com/6856382/222655292-1a292e17-ac1e-453e-8aa2-cc20eb66b2c7.png">

<img src="https://user-images.githubusercontent.com/6856382/222471318-b998eec7-b0fd-4818-858c-e5848be4e63b.png">

- `exporters` are used to go and instrument applications that we don't have source code for
- `exporters` live next to the application that we want to collect metrics from


## Types of Exporters

1. Database exporters
    - includes
        1. Console
        2. Memcache
        3. mySQL

2. Node Exporter
    - includes
        1. Hardware metrics
        2. Software metrics

3. HAProxy Service
    - includes
        1. messaging
        2. storage
        3. HTTP, inlcuding HAProxy

**Example**
- `Node Exporter` is used to gather data about kubernetes cluster

## How it works

1. `exporters` take requests from prometheus
2. `exporters` are going to go and gather the data
3. `exporters` formats it
4. `exporters` returns results back to prometheus




