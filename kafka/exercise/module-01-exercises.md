# Module 1 Exercises: Introduction to Apache Kafka

## Exercise 1: Understanding Event Streaming (15 minutes)

**Objective:** Differentiate between event streaming and traditional messaging

**Tasks:**
1. Draw a diagram comparing point-to-point messaging, pub-sub messaging, and event streaming
2. List 3 business scenarios where event streaming is better than request-response
3. Identify which pattern fits these use cases:
   - User login notification
   - Real-time stock price updates
   - Order processing workflow
   - Social media feed updates

**Verification:**
- Your diagram should show persistent storage in event streaming
- Event streaming should show multiple consumers reading the same events
- You should identify at least 2-3 key differences

---

## Exercise 2: Kafka Use Cases Analysis (20 minutes)

**Objective:** Analyze real-world Kafka use cases

**Tasks:**
1. Research how Netflix, LinkedIn, or Uber uses Kafka (pick one)
2. Document:
   - What problem does Kafka solve for them?
   - What volume of data do they process?
   - What are the key benefits they achieved?
3. Identify a use case in your organization or project where Kafka could add value

**Deliverable:**
- Write a 1-page summary of your findings
- Create a simple architecture diagram showing event flow

---

## Exercise 3: Event-Driven Architecture Design (30 minutes)

**Objective:** Design an event-driven system

**Scenario:**
You're building an online food delivery system with these components:
- Customer app
- Restaurant app
- Delivery driver app
- Payment service
- Notification service

**Tasks:**
1. Identify all events in the system (e.g., OrderPlaced, PaymentProcessed)
2. Draw an architecture diagram showing event flows
3. For each event, specify:
   - Event name
   - Producer service
   - Consumer services
   - Event payload (key fields)

**Sample Event:**
```json
{
  "eventType": "OrderPlaced",
  "eventId": "evt-12345",
  "timestamp": 1234567890,
  "data": {
    "orderId": "ord-001",
    "customerId": "cust-123",
    "restaurantId": "rest-456",
    "items": [...],
    "totalAmount": 29.99
  }
}
```

---

## Exercise 4: Kafka Ecosystem Exploration (20 minutes)

**Objective:** Understand Kafka ecosystem components

**Tasks:**
1. Research and summarize in 2-3 sentences:
   - Kafka Connect
   - Kafka Streams
   - Schema Registry
   - KSQL/ksqlDB
   - Kafka REST Proxy

2. Match each component to a use case:
   - "I need to sync data from MySQL to Kafka" → ?
   - "I need to aggregate events in real-time" → ?
   - "I want to query Kafka topics with SQL" → ?
   - "I need to enforce data contracts" → ?

**Answers:**
- Kafka Connect: _____
- Kafka Streams: _____
- ...

---

## Challenge Exercise: Event Streaming ROI Calculator (45 minutes)

**Objective:** Calculate business value of adopting Kafka

**Scenario:**
Your company processes 1 million transactions per day using batch processing (overnight ETL). Each transaction generates 5 events that need to be processed.

Current state:
- Batch processing runs every 24 hours
- Data latency: 24 hours
- Processing time: 4 hours
- Infrastructure cost: $5,000/month

Proposed state with Kafka:
- Real-time processing (< 1 second latency)
- Processing time: continuous
- Infrastructure cost: $8,000/month
- Enables 3 new real-time features

**Tasks:**
1. Calculate:
   - Events per second that Kafka needs to handle
   - Cost increase
   - Business benefits (quantify if possible)

2. Develop a recommendation: Should the company adopt Kafka?

3. What questions would you ask stakeholders to validate this decision?

---

## Solutions & Discussion

### Exercise 1: Key Differences
1. **Persistence**: Event streaming stores events durably (days/weeks)
2. **Multiple Consumers**: Multiple independent consumers can read the same events
3. **Replay**: Event streaming allows replaying historical events

### Exercise 3: Food Delivery Events
Key events:
1. `OrderPlaced` → Payment, Restaurant, Notification
2. `PaymentProcessed` → Restaurant, Customer
3. `OrderAccepted` → Delivery, Customer
4. `DeliveryStarted` → Customer, Restaurant
5. `OrderDelivered` → Payment, Customer, Analytics

### Challenge Exercise: Sample Calculation
- Events/sec: (1M transactions × 5 events) / 86,400 seconds = ~58 events/sec (average)
- Peak could be 5-10x higher: 300-600 events/sec
- Cost increase: $3,000/month
- Benefits: Real-time insights, better customer experience, competitive advantage

---

## Additional Resources

**Read:**
- ["I ❤️ Logs" by Jay Kreps](https://www.confluent.io/blog/i-heart-logs-event-data-stream-processing-and-apache-kafka/)
- [Kafka Introduction Documentation](https://kafka.apache.org/intro)

**Watch:**
- [Apache Kafka in 5 minutes](https://www.youtube.com/watch?v=PzPXRmVHMxI)

**Explore:**
- [Confluent Use Cases](https://www.confluent.io/use-cases/)

---

**Time to Complete:** 2-3 hours

**[Next: Module 2 Exercises →](module-02-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
