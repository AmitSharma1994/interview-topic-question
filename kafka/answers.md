# Apache Kafka Interview Questions and Answers

## Kafka Basics

### What is Apache Kafka?

**Answer:**
Apache Kafka is a distributed streaming platform designed for building real-time data pipelines and streaming applications. It's a publish-subscribe messaging system that can handle high-throughput, fault-tolerant, and scalable data streaming.

**Key characteristics:**
- **Distributed:** Runs as a cluster across multiple servers
- **Fault-tolerant:** Data is replicated across multiple brokers
- **High-throughput:** Can handle millions of messages per second
- **Low-latency:** Sub-millisecond latency for real-time processing
- **Durable:** Messages are persisted to disk
- **Scalable:** Can scale horizontally by adding more brokers

**Use cases:**
- Real-time analytics
- Log aggregation
- Event sourcing
- Stream processing
- Microservices communication
- Data integration

### What are the main components of Kafka?

**Answer:**
1. **Producer:** Applications that publish messages to Kafka topics
2. **Consumer:** Applications that subscribe to topics and process messages
3. **Broker:** Kafka servers that store and serve data
4. **Topic:** Categories or feed names to which messages are published
5. **Partition:** Ordered, immutable sequence of messages within a topic
6. **Zookeeper:** Coordination service for managing Kafka cluster metadata
7. **Consumer Group:** Group of consumers that work together to consume a topic

### What is the difference between Kafka and traditional messaging systems?

**Answer:**

| Feature | Kafka | Traditional MQ (JMS, RabbitMQ) |
|---------|-------|--------------------------------|
| Message Storage | Persistent, log-based | Queue-based, messages deleted after consumption |
| Throughput | Very high (millions/sec) | Moderate |
| Scalability | Horizontal scaling | Limited scaling |
| Message Ordering | Per-partition ordering | Global ordering (limited) |
| Consumer Model | Pull-based | Push/Pull-based |
| Replay Capability | Yes, can replay messages | No, messages consumed once |
| Durability | Configurable retention | Temporary storage |
| Use Case | Stream processing, big data | Traditional enterprise messaging |

## Kafka Architecture

### Explain Kafka's architecture

**Answer:**
Kafka follows a distributed architecture with the following components:

**1. Kafka Cluster:**
- Collection of Kafka brokers
- Each broker is identified by a unique ID
- Brokers coordinate through Zookeeper

**2. Topics and Partitions:**
- Topics are divided into partitions
- Each partition is an ordered, immutable log
- Partitions are distributed across brokers

**3. Replication:**
- Each partition has one leader and multiple followers
- Leader handles all reads and writes
- Followers replicate data from the leader

**4. Producers and Consumers:**
- Producers write to partition leaders
- Consumers read from partition leaders or followers
- Consumer groups enable parallel processing

**Visual Architecture:**
```
Producer → Kafka Cluster (Brokers) → Consumer
           ↓
        Zookeeper
```

### What is Zookeeper's role in Kafka?

**Answer:**
Zookeeper serves as the coordination service for Kafka cluster:

**Key responsibilities:**
1. **Broker Management:** Maintains list of active brokers
2. **Leader Election:** Elects partition leaders when brokers fail
3. **Configuration Management:** Stores topic and partition metadata
4. **Consumer Group Coordination:** Manages consumer group membership
5. **Access Control:** Stores ACL information for security

**Important Note:** Starting with Kafka 2.8, Kafka can run without Zookeeper using KRaft (Kafka Raft) mode, where Kafka manages its own metadata.

## Topics and Partitions

### What is a Kafka topic?

**Answer:**
A topic is a category or feed name to which messages are published. Topics are the fundamental unit of organization in Kafka.

**Key characteristics:**
- **Logical grouping:** Messages of the same type go to the same topic
- **Partitioned:** Topics are split into partitions for scalability
- **Immutable:** Messages in topics cannot be modified
- **Configurable retention:** Messages can be retained for a specified time or size

**Example:**
```bash
# Create a topic
kafka-topics.sh --create --topic user-events --bootstrap-server localhost:9092 --partitions 3 --replication-factor 2
```

### What are partitions and why are they important?

