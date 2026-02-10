# Module 3 Exercises: Setting Up Kafka

## Exercise 1: Local Kafka Installation (30 minutes)

**Objective:** Install and run Kafka locally

**Tasks:**

1. **Prerequisites Check:**
   ```bash
   # Verify Node.js installation
   node --version
   # Should be Node.js 16 or higher
   
   # Verify npm installation
   npm --version
   ```

2. **Download and Extract Kafka:**
   ```bash
   wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
   tar -xzf kafka_2.13-3.6.0.tgz
   cd kafka_2.13-3.6.0
   ```

3. **Start Kafka in KRaft Mode:**
   ```bash
   # Generate cluster ID
   KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
   
   # Format storage
   bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
   
   # Start Kafka
   bin/kafka-server-start.sh config/kraft/server.properties
   ```

4. **Verify Installation:**
   ```bash
   # Create test topic
   bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092
   
   # List topics
   bin/kafka-topics.sh --list --bootstrap-server localhost:9092
   ```

**Expected Output:**
```
Created topic test.
test
```

**Troubleshooting Checklist:**
- [ ] Node.js 16+ installed
- [ ] Port 9092 not in use
- [ ] Sufficient disk space (> 10 GB)
- [ ] No firewall blocking localhost

---

## Exercise 2: Docker Compose Setup (45 minutes)

**Objective:** Set up a 3-broker Kafka cluster using Docker

**Tasks:**

1. **Create docker-compose.yml:**

Save this to `docker-compose.yml`:

```yaml
version: '3.8'

services:
  kafka-1:
    image: apache/kafka:latest
    container_name: kafka-1
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_LOG_DIRS: /tmp/kraft-logs
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk

  kafka-2:
    image: apache/kafka:latest
    container_name: kafka-2
    ports:
      - "9093:9092"
    environment:
      KAFKA_NODE_ID: 2
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9093
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_LOG_DIRS: /tmp/kraft-logs
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk

  kafka-3:
    image: apache/kafka:latest
    container_name: kafka-3
    ports:
      - "9094:9092"
    environment:
      KAFKA_NODE_ID: 3
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9094
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_LOG_DIRS: /tmp/kraft-logs
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk
```

2. **Start the Cluster:**
   ```bash
   docker-compose up -d
   ```

3. **Verify All Brokers Running:**
   ```bash
   docker ps
   # Should show 3 Kafka containers
   ```

4. **Check Logs:**
   ```bash
   docker logs kafka-1
   # Look for "Kafka Server started"
   ```

5. **Test Connectivity:**
   ```bash
   # From host machine
   docker exec -it kafka-1 bash
   
   # Inside container
   kafka-broker-api-versions.sh --bootstrap-server localhost:9092
   ```

**Verification Checklist:**
- [ ] All 3 containers running
- [ ] No errors in logs
- [ ] Can connect to each broker
- [ ] Cluster ID matches across all brokers

---

## Exercise 3: CLI Tools Mastery (60 minutes)

**Objective:** Master essential Kafka CLI tools

### Part A: Topic Management

```bash
# Create topic with specific configuration
kafka-topics.sh --create \
  --topic user-activity \
  --bootstrap-server localhost:9092 \
  --partitions 6 \
  --replication-factor 3 \
  --config retention.ms=604800000 \
  --config compression.type=lz4

# Describe topic
kafka-topics.sh --describe \
  --topic user-activity \
  --bootstrap-server localhost:9092

# List all topics
kafka-topics.sh --list \
  --bootstrap-server localhost:9092

# Alter topic configuration
kafka-configs.sh --alter \
  --entity-type topics \
  --entity-name user-activity \
  --add-config retention.ms=1209600000 \
  --bootstrap-server localhost:9092

# Delete topic
kafka-topics.sh --delete \
  --topic user-activity \
  --bootstrap-server localhost:9092
```

**Tasks:**
1. Create 3 topics with different configurations:
   - `logs`: 12 partitions, 7-day retention
   - `metrics`: 6 partitions, 30-day retention, compaction
   - `events`: 3 partitions, 1-day retention

2. Describe each topic and verify configurations

3. Change `metrics` retention to 60 days

### Part B: Producer and Consumer CLI

