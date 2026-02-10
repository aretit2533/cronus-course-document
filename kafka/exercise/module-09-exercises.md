# Module 9 Exercises: Practical Application Project

## Project Overview

Build a complete **Real-Time E-Commerce Analytics Platform** from scratch. This comprehensive project synthesizes all concepts from Modules 1-8.

**Estimated Time:** 12-16 hours

---

## Phase 1: Project Setup and Planning (2 hours)

### Exercise 1.1: Architecture Design

**Tasks:**
1. Design complete system architecture
2. Identify all components and their interactions
3. Define data flow between components
4. Create sequence diagrams for key workflows

**Deliverable:** Architecture diagram showing:
- Event producers (API service, order service, inventory service)
- Kafka cluster (topics, partitions, replication)
- Stream processors (analytics, fraud detection)
- Consumers (notification, recommendations)
- Data sinks (Elasticsearch, PostgreSQL, S3)
- Monitoring stack (Prometheus, Grafana)

### Exercise 1.2: Topic Design

Design all required topics:

```
Topic Name              | Partitions | RF | Retention | Purpose
------------------------|------------|----|-----------|---------
user-events             | 12         | 3  | 7 days    | User interactions
orders                  | 6          | 3  | 30 days   | Order events
inventory-updates       | 6          | 3  | Compact   | Inventory state
enriched-events         | 12         | 3  | 7 days    | Processed events
fraud-alerts            | 3          | 3  | 90 days   | Fraud alerts
recommendations         | 6          | 3  | 1 day     | User recommendations
analytics-aggregates    | 6          | 3  | 30 days   | Computed metrics
```

**Tasks:**
1. Create all topics with appropriate configurations
2. Document retention rationale
3. Calculate storage requirements
4. Plan partition count for scale

---

## Phase 2: Infrastructure Setup (2 hours)

### Exercise 2.1: Docker Compose Setup

**Tasks:**
1. Create comprehensive `docker-compose.yml` with:
   - 3 Kafka brokers (KRaft mode)
   - Schema Registry
   - Kafka Connect
   - PostgreSQL
   - Elasticsearch
   - Prometheus
   - Grafana
   - Kafka UI

2. Configure volumes for persistence

3. Set up networks for component isolation

**Verification:**
```bash
docker-compose up -d
docker ps  # All services running
curl http://localhost:8081/subjects  # Schema Registry
curl http://localhost:9200  # Elasticsearch
```

### Exercise 2.2: Create Schemas

Define Avro schemas for all events:

**UserEvent.avsc:**
```json
{
  "type": "record",
  "name": "UserEvent",
  "namespace": "com.ecommerce.events",
  "fields": [
    {"name": "eventId", "type": "string"},
    {"name": "userId", "type": "string"},
    {"name": "eventType", "type": "string"},
    {"name": "timestamp", "type": "long"},
    {"name": "metadata", "type": {"type": "map", "values": "string"}}
  ]
}
```

**Order.avsc, InventoryUpdate.avsc, etc.**

**Tasks:**
1. Define all schemas
2. Register with Schema Registry
3. Test compatibility modes

---

## Phase 3: Event Producer Services (3 hours)

### Exercise 3.1: REST API Service

**Requirements:**
- Accept HTTP POST requests for user events
- Validate input
- Enrich with metadata (timestamp, IP, user agent)
- Publish to Kafka asynchronously
- Return event ID immediately
- Handle errors gracefully

**Implementation Checklist:**
- [ ] Spring Boot project setup
- [ ] Kafka producer configuration
- [ ] REST controller with validation
- [ ] Async event publishing
- [ ] Error handling and DLQ
- [ ] Health check endpoint
- [ ] Metrics (events published, failures)
- [ ] Unit tests
- [ ] Integration tests

**Sample API:**
```
POST /api/events
{
  "userId": "user-123",
  "eventType": "PAGE_VIEW",
  "metadata": {
    "productId": "prod-456",
    "category": "electronics"
  }
}

Response:
{
  "eventId": "evt-789",
  "status": "ACCEPTED",
  "timestamp": 1234567890
}
```

### Exercise 3.2: Order Service

