# Module 5 Exercises: Kafka Consumers

## Exercise 1: Basic Consumer Implementation (30 minutes)

**Objective:** Implement a basic Kafka consumer

### TypeScript Implementation

**Tasks:**
1. Create a simple consumer that reads from `test-events` topic
2. Print each message with key, value, partition, and offset

**BasicConsumer.ts:**
```typescript
import { Kafka, Consumer, EachMessagePayload } from 'kafkajs';

class BasicConsumer {
  private kafka: Kafka;
  private consumer: Consumer;

  constructor() {
    // TODO: Create Kafka instance
    this.kafka = new Kafka({
      clientId: 'my-consumer',
      brokers: ['localhost:9092']
    });
    
    // TODO: Create consumer
    this.consumer = this.kafka.consumer({
      groupId: 'my-consumer-group'
    });
  }

  async consume(): Promise<void> {
    // TODO: Connect and subscribe to topic
    await this.consumer.connect();
    await this.consumer.subscribe({ 
      topic: 'test-events', 
      fromBeginning: true 
    });
    
    try {
      // TODO: Poll for records
      await this.consumer.run({
        eachMessage: async ({ topic, partition, message }: EachMessagePayload) => {
          console.log(
            `key=${message.key?.toString()}, ` +
            `value=${message.value?.toString()}, ` +
            `partition=${partition}, ` +
            `offset=${message.offset}`
          );
        }
      });
    } finally {
      await this.consumer.disconnect();
    }
  }
}

// Usage
const consumer = new BasicConsumer();
consumer.consume().catch(console.error);
```

**Verification:**
- Produce some messages to `test-events`
- Run your consumer
- Verify all messages are consumed

---

## Exercise 2: Consumer Groups and Load Balancing (45 minutes)

**Objective:** Understand how consumer groups distribute work

**Setup:**
Create a topic with 6 partitions:
```bash
kafka-topics.sh --create \
  --topic load-balance-test \
  --partitions 6 \
  --replication-factor 1 \
  --bootstrap-server localhost:9092
```

### Part A: Single Consumer

**Task:**
- Start 1 consumer in group `test-group`
- Produce 100 messages
- Observe: Consumer handles all 6 partitions

### Part B: Three Consumers

**Task:**
- Start 3 consumers in the same group `test-group`
- Produce 100 more messages
- Observe partition assignment:
  ```
  Consumer 1 gets partitions: ?
  Consumer 2 gets partitions: ?
  Consumer 3 gets partitions: ?
  ```

### Part C: Six Consumers

**Task:**
- Start 6 consumers in the same group
- Observe: Each consumer gets 1 partition (ideal balance)

### Part D: Seven Consumers

**Task:**
- Start 7 consumers in the same group
- Observe: One consumer sits idle

**Questions:**
1. What's the maximum number of active consumers you can have?
2. What happens when a consumer joins or leaves?
3. How long does rebalancing take?
4. What happens to messages during rebalancing?

---

## Exercise 3: Offset Management Strategies (60 minutes)

**Objective:** Implement different offset commit strategies

### Strategy 1: Auto-Commit (Default)

```typescript
import { Kafka } from 'kafkajs';

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const consumer = kafka.consumer({ 
  groupId: 'my-group',
  // Auto-commit is enabled by default in KafkaJS
  // It commits after each batch is processed
});

// Messages are automatically committed after processing
```

**Test:**
- Process 50 messages
- Crash consumer after processing 30 messages
- Restart consumer
- Which messages are reprocessed?

### Strategy 2: Manual Commit (Synchronous)

```typescript
const consumer = kafka.consumer({ 
  groupId: 'my-group'
});

await consumer.connect();
await consumer.subscribe({ topic: 'my-topic' });

await consumer.run({
  autoCommit: false, // Disable auto-commit
  eachBatch: async ({ batch, resolveOffset, heartbeat, commitOffsetsIfNecessary }) => {
    for (const message of batch.messages) {
      await processRecord(message);
    }
    
    // TODO: Commit after processing all records in batch
    await commitOffsetsIfNecessary();
  }
});
```

**Test:** Same crash test as above. What's different?

### Strategy 3: Manual Commit (Asynchronous)

```typescript
// KafkaJS handles commits asynchronously by default
await consumer.run({
  autoCommit: false,
  eachBatch: async ({ batch, commitOffsetsIfNecessary }) => {
    for (const message of batch.messages) {
      await processRecord(message);
    }
    
    // Commit asynchronously (fire and forget)
    commitOffsetsIfNecessary().catch((error) => {
      console.error('Commit failed:', error.message);
    });
  }
});
```

