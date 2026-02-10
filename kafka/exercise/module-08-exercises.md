# Module 8 Exercises: Advanced Topics and Best Practices

## Exercise 1: SSL/TLS Security Setup (60 minutes)

**Objective:** Enable encrypted communication in Kafka cluster

### Part A: Generate Certificates

```bash
#!/bin/bash

# Create certificate authority (CA)
openssl req -new -x509 -keyout ca-key -out ca-cert -days 365 \
  -subj "/CN=KafkaCA" -passout pass:password

# Create truststore
keytool -keystore kafka.truststore.jks -alias CARoot -import \
  -file ca-cert -storepass password -noprompt

# For each broker, create keystore and certificate
for i in {1..3}; do
    # Generate keystore
    keytool -keystore kafka.broker$i.keystore.jks -alias localhost \
      -validity 365 -genkey -keyalg RSA \
      -storepass password -keypass password \
      -dname "CN=kafka-$i, OU=Engineering, O=Company, L=City, ST=State, C=US"
    
    # Create certificate signing request
    keytool -keystore kafka.broker$i.keystore.jks -alias localhost \
      -certreq -file cert-file-$i -storepass password
    
    # Sign certificate
    openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file-$i \
      -out cert-signed-$i -days 365 -CAcreateserial -passin pass:password
    
    # Import CA cert into keystore
    keytool -keystore kafka.broker$i.keystore.jks -alias CARoot \
      -import -file ca-cert -storepass password -noprompt
    
    # Import signed certificate
    keytool -keystore kafka.broker$i.keystore.jks -alias localhost \
      -import -file cert-signed-$i -storepass password -noprompt
done
```

### Part B: Configure Brokers for SSL

**server.properties:**
```properties
# Enable SSL listener
listeners=PLAINTEXT://localhost:9092,SSL://localhost:9093
advertised.listeners=PLAINTEXT://localhost:9092,SSL://localhost:9093

# SSL configuration
ssl.keystore.location=/opt/kafka/ssl/kafka.broker1.keystore.jks
ssl.keystore.password=password
ssl.key.password=password
ssl.truststore.location=/opt/kafka/ssl/kafka.truststore.jks
ssl.truststore.password=password

# Client authentication (optional)
ssl.client.auth=required

# Security protocol
security.inter.broker.protocol=SSL
```

### Part C: Configure Producer/Consumer for SSL

**producer.properties:**
```properties
bootstrap.servers=localhost:9093
security.protocol=SSL
ssl.truststore.location=/path/to/kafka.truststore.jks
ssl.truststore.password=password
ssl.keystore.location=/path/to/client.keystore.jks
ssl.keystore.password=password
ssl.key.password=password
```

**Tasks:**
1. Generate certificates for 3-broker cluster
2. Configure all brokers with SSL
3. Test producer and consumer with SSL
4. Measure performance impact (SSL vs plaintext)

**Verification:**
```bash
# Test SSL connection
openssl s_client -connect localhost:9093

# Produce with SSL
kafka-console-producer.sh --topic test \
  --bootstrap-server localhost:9093 \
  --producer.config producer-ssl.properties
```

---

## Exercise 2: SASL Authentication (60 minutes)

**Objective:** Implement authentication with SASL/SCRAM

### Part A: Create Users

```bash
# Create admin user
kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter --add-config 'SCRAM-SHA-256=[password=admin-secret]' \
  --entity-type users --entity-name admin

# Create producer user
kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter --add-config 'SCRAM-SHA-256=[password=producer-secret]' \
  --entity-type users --entity-name producer-user

# Create consumer user
kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter --add-config 'SCRAM-SHA-256=[password=consumer-secret]' \
  --entity-type users --entity-name consumer-user
```

### Part B: Configure Broker

**server.properties:**
```properties
# Enable SASL
listeners=SASL_SSL://localhost:9093
advertised.listeners=SASL_SSL://localhost:9093
security.inter.broker.protocol=SASL_SSL
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
sasl.enabled.mechanisms=SCRAM-SHA-256
```

