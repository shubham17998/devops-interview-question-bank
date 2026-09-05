# Kafka Interview Questions

## Basics
1. What is Kafka used for?
   Kafka is a distributed event streaming platform used to publish, store, and process high-throughput, ordered streams of records in real time. Common uses: decoupling microservices, log aggregation, event-driven architectures, stream processing pipelines, and buffering data between producers and consumers at scale.

2. What are Kafka topics and partitions?
   A **topic** is a named stream/category of records (like a table or a log). Each topic is split into one or more **partitions**, which are ordered, append-only logs. Partitioning is what lets Kafka scale horizontally — different partitions can live on different brokers and be consumed in parallel, while ordering is only guaranteed within a single partition.

3. How do consumer groups work?
   Consumers subscribe to a topic as part of a **consumer group** (identified by `group.id`). Kafka assigns each partition to exactly one consumer within the group, so messages are load-balanced across the group's consumers with no overlap. Different consumer groups are independent — each group gets its own copy of every message. If a consumer dies, its partitions are rebalanced to the remaining consumers in the group.

## Troubleshooting & Operations
4. How do you debug Kafka consumer lag?
   Check consumer lag with `kafka-consumer-groups.sh --describe --group <group>` (or a monitoring tool like Burrow/Prometheus JMX exporter) to see the offset gap per partition. Common causes: slow consumer processing (scale out consumers, up to the partition count), too few partitions for the throughput needed, a stuck/rebalancing consumer, or a downstream dependency (DB, API) throttling the consumer. Fix by scaling consumers/partitions, optimizing the processing logic, or increasing `max.poll.records`/`session.timeout.ms` tuning.

5. How do you secure Kafka?
   - **Encryption in transit:** enable TLS/SSL between clients and brokers, and between brokers.
   - **Authentication:** SASL (PLAIN, SCRAM, or Kerberos/GSSAPI) or mutual TLS to verify client identity.
   - **Authorization:** Kafka ACLs to restrict which principals can produce/consume/administer specific topics.
   - **Network isolation:** run brokers in private subnets/VPCs, restrict access via security groups/firewalls.
   - **Encryption at rest:** encrypt the underlying disks (e.g., EBS encryption).
   - Keep ZooKeeper (or KRaft controller) access restricted as well, since it holds cluster metadata.
