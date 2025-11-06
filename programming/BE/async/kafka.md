# Apache Kafka Interview Questions & Answers

## 1. What is Apache Kafka?

**Q: Explain Apache Kafka**

A: Apache Kafka is a distributed event streaming platform. High-throughput, low-latency publish-subscribe messaging system.

**Key characteristics**:

- Distributed architecture (runs on multiple servers)
- Fault-tolerant (replication across brokers)
- High throughput (millions of messages/second)
- Low latency (real-time processing)
- Persistent storage (messages stored on disk)
- Scalable (add brokers dynamically)
- Durable (data replication)

**Use cases**:

- Real-time analytics
- Log aggregation
- Stream processing
- Event sourcing
- Microservices communication
- Data pipeline
- Activity tracking
- IoT data

**vs Traditional Message Queues** (RabbitMQ, ActiveMQ):

```
Kafka:
- Pub-sub and queuing
- Replay messages (persistent)
- High throughput
- Stream processing

RabbitMQ:
- Traditional queuing
- Messages deleted after consumed
- Complex routing
- Lower throughput
```

---

## 2. What are core concepts?

**Q: Explain Kafka architecture**

A:

**Components**:

```
Producer — Sends messages
├─ Partitioner — Determines partition
├─ Compression — Optional
└─ Serializer — Converts to bytes

Topic — Named channel (like queue)
├─ Partition — Divided for parallelism
├─ Replica — Copy for durability
└─ Leader — Primary partition

Consumer — Reads messages
├─ Consumer Group — Multiple consumers
├─ Offset — Message position
└─ Deserializer — Converts from bytes

Broker — Kafka server instance
└─ Cluster — Multiple brokers
```

**Message flow**:

```
Producer → [Partitioner] → Partition 0 → Broker 1 (Leader)
                                      ↓
                                  Broker 2 (Replica)
                                  Broker 3 (Replica)

Consumer Group 1 → Reads from Topic
Consumer Group 2 → Also reads from Topic
(Both get all messages)
```

**Key terminology**:

```
Topic: myapp-events
Partition: 0, 1, 2
Replica Factor: 3
Leader: Broker with primary partition
ISR (In-Sync Replicas): Replicas in sync with leader
Offset: Message ID in partition
Consumer Group: Group of consumers sharing work
```

---

## 3. What are Producers?

**Q: How do you send messages to Kafka?**

A: Producer is client that publishes records to Kafka topics.

**Basic producer example** (Python):

```python
from kafka import KafkaProducer
import json

# Create producer
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Send message
future = producer.send('my-topic', {
    'name': 'John',
    'email': 'john@example.com'
})

# Wait for confirmation
record_metadata = future.get(timeout=10)
print(f"Sent to partition {record_metadata.partition} offset {record_metadata.offset}")

# Close producer
producer.close()
```

**Producer configurations**:

```python
KafkaProducer(
    bootstrap_servers=['localhost:9092'],

    # Key/Value serializers
    key_serializer=lambda k: k.encode('utf-8'),
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),

    # Acknowledgments
    acks='all',  # 'all', '1', '0'

    # Batching for better throughput
    batch_size=16384,  # bytes
    linger_ms=10,      # wait up to 10ms

    # Retries
    retries=3,
    retry_backoff_ms=100,

    # Compression
    compression_type='snappy',  # 'gzip', 'snappy', 'lz4'

    # Timeouts
    request_timeout_ms=30000
)
```

**Partitioning strategies**:

```python
# 1. Default (hash of key)
producer.send('topic', key=b'user-123', value=data)

# 2. Custom partitioner
class CustomPartitioner:
    def __call__(self, key, all_partitions, available_partitions):
        return hash(key) % len(all_partitions)

producer = KafkaProducer(
    partitioner=CustomPartitioner()
)

# 3. Explicit partition
producer.send('topic', value=data, partition=0)
```

**Acks and durability**:

