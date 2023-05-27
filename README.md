# Monitoring_with_Prometheus

In this demo we will use Prometheus to monitor the target node_exporter application that is configured by scraping metrics endpoints of the node_exporter. Afterwards we set up a Python Flask application to emit metrics and deploy that application so that Prometheus can monitor it.

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

```
docker run -d --name node-exporter2 -p 9102:9100 --network monitor bitnami/node-exporter:latest
```
```
docker run -d --name node-exporter3 -p 9103:9100 --network monitor bitnami/node-exporter:latest
```

2. Now, check if all the instances of node exporter are running by using the ```docker ps``` command and pipe it through the grep command to search for ```node-exporter```

```
docker ps | grep node-exporter
```

If everything started correctly, you should see output similar to the following coming back from the ```docker ps``` command:

<img width="1171" alt="Screenshot 2023-05-26 at 23 23 22" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/5aae71c3-370c-4e0c-82b7-43b15d118bb6">

## Step 3: Configure and run Prometheus

1. Use the ```touch``` command to create a file named prometheus.yml in the current directory. This is the file where you will configure the Prometheus to monitor the node exporter instances.

```
touch /home/ubuntu/prometheus.yml
```
2. Then, copy and paste the following configuration contents into the yaml file and save it:

```yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. The default is every 1 minute.

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter1:9100']
        labels:
          group: 'monitoring_node_ex1'
      - targets: ['node-exporter2:9100']
        labels:
          group: 'monitoring_node_ex2'
      - targets: ['node-exporter3:9100']
        labels:
          group: 'monitoring_node_ex3'
```
> Notice that while you access the node exporters externally on ports 9101, 9102, and 9103, they are internally all listening on port 9100, which is how Prometheus will communicate them on the monitor network.

- Set the scrape_interval to 15 seconds instead of the default of 1 minute. This is so that we can see results quicker during the lab, but the 1 minute interval is better for production use.
- The scrape_config section contains all the jobs that Prometheus is going to monitor. These job names have to be unique. You currently have one job called node. Later we will add another to monitor a Python application.
- Within each job, there is a static_configs section where you define the targets and define labels for easy identification and analysis. These will show up in the Prometheus UI under the Targets tab.
- The targets you enter here point to the base URL of the service running on each of the nodes. Prometheus will add the suffix /metrics and call that endpoint to collect the data to monitor from. (For example, node-exporter1:9100/metrics)

3. Finally, you can launch the Prometheus monitor in the terminal by executing the following ```docker run``` command passing the yaml configuration file as a volume mount with the ```-v``` parameter.

```
sudo docker run -d --name prometheus2 -p 9091:9090 --network monitor -v $(pwd)/prometheus.yml:/opt/bitnami/prometheus/conf/prometheus.yml bitnami/prometheus:latest
```

> Note: This Dockerized distribution of Prometheus from Bitnami expects its configuration file to be in the ```/opt/bitnami/prometheus/conf/prometheus.yml``` file, which is why you are mapping your ```prometheus.yml``` file to this location. Other distributions may look in other locations. Always check the documentation to be sure of where to mount the configuration file.

> Note: to see the process currently using the port ```sudo lsof -i :9091```
> <img width="522" alt="Screenshot 2023-05-27 at 09 03 48" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/843a8c30-cee4-4de7-ac9e-67435671272d">

You should see just the Prometheus container id returned, indicating that Docker has started Prometheus in the background.

<img width="951" alt="Screenshot 2023-05-27 at 09 06 16" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/31c70352-7dce-43d6-80f6-de667801678e">

## Step 4: Open the Prometheus UI

1. Open the Prometheus web UI by using the Public IPv4 DNS or just IP of your server plus the port assigned (9091 in our case, MAKE SURE IT IS OPEN). The Prometheus application UI opens up by default in the graph endpoint. 

<img width="911" alt="Screenshot 2023-05-27 at 09 14 47" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/6ff6a3f5-cc64-428e-8ba5-e71c9f15f86f">

2. Next, in the Prometheus application, click Status on the menu and choose Targets to see which targets are being monitored.

<img width="911" alt="Screenshot 2023-05-27 at 09 20 37" src="https://github.com/otammato/Monitoring_with_Prometheus/assets/104728608/e56863d4-8b72-47dd-b29e-feee96a895ad">