### Strategy 4: Commit After Each Message

```typescript
await consumer.run({
  autoCommit: false,
  eachMessage: async ({ topic, partition, message }) => {
    await processRecord(message);
    
    // Commit this specific offset
    await consumer.commitOffsets([{
      topic,
      partition,
      offset: (parseInt(message.offset) + 1).toString()
    }]);
  }
});
```

### Strategy 5: Store Offsets Externally

```typescript
// Store offsets in database
await consumer.run({
  autoCommit: false,
  eachMessage: async ({ topic, partition, message }) => {
    // Process in transaction
    await database.transaction(async (trx) => {
      try {
        await processRecord(message, trx);
        await trx('kafka_offsets').insert({
          topic,
          partition,
          offset: message.offset,
          timestamp: new Date()
        });
        // Transaction commits automatically on success
      } catch (error) {
        // Transaction rolls back automatically on error
        throw error;
      }
    });
  }
});
```

**Tasks:**
1. Implement all 5 strategies
2. Compare delivery guarantees:
   - At-most-once
   - At-least-once
   - Exactly-once
3. Measure performance impact

**Fill in:**
```
Strategy              | Delivery Guarantee | Performance | Complexity
----------------------|--------------------| ------------|-----------
Auto-commit           | ?                  | ?           | Low
Manual sync commit    | ?                  | ?           | Medium
Manual async commit   | ?                  | ?           | Medium
Per-message commit    | ?                  | ?           | High
External offset store | ?                  | ?           | Very High
```

---

## Exercise 4: Handling Consumer Rebalancing (50 minutes)

**Objective:** Implement graceful rebalancing

**Tasks:**

1. **Implement Rebalance Listener:**

```typescript
import { Kafka, Consumer, EachMessagePayload, ConsumerSubscribeTopics } from 'kafkajs';

class SaveOffsetsOnRebalance {
  private consumer: Consumer;
  
  constructor(consumer: Consumer) {
    this.consumer = consumer;
  }

  async onPartitionsRevoked(partitions: Array<{ topic: string; partition: number }>): Promise<void> {
    // TODO: Save offsets before partitions are reassigned
    console.log('Partitions revoked:', partitions);
    await this.consumer.commitOffsets(
      partitions.map(p => ({
        topic: p.topic,
        partition: p.partition,
        offset: '0' // Will commit current position
      }))
    );
  }
  
  async onPartitionsAssigned(partitions: Array<{ topic: string; partition: number }>): Promise<void> {
    // TODO: Initialize state for new partitions
    console.log('Partitions assigned:', partitions);
  }
}

// Usage with KafkaJS
const kafka = new Kafka({ brokers: ['localhost:9092'] });
const consumer = kafka.consumer({ groupId: 'my-group' });

await consumer.connect();
await consumer.subscribe({ topic: 'events' });

const rebalanceHandler = new SaveOffsetsOnRebalance(consumer);

// KafkaJS handles rebalancing automatically, but we can hook into it
consumer.on('consumer.group.rebalancing', async (event) => {
  await rebalanceHandler.onPartitionsRevoked(event.payload.partitions || []);
});

consumer.on('consumer.connect', async (event) => {
  // Partitions assigned after connection
});
```

2. **Test Rebalancing:**
   - Start 2 consumers
   - After 30 seconds, start a 3rd consumer
   - Observe rebalancing in logs
   - Verify no messages are lost or duplicated

3. **Measure Rebalance Duration:**
   - Track time from "partition revoked" to "partition assigned"
   - What factors affect rebalance duration?

---

## Exercise 5: Consumer Lag Monitoring (45 minutes)

**Objective:** Monitor and reduce consumer lag

### Part A: Generate Lag

**Tasks:**
1. Start a fast producer (1000 msg/sec)
2. Start a slow consumer (100 msg/sec)
3. Let it run for 2 minutes
4. Check lag:

```bash
kafka-consumer-groups.sh --describe \
  --group my-group \
  --bootstrap-server localhost:9092
```

**Expected Output:**
```
TOPIC     PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
events    0          1000            11000           10000
events    1          1200            11500           10300
...
```

### Part B: Reduce Lag

Implement these strategies and measure impact:

**Strategy 1: Increase max.poll.records**
```java
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "1000");  // Up from 500
```

