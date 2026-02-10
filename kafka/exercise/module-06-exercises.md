# Module 6 Exercises: Topics, Partitions, and Data Management

## Exercise 1: Topic Design and Planning (45 minutes)

**Objective:** Design topics for different use cases

### Scenario 1: E-Commerce Platform

Design topics for:
- User actions (clicks, views, searches)
- Order processing
- Inventory updates
- Payment transactions

**Tasks:**
For each topic, specify:
1. Topic name
2. Number of partitions
3. Replication factor
4. Retention policy (time/size/compact)
5. Key strategy
6. Expected throughput

**Template:**
```
Topic: user-actions
Partitions: ?
Replication Factor: ?
Retention: ?
Key: ?
Throughput: ? messages/sec
Rationale: ?
```

### Scenario 2: IoT Sensor Network

**Requirements:**
- 10,000 IoT devices
- Each sends data every 10 seconds
- Data: temperature, humidity, pressure
- Historical analysis for 90 days
- Real-time alerting

**Design:**
1. How many topics? (One topic vs topic per sensor vs topic per metric?)
2. Partition count calculation
3. Storage requirements
4. Compaction strategy

---

## Exercise 2: Partition Count Optimization (60 minutes)

**Objective:** Calculate optimal partition count

### Formula Review:
```
Partitions = max(
    Throughput_Required / Throughput_Per_Partition,
    Max_Consumers_Needed
)
```

### Part A: Throughput-Based Calculation

**Given:**
- Required throughput: 100 MB/sec
- Producer throughput per partition: 10 MB/sec
- Consumer throughput per partition: 10 MB/sec

**Questions:**
1. Minimum partitions needed for producers?
2. Minimum partitions needed for consumers?
3. Recommended partition count?
4. What about headroom for growth?

### Part B: Real-World Scenario

**Requirements:**
- Topic: `payment-events`
- Current throughput: 5,000 msg/sec
- Message size: 2 KB
- Expected growth: 3x in next year
- Current consumers: 3
- Max expected consumers: 10

**Calculate:**
1. Current throughput in MB/sec: ?
2. Future throughput in MB/sec: ?
3. Minimum partitions for current load: ?
4. Recommended partitions with growth factor: ?

**Implementation:**
```bash
# Create topic with calculated partitions
kafka-topics.sh --create \
  --topic payment-events \
  --partitions ? \
  --replication-factor 3 \
  --bootstrap-server localhost:9092
```

### Part C: Benchmark

**Test different partition counts:**
```bash
# Create topics with different partition counts
for partitions in 3 6 12 24; do
    kafka-topics.sh --create \
      --topic "perf-test-$partitions" \
      --partitions $partitions \
      --replication-factor 1 \
      --bootstrap-server localhost:9092
    
    # Run performance test
    kafka-producer-perf-test.sh \
      --topic "perf-test-$partitions" \
      --num-records 100000 \
      --record-size 1024 \
      --throughput -1 \
      --producer-props bootstrap.servers=localhost:9092
done
```

**Record results:**
```
Partitions | Throughput (MB/sec) | Avg Latency | Notes
-----------|---------------------|-------------|-------
3          | ?                   | ?           | ?
6          | ?                   | ?           | ?
12         | ?                   | ?           | ?
24         | ?                   | ?           | ?
```

---

## Exercise 3: Retention Configuration (45 minutes)

**Objective:** Configure retention for different scenarios

### Scenario 1: Application Logs

**Requirements:**
- Keep last 7 days
- Max 500 GB per topic

**Configuration:**
```bash
kafka-topics.sh --create \
  --topic app-logs \
  --partitions 12 \
  --replication-factor 3 \
  --config retention.ms=? \
  --config retention.bytes=? \
  --bootstrap-server localhost:9092
```

**Calculate:**
- `retention.ms` = ?
- `retention.bytes` per partition = ?

### Scenario 2: Financial Transactions

**Requirements:**
- Keep for 7 years (regulatory compliance)
- Compress data aggressively

**Configuration:**
```bash
kafka-topics.sh --create \
  --topic financial-transactions \
  --partitions 6 \
  --replication-factor 3 \
  --config retention.ms=? \
  --config compression.type=? \
  --config segment.ms=? \
  --bootstrap-server localhost:9092
```

### Scenario 3: Real-Time Cache

**Requirements:**
- Keep only latest value per key
- Unlimited time retention
- Fast lookup

