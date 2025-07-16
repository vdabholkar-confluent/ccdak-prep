# CCDAK Practice Exam Set 2 - 60 Questions

## Instructions
- Time limit: 90 minutes
- Passing score: 75%
- Answer format: Multiple choice (some questions may have multiple correct answers)
- Click on "Response" to reveal the answer and explanation

---

## Question 1:
You want to connect to a secured Kafka cluster that uses SSL encryption and requires SASL authentication with username and password. Your application needs to connect from a client machine that doesn't have the cluster's SSL certificate in its default truststore. Which properties must your client configuration include to establish a secure connection?

1. `security.protocol=SSL`, `ssl.truststore.location=/path/to/truststore.jks`, `ssl.truststore.password=password`
2. `security.protocol=SASL_SSL`, `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="user" password="pass";`, `ssl.truststore.location=/path/to/truststore.jks`
3. `security.protocol=SASL_PLAINTEXT`, `sasl.mechanism=PLAIN`, `sasl.jaas.config=...`
4. `security.protocol=SASL_SSL`, `ssl.keystore.location=/path/to/keystore.jks`, `sasl.mechanism=SCRAM-SHA-256`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `security.protocol=SASL_SSL`, `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="user" password="pass";`, `ssl.truststore.location=/path/to/truststore.jks`**

For SSL encryption with SASL authentication, you need:
- `security.protocol=SASL_SSL` for both SSL and SASL
- `sasl.jaas.config` for authentication credentials
- `ssl.truststore.location` to verify the broker's SSL certificate
- `sasl.mechanism=PLAIN` (default for username/password)

</details>

---

## Question 2:
You have an existing Kafka topic with a replication factor of 2 running on a 3-broker cluster. Due to increased importance of this topic's data, you need to change the replication factor to 3. Which approach should you use to modify the replication factor safely?

1. Use `kafka-topics.sh --alter --replication-factor 3` command
2. Use `kafka-configs.sh --alter --add-config replication.factor=3` command
3. Create a partition reassignment JSON file and use `kafka-reassign-partitions.sh --execute`
4. Delete the topic and recreate it with replication factor 3

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. Create a partition reassignment JSON file and use `kafka-reassign-partitions.sh --execute`**

The replication factor cannot be changed using `kafka-topics.sh` or `kafka-configs.sh`. You must use the partition reassignment tool:
1. Generate a reassignment JSON file specifying new replica assignments
2. Use `kafka-reassign-partitions.sh --execute --reassignment-json-file reassignment.json`
3. This will safely add replicas to achieve the desired replication factor

</details>

---

## Question 3:
Your Kafka consumer group has been running smoothly, but after a recent deployment, you notice that consumers are frequently being removed from the group and partitions are being rebalanced. The logs show that consumers are exceeding a specific timeout. Which consumer configuration should you investigate and potentially adjust?

1. `session.timeout.ms` - time to detect consumer failure
2. `max.poll.interval.ms` - maximum time between poll() calls
3. `heartbeat.interval.ms` - frequency of heartbeat messages
4. `request.timeout.ms` - maximum time for broker response

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `max.poll.interval.ms` - maximum time between poll() calls**

When consumers are being removed from the group due to timeout, it's typically because they're not calling `poll()` frequently enough. This happens when:
- Message processing takes too long between polls
- The consumer exceeds `max.poll.interval.ms` (default 5 minutes)
- The consumer is considered dead and removed from the group

</details>

---

## Question 4:
In a Kafka topic configured with `cleanup.policy=compact`, you produce the following messages in sequence:
- Key="user1", Value="name:John"
- Key="user2", Value="name:Jane"  
- Key="user1", Value="name:Johnny"
- Key="user1", Value=null

After log compaction runs, which messages will remain in the compacted log segment?

1. All four messages will remain
2. Key="user2", Value="name:Jane" and Key="user1", Value="name:Johnny"
3. Key="user2", Value="name:Jane" only
4. Key="user1", Value=null and Key="user2", Value="name:Jane"

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. Key="user2", Value="name:Jane" only**

Log compaction keeps the latest message for each key. Since the last message for "user1" has a null value (tombstone), it will be deleted during compaction, removing all messages for that key. Only the message for "user2" will remain.

</details>

---

