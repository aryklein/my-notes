# Kafka on Kubernetes with Strimzi

Kafka uses Zookeeper to manage the cluster. Zookeeper is used to coordinate the brokers/cluster topology. Zookeeper is
a consistent file system for configuration information. It gets used for leadership election for brokers, topicsa and
partitions.

Deploying a Kafka cluster is not difficult but maintaining a cluster could be. 

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

