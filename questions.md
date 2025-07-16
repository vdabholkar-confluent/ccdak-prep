# CCDAK Practice Exam - 60 Questions

## Instructions
- Time limit: 90 minutes
- Passing score: 75%
- Answer format: Multiple choice (some questions may have multiple correct answers)
- Click on "Details" to reveal the answer and explanation

---

## Question 1
You want to connect to a secured Kafka cluster that uses SSL encryption and SASL authentication with username/password. Which properties must your client include?

A. `security.protocol=SSL` and `ssl.truststore.location`
B. `security.protocol=SASL_SSL` and `sasl.jaas.config`
C. `security.protocol=SASL_PLAINTEXT` and `sasl.mechanism=PLAIN`
D. `security.protocol=SSL` and `sasl.jaas.config`

<details>
<summary>Answer</summary>

**B. `security.protocol=SASL_SSL` and `sasl.jaas.config`**

For SSL encryption with SASL authentication, you need:
- `security.protocol=SASL_SSL`
- `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="your-username" password="your-password";`
- `sasl.mechanism=PLAIN`

</details>

---

## Question 2
Which tool can you use to modify the replication factor of an existing topic?

A. `kafka-topics.sh`
B. `kafka-configs.sh`
C. `kafka-reassign-partitions.sh`
D. `kafka-consumer-groups.sh`

<details>
<summary>Answer</summary>

**C. `kafka-reassign-partitions.sh`**

The replication factor cannot be changed using `kafka-topics.sh` or `kafka-configs.sh`. You must use `kafka-reassign-partitions.sh` to create a reassignment plan that moves replicas to achieve the desired replication factor.

</details>

---

## Question 3
Which partition assignment strategy minimizes partition movements between assignments during consumer group rebalancing?

A. RangeAssignor
B. RoundRobinAssignor
C. StickyAssignor
D. CooperativeStickyAssignor

<details>
<summary>Answer</summary>

**C. StickyAssignor**

The StickyAssignor is designed to minimize partition movements by trying to keep the same partitions assigned to the same consumers across rebalances, while still maintaining balanced assignments.

</details>

---

## Question 4
What is true about topic compaction?

A. When a client produces a new event with an existing key, the old value is immediately deleted
B. Compaction happens in real-time as messages are produced
C. Compaction will keep exactly one message per key after compaction of inactive log segments
D. Compaction removes all messages older than the retention period

<details>
<summary>Answer</summary>

**C. Compaction will keep exactly one message per key after compaction of inactive log segments**

Log compaction ensures that at least the last known value for each message key is retained. It only operates on inactive (closed) log segments, not active ones.

</details>

---

## Question 5
You have a Kafka cluster with three brokers. You create a topic with `replication.factor=3`, `min.insync.replicas=2`, and `acks=all`. Which exception will be generated if two brokers are down?

A. `NotEnoughReplicasException`
B. `NetworkException`
C. `NotCoordinatorException`
D. `NotLeaderForPartitionException`

<details>
<summary>Answer</summary>

**A. `NotEnoughReplicasException`**

With `min.insync.replicas=2` and `acks=all`, the producer requires at least 2 replicas to acknowledge writes. If 2 brokers are down, there's only 1 replica available, which doesn't meet the minimum requirement.

</details>

---

## Question 6
You want to read messages from all partitions of a topic in every consumer instance of your application. How do you do this?

A. Use the `assign()` method using all topic partitions as argument
B. Use the `subscribe()` method with an empty consumer group name
C. Use the `subscribe()` method with a regular expression argument
D. Use the `assign()` method with the topic name as argument

<details>
<summary>Answer</summary>

**A. Use the `assign()` method using all topic partitions as argument**

To read from all partitions in every consumer instance, use manual partition assignment with `assign()` method, providing all TopicPartition objects for the topic.

</details>

---

## Question 7
Your application needs to be resilient to badly formatted records. You surround the `poll()` call with a try-catch block for `RecordDeserializationException`. What should you do in the catch block?

