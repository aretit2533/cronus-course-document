# Module 7 Exercises: Kafka Connect and Kafka Streams

## Exercise 1: JDBC Source Connector Setup (45 minutes)

**Objective:** Stream data from PostgreSQL to Kafka

### Part A: Setup Database

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username, email) VALUES
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com'),
    ('charlie', 'charlie@example.com');
```

### Part B: Configure JDBC Source Connector

**jdbc-source-connector.json:**
```json
{
  "name": "postgres-source",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:postgresql://localhost:5432/mydb",
    "connection.user": "postgres",
    "connection.password": "postgres",
    "table.whitelist": "users",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "postgres-",
    "poll.interval.ms": "1000"
  }
}
```

**Deploy connector:**
```bash
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d @jdbc-source-connector.json
```

### Part C: Verify Data Flow

```bash
# Check connector status
curl http://localhost:8083/connectors/postgres-source/status | jq

# Consume from topic
kafka-console-consumer.sh \
  --topic postgres-users \
  --from-beginning \
  --bootstrap-server localhost:9092
```

### Part D: Test Updates

```sql
-- Insert new user
INSERT INTO users (username, email) VALUES ('david', 'david@example.com');

-- Update existing user
UPDATE users SET email = 'alice.updated@example.com' WHERE username = 'alice';
```

**Questions:**
1. Does the connector detect inserts? Updates?
2. What mode would you use to detect updates? (Hint: `timestamp+incrementing`)
3. How does the connector handle deletes?

---

## Exercise 2: Elasticsearch Sink Connector (45 minutes)

**Objective:** Index Kafka events into Elasticsearch

### Part A: Setup

Start Elasticsearch:
```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  docker.elastic.co/elasticsearch/elasticsearch:8.11.0
```

### Part B: Configure Sink Connector

**elasticsearch-sink-connector.json:**
```json
{
  "name": "elasticsearch-sink",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "1",
    "topics": "user-events",
    "connection.url": "http://localhost:9200",
    "type.name": "_doc",
    "key.ignore": "false",
    "schema.ignore": "true",
    "behavior.on.null.values": "delete"
  }
}
```

### Part C: Produce Events

```bash
kafka-console-producer.sh \
  --topic user-events \
  --bootstrap-server localhost:9092

# Enter JSON events:
{"userId":"user1","action":"login","timestamp":1234567890}
{"userId":"user2","action":"purchase","amount":99.99,"timestamp":1234567891}
```

### Part D: Query Elasticsearch

```bash
# Check index
curl http://localhost:9200/_cat/indices

# Search documents
curl http://localhost:9200/user-events/_search?pretty

# Count documents
curl http://localhost:9200/user-events/_count
```

**Tasks:**
1. Index 100 events into Elasticsearch
2. Create a search query for events from specific user
3. Aggregate events by action type
4. Setup index template for proper mapping

---

## Exercise 3: S3 Sink Connector with Partitioning (60 minutes)

**Objective:** Archive Kafka data to S3 with time-based partitioning

### Part A: Setup MinIO (S3-compatible)

```yaml
# Add to docker-compose.yml
minio:
  image: minio/minio:latest
  ports:
    - "9000:9000"
    - "9001:9001"
  environment:
    MINIO_ROOT_USER: minioadmin
    MINIO_ROOT_PASSWORD: minioadmin
  command: server /data --console-address ":9001"
```

### Part B: Configure S3 Sink

**s3-sink-connector.json:**
```json
{
  "name": "s3-sink",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "tasks.max": "1",
    "topics": "events",
    "s3.bucket.name": "kafka-archive",
    "s3.region": "us-east-1",
    "store.url": "http://localhost:9000",
    "aws.access.key.id": "minioadmin",
    "aws.secret.access.key": "minioadmin",
    "flush.size": "100",
    "rotate.interval.ms": "60000",
    "storage.class": "io.confluent.connect.s3.storage.S3Storage",
    "format.class": "io.confluent.connect.s3.format.json.JsonFormat",
    "partitioner.class": "io.confluent.connect.storage.partitioner.TimeBasedPartitioner",
    "partition.duration.ms": "3600000",
    "path.format": "'year'=YYYY/'month'=MM/'day'=dd/'hour'=HH",
    "timestamp.extractor": "Record",
    "locale": "en-US",
    "timezone": "UTC"
  }
}
```

### Part C: Verify S3 Files

```bash
# Install MinIO client
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc

# Configure
./mc alias set myminio http://localhost:9000 minioadmin minioadmin

# List files
./mc ls myminio/kafka-archive/events/

# Download and view file
./mc cp myminio/kafka-archive/events/year=2024/month=01/day=15/hour=10/events+0+0000000000.json .
cat events+0+0000000000.json
```

**Tasks:**
1. Produce 1000 events over 2 hours
2. Verify files are partitioned by hour
3. Change partitioner to daily partitioning
4. Implement file retention (delete files older than 30 days)

---

## Exercise 4: Custom SMT (Single Message Transform) (60 minutes)

**Objective:** Create a custom transformation

**Scenario:** Mask sensitive email addresses in events

### Implement Custom SMT

**Note:** Kafka Connect SMTs are Java-only. For TypeScript/Node.js, implement transformations in a custom producer/consumer.

**EmailMaskingTransform.ts:**
```typescript
import { Kafka, Producer, Consumer, EachMessagePayload } from 'kafkajs';

interface EventData {
  [key: string]: any;
  email?: string;
}

class EmailMaskingTransform {
  private kafka: Kafka;
  private consumer: Consumer;
  private producer: Producer;
  
  constructor() {
    this.kafka = new Kafka({ brokers: ['localhost:9092'] });
    this.consumer = this.kafka.consumer({ groupId: 'email-masker' });
    this.producer = this.kafka.producer();
  }
  
  async start(): Promise<void> {
    await this.consumer.connect();
    await this.producer.connect();
    await this.consumer.subscribe({ topic: 'source-topic' });
    
    await this.consumer.run({
      eachMessage: async ({ message }: EachMessagePayload) => {
        const masked = this.apply(message);
        
        await this.producer.send({
          topic: 'masked-topic',
          messages: [masked]
        });
      }
    });
  }
  
  apply(message: any): any {
    if (message.value) {
      const data: EventData = JSON.parse(message.value.toString());
      const maskedData = this.maskEmail(data);
      
      return {
        key: message.key,
        value: JSON.stringify(maskedData),
        headers: message.headers
      };
    }
    return message;
  }
  
  private maskEmail(data: EventData): EventData {
    if (data.email && typeof data.email === 'string') {
      data.email = this.maskEmailAddress(data.email);
    }
    return data;
  }
  
  private maskEmailAddress(email: string): string {
    const atIndex = email.indexOf('@');
    if (atIndex > 0) {
      const prefix = email.substring(0, atIndex);
      const domain = email.substring(atIndex);
      return prefix.charAt(0) + '***' + domain;
    }
    return email;
  }
}

// Usage
const transform = new EmailMaskingTransform();
transform.start().catch(console.error);
```
    public void close() {}
    
    @Override
    public void configure(Map<String, ?> configs) {}
}
```

### Use in Connector

```json
{
  "name": "postgres-source-masked",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    ...
    "transforms": "maskEmail",
    "transforms.maskEmail.type": "com.example.transforms.EmailMaskingTransform"
  }
}
```

**Tasks:**
1. Package and deploy the custom SMT
2. Test with sample data
3. Verify emails are masked in output topic

---

## Exercise 5: Kafka Streams - Word Count (45 minutes)

**Objective:** Implement the classic word count example

### Implementation

**Note:** Kafka Streams is a Java library. For TypeScript, implement stream processing with KafkaJS consumers/producers.

**WordCountApp.ts:**
```typescript
import { Kafka, Consumer, Producer } from 'kafkajs';

class WordCountApp {
  private kafka: Kafka;
  private consumer: Consumer;
  private producer: Producer;
  private wordCounts: Map<string, number> = new Map();
  
  constructor() {
    this.kafka = new Kafka({
      clientId: 'word-count',
      brokers: ['localhost:9092']
    });
    
    this.consumer = this.kafka.consumer({
      groupId: 'word-count-group'
    });
    
    this.producer = this.kafka.producer();
  }
  
  async start(): Promise<void> {
    await this.consumer.connect();
    await this.producer.connect();
    
    // Read from input topic
    await this.consumer.subscribe({ topic: 'text-input' });
    
    await this.consumer.run({
      eachMessage: async ({ message }) => {
        const textLine = message.value?.toString() || '';
        
        // Process: split into words and count
        const words = textLine.toLowerCase().split(/\W+/).filter(w => w.length > 0);
        
        for (const word of words) {
          const currentCount = this.wordCounts.get(word) || 0;
          this.wordCounts.set(word, currentCount + 1);
        }
        
        // Write to output topic
        const messages = Array.from(this.wordCounts.entries()).map(([word, count]) => ({
          key: word,
          value: count.toString()
        }));
        
        await this.producer.send({
          topic: 'word-count-output',
          messages
        });
      }
    });
    
    // Graceful shutdown
    process.on('SIGTERM', async () => {
      await this.consumer.disconnect();
      await this.producer.disconnect();
    });
  }
}

// Usage
const app = new WordCountApp();
app.start().catch(console.error);
```

