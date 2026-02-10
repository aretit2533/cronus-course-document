# Module 4 Exercises: Kafka Producers

## Exercise 1: Basic Producer Implementation (30 minutes)

**Objective:** Implement a basic Kafka producer in your preferred language

### Java Implementation

**Tasks:**
1. Create a new Maven/Gradle project
2. Add Kafka dependencies
3. Implement a simple producer

**BasicProducer.java:**
```java
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.Properties;

public class BasicProducer {
    public static void main(String[] args) {
        // TODO: Configure producer properties
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        
        // TODO: Create producer
        
        // TODO: Send 100 messages
        
        // TODO: Close producer
    }
}
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
```java
producer.send(record);
```

**2. Synchronous:**
```java
RecordMetadata metadata = producer.send(record).get();
System.out.println("Sent to partition " + metadata.partition() + ", offset " + metadata.offset());
```

**3. Asynchronous with Callback:**
```java
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        exception.printStackTrace();
    } else {
        System.out.println("Sent to partition " + metadata.partition());
    }
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

```java
public class Event {
    private String eventId;
    private String userId;
    private String eventType;
    private long timestamp;
    private Map<String, String> metadata;
    
    // Getters, setters, constructor
}
```

**Tasks:**

1. **Implement Custom Serializer:**

```java
import org.apache.kafka.common.serialization.Serializer;
import com.fasterxml.jackson.databind.ObjectMapper;

public class EventSerializer implements Serializer<Event> {
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public byte[] serialize(String topic, Event event) {
        // TODO: Implement JSON serialization
        return null;
    }
}
```

2. **Configure Producer to Use Custom Serializer:**
```java
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, EventSerializer.class);
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

```java
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;

public class VIPPartitioner implements Partitioner {
    
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes, Cluster cluster) {
        int partitionCount = cluster.partitionCountForTopic(topic);
        
        String userId = (String) key;
        
        // TODO: Implement partitioning logic
        if (userId.endsWith("_vip")) {
            return 0;  // VIP partition
        } else {
            // Regular customers - round robin to partitions 1-5
            // Hint: Use Math.abs(userId.hashCode()) % 5 + 1
            return ?;
        }
    }
    
    @Override
    public void close() {}
    
    @Override
    public void configure(Map<String, ?> configs) {}
}
```

2. **Configure Producer:**
```java
props.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, VIPPartitioner.class);
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

```java
Properties highThroughput = new Properties();
highThroughput.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
highThroughput.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
highThroughput.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

// TODO: Add configurations for maximum throughput
highThroughput.put(ProducerConfig.ACKS_CONFIG, "?");
highThroughput.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "?");
highThroughput.put(ProducerConfig.LINGER_MS_CONFIG, "?");
highThroughput.put(ProducerConfig.BATCH_SIZE_CONFIG, "?");
highThroughput.put(ProducerConfig.BUFFER_MEMORY_CONFIG, "?");
```

### Part B: Configure for Maximum Reliability

```java
Properties highReliability = new Properties();
highReliability.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
highReliability.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
highReliability.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

// TODO: Add configurations for maximum reliability
highReliability.put(ProducerConfig.ACKS_CONFIG, "?");
highReliability.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "?");
highReliability.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, "?");
highReliability.put(ProducerConfig.RETRIES_CONFIG, "?");
highReliability.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, "?");
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

```java
public class ResilientProducer {
    private final KafkaProducer<String, String> producer;
    private final int maxRetries = 3;
    
    public void sendWithRetry(ProducerRecord<String, String> record) {
        int attempt = 0;
        while (attempt < maxRetries) {
            try {
                RecordMetadata metadata = producer.send(record).get();
                System.out.println("Sent successfully to " + metadata.partition());
                return;
            } catch (Exception e) {
                attempt++;
                if (attempt >= maxRetries) {
                    // TODO: Send to Dead Letter Queue
                    handleFailure(record, e);
                } else {
                    // TODO: Exponential backoff
                    long backoffMs = (long) Math.pow(2, attempt) * 1000;
                    try {
                        Thread.sleep(backoffMs);
                    } catch (InterruptedException ie) {
                        Thread.currentThread().interrupt();
                    }
                }
            }
        }
    }
    