## Question 5:
You have a Kafka cluster with 5 brokers and a topic configured with `replication.factor=3` and `min.insync.replicas=2`. A producer is sending messages with `acks=all`. During a network partition, 3 brokers become unreachable. What will happen to the producer's write operations?

1. Writes will succeed as long as the partition leader is available
2. Writes will fail with `NotEnoughReplicasException`
3. Writes will be buffered until the network partition is resolved
4. Writes will succeed but with reduced durability guarantees

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Writes will fail with `NotEnoughReplicasException`**

With `acks=all` and `min.insync.replicas=2`, the producer requires at least 2 replicas to acknowledge writes. If 3 brokers are unreachable, many partitions will have fewer than 2 in-sync replicas, causing writes to fail with `NotEnoughReplicasException`.

</details>

---

## Question 6:
Your application needs to consume messages from multiple topics where each consumer instance should process all messages from all topics independently. The topics are: `orders`, `payments`, and `inventory`. How should you configure your consumers?

1. Use `consumer.subscribe(Arrays.asList("orders", "payments", "inventory"))` with different consumer group IDs
2. Use `consumer.assign()` with all TopicPartition objects from all three topics
3. Use `consumer.subscribe(Pattern.compile("orders|payments|inventory"))` with the same consumer group ID
4. Use separate consumer instances with different topics and the same consumer group ID

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Use `consumer.assign()` with all TopicPartition objects from all three topics**

To have each consumer instance process ALL messages from ALL topics independently, you need to use manual partition assignment with `assign()`. This bypasses consumer group management and allows each consumer to read from all partitions of all topics.

</details>

---

## Question 7:
You're consuming from a topic with Avro-serialized messages. Occasionally, you encounter corrupted messages that cause `SerializationException` during deserialization. You want to log these bad records and continue processing other messages. What should you do in your consumer loop?

1. Catch `SerializationException`, log the error, and continue with the next `poll()`
2. Catch `SerializationException`, log the error, seek to the next offset, and continue
3. Set `errors.tolerance=all` in consumer configuration
4. Use a custom deserializer that handles corrupted messages

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Catch `SerializationException`, log the error, seek to the next offset, and continue**

When deserialization fails, you must:
1. Catch the `SerializationException` 
2. Log the problematic record details
3. Use `consumer.seek(topicPartition, offset + 1)` to move past the corrupted message
4. Continue processing - otherwise you'll be stuck on the same corrupted message

</details>

---

## Question 8:
You have a topic named `user-events` with 8 partitions. You want to increase the partition count to 12 to improve parallelism. Which statement is correct about this operation?

1. You can increase partitions and Kafka will redistribute existing messages evenly
2. You can increase partitions and existing messages will remain on their current partitions
3. You cannot increase partitions on an existing topic
4. You can increase partitions but message ordering will be affected globally

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. You can increase partitions and existing messages will remain on their current partitions**

When you increase partition count:
- Existing messages stay on their current partitions
- Only new messages will be distributed across all partitions (including new ones)
- This may affect keyed messages since the same key might now go to different partitions
- Ordering is maintained within each partition

</details>

---

## Question 9:
Match the following testing tools with their appropriate use cases in Kafka development:

Tools: `TopologyTestDriver`, `EmbeddedKafka`, `MockProducer`, `Testcontainers`

Use Cases:
A. Unit testing Kafka Streams topologies
B. Integration testing with real Kafka cluster
C. Unit testing producer logic
D. Integration testing with containerized Kafka

1. TopologyTestDriver→A, EmbeddedKafka→B, MockProducer→C, Testcontainers→D
2. TopologyTestDriver→A, EmbeddedKafka→D, MockProducer→C, Testcontainers→B
3. TopologyTestDriver→C, EmbeddedKafka→A, MockProducer→B, Testcontainers→D
4. TopologyTestDriver→A, EmbeddedKafka→C, MockProducer→B, Testcontainers→D

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **1. TopologyTestDriver→A, EmbeddedKafka→B, MockProducer→C, Testcontainers→D**

Correct mappings:
- TopologyTestDriver: Unit testing Kafka Streams topologies without real Kafka
- EmbeddedKafka: Integration testing with embedded Kafka cluster
- MockProducer: Unit testing producer logic without real Kafka
- Testcontainers: Integration testing with containerized Kafka instances

</details>