A. Log the bad record, no other action needed
B. Log the bad record and seek the consumer to the offset of the next record
C. Log the bad record and call `consumer.skip()` method
D. Throw a runtime exception to restart the application

<details>
<summary>Answer</summary>

**B. Log the bad record and seek the consumer to the offset of the next record**

After catching the deserialization exception, you need to log the error and use `seek()` to move past the problematic record to continue processing.

</details>

---

## Question 8
You have an existing topic with four partitions. Which statement is correct about changing the number of partitions?

A. You can increase the partition count and Kafka will leave existing data on original partitions
B. You can decrease the partition count if you increase the replication factor
C. You can decrease the partition count if you change the partitioning algorithm
D. You can increase the partition count and Kafka will redistribute all existing data

<details>
<summary>Answer</summary>

**A. You can increase the partition count and Kafka will leave existing data on original partitions**

Kafka only allows increasing partition count, never decreasing. Existing data remains on the original partitions and is not redistributed.

</details>

---

## Question 9
Match the following tools with their primary use cases:

Tools: Testcontainers, Trogdor, MockProducer, Connect Datagen
Use Cases: Unit Testing, Integration Testing, Performance Testing, Mock Data Generation

A. Testcontainers→Unit Testing, Trogdor→Performance Testing
B. MockProducer→Unit Testing, Connect Datagen→Mock Data Generation
C. Testcontainers→Integration Testing, Trogdor→Performance Testing
D. All of the above

<details>
<summary>Answer</summary>

**D. All of the above**

Correct mappings:
- Testcontainers → Integration Testing
- Trogdor → Performance Testing  
- MockProducer → Unit Testing
- Connect Datagen → Mock Data Generation

</details>

---

## Question 10
Which property must be set at the producer to prepare it for transactions?

A. `enable.transactions=true`
B. `transactional.id=unique-transaction-id`
C. `acks=all`
D. `enable.idempotence=true`

<details>
<summary>Answer</summary>

**B. `transactional.id=unique-transaction-id`**

Setting `transactional.id` to a unique value automatically enables idempotence and prepares the producer for transactional operations.

</details>

---

## Question 11
You need to update your consumer's offset to start reading from the beginning of a topic. Which action should you take?

A. Set `auto.offset.reset=earliest` in consumer configuration
B. Start a new consumer with the same group ID
C. Use `kafka-consumer-groups.sh` to reset offsets to earliest position
D. Temporarily set topic's `retention.ms` to 0

<details>
<summary>Answer</summary>

**C. Use `kafka-consumer-groups.sh` to reset offsets to earliest position**

For an existing consumer group, use the CLI tool to reset offsets:
`kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-group --reset-offsets --to-earliest --execute --topic my-topic`

</details>

---

## Question 12
You're integrating Kafka client logs with your application using log4j2. Which dependency must you include?

A. `org.slf4j:slf4j-log4j12`
B. `org.apache.logging.log4j:log4j-slf4j-impl`
C. None, the Kafka client includes the necessary dependencies
D. Only the log4j2 dependency

<details>
<summary>Answer</summary>

**C. None, the Kafka client includes the necessary dependencies**

The Kafka client dependency includes SLF4J by transitivity, so no additional logging dependency is required.

</details>

---

## Question 13
Your application has a consumer group with 2 consumers reading from a topic with 4 partitions. The lag is increasing despite smooth distribution. What should you do?

A. Add more consumers to increase parallelism
B. Add more partitions to the topic
C. Increase `max.poll.records` property
D. Decrease `max.poll.records` property

<details>
<summary>Answer</summary>

**A. Add more consumers to increase parallelism**

With 4 partitions and 2 consumers, you can add up to 2 more consumers (total 4) to match the number of partitions and increase processing parallelism.

</details>

---

## Question 14
You add a new node to your Kafka cluster. Which statements are true? (Select two)