**kafka_server_jaas.conf:**
```
KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="admin"
    password="admin-secret";
};
```

### Part C: Configure Client

**producer-jaas.conf:**
```
KafkaClient {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="producer-user"
    password="producer-secret";
};
```

**producer.properties:**
```properties
bootstrap.servers=localhost:9093
security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="producer-user" \
    password="producer-secret";
ssl.truststore.location=/path/to/truststore.jks
ssl.truststore.password=password
```

**Tasks:**
1. Set up SASL/SCRAM authentication
2. Create users with different permissions
3. Test authentication with valid and invalid credentials
4. Measure authentication overhead

---

## Exercise 3: ACL Management (45 minutes)

**Objective:** Implement fine-grained authorization

### Part A: Enable ACLs

**server.properties:**
```properties
authorizer.class.name=kafka.security.authorizer.AclAuthorizer
super.users=User:admin
allow.everyone.if.no.acl.found=false
```

### Part B: Grant Permissions

```bash
# Allow producer to write to specific topic
kafka-acls.sh --bootstrap-server localhost:9092 \
  --add --allow-principal User:producer-user \
  --operation Write --topic orders

# Allow consumer to read from topic
kafka-acls.sh --bootstrap-server localhost:9092 \
  --add --allow-principal User:consumer-user \
  --operation Read --topic orders \
  --group order-processor

# Allow consumer to read from consumer group
kafka-acls.sh --bootstrap-server localhost:9092 \
  --add --allow-principal User:consumer-user \
  --operation Read --group order-processor

# List all ACLs
kafka-acls.sh --bootstrap-server localhost:9092 --list

# Remove ACL
kafka-acls.sh --bootstrap-server localhost:9092 \
  --remove --allow-principal User:producer-user \
  --operation Write --topic orders
```

### Part C: Test ACLs

**Tasks:**
1. Try to produce to topic without permission (should fail)
2. Grant write permission and retry (should succeed)
3. Try to consume without permission (should fail)
4. Grant read permission and retry

**Verification:**
```bash
# This should fail
kafka-console-producer.sh --topic orders \
  --bootstrap-server localhost:9093 \
  --producer.config unauthorized-producer.properties

# After granting ACLs, this should succeed
kafka-console-producer.sh --topic orders \
  --bootstrap-server localhost:9093 \
  --producer.config authorized-producer.properties
```

---

## Exercise 4: Performance Tuning (90 minutes)

**Objective:** Optimize Kafka cluster for maximum throughput

### Part A: OS-Level Tuning

```bash
# Increase file descriptors
ulimit -n 100000

# Disable swap
sudo swapoff -a

# Tune network settings
sudo sysctl -w net.core.rmem_max=134217728
sudo sysctl -w net.core.wmem_max=134217728
sudo sysctl -w net.core.rmem_default=134217728
sudo sysctl -w net.core.wmem_default=134217728
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 134217728"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 134217728"
```

### Part B: Broker Tuning

**server.properties:**
```properties
# Increase threads
num.network.threads=8
num.io.threads=16

# Socket buffer sizes
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# Log settings
log.segment.bytes=1073741824
log.flush.interval.messages=10000
log.flush.interval.ms=1000

# Replication settings
replica.fetch.max.bytes=1048576
replica.fetch.wait.max.ms=500

# Compression
compression.type=lz4
```

### Part C: Producer Tuning

**High Throughput Configuration:**
```java
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ProducerConfig.ACKS_CONFIG, "1");
props.put(ProducerConfig.LINGER_MS_CONFIG, "10");
props.put(ProducerConfig.BATCH_SIZE_CONFIG, "32768");  // 32 KB
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, "67108864");  // 64 MB
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4");
props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, "5");
```

