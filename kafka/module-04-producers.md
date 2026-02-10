# Module 4: Kafka Producers

## Overview
This module focuses on Kafka producers - applications that publish events to Kafka topics. You'll learn how to build producers in multiple programming languages, implement custom partitioning logic, configure reliability settings, and optimize performance.

**Duration:** 3 hours

## Learning Objectives
By the end of this module, you will be able to:
- Build producers in Java, Python, and Node.js
- Implement custom serialization logic
- Choose appropriate partitioning strategies
- Configure producers for reliability and performance
- Handle errors and implement retry logic
- Monitor producer metrics

## Table of Contents
1. [Producer Architecture](#producer-architecture)
2. [Creating Producers](#creating-producers)
3. [Producer Configuration](#producer-configuration)
4. [Message Serialization](#message-serialization)
5. [Partitioning Strategies](#partitioning-strategies)
6. [Acknowledgments and Reliability](#acknowledgments-and-reliability)
7. [Idempotent Producers](#idempotent-producers)
8. [Error Handling and Retries](#error-handling-and-retries)
9. [Batching and Compression](#batching-and-compression)
10. [Producer Metrics](#producer-metrics)
11. [Best Practices](#best-practices)
12. [Summary](#summary)

---

## Producer Architecture

### How Producers Work

```
┌─────────────────────────────────────────────────────────┐
│                    Application                          │
│                                                         │
│  1. Create ProducerRecord(topic, key, value)           │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  2. Serializer (Convert key/value to bytes)            │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  3. Partitioner (Determine target partition)           │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  4. Record Accumulator (Batch records per partition)   │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  5. Sender (Send batches to brokers)                   │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│              Kafka Broker (Leader)                      │
│                                                         │
│  6. Write to log, replicate, acknowledge                │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  7. Callback (Success or error)                        │
└─────────────────────────────────────────────────────────┘
```

### Producer Components

1. **ProducerRecord**: Contains topic, partition (optional), key (optional), value
2. **Serializer**: Converts objects to byte arrays
3. **Partitioner**: Determines which partition to send to
4. **Record Accumulator**: Buffers records before sending
5. **Sender Thread**: Sends batches to brokers
6. **Callback**: Handles success or failure

---

## Creating Producers

### Java Producer

**Add dependency (Maven):**
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.6.1</version>
</dependency>
```

**Simple Producer:**
```typescript
import { Kafka, Producer, ProducerRecord, RecordMetadata } from 'kafkajs';

interface ProducerMessage {
  topic: string;
  key: string;
  value: string;
}

class SimpleProducer {
  private producer: Producer;

  constructor() {
    const kafka = new Kafka({
      clientId: 'my-producer',
      brokers: ['localhost:9092']
    });
    
    this.producer = kafka.producer();
  }

  async connect(): Promise<void> {
    await this.producer.connect();
  }

  async sendMessage(msg: ProducerMessage): Promise<RecordMetadata[]> {
    const result = await this.producer.send({
      topic: msg.topic,
      messages: [
        {
          key: msg.key,
          value: msg.value
        }
      ]
    });

    console.log(`Sent: topic=${result[0].topicName}, partition=${result[0].partition}, ` +
      `offset=${result[0].baseOffset}`);
    
    return result;
  }

  async disconnect(): Promise<void> {
    await this.producer.disconnect();
  }
}

// Usage
async function main() {
  const producer = new SimpleProducer();
  
  try {
    await producer.connect();
    
    await producer.sendMessage({
      topic: 'my-topic',
      key: 'key-1',
      value: 'Hello, Kafka!'
    });
  } catch (error) {
    console.error('Error:', error);
  } finally {
    await producer.disconnect();
  }
}

main();
```

**Synchronous Send:**
```typescript
try {
  const metadata = await producer.send({
    topic: 'my-topic',
    messages: [{ key: 'key-1', value: 'Hello!' }]
  });
  console.log('Sent to partition', metadata[0].partition);
} catch (error) {
  console.error(error);
}
```

### Python Producer

**Install library:**
```bash
npm install kafkajs
```

**Simple Producer:**
```typescript
import { Kafka } from 'kafkajs';

interface MessageValue {
  message: string;
  count: number;
}

class KafkaProducerService {
  private kafka: Kafka;
  private producer;

  constructor(brokers: string[]) {
    this.kafka = new Kafka({
      clientId: 'my-producer',
      brokers
    });
    this.producer = this.kafka.producer();
  }

  async connect(): Promise<void> {
    await this.producer.connect();
  }

  async sendMessage(
    topic: string,
    key: string,
    value: MessageValue
  ): Promise<void> {
    try {
      const result = await this.producer.send({
        topic,
        messages: [
          {
            key,
            value: JSON.stringify(value)
          }
        ]
      });

      console.log(
        `Sent: topic=${result[0].topicName}, ` +
        `partition=${result[0].partition}, ` +
        `offset=${result[0].baseOffset}`
      );
    } catch (error) {
      console.error('Error:', error);
      throw error;
    }
  }

  async disconnect(): Promise<void> {
    await this.producer.disconnect();
  }
}

// Usage
async function main() {
  const producer = new KafkaProducerService(['localhost:9092']);
  await producer.connect();
  
  await producer.sendMessage(
    'my-topic',
    'key-1',
    { message: 'Hello, Kafka!', count: 1 }
  );
  
  await producer.disconnect();
}

main();
```

**With Callback:**
```typescript
import { Kafka } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer();

await producer.connect();

try {
  const metadata = await producer.send({
    topic: 'my-topic',
    messages: [{ value: JSON.stringify({ data: 'test' }) }]
  });
  console.log(`Success: ${metadata[0].topicName}-${metadata[0].partition}@${metadata[0].baseOffset}`);
} catch (error) {
  console.error(`Error: ${error}`);
}

await producer.disconnect();
```

### Node.js Producer

**Install library:**
```bash
npm install kafkajs
```

**Simple Producer:**
```typescript
import { Kafka, Producer, RecordMetadata } from 'kafkajs';

interface MessageData {
  message: string;
  timestamp: number;
}

class NodeKafkaProducer {
  private kafka: Kafka;
  private producer: Producer;

  constructor(clientId: string, brokers: string[]) {
    this.kafka = new Kafka({ clientId, brokers });
    this.producer = this.kafka.producer();
  }

  async sendMessage(
    topic: string,
    key: string,
    value: MessageData
  ): Promise<RecordMetadata[]> {
    try {
      // Connect
      await this.producer.connect();
      
      // Send message
      const result = await this.producer.send({
        topic,
        messages: [
          {
            key,
            value: JSON.stringify(value)
          }
        ]
      });
      
      console.log('Sent:', result);
      return result;
      
    } catch (error) {
      console.error('Error:', error);
      throw error;
    } finally {
      await this.producer.disconnect();
    }
  }
}

// Usage
async function main() {
  const producer = new NodeKafkaProducer(
    'my-producer',
    ['localhost:9092']
  );
  
  await producer.sendMessage(
    'my-topic',
    'key-1',
    { message: 'Hello, Kafka!', timestamp: Date.now() }
  );
}

main();
```

**Batch Send:**
```typescript
import { Kafka } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer();

await producer.connect();

await producer.sendBatch({
  topicMessages: [
    {
      topic: 'topic-1',
      messages: [
        { key: 'key1', value: 'value1' },
        { key: 'key2', value: 'value2' }
      ]
    },
    {
      topic: 'topic-2',
      messages: [
        { key: 'key3', value: 'value3' }
      ]
    }
  ]
});

await producer.disconnect();
```

---

## Producer Configuration

### Essential Properties

| Property | Description | Default | Recommendation |
|----------|-------------|---------|----------------|
| `bootstrap.servers` | Broker addresses | None (required) | Multiple brokers |
| `key.serializer` | Key serializer class | None (required) | StringSerializer |
| `value.serializer` | Value serializer class | None (required) | StringSerializer |
| `acks` | Acknowledgment level | 1 | all (for reliability) |
| `retries` | Number of retries | 2147483647 | Keep default |
| `max.in.flight.requests.per.connection` | Max unacked requests | 5 | 5 (or 1 for ordering) |
| `enable.idempotence` | Idempotent producer | true | true |
| `compression.type` | Compression algorithm | none | lz4 or snappy |
| `batch.size` | Batch size in bytes | 16384 | 32768 for throughput |
| `linger.ms` | Wait time before send | 0 | 10-100 for throughput |
| `buffer.memory` | Total memory for buffering | 33554432 | Increase for high load |

### Configuration Examples

**High Throughput:**
```typescript
import { Kafka, CompressionTypes } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });

const producer = kafka.producer({
  allowAutoTopicCreation: false,
  transactionTimeout: 30000,
  // High throughput config
  compression: CompressionTypes.LZ4,
  // Batch settings
  maxInFlightRequests: 5,
  idempotent: false,
  // Adjust batch size and linger time
  retry: {
    maxRetryTime: 30000,
    initialRetryTime: 300,
    retries: 10
  }
});
```

**High Reliability:**
```typescript
import { Kafka, CompressionTypes } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });

const producer = kafka.producer({
  // Idempotent producer for exactly-once
  idempotent: true,
  // Maximum reliability
  maxInFlightRequests: 5,
  // Enable compression
  compression: CompressionTypes.Snappy,
  // Retry configuration
  retry: {
    maxRetryTime: 60000,
    initialRetryTime: 300,
    retries: Number.MAX_SAFE_INTEGER
  },
  // Require all ISR acknowledgment
  timeout: 30000
});
```

**Low Latency:**
```typescript
import { Kafka, CompressionTypes } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });

const producer = kafka.producer({
  // Minimal batching
  maxInFlightRequests: 1,
  // No compression
  compression: CompressionTypes.None,
  // Send immediately
  timeout: 5000,
  idempotent: false
});
```

---

## Message Serialization

### Built-in Serializers

- **StringSerializer**: Converts strings to UTF-8 bytes
- **ByteArraySerializer**: No conversion
- **IntegerSerializer**: Converts integers
- **LongSerializer**: Converts longs
- **DoubleSerializer**: Converts doubles

### JSON Serialization (TypeScript)

**Using Built-in JSON:**
```typescript
import { Kafka } from 'kafkajs';

interface OrderEvent {
  orderId: string;
  amount: number;
  userId: string;
}

class JsonProducer {
  private producer;

  constructor(kafka: Kafka) {
    this.producer = kafka.producer();
  }

  async sendOrder(topic: string, order: OrderEvent): Promise<void> {
    await this.producer.send({
      topic,
      messages: [
        {
          key: order.orderId,
          value: JSON.stringify(order)
        }
      ]
    });
  }
}

// Usage
const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = new JsonProducer(kafka);

await producer.sendOrder('orders', {
  orderId: 'order-123',
  amount: 99.99,
  userId: 'user-456'
});
```

### Avro Serialization

**Schema:**
```typescript
// Define Avro schema
const orderEventSchema = {
  type: 'record',
  name: 'OrderEvent',
  namespace: 'com.example',
  fields: [
    { name: 'orderId', type: 'string' },
    { name: 'amount', type: 'double' },
    { name: 'userId', type: 'string' }
  ]
};
```

**With Schema Registry:**
```typescript
import { Kafka } from 'kafkajs';
import { SchemaRegistry } from '@kafkajs/confluent-schema-registry';

const registry = new SchemaRegistry({ 
  host: 'http://localhost:8081' 
});

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer();

// Register schema
const { id } = await registry.register(orderEventSchema);

// Encode and send
const payload = {
  orderId: 'order-123',
  amount: 99.99,
  userId: 'user-456'
};

const encodedValue = await registry.encode(id, payload);

await producer.send({
  topic: 'orders',
  messages: [{ value: encodedValue }]
});
```

---

## Partitioning Strategies

### Default Partitioning

**With Key:**
```
partition = hash(key) % num_partitions
```

**Without Key (Round-Robin):**
```
partition = (current_partition + 1) % num_partitions
```

### Custom Partitioner (TypeScript)

```typescript
import { Kafka, Partitioners, Message } from 'kafkajs';

interface CustomPartitionerArgs {
  topic: string;
  partitionMetadata: Array<{ partitionId: number }>;
  message: Message;
}

// Custom partitioner function
const customPartitioner = (): ((args: CustomPartitionerArgs) => number) => {
  return ({ topic, partitionMetadata, message }: CustomPartitionerArgs) => {
    const numPartitions = partitionMetadata.length;
    const key = message.key?.toString() || '';
    
    // Example: Route premium users to partition 0
    if (key.startsWith('premium-')) {
      return 0;
    }
    
    // Others use hashing on remaining partitions
    const hash = hashCode(key);
    return (Math.abs(hash) % (numPartitions - 1)) + 1;
  };
};

// Simple hash function
function hashCode(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash |= 0; // Convert to 32bit integer
  }
  return hash;
}

// Usage
const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer({
  createPartitioner: customPartitioner
});

await producer.connect();

await producer.send({
  topic: 'orders',
  messages: [
    { key: 'premium-user-123', value: 'High priority order' },
    { key: 'regular-user-456', value: 'Normal order' }
  ]
});
```

**Configure:**
```java
props.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, CustomPartitioner.class);
```

### Sticky Partitioning

**Kafka 2.4+ default for null keys:**
- Sticks to one partition until batch is full
- Then switches to another partition
- Better than round-robin for throughput

---

## Acknowledgments and Reliability

### acks=0 (Fire and Forget)

```
Producer → Broker
         ← (No wait)
```

**Characteristics:**
- No acknowledgment waited
- Highest throughput
- **Can lose data**

**Use case:** Logging, metrics where some loss is acceptable

### acks=1 (Leader Acknowledgment)

```
Producer → Leader → Write to log
         ← Ack
```

**Characteristics:**
- Wait for leader to write
- Moderate throughput
- **Can lose data** if leader fails before replication

**Use case:** General purpose, balance of speed and reliability

### acks=all (All ISR Acknowledgment)

```
Producer → Leader → Write to log
                 → Follower 1 replicates
                 → Follower 2 replicates
         ← Ack (after all ISR acknowledge)
```

**Characteristics:**
- Wait for all in-sync replicas
- Lowest throughput
- **No data loss** (with `min.insync.replicas`)

**Use case:** Financial transactions, critical data

### Combining with min.insync.replicas

**Topic config:**
```bash
kafka-configs.sh --alter \
  --topic critical-topic \
  --add-config min.insync.replicas=2 \
  --bootstrap-server localhost:9092
```

**Producer config:**
```java
props.put(ProducerConfig.ACKS_CONFIG, "all");
```

**Result:**
- Replication Factor = 3
- min.insync.replicas = 2
- acks = all
- **Guarantees:** Data written to at least 2 replicas before ack

---

## Idempotent Producers

### The Problem: Duplicate Messages

```
Producer → Broker (Leader writes, then crashes before ack)
         ← (timeout, no ack received)
Producer → Broker (retry, writes again)
Result: Duplicate message! ❌
```

### Solution: Idempotent Producer

**Enable:**
```java
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
```

**How it works:**
- Producer assigns sequence number to each message
- Broker tracks sequence numbers per producer
- Duplicates detected and ignored

```
Producer (ID=123) → Broker [Seq 0] "message 1"
                  → Broker [Seq 1] "message 2"
                  ← timeout
                  → Broker [Seq 1] "message 2" (retry)
                  ← Broker: "Duplicate, ignored"
```

**Implications:**
- Automatic deduplication
- Ordering guaranteed per partition
- No application changes needed
- Small performance overhead

**Requirements:**
- `enable.idempotence=true`
- `acks=all`
- `retries > 0`
- `max.in.flight.requests.per.connection ≤ 5`

---

## Error Handling and Retries

### Types of Errors

#### 1. **Retriable Errors**
- Network errors
- Leader not available
- Not enough replicas

**Kafka retries automatically**

#### 2. **Non-Retriable Errors**
- Invalid topic
- Message too large
- Serialization errors

**Application must handle**

### Retry Configuration

```java
// Retry configuration
props.put(ProducerConfig.RETRIES_CONFIG, Integer.MAX_VALUE);
props.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 100);
props.put(ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG, 30000);
props.put(ProducerConfig.DELIVERY_TIMEOUT_MS_CONFIG, 120000);
```

### Error Handling Patterns

**Pattern 1: Callback**
```java
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        if (exception instanceof RetriableException) {
            // Kafka will retry automatically
            log.warn("Retriable error: {}", exception.getMessage());
        } else {
            // Non-retriable, handle manually
            log.error("Fatal error: {}", exception.getMessage());
            // Send to DLQ, alert, etc.
        }
    }
});
```

**Pattern 2: Try-Catch with get()**
```java
try {
    RecordMetadata metadata = producer.send(record).get();
    log.info("Sent successfully to {}-{}", metadata.topic(), metadata.partition());
} catch (ExecutionException e) {
    Throwable cause = e.getCause();
    if (cause instanceof RetriableException) {
        // Already retried, still failed
        handleRetryExhausted(record);
    } else {
        // Non-retriable
        handleFatalError(record, cause);
    }
}
```

**Pattern 3: Dead Letter Queue**
```java
producer.send(record, (metadata, exception) -> {
    if (exception != null && !(exception instanceof RetriableException)) {
        // Send to DLQ
        ProducerRecord<String, String> dlqRecord = new ProducerRecord<>(
            "dlq-topic",
            record.key(),
            record.value()
        );
        producer.send(dlqRecord);
    }
});
```

---

## Batching and Compression

### Batching

**Configuration:**
```java
// Batch size in bytes (default: 16KB)
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 32768); // 32KB

