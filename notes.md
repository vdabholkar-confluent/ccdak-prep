## CCDAK Study Notes: Key Sections \& Exam Concepts

These concise notes are organized to give you a rapid, effective MCQ-focused review for the Confluent Certified Developer for Apache Kafka (CCDAK) exam. Each section covers definitions, configurations, behaviors, and real exam tips, derived from the best open-source guides, your repo, and recent expert recommendations[^1][^2][^3][^4][^5][^6].

### 1. Producer

- **Producer Basics**
    - Publishes records to Kafka topics; can specify partition, key, and headers.
    - *Key configs*:
        - `acks`: Controls durability. `acks=all` ensures higher durability.
        - `compression.type`: Enables compression for efficiency.
        - `batch.size`, `linger.ms`: Tune these for batching vs. latency.
        - `buffer.memory`, `max.request.size`: Producer buffer and individual message limits (default max message size: 1MB).
        - `enable.idempotence=true`: Guarantees no duplicates (exactly-once semantics).
        - `transactional.id`: Prepares producer for transactions (used with exactly-once delivery).
        - `retries`: How many times to retry on transient errors.
    - **Partitioning**
        - Records with the same key go to the same partition (guarantees per-key order).
        - Custom partitioner can be used.
- **Error Handling**
    - Retriable errors: `LEADER_NOT_AVAILABLE`, `NOT_LEADER_FOR_PARTITION`.
    - Non-retriable errors: `RecordTooLargeException`, `TopicAuthorizationException`.
    - Use callback mechanisms and handle errors from `send()` gracefully[^7][^8].
- **Performance Tips**
    - Increase `batch.size`, and adjust `linger.ms` for higher throughput when batching is desired.
    - Compression reduces bandwidth and disk usage.
    - Monitoring producer metrics: Look for batch sizes, I/O thread ratios.


### 2. Consumer

- **Consumer Basics**
    - Reads records from topics and partitions.
    - *Key configs*:
        - `group.id`: Consumers with same group share load by partition.
        - `enable.auto.commit`: If `false`, commit offsets manually.
        - `auto.offset.reset`: `earliest`/`latest`/`none` for where to start if no offset.
        - `max.poll.records`: Controls how many records fetched per poll.
        - `isolation.level`: `read_committed` to only see committed records.
    - **Partition Assignment Strategies**
        - *Range*, *Round Robin*, *Sticky*, *Cooperative Sticky* (sticky minimizes partition movement).
    - **Offset Management**
        - Offsets are stored in `__consumer_offsets` topic.
        - Use `commitSync()` or CLI to reset offsets as needed.
    - **Consumer Groups**
        - Each partition goes to one consumer per group; multiple groups can read whole topic independently.
        - To have each consumer read all partitions, use `assign()` with all partitions; *not* subscribe.
    - **Error Handling**
        - For bad records ("poison pills"), catch `RecordDeserializationException`, log, then use `seek()` to skip.
    - **Best Practices**
        - One consumer instance per thread—Kafka consumers are not thread-safe.
        - Use monitoring (lag) to check if consumers keep up.


### 3. Broker \& Cluster Configuration

- **Broker Role**
    - Hosts partitions, manages reads/writes, and replication.
    - *Key broker configs*:
        - `broker.id`: Unique per broker.
        - `log.dirs`: Disk location for data.
        - `default.replication.factor`, `min.insync.replicas`: Replication and durability.
        - `unclean.leader.election.enable`: Should be `false` for consistency (prevents out-of-sync replica as leader).
        - `auto.create.topics.enable`: If `true`, topics made on-the-fly if missing (not best practice in prod).
    - *Controller*: Manages partition leaders and fails over if broker dies.
    - *ZooKeeper/KRaft*: Used for metadata and controller election (CCDAK now also covers KRaft).
- **Scaling \& Reliability**
    - Brokers are stateless; adding brokers increases capacity.
    - Topics: More partitions = more parallelism, but overhead.
    - Replication ensures fault-tolerance (rep. factor ≥ 3 is best practice).
- **Internal Topics**
    - `__consumer_offsets` (commits), `__transaction_state`, `_schemas` (schema registry), Connect topics, etc.


### 4. Kafka Connect

- **Purpose**: To integrate Kafka with external systems (DBs, files, etc.) via connectors.
- **Modes**: Standalone (single) or Distributed (clustered, fault-tolerant).
- **Connectors**:
    - *Source connector*: Imports data into Kafka.
    - *Sink connector*: Exports data from Kafka to systems.
- **Core Concepts**
    - *Workers*: Run connectors/tasks.
    - *Tasks*: Units of parallel work.
    - *Deploy*: Via REST API or config files (JSON or properties).
    - *Internal Topics*: Offset, config, and status topics for state management.