---

## Question 10:
You want to implement exactly-once processing in your Kafka producer. Which configuration property is essential to enable this capability?

1. `enable.idempotence=true`
2. `transactional.id=unique-producer-id`
3. `acks=all`
4. `retries=Integer.MAX_VALUE`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `transactional.id=unique-producer-id`**

Setting `transactional.id` enables exactly-once semantics by:
- Automatically enabling idempotence
- Allowing the producer to participate in transactions
- Ensuring each producer instance has a unique identity
- Enabling recovery from producer failures

</details>

---

## Question 11:
You have a consumer group that's falling behind on processing messages. The consumer group has 3 consumers processing a topic with 6 partitions. Consumer lag is increasing despite even partition distribution. What's the most effective action to reduce lag?

1. Increase `max.poll.records` to fetch more messages per poll
2. Add more consumers to the consumer group (up to 6 total)
3. Increase the number of partitions in the topic
4. Decrease `max.poll.records` to process smaller batches

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Add more consumers to the consumer group (up to 6 total)**

With 6 partitions, you can have up to 6 consumers for maximum parallelism. Adding more consumers (up to the number of partitions) will distribute the load and reduce processing lag more effectively than adjusting poll settings.

</details>

---

## Question 12:
Your Java application uses Kafka clients and you want to integrate Kafka's logging with your existing Log4j2 setup. When you add the Kafka client dependency to your project, which additional dependency do you need for proper logging integration?

1. `org.slf4j:slf4j-log4j12`
2. `org.apache.logging.log4j:log4j-slf4j-impl`
3. `ch.qos.logback:logback-classic`
4. No additional dependency is needed

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **4. No additional dependency is needed**

The Kafka client already includes SLF4J dependencies. Your application's Log4j2 configuration will automatically handle Kafka client logs through SLF4J's transitivity. No additional logging dependencies are required.

</details>

---

## Question 13:
You have a consumer group with 4 consumers reading from a topic with 6 partitions. The application is processing messages correctly, but you notice that the consumer lag is steadily increasing. Monitoring shows that message processing time is consistent. What should you do?

1. Increase `session.timeout.ms` to give consumers more time
2. Add more consumers to increase processing parallelism
3. Decrease `max.poll.interval.ms` to make consumers poll more frequently
4. Increase `fetch.min.bytes` to get more data per request

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Add more consumers to increase processing parallelism**

With 6 partitions and 4 consumers, you can add 2 more consumers to achieve maximum parallelism (1 consumer per partition). This will distribute the processing load and help reduce the increasing lag.

</details>

---

## Question 14:
You're adding a new broker to your existing 3-broker Kafka cluster to handle increased load. Which statements are true about adding this new broker? (Select two)

1. The new broker will automatically receive partitions from existing brokers
2. The new broker will be assigned a unique `broker.id` 
3. Existing topics will automatically rebalance to the new broker
4. You can add the broker without stopping the existing cluster

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. The new broker will be assigned a unique `broker.id`** and **4. You can add the broker without stopping the existing cluster**

When adding a broker:
- It gets a unique broker.id (auto-assigned or configured)
- The cluster continues running without interruption
- Existing partitions don't automatically move to the new broker
- You need to manually reassign partitions or create new topics to utilize the new broker

</details>

---

## Question 15:
You're developing unit tests for a Kafka Streams application with a complex topology that includes multiple processors, state stores, and output topics. You want to test the topology without connecting to a real Kafka cluster. Which tool should you use?

1. `TestInputTopic` and `TestOutputTopic`
2. `TopologyTestDriver`
3. `MockSchemaRegistryClient`
4. `EmbeddedKafkaCluster`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `TopologyTestDriver`**

`TopologyTestDriver` is specifically designed for unit testing Kafka Streams topologies. It provides a lightweight testing framework that doesn't require a real Kafka cluster and allows you to test your topology logic in isolation.

</details>

---

## Question 16:
Your Kafka producer is experiencing low throughput. You check the producer metrics and observe that the `request-rate` is low and `batch-size-avg` is small. The `buffer-available-bytes` metric shows high values. What's the most likely cause?

1. Network bandwidth limitations
2. Small `batch.size` configuration limiting batching efficiency
3. High `linger.ms` value causing delays
4. Broker-side processing bottlenecks

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Small `batch.size` configuration limiting batching efficiency**

