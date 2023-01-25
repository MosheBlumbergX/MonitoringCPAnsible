# MonitoringCPAnsible

Repo to show case monitoring of CP platform deployed via cp-ansible

Users should be able to pull JMX from the cluster using the following ports: 

Supports metrics aggregation using JMX/Jolokia.
Supports aggregated metrics export to Prometheus.


## Jolokia 

 [Jolokia](https://jolokia.org/) is a JMX-HTTP bridge giving an alternative to JSR-160 connectors. It is an agent based approach with support for many platforms. In addition to basic JMX operations it enhances JMX remoting with unique features like bulk requests and fine grained security policies. 

Since the Jolokia run in agent mode on Kafka's class path, it will connect to Kafka monitoring MBeans and will provide metrics on HTTP interface.


## Prometheus

Prometheus is a tool used for aggregating multiple platform metrics while scraping hundreds of endpoints. It is purpose-built for scrape and aggregation use cases.

How does Prometheus work?

Prometheus is an ecosystem with two major components: the server-side component and the client-side configuration. The server-side component is responsible for storing all the metrics and scraping all clients as well. Prometheus differs from services like Elasticsearch and Splunk, which generally use an intermediate component responsible for scraping data from clients and shipping it to the servers. Because there is no intermediate component scraping Prometheus metrics, all poll-related configurations are present on the server itself.

![alt text](images/prometheus-server-e1616955998879-768x627.png)


There are two core pieces in this diagram:

1. Prometheus server: This component is responsible for polling all of the processes/clients with their metrics exposed on a specific port. The Prometheus server internally maintains a configuration file that lists all the server IP addresses/hostnames and ports on which Prometheus metrics are exposed. The scrape target configuration is the file that keeps all target mapping within Prometheus. Scrape targets are required when we are deploying everything manually without any automation. Prometheus also supports service discovery modules, which it can leverage to discover any available services that are exposing metrics. This auto-discovery is an amazing tool when used with Kubernetes-based deployments, where Pod names (among other elements) are ephemeral. To keep it simple, Prometheus service discovery won't be covered in this post.

2. Client processes: All clients that want to leverage Prometheus will need two configuration pieces. First, they must use the Prometheus client library to expose metrics in a Prometheus compatible format (OpenMetrics). Secondly, they must use a YAML configuration file for extracting JMX metrics. This configuration file is used for converting, renaming, and filtering some of the attributes for consumption. The YAML configuration file is necessary for the JVM client, as the JVM MBeans are exposed, converted, and/or renamed to a specific format for consumption using this configuration file.

## Grafana

When you have metrics data streaming into the Prometheus server, we can start dashboarding our metrics. The tool of choice in our stack is Grafana. Conceptually, here's how the process will look once we have connected Grafana to Prometheus:

![alt text](images/grafana-to-prometheus-e1616975067603-1024x556.png)


## CP-ansible

With [cp-ansible](https://docs.confluent.io/ansible/current/overview.html) you can with 2 properties enabled jolokia and Prometheus agents. 

```
  vars:
    ansible_connection: ssh
    ansible_user: ec2-user 
    ansible_become: true
    ansible_ssh_private_key_file: key.pem
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'

    jmxexporter_enabled: true
    jolokia_enabled: true
```


### Enable JMX Exporter

JMX Exporter is disabled by default. When enabled, the JMX Exporter jar is pulled from the internet and enabled on all Confluent Platform components besides Confluent Control Center.

#### Enable JMX Exporter in hosts.yml as below:

```
all:
  vars:
    jmxexporter_enabled: true
```

### Enable Jolokia

Jolokia monitoring is disabled by default for Confluent Platform components when installed by Ansible Playbooks for Confluent Platform.

#### Enable Jolokia in hosts.yml as shown below:

```
all:
  vars:
    jolokia_enabled: true
```


## Reference 

[Confluent Docs](https://docs.confluent.io/ansible/current/ansible-configure.html#enable-jmx-exporter)
[Confluent Blog](https://www.confluent.io/blog/monitor-kafka-clusters-with-prometheus-grafana-and-confluent/?_ga=2.28670844.1574239109.1674464926-1092077398.1613042839)