A. A node ID will be assigned automatically
B. The new node will have controller role by default
C. The new node won't have partitions unless you create new topics
D. You can add the node without stopping existing nodes

<details>
<summary>Answer</summary>

**A. A node ID will be assigned automatically**
**D. You can add the node without stopping existing nodes**

Kafka allows adding nodes dynamically without downtime. The broker.id can be auto-generated if not specified.

</details>

---

## Question 15
For unit testing a Kafka Streams application with complex topology, which tool should you use?

A. TopologyTestDriver
B. MockProducer and MockConsumer
C. TestProducer and TestConsumer
D. KafkaUnitTestDriver

<details>
<summary>Answer</summary>

**A. TopologyTestDriver**

TopologyTestDriver is the official unit testing framework for Kafka Streams, allowing you to test topologies without a real Kafka cluster.

</details>

---

## Question 16
Your Java producer has low throughput. Metrics show low I/O thread ratio and low I/O wait ratio. What's the most likely cause?

A. Compression is enabled
B. Producer is sending large batches
C. Slow network connection
D. Expensive callback function in producer code

<details>
<summary>Answer</summary>

**D. Expensive callback function in producer code**

Low I/O thread ratios indicate the producer thread is spending time in application code rather than I/O operations, suggesting an expensive callback function.

</details>

---

## Question 17
You need to define custom processors in Kafka Streams. Which tool should you use?

A. Kafka Streams DSL
B. Processor API
C. TopologyTestDriver
D. Custom Transformation Language

<details>
<summary>Answer</summary>

**B. Processor API**

The Processor API is the low-level API in Kafka Streams that allows you to define custom processors for complex transformation logic.

</details>

---

## Question 18
What is the default maximum message size that can be produced to a Kafka topic?

A. 512 KB
B. 1 MB
C. 4 MB
D. 10 MB

<details>
<summary>Answer</summary>

**B. 1 MB**

The default value for `message.max.bytes` is 1,000,000 bytes (approximately 1 MB).

</details>

---

## Question 19
Based on this schema snippet, which format is being used?

```json
{
  "type": "record",
  "name": "UserProfile",
  "namespace": "com.example.users",
  "fields": [...]
}
```

A. Avro
B. Protobuf
C. JSON Schema
D. YAML Schema

<details>
<summary>Answer</summary>

**A. Avro**

The schema structure with "type": "record", "name", "namespace", and "fields" is characteristic of Avro schema format.

</details>

---

## Question 20
A producer sends messages with `acks=1`. What does this mean?

A. Producer waits for acknowledgment from all replicas
B. Producer waits for acknowledgment from the leader only
C. Producer doesn't wait for any acknowledgment
D. Producer waits for acknowledgment from majority of replicas

<details>
<summary>Answer</summary>

**B. Producer waits for acknowledgment from the leader only**

With `acks=1`, the producer waits for acknowledgment from the partition leader only, not from followers.

</details>

---

## Question 21
In a Kafka Streams application, what does the `application.id` property do?

A. Identifies the application instance
B. Serves as consumer group ID and prefix for internal topics
C. Sets the client ID for the producer
D. Defines the application name in monitoring

<details>
<summary>Answer</summary>

**B. Serves as consumer group ID and prefix for internal topics**

The `application.id` is used as the consumer group ID and as a prefix for internal topics like state stores and repartition topics.

</details>

---

## Question 22
Which Kafka Connect mode provides automatic load balancing and fault tolerance?

A. Standalone mode
B. Distributed mode
C. Embedded mode
D. Cluster mode

<details>
<summary>Answer</summary>

**B. Distributed mode**

Distributed mode provides automatic load balancing, fault tolerance, and scalability by distributing connector tasks across multiple worker instances.

</details>

---

## Question 23
What happens when you enable log compaction with `cleanup.policy=compact`?

A. Old messages are deleted based on time
B. Messages are compressed using a compression algorithm
C. Only the latest message for each key is retained
D. All messages are deleted immediately