**Answer:**
Partitions are ordered, immutable sequences of messages within a topic. Each partition is a separate log file.

**Importance:**
1. **Scalability:** Allows parallel processing across multiple consumers
2. **Ordering:** Guarantees message order within a partition
3. **Distribution:** Partitions are distributed across different brokers
4. **Performance:** Enables high throughput by parallel processing

**Key concepts:**
- **Partition Key:** Determines which partition a message goes to
- **Offset:** Unique identifier for each message within a partition
- **Leader/Follower:** Each partition has one leader and multiple replicas

**Example:**
```java
// Producer with partition key
producer.send(new ProducerRecord<>("user-events", userId, userEvent));
```

### How does Kafka ensure message ordering?

**Answer:**
Kafka provides ordering guarantees at the partition level:

**Ordering guarantees:**
1. **Within partition:** Messages are strictly ordered by offset
2. **Across partitions:** No ordering guarantee
3. **Producer ordering:** Messages from same producer to same partition are ordered
4. **Consumer ordering:** Consumers read messages in offset order

**Best practices for ordering:**
- Use partition keys to route related messages to same partition
- Use single partition for strict global ordering (limits scalability)
- Design application to handle out-of-order messages across partitions

**Example:**
```java
// Ensure user events are ordered by using userId as partition key
producer.send(new ProducerRecord<>("user-events", user.getId(), event));
```

## Producers

### How does a Kafka producer work?

**Answer:**
A Kafka producer publishes messages to Kafka topics. The producer handles message serialization, partitioning, and delivery.

**Producer workflow:**
1. **Serialize:** Convert message to bytes
2. **Partition:** Determine target partition
3. **Batch:** Group messages for efficiency
4. **Send:** Transmit to broker
5. **Acknowledge:** Receive confirmation

**Key configurations:**
- `bootstrap.servers`: Kafka broker addresses
- `key.serializer`: Serializer for message keys
- `value.serializer`: Serializer for message values
- `acks`: Acknowledgment level (0, 1, all)
- `retries`: Number of retry attempts

**Example:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("acks", "all");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("my-topic", "key", "value"));
```

### What are the different acknowledgment levels in Kafka?

**Answer:**
Acknowledgment levels control the durability guarantees for produced messages:

**1. acks=0 (Fire and forget):**
- Producer doesn't wait for acknowledgment
- Highest throughput, lowest durability
- Messages may be lost if broker fails

**2. acks=1 (Leader acknowledgment):**
- Producer waits for leader broker acknowledgment
- Balanced throughput and durability
- Messages may be lost if leader fails before replication

**3. acks=all or acks=-1 (All replicas):**
- Producer waits for all in-sync replicas to acknowledge
- Highest durability, lower throughput
- Messages are not lost as long as one replica survives

**Example:**
```java
props.put("acks", "all");  // Wait for all replicas
props.put("min.insync.replicas", "2");  // Minimum replicas that must acknowledge
```

### What is idempotent producer?

**Answer:**
An idempotent producer ensures that duplicate messages are not written to Kafka, even if the producer retries sending the same message.

**How it works:**
- Producer assigns sequence numbers to messages
- Broker tracks sequence numbers per producer
- Duplicate messages (same sequence number) are ignored
- Enabled by setting `enable.idempotence=true`

**Benefits:**
- Prevents duplicate messages during retries
- Maintains exactly-once semantics
- No application-level deduplication needed

**Example:**
```java
props.put("enable.idempotence", "true");
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
props.put("max.in.flight.requests.per.connection", "5");
```

## Consumers

### How does a Kafka consumer work?

**Answer:**
A Kafka consumer subscribes to topics and processes messages. Consumers pull messages from brokers rather than having messages pushed to them.

**Consumer workflow:**
1. **Subscribe:** Subscribe to one or more topics
2. **Poll:** Request messages from broker
3. **Deserialize:** Convert bytes back to objects
4. **Process:** Handle the message
5. **Commit:** Acknowledge message processing

**Key configurations:**
- `bootstrap.servers`: Kafka broker addresses
- `group.id`: Consumer group identifier
- `key.deserializer`: Deserializer for keys
- `value.deserializer`: Deserializer for values
- `auto.offset.reset`: What to do when no offset exists

**Example:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-consumer-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println("Key: " + record.key() + ", Value: " + record.value());
    }
}
```

