# Scrapping

<img src="https://user-images.githubusercontent.com/6856382/222879182-72a387c1-28e3-41d6-9c45-ceb172342cda.png">

<img src="https://user-images.githubusercontent.com/6856382/222877864-c2812e76-3cbb-455a-b12e-ccb31e33e858.png">

## Pull-based System
- `Pull-based system` decides when and what needs to be scraped
- `Pull-based system` is what prometheus is based on
- `Pull-based system` is performed in prometheus using http endpoints

## Push-based System
- `Push-based system` uses monitoring system
- `Push-based system` uses monitoring target to determine when it triggers and sends data to server

## Prometheus and Pull-based System
- `Prometheus` makes a request that is being made to http endpoint
- `Prometheus` parses and ingests received data to storage