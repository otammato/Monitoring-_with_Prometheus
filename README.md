# Monitoring_with_Prometheus

In this demo I will use Prometheus to monitor the target node_exporter application that is configured by scraping metrics endpoints of the node_exporter. Afterwards I set up a Python Flask application to emit metrics and deploy that application so that Prometheus can monitor it.

- Configure the targets for Prometheus to monitor
- Create queries to get the metrics about the target
- Determine the status of the targets
- Identify information about the targets and visualize it with graphs
- Instrument a Python Flask application to be monitored by Prometheus

I use Docker to run both Prometheus, and special Node Exporters, which will behave like servers that can be monitored. As a prerequisite, I will pull down the bitnami/prometheus:latest image and the bitnami/node-exporter image from Docker Hub. I use these images to run Prometheus and create three instances of node exporters to be monitored.

1. Use the following docker ```pull``` command to pull down the ```bitnami/node-exporter``` image from Docker Hub that you will use to simulate three servers being monitored.
