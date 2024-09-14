📜 Security Best Practices for the EFK Stack on Kubernetes
The EFK (Elasticsearch, Fluentd, Kibana) stack is a popular logging stack for Kubernetes, providing a powerful platform for logging, searching, and visualizing data. However, securing the EFK stack is crucial to ensure that sensitive data remains protected and that the integrity and availability of the logging infrastructure are maintained. This guide will cover key security best practices for deploying the EFK stack on Kubernetes, focusing on securing inter-component communications, restricting access to Kibana with authentication and authorization, and using Kubernetes network policies to control traffic flow.

🔐 Restricting Access to Elasticsearch
Elasticsearch is the backend that stores logs and transfers them over to Kibana. It is very important to secure this repository of log information that may be critical for business operations.

Authentication and Authorization
The first step to implement authentication in Elasticsearch is to add xpack security related environment variables to its deployment definition files.

xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
Thereafter, certificates for the user can be generated and converted to a Secret. This secret can be mounted to the pod as an environment variable:

- name: ELASTIC_PASSWORD
  valueFrom:
    secretKeyRef:
      name: es-credentials
      key: password
An alternative to this approach is use of executable binaries within the running elasticsearch pod. After adding the xpack environment variables to the configuration and applying the file, binary utilities such as elasticsearch-setup-passwords or elasticsearch-users can be used to add a new user.

🔒 Restricting Access to Kibana
Kibana, as the user interface of the EFK stack, requires careful attention to access control to protect sensitive data.

Authentication and Authorization
Implementing authentication and authorization mechanisms is crucial for controlling access to Kibana.

Elasticsearch Security Features
Elasticsearch provides built-in security features that can be leveraged to secure Kibana:

xpack.security.enabled: true
Configure Elasticsearch users and roles to define access permissions, and use these credentials in Kibana configuration file to enforce authentication and authorization. These credentials can be specified as environment variables.

Proxy Authentication
Alternatively, use a reverse proxy (such as Nginx or Apache) in front of Kibana to handle authentication. This approach allows for the integration with external authentication providers (LDAP, OAuth, etc.):

location / {
  proxy_pass http://kibana:5601;
  auth_basic "Restricted Access";
  auth_basic_user_file /etc/nginx/.htpasswd;
}
Session Management
Configure session management in Kibana to control session timeouts and ensure that users are logged out after periods of inactivity.

We can also use Role-Based Access Control (RBAC) in Kubernetes to secure our cluster by controlling who can access and perform actions within the cluster.

Also, it is important to use stable images when deploying containers in the kubernetes cluster. To further identify and mitigate potential security vulnerabilities, these images should be scanned.

🔑 🔓 Enabling Fluentd Connection with a Restricted Elasticsearch Pod
The fluentd configuration file can be edited to ship logs to the secured Elasticsearch URL by utilizing the same credentials.
Few ssl related boolean values need to be changed to true.

🧾 Kubernetes Network Policies
Kubernetes network policies are essential for controlling the flow of traffic between stack components and ensuring that only authorized components can communicate with each other.

Diagram

