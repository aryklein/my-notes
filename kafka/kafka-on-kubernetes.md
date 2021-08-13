# Kafka on Kubernetes with Strimzi

Kafka uses Zookeeper to manage the cluster. Zookeeper is used to coordinate the brokers/cluster topology. Zookeeper is
a consistent file system for configuration information. It gets used for leadership election for brokers, topicsa and
partitions.

Deploying a Kafka cluster is not difficult but maintaining a cluster could be. 

## Running Kafka without automation

If you want to deploy a Kafka cluster, at glance you should follow these steps:

- Setting up a ZooKeeper ensemble:
    - Create a data directory for Zookeeper.
    - Download and extract the Zookeeper binaries.
    - In the config file of all instances, specify the zookeeper servers that will be part of the ensemble.
    - set a different node ID in every node.
- Setting up a Kafka cluster:
    - Download and extract the Kafka binaries in all nodes.
    - configure a different `broker.id` in every node.
    - Set same `zookeeper.connect` property on all nodes with a comma-separated string listing the IP addresses and
ports numbers of all the ZooKeeper instances.

## Running Kafka on Docker containers

You can simplify the deployment of a Kafka cluster by using the Kafka and Zookeeper Docker images from Bitnami:

- https://github.com/bitnami/bitnami-docker-kafka
- https://github.com/bitnami/bitnami-docker-zookeeper

A simple example using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
     - '2181:2181'
    environment:
     - ALLOW_ANONYMOUS_LOGIN=yes
  kafka1:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
  kafka2:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
  kafka3:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
```

This setup runs all brokers and Zookeeper in the same host. It could work for testing. Moving this setup to production
(using multiple servers) could involve some Ansible work.

## Running Kafka on Kubernetes

The steps mentioned above are good options, but they require a lot of work on automation and maintenance. A good option
to avoid a headache is to run Kafka in Kubernetes cluster with the help of an operator. 

At the time of writting this document, [Strimzi](https://strimzi.io) operator seems to be the most mature and complete.

This operators simplify the process of:

- Deploying and running Kafka clusters
- Configuring access to Kafka
- Securing access to Kafka
- Upgrading Kafka
- Managing brokers
- Creating and managing topics
- Creating and managing users

It also provides some nice components like:

- `Kafka MirrorMaker` to mirror the Kafka cluster in a secondary cluster
- `Kafka exporter` to extract data for Prometheus
- `Cruise Control` to provide support for rebalancing the clusters based on workload data

### Installing the operator

Follow this [document](https://github.com/aryklein/my-notes/blob/main/kafka/strimzi.md)

### Deploying a Kafka cluster

In this section I will show you an example about how to create a Kafka cluster with 3 brokers and 3 Zookeeper nodes. If
you want to delve into the subject, I strongy recommend to read the
[official documentation](https://strimzi.io/docs/operators/latest/using.html).

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: kafka-cluster
  namespace: kafka
spec:
  ## Kafka brokers definition
  kafka:
    replicas: 3 (1)
    version: 2.8.0 (2)
    resources: (3)
      requests:
        memory: 4Gi
        cpu: "2"
    listeners: (4)
      - name: plain
        port: 9092
        type: internal
        tls: false
        authentication: (5)
          type: scram-sha-512
      - name: external
        port: 9092
        type: nodeport
        tls: false
        authentication:
          type: scram-sha-512
        configuration:
          preferredNodePortAddressType: InternalIP
    authorization: (6)
      type: simple
      superUsers:
        - user1
        - user2
#   image: my/tweaked-image:latest (7)
    template: (8)
      pod:
        metadata:
          annotations:
            some/annotation: "true"
        affinity: 
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                    - key: strimzi.io/name
                      operator: In
                      values: 
                        - kafka-cluster-kafka
                topologyKey: "kubernetes.io/hostname"
            preferredDuringSchedulingIgnoredDuringExecution:
              - weight: 1
                podAffinityTerm:
                  labelSelector:
                    matchExpressions:
                      - key: strimzi.io/name
                        operator: In
                        values:
                          - kafka-cluster-kafka
                  topologyKey: "topology.kubernetes.io/zone"
#      kafkaContainer: (9)
#        env:
#          - name: foo
#            value: bar
#          - name: foofoo
#            value: barbar

    storage: (10)
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 150Gi
        deleteClaim: false
    config: (11)
      offsets.topic.replication.factor: 1
      transaction.state.log.replication.factor: 1
      transaction.state.log.min.isr: 1
      log.retention.hours: 24
      log.retention.bytes: 30000000000
  ## cruice control definition
  cruiseControl: (12)
    brokerCapacity:
      disk: 150Gi
#      cpuUtilization: 100
#      inboundNetwork: 10000KiB/s
#      outboundNetwork: 10000KiB/s
#    config:
#      default.goals: >
#        com.linkedin.kafka.cruisecontrol.analyzer.goals.DiskCapacityGoal,
#        com.linkedin.kafka.cruisecontrol.analyzer.goals.CpuCapacityGoal
  ## Kafka exporter definition
  kafkaExporter: (11)
    groupRegex: ".*"
    topicRegex: ".*"
    resources:
      requests:
        cpu: 200m
        memory: 64Mi
      limits:
        cpu: 500m
        memory: 128Mi
    template:
      pod:
        metadata:
          annotations:
            some/annotation: "true"
#      container:
#        env:
#          - name: foo
#            value: bar
#
  ## Zookeeper definition
  zookeeper: (12)
    template:
      pod:
        affinity: (13)
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                    - key: strimzi.io/name
                      operator: In
                      values:
                        - kafka-cluster-zookeeper
                topologyKey: "kubernetes.io/hostname"
            preferredDuringSchedulingIgnoredDuringExecution:
              - weight: 1
                podAffinityTerm:
                  labelSelector:
                    matchExpressions:
                      - key: strimzi.io/name
                        operator: In
                        values:
                          - kafka-cluster-zookeeper
                  topologyKey: "topology.kubernetes.io/zone"
    replicas: 3 (14)
    resources: (15)
      requests:
        memory: 1Gi
        cpu: "1"
    storage: (16)
      type: persistent-claim
      size: 10Gi
      deleteClaim: false
  ## Entity operator
  entityOperator: (17)
    topicOperator: {}
    userOperator: {}
```