### What is a consumer group?

**Answer:**
A consumer group is a group of consumers that work together to consume messages from a topic. Each message is delivered to only one consumer within the group.

**Key characteristics:**
- **Load balancing:** Partitions are distributed among consumers
- **Fault tolerance:** If a consumer fails, its partitions are reassigned
- **Scalability:** Add more consumers to increase processing capacity
- **Unique consumption:** Each message is consumed by only one consumer in the group

**Partition assignment:**
- Each partition is assigned to exactly one consumer in the group
- A consumer can handle multiple partitions
- Number of consumers ≤ number of partitions for optimal utilization

**Example:**
```java
// Consumer 1
props.put("group.id", "payment-processors");
consumer1.subscribe(Arrays.asList("payment-events"));

// Consumer 2 (same group)
props.put("group.id", "payment-processors");
consumer2.subscribe(Arrays.asList("payment-events"));
```

### What is offset management in Kafka?

**Answer:**
Offset management tracks the position of each consumer in each partition, enabling consumers to resume from where they left off.

**Offset types:**
1. **Current offset:** Last message read by consumer
2. **Committed offset:** Last offset confirmed as processed
3. **Log-end offset:** Latest offset in partition

**Offset commit strategies:**
1. **Auto-commit:** Automatically commits offsets periodically
2. **Manual commit:** Application explicitly commits offsets
3. **Sync commit:** Blocks until commit completes
4. **Async commit:** Non-blocking commit with callback

**Example:**
```java
// Auto-commit (default)
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "1000");

// Manual commit
props.put("enable.auto.commit", "false");
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }
    consumer.commitSync();  // Commit after processing all messages
}
```

## Brokers and Clusters

### What is a Kafka broker?

**Answer:**
A Kafka broker is a server that stores and serves data. Multiple brokers form a Kafka cluster.

**Broker responsibilities:**
1. **Message storage:** Persist messages to disk
2. **Client requests:** Handle producer and consumer requests
3. **Replication:** Maintain replicas of partitions
4. **Leadership:** Act as leader for assigned partitions
5. **Metadata management:** Coordinate with other brokers

**Broker identification:**
- Each broker has a unique `broker.id`
- Brokers register themselves with Zookeeper
- Clients discover brokers through bootstrap servers

### How does Kafka achieve high availability?

**Answer:**
Kafka achieves high availability through replication and distributed architecture:

**1. Replication:**
- Each partition has multiple replicas across different brokers
- One replica is the leader, others are followers
- Replication factor determines number of replicas

**2. Leader election:**
- If leader fails, a follower becomes the new leader
- In-sync replicas (ISR) are eligible for leadership
- Automatic failover ensures continuous availability

**3. Distributed storage:**
- Data is distributed across multiple brokers
- No single point of failure
- Cluster continues operating even if some brokers fail

**Example configuration:**
```bash
# Create topic with replication factor 3
kafka-topics.sh --create --topic critical-events --bootstrap-server localhost:9092 --partitions 6 --replication-factor 3
```

### What are In-Sync Replicas (ISR)?

**Answer:**
In-Sync Replicas are replicas that are caught up with the leader partition and are eligible to become leaders.

**ISR criteria:**
- Replica is alive and connected to Zookeeper
- Replica has fetched messages from leader within `replica.lag.time.max.ms`
- Replica is not too far behind in terms of messages

**ISR management:**
- Leader maintains ISR list
- Replicas are added/removed from ISR dynamically
- Only ISR members can become leaders
- `min.insync.replicas` sets minimum ISR size for writes

**Example:**
```bash
# Check ISR status
kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092
```

## Replication and Fault Tolerance

### How does Kafka replication work?

**Answer:**
Kafka replication ensures data durability and availability by maintaining multiple copies of each partition.

**Replication process:**
1. **Leader assignment:** One replica becomes the leader
2. **Follower replication:** Followers continuously fetch from leader
3. **ISR maintenance:** Track which replicas are in-sync
4. **Failover:** Elect new leader if current leader fails