**Producer:**
```bash
# Basic producer
kafka-console-producer.sh \
  --topic events \
  --bootstrap-server localhost:9092

# Producer with keys
kafka-console-producer.sh \
  --topic events \
  --bootstrap-server localhost:9092 \
  --property "parse.key=true" \
  --property "key.separator=:"

# Example input:
# user1:{"action":"login","timestamp":123}
# user2:{"action":"purchase","timestamp":456}
```

**Consumer:**
```bash
# Basic consumer
kafka-console-consumer.sh \
  --topic events \
  --from-beginning \
  --bootstrap-server localhost:9092

# Consumer with keys
kafka-console-consumer.sh \
  --topic events \
  --from-beginning \
  --bootstrap-server localhost:9092 \
  --property print.key=true \
  --property key.separator=":"

# Consumer group
kafka-console-consumer.sh \
  --topic events \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group
```

**Tasks:**
1. Produce 10 messages with keys to `events` topic
2. Consume from beginning and verify all messages
3. Start 2 consumers in the same group
4. Produce more messages and observe load balancing

### Part C: Consumer Groups Management

```bash
# List consumer groups
kafka-consumer-groups.sh --list \
  --bootstrap-server localhost:9092

# Describe consumer group
kafka-consumer-groups.sh --describe \
  --group my-consumer-group \
  --bootstrap-server localhost:9092

# Reset offsets to beginning
kafka-consumer-groups.sh --reset-offsets \
  --group my-consumer-group \
  --topic events \
  --to-earliest \
  --execute \
  --bootstrap-server localhost:9092

# Reset offsets to specific timestamp
kafka-consumer-groups.sh --reset-offsets \
  --group my-consumer-group \
  --topic events \
  --to-datetime 2024-01-01T00:00:00.000 \
  --execute \
  --bootstrap-server localhost:9092

# Delete consumer group
kafka-consumer-groups.sh --delete \
  --group my-consumer-group \
  --bootstrap-server localhost:9092
```

**Tasks:**
1. Create a consumer group and consume some messages
2. Stop the consumer
3. Check the lag for the consumer group
4. Reset offsets to beginning
5. Verify offsets were reset

---

## Exercise 4: Kafka UI Setup (20 minutes)

**Objective:** Set up a web UI for easier Kafka management

**Option 1: Kafka UI by Provectus**

Add to your `docker-compose.yml`:
```yaml
  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka-1:9092,kafka-2:9092,kafka-3:9092
    depends_on:
      - kafka-1
      - kafka-2
      - kafka-3
```

**Tasks:**
1. Add Kafka UI to your Docker Compose
2. Start the service: `docker-compose up -d kafka-ui`
3. Open http://localhost:8080
4. Explore:
   - Brokers tab
   - Topics list
   - Consumer groups
   - Create a new topic via UI
   - Produce a message via UI
   - Browse messages

---

## Exercise 5: Configuration Tuning (30 minutes)

**Objective:** Configure Kafka for production-like performance

**Tasks:**

1. **Modify server.properties for performance:**

```properties
# Network settings
num.network.threads=8
num.io.threads=16
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# Log settings
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000

# Replication settings
default.replication.factor=3
min.insync.replicas=2
replica.lag.time.max.ms=30000

# Compression
compression.type=lz4

# Memory
log.flush.interval.messages=10000
log.flush.interval.ms=1000
```

2. **Create topics with different retention policies:**

Create configuration files:

**high-throughput-topic.properties:**
```properties
compression.type=lz4
retention.ms=86400000
segment.ms=600000
```

**long-retention-topic.properties:**
```properties
compression.type=gzip
retention.ms=31536000000
segment.ms=86400000
```

Apply configurations:
```bash
kafka-topics.sh --create \
  --topic high-throughput \
  --bootstrap-server localhost:9092 \
  --partitions 12 \
  --replication-factor 3 \
  --config compression.type=lz4 \
  --config retention.ms=86400000
```

3. **Verify configurations:**
```bash
kafka-configs.sh --describe \
  --entity-type topics \
  --entity-name high-throughput \
  --bootstrap-server localhost:9092
```

---

## Exercise 6: Performance Benchmarking (40 minutes)

**Objective:** Benchmark your Kafka cluster