**Configuration:**
```bash
kafka-topics.sh --create \
  --topic user-cache \
  --partitions 6 \
  --replication-factor 3 \
  --config cleanup.policy=? \
  --config min.cleanable.dirty.ratio=? \
  --config segment.ms=? \
  --bootstrap-server localhost:9092
```

### Scenario 4: Session Data

**Requirements:**
- Keep for 24 hours OR max 100 GB
- Whichever comes first

**Configuration:**
```bash
# Both retention.ms and retention.bytes
```

---

## Exercise 4: Log Compaction Deep Dive (60 minutes)

**Objective:** Understand and implement log compaction

### Part A: Setup Compacted Topic

```bash
kafka-topics.sh --create \
  --topic user-profiles \
  --partitions 3 \
  --replication-factor 1 \
  --config cleanup.policy=compact \
  --config segment.ms=60000 \
  --config min.cleanable.dirty.ratio=0.01 \
  --config delete.retention.ms=86400000 \
  --bootstrap-server localhost:9092
```

### Part B: Test Compaction

**Produce data:**
```bash
# Produce user profiles with updates
kafka-console-producer.sh \
  --topic user-profiles \
  --bootstrap-server localhost:9092 \
  --property "parse.key=true" \
  --property "key.separator=:"

# Enter:
user1:{"name":"Alice","email":"alice@example.com","v":1}
user2:{"name":"Bob","email":"bob@example.com","v":1}
user1:{"name":"Alice","email":"alice@newdomain.com","v":2}
user3:{"name":"Charlie","email":"charlie@example.com","v":1}
user1:{"name":"Alice Updated","email":"alice@newdomain.com","v":3}
user2:null
```

**Wait for compaction, then consume:**
```bash
kafka-console-consumer.sh \
  --topic user-profiles \
  --from-beginning \
  --bootstrap-server localhost:9092 \
  --property print.key=true
```

**Expected Output:**
```
user1:{"name":"Alice Updated","email":"alice@newdomain.com","v":3}
user3:{"name":"Charlie","email":"charlie@example.com","v":1}
# Note: user2 is deleted (null value)
```

### Part C: Implement Compaction-Aware Consumer

```typescript
import { Kafka, Consumer } from 'kafkajs';

interface UserProfile {
  name: string;
  email: string;
  version: number;
}

class CompactedTopicConsumer {
  private kafka: Kafka;
  private consumer: Consumer;
  
  constructor() {
    this.kafka = new Kafka({
      brokers: ['localhost:9092']
    });
    
    this.consumer = this.kafka.consumer({
      groupId: 'compacted-reader'
    });
  }
  
  // Build current state from compacted topic
  async buildState(): Promise<Map<string, UserProfile>> {
    const state = new Map<string, UserProfile>();
    
    await this.consumer.connect();
    await this.consumer.subscribe({ topic: 'user-profiles', fromBeginning: true });
    
    let messagesProcessed = 0;
    let noMessagesCount = 0;
    
    await this.consumer.run({
      eachMessage: async ({ message }) => {
        messagesProcessed++;
        
        const key = message.key?.toString();
        if (!key) return;
        
        if (message.value === null) {
          // Tombstone - delete key
          state.delete(key);
        } else {
          // Update state
          const profile: UserProfile = JSON.parse(message.value.toString());
          state.set(key, profile);
        }
      }
    });
    
    // Wait until we've consumed all existing messages
    // In practice, you'd handle this with a timeout or offset tracking
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    await this.consumer.disconnect();
    return state;
  }
  
  private parseProfile(value: string): UserProfile {
    return JSON.parse(value);
  }
}

// Usage
const consumer = new CompactedTopicConsumer();
const state = await consumer.buildState();
console.log(`Loaded ${state.size} user profiles`);
```

**Tasks:**
1. Implement the state builder
2. Produce 1000 user profiles with random updates
3. Let compaction run
4. Rebuild state and verify consistency

---

## Exercise 5: Schema Management with Schema Registry (60 minutes)

**Objective:** Implement schema evolution

### Part A: Setup Schema Registry

Add to `docker-compose.yml`:
```yaml
schema-registry:
  image: confluentinc/cp-schema-registry:latest
  ports:
    - "8081:8081"
  environment:
    SCHEMA_REGISTRY_HOST_NAME: schema-registry
    SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: kafka-1:9092
```

### Part B: Define Avro Schema

**user-v1.avsc:**
```json
{
  "type": "record",
  "name": "User",
  "namespace": "com.example",
  "fields": [
    {"name": "id", "type": "string"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": "string"}
  ]
}
```

### Part C: Register Schema

