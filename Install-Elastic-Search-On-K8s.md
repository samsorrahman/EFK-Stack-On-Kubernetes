# EFK-Stack-On-Kubernetes

#📜  # Deploying Elasticsearch on Kubernetes <br>
Welcome to the second lab of our hands-on course, "Learn by Doing - EFK Stack"! 🎉 In this lab, we'll guide you through the process of deploying Elasticsearch within a Kubernetes cluster. We'll cover setting up a StatefulSet for Elasticsearch, configuring Persistent Volumes for data storage, and using Services for network access. Each section builds upon the last, ensuring a comprehensive understanding of deploying Elasticsearch in a Kubernetes environment. 📚

🤔 Understanding Kubernetes and Elasticsearch
Before diving into the deployment process, let's quickly recap the basics of Kubernetes and Elasticsearch.

Kubernetes Overview
Kubernetes is an open-source platform designed to automate deploying, scaling, and operating application containers. It groups containers that make up an application into logical units for easy management and discovery.

Elasticsearch Overview
Elasticsearch is a powerful open-source search and analytics engine designed for horizontal scalability, reliability, and real-time search. It's commonly used for log or event data analysis and full-text search.
Using Elasticsearch in conjunction with Kubernetes, allows it to reap benefits from Kubernetes' self-healing and scalability features.

🏰 Setting Up a StatefulSet for Elasticsearch<br>
StatefulSets are ideal for stateful applications like Elasticsearch. They provide stable, unique network identifiers and persistent storage for each pod.
<br>
Why Use StatefulSet?<br>
Stable Network Identity: Each pod gets a unique and stable hostname.
Ordered, Graceful Deployment and Scaling: Pods are created and deleted in a predictable order.
Persistent Storage: Each pod can be associated with its storage, which persists across pod rescheduling.
Creating a StatefulSet
Below is a basic example of a StatefulSet definition for Elasticsearch. Note that you should adjust values such as volumeClaimTemplates according to your environment and needs.

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
spec:
  serviceName: "elasticsearch"
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.9.3
        ports:
        - containerPort: 9200
          name: es-http
        volumeMounts:
        - name: es-data
          mountPath: /usr/share/elasticsearch/data
  volumeClaimTemplates:
  - metadata:
      name: es-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

This YAML file defines a StatefulSet named elasticsearch with three replicas. It specifies the use of the Elasticsearch 7.9.3 Docker image and a persistent volume claim for data storage.

💾 Configuring Persistent Volumes for Data Storage
Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) are critical in managing storage in Kubernetes.

Understanding PVs and PVCs
Persistent Volume (PV): Represents a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.
Persistent Volume Claim (PVC): A request for storage by a user. It specifies size, and access modes, and can be linked to specific PVs.
Example: Persistent Volume Claim
The volumeClaimTemplates section in the StatefulSet YAML file automatically creates a PVC for each pod in the StatefulSet. Here's what happens behind the scenes:

```
volumeClaimTemplates:
- metadata:
    name: es-data
  spec:
    accessModes: [ "ReadWriteOnce" ]
    resources:
      requests:
        storage: 10Gi
```

This configuration requests a 10Gi volume with ReadWriteOnce access mode for each Elasticsearch pod. The Kubernetes cluster automatically provisions this storage if dynamic provisioning is set up or binds the PVC to a manually created PV.

🌐 Using Services for Network Access

🌐 Using Services for Network Access
Services in Kubernetes provide a way to expose an application running on a set of Pods as a network service.

Why Services?
Stable IP Addresses: Services provide stable IP addresses for pods.
Load Balancing: Services can load-balance traffic to multiple pods.
Service Discovery: Services allow other applications to discover and communicate with your Elasticsearch cluster through a stable endpoint.
Creating a Service for Elasticsearch
Here's an example of a Service definition for Elasticsearch. This service exposes the Elasticsearch HTTP port (9200) so that other applications can communicate with the cluster.


```
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
spec:
  selector:
    app: elasticsearch
  ports:
  - port: 9200
    targetPort: es-http
  type: ClusterIP
```

This YAML file creates a Service named elasticsearch, which targets pods with the label app: elasticsearch. It makes the Elasticsearch HTTP API accessible within the cluster through the cluster IP of the Service.

🎉 Conclusion
Deploying Elasticsearch on Kubernetes involves setting up a StatefulSet for stable pod identification and storage, configuring Persistent Volumes for data persistence, and using Services for stable network access. By following these steps, you can ensure a resilient and scalable Elasticsearch cluster within your Kubernetes environment. This guide aimed to provide a beginner-friendly introduction to deploying Elasticsearch on Kubernetes, covering the essential components and configurations.