### Part A: Test

```bash
# Create topics
kafka-topics.sh --create --topic text-input --partitions 3 --bootstrap-server localhost:9092
kafka-topics.sh --create --topic word-count-output --partitions 3 --bootstrap-server localhost:9092

# Produce text
kafka-console-producer.sh --topic text-input --bootstrap-server localhost:9092
# Enter:
hello world
hello kafka
kafka streams

# Consume results
kafka-console-consumer.sh --topic word-count-output \
  --from-beginning \
  --bootstrap-server localhost:9092 \
  --property print.key=true \
  --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

### Part B: Extend

Add these features:
1. Filter out stop words ("the", "a", "an", etc.)
2. Count only words longer than 3 characters
3. Track word counts per 5-minute window

---

## Exercise 6: Real-Time Analytics with Kafka Streams (90 minutes)

**Objective:** Build a real-time analytics pipeline

**Scenario:** E-commerce clickstream analysis

### Requirements
- Count page views per product (5-minute windows)
- Calculate conversion rate (views → purchases)
- Detect trending products (rapid view increase)
- Alerting for low conversion rates

### Implementation

**Note:** Kafka Streams windowing is Java-specific. Here's a TypeScript equivalent using time-based aggregation.

**ClickstreamAnalytics.ts:**
```typescript
import { Kafka, Consumer, Producer } from 'kafkajs';

interface Event {
  eventType: string;
  productId: string;
  userId: string;
  timestamp: number;
}

interface ConversionMetrics {
  views: number;
  purchases: number;
  conversionRate: number;
}

interface Alert {
  type: string;
  productId: string;
  conversionRate: number;
}

class ClickstreamAnalytics {
  private kafka: Kafka;
  private consumer: Consumer;
  private alertProducer: Producer;
  private windowSize = 5 * 60 * 1000; // 5 minutes
  private windows: Map<string, Map<string, { views: number; purchases: number }>> = new Map();
  
  constructor() {
    this.kafka = new Kafka({ brokers: ['localhost:9092'] });
    this.consumer = this.kafka.consumer({ groupId: 'clickstream-analytics' });
    this.alertProducer = this.kafka.producer();
  }
  
  async start(): Promise<void> {
    await this.consumer.connect();
    await this.alertProducer.connect();
    await this.consumer.subscribe({ topic: 'user-events' });
    
    await this.consumer.run({
      eachMessage: async ({ message }) => {
        const event: Event = JSON.parse(message.value?.toString() || '{}');
        
        // Determine window
        const windowKey = this.getWindowKey(event.timestamp);
        
        // Initialize window if needed
        if (!this.windows.has(windowKey)) {
          this.windows.set(windowKey, new Map());
        }
        const window = this.windows.get(windowKey)!;
        
        // Initialize product metrics if needed
        if (!window.has(event.productId)) {
          window.set(event.productId, { views: 0, purchases: 0 });
        }
        const metrics = window.get(event.productId)!;
        
        // Update counts
        if (event.eventType === 'PAGE_VIEW') {
          metrics.views++;
        } else if (event.eventType === 'PURCHASE') {
          metrics.purchases++;
        }
        
        // Calculate conversion rate
        const conversionRate = metrics.views > 0 
          ? metrics.purchases / metrics.views 
          : 0;
        
        // Alert on low conversion rate
        if (metrics.views > 100 && conversionRate < 0.01) {
          await this.sendAlert({
            type: 'LOW_CONVERSION',
            productId: event.productId,
            conversionRate
          });
        }
      }
    });
    
    // Clean up old windows periodically
    setInterval(() => this.cleanupOldWindows(), 60000);
  }
  