**Replication configurations:**
- `replication.factor`: Number of replicas per partition
- `min.insync.replicas`: Minimum replicas for writes
- `unclean.leader.election.enable`: Allow out-of-sync replicas to become leaders

**Example:**
```java
// Producer configuration for strong durability
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
props.put("enable.idempotence", "true");
```

### What happens when a broker fails?

**Answer:**
When a broker fails, Kafka automatically handles the failure:

**Immediate actions:**
1. **Detection:** Zookeeper detects broker failure
2. **Leader election:** New leaders elected for affected partitions
3. **ISR update:** Failed broker removed from ISR lists
4. **Client redirection:** Clients redirect to new leaders

**Recovery process:**
1. **Broker restart:** Failed broker rejoins cluster
2. **Replica sync:** Catches up with current leaders
3. **ISR rejoin:** Added back to ISR when caught up
4. **Load rebalancing:** May become leader for some partitions

**Impact mitigation:**
- Clients experience brief unavailability during leader election
- No data loss if `min.insync.replicas` is properly configured
- Automatic recovery without manual intervention

## Performance and Optimization

### How can you optimize Kafka performance?

**Answer:**
Kafka performance can be optimized at multiple levels:

**1. Producer optimizations:**
```java
// Batch configuration
props.put("batch.size", 16384);
props.put("linger.ms", 5);
props.put("buffer.memory", 33554432);

// Compression
props.put("compression.type", "snappy");

// Parallelism
props.put("max.in.flight.requests.per.connection", 5);
```

**2. Consumer optimizations:**
```java
// Fetch configuration
props.put("fetch.min.bytes", 1024);
props.put("fetch.max.wait.ms", 500);
props.put("max.partition.fetch.bytes", 1048576);

// Processing parallelism
props.put("max.poll.records", 500);
```

**3. Broker optimizations:**
- Use SSDs for better I/O performance
- Increase `num.network.threads` and `num.io.threads`
- Tune JVM heap size and garbage collection
- Configure appropriate `log.segment.bytes` and `log.retention.hours`

**4. Topic/Partition design:**
- Choose optimal number of partitions
- Use appropriate partition keys
- Balance partition distribution across brokers

### What is the impact of partition count on performance?

**Answer:**
Partition count significantly affects Kafka performance:

**Benefits of more partitions:**
- Higher parallelism for producers and consumers
- Better load distribution across brokers
- Increased throughput potential

**Drawbacks of too many partitions:**
- Higher memory usage (each partition uses memory)
- Longer leader election times
- More file handles and network connections
- Increased end-to-end latency

**Guidelines:**
- Start with moderate partition count (6-12 per broker)
- Consider consumer parallelism requirements
- Monitor resource usage and adjust accordingly
- Plan for future growth but avoid over-partitioning

**Example calculation:**
```
Target throughput: 100 MB/s
Single partition throughput: 10 MB/s
Minimum partitions needed: 100/10 = 10 partitions
```

## Kafka Connect

### What is Kafka Connect?

**Answer:**
Kafka Connect is a framework for connecting Kafka with external systems like databases, file systems, and cloud services.

**Key features:**
- **Scalable:** Distributed mode for high throughput
- **Fault-tolerant:** Automatic failure handling and recovery
- **Configuration-driven:** No coding required for standard connectors
- **Pluggable:** Extensible with custom connectors

**Connector types:**
1. **Source connectors:** Import data from external systems to Kafka
2. **Sink connectors:** Export data from Kafka to external systems

**Example configuration:**
```json
{
  "name": "jdbc-source-connector",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url": "jdbc:mysql://localhost:3306/mydb",
    "table.whitelist": "users,orders",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "mysql-"
  }
}
```

### What are the different modes of Kafka Connect?

**Answer:**
Kafka Connect can run in two modes:

**1. Standalone mode:**
- Single process runs connectors
- Configuration stored in files
- Suitable for development and small deployments
- No fault tolerance or scalability

**2. Distributed mode:**
- Multiple worker processes form a cluster
- Configuration stored in Kafka topics
- Automatic load balancing and fault tolerance
- Scalable by adding more workers

**Comparison:**