**Low Latency Configuration:**
```java
props.put(ProducerConfig.ACKS_CONFIG, "1");
props.put(ProducerConfig.LINGER_MS_CONFIG, "0");
props.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");  // 16 KB
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "none");
props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, "1");
```

### Part D: Consumer Tuning

```java
Properties props = new Properties();
props.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, "50000");
props.put(ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG, "500");
props.put(ConsumerConfig.MAX_PARTITION_FETCH_BYTES_CONFIG, "1048576");
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "1000");
```

### Part E: Benchmark

**Tasks:**
1. Baseline performance test:
   ```bash
   kafka-producer-perf-test.sh \
     --topic perf-test \
     --num-records 1000000 \
     --record-size 1024 \
     --throughput -1 \
     --producer-props bootstrap.servers=localhost:9092
   ```

2. Apply each tuning and retest

3. Record results:
   ```
   Configuration       | Throughput | Latency (p99) | Notes
   --------------------|------------|---------------|-------
   Baseline            | ?          | ?             | ?
   OS tuning           | ?          | ?             | ?
   Broker tuning       | ?          | ?             | ?
   Producer HT         | ?          | ?             | ?
   Producer LL         | ?          | ?             | ?
   Consumer tuning     | ?          | ?             | ?
   All combined        | ?          | ?             | ?
   ```

---

## Exercise 5: Monitoring and Alerting (90 minutes)

**Objective:** Set up comprehensive monitoring

### Part A: JMX Exporter Setup

**kafka-jmx-exporter.yml:**
```yaml
lowercaseOutputName: true
lowercaseOutputLabelNames: true
whitelistObjectNames:
  - kafka.server:type=BrokerTopicMetrics,name=*
  - kafka.server:type=ReplicaManager,name=*
  - kafka.server:type=KafkaRequestHandlerPool,name=*
  - kafka.controller:type=KafkaController,name=*
  - kafka.network:type=RequestMetrics,name=*
  - kafka.network:type=SocketServer,name=*
  - kafka.log:type=LogFlushStats,name=*
  - java.lang:type=GarbageCollector,name=*
  - java.lang:type=Memory
```

**Start broker with JMX exporter:**
```bash
export KAFKA_OPTS="-javaagent:/opt/jmx_exporter/jmx_prometheus_javaagent.jar=7071:/opt/jmx_exporter/kafka-jmx-exporter.yml"
kafka-server-start.sh config/server.properties
```

### Part B: Prometheus Configuration

**prometheus.yml:**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets:
          - 'kafka-1:7071'
          - 'kafka-2:7071'
          - 'kafka-3:7071'
```

### Part C: Alert Rules

**kafka-alerts.yml:**
```yaml
groups:
  - name: kafka_alerts
    interval: 30s
    rules:
      - alert: UnderReplicatedPartitions
        expr: kafka_server_replicamanager_underreplicatedpartitions > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Kafka has under-replicated partitions"
          description: "{{ $value }} partitions are under-replicated on {{ $labels.instance }}"

      - alert: OfflinePartitions
        expr: kafka_controller_kafkacontroller_offlinepartitionscount > 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Kafka has offline partitions"

      - alert: ActiveControllerCount
        expr: sum(kafka_controller_kafkacontroller_activecontrollercount) != 1
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Invalid number of active controllers"

      - alert: HighConsumerLag
        expr: kafka_consumer_group_lag > 10000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High consumer lag detected"
          description: "{{ $labels.group }} has lag of {{ $value }}"

      - alert: LowISRCount
        expr: kafka_server_replicamanager_underminisrpartitioncount > 0
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Partitions below min ISR"
```

### Part D: Grafana Dashboard

**Key Panels:**
1. Broker Health
   - Under-replicated partitions
   - Offline partitions
   - Active controller count

2. Throughput
   - Messages in/sec
   - Bytes in/sec
   - Bytes out/sec

3. Latency
   - Producer request latency (p95, p99)
   - Consumer fetch latency (p95, p99)

4. Consumer Groups
   - Lag by consumer group
   - Lag by partition

5. System Metrics
   - CPU usage
   - Memory usage
   - Disk I/O
   - Network I/O

**Tasks:**
1. Import Kafka dashboard from Grafana.com
2. Customize panels for your use case
3. Create custom dashboard for application metrics
4. Set up alert notifications (Slack, PagerDuty, email)

---

## Exercise 6: Disaster Recovery (60 minutes)

**Objective:** Implement backup and recovery strategies

### Part A: MirrorMaker 2.0 Setup

**mm2.properties:**
```properties
# Clusters
clusters = primary, secondary
primary.bootstrap.servers = primary-kafka:9092
secondary.bootstrap.servers = secondary-kafka:9092