**Strategy 2: Parallel Processing**
```typescript
import { Worker } from 'worker_threads';
import pLimit from 'p-limit';

const limit = pLimit(10); // Limit to 10 concurrent tasks

await consumer.run({
  autoCommit: false,
  eachBatch: async ({ batch, resolveOffset, commitOffsetsIfNecessary, heartbeat }) => {
    const promises: Promise<void>[] = [];
    
    for (const message of batch.messages) {
      const promise = limit(async () => {
        await processRecord(message);
        // Resolve offset to mark as processed
        resolveOffset(message.offset);
        await heartbeat();
      });
      promises.push(promise);
    }
    
    // Wait for all processing to complete
    await Promise.all(promises);
    
    // Commit after all messages processed
    await commitOffsetsIfNecessary();
  }
});
```

**Strategy 3: Add More Consumers**
- Scale from 1 to 3 consumers
- Measure lag reduction

**Fill in Results:**
```
Strategy                | Initial Lag | Final Lag | Time to Catch Up
------------------------|-------------|-----------|------------------
Baseline (1 consumer)   | 100,000     | ?         | ?
Increase max.poll.rec   | 100,000     | ?         | ?
Parallel processing     | 100,000     | ?         | ?
3 consumers             | 100,000     | ?         | ?
```

---

## Exercise 6: Error Handling and Dead Letter Queue (60 minutes)

**Objective:** Handle processing failures gracefully

**Scenario:**
Some messages fail processing (e.g., invalid JSON, business rule violations).

**Requirements:**
- Retry failed messages up to 3 times
- After 3 failures, send to Dead Letter Queue
- Don't block processing of other messages

**Implementation:**

```typescript
import { Kafka, Producer, Consumer, EachMessagePayload } from 'kafkajs';

interface RetryTracker {
  [key: string]: number;
}

class ResilientConsumer {
  private consumer: Consumer;
  private dlqProducer: Producer;
  private retryCount: RetryTracker = {};
  
  constructor(kafka: Kafka) {
    this.consumer = kafka.consumer({ groupId: 'resilient-group' });
    this.dlqProducer = kafka.producer();
  }

  async consume(): Promise<void> {
    await this.consumer.connect();
    await this.dlqProducer.connect();
    await this.consumer.subscribe({ topic: 'events' });
    
    await this.consumer.run({
      eachMessage: async ({ topic, partition, message }: EachMessagePayload) => {
        const messageKey = message.key?.toString() || `${partition}-${message.offset}`;
        let attempts = this.retryCount[messageKey] || 0;
        
        try {
          await this.processRecord(message);
          delete this.retryCount[messageKey];
        } catch (error) {
          attempts++;
          if (attempts >= 3) {
            await this.sendToDeadLetterQueue(message, error as Error);
            delete this.retryCount[messageKey];
          } else {
            this.retryCount[messageKey] = attempts;
            // Re-publish to same topic for retry
            await this.republishForRetry(message);
          }
        }
      }
    });
  }
  
  private async processRecord(message: any): Promise<void> {
    // Process record logic
    const value = message.value?.toString();
    // ... processing logic that might throw
  }
  
  private async sendToDeadLetterQueue(message: any, error: Error): Promise<void> {
    // TODO: Implement DLQ logic
    const dlqMessage = {
      originalTopic: 'events',
      originalKey: message.key?.toString(),
      originalValue: message.value?.toString(),
      error: error.message,
      timestamp: Date.now()
    };
    
    await this.dlqProducer.send({
      topic: 'dlq-events',
      messages: [{
        key: message.key,
        value: JSON.stringify(dlqMessage)
      }]
    });
  }

  private async republishForRetry(message: any): Promise<void> {
    // Republish to original topic with delay header
    await this.dlqProducer.send({
      topic: 'events',
      messages: [{
        key: message.key,
        value: message.value,
        headers: {
          'retry-count': this.retryCount[message.key?.toString() || ''].toString()
        }
      }]
    });
  }
}
```

**Tasks:**
1. Implement the full resilient consumer
2. Test with messages that randomly fail (20% failure rate)
3. Verify failed messages end up in DLQ after 3 attempts
4. Monitor DLQ and implement alerting

---

## Exercise 7: Exactly-Once Semantics (60 minutes)

**Objective:** Implement read-process-write pattern with exactly-once semantics

**Scenario:**
Read from `input-topic`, transform, write to `output-topic` with exactly-once guarantee.

**Implementation:**

