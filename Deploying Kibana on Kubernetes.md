📜 Deploying Kibana on Kubernetes for Log Visualization
In the vast sea of data that modern applications generate, logs hold a treasure trove of insights. Visualizing and analyzing these logs can be like navigating through a dense fog without a compass. This is where Kibana, a powerful open-source visualization tool, comes into play. When paired with Elasticsearch for storage and search, and Kubernetes for orchestration, it forms a robust setup for managing application logs. This guide will walk you through deploying Kibana on Kubernetes to tap into the insights your logs offer.

⛪ Understanding Kibana
Imagine Kibana as the lens that brings the details of your data into focus, allowing you to navigate through the complexities with ease. It connects to Elasticsearch, where your logs are stored, and lets you create visualizations such as charts and graphs. These visualizations are then organized into dashboards, providing you with insights at a glance.

✍ Architecture Diagram
Diagram

![efkqjbr5jdfveglfieff](https://github.com/user-attachments/assets/e3ed2674-e15a-485e-ba63-496d0c18675d)


🚀 Deploying Kibana
Deploying Kibana involves creating a Deployment and a Service in Kubernetes. The Deployment ensures Kibana is running and manages replicas, while the Service makes Kibana accessible.

Creating the Kibana Deployment
Configuration File: Start by creating a configuration file for the Kibana Deployment. Let's name it kibana-deployment.yaml.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:7.10.0
        env:
          - name: ELASTICSEARCH_HOSTS
            value: "http://elasticsearch:9200"
        ports:
        - containerPort: 5601

```
This YAML file defines a Deployment named kibana with a single replica. It uses the official Kibana image and connects to Elasticsearch via the ELASTICSEARCH_HOSTS environment variable.
Kibana uses port 5601 by default and communicates with ELasticsearch using an HTTP REST API. To secure Kibana, the security features of Elasticsearch should be enabled.

Deploying Kibana: With the configuration file ready, deploy Kibana using kubectl.
kubectl apply -f kibana-deployment.yaml

Exposing Kibana Through a Service
After deploying Kibana, you need to make it accessible. This is done by creating a Kubernetes Service.

Service Configuration File: Create a file named kibana-service.yaml.

```
apiVersion: v1
kind: Service
metadata:
  name: kibana
spec:
  type: NodePort
  ports:
  - port: 5601
    targetPort: 5601
    nodePort: 30001
  selector:
    app: kibana

```

This Service exposes Kibana on a NodePort, making it accessible from outside the cluster.

Creating the Service: Apply the Service configuration using kubectl.
kubectl apply -f kibana-service.yaml

✨ Accessing Kibana
With Kibana deployed and exposed, you can now access its interface using the IP address of any node in your cluster and the node port specified in the Service configuration (e.g., http://<node-ip>:30001).

To explore the raw logs shipped by Fluentd, you can access the Discover tab from the menu. You can use the Kibana Query Language (KQL) to search your data. However, you will first be required to specify an index pattern in order to select the data that has to be explored.

After having explored the logs, you can create dashboards to aggregate your data from various search operations. You can also import or export dashboards. Kibana dashboards are exported in the .ndjson format.


# Tasks 

Create a deployment for Kibana in the elastic-stack namespace.


Use the following specs:

name: kibana
label: kibana
replicas: 1
image: docker.elastic.co/kibana/kibana:7.1.0
containerPort: 5601
Deploy Kibana on Kubernetes by applying this configuration file.

Note: Be sure to use the 'kubectl' command to apply the file.

Solution:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: elastic-stack
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:7.1.0
        ports:
        - containerPort: 5601
```
kubectl apply -f kibana-deploy.yaml

-----------------------------------------------------------------
Expose Kibana through a Kubernetes Service of type NodePort in the elastic-stack namespace.

Name the service as kibana.


Ensure the Service type is set to NodePort and the default kibana port 5601 is mapped to nodePort 30601.

Solution :
```
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: elastic-stack
spec:
  type: NodePort
  selector:
    app: kibana
  ports:
  - protocol: TCP
    port: 5601
    nodePort: 30601
```

kubectl apply -f kibana-service.yaml