<details>
<summary>Answer</summary>

**C. Only the latest message for each key is retained**

Log compaction ensures that at least the latest value for each message key is retained, removing older values for the same key.

</details>

---

## Question 24
Which configuration controls how often a consumer sends heartbeats to the coordinator?

A. `session.timeout.ms`
B. `heartbeat.interval.ms`
C. `max.poll.interval.ms`
D. `request.timeout.ms`

<details>
<summary>Answer</summary>

**B. `heartbeat.interval.ms`**

This property controls the frequency of heartbeat messages sent to the group coordinator to indicate the consumer is alive.

</details>

---

## Question 25
In Schema Registry, what does `BACKWARD` compatibility mean?

A. New schema can read data written with previous schema
B. Previous schema can read data written with new schema
C. Both new and previous schemas can read each other's data
D. No compatibility requirements

<details>
<summary>Answer</summary>

**A. New schema can read data written with previous schema**

BACKWARD compatibility means consumers using the new schema can read data produced with the previous schema.

</details>

---

## Question 26
What is the purpose of `__consumer_offsets` topic?

A. Store consumer group metadata
B. Store committed offsets for consumer groups
C. Store topic configuration
D. Store producer transaction state

<details>
<summary>Answer</summary>

**B. Store committed offsets for consumer groups**

The `__consumer_offsets` topic stores the committed offsets for each consumer group, allowing consumers to resume from the correct position.

</details>

---

## Question 27
Which property should you set to enable exactly-once semantics in Kafka Streams?

A. `processing.guarantee=exactly_once`
B. `enable.idempotence=true`
C. `isolation.level=read_committed`
D. `acks=all`

<details>
<summary>Answer</summary>

**A. `processing.guarantee=exactly_once`**

This property enables exactly-once processing guarantees in Kafka Streams applications.

</details>

---

## Question 28
What is the maximum number of partitions a single consumer can be assigned in a consumer group?

A. 1
B. Equal to the number of topic partitions
C. Unlimited
D. Equal to the number of consumers

<details>
<summary>Answer</summary>

**B. Equal to the number of topic partitions**

A single consumer can be assigned multiple partitions, but the total number cannot exceed the number of partitions in the subscribed topics.

</details>

---

## Question 29
Which tool is used for performance testing of Kafka clusters?

A. Testcontainers
B. Trogdor
C. MockProducer
D. JMeter

<details>
<summary>Answer</summary>

**B. Trogdor**

Trogdor is Kafka's built-in performance testing framework designed specifically for testing Kafka clusters.

</details>

---

## Question 30
What happens when a consumer exceeds `max.poll.interval.ms`?

A. The consumer throws an exception
B. The consumer is removed from the group and partitions are rebalanced
C. The consumer continues processing but logs a warning
D. The consumer automatically commits offsets

<details>
<summary>Answer</summary>

**B. The consumer is removed from the group and partitions are rebalanced**

If a consumer doesn't call `poll()` within `max.poll.interval.ms`, it's considered dead and removed from the group, triggering a rebalance.

</details>

---

## Question 31
Which serialization format requires a Schema Registry?

A. JSON
B. Avro
C. String
D. ByteArray

<details>
<summary>Answer</summary>

**B. Avro**

Avro serialization typically requires a Schema Registry to store and retrieve schema definitions, although it can work without one in some cases.

</details>

---

## Question 32
What is the purpose of `min.insync.replicas`?

A. Minimum number of replicas required for a partition
B. Minimum number of replicas that must acknowledge a write
C. Minimum number of replicas that must be available for reads
D. Minimum number of replicas per topic

<details>
<summary>Answer</summary>

**B. Minimum number of replicas that must acknowledge a write**

This setting defines the minimum number of in-sync replicas that must acknowledge a write for it to be considered successful.

</details>

---

## Question 33
Which window type in Kafka Streams has a fixed size but can overlap?