```
acks='0' — Fire and forget (fastest, least durable)
acks='1' — Wait for leader acknowledgment (default)
acks='all' — Wait for all replicas (slowest, most durable)
```

**Error handling**:

```python
def on_send_success(record_metadata):
    print(f"Message sent to {record_metadata.topic}")

def on_send_error(exc):
    print(f"Error: {exc}")

future = producer.send('topic', value=data)
future.add_callback(on_send_success)
future.add_errback(on_send_error)
```

---

## 4. What are Consumers?

**Q: How do you read messages from Kafka?**

A: Consumer is client that subscribes to topics and reads records.

**Basic consumer example**:

```python
from kafka import KafkaConsumer
import json

# Create consumer
consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='my-group',
    value_deserializer=lambda m: json.loads(m.decode('utf-8')),
    auto_offset_reset='earliest',  # or 'latest'
    enable_auto_commit=True,
    auto_commit_interval_ms=1000
)

# Consume messages
for message in consumer:
    print(f"Topic: {message.topic}")
    print(f"Partition: {message.partition}")
    print(f"Offset: {message.offset}")
    print(f"Value: {message.value}")
    print(f"Timestamp: {message.timestamp}")
```

**Consumer group coordination**:

```
Topic with 3 partitions: [P0, P1, P2]

Scenario 1: 3 consumers in group
- Consumer 1 → P0
- Consumer 2 → P1
- Consumer 3 → P2
(Each gets 1 partition)

Scenario 2: 5 consumers in group
- Consumer 1 → P0
- Consumer 2 → P1
- Consumer 3 → P2
- Consumer 4 → (idle)
- Consumer 5 → (idle)
(3 idle consumers, waste of resources)

Scenario 3: 1 consumer in group
- Consumer 1 → P0, P1, P2
(One consumer gets all partitions)
```

**Consumer configurations**:

```python
KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],

    # Consumer group
    group_id='my-group',

    # Offset reset strategy
    auto_offset_reset='earliest',  # or 'latest', 'none'

    # Auto commit
    enable_auto_commit=True,
    auto_commit_interval_ms=1000,

    # Deserialization
    value_deserializer=lambda m: json.loads(m.decode('utf-8')),

    # Fetch sizes
    fetch_min_bytes=1,
    fetch_max_wait_ms=500,
    max_partition_fetch_bytes=1048576,  # 1MB

    # Session
    session_timeout_ms=10000,
    heartbeat_interval_ms=3000,

    # Isolation level
    isolation_level='read_committed'  # or 'read_uncommitted'
)
```

**Manual offset management**:

```python
consumer = KafkaConsumer(
    enable_auto_commit=False  # Manual control
)

for message in consumer:
    try:
        process_message(message)
        # Only commit after successful processing
        consumer.commit()
    except Exception as e:
        print(f"Error: {e}")
        # Offset not committed, message will be reprocessed
```

**Offset reset strategies**:

```
earliest — Start from beginning of topic
latest — Start from end of topic
none — Throw error if no committed offset
```

---

## 5. What are Topics and Partitions?

**Q: How do topics work?**

A: Topic is logical channel. Partitions provide parallelism and scalability.

**Topic structure**:

```
Topic: orders
├─ Partition 0: [msg0, msg1, msg2, ...]
├─ Partition 1: [msg0, msg1, msg2, ...]
├─ Partition 2: [msg0, msg1, ...]
└─ Partition 3: [msg0, msg1, ...]

Each partition is independent sequence (log)
Offsets are per partition, not per topic
```

**Replication**:

```
Topic: orders, Replication Factor: 3

Broker 1 (Leader)   Broker 2 (Replica)   Broker 3 (Replica)
├─ P0 (Leader)      ├─ P0 (Follower)     ├─ P0 (Follower)
├─ P1 (Replica)     ├─ P1 (Leader)       ├─ P1 (Replica)
├─ P2 (Replica)     ├─ P2 (Replica)      ├─ P2 (Leader)
└─ P3 (Leader)      └─ P3 (Leader)       └─ P3 (Replica)

In-Sync Replicas (ISR) — All replicas caught up with leader
```