Low request rate with small batch sizes and high buffer availability indicates that the producer is not batching messages efficiently. Increasing `batch.size` or `linger.ms` would allow more messages to be batched together, improving throughput.

</details>

---

## Question 17:
You need to implement a custom message transformation in your Kafka Streams application that requires access to the record's metadata (timestamp, headers, partition). Which API should you use?

1. Kafka Streams DSL with `map()` operation
2. Processor API with custom `Processor` implementation
3. Kafka Streams DSL with `transform()` operation
4. Kafka Streams DSL with `flatMap()` operation

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Processor API with custom `Processor` implementation**

The Processor API provides low-level access to record metadata including timestamps, headers, and partition information. The DSL operations have limited access to this metadata, making the Processor API the right choice for complex transformations requiring full record context.

</details>

---

## Question 18:
You want each consumer instance in your application to read from all partitions of a topic named `notifications`, ensuring that every consumer processes every message independently. How should you configure your consumers?

1. Use `subscribe()` with the topic name and different consumer group IDs
2. Use `assign()` with all TopicPartition objects for the topic
3. Use `subscribe()` with a consumer group ID and set `partition.assignment.strategy=RoundRobinAssignor`
4. Use `subscribe()` with no consumer group ID specified

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Use `assign()` with all TopicPartition objects for the topic**

To have each consumer read from ALL partitions independently, use manual partition assignment with `assign()`. This bypasses consumer group coordination and allows each consumer to read from all partitions of the topic.

</details>

---

## Question 19:
What is the default maximum message size that a Kafka producer can send to a topic?

1. 512 KB
2. 1 MB
3. 4 MB
4. 16 MB

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. 1 MB**

The default value for `message.max.bytes` (broker-side) and `max.request.size` (producer-side) is 1,048,576 bytes (1 MB). This limits the maximum size of a single message that can be produced to a topic.

</details>

---

## Question 20:
You're working with a schema that has the following structure. Which serialization format is this?

```json
{
  "type": "record",
  "name": "OrderEvent",
  "namespace": "com.company.events",
  "fields": [
    {"name": "orderId", "type": "string"},
    {"name": "amount", "type": "double"}
  ]
}
```

1. JSON Schema
2. Avro Schema
3. Protobuf Schema
4. Thrift Schema

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Avro Schema**

The structure with `"type": "record"`, `"name"`, `"namespace"`, and `"fields"` array is characteristic of Apache Avro schema definition format.

</details>

---

## Question 21:
You have a Kafka Streams application that processes user activity events. The application needs to count the number of events per user over a 5-minute tumbling window. Which operation should you use?

1. `groupByKey().count(TimeWindows.of(Duration.ofMinutes(5)))`
2. `groupByKey().aggregate(TimeWindows.of(Duration.ofMinutes(5)))`
3. `groupByKey().windowedBy(TimeWindows.of(Duration.ofMinutes(5))).count()`
4. `groupByKey().count().windowedBy(TimeWindows.of(Duration.ofMinutes(5)))`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. `groupByKey().windowedBy(TimeWindows.of(Duration.ofMinutes(5))).count()`**

The correct syntax for windowed aggregation in Kafka Streams is to apply the window specification using `windowedBy()` after grouping, then apply the aggregation operation like `count()`.

</details>

---

## Question 22:
Your Kafka Connect cluster is running in distributed mode. Which configuration property must be the same across all worker instances to form a single Connect cluster?

1. `bootstrap.servers`
2. `group.id`
3. `plugin.path`
4. `rest.port`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `group.id`**

All workers in a distributed Kafka Connect cluster must have the same `group.id` to coordinate and share connector tasks. This forms the Connect cluster identity and enables load balancing and fault tolerance.

</details>

---

## Question 23:
You have a topic configured with `cleanup.policy=compact` and `delete.retention.ms=86400000` (1 day). What happens when you produce a message with a null value (tombstone)?

1. The message is immediately deleted from the log
2. The message is kept for 1 day, then removed along with all previous messages for that key
3. The message is ignored and not written to the log
4. The message causes an error because null values are not allowed

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. The message is kept for 1 day, then removed along with all previous messages for that key**