A. Tumbling Window
B. Hopping Window
C. Session Window
D. Sliding Window

<details>
<summary>Answer</summary>

**B. Hopping Window**

Hopping windows have a fixed size but advance by a specified interval, potentially creating overlapping windows.

</details>

---

## Question 34
What does the `fetch.min.bytes` property control in a consumer?

A. Maximum bytes per partition fetch
B. Minimum bytes before server returns response
C. Maximum bytes per fetch request
D. Minimum bytes per message

<details>
<summary>Answer</summary>

**B. Minimum bytes before server returns response**

The server will wait until at least `fetch.min.bytes` of data is available before returning a response to the fetch request.

</details>

---

## Question 35
Which Kafka Connect component is responsible for data format conversion?

A. Connector
B. Task
C. Converter
D. Transform

<details>
<summary>Answer</summary>

**C. Converter**

Converters are responsible for converting data between Connect's internal format and the serialized form stored in Kafka.

</details>

---

## Question 36
What is the default partition assignment strategy in Kafka consumers?

A. RangeAssignor
B. RoundRobinAssignor
C. StickyAssignor
D. CooperativeStickyAssignor

<details>
<summary>Answer</summary>

**D. CooperativeStickyAssignor**

As of Kafka 2.4+, the default partition assignment strategy is CooperativeStickyAssignor.

</details>

---

## Question 37
Which isolation level ensures consumers only read committed transactional messages?

A. `read_uncommitted`
B. `read_committed`
C. `read_atomic`
D. `read_consistent`

<details>
<summary>Answer</summary>

**B. `read_committed`**

This isolation level ensures consumers only read messages from committed transactions, filtering out aborted transactions.

</details>

---

## Question 38
What is the purpose of `auto.offset.reset` configuration?

A. Automatically commit offsets at intervals
B. Reset offsets when consumer starts
C. Behavior when no initial offset exists
D. Reset offsets when consumer fails

<details>
<summary>Answer</summary>

**C. Behavior when no initial offset exists**

This property controls what happens when there's no initial offset in Kafka or the current offset no longer exists.

</details>

---

## Question 39
Which metric indicates the number of partitions without a leader?

A. `UnderReplicatedPartitions`
B. `OfflinePartitionsCount`
C. `ActiveControllerCount`
D. `LeaderCount`

<details>
<summary>Answer</summary>

**B. `OfflinePartitionsCount`**

This metric shows the number of partitions that don't have an active leader, indicating availability issues.

</details>

---

## Question 40
In ksqlDB, what does the `EMIT CHANGES` clause do?

A. Creates a push query that continuously emits results
B. Enables change data capture
C. Commits changes to the database
D. Emits schema changes

<details>
<summary>Answer</summary>

**A. Creates a push query that continuously emits results**

`EMIT CHANGES` creates a push query that continuously outputs results as new data arrives.

</details>

---

## Question 41
Which property controls the maximum time a producer will wait for a response?

A. `request.timeout.ms`
B. `delivery.timeout.ms`
C. `max.block.ms`
D. `linger.ms`

<details>
<summary>Answer</summary>

**A. `request.timeout.ms`**

This property controls the maximum time the producer will wait for a response from the server for a request.

</details>

---

## Question 42
What happens when you set `enable.auto.commit=false` in a consumer?

A. Consumer cannot commit offsets
B. Consumer must manually commit offsets
C. Offsets are committed synchronously
D. Offsets are never committed

<details>
<summary>Answer</summary>

**B. Consumer must manually commit offsets**

When auto-commit is disabled, the application must manually commit offsets using `commitSync()` or `commitAsync()`.

</details>

---

## Question 43
Which Kafka Streams operation creates a new stream from an existing one?

A. `foreach`
B. `map`
C. `peek`
D. `print`

<details>
<summary>Answer</summary>

**B. `map`**

The `map` operation transforms each record and returns a new KStream, while `foreach` and `peek` are terminal/side-effect operations.