  private getWindowKey(timestamp: number): string {
    const windowStart = Math.floor(timestamp / this.windowSize) * this.windowSize;
    return windowStart.toString();
  }
  
  private cleanupOldWindows(): void {
    const now = Date.now();
    const cutoff = now - (this.windowSize * 2); // Keep 2 windows
    
    for (const [windowKey] of this.windows) {
      if (parseInt(windowKey) < cutoff) {
        this.windows.delete(windowKey);
      }
    }
  }
  
  private async sendAlert(alert: Alert): Promise<void> {
    await this.alertProducer.send({
      topic: 'conversion-alerts',
      messages: [{
        key: alert.productId,
        value: JSON.stringify(alert)
      }]
    });
  }
}

// Usage
const analytics = new ClickstreamAnalytics();
analytics.start().catch(console.error);
```

**Tasks:**
1. Complete the implementation
2. Add trend detection (compare current window to previous)
3. Create interactive queries endpoint to query current metrics
4. Write unit tests with `TopologyTestDriver`

---

## Exercise 7: State Store and Interactive Queries (60 minutes)

**Objective:** Query stream processing state in real-time

### Part A: Create State Store

**Note:** Kafka Streams state stores are Java-specific. Use in-memory storage or Redis for TypeScript.

**StatefulProcessor.ts:**
```typescript
import { Kafka, Consumer } from 'kafkajs';
import { createClient } from 'redis';

class StatefulProcessor {
  private kafka: Kafka;
  private consumer: Consumer;
  private stateStore: Map<string, number>; // In-memory state store
  // Or use Redis for persistent state:
  // private redis: ReturnType<typeof createClient>;
  
  constructor() {
    this.kafka = new Kafka({ brokers: ['localhost:9092'] });
    this.consumer = this.kafka.consumer({ groupId: 'stateful-processor' });
    this.stateStore = new Map();
  }
  
  async start(): Promise<void> {
    await this.consumer.connect();
    await this.consumer.subscribe({ topic: 'product-views' });
    
    await this.consumer.run({
      eachMessage: async ({ message }) => {
        const productId = message.key?.toString();
        if (!productId) return;
        
        // Update state store
        const currentCount = this.stateStore.get(productId) || 0;
        this.stateStore.set(productId, currentCount + 1);
        
        console.log(`Product ${productId}: ${currentCount + 1} views`);
      }
    });
  }
  
  // Query current state (for interactive queries)
  getProductViews(productId: string): number {
    return this.stateStore.get(productId) || 0;
  }
  
  getAllProductViews(): Map<string, number> {
    return new Map(this.stateStore);
  }
}

// Usage with REST API for interactive queries
import express from 'express';

const processor = new StatefulProcessor();
processor.start().catch(console.error);

const app = express();

// Interactive query endpoint
app.get('/state/product/:id', (req, res) => {
  const views = processor.getProductViews(req.params.id);
  res.json({ productId: req.params.id, views });
});

app.get('/state/all', (req, res) => {
  const allViews = Object.fromEntries(processor.getAllProductViews());
  res.json(allViews);
});

app.listen(3000);
```
            );
        builder.addStateStore(storeBuilder);
        
        // Process stream
        KStream<String, Event> events = builder.stream("events");
        
        events.process(() -> new Processor<String, Event, Void, Void>() {
            private KeyValueStore<String, Long> stateStore;
            
            @Override
            public void init(ProcessorContext<Void, Void> context) {
                stateStore = context.getStateStore("product-views");
            }
            
            @Override
            public void process(Record<String, Event> record) {
                String productId = record.value().getProductId();
                Long currentCount = stateStore.get(productId);
                long newCount = (currentCount != null ? currentCount : 0) + 1;
                stateStore.put(productId, newCount);
            }
        }, "product-views");
        
        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();
    }
}
```

### Part B: Interactive Query Service

```java
@RestController
public class StreamsQueryController {
    
    @Autowired
    private KafkaStreams streams;
    
    @GetMapping("/product-views/{productId}")
    public ResponseEntity<Long> getProductViews(@PathVariable String productId) {
        ReadOnlyKeyValueStore<String, Long> store = 
            streams.store(StoreQueryParameters.fromNameAndType(
                "product-views",
                QueryableStoreTypes.keyValueStore()
            ));
        
        Long count = store.get(productId);
        return ResponseEntity.ok(count != null ? count : 0L);
    }
    
    @GetMapping("/product-views/all")
    public ResponseEntity<Map<String, Long>> getAllViews() {
        ReadOnlyKeyValueStore<String, Long> store = 
            streams.store(StoreQueryParameters.fromNameAndType(
                "product-views",
                QueryableStoreTypes.keyValueStore()
            ));
        
        Map<String, Long> result = new HashMap<>();
        KeyValueIterator<String, Long> iterator = store.all();
        
        while (iterator.hasNext()) {
            KeyValue<String, Long> entry = iterator.next();
            result.put(entry.key, entry.value);
        }
        iterator.close();
        
        return ResponseEntity.ok(result);
    }
}
```