```typescript
import { Kafka, Producer, Consumer, EachBatchPayload } from 'kafkajs';

class ExactlyOnceProcessor {
  private kafka: Kafka;
  private consumer: Consumer;
  private producer: Producer;
  
  constructor() {
    // Kafka configuration
    this.kafka = new Kafka({
      clientId: 'eos-processor',
      brokers: ['localhost:9092']
    });
    
    // Consumer configuration with read_committed isolation
    this.consumer = this.kafka.consumer({
      groupId: 'eos-processor',
      // KafkaJS doesn't support read_committed isolation level yet
      // This is a limitation compared to Java
    });
    
    // Producer configuration with idempotence and transactions
    this.producer = this.kafka.producer({
      transactionalId: 'eos-processor-1',
      idempotent: true,
      maxInFlightRequests: 1
    });
  }
  
  async processWithEOS(): Promise<void> {
    await this.consumer.connect();
    await this.producer.connect();
    await this.consumer.subscribe({ topic: 'input-topic' });
    
    await this.consumer.run({
      autoCommit: false,
      eachBatch: async ({ batch, resolveOffset, commitOffsetsIfNecessary }: EachBatchPayload) => {
        if (batch.messages.length === 0) return;
        
        // Begin transaction
        const transaction = await this.producer.transaction();
        
        try {
          for (const message of batch.messages) {
            // Transform
            const transformedValue = this.transform(message.value?.toString() || '');
            
            // Write to output topic within transaction
            await transaction.send({
              topic: 'output-topic',
              messages: [{
                key: message.key,
                value: transformedValue
              }]
            });
            
            // Mark as resolved
            resolveOffset(message.offset);
          }
          
          // Note: KafkaJS doesn't support sendOffsetsToTransaction
          // So we commit separately (not true exactly-once)
          await transaction.commit();
          await commitOffsetsIfNecessary();
          
        } catch (error) {
          await transaction.abort();
          console.error('Transaction aborted:', error);
        }
      }
    });
  }
  
  private transform(input: string): string {
    // Simple transformation
    return input.toUpperCase();
  }
}

// Usage
const processor = new ExactlyOnceProcessor();
processor.processWithEOS().catch(console.error);
```

**Note:** KafkaJS has limitations with exactly-once semantics compared to Java:
- No `sendOffsetsToTransaction` support yet
- No `read_committed` isolation level
- For true exactly-once, consider using Java Kafka Streams or implement idempotency at application level

**Tasks:**
1. Implement the processor
2. Test:
   - Send 100 messages to `input-topic`
   - Run processor
   - Crash it after processing 50 messages
   - Restart
   - Verify exactly 100 messages in `output-topic` (no duplicates)
3. Compare with non-transactional version

---

## Challenge Exercise: Real-Time Analytics Consumer (120 minutes)

**Objective:** Build a consumer that computes real-time analytics

**Requirements:**
- Consume events from `user-events` topic
- Compute metrics in 1-minute windows:
  - Event count by type
  - Unique users
  - Average session duration
- Store results in database
- Expose metrics via REST API
- Handle late-arriving events

**Architecture:**
```
Kafka → Consumer → In-Memory State → Database
                        ↓
                    REST API
```

**Implementation:**

