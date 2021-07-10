# Kafka for dummies

Apache Kafka is a distributed, horizontally scalable streaming platform. Kafka is based on the pub/sub model. It's
similar to any messaging system. Applications (`producers`) send messages (`records`) to a Kafka node (broker) and
messages are processed by other applications called `consumers`. Messages are stored in `topics` and consumers can
subscribe to a topic and listen to those messages. Messages can be anything like sensors measurements, meter readings,
user actions, etc.

Summarizing, producers publish messages to a topic, the broker stores them in the received order, and consumers
subscribe and read messages from the topic.

![kafka-01](../images/kafka-01.png)
