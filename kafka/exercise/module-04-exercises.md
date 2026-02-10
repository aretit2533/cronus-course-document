# Module 4 Exercises: Kafka Producers

## Exercise 1: Basic Producer Implementation (30 minutes)

**Objective:** Implement a basic Kafka producer using TypeScript

### TypeScript Implementation

**Tasks:**
1. Create a new Node.js/TypeScript project
2. Install KafkaJS dependencies: `npm install kafkajs`
3. Implement a simple producer

**basic-producer.ts:**
```typescript
import { Kafka, Producer, ProducerRecord } from 'kafkajs';

async function main() {
  // TODO: Configure Kafka client
  const kafka = new Kafka({
    clientId: 'basic-producer',
    brokers: ['localhost:9092']
  });
  
  // TODO: Create producer
  const producer: Producer = kafka.producer();
  
  // TODO: Connect to Kafka
  await producer.connect();
  
  // TODO: Send 100 messages
  
  // TODO: Disconnect producer
  await producer.disconnect();
}

main().catch(console.error);
```

**Requirements:**
- Send 100 messages to topic `test-events`
- Use user ID as key (e.g., "user-1", "user-2", ...)
- Message format: `{"userId":"user-X","action":"click","timestamp":123}`
- Print confirmation for each message sent

**Verification:**
```bash
# Consume messages
kafka-console-consumer.sh --topic test-events \
  --from-beginning \
  --bootstrap-server localhost:9092 \
  --property print.key=true
```

---

## Exercise 2: Fire-and-Forget vs Synchronous vs Asynchronous Send (45 minutes)

**Objective:** Compare different send methods and understand trade-offs

### Part A: Implement Three Sending Patterns

**1. Fire-and-Forget:**
```typescript
// Not waiting for result (fire-and-forget)
producer.send({
  topic: 'test-topic',
  messages: [{ key: 'key1', value: 'value1' }]
}).catch(console.error); // Handle errors but don't wait
```

**2. Synchronous (using await):**
```typescript
const metadata = await producer.send({
  topic: 'test-topic',
  messages: [{ key: 'key1', value: 'value1' }]
});
console.log(`Sent to partition ${metadata[0].partition}, offset ${metadata[0].baseOffset}`);
```

**3. Asynchronous with Promise handling:**
```typescript
producer.send({
  topic: 'test-topic',
  messages: [{ key: 'key1', value: 'value1' }]
})
.then(metadata => {
  console.log(`Sent to partition ${metadata[0].partition}`);
})
.catch(error => {
  console.error('Error:', error);
});
```

### Part B: Performance Comparison

**Tasks:**
1. Send 10,000 messages using each method
2. Measure time taken
3. Fill in results:

```
Method           | Time (ms) | Messages/sec | Notes
-----------------|-----------|--------------|-------
Fire-and-forget  | ?         | ?            | ?
Synchronous      | ?         | ?            | ?
Asynchronous     | ?         | ?            | ?
```

**Questions:**
1. Which method is fastest? Why?
2. Which method is most reliable? Why?
3. When would you use each method?
4. What happens if Kafka is unavailable with fire-and-forget?

---

## Exercise 3: Custom Serializer (40 minutes)

**Objective:** Create a custom serializer for complex objects

**Scenario:**
You need to send `Event` objects to Kafka:

```typescript
interface Event {
  eventId: string;
  userId: string;
  eventType: string;
  timestamp: number;
  metadata: Record<string, string>;
}
```

**Tasks:**

1. **Implement Custom Serializer:**

```typescript
import { Kafka, Producer } from 'kafkajs';

class EventProducer {
  private producer: Producer;

  constructor(kafka: Kafka) {
    this.producer = kafka.producer();
  }

  async sendEvent(topic: string, event: Event): Promise<void> {
    try {
      // Serialize Event object to JSON
      const serialized = JSON.stringify(event);
      
      await this.producer.send({
        topic,
        messages: [{
          key: event.eventId,
          value: serialized
        }]
      });
    } catch (error) {
      console.error('Serialization error:', error);
      throw error;
    }
  }
}
```