    private void handleFailure(ProducerRecord<String, String> record, Exception e) {
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

```java
public class TransactionalProducer {
    private final KafkaProducer<String, String> producer;
    
    public TransactionalProducer() {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        
        // Transaction configuration
        props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "my-transactional-id");
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        
        this.producer = new KafkaProducer<>(props);
        this.producer.initTransactions();
    }
    
    public void sendInTransaction(List<ProducerRecord<String, String>> records) {
        try {
            producer.beginTransaction();
            
            for (ProducerRecord<String, String> record : records) {
                producer.send(record);
            }
            
            producer.commitTransaction();
            System.out.println("Transaction committed successfully");
        } catch (Exception e) {
            producer.abortTransaction();
            System.err.println("Transaction aborted: " + e.getMessage());
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

1. **Create Spring Boot Service:**

```java
@RestController
@RequestMapping("/api/events")
public class EventController {
    
    @Autowired
    private EventProducerService producerService;
    
    @PostMapping
    public ResponseEntity<EventResponse> sendEvent(@RequestBody Event event) {
        String eventId = producerService.sendAsync(event);
        return ResponseEntity.accepted()
            .body(new EventResponse(eventId, "ACCEPTED"));
    }
    
    @GetMapping("/health")
    public ResponseEntity<HealthStatus> health() {
        // TODO: Check Kafka connectivity
        return ResponseEntity.ok(new HealthStatus("UP"));
    }
}
```

2. **Implement Producer Service:**

```java
@Service
public class EventProducerService {
    private final KafkaTemplate<String, Event> kafkaTemplate;
    private final MeterRegistry meterRegistry;
    private final ExecutorService executor = Executors.newFixedThreadPool(10);
    
    public String sendAsync(Event event) {
        event.setEventId(UUID.randomUUID().toString());
        
        CompletableFuture.runAsync(() -> {
            try {
                kafkaTemplate.send("events", event.getUserId(), event)
                    .get(5, TimeUnit.SECONDS);
                meterRegistry.counter("events.sent.success").increment();
            } catch (Exception e) {
                meterRegistry.counter("events.sent.failure").increment();
                sendToDeadLetterQueue(event, e);
            }
        }, executor);
        
        return event.getEventId();
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
```java
acks = "1"
compression.type = "lz4"
linger.ms = "10"
batch.size = "32768"
buffer.memory = "67108864"  // 64 MB
```

**Maximum Reliability:**
```java
acks = "all"
enable.idempotence = "true"
max.in.flight.requests.per.connection = "5"
retries = Integer.MAX_VALUE
retry.backoff.ms = "100"
compression.type = "lz4"
```

### Exercise 6 DLQ Implementation

```java
private void handleFailure(ProducerRecord<String, String> record, Exception e) {
    FailedMessage failed = new FailedMessage();
    failed.setOriginalTopic(record.topic());
    failed.setKey(record.key());
    failed.setValue(record.value());
    failed.setErrorMessage(e.getMessage());
    failed.setTimestamp(System.currentTimeMillis());
    
    ProducerRecord<String, String> dlqRecord = new ProducerRecord<>(
        "failed-events",
        record.key(),
        new ObjectMapper().writeValueAsString(failed)
    );
    
    producer.send(dlqRecord);
}
```

---

## Additional Resources

**Read:**
- [Kafka Producer Internals](https://kafka.apache.org/documentation/#producerapi)
- [Idempotent Producer](https://kafka.apache.org/documentation/#semantics)

**Practice:**
- Implement producer in Python using `kafka-python`
- Implement producer in Node.js using `kafkajs`

---

**Time to Complete:** 6-7 hours

**[← Previous: Module 3 Exercises](module-03-exercises.md)** | **[Next: Module 5 Exercises →](module-05-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