Tombstone messages (null values) are retained for the `delete.retention.ms` period to ensure all consumers see the deletion. After this period, the tombstone and all previous messages for that key are removed during compaction.

</details>

---

## Question 24:
Which consumer configuration property controls the frequency of heartbeat messages sent to the group coordinator?

1. `session.timeout.ms`
2. `heartbeat.interval.ms`
3. `max.poll.interval.ms`
4. `connections.max.idle.ms`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `heartbeat.interval.ms`**

This property controls how frequently the consumer sends heartbeat messages to the group coordinator to indicate it's still alive. The default is 3 seconds, and it should be set to a value lower than `session.timeout.ms`.

</details>

---

## Question 25:
In Schema Registry, you have a subject with compatibility set to `BACKWARD`. You want to evolve the schema by adding a new required field. What will happen?

1. The schema registration will succeed
2. The schema registration will fail due to compatibility violation
3. The schema will be registered with a warning
4. The schema will be registered but marked as deprecated

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. The schema registration will fail due to compatibility violation**

`BACKWARD` compatibility means new schema can read data written with previous schema. Adding a required field breaks this compatibility because the new schema cannot read old data that lacks this field.

</details>

---

## Question 26:
What is the purpose of the `__consumer_offsets` topic in Kafka?

1. Store consumer group membership information
2. Store committed offsets for consumer groups
3. Store consumer configuration settings
4. Store consumer performance metrics

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Store committed offsets for consumer groups**

The `__consumer_offsets` topic is an internal topic that stores the committed offsets for each consumer group and partition, allowing consumers to resume from the correct position after restarts or rebalances.

</details>

---

## Question 27:
You want to enable exactly-once processing in your Kafka Streams application. Which configuration should you set?

1. `enable.idempotence=true`
2. `processing.guarantee=exactly_once_v2`
3. `isolation.level=read_committed`
4. `acks=all`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `processing.guarantee=exactly_once_v2`**

This property enables exactly-once processing semantics in Kafka Streams. The `_v2` version is the improved implementation that provides better performance than the original `exactly_once` setting.

</details>

---

## Question 28:
In a consumer group with 5 consumers subscribing to a topic with 3 partitions, how many consumers will remain idle?

1. 0
2. 1
3. 2
4. 3

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. 2**

With 3 partitions and 5 consumers, only 3 consumers can be assigned partitions (one per partition). The remaining 2 consumers will be idle since there are no more partitions to assign.

</details>

---

## Question 29:
Which tool is specifically designed for performance testing and fault injection in Kafka clusters?

1. Kafka Console Producer
2. Trogdor
3. JMeter
4. Gatling

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Trogdor**

Trogdor is Kafka's built-in performance testing and fault injection framework, designed specifically for testing Kafka clusters under various load and failure scenarios.

</details>

---

## Question 30:
A consumer fails to call `poll()` within the `max.poll.interval.ms` timeout period. What happens next?

1. The consumer throws a `TimeoutException`
2. The consumer is marked as failed and removed from the group, triggering a rebalance
3. The consumer automatically commits its offsets
4. The consumer continues processing but logs a warning

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. The consumer is marked as failed and removed from the group, triggering a rebalance**

When a consumer doesn't call `poll()` within `max.poll.interval.ms`, it's considered dead and automatically removed from the consumer group, causing a rebalance to redistribute its partitions to other consumers.

</details>

---

## Question 31:
You're using Avro serialization for your Kafka messages with Schema Registry. Which component is responsible for schema evolution and compatibility checks?

1. Kafka broker
2. Schema Registry
3. Avro serializer
4. Kafka Connect

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Schema Registry**

Schema Registry manages schema evolution, enforces compatibility rules, and stores schema versions. It validates new schemas against existing ones based on the configured compatibility level.

</details>

---

## Question 32:
You have a producer configured with `retries=3` and `retry.backoff.ms=100`. If a send operation fails with a retriable error, what is the maximum total time the producer will spend retrying?

1. 300 ms
2. 400 ms
3. 700 ms
4. 1000 ms

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. 400 ms**

With 3 retries and 100ms backoff:
- Initial attempt: 0ms
- First retry: 100ms wait + attempt
- Second retry: 100ms wait + attempt  
- Third retry: 100ms wait + attempt
- Total backoff time: 3 × 100ms = 300ms
- But the question asks for total time including the initial attempt timing, making it approximately 400ms

