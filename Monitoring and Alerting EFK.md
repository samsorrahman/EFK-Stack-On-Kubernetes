📜 Monitoring and Alerting for the EFK Stack on Kubernetes
In this guide, we will explore how to set up monitoring and alerting for the Elasticsearch, Fluentd, and Kibana (EFK) stack components on Kubernetes. We'll integrate with Kubernetes monitoring solutions like Prometheus and Grafana for metrics collection and visualization. Additionally, we'll configure alerting based on log data and performance metrics. This guide is designed to be beginner-friendly, breaking down complex concepts with simple analogies and practical examples.

🔬 Introduction to the EFK Stack
The EFK stack is a popular logging solution for Kubernetes, consisting of Elasticsearch for storage, Fluentd for log aggregation, and Kibana for visualization. It's essential for debugging applications and monitoring cluster health.

Why Monitor the EFK Stack?
Monitoring the EFK stack helps ensure that your logging system is performing optimally. It allows you to detect issues early, such as Elasticsearch running out of storage or Fluentd experiencing high latency.

![wzrmohzi1rai7hhqpdlt](https://github.com/user-attachments/assets/03ce7206-875f-4889-a040-26b412799cbc)

🧷 Integrating Prometheus with EFK
Prometheus is an open-source monitoring solution for collecting and storing metrics. Integrating Prometheus with the EFK stack enables you to collect metrics about the performance and health of each component.
Prometheus allows per job scrape interval configuration that provides great flexibility in terms of which targets to scrape and how frequently.

Setting Up Prometheus
First, install Prometheus in your Kubernetes cluster. You can use the Prometheus Operator for easier management.

```
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml
```
Configuring Prometheus to Monitor EFK Components
Elasticsearch Metrics: Use the elasticsearch-exporter to expose Elasticsearch metrics to Prometheus.
```
   apiVersion: monitoring.coreos.com/v1
   kind: ServiceMonitor
   metadata:
     name: elasticsearch-exporter
     labels:
       team: backend
   spec:
     selector:
       matchLabels:
         app: elasticsearch-exporter
     endpoints:
     - port: http
```
Fluentd Metrics: Fluentd metrics can be exposed using the built-in Prometheus plugin.
```
   <source>
     @type prometheus
     port 24231
   </source>
```
Kibana Metrics: Kibana does not natively export Prometheus metrics, but you can use the kibana-prometheus-exporter plugin.
However, note that sensitive information and configuration data like secrets are not typically monitored using Prometheus.

📽 Visualizing Metrics with Grafana
Grafana is a powerful tool for visualizing metrics. Once Prometheus is collecting metrics from the EFK stack, you can use Grafana to create dashboards.

Install Grafana:

```
   kubectl create -f grafana.yaml
```
where the yaml file would contain the required components for deployment:
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        fsGroup: 472
        supplementalGroups:
          - 0
      containers:
        - name: grafana
          image: grafana/grafana:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
              name: http-grafana
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /robots.txt
              port: 3000
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 30
            successThreshold: 1
            timeoutSeconds: 2
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 3000
            timeoutSeconds: 1
          resources:
            requests:
              cpu: 250m
              memory: 750Mi
          volumeMounts:
            - mountPath: /var/lib/grafana
              name: grafana-pv
      volumes:
        - name: grafana-pv
          persistentVolumeClaim:
            claimName: grafana-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
spec:
  ports:
    - port: 3000
      protocol: TCP
      targetPort: http-grafana
  selector:
    app: grafana
  sessionAffinity: None
  type: LoadBalancer

```
Note: This is a sample snippet. For attempting the tasks, use the URLs provided in the respective task statements.

Configure Grafana DataSource:

Add Prometheus as a DataSource in Grafana to start creating dashboards for your EFK metrics.
Grafana supports multiple other datasources like Elasticserch, Graphite, InfluxDB, MySQL and even Google sheets.

A useful feature in Grafana is templating which allows creation of dynamic and reusable dashboards.

🕬 Configuring Alerting
Alerting is crucial for being proactively notified about issues in your EFK stack before they escalate.

Alerting with Prometheus Alertmanager
Prometheus Alertmanager can manage alerts sent by Prometheus. Define alert rules based on EFK metrics.

Define Alert Rules:

For example, create an alert if Elasticsearch's heap usage is too high.

```
   groups:
   - name: elasticsearch-alerts
     rules:
     - alert: ElasticsearchHighHeapUsage
       expr: elasticsearch_jvm_memory_used_bytes / elasticsearch_jvm_memory_max_bytes > 0.9
       for: 5m
       labels:
         severity: critical
       annotations:
         summary: "High Heap Usage (instance {{ $labels.instance }})"
         description: "Elasticsearch node {{ $labels.node }} heap usage is above 90%."
```
Configure Alertmanager:

Set up Alertmanager to send notifications through email, Slack, or other channels when alerts are triggered.
```
   global:
     resolve_timeout: 5m
   route:
     receiver: 'slack-notifications'
   receivers:
   - name: 'slack-notifications'
     slack_configs:
     - channel: '#alerts'
```
🎉 Conclusion
Congratulations! You've successfully deployed multiple tools to monitor the EFK stack on Kubernetes.

Monitoring and alerting are essential components of maintaining a healthy EFK stack on Kubernetes. By integrating Prometheus and Grafana, you gain visibility into the performance and health of Elasticsearch, Fluentd, and Kibana.

Configuring alerting with Prometheus Alertmanager ensures you are promptly notified of potential issues, allowing for proactive management of your logging infrastructure.

This guide provides just a starting point for securing and optimizing your EFK stack through effective monitoring and alerting strategies.

--------------------------------------------------------------------

## Tasks
Install and run the Elasticsearch exporter to scrape metrics from Elasticsearch and send them to Prometheus.


Download the exporter from here: https://github.com/prometheus-community/elasticsearch_exporter/releases

Note: Elasticsearch is accessible on port 30200.
Solution:

Download the Elasticsearch exporter using the wget command:
```
wget https://github.com/prometheus-community/elasticsearch_exporter/releases/download/v1.7.0/elasticsearch_exporter-1.7.0.linux-amd64.tar.gz
```
Then extract the newly downloaded .tar.gz extension file:
```
tar xzf elasticsearch_exporter-1.7.0.linux-amd64.tar.gz -C ./
```
A folder titled elasticsearch_exporter-1.7.0.linux-amd64 will be visible under the root directory. Navigate within this folder and you will see few files including a deployment.yml file and a binary file.

Let's proceed with the binary approach for running elasticsearch exporter. Run the following command:

```
nohup ./elasticsearch_exporter --es.uri="http://localhost:30200" &
```

Ensure to run the above command in the background.

By default, the exporter exposes metrics at port 9114. Curl the localhost at this port to verify if metrics are being exposed:

```
curl http://localhost:9114/metrics
```

----------------------------------------------------------------------------------------------
Now that we have our Elasticsearch exporter running and exposing metrics, let's install prometheus to scrape these metrics.

From the Prometheus downloads page, download the prometheus binary and extract it.

Configure the prometheus.yml file and run prometheus so that it scrapes the metrics being exposed by the elasticsearch exporter at port 9114.

Solution
```
wget https://github.com/prometheus/prometheus/releases/download/v2.51.1/prometheus-2.51.1.linux-amd64.tar.gz

```
Then extract the newly downloaded .tar.gz extension file:
```
tar xzf prometheus-2.51.1.linux-amd64.tar.gz -C ./

```
A folder titled prometheus-2.51.1.linux-amd64 will be visible under the root directory. Navigate within this folder and you will see few binaries and a prometheus.yml file.

Edit the prometheus.yml file and append the following to the scrape_configs section:

```
- job_name: "elasticsearch"
  static_configs:
  - targets: ["localhost:9114"]
```

Here, we have added a new job named elasticsearch to prometheus, to scrape metrics from port 9114 on our host.

Let's run the prometheus binary using our prometheus.yml file as the config:
```
nohup ./prometheus --config.file=prometheus.yml &
```

Ensure to run the above command in the background.

By default, prometheus runs at port 9090. Using the View Port utility on the top right of this page, access this port.
Once on the Prometheus query page, navigate to Status -> Targets. Here, you can see two jobs in the UP state:
prometheus at port 9090
elasticsearch at port 9114
-------------------------------------------------------------------------------------------------------------

Install Grafana on your Kubernetes cluster to visualize Elasticsearch metrics from Prometheus.

Use the official Grafana downloads page for the Linux platform: https://grafana.com/grafana/download?platform=linux

Extract this file and run the grafana-server binary in the background.


Default username and password for Grafana is admin. Use it to login for the first time, and then change the password to grafana_user123.

Default port for Grafana is 3000.

Solution

Download Grafana using the wget command:
```
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-10.4.1.linux-amd64.tar.gz

```
Then extract and unzip the newly downloaded .tar.gz extension file:

```
tar xzf grafana-enterprise-10.4.1.linux-amd64.tar.gz -C ./
```
A folder titled grafana-v10.4.1 will be visible under the root directory. Navigate within this folder and inspect the bin directory. This directory contains the binary file grafana-server that we will be using to run our grafana instance.

Execute the following command from within the grafana-v10.4.1 folder:

```
nohup ./bin/grafana-server &

```
Grafana runs at port number 3000 by default. Navigate to this port using the View Port utility at the top right of this page.

Enter admin as both the username and password in the Grafana login page. Next, you will be asked to update the password. Update it to grafana_user123.

Open the menu on the left of the Grafana Welcome page under Home. Under the Connections submenu, click on Data sources -> Add data source and select Prometheus.

Under Connection, enter http://localhost:9090/ for Prometheus server URL. Scroll to the bottom of this page and click on Save and Test.

---------------------------------------------------------------------------------------------------------------------------------

Having set up our basic observability stack for the elasticsearch metrics, let's learn how to query our metrics in the Grafana dashboard.

Select Explore from the Grafana menu. You will be presented with a workspace to create and run your queries. Prometheus as a data source has already been selected for you as a result of your previous task.

There are two ways through which you can create a query here - either by using the builder or by using the code.

Let's select builder for now and view the value of a simple metric. From the Metric dropdown, select elasticsearch_cluster_health_active_primary_shards and cluster and docker-cluster as the key-value pair under Label filters. After clicking on Run query, you can view the value of the current number of primary shards in the cluster that are active and assigned to nodes.

You can also select other metrics, and create a dashboard for visualizing the vitals of your Elasticsearch components.

This dashboard can be used as a reference:
https://grafana.com/grafana/dashboards/14191-elasticsearch-overview/

Reference for elasticsearch metrics:
https://github.com/prometheus-community/elasticsearch_exporter?tab=readme-ov-file#configuration


----------------------------------------------------------------------------------------------------------
Define an alert rule in Prometheus for high Elasticsearch heap usage.

Name the alert as HighElasticsearchHeapUsage.

The alert should trigger if the heap usage is above 5% for more than 2 minutes, and its severity should be critical.

Place the alert rule file inside the extracted prometheus folder.


Refer to the Description page to view the syntax of an alert rules file.

Note: This low heap usage percentage is just for demonstration purpose. In an actual setting, you would want this to be atleast 90%.

Solution
Navigate to the prometheus-2.51.1.linux-amd64 folder and create a file named rules.yml with the following content:
```
groups:
- name: elasticsearch
  rules:
  - alert: HighElasticsearchHeapUsage
    expr: elasticsearch_jvm_memory_used_bytes{job="elasticsearch"} / elasticsearch_jvm_memory_max_bytes{job="elasticsearch"} > 0.05
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High Elasticsearch heap usage (instance {{ $labels.instance }})"
      description: "Elasticsearch instance {{ $labels.instance }} has high heap usage ({{ $value }} bytes used) for more than 2 minutes."
```
Edit the prometheus.yml file and add the newly created rules file to it under the rule_files section:
```
rule_files:
   - "rules.yml"
```
Save and exit.

Identify the PID of the prometheus process:
```
ps aux | grep prometheus
```
and kill the process:
```
kill -9 <pid>
```
Re-start the process in the background so that it picks the updated file this time:
```
nohup ./prometheus --config.file=prometheus.yml &
```
In the Prometheus UI, navigate to the Alerts section. You will find your defined alert HighElasticsearchHeapUsage. This will remain in pending state for upto two minutes, and then go into the firing state.


--------------------------------------------------------------------------------------------------------------------------------------------------
Let's now download and run alertmanager to deal with the alert that we configured in the previous task.

Use the official Prometheus downloads page to download alertmanager:
https://prometheus.io/download/

Run the alertmanager binary using the default configuration file.

After the alerts are visible on alertmanager, to demonstrate how these alerts can be redirected using alertmanager, run the python script present in the /root/scripts path in a new terminal. You must see all the alerts received so far.


Default port for Alertmanager is 9093.

Here we have redirected the alerts to a local HTTP server. Refer to this document to learn how to redirect to other receiver targets:
https://prometheus.io/docs/alerting/latest/configuration/#receiver-integration-settings


Solutions:

Download alertmanager using the wget command:
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
```
Then extract the newly downloaded .tar.gz extension file:
```
tar xzf alertmanager-0.27.0.linux-amd64.tar.gz -C ./
```
A folder titled alertmanager-0.27.0.linux-amd64 will be visible under the root directory. Navigate within this folder and you will see few binaries and an alertmanager.yml file.

Inspect this file but do not make any changes to it. You can see that the receiver is mentioned as the port 5001 of our machine. Run the following command to start alertmanager at the default port 9093:
```
nohup ./alertmanager --config.file=alertmanager.yml &
```
Identify the PID of the prometheus process:
```
ps aux | grep prometheus
```
and kill the process:
```
kill -9 <pid>
```
Navigate to the prometheus-2.51.1.linux-amd64 folder and edit the prometheus.yml file to include alertmanager under the alerting section:
```
alerting:
   alertmanagers:
      - static_configs:
            - targets: ["localhost:9093"]
```
Re-start the process in the background so that it picks the updated configuration:
```
nohup ./prometheus --config.file=prometheus.yml &
```
Using the View Ports utility, access port number 9093. You can see your HighElasticsearchHeapUsage alert on the Alerts page.

Open a new terminal, navigate to /root/scripts and run the python script:
```
python3 alert_receiver.py
```
You can see all the alerts on your terminal now.