**Creating topic** (CLI):

```bash
# Create topic
kafka-topics.sh --create \
    --topic orders \
    --partitions 4 \
    --replication-factor 3 \
    --bootstrap-server localhost:9092

# List topics
kafka-topics.sh --list \
    --bootstrap-server localhost:9092

# Describe topic
kafka-topics.sh --describe \
    --topic orders \
    --bootstrap-server localhost:9092

# Delete topic
kafka-topics.sh --delete \
    --topic orders \
    --bootstrap-server localhost:9092

# Alter topic (change partitions)
kafka-topics.sh --alter \
    --topic orders \
    --partitions 6 \
    --bootstrap-server localhost:9092
```

**Choosing partition count**:

```
Factors:
1. Throughput needed (each partition is sequential)
2. Number of consumers (max = partition count)
3. Retention requirements (each partition takes space)
4. Consumer lag tolerance

Rule of thumb:
- Start with 2-3 partitions per topic
- Increase if needed (can add, but not remove)
- Match number of consuming threads per topic
```

---

## 6. What are Consumer Groups?

**Q: How consumer groups work**

A: Multiple consumers share work of reading topic.

**How rebalancing works**:

```
Initial state: 3 consumers, 4 partitions
Consumer 1 → P0
Consumer 2 → P1, P2
Consumer 3 → P3

New consumer joins (4 consumers total)
REBALANCING...
Pause consumption during rebalancing

After rebalancing:
Consumer 1 → P0
Consumer 2 → P1
Consumer 3 → P2
Consumer 4 → P3

Resume consumption
```

**Rebalancing triggers**:

- Consumer joins group
- Consumer leaves group
- Consumer crashes
- Topics added/removed
- Partitions added

**Rebalancing strategies**:

```
RangeAssignor (default):
- Assign partitions sequentially
- Consumer 0: [0, 1], Consumer 1: [2, 3]
- Can cause imbalance

RoundRobinAssignor:
- Distribute evenly
- Consumer 0: [0, 2], Consumer 1: [1, 3]
- Better load balance

StickyAssignor:
- Minimize partition movement
- Less rebalancing overhead

CooperativeStickyAssignor:
- Newer, cooperative rebalancing
- Less pause time
```

**Stop-the-world problem**:

```
Traditional rebalancing:
1. All consumers pause
2. Rebalance happens
3. Resume consumption
(downtime during rebalance)

Cooperative rebalancing:
1. Some consumers pause
2. Rebalance happens
3. Resume with minimal downtime
```

---

## 7. What is replication and fault tolerance?

**Q: How Kafka ensures durability**

A:

**Replication mechanism**:

```
Producer sends message to leader
Leader appends to log
Leader waits for replicas to acknowledge
Only then acks to producer

If leader fails:
- Replicas detect failure
- New leader elected from ISR
- Consumers switch to new leader
```

**ISR (In-Sync Replicas)**:

```
Replicas that are caught up with leader
Configuration: min.insync.replicas=2

If < 2 replicas in ISR:
- acks='all' requests fail
- Prevents data loss
- At cost of availability
```

**Broker failure scenarios**:

```
Scenario 1: Leader fails, 1 replica behind
New leader elected from in-sync replica
No data loss

Scenario 2: Leader + 1 replica fail, 1 remains
Remaining replica becomes leader
No data loss (replication factor was 3)

Scenario 3: All replicas fail
Data lost (unrecoverable)
```

**Configuration**:

```
replication.factor=3         # Replicate to 3 brokers
min.insync.replicas=2        # At least 2 must be in sync
default.replication.factor=3 # Default for new topics
```

---

## 8. What are offsets and commits?

**Q: Track consumption progress**

A: Offset is position in partition. Consumer commits offset after processing.

**Offset management**:

```
Partition 0 log:
[0] Message 1
[1] Message 2
[2] Message 3
[3] Message 4
[4] Message 5

Consumer reads:
- Read offset 0-4
- Process messages
- Commit offset 4
- Next session starts from offset 5
```

**Offset storage locations**:

```
__consumer_offsets topic (default)
├─ Compact topic (keep latest offset per partition)
├─ Replicated
├─ Default group coordinator
└─ Automatic offset management

External (optional):
├─ Database
├─ Redis
├─ Custom store
```

**Offset commit options**:

```python
# Auto-commit (simpler, less control)
KafkaConsumer(
    enable_auto_commit=True,
    auto_commit_interval_ms=1000
)

# Manual commit (more control)
consumer = KafkaConsumer(
    enable_auto_commit=False
)

for message in consumer:
    process(message)

    # Commit after processing
    consumer.commit()

    # Or async commit
    consumer.commit_async()
```

**Offset reset behavior**:

```
Consumer group doesn't have committed offset:

auto_offset_reset='earliest' → Start from beginning
auto_offset_reset='latest'   → Start from end
auto_offset_reset='none'     → Raise error
```

---

## 9. What is exactly-once semantics?

**Q: Guarantee message processing**

A: Ensure messages processed exactly once, not lost or duplicated.

**Delivery guarantees**:

```
At-most-once: Message processed 0 or 1 time
              (can lose message)

At-least-once: Message processed 1 or more times
               (can duplicate)

Exactly-once: Message processed exactly 1 time
              (preferred but hardest)
```

**Achieving exactly-once**:

```
1. Idempotent producer (acks='all')
   - Deduplication on broker side

2. Transactional producer
   - Atomically write to multiple partitions
   - All-or-nothing

3. Consumer offset management
   - Commit offset after processing
   - If reprocess, use same offset

4. Idempotent consumer
   - Store processed IDs
   - Skip duplicates
```

**Transactional example**:

```python
producer = KafkaProducer(
    enable_idempotence=True,
    transactional_id='my-producer-1'
)

with producer.transaction():
    producer.send('orders', key=b'order-1', value=b'data1')
    producer.send('notifications', key=b'notif-1', value=b'data2')
    # Either both succeed or both rollback

# Consumer with exactly-once
consumer = KafkaConsumer(
    isolation_level='read_committed'  # Only read committed messages
)

for message in consumer:
    process(message)
    consumer.commit()  # Idempotent if reprocessed
```

---

## 10. What is retention and compaction?

**Q: How Kafka stores data long-term**

A:

**Retention policies**:

```
Time-based: Delete after 7 days
Size-based: Delete when partition exceeds 1GB
Log compaction: Keep latest version of each key

Configuration:
retention.ms=604800000      # 7 days
retention.bytes=1073741824  # 1GB
retention.policy=delete     # or 'compact'
```

**Segment storage**:

```
Topic partition as multiple segments:
[Segment 1 - closed (1GB)]
[Segment 2 - closed (1GB)]
[Segment 3 - active (growing)]

Old segments deleted based on retention
```

**Log compaction** (cleanup.policy='compact'):

```
Before compaction:
Key=user-1: Value=v1, offset=0
Key=user-2: Value=v2, offset=1
Key=user-1: Value=v3, offset=2  ← Duplicate key
Key=user-2: Value=v4, offset=3  ← Duplicate key
Key=user-1: Value=v5, offset=4  ← Duplicate key

After compaction (keep latest per key):
Key=user-1: Value=v5, offset=4
Key=user-2: Value=v4, offset=3

Useful for: State changes, changelog topics
```

---

## 11. What is Kafka Connect?

**Q: Integrate external systems**

A: Framework for building connectors to move data in/out of Kafka.

**Connector types**:

```
Source Connector — Pull from external system → Kafka
├─ Database CDC
├─ File systems
├─ APIs
└─ IoT devices

Sink Connector — Pull from Kafka → External system
├─ Databases
├─ Data warehouses
├─ Cloud storage
└─ Search engines
```

