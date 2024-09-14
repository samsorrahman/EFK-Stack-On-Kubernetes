🚿 Deploying Fluentd on Kubernetes as a DaemonSet
Welcome to the third lab of our hands-on course! 🎉 In this lab, we'll explore how to deploy Fluentd on Kubernetes as a DaemonSet. This ensures that a Fluentd instance runs on every node, enabling efficient log collection from both the nodes and the pods. We'll also cover how to configure Fluentd to forward these logs to Elasticsearch, providing a centralized logging solution. 📚

🤔 Introduction to Fluentd and Kubernetes
Before we dive into the deployment process, let's understand the key components involved.

What is Fluentd?
Fluentd is an open-source data collector designed for processing logs and events. It's highly flexible, allowing you to collect data from multiple sources, transform it as needed, and send it to various destinations.

Why Kubernetes?
Kubernetes is a container orchestration platform that automates the deployment, scaling, and management of containerized applications. By deploying Fluentd on Kubernetes, we can harness its scalability and management features for efficient log collection.

🚀 Preparing Your Kubernetes Cluster
Ensure your Kubernetes cluster is up and running by executing the following command in the attached terminal:

kubectl get nodes
You should see controlplane in READY state in the output.

🌿 Deploying Fluentd as a DaemonSet
A DaemonSet ensures that a copy of the pod runs on each node in the cluster. This is perfect for log collection, as we want Fluentd to collect logs from every node.

Creating the Fluentd DaemonSet
Create a Fluentd Configuration File
First, create a Fluentd configuration file (say fluentd-config.conf) that specifies how logs should be collected and forwarded.

```
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<match **>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
</match>
```
Here, source and match are directives. Following is a list of all directives and the respective aspects of logging they relate to:

source: Input sources.
match: Output destinations.
filter: Event processing pipelines.
system: System-wide configuration.
label: Group the output and filter for internal routing.
worker: Limit to the specific workers.
@include: Include other files.
Apart from directives, fluentd also uses various input and output plugins.

Define the DaemonSet
Next, create a Kubernetes DaemonSet definition (say fluentd-daemonset.yaml). This file describes the Fluentd pod that will be deployed to each node.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.11-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch.default.svc.cluster.local"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: dockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: dockercontainers
        hostPath:
          path: /var/lib/docker/containers
```


Deploy the DaemonSet
Apply the DaemonSet definition to your cluster.

kubectl apply -f fluentd-daemonset.yaml
⚙️ Configuring Fluentd for Kubernetes Logs
Fluentd needs to be configured to collect logs from both Kubernetes nodes and pods. The configuration we defined earlier in fluentd-config.conf sets Fluentd to listen for logs and forward them to Elasticsearch.

Collecting Node Logs
The DaemonSet configuration mounts the /var/log directory from the host into the Fluentd container, allowing Fluentd to access and collect system and application logs from the nodes.

Collecting Pod Logs
Similarly, mounting /var/lib/docker/containers enables Fluentd to collect logs from Docker containers, including those managed by Kubernetes pods.

Forwarding Logs to Elasticsearch
The Fluentd configuration specifies Elasticsearch as the destination for logs. Ensure that Elasticsearch is running and accessible from within your Kubernetes cluster.

```
<match **>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
</match>
```


The above configuration forwards all collected logs to Elasticsearch, where they can be indexed and made searchable.

This has been made possible because of the use of output plugin fluent-plugin-elasticsearch which is used to forward data to Elasticsearch. The match section displays the parameters it uses.

🎉 Conclusion
Deploying Fluentd as a DaemonSet on Kubernetes allows for efficient log collection across all nodes and pods. By forwarding these logs to Elasticsearch, you gain a powerful tool for log analysis and monitoring. This setup is crucial for maintaining insight into the health and performance of your Kubernetes applications.

Remember, the configurations shown here can be customized to suit your specific logging needs. Happy logging! 🌿


# Tasks and Solutions
In this lab, we'll explore the deployment of Fluentd on Kubernetes as a DaemonSet. This approach ensures efficient log collection across all nodes and pods, forwarding them to Elasticsearch for centralized logging and analysis. This setup is vital for maintaining insights into the health and performance of Kubernetes applications.
Fluentd is an open-source data collector for unified logging layers, allowing you to collect data from different sources, transform it, and send it to multiple destinations, such as Elasticsearch, making it a versatile tool for log management in complex environments like Kubernetes.

What is the main reason to deploy Fluentd as a DaemonSet in a Kubernetes cluster?
to run fluented on each node for log collection
What is the primary purpose of a DaemonSet in Kubernetes?
to replicate pos accrose every node in the cluster

Which plugin is used by Fluentd to forward logs to Elasticsearch?
fluent-plugin-elasticsearch


## Tasks
Check the current namespace. It should be elastic-stack. If not, then switch to it.

In this namespace, we have deployed two pods named app and elasticsearch-0.

The elasticsearch-0 pod is the same that we configured in the previous lab. Here, it has been pre-configured for you.

The app pod is generating a log file at location /log inside the container, periodically entering log lines using an event simulator.

Now, let's configure fluentd log shipper so that the container logs are shipped to elasticsearch.

Right now, no logs are being forwarded to Elasticsearch. You can check this by clicking on Elasticsearch on the top right of this page, and then adding /_search?q=*:*&pretty at the end of the resultant URL.

Create a new directory called fluentd inside the /root folder. We will place all fluentd configurations here.

As a first step, let's create a ServiceAccount for fluentd.


Create a ServiceAccount with the following specs:
```
name: fluentd
namespace: elastic-stack
label: fluentd

```

solution:
Create a file called fluentd-sa.yaml in the /root/fluentd directory with the following contents:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: elastic-stack
  labels:
    app: fluentd
```
Apply this file using the below command:

kubectl apply -f fluentd-sa.yaml

Tasks
Next, we will create a ClusterRole for fluentd so that it has the requisite permissions to access the pods and namespaces in the Cluster.


Use the following specs to create a ClusterRole for fluentd:
```
name: fluentd
label: fluentd
permissions: get, list and watch
```

Create a file called fluentd-clusterrole.yaml in the /root/fluentd directory with the following contents:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluentd
  labels:
    app: fluentd
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch
```
Apply this file using the command:
kubectl apply -f fluentd-clusterrole.yaml

----------------------------------------------
Having created the ServiceAccount and ClusterRole, let's now create a ClusterRoleBinding to connect these two.


Create a ClusterRoleBinding named fluentd that grants permissions defined in the ClusterRole fluentd to the ServiceAccount fluentd.

Solution
Create a file named fluentd-clusterrolebinding.yaml in the /root/fluentd directory with the following contents:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: elastic-stack

```
Apply this file using the following command:

kubectl apply -f fluentd-clusterrolebinding.yaml

---------------------------------------------------
Next, let's create the fluentd configuration file named fluent.conf that ships logs created by the app container.

You can find these logs under the /var/log/containers/ directory.

Place this file in the following path: /root/fluentd/etc/fluent.conf

Create the required directories in the above path, if not present. This specific path will ensure that our custom configuration gets mounted to the pod.

Use the standard Fluentd source, parse, filter, and match directives to collect logs from all containers and forward them to Elasticsearch at elasticsearch.elastic-stack.svc.cluster.local on port 9200.

You can also include other file paths in this config file.


Refer to the official Fluentd documentation on how to create this file:
https://docs.fluentd.org/configuration/config-file

solution

creat a dirctory etc and below firl name fluent.conf

```
<label @FLUENT_LOG>
  <match fluent.**>
    @type null
    @id ignore_fluent_logs
  </match>
</label>
<source>
  @type tail
  @id in_tail_container_logs
  path "/var/log/containers/*.log"
  pos_file "/var/log/fluentd-containers.log.pos"
  tag "kubernetes.*"
  exclude_path /var/log/containers/fluent*
  read_from_head true
  <parse>
    @type "/^(?<time>.+) (?<stream>stdout|stderr)( (?<logtag>.))? (?<log>.*)$/"
    time_format "%Y-%m-%dT%H:%M:%S.%NZ"
    unmatched_lines
    expression ^(?<time>.+) (?<stream>stdout|stderr)( (?<logtag>.))? (?<log>.*)$
ignorecase false
    multiline false
  </parse>
</source>
<match **>
  @type elasticsearch
  @id out_es
  @log_level "info"
  include_tag_key true
  host "elasticsearch.elastic-stack.svc.cluster.local"
  port 9200
  path ""
  scheme http
  ssl_verify false
  ssl_version TLSv1_2
  user
  password xxxxxx
  reload_connections false
  reconnect_on_error true
  reload_on_failure true
  log_es_400_reason false
logstash_prefix "fluentd"
  logstash_dateformat "%Y.%m.%d"
  logstash_format true
  index_name "logstash"
  target_index_key
  type_name "fluentd"
  include_timestamp false
  template_name
  template_file
  template_overwrite false
  sniffer_class_name "Fluent::Plugin::ElasticsearchSimpleSniffer"
  request_timeout 5s
  application_name default
  suppress_type_name true
  enable_ilm false
  ilm_policy_id logstash-policy
  ilm_policy {}
  ilm_policy_overwrite false
<buffer>
    flush_thread_count 8
    flush_interval 5s
    chunk_limit_size 2M
    queue_limit_length 32
    retry_max_interval 30
    retry_forever true
  </buffer>
</match>

```

---------------------------------------------------------
Finally, let's create a fluentd DaemonSet to collect and forward logs to Elasticsearch.

Create a file named fluentd.yaml in the /root/fluentd directory with the configuration for a DaemonSet named fluentd in the elastic-stack namespace.


Use ServiceAccount fluentd in the configuration.

For the fluentd container, use the following image:
fluent/fluentd-kubernetes-daemonset:v1.14.1-debian-elasticsearch7-1.0

Use appropriate environment variables for the elasticsearch address and port above.
Also, add a variable FLUENT_ELASTICSEARCH_LOGSTASH_PREFIX with the value fluentd.

Mount the /var/log and /var/lib/docker/containers directories from the host to the pods, with the /var/lib/docker/containers directory mounted as read-only.

Also, mount the config file located under the fluentd/etc folder to the pod.

solution
create a file name fluentd.yaml

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: elastic-stack
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.14.1-debian-elasticsearch7-1.0
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch.elastic-stack.svc.cluster.local"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "http"
          - name: FLUENTD_SYSTEMD_CONF
            value: disable
          - name: FLUENT_CONTAINER_TAIL_EXCLUDE_PATH
            value: /var/log/containers/fluent*
          - name: FLUENT_ELASTICSEARCH_SSL_VERIFY
            value: "false"
          - name: FLUENT_CONTAINER_TAIL_PARSER_TYPE
            value: /^(?<time>.+) (?<stream>stdout|stderr)( (?<logtag>.))? (?<log>.*)$/ 
          - name:  FLUENT_ELASTICSEARCH_LOGSTASH_PREFIX
            value: "fluentd"
        resources:
          limits:
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: configpath
          mountPath: /fluentd/etc
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: configpath
        hostPath:
          path: /root/fluentd/etc
```

Apply the above file with the following command:

kubectl apply -f fluentd.yaml

kubectl logs app

You will be redirected to a new URL. To this URL, add /_search?q=*:*&pretty at the end and hit enter. Fluentd might take some time to ship the logs to Elasticsearch.