- **Offset \& Fault Tolerance**
    - Offsets for source connectors managed in Kafka topics.
- **Transforms (SMT)**: Lightweight inline transformations.
- **Exactly-once Semantics**: Possible with proper configs.


### 5. Kafka Streams

- **Purpose**: Stream processing API for real-time analytics on Kafka topics.
- **Core Abstractions**
    - *KStream*: Unbounded stream of records.
    - *KTable*: Changelog stream; like a table snapshot.
    - *GlobalKTable*: Fully replicated for non-partitioned state.
- **Operations**
    - Stateless: `map`, `filter`, `foreach`.
    - Stateful: `aggregate`, `reduce`, `join`.
    - Time Windows: *Tumbling*, *hopping*, *sliding*, *session*.
- **Topology**
    - Chain together sources, processors, sinks; can be complex for custom logic.
- **Processing Guarantees**
    - At-least-once (default), Exactly-once with proper setup.
    - Enable RocksDB state store persistence for fast failover[^1].
- **Testing**
    - Use `TopologyTestDriver` for unit testing Kafka Streams logic.


### 6. Schema Registry

- **Purpose**: Central schema management for Avro, JSON Schema, or Protobuf data.
- **Features**
    - Stores schemas under a subject per topic and data type (key/value).
    - Manages versioning and compatibility (`BACKWARD`, `FORWARD`, `FULL`).
    - Supports compatibility checks to prevent bad schema changes.
- **Config**
    - Connectors, Producers, and Consumers work with schemas via schemas IDs stored in payload headers.
    - Integrates with Connect, REST Proxy, ksqlDB, and Streams.
- **Common Tasks**
    - Register/configure schemas via REST API.
    - Evolve schemas with backward/forward compatibility modes as per use cases[^9].


### 7. Security

- **Encrypt in Transit**
    - Use SSL/TLS: Configure `security.protocol=SSL` or `SASL_SSL`.
    - Manage trusted certificates and client/server authentication.
- **Authentication**
    - SASL/PLAIN (username/password), SASL/SCRAM, SASL/GSSAPI (Kerberos).
    - Configure `sasl.mechanism`, `sasl.jaas.config`.
- **Authorization**
    - ACLs: Set permissions using CLI (`kafka-acls.sh`).
    - Define rules for topics, consumer groups, clusters.
- **Hardening**
    - Restrict PLAINTEXT listeners in production.
    - Rotate secrets and monitor audit logs.
- **Secure Connect \& REST**
    - REST API endpoints can use HTTPS.


### 8. Monitoring \& Metrics

- **Brokers**
    - Enable JMX; monitor via Prometheus/Grafana, Confluent Control Center, or third-party tools.
    - Key Metrics:
        - `UnderReplicatedPartitions`, `OfflinePartitionsCount`, `RequestHandlerAvgIdlePercent`, consumer lag, `BytesInPerSec`, `BytesOutPerSec`, leader/follower metrics.
- **Consumers**
    - Monitor lag per consumer group; verify if consumers are falling behind.
    - Use CLI (`kafka-consumer-groups.sh`) for quick checks.
- **Producers**
    - Check error rates, retries, and batch sizes.
- **Kafka Connect**
    - Monitor connector/task status, error logs, and dead-letter queues.


## MCQ-Ready Shortlist: Things You MUST Know

- **Partitioning dictates parallelism and ordering.**
- **Producers can guarantee exactly-once if `enable.idempotence` and transactional APIs are used.**
- **Consumers must be in groups for scalable processing—with right assignor (Sticky, etc.).**
- **Default topic auto-creation is often enabled, but not recommended for prod.**
- **You can’t decrease topic partition count—only increase.**
- **Kafka Connect \& Streams are tested for their setup, configuration, and use cases.**
- **Schema Registry is central for Avro/Protobuf/JSON integration.**
- **Security: Know both SSL \& SASL configs, and ACL basics.**
- **Monitor with JMX, Control Center, CLI tools, and understand key health metrics.**

**Exam Tips**

- Be familiar with the most common CLI commands (`kafka-topics.sh`, `kafka-console-producer.sh`, `kafka-console-consumer.sh`, `kafka-configs.sh`, `kafka-consumer-groups.sh`).
- Visualize which feature or property belongs where: e.g., offset commits (`commitSync`), consumer group rebalancing triggers, transactional settings, Kafka Connect modes.
- Relate guarantee or error—e.g., `NotEnoughReplicasException` with `acks=all` and missing in-sync replicas.
- Practice with updated MCQ sets and review the official Confluent documentation for last-minute changes.