**Example: JDBC Source Connector**:

```json
{
  "name": "jdbc-source-postgres",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.source.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:postgresql://localhost:5432/mydb",
    "connection.user": "postgres",
    "connection.password": "password",
    "table.whitelist": "orders",
    "topic.prefix": "postgres_",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "poll.interval.ms": "5000"
  }
}
```

**Deploying connector** (CLI):

```bash
# Post connector config
curl -X POST \
  -H "Content-Type: application/json" \
  -d @connector.json \
  http://localhost:8083/connectors

# List connectors
curl http://localhost:8083/connectors

# Delete connector
curl -X DELETE http://localhost:8083/connectors/jdbc-source-postgres
```

---

## 12. What is Kafka Streams?

**Q: Stream processing in Kafka**

A: Library for building stream processing applications within Kafka.

**Basic topology**:

```
Source (Topic) → Stream → Process → Sink (Topic)

Process can be:
- Map: Transform each record
- Filter: Keep only matching records
- FlatMap: Transform to multiple records
- Aggregate: Combine records
- Join: Combine with other streams
```

**Example: Stream processing**:

```python
from kafka import KafkaConsumer, KafkaProducer
import json

consumer = KafkaConsumer(
    'orders',
    bootstrap_servers=['localhost:9092'],
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Stream processing
for message in consumer:
    order = message.value

    # Filter
    if order['amount'] > 100:
        # Transform
        event = {
            'order_id': order['id'],
            'status': 'high-value',
            'timestamp': order['timestamp']
        }

        # Send to another topic
        producer.send('high-value-orders', event)
```

**Kafka Streams (Java/Scala preferred)**:

```java
StreamsBuilder builder = new StreamsBuilder();

builder.stream("orders", Consumed.with(Serdes.String(), orderSerde))
    .filter((key, order) -> order.getAmount() > 100)
    .map((key, order) -> KeyValue.pair(order.getId(),
        new HighValueOrder(order)))
    .to("high-value-orders", Produced.with(Serdes.String(), hvoSerde));

KafkaStreams streams = new KafkaStreams(builder.build(),
    new StreamsConfig(getProperties()));
streams.start();
```

---

## 13. What are performance tuning?

**Q: Optimize Kafka**

A:

**Producer tuning**:

```
batch.size: Larger batch = higher throughput, higher latency
           Default: 16KB, Increase for throughput

linger.ms: Wait time before sending batch
          Default: 0 (send immediately)
          Increase to batch more messages

compression.type: snappy, gzip, lz4, zstd
                  Reduces network usage

buffer.memory: Total memory for pending messages
              Default: 32MB
              Increase if many partitions

acks: 'all' for durability, '1' for balance, '0' for speed
```

**Consumer tuning**:

```
fetch.min.bytes: Wait for minimum bytes before responding
                Default: 1 byte
                Increase to reduce network round trips

fetch.max.wait.ms: Maximum wait time
                  Default: 500ms
                  Increase to batch requests

max.poll.records: Maximum records returned per poll
                 Default: 500
                 Increase for higher throughput

session.timeout.ms: Heartbeat timeout
                   Default: 10s
                   Increase to handle slow processing
```

**Broker tuning**:

```
num.network.threads: Network IO threads
                    Increase for more connections

num.io.threads: Background IO threads
               Increase for throughput

num.replica.fetchers: Replication threads per broker
                     Increase for faster replication

log.segment.bytes: Max size per segment
                  Larger = fewer segments, less housekeeping
                  Default: 1GB

num.partitions: Default partitions per topic
               1 partition per 1MB/sec throughput target
```

---

## 14. What is monitoring and operations?

**Q: Manage Kafka cluster**

A:

**Key metrics to monitor**:

```
Producer:
- Record send rate
- Record send latency
- Record error rate

Consumer:
- Record lag (offset behind latest)
- Records consumed rate
- Consumer lag per partition

Broker:
- CPU usage
- Memory usage
- Disk usage
- Network throughput
- Replication lag
- Active client connections
```