![8th](https://github.com/user-attachments/assets/88947403-8cf8-49c9-b07d-f78a79ff4c22)


Defining Network Policies
Create network policies that specifically allow traffic between Elasticsearch, Fluentd, and Kibana, while denying all other unauthorized traffic.

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: efk-stack-policy
  namespace: logging
spec:
  podSelector:
    matchLabels:
      app: efk-stack
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: fluentd
    - podSelector:
        matchLabels:
          app: kibana
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: elasticsearch
This policy ensures that only Fluentd and Kibana can send traffic to Elasticsearch, and only Elasticsearch can receive traffic from these components, enhancing the security posture of your EFK stack.

🤞 Securing Inter-Component Communications
Securing communication between Elasticsearch, Fluentd, and Kibana ensures that data in transit cannot be intercepted or tampered with by unauthorized entities.
Internet Protocol Security Virtual Private Network (IPSec VPN), a network security protocol, can be used to encrypt traffic between pods in Kubernetes. Other methods are discussed below.

TLS Encryption
Implementing TLS (Transport Layer Security) encryption is fundamental in protecting the data exchanged between EFK stack components.

Elasticsearch
Start by generating TLS certificates for Elasticsearch. You can use tools like cert-manager on Kubernetes to automate certificate management.

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: elasticsearch-cert
  namespace: logging
spec:
  secretName: elasticsearch-tls
  duration: 2160h # 90d
  renewBefore: 360h # 15d
  commonName: elasticsearch.logging.svc.cluster.local
  issuerRef:
    name: ca-issuer
    kind: ClusterIssuer
Configure Elasticsearch to use these certificates for HTTPS:

apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch
  namespace: logging
spec:
  version: 7.9.3
  http:
    tls:
      selfSignedCertificate:
        disabled: true
  nodeSets:
  - name: default
    count: 3
    config:
      http:
        ssl:
          certificate_authorities: ["/usr/share/elasticsearch/config/certs/ca.crt"]
          certificate: "/usr/share/elasticsearch/config/certs/tls.crt"
          key: "/usr/share/elasticsearch/config/certs/tls.key"
Fluentd and Kibana
Similarly, ensure Fluentd and Kibana are configured to use TLS for secure communication. Reference the official documentation for each component for specific configuration details.

Mutual TLS Authentication
Beyond encrypting traffic, consider implementing mutual TLS (mTLS) for an added layer of security. mTLS requires both the client and server to authenticate each other, ensuring that only trusted components can communicate with each other.

🎉 Conclusion
Congratulations! You've successfully gained practical knowledge about multiple strategies to secure the EFK stack on Kubernetes.

Securing the EFK stack on Kubernetes involves a combination of securing inter-component communications with TLS, restricting access to Kibana through authentication and authorization, and controlling traffic flow with Kubernetes network policies.

By adhering to these best practices, you can ensure that your logging infrastructure is robust against unauthorized access and data breaches.

🎓 What's Next?
In this lab, we will walk you through securing the EFK stack on Kubernetes, by setting up individual and mutual authentication alongwith setting up network policies on Kubernetes.

Click on the Tasks tab to put your knowledge to test and complete the hands-on exercises for this lab. Happy learning! 🚀



------------------------------------------------------------------------------------------------------------------------------------

## Tasks

Access the Elasticsearch UI from the top right of this page. You can see the details of the deployment and logs here. Let's secure these details.

Modify the Elasticsearch statefulset configuration, within the /root/elastic-search folder, to enable xpack security by adding the appropriate environment variables.

Configure Elasticsearch to accept a username and password to enable user to view logs on the Elasticsearch UI by editing its statefulset.

Within the running elasticsearch container, use the approporiate utility from the bin folder to add the required user.


Username: elastic_user
Password: elasticPass123

Solution
----------------
Navigate to the /root/elastic-search directory.
Edit the es-statefulset.yaml file and add the following variables under the env section:
env:
- name: discovery.type
  value: single-node
- name: ES_JAVA_OPTS
  value: "-Xms1g -Xmx1g"
- name: xpack.security.enabled
  value: "true"
- name: xpack.security.transport.ssl.enabled
  value: "true"
Apply this new statefulset:
kubectl apply -f es-statefulset.yaml
Refresh the Elasticsearch UI. It may take some time for it to restart.

As soon as the UI loads, you will be prompted for a username and password. However, we first need to configure this to login.

In your terminal, exec into the elasticsearch-0 container and use the elasticsearch-users utility under bin folder to add a new user:

kubectl exec -it elasticsearch-0 -- bin/elasticsearch-users useradd elastic_user -r superuser -p elasticPass123
Here, we are creating a user with username elastic_user and password elasticPass123. The role granted to this user is superuser.

In the Elasticsearch UI, enter the same username and password and you should see the deployment details now.

-------------------------------------------------------------------------------------------------------

Now, try to access the Kibana UI. You can see that the page displays Cannot connect to the Elasticsearch cluster.

This is due to the setup of our elasticsearch username and password in the previous task.

Modify the Kibana deployment configuration within the /root/kibana folder to enable its connection with the elasticsearch cluster.


Employ the same credentials as used in the previous task:
Username: elastic_user
Password: elasticPass123

Note: Kibana may take some time to come up after the changes.

Solution 
-------------------------
Navigate to the /root/kibana directory.

Edit the kibana-deployment.yaml file and append an env section with the following variables:

env:
- name: ELASTICSEARCH_USERNAME
  value: "elastic_user"
- name: ELASTICSEARCH_PASSWORD
  value: "elasticPass123"
The complete configuration appears as:

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
        env:
        - name: ELASTICSEARCH_USERNAME
          value: "elastic_user"
        - name: ELASTICSEARCH_PASSWORD
          value: "elasticPass123"
Apply this configuration:
kubectl apply -f kibana-deployment.yaml
Refresh the Kibana UI. The changes may take some time to load.
You will then be presented with a prompt requiring you to enter a username and password.
Enter username as elastic_user and password as elasticPass123.

You can now access the Kibana UI and see your profile as elastic-user.


---------------------------------------------------------------------------------------------------------

There is a web-server pod deployed in the default namespace. Try accessing the Elasticsearch URL http://elasticsearch.elastic-stack.svc.cluster.local:9200 from within this pod.

Ensure to use the username [elastic_user] and password [elasticPass123] provisioned in one of the previous tasks to access the above URL.

The details of the elasticsearch deployment will be displayed.

Create a Kubernetes network policy named efk-stack-policy in the elastic-stack namespace to allow traffic exclusively from the elastic-stack namespace and block traffic from all other namespaces.


Refer to the network policy structure provided under the Kubernetes Network Policies section in the Description.

Test your policy by trying to access the Elasticsearch URL from the web-server pod deployed in the default namespace.


Solution
-------------------


Create a yaml file with the following contents:
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: efk-stack-policy
  namespace: elastic-stack
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: {}
      namespaceSelector:
        matchLabels:
          name: elastic-stack
  egress:
  - to:
    - podSelector: {}
      namespaceSelector:
        matchLabels:
          name: elastic-stack
Apply this policy file:
kubectl apply -f networkPolicy.yaml
Now, curl the Elasticsearch URL again from the web-server pod in the default namespace:
kubectl exec -it web-server-bf5bd9d79-kcffb -n default -- curl -u elastic_user:elasticPass123 http://elasticsearch.elastic-stack.svc.cluster.local:9200
You can get your pod name using the following command:

kubectl get pods -n default
You should see the command become unresponsive and eventually time out, indicating that our policy has been applied.


----------------------------------------------------------------------------------------------------------------------------
With the addition of a user profile to our Elasticsearch and Kibana pods, fluentd is now unable to communicate with and ship logs to Elasticsearch.

Configure TLS/SSL for encrypted communications between Fluentd and Elasticsearch within the elastic-stack namespace.

Modify the Fluentd configuration at /root/fluentd/etc to reference the Elasticsearch credentials:
Username: elastic_user
Password: elasticPass123


Delete the existing fluentd pod so that a new one, with the updated settings, is created.


solution

Inspect the logs of the fluentd pod:
kubectl logs <fluentd-pod-name>
You can see a message stating that fluentd is unable to connect to elasticsearch due to authorization error as a result of implementation of a new security policy.

Also, in the Kibana UI, you can see no elasticsearch data shipped.

Now, navigate to the /root/fluentd/etc folder. Scroll down to the <match **> section at the end of this configuration. This part pertains to the recipient elasticsearch.

Modify the value of ssl_verify parameter to true from false:

ssl_verify true
Add the elasticsearch username and password for the user and password parameters:
user elastic_user
password elasticPass123
Save this configuration and delete the existing fluentd pod:
kubectl delete pod <fluentd-pod-name>
A new fluentd pod will be spawned immediately with our new config settings.

The logs now show data chunks being pushed, and querying for elasticsearch data on Kibana UI will result in success.