```bash
curl -X POST http://localhost:8081/subjects/users-value/versions \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{
    "schema": "{\"type\":\"record\",\"name\":\"User\",\"namespace\":\"com.example\",\"fields\":[{\"name\":\"id\",\"type\":\"string\"},{\"name\":\"name\",\"type\":\"string\"},{\"name\":\"email\",\"type\":\"string\"}]}"
  }'
```

### Part D: Produce with Schema

```java
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, KafkaAvroSerializer.class);
props.put("schema.registry.url", "http://localhost:8081");

KafkaProducer<String, GenericRecord> producer = new KafkaProducer<>(props);

// Create Avro record
Schema schema = new Schema.Parser().parse(new File("user-v1.avsc"));
GenericRecord user = new GenericData.Record(schema);
user.put("id", "user-1");
user.put("name", "Alice");
user.put("email", "alice@example.com");

ProducerRecord<String, GenericRecord> record = 
    new ProducerRecord<>("users", "user-1", user);
producer.send(record);
```

### Part E: Evolve Schema (Add Field)

**user-v2.avsc:**
```json
{
  "type": "record",
  "name": "User",
  "namespace": "com.example",
  "fields": [
    {"name": "id", "type": "string"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": "string"},
    {"name": "phone", "type": ["null", "string"], "default": null}
  ]
}
```

**Tasks:**
1. Register new schema version
2. Produce records with v2 schema
3. Verify old consumers can still read (backward compatibility)
4. Verify new consumers can read old records (forward compatibility)

### Part F: Test Schema Compatibility

```bash
# Set compatibility mode
curl -X PUT http://localhost:8081/config/users-value \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{"compatibility": "BACKWARD"}'

# Test compatibility before registering
curl -X POST http://localhost:8081/compatibility/subjects/users-value/versions/latest \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{"schema": "..."}'
```

---

## Exercise 6: Storage Capacity Planning (45 minutes)

**Objective:** Calculate storage requirements

### Scenario

**Given:**
- Topic: `transactions`
- Messages per day: 10 million
- Average message size: 5 KB
- Retention: 30 days
- Replication factor: 3
- Partitions: 12

**Calculate:**

1. **Daily data volume:**
   ```
   10M messages × 5 KB = ? GB/day
   ```

2. **Total retained data:**
   ```
   Daily volume × 30 days × RF = ? GB
   ```

3. **Storage per broker** (3 brokers):
   ```
   Total / 3 = ? GB per broker
   ```

4. **With 20% overhead:**
   ```
   Storage per broker × 1.2 = ? GB
   ```

5. **Peak write throughput:**
   ```
   Assuming 4-hour peak (6AM-10AM) with 50% of daily volume:
   (10M × 0.5) / (4 × 3600) = ? messages/sec
   ? messages/sec × 5 KB = ? MB/sec
   ```

### Implementation

**Create topic with calculated settings:**
```bash
kafka-topics.sh --create \
  --topic transactions \
  --partitions 12 \
  --replication-factor 3 \
  --config retention.bytes=? \
  --config segment.bytes=1073741824 \
  --bootstrap-server localhost:9092
```

---

## Exercise 7: Topic Administration (30 minutes)

**Objective:** Master topic management operations

### Part A: Alter Topic Configuration

```bash
# View current configuration
kafka-configs.sh --describe \
  --entity-type topics \
  --entity-name my-topic \
  --bootstrap-server localhost:9092

# Change retention
kafka-configs.sh --alter \
  --entity-type topics \
  --entity-name my-topic \
  --add-config retention.ms=604800000 \
  --bootstrap-server localhost:9092

# Change compression
kafka-configs.sh --alter \
  --entity-type topics \
  --entity-name my-topic \
  --add-config compression.type=lz4 \
  --bootstrap-server localhost:9092

# Remove configuration (use default)
kafka-configs.sh --alter \
  --entity-type topics \
  --entity-name my-topic \
  --delete-config retention.ms \
  --bootstrap-server localhost:9092
```

### Part B: Increase Partition Count

```bash
# Current partition count
kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092

# Increase partitions
kafka-topics.sh --alter \
  --topic my-topic \
  --partitions 12 \
  --bootstrap-server localhost:9092
```

**Warning:** Cannot decrease partition count!

### Part C: Delete Topic

```bash
# Delete topic
kafka-topics.sh --delete \
  --topic my-topic \
  --bootstrap-server localhost:9092

# Verify deletion
kafka-topics.sh --list --bootstrap-server localhost:9092
```