# Replication flows
primary->secondary.enabled = true
primary->secondary.topics = .*
secondary->primary.enabled = false

# Replication settings
replication.factor = 3
sync.topic.acls.enabled = true
sync.topic.configs.enabled = true

# Performance
tasks.max = 4
replication.policy.class = org.apache.kafka.connect.mirror.DefaultReplicationPolicy

# Offset sync
offset-syncs.topic.replication.factor = 3
checkpoints.topic.replication.factor = 3
```

**Start MirrorMaker:**
```bash
connect-mirror-maker.sh mm2.properties
```

### Part B: Backup Topic Data

**Backup to S3:**
```bash
# Using Kafka Connect S3 Sink
{
  "name": "backup-connector",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "topics": "critical-topic",
    "s3.bucket.name": "kafka-backups",
    "flush.size": "10000",
    "rotate.interval.ms": "3600000",
    "storage.class": "io.confluent.connect.s3.storage.S3Storage",
    "format.class": "io.confluent.connect.s3.format.parquet.ParquetFormat",
    "partitioner.class": "io.confluent.connect.storage.partitioner.DailyPartitioner"
  }
}
```

### Part C: Test Failover

**Scenario:** Primary datacenter fails

**Tasks:**
1. Stop primary cluster
2. Promote secondary to primary
3. Update clients to point to secondary
4. Verify no data loss
5. Measure RPO (Recovery Point Objective) and RTO (Recovery Time Objective)

**Failover Script:**
```bash
#!/bin/bash

# Stop replication
systemctl stop mm2

# Update DNS or load balancer
# Point clients to secondary cluster

# Verify data integrity
kafka-consumer-groups.sh --bootstrap-server secondary:9092 --list

# Check for lag
kafka-consumer-groups.sh --bootstrap-server secondary:9092 \
  --describe --all-groups

# Resume operations
echo "Failover complete"
```

---

## Exercise 7: Capacity Planning (45 minutes)

**Objective:** Size Kafka cluster for production

### Scenario

**Requirements:**
- Peak throughput: 500 MB/sec
- Average message size: 5 KB
- Retention: 7 days
- Replication factor: 3
- Number of topics: 100
- Average partitions per topic: 12

### Calculate

**1. Messages per second:**
```
500 MB/sec ÷ 5 KB = 100,000 messages/sec
```

**2. Total partitions:**
```
100 topics × 12 partitions = 1,200 partitions
```

**3. Daily data volume:**
```
500 MB/sec × 86,400 sec/day = 43,200 GB/day ≈ 42.2 TB/day
```

**4. Total storage (7 days, RF=3):**
```
42.2 TB/day × 7 days × 3 RF = 886.2 TB ≈ 900 TB
```

**5. Brokers needed (storage):**
```
Assuming 10 TB per broker: 900 TB ÷ 10 TB = 90 brokers
(Impractical - use tiered storage or shorter retention)
```

**6. Brokers needed (throughput):**
```
Assuming 100 MB/sec per broker:
Writes: 500 MB/sec ÷ 100 MB/sec = 5 brokers
Reads: (assume 2x writes) 1000 MB/sec ÷ 100 MB/sec = 10 brokers
```

**7. Recommended broker count:**
```
max(storage requirement, throughput requirement) with 30% headroom
= 10 brokers × 1.3 = 13 brokers
```

**8. Memory per broker:**
```
Page cache = message.max.bytes × num.replica.fetchers × num.partitions
= 1 MB × 4 × (1200/13) ≈ 370 MB per broker