</details>

---

## Question 33:
In Kafka Streams, which type of join requires both input streams to be windowed?

1. KStream-KTable join
2. KTable-KTable join
3. KStream-KStream join
4. KStream-GlobalKTable join

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. KStream-KStream join**

KStream-KStream joins require windowing because both streams are unbounded and need a time boundary to determine which records should be joined together.

</details>

---

## Question 34:
What does the `fetch.min.bytes` consumer configuration control?

1. Minimum number of bytes per message
2. Minimum data server must return before responding to fetch request
3. Minimum bytes per partition in a fetch request
4. Minimum bytes to trigger a consumer rebalance

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Minimum data server must return before responding to fetch request**

`fetch.min.bytes` controls the minimum amount of data the server should return for a fetch request. The server waits until this amount of data is available before responding, which can improve throughput by reducing the number of requests.

</details>

---

## Question 35:
In Kafka Connect, which component is responsible for converting data between Connect's internal format and the external system's format?

1. Connector
2. Task
3. Converter
4. Transform

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. Converter**

Converters handle the serialization/deserialization between Connect's internal data format and the format used in Kafka topics (e.g., JSON, Avro, String).

</details>

---

## Question 36:
What is the current default partition assignment strategy for Kafka consumers?

1. RangeAssignor
2. RoundRobinAssignor
3. StickyAssignor
4. CooperativeStickyAssignor

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **4. CooperativeStickyAssignor**

Since Kafka 2.4, the default partition assignment strategy is `CooperativeStickyAssignor`, which provides better rebalancing behavior by minimizing partition movement and reducing rebalance time.

</details>

---

## Question 37:
You want to ensure that consumers only read committed transactional messages. Which configuration should you set?

1. `isolation.level=read_committed`
2. `enable.auto.commit=false`
3. `auto.offset.reset=earliest`
4. `fetch.max.wait.ms=500`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **1. `isolation.level=read_committed`**

This setting ensures that consumers only read messages from committed transactions, filtering out messages from aborted transactions.

</details>

---

## Question 38:
When a consumer has no committed offset and `auto.offset.reset=none`, what happens?

1. Consumer starts from earliest offset
2. Consumer starts from latest offset
3. Consumer throws an exception
4. Consumer waits for the next message

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. Consumer throws an exception**

When `auto.offset.reset=none` and no committed offset exists, the consumer throws an `OffsetOutOfRangeException` to prevent data loss or unintended behavior.

</details>

---

## Question 39:
Which JMX metric indicates the number of partitions that don't have an active leader?

1. `kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions`
2. `kafka.controller:type=KafkaController,name=OfflinePartitionsCount`
3. `kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec`
4. `kafka.server:type=ReplicaManager,name=LeaderCount`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `kafka.controller:type=KafkaController,name=OfflinePartitionsCount`**

This metric tracks the number of partitions that don't have an active leader, which indicates availability issues in the cluster.

</details>

---

## Question 40:
In ksqlDB, what type of query does `SELECT * FROM users EMIT CHANGES;` create?

1. Pull query
2. Push query
3. Batch query
4. Stream query

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Push query**

`EMIT CHANGES` creates a push query that continuously streams results as new data arrives, rather than returning a one-time result set.

</details>

---

## Question 41:
Which producer configuration controls the maximum time the producer will wait for a batch to fill before sending?

1. `batch.size`
2. `linger.ms`
3. `buffer.memory`
4. `max.block.ms`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `linger.ms`**

`linger.ms` controls how long the producer waits for additional messages to fill a batch before sending it. This can improve throughput by allowing better batching at the cost of increased latency.

</details>

---

## Question 42:
You disable auto-commit in your consumer by setting `enable.auto.commit=false`. What must you do to track your processing progress?

1. Consumer automatically commits on each poll
2. You must manually commit offsets using `commitSync()` or `commitAsync()`
3. Offsets are committed when the consumer shuts down
4. No offset management is needed

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. You must manually commit offsets using `commitSync()` or `commitAsync()`**

With auto-commit disabled, the application is responsible for committing offsets manually to track processing progress and ensure proper recovery after failures.

</details>

---

## Question 43:
Which Kafka Streams operation creates a new stream by transforming each record?