2. **Use Custom Serializer:**
```typescript
const kafka = new Kafka({ brokers: ['localhost:9092'] });
const eventProducer = new EventProducer(kafka);
await eventProducer.sendEvent('events-topic', myEvent);
```

3. **Test:**
   - Send 50 Event objects
   - Verify with console consumer
   - Handle serialization errors gracefully

**Bonus Challenge:**
Implement a custom deserializer and consume the events properly.

---

## Exercise 4: Custom Partitioner (45 minutes)

**Objective:** Implement a custom partitioning strategy

**Scenario:**
You have a multi-tenant application where events from VIP customers should go to specific partitions for priority processing.

**Requirements:**
- VIP customers (user IDs ending in "_vip") → Partition 0
- Regular customers → Round-robin across partitions 1-5

**Tasks:**

1. **Implement Custom Partitioner:**

```typescript
import { Kafka, Partitioners, PartitionerArgs } from 'kafkajs';

// Custom partitioner function
const vipPartitioner = (): ((args: PartitionerArgs) => number) => {
  return ({ topic, partitionMetadata, message }: PartitionerArgs) => {
    const partitionCount = partitionMetadata.length;
    const userId = message.key?.toString() || '';
    
    // TODO: Implement partitioning logic
    if (userId.endsWith('_vip')) {
      return 0; // VIP partition
    } else {
      // Regular customers - round robin to partitions 1-5
      // Calculate hash of userId
      const hash = Math.abs(hashString(userId));
      return (hash % 5) + 1;
    }
  };
};

// Simple hash function
function hashString(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash |= 0; // Convert to 32bit integer
  }
  return hash;
}
```

2. **Configure Producer:**
```typescript
const producer = kafka.producer({
  createPartitioner: vipPartitioner
});
```

3. **Test:**
   - Send 20 messages with keys: "user-1", "user-2", ..., "user-10_vip", "user-11_vip"
   - Verify VIP users go to partition 0
   - Verify regular users distributed across partitions 1-5

**Verification:**
```bash
kafka-console-consumer.sh --topic vip-events \
  --from-beginning \
  --bootstrap-server localhost:9092 \
  --property print.key=true \
  --property print.partition=true
```

---

## Exercise 5: Producer Configuration and Reliability (60 minutes)

**Objective:** Configure producer for different reliability requirements

### Part A: Configure for Maximum Throughput

```typescript
import { Kafka, CompressionTypes } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });

const highThroughputProducer = kafka.producer({
  // TODO: Add configurations for maximum throughput
  compression: CompressionTypes.LZ4, // or use ? to fill
  maxInFlightRequests: 5,
  idempotent: false, // Disable for higher throughput
  retry: {
    retries: 3
  },
  // Note: batch size and linger handled internally by KafkaJS
});
```

### Part B: Configure for Maximum Reliability

```typescript
import { Kafka, CompressionTypes } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });

const highReliabilityProducer = kafka.producer({
  // TODO: Add configurations for maximum reliability
  idempotent: true, // or use ?
  maxInFlightRequests: 5, // or use ?
  compression: CompressionTypes.Snappy,
  retry: {
    retries: Number.MAX_SAFE_INTEGER, // or use ?
    initialRetryTime: 100, // or use ?
    maxRetryTime: 30000
  },
  timeout: 30000
});
```

### Part C: Benchmark Both Configurations

**Tasks:**
1. Send 100,000 messages using each configuration
2. Measure:
   - Throughput (messages/sec)
   - Average latency
   - P99 latency
3. Compare results

**Fill in:**
```
Configuration         | Throughput | Avg Latency | P99 Latency
----------------------|------------|-------------|------------
Maximum Throughput    | ?          | ?           | ?
Maximum Reliability   | ?          | ?           | ?
```

**Questions:**
1. What's the trade-off between throughput and reliability?
2. In what scenarios would you prioritize throughput?
3. In what scenarios would you prioritize reliability?