# Run the Elastic Search for the first time

Throughout the lab, we'll apply fundamental Kubernetes concepts to ensure Elasticsearch runs efficiently. This involves creating StatefulSets for stable pod management, configuring Persistent Volume Claims (PVCs) to handle storage, and setting up Services to facilitate network access to the Elasticsearch cluster.

What is the primary purpose of using a StatefulSet for deploying Elasticsearch?
To provide a network identifier
Why are Persistent Volumes important for Elasticsearch in Kubernetes?
They ensure data persists across pods restarts
How does a Kubernetes Service benefit the Elasticsearch deployment?
It facilitates network access across pods
What is the significance of the volumeClaimTemplates in a StatefulSet?
To provide dynamic provisions
Which of the following is a correct statement about deploying Elasticsearch on Kubernetes?
Elastic search benefits from k8s self healing and scalability features.


# Tasks
Now, we will get started with deploying Elasticsearch on Kubernetes.

For this, first let's create a namespace called elastic-stack and switch to it.

Is the namespace elastic-stack created?

Did you switch to the elastic-stack namespace?

# Solution
To create a namespace named elastic-stack, use the following command:

kubectl create namespace elastic-stack
To switch to this namespace, use the following command:

kubectl config set-context --current --namespace=elastic-stack

## Task 2
As a next step, create a persistent volume with the following specifications:


name: pv-elasticsearch
storage capacity: 5Gi
accessModes: ReadWriteOnce
host path: /data/elasticsearch

Has the PV been created?

Solution
Create a file es-pvolume.yaml with the following contents:
```

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-elasticsearch
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/elasticsearch
Apply the configuration using the command:
kubectl apply -f es-pvolume.yaml
```

## task 3

Create a service named elasticsearch to expose the ElasticSearch StatefulSet within the cluster.

Since elasticsearch uses ports 9200 and 9300 as defaults for REST HTTP API and inter-node communication, respectively, add the following target ports to the service file:

9200: Name it as port1 and make it accessible on port 30200 of the node.

9300: Name it as port2 and make it accessible on port 30300 of the node.


Note: Specify the type of nodes as NodePort

Is the service elasticsearch created?

Is the service elasticsearch exposing ports 9200 and 9300?

Solution


Create a file es-service.yaml with the following contents:
```
kind: Service
apiVersion: v1
metadata:
  name: elasticsearch
  namespace: elastic-stack
spec:
  selector:
    app: elasticsearch
  ports:
  - port: 9200
    targetPort: 9200
    nodePort: 30200  
    name: port1

  - port: 9300
    targetPort: 9300
    nodePort: 30300  
    name: port2
  type: NodePort
```
Apply the Elasticsearch service yaml file using the following command:
kubectl apply -f es-service.yaml

## Task 4
Finally, create a StatefulSet named elasticsearch.

Use VolumeClaimTemplates to claim the Persistent Volume that we created in one of the previous steps. Access it using any label here, but ensure that the specs are same as the volume that we created earlier.

Refer to the Overview tab to mount this volume to the correct path within the container.

Use the following image for the elasticsearch container:
docker.elastic.co/elasticsearch/elasticsearch:7.1.0

Add the discovery.type environment variable to this container with the value single-node.

Also, do not forget to fix the permissions for the elasticsearch data directory so that the directory is accessible to elasticsearch.


For now, skip using replicas.

Note: The statefulset might take some time before it starts running.




Is the StatefulSet elasticsearch deployed?

Does the statefulset claim the persistent volume pv-elasticsearch?

Does the elasticsearch container use the specified environment variable?

Does the elasticsearch container use the correct image?

Is the statefulset running?

Solution:
Create a file es-statefulset.yaml with the following contents:

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: elastic-stack  
spec:
  serviceName: "elasticsearch"
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
        ports:
        - containerPort: 9200
          name: port1
        - containerPort: 9300
          name: port2
        env:
        - name: discovery.type
          value: single-node
        volumeMounts:
        - name: es-data
          mountPath: /usr/share/elasticsearch/data
      initContainers:
      - name: fix-permissions
        image: busybox
        command: ["sh", "-c", "chown -R 1000:1000 /usr/share/elasticsearch/data"]
        securityContext:
            privileged: true
        volumeMounts:
        - name: es-data
          mountPath: /usr/share/elasticsearch/data
  volumeClaimTemplates:
  - metadata:
      name: es-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
```
Apply the Elasticsearch StatefulSet YAML: kubectl apply -f /root/es-statefulset.yaml.