**Producer Performance Test:**
```bash
kafka-producer-perf-test.sh \
  --topic perf-test \
  --num-records 1000000 \
  --record-size 1024 \
  --throughput -1 \
  --producer-props \
    bootstrap.servers=localhost:9092 \
    acks=1 \
    compression.type=lz4
```

**Consumer Performance Test:**
```bash
kafka-consumer-perf-test.sh \
  --topic perf-test \
  --messages 1000000 \
  --bootstrap-server localhost:9092 \
  --threads 1
```

**Tasks:**

1. Run producer performance test with:
   - `acks=1`
   - `acks=all`
   - Different compression types (none, gzip, lz4, snappy)

2. Record results:
   ```
   Configuration | Records/sec | MB/sec | Avg Latency | Max Latency
   acks=1, lz4   | ?          | ?      | ?           | ?
   acks=all,lz4  | ?          | ?      | ?           | ?
   ```

3. Run consumer performance test and compare with producer throughput

4. Analyze:
   - What's the bottleneck?
   - CPU, disk, or network?
   - Can your cluster handle your production load?

---

## Challenge Exercise: High Availability Setup (90 minutes)

**Objective:** Set up a production-ready HA Kafka cluster

**Requirements:**
- 3 Kafka brokers
- Schema Registry
- Kafka Connect
- Monitoring (Prometheus + Grafana)

**Tasks:**

1. **Create comprehensive docker-compose.yml** with:
   - 3 Kafka brokers  
   - Schema Registry
   - Kafka Connect
   - Prometheus
   - Grafana
   - Kafka UI

2. **Configure broker properties** for HA:
   - Proper replication factors
   - min.insync.replicas
   - Rack awareness (if applicable)

3. **Test failure scenarios:**
   - Stop one broker - verify cluster still works
   - Stop two brokers - what happens?
   - Restart brokers - verify data intact

4. **Set up monitoring dashboards:**
   - Broker health
   - Topic metrics
   - Consumer lag
   - Disk usage

**Deliverable:**
- Complete docker-compose.yml
- Configuration files
- Test results document
- Dashboard screenshots

---

## Troubleshooting Common Issues

### Issue 1: "Connection refused"
```bash
# Check if Kafka is running
docker ps

# Check Kafka logs
docker logs kafka-1

# Verify port is not blocked
telnet localhost 9092
```

### Issue 2: "Topic already exists"
```bash
# Delete topic
kafka-topics.sh --delete --topic my-topic --bootstrap-server localhost:9092

# Or use force delete
kafka-topics.sh --delete --topic my-topic --bootstrap-server localhost:9092 --force
```

### Issue 3: "Not enough in-sync replicas"
```bash
# Check topic configuration
kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092

# Verify min.insync.replicas setting
kafka-configs.sh --describe --entity-type topics --entity-name my-topic \
  --bootstrap-server localhost:9092

# Lower min.insync.replicas if needed (NOT for production)
kafka-configs.sh --alter --entity-type topics --entity-name my-topic \
  --add-config min.insync.replicas=1 --bootstrap-server localhost:9092
```

---

## Solutions & Tips

### Exercise 3 Solutions

**Creating topics with specific configs:**
```bash
# Logs topic
kafka-topics.sh --create --topic logs --bootstrap-server localhost:9092 \
  --partitions 12 --replication-factor 3 --config retention.ms=604800000

# Metrics topic (with compaction)
kafka-topics.sh --create --topic metrics --bootstrap-server localhost:9092 \
  --partitions 6 --replication-factor 3 \
  --config cleanup.policy=compact --config retention.ms=2592000000

# Events topic
kafka-topics.sh --create --topic events --bootstrap-server localhost:9092 \
  --partitions 3 --replication-factor 3 --config retention.ms=86400000
```

### Exercise 6 Results (Sample)

Typical results on a modern laptop:
```
acks=1, lz4:      ~200K records/sec, ~200 MB/sec, 5ms avg latency
acks=all, lz4:    ~100K records/sec, ~100 MB/sec, 12ms avg latency
acks=all, gzip:   ~80K records/sec, ~60 MB/sec, 15ms avg latency
```

---

**Time to Complete:** 4-5 hours

**[← Previous: Module 2 Exercises](module-02-exercises.md)** | **[Next: Module 4 Exercises →](module-04-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