1. `foreach`
2. `map`
3. `filter`
4. `peek`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `map`**

The `map` operation transforms each record and produces a new KStream with the transformed records. `foreach` is a terminal operation, `filter` selects records, and `peek` is for side effects.

</details>

---

## Question 44:
What does the `__transaction_state` topic store in Kafka?

1. Consumer group states
2. Producer transaction coordination information
3. Topic partition assignments
4. Broker configuration changes

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Producer transaction coordination information**

The `__transaction_state` topic stores transaction metadata and coordination information used by the transaction coordinator to manage exactly-once semantics.

</details>

---

## Question 45:
You want to enable message compression in your Kafka producer. Which configuration should you set?

1. `compression.type=gzip`
2. `enable.compression=true`
3. `compression.codec=snappy`
4. `message.compression=lz4`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **1. `compression.type=gzip`**

Set `compression.type` to one of: `none`, `gzip`, `snappy`, `lz4`, or `zstd` to enable compression. The default is `none` (no compression).

</details>

---

## Question 46:
What determines the maximum parallelism for a Kafka Streams application?

1. Number of application instances
2. Number of input topic partitions
3. Number of available CPU cores
4. Number of stream threads

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Number of input topic partitions**

The maximum parallelism is limited by the number of partitions in the input topics, since each partition can only be processed by one task at a time.

</details>

---

## Question 47:
You have a Kafka Streams application that needs to join two topics. The topics have different numbers of partitions. What must you do?

1. Repartition both topics to have the same number of partitions
2. Use a GlobalKTable for one of the topics
3. Configure the application to handle different partition counts
4. Both A and B are valid approaches

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **4. Both A and B are valid approaches**

For joins in Kafka Streams, you can either:
- A. Repartition both topics to have the same number of partitions and ensure co-partitioning
- B. Use a GlobalKTable for one topic, which doesn't require co-partitioning

</details>

---

## Question 48:
Setting `unclean.leader.election.enable=false` prevents what type of situation?

1. Any leader election from occurring
2. Electing a leader from out-of-sync replicas
3. Elections during broker restarts
4. Elections when all brokers are available

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Electing a leader from out-of-sync replicas**

This setting prevents electing a leader from replicas that are not in the ISR (In-Sync Replica) set, which could result in data loss but maintains data consistency.

</details>

---

## Question 49:
Which command-line tool should you use to check the current lag of all consumer groups?

1. `kafka-topics.sh --describe`
2. `kafka-consumer-groups.sh --describe --all-groups`
3. `kafka-configs.sh --describe --entity-type groups`
4. `kafka-run-class.sh kafka.tools.ConsumerGroupCommand`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `kafka-consumer-groups.sh --describe --all-groups`**

This command shows the lag information for all consumer groups, including current offset, log end offset, and lag for each partition.

</details>

---

## Question 50:
In Kafka Connect, what are Single Message Transforms (SMTs) used for?

1. Converting between different message formats
2. Applying lightweight transformations to individual records
3. Transforming entire batches of messages
4. Managing connector lifecycle

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Applying lightweight transformations to individual records**

SMTs are used to make simple modifications to individual records as they flow through a connector, such as renaming fields, filtering, or format changes.

</details>

---

## Question 51:
How often does the log retention checker run by default?

1. Every minute
2. Every 5 minutes
3. Every 30 minutes
4. Every hour

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Every 5 minutes**

The default value for `log.retention.check.interval.ms` is 300,000 milliseconds (5 minutes), which controls how frequently the log cleaner checks for segments eligible for deletion.

</details>

---

## Question 52:
What is the purpose of `group.initial.rebalance.delay.ms`?

1. Delay between consumer rebalances
2. Delay before the first rebalance when a group forms
3. Delay before consumers join existing groups
4. Delay between heartbeat messages

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Delay before the first rebalance when a group forms**

This property introduces a delay before the first rebalance when a consumer group is initially formed, allowing more consumers to join before the first partition assignment occurs.

</details>

---

## Question 53:
Which type of Kafka Streams join requires both streams to be within a time window?

1. KStream-KTable join
2. KTable-KTable join
3. KStream-KStream join
4. KStream-GlobalKTable join

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **3. KStream-KStream join**

KStream-KStream joins are windowed joins because both streams are unbounded, requiring a time window to define which records should be joined together.