</details>

---

## Question 44
What is the purpose of the `__transaction_state` topic?

A. Store consumer offsets
B. Store producer transaction state
C. Store topic configurations
D. Store connector states

<details>
<summary>Answer</summary>

**B. Store producer transaction state**

This internal topic stores the state of ongoing transactions for exactly-once semantics.

</details>

---

## Question 45
Which configuration enables compression for a producer?

A. `compression.type=gzip`
B. `enable.compression=true`
C. `compression.level=9`
D. `compression.algorithm=snappy`

<details>
<summary>Answer</summary>

**A. `compression.type=gzip`**

Set `compression.type` to values like `gzip`, `snappy`, `lz4`, or `zstd` to enable compression.

</details>

---

## Question 46
What is the maximum parallelism possible for a Kafka Streams application?

A. Number of application instances
B. Number of topic partitions
C. Number of CPU cores
D. Number of threads per instance

<details>
<summary>Answer</summary>

**B. Number of topic partitions**

The maximum parallelism is limited by the number of partitions in the input topics, as each partition can only be processed by one task.

</details>

---

## Question 47
Which property controls how long a Kafka Streams application waits before processing a record?

A. `commit.interval.ms`
B. `poll.ms`
C. `processing.delay.ms`
D. There is no such property

<details>
<summary>Answer</summary>

**D. There is no such property**

Kafka Streams processes records as soon as they arrive; there's no built-in delay mechanism.

</details>

---

## Question 48
What does `unclean.leader.election.enable=false` prevent?

A. Any leader elections
B. Elections when all ISRs are unavailable
C. Elections during network partitions
D. Elections when brokers restart

<details>
<summary>Answer</summary>

**B. Elections when all ISRs are unavailable**

This setting prevents electing a leader from out-of-sync replicas, prioritizing consistency over availability.

</details>

---

## Question 49
Which tool can you use to check the lag of a consumer group?

A. `kafka-topics.sh`
B. `kafka-consumer-groups.sh`
C. `kafka-configs.sh`
D. `kafka-log-dirs.sh`

<details>
<summary>Answer</summary>

**B. `kafka-consumer-groups.sh`**

Use `kafka-consumer-groups.sh --describe --group <group-name>` to check consumer group lag.

</details>

---

## Question 50
In Kafka Connect, what is the purpose of Single Message Transforms (SMTs)?

A. Transform entire batches of messages
B. Apply lightweight transformations to individual records
C. Transform message keys only
D. Transform message formats between topics

<details>
<summary>Answer</summary>

**B. Apply lightweight transformations to individual records**

SMTs perform simple transformations on individual messages as they flow through a connector.

</details>

---

## Question 51
Which configuration determines how often log segments are checked for deletion?

A. `log.retention.check.interval.ms`
B. `log.segment.delete.delay.ms`
C. `log.cleanup.interval.ms`
D. `log.retention.hours`

<details>
<summary>Answer</summary>

**A. `log.retention.check.interval.ms`**

This property controls how frequently the log cleaner checks for logs eligible for deletion.

</details>

---

## Question 52
What is the purpose of the `group.initial.rebalance.delay.ms` property?

A. Delay between rebalances
B. Delay before first rebalance when group starts
C. Delay before consumer joins group
D. Delay between heartbeats

<details>
<summary>Answer</summary>

**B. Delay before first rebalance when group starts**

This property delays the initial rebalance to allow more consumers to join the group before the first assignment.

</details>

---

## Question 53
Which Kafka Streams join requires a windowed operation?

A. KStream-KStream join
B. KTable-KTable join
C. KStream-KTable join
D. GlobalKTable-KStream join

<details>
<summary>Answer</summary>

**A. KStream-KStream join**

KStream-KStream joins require a time window because streams are unbounded and need a temporal boundary for the join.

</details>

---

## Question 54
What is the default value for `acks` in a Kafka producer?

A. 0
B. 1
C. all
D. -1

