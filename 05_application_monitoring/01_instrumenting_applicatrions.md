# Instrumenting Applications

<img src="https://user-images.githubusercontent.com/6856382/222981847-84bb2401-5225-4b3b-b80d-16fdba01811c.png">

- this part is about how to
    1. gather metrics from applications
    2. have the data get scrapped into Prometheus

## Instrumentation

- `instrumentation` is a fancy way of saying **we need to wrtie some code in our application to provide the metrics endpoint**
- `instrumentation` has a simple way which is using a client library

<img src="https://user-images.githubusercontent.com/6856382/222985854-419ed379-e881-4681-811a-81985f505a67.png">


## Swagger-stats
- `swagger-stats` is a third party client library
- `swagger-stats` is a api telemetry library for node.js
- `swagger-stats` has its own built in dashboard

<img src="https://user-images.githubusercontent.com/6856382/222985921-63c6d668-bff6-499d-9e46-1dcf56a77dec.png">

#