- (1): The number of Kafka brokers. If your cluster already has topics defined, you can scale the cluster.
- (2): Kafka version.
- (3): Requests for reservation of supported resources, currently CPU and memory, and limits to specify the maximum
resources that can be consumed.
- (4): Listeners configure how clients connect to the Kafka cluster via bootstrap addresses.
- (5): Listener authentication mechanism.
- (6): Configures authorization for Kafka brokers. For a list of supported authorization options, check
[here](https://strimzi.io/docs/operators/latest/using.html#con-securing-kafka-authorization-str).
- (7): Overrides Kafka container image. It's recommended only in special situations where you need to use a different
container registry or a customized image.
- (8): Template customization. In this example, pods are scheduled with anti-affinity so the pods are not scheduled on
nodes with the same hostname and preferably in different zones. Pods also have a customized annotation.
- (9): Custom environment variables. Prefixed ENVs with `KAFKA_` are internal to Strimzi and should be avoided.
- (10):
- (10):

### Listeners

Listeners configure how clients connect to the Kafka cluster via bootstrap addresses. Listeners are configured as
internal or external listeners for connection from inside or outside the Kubernetes cluster. Each listener is defined
as an array in the Kafka resource. You can configure as many listeners as required, as long as their names and ports are
unique.

The listener `type` could be set as `internal` for clients running in the same Kubernetes cluster or `route`,
`loadbalancer`, `nodeport` and `ingress` for external clients that run outside the Kubernetes cluster. More information
[here](https://strimzi.io/docs/operators/latest/using.html#type-GenericKafkaListener-reference).