| Feature | Standalone | Distributed |
|---------|------------|-------------|
| Scalability | Limited | High |
| Fault tolerance | None | Automatic |
| Configuration | File-based | Topic-based |
| Use case | Development | Production |

## Kafka Streams

### What is Kafka Streams?

**Answer:**
Kafka Streams is a client library for building stream processing applications that process data stored in Kafka.

**Key features:**
- **Stream processing:** Real-time data transformation
- **Stateful operations:** Windowing, joins, aggregations
- **Fault tolerance:** Automatic recovery and rebalancing
- **Exactly-once semantics:** Guaranteed message processing
- **No external dependencies:** Runs as part of your application

**Core concepts:**
- **Stream:** Unbounded sequence of key-value pairs
- **Table:** Changelog stream representing current state
- **Topology:** Processing graph of streams and operations

**Example:**
```java
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> source = builder.stream("input-topic");

source.filter((key, value) -> value.length() > 5)
      .mapValues(value -> value.toUpperCase())
      .to("output-topic");

KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

### What are the different types of operations in Kafka Streams?

**Answer:**
Kafka Streams provides two types of operations:

**1. Stateless operations:**
- Don't require state storage
- Process each record independently
- Examples: filter, map, flatMap, foreach

**2. Stateful operations:**
- Require state storage (backed by RocksDB)
- Process records in context of previous records
- Examples: aggregate, reduce, join, windowing

**Stateless examples:**
```java
// Filter records
stream.filter((key, value) -> value.contains("important"));

// Transform values
stream.mapValues(value -> value.toUpperCase());

// Flat map
stream.flatMapValues(value -> Arrays.asList(value.split(" ")));
```

**Stateful examples:**
```java
// Count words
stream.groupByKey()
      .count(Materialized.as("word-counts"));

// Windowed aggregation
stream.groupByKey()
      .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
      .aggregate(initializer, aggregator, materialized);
```

## Security

### How do you secure a Kafka cluster?

**Answer:**
Kafka security involves multiple layers:

**1. Authentication:**
- **SASL/PLAIN:** Username/password authentication
- **SASL/SCRAM:** Salted Challenge Response Authentication
- **SASL/GSSAPI:** Kerberos authentication
- **SSL:** Certificate-based authentication

**2. Authorization:**
- **ACLs (Access Control Lists):** Fine-grained permissions
- **Resource types:** Topics, consumer groups, cluster
- **Operations:** Read, write, create, delete, etc.

**3. Encryption:**
- **SSL/TLS:** Encrypt data in transit
- **Client-broker encryption:** Secure client connections
- **Inter-broker encryption:** Secure broker communication

**Example SSL configuration:**
```properties
# Broker configuration
listeners=SSL://localhost:9093
security.inter.broker.protocol=SSL
ssl.keystore.location=/path/to/kafka.server.keystore.jks
ssl.keystore.password=password
ssl.key.password=password
ssl.truststore.location=/path/to/kafka.server.truststore.jks
ssl.truststore.password=password
```

### What are Kafka ACLs?

**Answer:**
Access Control Lists (ACLs) provide fine-grained authorization for Kafka resources.

**ACL components:**
- **Principal:** User or service account
- **Resource:** Topic, consumer group, cluster
- **Operation:** Read, write, create, delete, etc.
- **Permission:** Allow or deny

**Common ACL patterns:**
```bash
# Allow user to produce to topic
kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 \
  --add --allow-principal User:producer-user \
  --operation Write --topic my-topic

# Allow user to consume from topic
kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 \
  --add --allow-principal User:consumer-user \
  --operation Read --topic my-topic \
  --operation Read --group my-consumer-group