**Requirements:**
- Create order endpoint
- Validate order data
- Publish `OrderCreated` event
- Handle inventory reservation
- Implement exactly-once semantics

**Tasks:**
1. Implement order creation flow
2. Add transactional producer
3. Implement saga pattern for distributed transaction
4. Add compensation logic for failures
5. Test failure scenarios

### Exercise 3.3: Inventory Service

**Requirements:**
- Manage product inventory
- Publish inventory updates to compacted topic
- Handle reservation and release
- Maintain consistency

**Tasks:**
1. Implement inventory operations
2. Use compacted topic for current state
3. Handle concurrent updates
4. Test edge cases (overselling prevention)

---

## Phase 4: Stream Processing (3 hours)

### Exercise 4.1: Event Enrichment Processor

**Requirements:**
- Read from `user-events` topic
- Join with user profiles (compacted topic)
- Enrich with user segment, tier, preferences
- Write to `enriched-events` topic

**Implementation:**
```java
KStream<String, UserEvent> events = builder.stream("user-events");
KTable<String, UserProfile> profiles = builder.table("user-profiles");

KStream<String, EnrichedEvent> enriched = events
    .join(profiles, (event, profile) -> enrich(event, profile));

enriched.to("enriched-events");
```

**Tasks:**
1. Implement enrichment logic
2. Handle missing profiles gracefully
3. Add metrics and logging
4. Write tests with TopologyTestDriver

### Exercise 4.2: Real-Time Analytics Processor

**Requirements:**
- Count events by type (5-minute windows)
- Calculate unique users per window
- Compute conversion metrics (views → carts → purchases)
- Detect trending products
- Write results to `analytics-aggregates` topic

**Tasks:**
1. Implement windowed aggregations
2. Calculate derived metrics
3. Detect anomalies (sudden spikes/drops)
4. Create interactive queries API
5. Test with realistic data volumes

### Exercise 4.3: Fraud Detection Processor

**Requirements:**
- Detect suspicious patterns:
  * High-value orders (> $10,000)
  * Multiple orders in short time
  * Velocity checks (too many from same IP)
- Score each order for fraud risk
- Generate alerts for high-risk orders

**Tasks:**
1. Implement pattern detection rules
2. Score orders (0-100 risk score)
3. Create alerting logic
4. Test with synthetic fraud scenarios
5. Tune false positive rate

---

## Phase 5: Consumer Services (2 hours)

### Exercise 5.1: Notification Service

**Requirements:**
- Consume order events
- Send email confirmations
- Send fraud alerts
- Handle delivery failures with retry logic

**Tasks:**
1. Implement consumer with manual offset management
2. Integrate with email provider (SendGrid, SES)
3. Add retry logic with exponential backoff
4. Implement DLQ for permanent failures
5. Track notification delivery rate

### Exercise 5.2: Recommendation Engine Consumer

**Requirements:**
- Consume enriched events
- Generate product recommendations based on:
  * User browsing history
  * Similar user behavior
  * Trending products
- Publish to `recommendations` topic
- Expire recommendations after 1 hour

**Tasks:**
1. Implement recommendation algorithm
2. Use in-memory cache for recent events
3. Batch recommendations for efficiency
4. Handle consumer lag gracefully
5. Test recommendation quality

---

## Phase 6: Data Integration (2 hours)

### Exercise 6.1: Elasticsearch Sink

**Requirements:**
- Index all enriched events
- Index fraud alerts
- Enable full-text search
- Create Kibana dashboards

**Connector Configuration:**
```json
{
  "name": "elasticsearch-sink-events",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "3",
    "topics": "enriched-events,fraud-alerts",
    "connection.url": "http://elasticsearch:9200",
    "key.ignore": "false",
    "batch.size": "100"
  }
}
```

**Tasks:**
1. Deploy connector
2. Create index templates
3. Build Kibana dashboards
4. Create searches and visualizations

### Exercise 6.2: PostgreSQL Sink

**Requirements:**
- Persist orders to database
- Store aggregated analytics
- Enable historical queries

**Tasks:**
1. Create database schema
2. Deploy JDBC sink connector
3. Test data integrity
4. Create SQL views for analytics

### Exercise 6.3: S3 Archival