---

## Challenge Exercise: Multi-Tiered Storage Implementation (90 minutes)

**Objective:** Implement hot/cold data strategy

**Scenario:**
- Recent data (< 7 days): Fast SSD storage, high availability
- Old data (7-30 days): Slower HDD storage
- Archive (> 30 days): S3 for long-term retention

**Tasks:**

1. **Configure Tiered Storage Topics:**

```bash
# Hot tier (SSD)
kafka-topics.sh --create \
  --topic events-hot \
  --partitions 12 \
  --replication-factor 3 \
  --config retention.ms=604800000 \
  --config log.dirs=/data/ssd/kafka-logs \
  --bootstrap-server localhost:9092

# Cold tier (HDD)
kafka-topics.sh --create \
  --topic events-cold \
  --partitions 6 \
  --replication-factor 2 \
  --config retention.ms=2592000000 \
  --config log.dirs=/data/hdd/kafka-logs \
  --bootstrap-server localhost:9092
```

2. **Implement Data Mover:**

```java
public class TierMover {
    
    @Scheduled(fixedRate = 3600000)  // Hourly
    public void moveToColddTier() {
        // Read from hot topic where timestamp > 7 days
        ConsumerRecords<String, Event> oldRecords = 
            readOldRecordsFromHotTopic();
        
        // Write to cold topic
        for (ConsumerRecord<String, Event> record : oldRecords) {
            coldProducer.send(new ProducerRecord<>(
                "events-cold",
                record.key(),
                record.value()
            ));
        }
    }
}
```

3. **Implement S3 Archiver:**

Use Kafka Connect S3 Sink Connector:
```json
{
  "name": "s3-archive-connector",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "topics": "events-cold",
    "s3.bucket.name": "kafka-archive",
    "flush.size": "10000",
    "rotate.interval.ms": "3600000",
    "storage.class": "io.confluent.connect.s3.storage.S3Storage",
    "format.class": "io.confluent.connect.s3.format.parquet.ParquetFormat"
  }
}
```

4. **Create Query Service:**

```java
@RestController
public class EventQueryController {
    
    @GetMapping("/events/{userId}")
    public List<Event> getEvents(
        @PathVariable String userId,
        @RequestParam long startTime,
        @RequestParam long endTime
    ) {
        List<Event> events = new ArrayList<>();
        
        // Query hot tier
        if (endTime > System.currentTimeMillis() - 7 * 86400000) {
            events.addAll(queryHotTier(userId, startTime, endTime));
        }
        
        // Query cold tier
        if (startTime < System.currentTimeMillis() - 7 * 86400000) {
            events.addAll(queryColdTier(userId, startTime, endTime));
        }
        
        // Query S3 archive if needed
        if (startTime < System.currentTimeMillis() - 30 * 86400000) {
            events.addAll(queryS3Archive(userId, startTime, endTime));
        }
        
        return events;
    }
}
```

**Deliverables:**
- Complete implementation
- Performance comparison (query latency by tier)
- Cost analysis
- Monitoring dashboard

---

## Solutions & Discussion

### Exercise 2: Partition Count Calculation

**Part A:**
- Producer partitions needed: 100 MB/sec ÷ 10 MB/sec = 10
- Consumer partitions needed: 100 MB/sec ÷ 10 MB/sec = 10
- Recommended: 12 partitions (with headroom)

**Part B:**
- Current: 5,000 msg/sec × 2 KB = 10 MB/sec
- Future: 30 MB/sec
- Minimum: 3 partitions
- Recommended: 12 partitions (allows for 10 consumers + growth)

### Exercise 3: Retention Configurations

**Scenario 1:**
```properties
retention.ms=604800000  # 7 days
retention.bytes=41943040000  # 500GB / 12 partitions
```

**Scenario 2:**
```properties
retention.ms=220752000000  # 7 years
compression.type=gzip
segment.ms=86400000  # 1 day segments
```

**Scenario 3:**
```properties
cleanup.policy=compact
min.cleanable.dirty.ratio=0.5
segment.ms=3600000  # 1 hour
```

### Exercise 6: Storage Calculation

1. Daily: 10M × 5 KB = 50 GB/day
2. Total: 50 GB × 30 × 3 = 4,500 GB = 4.5 TB
3. Per broker: 1.5 TB
4. With overhead: 1.8 TB per broker

---

**Time to Complete:** 5-6 hours

**[← Previous: Module 5 Exercises](module-05-exercises.md)** | **[Next: Module 7 Exercises →](module-07-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