// Wait time before sending batch (default: 0ms)
props.put(ProducerConfig.LINGER_MS_CONFIG, 10); // 10ms

// Total memory for buffering (default: 32MB)
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 67108864); // 64MB
```

**How it works:**
```
Messages arrive → Buffer per partition → Wait for:
                                          - Batch full (batch.size)
                                          - OR timeout (linger.ms)
                                       → Send batch
```

**Trade-offs:**
- **Larger batch.size**: More throughput, higher latency
- **Higher linger.ms**: More batching, higher latency
- **Lower linger.ms**: Less batching, lower latency

### Compression

**Algorithms:**
- **none**: No compression (default)
- **gzip**: Best compression ratio, CPU intensive
- **snappy**: Good compression, fast
- **lz4**: Fastest, good compression
- **zstd**: Balance of speed and compression

**Configuration:**
```java
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4");
```

**Compression happens:**
- At producer (before sending)
- Per batch (not per message)
- Decompression at consumer

**Benefits:**
- Reduced network bandwidth
- Reduced broker storage
- Higher throughput

**Example sizes:**
```
Original:   100KB
gzip:       20KB  (5x compression, slow)
snappy:     35KB  (2.8x compression, fast)
lz4:        30KB  (3.3x compression, fastest)
zstd:       25KB  (4x compression, balanced)
```

---

## Producer Metrics

### Key Metrics to Monitor

**Throughput:**
- `record-send-rate`: Records sent per second
- `byte-rate`: Bytes sent per second
- `request-rate`: Requests per second

**Latency:**
- `record-queue-time-avg`: Time in send buffer
- `request-latency-avg`: Round-trip time to broker

**Errors:**
- `record-error-rate`: Failed sends per second
- `record-retry-rate`: Retries per second

**Batching:**
- `batch-size-avg`: Average batch size
- `records-per-request-avg`: Records per batch

### Accessing Metrics (Java)

```java
Map<MetricName, ? extends Metric> metrics = producer.metrics();