**Requirements:**
- Archive all events to S3
- Partition by date
- Use Parquet format for compression
- Implement retention policy

**Tasks:**
1. Deploy S3 sink connector
2. Configure partitioning strategy
3. Verify file creation
4. Query with Athena/Presto

---

## Phase 7: Monitoring and Operations (2 hours)

### Exercise 7.1: Metrics Collection

**Tasks:**
1. Configure JMX exporters for all brokers
2. Set up Prometheus scraping
3. Create custom application metrics:
   - Events published per service
   - Processing latency
   - Consumer lag
   - Error rates

### Exercise 7.2: Grafana Dashboards

**Create dashboards for:**

**1. Kafka Cluster Health**
- Under-replicated partitions
- Offline partitions
- ISR status
- Controller status

**2. Application Metrics**
- Events per second (by type)
- End-to-end latency
- Consumer lag (by group)
- Error rates

**3. Business Metrics**
- Orders per minute
- Conversion rate
- Fraud detection rate
- Top products viewed

### Exercise 7.3: Alerting

**Configure alerts for:**
- Under-replicated partitions
- High consumer lag (> 10,000)
- Processing errors > 5%
- Fraud alert spike
- Service unavailability

---

## Phase 8: Testing  and Validation (2 hours)

### Exercise 8.1: Unit Tests

**Test each component:**
- Producers (with mocked Kafka)
- Stream processors (with TopologyTestDriver)
- Consumers (with embedded Kafka)

### Exercise 8.2: Integration Tests

**End-to-end test:**
```java
@Test
public void testOrderFlow() {
    // 1. Publish user events
    publishUserEvents();
    
    // 2. Create order
    Order order = orderService.createOrder(request);
    
    // 3. Wait for processing
    await().atMost(10, SECONDS).until(() -> 
        analyticsService.getMetrics() != null);
    
    // 4. Verify enriched events created
    assertEnrichedEventsExist();
    
    // 5. Verify recommendation generated
    assertRecommendationsExist();
    
    // 6. Verify data in Elasticsearch
    assertElasticsearchContainsOrder();
}
```

### Exercise 8.3: Load Testing

**Requirements:**
- Simulate 10,000 events/second
- Measure throughput and latency
- Test with and without failures
- Verify no data loss

**Use tools:**
- kafka-producer-perf-test
- JMeter
- Gatling

**Collect metrics:**
- Producer throughput
- Consumer lag
- Processing latency (p50, p95, p99)
- Error rate

### Exercise 8.4: Failure Testing

**Test scenarios:**
1. Broker failure
2. Consumer crash
3. Network partition
4. High load
5. Schema evolution

---

## Phase 9: Deployment and Optimization (2 hours)

### Exercise 9.1: Docker Deployment

**Tasks:**
1. Build Docker images for all services
2. Create production docker-compose.yml
3. Configure resource limits
4. Set up health checks
5. Test deployment

### Exercise 9.2: Kubernetes Deployment (Optional)

**Create Kubernetes manifests:**
- StatefulSet for Kafka brokers
- Deployments for services
- Services for networking
- ConfigMaps for configuration
- PersistentVolumeClaims for storage

### Exercise 9.3: Performance Optimization

**Tune based on load test results:**

**Identify bottlenecks:**
- Is it CPU, memory, disk, or network?
- Which component is limiting throughput?

**Optimize:**
- Adjust partition count
- Tune producer/consumer configs
- Scale out services
- Optimize queries

**Target metrics:**
- 99th percentile latency < 100ms
- Zero data loss
- Consumer lag < 1000
- Error rate < 0.1%

---

## Final Deliverables

### 1. Source Code
- All service implementations
- Configuration files
- Docker Compose setup
- Test suites

### 2. Documentation
- Architecture design document
- API documentation
- Deployment guide
- Operations runbook
- Troubleshooting guide

### 3. Dashboards
- Grafana dashboard JSON exports
- Kibana dashboard exports
- Screenshots

### 4. Test Reports
- Unit test coverage report
- Integration test results
- Load test results
- Failure test results

### 5. Performance Analysis
- Throughput analysis
- Latency breakdown
- Bottleneck identification
- Optimization recommendations

---

## Evaluation Criteria