**Monitoring with JMX**:

```bash
# Enable JMX
export KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote ..."
kafka-broker-start.sh

# Use tools like:
# - Prometheus + Grafana
# - JConsole
# - Datadog
# - New Relic
```

**Checking lag** (CLI):

```bash
# Consumer group lag
kafka-consumer-groups.sh \
    --bootstrap-server localhost:9092 \
    --group my-group \
    --describe

# Shows current offset vs log end offset per partition
```

**Common operations**:

```bash
# Add new broker
# 1. Start new broker with unique broker.id
# 2. Run partition reassignment
kafka-reassign-partitions.sh \
    --bootstrap-server localhost:9092 \
    --reassignment-json-file reassignment.json \
    --execute

# Remove broker
# 1. Reassign partitions away
# 2. Shutdown broker

# Rolling restart
# 1. Stop broker
# 2. Perform maintenance (upgrade, config change)
# 3. Start broker
# 4. Wait for recovery (ISR catch up)
# 5. Next broker
```

---

## 15. What is schema management?

**Q: Handle data formats**

A: Manage data schemas for Kafka messages.

**Options**:

```
1. JSON Schema
   - Schema as JSON
   - Validation at producer/consumer

2. Apache Avro
   - Compact binary format
   - Schema evolution
   - Used by Confluent

3. Protocol Buffers
   - Binary format
   - Schema evolution
   - Language agnostic

4. Apache Thrift
   - Similar to Protobuf
   - Less common
```

**Confluent Schema Registry**:

```python
from confluent_kafka import avro
from confluent_kafka.avro import AvroProducer

# Define schema
value_schema_str = """
{
    "type": "record",
    "name": "Order",
    "fields": [
        {"name": "order_id", "type": "string"},
        {"name": "customer_id", "type": "string"},
        {"name": "amount", "type": "double"},
        {"name": "timestamp", "type": "long"}
    ]
}
"""

# Produce with schema
conf = {'bootstrap.servers': 'localhost:9092',
        'schema.registry.url': 'http://localhost:8081'}

producer = AvroProducer(conf)

producer.produce(
    topic='orders',
    value={
        'order_id': '123',
        'customer_id': 'cust-456',
        'amount': 99.99,
        'timestamp': 1234567890
    }
)

producer.flush()
```

---

## 16. What are common use cases?

**Q: Real-world Kafka applications**

A:

**Log aggregation**:

```
Multiple servers → Kafka → Elasticsearch → Kibana
Each server sends logs to Kafka topic
Centralized log processing and search
```

**Real-time analytics**:

```
Events → Kafka → Stream processor → Analytics store
E.g., user events → Kafka → Flink → ClickHouse
Real-time dashboards and alerts
```

**Event sourcing**:

```
Events → Kafka (source of truth)
     ↓
Current state computed from event log
Can replay events to rebuild state
```

**CDC (Change Data Capture)**:

```
Database changes → Kafka → Multiple destinations
Sync data across systems
Maintain consistency
```

**Microservices communication**:

```
Service A → Kafka → Service B
Decoupled communication
Async requests
```

---

## 17. What is Kafka in cloud?

**Q: Managed Kafka services**

A:

**Confluent Cloud**:

- Fully managed Kafka
- Global deployment
- 99.95% SLA
- REST API
- ksqlDB included

**AWS MSK** (Managed Streaming for Kafka):

- AWS-managed Kafka
- Multi-AZ deployment
- Deep VPC integration
- CloudWatch monitoring
- IAM authentication

**Azure Event Hubs**:

- Compatible with Kafka protocol
- Fully managed
- Global scalability
- Azure integration

**Google Cloud Pub/Sub**:

- Not Kafka-compatible
- Different model (push vs pull)

```

---

## 18. What are best practices?

**Q: Kafka design patterns**

A:

**Topic design**:
```

