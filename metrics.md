# OpenRMF Metrics with Prometheus and Grafana

OpenRMF tracks metrics used by Prometheus starting with version 0.10.7. We currently use the https://github.com/prometheus-net/prometheus-net component for the .NET Core Web APIs and the https://github.com/nats-io/prometheus-nats-exporter container for exporting NATS 2.0 metrics out to a Prometheus endpoint. With that information, you can setup the below Grafana dashboards to show usage, memory, requests, errors, and the like. 

The docker-compose file for running OpenRMF locally uses a container definition for Prometheus and Grafana locally. The Kubernetes deployment does not. For Kubernetes you can hook to your existing Prometheus and Grafana setup. Or you can deploy separately to namespaces within Kubernetes and then setup the data sources there.

## .NET Core Default Metrics
Once you have Prometheus setup, you can use the https://grafana.com/grafana/dashboards/10427 to display .NET Core metrics like the below chart. 

![Image](./img/metrics/metrics-aspnet-core-default.png?raw=true)

## .NET Core API Controller Metrics
Once you have Prometheus setup, you can use the https://grafana.com/grafana/dashboards/10915 to display .NET Core metrics like the below chart. 

![Image](./img/metrics/metrics-api-controller-summary.png?raw=true)

## NATS Server Metrics
Once you have Prometheus setup and you deploy the container for https://github.com/nats-io/prometheus-nats-exporter, you can use the https://grafana.com/grafana/dashboards/2279 to display NATS Server Core metrics like the below chart. 

![Image](./img/metrics/metrics-nats-server.png?raw=true)

## Prometheus Configuration Setup
Below is the prometheus.yml file configuration we use when running the local or docker-compose setup of OpenRMF. You can adjust 
the interval and options as required. Just restart the `docker-compose up -d` command to relaunch and use the new configuration.

```
global:
  scrape_interval:     15s # By default, scrape targets every 5 seconds.

# A scrape configuration containing exactly one endpoint to scrape:
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'nats-openrmf-server'
    static_configs:
      - targets: ['natspromexporter:7777']
  - job_name: 'openrmf-api-read-prometheus'
    # metrics_path defaults to '/metrics'
    static_configs:
    - targets: ['openrmfapi-read:8084']
  - job_name: 'openrmf-api-save-prometheus'
    # metrics_path defaults to '/metrics'
    static_configs:
    # replace the IP with your local IP for development
    # localhost is not it, as that is w/in the container :)
    - targets: ['openrmfapi-save:8082']
  - job_name: 'openrmf-api-template-prometheus'
    # metrics_path defaults to '/metrics'
    static_configs:
    # replace the IP with your local IP for development
    # localhost is not it, as that is w/in the container :)
    - targets: ['openrmfapi-template:8088']
  - job_name: 'openrmf-api-controls-prometheus'
    # metrics_path defaults to '/metrics'
    static_configs:
    # replace the IP with your local IP for development
    # localhost is not it, as that is w/in the container :)
    - targets: ['openrmfapi-controls:8094']
  - job_name: 'openrmf-api-compliance-prometheus'
    # metrics_path defaults to '/metrics'
    static_configs:
    # replace the IP with your local IP for development
    # localhost is not it, as that is w/in the container :)
    - targets: ['openrmfapi-compliance:8092']
  - job_name: 'openrmf-api-scoring-prometheus'
    # metrics_path defaults to '/metrics'
    static_configs:
    # replace the IP with your local IP for development
    # localhost is not it, as that is w/in the container :)
    - targets: ['openrmfapi-scoring:8090']
  - job_name: 'openrmf-api-upload-prometheus'
    # metrics_path defaults to '/metrics'
    static_configs:
    # replace the IP with your local IP for development
    # localhost is not it, as that is w/in the container :)
    - targets: ['openrmfapi-upload:8086']
```

## Additional Links

Prometheus: https://prometheus.io/docs/prometheus/latest/querying/basics/

Grafana:  https://grafana.com/