---

## Exercise 6: Error Handling and Retries (50 minutes)

**Objective:** Implement robust error handling

**Scenario:**
Your producer needs to handle various failures gracefully.

**Tasks:**

1. **Implement Retry Logic with Exponential Backoff:**

```typescript
import { Kafka, Producer, Message } from 'kafkajs';

class ResilientProducer {
  private producer: Producer;
  private readonly maxRetries = 3;

  constructor(kafka: Kafka) {
    this.producer = kafka.producer();
  }

  async sendWithRetry(topic: string, message: Message): Promise<void> {
    let attempt = 0;
    
    while (attempt < this.maxRetries) {
      try {
        const metadata = await this.producer.send({
          topic,
          messages: [message]
        });
        console.log(`Sent successfully to partition ${metadata[0].partition}`);
        return;
      } catch (error) {
        attempt++;
        if (attempt >= this.maxRetries) {
          // TODO: Send to Dead Letter Queue
          await this.handleFailure(topic, message, error as Error);
        } else {
          // TODO: Exponential backoff
          const backoffMs = Math.pow(2, attempt) * 1000;
          await this.sleep(backoffMs);
        }
      }
    }
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  private async handleFailure(topic: string, message: Message, error: Error): Promise<void> {
    // TODO: Implement Dead Letter Queue logic
  }
}
```

2. **Implement Dead Letter Queue (DLQ):**
   - Failed messages should be sent to `failed-events` topic
   - Include original topic, error message, and timestamp

3. **Test Error Scenarios:**
   - Stop Kafka broker during sending
   - Send messages larger than `max.request.size`
   - Send to non-existent topic

---

## Exercise 7: Transaction Support (60 minutes)

**Objective:** Implement exactly-once semantics with transactions

**Scenario:**
You need to atomically send multiple related messages (all succeed or all fail).

**Tasks:**

1. **Implement Transactional Producer:**

```typescript
import { Kafka, Producer, Message, Transaction } from 'kafkajs';

class TransactionalProducer {
  private producer: Producer;

  constructor(kafka: Kafka) {
    this.producer = kafka.producer({
      transactionalId: 'my-transactional-id',
      idempotent: true,
      maxInFlightRequests: 1
    });
  }

  async sendInTransaction(topic: string, messages: Message[]): Promise<void> {
    const transaction: Transaction = await this.producer.transaction();
    
    try {
      for (const message of messages) {
        await transaction.send({
          topic,
          messages: [message]
        });
      }
      
      await transaction.commit();
      console.log('Transaction committed successfully');
    } catch (error) {
      await transaction.abort();
      console.error(`Transaction aborted: ${(error as Error).message}`);
    }
  }
}
```

2. **Test Transactional Sending:**
   - Send 5 order-related messages in one transaction:
     * `order-created`
     * `inventory-reserved`
     * `payment-initiated`
     * `notification-sent`
     * `order-confirmed`
   - Simulate failure after 3rd message
   - Verify all-or-nothing behavior

3. **Verify with Consumer:**
   - Configure consumer with `isolation.level=read_committed`
   - Verify it only sees committed messages

---

## Challenge Exercise: Production-Ready Producer Service (120 minutes)

**Objective:** Build a complete, production-ready producer microservice

**Requirements:**
- REST API to accept events
- Async processing with queue
- Monitoring (metrics, health checks)
- Error handling with DLQ
- Configuration management
- Unit and integration tests

**Architecture:**
```
REST API → In-Memory Queue → Producer Thread Pool → Kafka
                                     ↓
                              Dead Letter Queue
```

**Tasks:**

1. **Create NestJS Service:**

```typescript
import { Controller, Post, Get, Body } from '@nestjs/common';
import { EventProducerService } from './event-producer.service';

interface EventResponse {
  eventId: string;
  status: string;
}

interface HealthStatus {
  status: string;
}

@Controller('api/events')
export class EventController {
  constructor(private readonly producerService: EventProducerService) {}
  
  @Post()
  async sendEvent(@Body() event: Event): Promise<EventResponse> {
    const eventId = await this.producerService.sendAsync(event);
    return { eventId, status: 'ACCEPTED' };
  }
  
  @Get('/health')
  async health(): Promise<HealthStatus> {
    // TODO: Check Kafka connectivity
    return { status: 'UP' };
  }
}
```