- One topic per data type
- Partition by business logic (user_id, region, etc.)
- Start with 2-3 partitions, scale as needed
- Use naming conventions (events-orders, events-users)

```

**Producer**:
```

- Use appropriate acks level (balance speed/durability)
- Handle retries and backoff
- Monitor error rates
- Use batching for throughput
- Compress data for large messages

```

**Consumer**:
```

- Process idempotently (handle duplicates)
- Commit offsets after processing
- Use consumer groups for scaling
- Monitor lag
- Handle rebalancing gracefully

```

**Operations**:
```

- Monitor lag and errors
- Plan replication factor (minimum 3 for production)
- Use min.insync.replicas for safety
- Regular backups (export to external storage)
- Version control for connector configs
- Plan for capacity growth

```

---

## 19. What is troubleshooting?

**Q: Common Kafka issues**

A:

**Consumer lag**:
```

Symptoms: Consumers can't keep up
Causes: Slow processing, few consumers, few partitions

Solution:

- Increase partition count
- Add consumers (up to partition count)
- Optimize processing logic
- Increase max.poll.records

```

**Rebalancing issues**:
```

Symptoms: Frequent rebalancing, long stop time
Causes: Session timeout, slow processing, heartbeat issues

Solution:

- Increase session.timeout.ms
- Reduce processing time
- Tune heartbeat_interval_ms
- Check broker performance

```

**Data loss**:
```

Symptoms: Messages disappear
Causes: Low replication factor, acks=0, partition loss

Solution:

- Increase replication.factor
- Use acks='all'
- Monitor ISR (in-sync replicas)
- Implement monitoring/alerting

```

**Broker failures**:
```

Symptoms: Partition unavailable
Causes: Broker down, ISR too small

Solution:

- Bring broker back online
- Check disk space
- Review logs for errors
- Ensure sufficient min.insync.replicas

```

---

## 20. What is real-world architecture?

**Q: Complete Kafka system**

A: E-commerce event platform

```

Sources:
├─ Web app → Events producer
├─ Mobile app → Events producer
├─ Webhook → Events producer
└─ IoT devices → Kafka Connect source

Kafka Topics:
├─ events-clicks (4 partitions)
├─ events-orders (6 partitions)
├─ events-pageviews (4 partitions)
├─ events-inventory (3 partitions)
└─ events-errors (2 partitions)

Processing:
├─ Kafka Streams app → Real-time analytics
├─ Flink job → Complex event processing
├─ Spark job → Batch processing
└─ ksqlDB → SQL queries on streams

Sinks:
├─ Elasticsearch ← analytics-results topic
├─ PostgreSQL ← Kafka Connect sink
├─ S3 ← Kafka Connect sink
├─ Redis ← Cache service
└─ Snowflake ← Data warehouse

Monitoring:
├─ Prometheus → Metrics
├─ Grafana → Dashboards
├─ PagerDuty → Alerting
└─ ELK Stack → Logs

Configuration:
├─ 5 brokers in production
├─ Replication factor: 3
├─ Min ISR: 2
├─ Retention: 7 days
├─ Compression: snappy
└─ Security: SASL/SSL

```

---

## Kafka Interview Tips

1. **Core concepts** — Topics, partitions, brokers, offset
2. **Producers** — Serialization, partitioning, acks
3. **Consumers** — Groups, offset management, rebalancing
4. **Guarantees** — At-least-once, exactly-once semantics
5. **Replication** — ISR, durability, failures
6. **Performance** — Tuning, throughput, latency
7. **Monitoring** — Lag, errors, metrics
8. **Operations** — Scaling, maintenance, upgrades
9. **Integrations** — Connect, Streams, schemas
10. **Cloud options** — Confluent, AWS MSK, Azure
11. **Use cases** — Real-world patterns
12. **Architecture** — System design with Kafka
13. **Troubleshooting** — Common issues and fixes
14. **Best practices** — Design patterns
15. **Production experience** — Real Kafka deployments
```