<details>
<summary>Answer</summary>

**B. 1**

The default value for `acks` is 1, meaning the producer waits for acknowledgment from the leader replica only.

</details>

---

## Question 55
Which property controls the maximum amount of time a consumer will block when calling `poll()`?

A. `max.poll.interval.ms`
B. `fetch.max.wait.ms`
C. `session.timeout.ms`
D. `request.timeout.ms`

<details>
<summary>Answer</summary>

**B. `fetch.max.wait.ms`**

This property controls the maximum time the server will block before answering the fetch request if there isn't sufficient data to immediately satisfy the `fetch.min.bytes` requirement.

</details>

---

## Question 56
In Kafka Streams, what happens when you call `KStream.to()`?

A. Creates a new KStream
B. Writes the stream to a topic (terminal operation)
C. Converts KStream to KTable
D. Filters the stream

<details>
<summary>Answer</summary>

**B. Writes the stream to a topic (terminal operation)**

`KStream.to()` is a terminal operation that writes the stream records to a specified Kafka topic.

</details>

---

## Question 57
Which configuration enables rack-aware replica placement?

A. `rack.aware.replication.enable=true`
B. `broker.rack=rack-id`
C. `replica.rack.placement=aware`
D. `enable.rack.awareness=true`

<details>
<summary>Answer</summary>

**B. `broker.rack=rack-id`**

Setting `broker.rack` on each broker enables rack-aware replica placement, ensuring replicas are distributed across different racks.

</details>

---

## Question 58
What is the purpose of the `retries` configuration in a Kafka producer?

A. Number of times to retry failed requests
B. Number of times to retry connecting to brokers
C. Number of times to retry serialization
D. Number of times to retry partition assignment

<details>
<summary>Answer</summary>

**A. Number of times to retry failed requests**

The `retries` property specifies how many times the producer will retry sending a record if it receives a retriable error.

</details>

---

## Question 59
Which Kafka Connect error handling configuration sends failed records to a dead letter queue?

A. `errors.tolerance=all`
B. `errors.deadletterqueue.topic.name=dlq-topic`
C. `errors.log.enable=true`
D. `errors.retry.timeout=30000`

<details>
<summary>Answer</summary>

**B. `errors.deadletterqueue.topic.name=dlq-topic`**

Setting this property to a topic name enables dead letter queue functionality, sending failed records to the specified topic.

</details>

---

## Question 60
In a Kafka cluster with KRaft mode (no ZooKeeper), what stores the cluster metadata?

A. Kafka brokers' local storage
B. A dedicated metadata topic
C. External database
D. In-memory only

<details>
<summary>Answer</summary>

**B. A dedicated metadata topic**

In KRaft mode, cluster metadata is stored in a dedicated internal topic (`__cluster_metadata`) managed by the controller nodes.

</details>

---

## Exam Summary

**Time**: 90 minutes  
**Total Questions**: 60  
**Passing Score**: 75% (45 correct answers)  
**Question Distribution**:
- Producer/Consumer: 25%
- Kafka Streams: 20%
- Kafka Connect: 15%
- Broker Configuration: 15%
- Security: 10%
- Monitoring: 10%
- CLI Tools: 5%

**Key Topics Covered**:
- Producer configurations (`acks`, `retries`, `compression.type`)
- Consumer configurations (`auto.offset.reset`, `isolation.level`, `max.poll.interval.ms`)
- Kafka Streams operations (joins, windowing, state stores)
- Kafka Connect architecture (connectors, tasks, converters, SMTs)
- Security configurations (SASL, SSL, ACLs)
- Monitoring and metrics
- CLI tools usage
- Topic management and configuration
- Schema Registry and serialization

**Study Recommendations**:
- Focus on practical configurations and their effects
- Understand the relationship between different properties
- Practice with CLI tools and their parameters
- Study error handling and troubleshooting scenarios
- Review security configuration patterns
- Understand Kafka Streams topology and operations