metrics.forEach((name, metric) -> {
    if (name.name().equals("record-send-rate")) {
        System.out.println("Send rate: " + metric.metricValue());
    }
});
```

### Monitoring with JMX

**Enable JMX:**
```bash
export KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote \
  -Dcom.sun.management.jmxremote.port=9999 \
  -Dcom.sun.management.jmxremote.authenticate=false \
  -Dcom.sun.management.jmxremote.ssl=false"
```

**MBeans:**
- `kafka.producer:type=producer-metrics,client-id=*`
- `kafka.producer:type=producer-topic-metrics,client-id=*,topic=*`

---

## Best Practices

### 1. Use Idempotent Producers
```java
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
props.put(ProducerConfig.ACKS_CONFIG, "all");
```

### 2. Choose Appropriate acks
- **acks=all** for critical data
- **acks=1** for general use
- **acks=0** only for non-critical data

### 3. Use Compression
```java
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4");
```

### 4. Tune Batching for Your Use Case
```java
// High throughput
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 32768);
props.put(ProducerConfig.LINGER_MS_CONFIG, 10);

// Low latency
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 0);
props.put(ProducerConfig.LINGER_MS_CONFIG, 0);
```

### 5. Handle Errors Properly
```java
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        // Log, alert, send to DLQ
    }
});
```

### 6. Close Producers Gracefully
```java
// Flush all buffered records
producer.flush();