Heap = 6-8 GB for broker
Total RAM = Heap + Page Cache + OS = 32 GB
```

**9. Network bandwidth:**
```
Per broker peak: (500 MB/sec × 3 RF) / 13 = 115 MB/sec
Recommended: 1 Gbps or higher
```

### Tasks

1. Calculate sizing for your requirements
2. Create sizing spreadsheet
3. Consider cost optimization strategies
4. Plan for growth (2x, 5x)

---

## Challenge Exercise: Production Readiness Checklist (120 minutes)

**Objective:** Prepare Kafka cluster for production

### Checklist

#### Security
- [ ] SSL/TLS encryption enabled
- [ ] SASL authentication configured
- [ ] ACLs defined for all topics
- [ ] Secrets management (Vault/AWS Secrets Manager)
- [ ] Network segmentation

#### Reliability
- [ ] Replication factor ≥ 3
- [ ] min.insync.replicas = 2
- [ ] Rack awareness configured
- [ ] Unclean leader election disabled
- [ ] Producer idempotence enabled
- [ ] Consumer group commit strategy defined

#### Performance
- [ ] OS tuning applied
- [ ] Broker tuning configured
- [ ] Partition count optimized
- [ ] Compression enabled
- [ ] Batch size tuned

#### Monitoring
- [ ] JMX metrics exported
- [ ] Prometheus scraping configured
- [ ] Grafana dashboards created
- [ ] Alert rules defined
- [ ] Runbooks documented

#### Operations
- [ ] Backup strategy implemented
- [ ] Disaster recovery tested
- [ ] Capacity planning completed
- [ ] Upgrade procedure documented
- [ ] Troubleshooting guide created

#### Testing
- [ ] Load testing completed
- [ ] Failover testing done
- [ ] Security testing performed
- [ ] Chaos engineering scenarios tested

**Deliverables:**
1. Completed checklist
2. Configuration files
3. Monitoring dashboards
4. Runbooks
5. Test reports

---

## Solutions & Discussion

### Exercise 4: Expected Performance Results

```
Configuration       | Throughput     | Latency (p99) | Notes
--------------------|----------------|---------------|-------
Baseline            | 50K msg/sec    | 100 ms        | Default settings
OS tuning           | 70K msg/sec    | 80 ms         | Better I/O
Broker tuning       | 100K msg/sec   | 60 ms         | More threads
Producer HT         | 150K msg/sec   | 50 ms         | Batching
Producer LL         | 80K msg/sec    | 10 ms         | No batching
Consumer tuning     | 120K msg/sec   | 40 ms         | Larger fetch
All combined        | 200K msg/sec   | 40 ms         | Optimal
```

### Exercise 6: RTO/RPO Analysis

**Active-Passive:**
- RPO: 1-5 minutes (replication lag)
- RTO: 5-15 minutes (manual failover)

**Active-Active:**
- RPO: Near-zero (both clusters active)
- RTO: 0 (automatic client failover)

### Best Practices Summary

1. **Security first**: Always enable encryption and authentication
2. **Monitor everything**: Metrics, logs, traces
3. **Test failures**: Regular DR drills
4. **Document thoroughly**: Runbooks for all scenarios
5. **Automate operations**: Reduce human error
6. **Plan capacity**: Stay ahead of growth
7. **Tune incrementally**: Measure before and after
8. **Keep it simple**: Complexity is the enemy of reliability

---

**Time to Complete:** 7-8 hours

**[← Previous: Module 7 Exercises](module-07-exercises.md)** | **[Next: Module 9 Exercises →](module-09-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