```

## Monitoring and Operations

### How do you monitor Kafka?

**Answer:**
Kafka monitoring involves tracking multiple metrics:

**1. Broker metrics:**
- CPU, memory, disk usage
- Network I/O
- Request rates and latencies
- Under-replicated partitions

**2. Topic/Partition metrics:**
- Message rates
- Partition sizes
- Leader distribution
- ISR status

**3. Producer metrics:**
- Send rates
- Batch sizes
- Error rates
- Request latencies

**4. Consumer metrics:**
- Lag per partition
- Consumption rates
- Commit frequencies
- Rebalance frequency

**Monitoring tools:**
- **JMX:** Built-in metrics exposure
- **Kafka Manager:** Web-based cluster management
- **Confluent Control Center:** Commercial monitoring solution
- **Prometheus + Grafana:** Open-source monitoring stack

**Example JMX metrics:**
```bash
# Check consumer lag
kafka.consumer:type=consumer-fetch-manager-metrics,client-id=consumer-1,topic=my-topic,partition=0
```

### What are the key operational tasks for Kafka?

**Answer:**
Key operational tasks include:

**1. Cluster management:**
- Adding/removing brokers
- Rebalancing partitions
- Upgrading Kafka versions
- Managing configurations

**2. Topic management:**
- Creating/deleting topics
- Modifying partition counts
- Changing retention policies
- Managing replication factors

**3. Performance tuning:**
- Monitoring metrics
- Adjusting configurations
- Optimizing partition distribution
- Scaling consumers/producers

**4. Backup and recovery:**
- Regular configuration backups
- Data replication strategies
- Disaster recovery planning
- Testing recovery procedures

**Example operations:**
```bash
# Add partitions to topic
kafka-topics.sh --alter --topic my-topic --partitions 10 --bootstrap-server localhost:9092

# Reassign partitions
kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --reassignment-json-file reassignment.json --execute
```

## Advanced Topics

### What is exactly-once semantics in Kafka?

**Answer:**
Exactly-once semantics ensures that each message is processed exactly once, even in the presence of failures.

**Components:**
1. **Idempotent producer:** Prevents duplicate writes
2. **Transactional producer:** Atomic writes across partitions
3. **Transactional consumer:** Reads only committed transactions

**Implementation:**
```java
// Producer configuration
props.put("enable.idempotence", "true");
props.put("transactional.id", "my-transactional-id");

// Initialize transactions
producer.initTransactions();

try {
    producer.beginTransaction();
    producer.send(new ProducerRecord<>("topic1", "key", "value"));
    producer.send(new ProducerRecord<>("topic2", "key", "value"));
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
}
```

### What is log compaction in Kafka?

**Answer:**
Log compaction is a retention policy that keeps only the latest value for each key in a topic.

**How it works:**
- Kafka maintains the latest value for each key
- Older values for the same key are eventually removed
- Tombstone records (null values) delete keys
- Compaction runs in background threads

**Use cases:**
- Database change streams
- Configuration management
- User profile updates
- Any scenario requiring latest state per key

**Configuration:**
```properties
# Enable log compaction
log.cleanup.policy=compact
log.segment.ms=604800000
log.min.cleanable.dirty.ratio=0.1
```

### What are Kafka quotas?

**Answer:**
Kafka quotas limit the rate at which clients can produce or consume data, preventing any single client from overwhelming the cluster.

**Quota types:**
1. **Producer quotas:** Limit produce request rate
2. **Consumer quotas:** Limit fetch request rate
3. **Request quotas:** Limit request rate per client

**Configuration:**
```bash
# Set producer quota (bytes/sec)
kafka-configs.sh --bootstrap-server localhost:9092 --alter \
  --add-config 'producer_byte_rate=1024000' \
  --entity-type clients --entity-name client-id

# Set consumer quota
kafka-configs.sh --bootstrap-server localhost:9092 --alter \
  --add-config 'consumer_byte_rate=2048000' \
  --entity-type clients --entity-name client-id
```

### What is KRaft mode in Kafka?

**Answer:**
KRaft (Kafka Raft) is Kafka's new consensus protocol that eliminates the dependency on Zookeeper.

**Benefits:**
- Simplified architecture
- Better scalability
- Faster metadata operations
- Reduced operational complexity
- Improved recovery times

**Key concepts:**
- **Controller quorum:** Replicated log for metadata
- **Raft consensus:** Leader election and log replication
- **Metadata log:** Single source of truth for cluster state

**Migration considerations:**
- Available from Kafka 2.8+ (experimental)
- Production-ready from Kafka 3.3+
- Requires careful migration planning
- Not compatible with Zookeeper-based clusters