2. **Implement Producer Service:**

```typescript
import { Injectable } from '@nestjs/common';
import { Kafka, Producer } from 'kafkajs';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class EventProducerService {
  private producer: Producer;
  private successCount = 0;
  private failureCount = 0;
  
  constructor(private readonly kafka: Kafka) {
    this.producer = kafka.producer();
  }
  
  async sendAsync(event: Event): Promise<string> {
    event.eventId = uuidv4();
    
    // Process asynchronously
    setImmediate(async () => {
      try {
        await this.producer.send({
          topic: 'events',
          messages: [{
            key: event.userId,
            value: JSON.stringify(event)
          }]
        });
        this.successCount++;
      } catch (error) {
        this.failureCount++;
        await this.sendToDeadLetterQueue(event, error as Error);
      }
    });
    
    return event.eventId;
  }
  
  private async sendToDeadLetterQueue(event: Event, error: Error): Promise<void> {
    // TODO: Implement DLQ logic
  }
}
```

3. **Add Metrics:**
   - Events sent (success/failure)
   - Latency histogram
   - DLQ messages
   - Queue size

4. **Write Tests:**
   - Unit tests with mocked Kafka
   - Integration tests with embedded Kafka
   - Load tests (1000 req/sec)

5. **Create Docker Compose Setup:**
   - Application
   - Kafka
   - Prometheus
   - Grafana

**Deliverables:**
- Complete source code
- README with setup instructions
- API documentation
- Test results
- Performance benchmarks

---

## Solutions & Discussion

### Exercise 2 Expected Results

```
Method           | Time (ms) | Messages/sec | Notes
-----------------|-----------|--------------|-------
Fire-and-forget  | 2000      | 5000        | Fastest but unreliable
Synchronous      | 30000     | 333         | Slowest but reliable
Asynchronous     | 3000      | 3333        | Good balance
```

### Exercise 5 Recommended Configurations

**Maximum Throughput:**
```typescript
const producer = kafka.producer({
  compression: CompressionTypes.LZ4,
  maxInFlightRequests: 5,
  idempotent: false,
  retry: { retries: 3 }
});
// Note: In KafkaJS, batch size and linger are managed internally
```

**Maximum Reliability:**
```typescript
const producer = kafka.producer({
  idempotent: true,
  maxInFlightRequests: 5,
  compression: CompressionTypes.LZ4,
  retry: {
    retries: Number.MAX_SAFE_INTEGER,
    initialRetryTime: 100
  },
  timeout: 30000
});
```

### Exercise 6 DLQ Implementation

```typescript
interface FailedMessage {
  originalTopic: string;
  key: string | null;
  value: string | null;
  errorMessage: string;
  timestamp: number;
}

private async handleFailure(
  topic: string,
  message: Message,
  error: Error
): Promise<void> {
  const failed: FailedMessage = {
    originalTopic: topic,
    key: message.key?.toString() || null,
    value: message.value?.toString() || null,
    errorMessage: error.message,
    timestamp: Date.now()
  };
  
  await this.producer.send({
    topic: 'failed-events',
    messages: [{
      key: message.key,
      value: JSON.stringify(failed)
    }]
  });
}
```

---

## Additional Resources

**Read:**
- [Kafka Producer Internals](https://kafka.apache.org/documentation/#producerapi)
- [Idempotent Producer](https://kafka.apache.org/documentation/#semantics)

**Practice:**
- Implement producer in Python using `kafka-python`
- Try different configuration combinations
- Experiment with transactions and error handling

---

**Time to Complete:** 6-7 hours

**[← Previous: Module 3 Exercises](module-03-exercises.md)** | **[Next: Module 5 Exercises →](module-05-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