// Close with timeout
producer.close(Duration.ofSeconds(30));
```

### 7. Use Keys for Ordering
```java
// Messages with same key go to same partition (ordered)
ProducerRecord<String, String> record = new ProducerRecord<>(
    "topic", "user-123", "message"
);
```

### 8. Monitor Metrics
```java
// Periodically check metrics
metrics.forEach((name, metric) -> {
    if (name.name().equals("record-error-rate")) {
        double errorRate = (double) metric.metricValue();
        if (errorRate > 0.01) {
            // Alert!
        }
    }
});
```

### 9. Use Thread-Safe Producers
```java
// One producer instance shared across threads
private static final KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// Safe to use from multiple threads
executor.submit(() -> producer.send(record1));
executor.submit(() -> producer.send(record2));
```

### 10. Avoid Blocking Operations
```java
// ❌ Don't do this
producer.send(record).get(); // Blocks!

// ✅ Do this
producer.send(record, callback); // Async
```

---

## Summary

In this module, you learned:

1. **Producer Architecture**: How messages flow from application to Kafka brokers
2. **Creating Producers**: Building producers in Java, Python, and Node.js
3. **Configuration**: Essential properties for different use cases
4. **Serialization**: Converting objects to bytes with various formats
5. **Partitioning**: Default and custom partitioning strategies
6. **Reliability**: acks levels and their guarantees
7. **Idempotence**: Preventing duplicate messages
8. **Error Handling**: Retry strategies and error patterns
9. **Optimization**: Batching and compression for performance
10. **Monitoring**: Key metrics to track producer health

---

## Key Takeaways

✅ **Producers are thread-safe** - One instance can be shared

✅ **Idempotence prevents duplicates** - Enable it for critical data

✅ **acks=all provides durability** - Combine with min.insync.replicas

✅ **Batching improves throughput** - Tune batch.size and linger.ms

✅ **Compression reduces bandwidth** - Use lz4 or snappy

✅ **Keys ensure ordering** - Messages with same key are ordered

✅ **Handle errors properly** - Not all errors are retriable

✅ **Monitor metrics** - Track throughput, latency, errors

---

## What's Next?

Now that you can produce messages, you need to consume them!

The next module will cover:
- Building consumers in multiple languages
- Consumer groups and partition assignment
- Offset management strategies
- Handling rebalancing
- Consumer performance tuning

**Continue to [Module 5: Kafka Consumers](module-05-consumers.md)**

---

## Additional Resources

- [Kafka Producer API Documentation](https://kafka.apache.org/documentation/#producerapi)
- [Producer Configs Reference](https://kafka.apache.org/documentation/#producerconfigs)
- [Idempotent Producer Documentation](https://kafka.apache.org/documentation/#producerconfigs_enable.idempotence)
- [KafkaJS Producer Documentation](https://kafka.js.org/docs/producing)
- [kafka-python Producer Documentation](https://kafka-python.readthedocs.io/en/master/apidoc/KafkaProducer.html)

---

**[📝 Practice Exercises](exercise/module-04-exercises.md)** | **[📚 Back to Course Home](README.md)**