```typescript
import { Kafka, Consumer, EachMessagePayload } from 'kafkajs';
import express from 'express';

interface Event {
  timestamp: number;
  eventType: string;
  userId: string;
  sessionDuration?: number;
}

class WindowedMetrics {
  private eventCounts: Map<string, number> = new Map();
  private uniqueUsers: Set<string> = new Set();
  private sessionDurations: number[] = [];
  public windowStart: number;
  public windowEnd: number;
  
  constructor(windowStart: number) {
    this.windowStart = windowStart;
    this.windowEnd = windowStart + 60000; // 1 minute window
  }
  
  addEvent(event: Event): void {
    const currentCount = this.eventCounts.get(event.eventType) || 0;
    this.eventCounts.set(event.eventType, currentCount + 1);
    this.uniqueUsers.add(event.userId);
    
    if (event.sessionDuration) {
      this.sessionDurations.push(event.sessionDuration);
    }
  }
  
  isComplete(): boolean {
    // Window is complete if current time > window end + grace period
    return Date.now() > this.windowEnd + 30000; // 30s grace
  }
  
  getMetrics() {
    const avgDuration = this.sessionDurations.length > 0
      ? this.sessionDurations.reduce((a, b) => a + b, 0) / this.sessionDurations.length
      : 0;
    
    return {
      windowStart: this.windowStart,
      windowEnd: this.windowEnd,
      eventCounts: Object.fromEntries(this.eventCounts),
      uniqueUsers: this.uniqueUsers.size,
      avgSessionDuration: avgDuration
    };
  }
}

class AnalyticsConsumer {
  private kafka: Kafka;
  private consumer: Consumer;
  private windowedMetrics: Map<string, WindowedMetrics> = new Map();
  private app: express.Application;
  private database: any; // Your database instance
  
  constructor() {
    this.kafka = new Kafka({
      clientId: 'analytics-consumer',
      brokers: ['localhost:9092']
    });
    
    this.consumer = this.kafka.consumer({
      groupId: 'analytics'
    });
    
    this.app = express();
    this.setupRoutes();
    
    // Flush metrics every minute
    setInterval(() => this.flushMetrics(), 60000);
  }
  
  async consume(payload: EachMessagePayload): Promise<void> {
    const event: Event = JSON.parse(payload.message.value?.toString() || '{}');
    const windowKey = this.getWindowKey(event.timestamp);
    
    if (!this.windowedMetrics.has(windowKey)) {
      this.windowedMetrics.set(windowKey, new WindowedMetrics(
        parseInt(windowKey)
      ));
    }
    
    this.windowedMetrics.get(windowKey)!.addEvent(event);
  }
  
  async start(): Promise<void> {
    await this.consumer.connect();
    await this.consumer.subscribe({ topic: 'user-events' });
    
    await this.consumer.run({
      eachMessage: async (payload) => {
        await this.consume(payload);
      }
    });
    
    // Start REST API
    this.app.listen(3000, () => {
      console.log('Analytics API listening on port 3000');
    });
  }
  
  private flushMetrics(): void {
    for (const [windowKey, metrics] of this.windowedMetrics.entries()) {
      if (metrics.isComplete()) {
        // Save to database
        this.database.save(metrics.getMetrics());
        this.windowedMetrics.delete(windowKey);
      }
    }
  }
  
  private getWindowKey(timestamp: number): string {
    const windowStart = Math.floor(timestamp / 60000) * 60000;
    return windowStart.toString();
  }
  
  private setupRoutes(): void {
    // GET /metrics/current - Current window metrics
    this.app.get('/metrics/current', (req, res) => {
      const currentMetrics: any[] = [];
      for (const metrics of this.windowedMetrics.values()) {
        currentMetrics.push(metrics.getMetrics());
      }
      res.json(currentMetrics);
    });
    
    // GET /metrics/history?start=X&end=Y - Historical metrics
    this.app.get('/metrics/history', async (req, res) => {
      const { start, end } = req.query;
      const historical = await this.database.query({
        windowStart: { $gte: parseInt(start as string) },
        windowEnd: { $lte: parseInt(end as string) }
      });
      res.json(historical);
    });
  }
}

// Usage
const analyticsConsumer = new AnalyticsConsumer();
analyticsConsumer.start().catch(console.error);
```

**Tasks:**
1. Implement complete analytics consumer
2. Add REST endpoints:
   - `GET /metrics/current` - Current window metrics
   - `GET /metrics/history?start=X&end=Y` - Historical metrics
3. Handle consumer scaling (multiple instances)
4. Add monitoring and alerting
5. Performance test with 10,000 events/sec

**Deliverables:**
- Source code
- API documentation
- Performance test results
- Grafana dashboard

---

## Solutions & Discussion

### Exercise 3: Offset Management

**Delivery Guarantees:**
- **Auto-commit:** At-most-once (messages may be lost if consumer crashes after commit but before processing)
- **Manual sync commit:** At-least-once (messages may be reprocessed if consumer crashes after processing but before commit)
- **Per-message commit:** At-least-once with minimal duplicates (but slow)
- **External offset store:** Exactly-once (if implemented with transactions)

### Exercise 5: Expected Lag Reduction

```
Strategy                | Initial Lag | Final Lag | Time to Catch Up
------------------------|-------------|-----------|------------------
Baseline (1 consumer)   | 100,000     | 100,000   | Never (producer faster)
Increase max.poll.rec   | 100,000     | 50,000    | 5 minutes
Parallel processing     | 100,000     | 10,000    | 2 minutes
3 consumers             | 100,000     | 0         | 1 minute
```

**Best Strategy:** Combination of all three for maximum throughput

---

## Additional Resources

**Read:**
- [Kafka Consumer Configuration](https://kafka.apache.org/documentation/#consumerconfigs)
- [Exactly-Once Semantics](https://kafka.apache.org/documentation/#semantics)

**Practice:**
- Implement consumer in Python
- Implement consumer in Node.js
- Build a consumer monitoring dashboard

---

**Time to Complete:** 6-7 hours

**[← Previous: Module 4 Exercises](module-04-exercises.md)** | **[Next: Module 6 Exercises →](module-06-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
