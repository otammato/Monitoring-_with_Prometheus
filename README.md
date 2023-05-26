# Monitoring_with_Prometheus

In this demo I will use Prometheus to monitor the target node_exporter application that is configured by scraping metrics endpoints of the node_exporter. Afterwards I set up a Python Flask application to emit metrics and deploy that application so that Prometheus can monitor it.

- Configure the targets for Prometheus to monitor
- Create queries to get the metrics about the target
- Determine the status of the targets
- Identify information about the targets and visualize it with graphs
- Instrument a Python Flask application to be monitored by Prometheus

## Prerequisites

I use Docker to run both Prometheus, and special Node Exporters, which will behave like servers that can be monitored. As a prerequisite, I will pull down the bitnami/prometheus:latest image and the bitnami/node-exporter image from Docker Hub. I use these images to run Prometheus and create three instances of node exporters to be monitored.

1. Use the following docker ```pull``` command to pull down the ```bitnami/node-exporter``` image from Docker Hub that we will use to simulate three servers being monitored.

```
docker pull bitnami/node-exporter:latest
```
<img width="591" alt="Screenshot 2023-05-26 at 22 47 53" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/d1b566f2-7d74-4892-be73-c0bb31f01cde">

2. Then, pull the Prometheus docker image.

```
docker pull bitnami/prometheus:latest
```

<img width="608" alt="Screenshot 2023-05-26 at 22 50 35" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/29bd3076-f045-408e-8ef8-1503f30afa1f">

## Step 1: Start the first node exporter

1. ```docker network``` command to create a network called ```monitor``` within which we will run all of the docker containers.

```
docker network create monitor
```

2. ```docker run``` command to start a node exporter instance on the ```monitor``` network, listening at port ```9101``` externally and forwarding to port ```9100``` internally.

```
docker run -d --name node-exporter1 -p 9101:9100 --network monitor bitnami/node-exporter:latest
```
<img width="923" alt="Screenshot 2023-05-26 at 23 11 15" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/cf8334c0-7b20-4df4-9443-c22725a70c61">

3. Check if the instance is running on port 9101:

You should see the Node Exporter page open up with a hyperlink to Metrics. These are the metrics the Prometheus instance is going to monitor.

<img width="1064" alt="Screenshot 2023-05-26 at 22 59 39" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/f49da46a-f39e-43fd-b8d5-5501c4bc2f2d">

4. Finally, click the Metrics link to see the metrics.

<img width="1055" alt="Screenshot 2023-05-26 at 23 02 27" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/5f013e6c-4eca-407f-8bf5-3b36cb3b796a">

## Step 2: Start two more node exporters

1. In the terminal, run the following commands to start two more instances of node exporter.