**Tasks:**
1. Implement the stateful processor
2. Create REST API for querying
3. Test with multiple application instances (distributed state)
4. Implement state restore after crash

---

## Challenge Exercise: Complex Event Processing Pipeline (120 minutes)

**Objective:** Build end-to-end CEP system

**Scenario:** Fraud detection for financial transactions

### Requirements

1. **Data Ingestion**: JDBC source from transactions table
2. **Enrichment**: Join with customer profile data
3. **Stream Processing**:
   - Detect transactions > $10,000
   - Detect 3+ transactions within 5 minutes from same customer
   - Calculate rolling average transaction amount per customer
4. **Alerting**: Kafka Streams alerts to Elasticsearch
5. **Archiving**: S3 sink for all transactions

### Architecture

```
PostgreSQL → JDBC Source → Kafka (transactions)
                              ↓
                         Kafka Streams
                         (fraud detection)
                              ↓
                         Kafka (fraud-alerts)
                              ↓
                     Elasticsearch Sink
                              ↓
                         S3 Sink (archive)
```

**Deliverables:**
1. All connector configurations
2. Kafka Streams application with fraud detection logic
3. Integration tests
4. Kibana dashboard showing fraud alerts
5. Performance benchmarks (throughput, latency)

---

## Solutions & Discussion

### Exercise 5: Word Count with Enhancements

```typescript
const STOP_WORDS = new Set(['the', 'a', 'an', 'and', 'or', 'but']);

class EnhancedWordCount {
  private wordCounts: Map<string, number> = new Map();
  private windowSize = 5 * 60 * 1000; // 5 minutes
  private windows: Map<string, Map<string, number>> = new Map();
  
  async processMessage(message: any): Promise<void> {
    const textLine = message.value?.toString() || '';
    const timestamp = parseInt(message.timestamp);
    const windowKey = this.getWindowKey(timestamp);
    
    // Get or create window
    if (!this.windows.has(windowKey)) {
      this.windows.set(windowKey, new Map());
    }
    const windowCounts = this.windows.get(windowKey)!;
    
    // Process words
    const words = textLine.toLowerCase()
      .split(/\W+/)
      .filter(word => word.length > 3)  // Length filter
      .filter(word => !STOP_WORDS.has(word));  // Stop words filter
    
    for (const word of words) {
      const count = windowCounts.get(word) || 0;
      windowCounts.set(word, count + 1);
    }
  }
  
  private getWindowKey(timestamp: number): string {
    const windowStart = Math.floor(timestamp / this.windowSize) * this.windowSize;
    return windowStart.toString();
  }
}
```

### Exercise 6: Trend Detection

```typescript
interface TrendingProduct {
  productId: string;
  currentViews: number;
  previousViews: number;
  growthFactor: number;
}

class TrendDetector {
  private currentWindow: Map<string, number> = new Map();
  private previousWindow: Map<string, number> = new Map();
  
  async detectTrends(): Promise<TrendingProduct[]> {
    const trending: TrendingProduct[] = [];
    
    // Compare current window to previous window
    for (const [productId, currentCount] of this.currentWindow) {
      const previousCount = this.previousWindow.get(productId) || 0;
      
      if (currentCount > previousCount * 2 && previousCount > 0) {
        trending.push({
          productId,
          currentViews: currentCount,
          previousViews: previousCount,
          growthFactor: currentCount / previousCount
        });
      }
    }
    
    return trending;
  }
  
  rotateWindow(): void {
    // Move current to previous, start new current
    this.previousWindow = new Map(this.currentWindow);
    this.currentWindow = new Map();
  }
}
```

---

**Time to Complete:** 6-7 hours

**[← Previous: Module 6 Exercises](module-06-exercises.md)** | **[Next: Module 8 Exercises →](module-08-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