</details>

---

## Question 54:
What is the default acknowledgment setting for Kafka producers?

1. `acks=0`
2. `acks=1`
3. `acks=all`
4. `acks=-1`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `acks=1`**

The default acknowledgment setting is `acks=1`, which means the producer waits for acknowledgment from the leader replica only, providing a balance between performance and durability.

</details>

---

## Question 55:
Which configuration controls the maximum time a consumer will wait for the broker to return data in a fetch request?

1. `fetch.max.wait.ms`
2. `request.timeout.ms`
3. `session.timeout.ms`
4. `max.poll.interval.ms`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **1. `fetch.max.wait.ms`**

This property controls the maximum time the broker will wait before returning data if the `fetch.min.bytes` requirement isn't met, balancing between latency and throughput.

</details>

---

## Question 56:
In Kafka Streams, calling `KStream.to("output-topic")` does what?

1. Creates a new KStream
2. Writes records to the specified topic (terminal operation)
3. Converts the KStream to a KTable
4. Filters records before writing

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Writes records to the specified topic (terminal operation)**

`KStream.to()` is a terminal operation that writes the stream's records to a Kafka topic, ending the processing chain for that stream.

</details>

---

## Question 57:
To enable rack-aware replica assignment, which broker configuration must be set?

1. `replica.rack.aware=true`
2. `broker.rack=<rack-id>`
3. `enable.rack.awareness=true`
4. `rack.assignment.strategy=RackAwareAssignor`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `broker.rack=<rack-id>`**

Setting `broker.rack` to identify which rack each broker is in enables Kafka to distribute replicas across different racks for better fault tolerance.

</details>

---

## Question 58:
The `retries` configuration in a Kafka producer specifies:

1. Number of connection retry attempts
2. Number of times to retry failed send requests
3. Number of times to retry metadata requests
4. Number of times to retry offset commits

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. Number of times to retry failed send requests**

The `retries` setting controls how many times the producer will retry sending a record if it encounters a retriable error (like network timeouts or broker unavailability).

</details>

---

## Question 59:
Which Kafka Connect configuration sends failed records to a dead letter queue?

1. `errors.tolerance=all`
2. `errors.deadletterqueue.topic.name=<topic-name>`
3. `errors.log.enable=true`
4. `errors.retry.delay.max.ms=60000`

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. `errors.deadletterqueue.topic.name=<topic-name>`**

Setting this property to a topic name enables the dead letter queue functionality, directing failed records to the specified topic for later analysis or reprocessing.

</details>

---

## Question 60:
In KRaft mode (without ZooKeeper), where is the cluster metadata stored?

1. In each broker's local file system
2. In a special `__cluster_metadata` topic
3. In an external database
4. In memory across controller nodes

Choose the correct answer.

<details>
<summary>Response:</summary>

The correct answer is **2. In a special `__cluster_metadata` topic**

KRaft mode uses a dedicated internal topic to store cluster metadata, managed by the controller quorum, eliminating the need for ZooKeeper.

</details>

---

## Exam Summary

**Time**: 90 minutes  
**Total Questions**: 60  
**Passing Score**: 75% (45 correct answers)  

**Question Distribution**:
- Security & Authentication: 10%
- Producer Configuration: 15%
- Consumer Configuration: 20%
- Kafka Streams: 20%
- Kafka Connect: 10%
- Topic Management: 10%
- Monitoring & Troubleshooting: 10%
- CLI Tools: 5%

**Key Focus Areas Based on Real Exam Questions**:
- SSL/SASL authentication configurations
- Partition assignment strategies and consumer group behavior
- Log compaction and cleanup policies
- Exception handling in consumers (especially deserialization)
- Kafka Streams windowing and joins
- Testing tools and their proper usage
- Producer batching and performance tuning
- Offset management and consumer lag
- Schema Registry and Avro serialization
- Exactly-once processing configurations

**Critical Concepts to Master**:
- Understanding when to use `assign()` vs `subscribe()`
- Proper error handling with `seek()` for poison messages
- Difference between various assignment strategies
- Producer acknowledgment levels and their implications
- Kafka Streams topology testing with TopologyTestDriver
- Transaction configuration with `transactional.id`
- Consumer group rebalancing triggers and behavior
- Log compaction behavior with tombstone messages
