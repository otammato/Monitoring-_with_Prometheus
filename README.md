# Monitoring_with_Prometheus

In this demo I will use Prometheus to monitor the target node_exporter application that is configured by scraping metrics endpoints of the node_exporter. Afterwards I set up a Python Flask application to emit metrics and deploy that application so that Prometheus can monitor it.

- Configure the targets for Prometheus to monitor
- Create queries to get the metrics about the target
- Determine the status of the targets
- Identify information about the targets and visualize it with graphs
- Instrument a Python Flask application to be monitored by Prometheus