### Functionality (40%)
- ✓ All components implemented
- ✓ End-to-end flow works
- ✓ Events processed correctly
- ✓ Data persisted accurately

### Reliability (20%)
- ✓ Handles failures gracefully
- ✓ No data loss
- ✓ Consumer lag managed
- ✓ Errors logged and handled

### Performance (15%)
- ✓ Meets throughput targets
- ✓ Low latency
- ✓ Efficient resource usage
- ✓ Scalable design

### Code Quality (10%)
- ✓ Clean, readable code
- ✓ Proper error handling
- ✓ Good test coverage
- ✓ Documentation

### Operations (15%)
- ✓ Comprehensive monitoring
- ✓ Useful dashboards
- ✓ Effective alerts
- ✓ Clear runbooks

---

## Bonus Challenges

### Challenge 1: Multi-Datacenter Deployment
Deploy across two datacenters with MirrorMaker 2.0 replication

### Challenge 2: Schema Evolution
Evolve schemas while maintaining compatibility:
- Add new field to UserEvent
- Deploy new producers
- Verify old consumers still work

### Challenge 3: Blue-Green Deployment
Implement zero-downtime deployment strategy

### Challenge 4: Chaos Engineering
Use chaos tools to inject failures:
- Random pod kills
- Network delays
- Disk full scenarios

### Challenge 5: Cost Optimization
Analyze and optimize costs:
- Use tiered storage
- Implement retention policies
- Optimize compression
- Right-size infrastructure

---

## Sample Solutions

### Order Processing Flow

```java
@Transactional
public Order createOrder(OrderRequest request) {
    // 1. Validate
    validate(request);
    
    // 2. Create order entity
    Order order = buildOrder(request);
    orderRepository.save(order);
    
    // 3. Publish events transactionally
    kafkaTemplate.executeInTransaction(ops -> {
        // Order created event
        ops.send("orders", order.getId(), order);
        
        // Reserve inventory
        for (OrderItem item : order.getItems()) {
            ops.send("inventory-reservations", 
                item.getProductId(),
                new InventoryReservation(order.getId(), item.getProductId(), item.getQuantity())
            );
        }
        
        // Initiate payment
        ops.send("payment-requests",
            order.getId(),
            new PaymentRequest(order.getId(), order.getTotalAmount())
        );
        
        return true;
    });
    
    return order;
}
```

### Fraud Detection Logic

```java
KStream<String, Order> orders = builder.stream("orders");

// Pattern 1: High value
KStream<String, FraudAlert> highValueAlerts = orders
    .filter((key, order) -> order.getAmount() > 10000)
    .mapValues(order -> new FraudAlert("HIGH_VALUE", order));

// Pattern 2: Velocity check
KTable<Windowed<String>, Long> orderVelocity = orders
    .groupBy((key, order) -> order.getUserId())
    .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(10)))
    .count();

KStream<String, FraudAlert> velocityAlerts = orderVelocity.toStream()
    .filter((windowedKey, count) -> count > 5)
    .map((windowedKey, count) -> 
        KeyValue.pair(windowedKey.key(), 
            new FraudAlert("HIGH_VELOCITY", windowedKey.key(), count))
    );

// Merge all alerts
highValueAlerts.merge(velocityAlerts).to("fraud-alerts");
```

---

## Resources

**Sample Data Generators:**
- faker.js for generating realistic test data
- Mockaroo for CSV/JSON data generation

**Testing Tools:**
- TestContainers for integration tests
- Kafka Streams Test Utils
- WireMock for mocking external APIs

**Monitoring:**
- Kafka exporter for Prometheus
- Grafana Kafka dashboard templates
- ELK stack for log aggregation

---

**Congratulations!** 🎉

Upon completing this project, you'll have:
- Built a production-ready event-driven system
- Applied all Kafka concepts in practice
- Demonstrated end-to-end expertise
- Created a portfolio-worthy project

**Next Steps:**
- Deploy to cloud (AWS, GCP, Azure)
- Add more features (A/B testing, personalization)
- Optimize for larger scale
- Present your project!

---

**[← Previous: Module 8 Exercises](module-08-exercises.md)** | **[📚 Back to Exercises Home](README.md)** | **[📖 Course Home](../README.md)**
