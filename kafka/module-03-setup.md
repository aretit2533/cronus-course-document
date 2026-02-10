# Module 3: Setting Up Kafka

## Overview
This module provides hands-on guidance for setting up Apache Kafka in various environments. You'll learn how to install Kafka locally, run it with Docker, configure brokers, and use essential CLI tools to interact with your Kafka cluster.

**Duration:** 2 hours

## Learning Objectives
By the end of this module, you will be able to:
- Install Kafka locally on your machine
- Run Kafka using Docker and Docker Compose
- Configure Kafka brokers for different scenarios
- Start and stop Kafka services
- Use CLI tools for basic operations
- Explore Kafka UI tools for management
- Verify Kafka installation and connectivity

## Table of Contents
1. [Installation Options](#installation-options)
2. [Installing Kafka Locally](#installing-kafka-locally)
3. [Running Kafka with Docker](#running-kafka-with-docker)
4. [Kafka Configuration Files](#kafka-configuration-files)
5. [Starting and Stopping Kafka](#starting-and-stopping-kafka)
6. [Basic CLI Tools](#basic-cli-tools)
7. [Kafka UI Tools](#kafka-ui-tools)
8. [Verification and Troubleshooting](#verification-and-troubleshooting)
9. [Summary](#summary)

---

## Installation Options

### Option 1: Local Installation
**Best for:** Learning, development, full control

**Pros:**
- Complete control over configuration
- Direct access to logs and data
- No containerization overhead

**Cons:**
- Manual installation and configuration
- Platform-specific setup
- Cleanup required

### Option 2: Docker
**Best for:** Quick setup, consistent environments, isolation

**Pros:**
- Quick setup (minutes)
- Consistent across platforms
- Easy cleanup (remove containers)
- Multiple versions side-by-side

**Cons:**
- Requires Docker installed
- Slight performance overhead
- Networking complexity

### Option 3: Cloud (Managed Services)
**Best for:** Production, managed infrastructure

**Options:**
- **Confluent Cloud**
- **Amazon MSK** (Managed Streaming for Kafka)
- **Azure Event Hubs** (Kafka-compatible)
- **Aiven for Apache Kafka**

**Pros:**
- No infrastructure management
- High availability built-in
- Automatic scaling and updates

**Cons:**
- Cost (pay per usage)
- Less control
- Vendor lock-in

---

## Installing Kafka Locally

### Prerequisites

- **Java 11 or later** (Kafka is written in Scala/Java)
- **Command-line access** (Terminal/PowerShell)
- **At least 4GB RAM** (recommended)
- **5-10GB disk space**

### Step 1: Install Java

**Check if Java is installed:**
```bash
java -version
```

**Expected output:**
```
openjdk version "11.0.20" 2023-07-18
OpenJDK Runtime Environment (build 11.0.20+8)
OpenJDK 64-Bit Server VM (build 11.0.20+8, mixed mode)
```

**Install Java (if needed):**

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

**macOS (Homebrew):**
```bash
brew install openjdk@11
```

**Windows:**
Download from [AdoptOpenJDK](https://adoptopenjdk.net/)

### Step 2: Download Kafka

```bash
# Download Kafka 3.6.1 (latest stable as of Feb 2026)
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz

# Extract
tar -xzf kafka_2.13-3.6.1.tgz

# Move to /opt (optional, recommended)
sudo mv kafka_2.13-3.6.1 /opt/kafka
cd /opt/kafka
```

**Directory structure:**
```
/opt/kafka/
├── bin/                  # Scripts and CLI tools
├── config/               # Configuration files
├── libs/                 # Kafka libraries
├── licenses/             # License files
├── logs/                 # Log files (created at runtime)
└── site-docs/            # Documentation
```

### Step 3: Configure Environment Variables

**Add to ~/.bashrc or ~/.zshrc:**
```bash
export KAFKA_HOME=/opt/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```

**Apply changes:**
```bash
source ~/.bashrc  # or ~/.zshrc
```

### Step 4: Start Kafka (KRaft Mode)

**Generate Cluster ID:**
```bash
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
echo $KAFKA_CLUSTER_ID
```

**Format Storage:**
```bash
bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
```

**Start Kafka:**
```bash
bin/kafka-server-start.sh config/kraft/server.properties
```

**Expected output:**
```
[2026-02-10 10:30:00,123] INFO Kafka Server started (kafka.server.KafkaServer)
[2026-02-10 10:30:00,124] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
```

### Step 5: Verify Installation

**In a new terminal:**
```bash
# Create a test topic
bin/kafka-topics.sh --create \
  --topic test \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1

# List topics
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

---

## Running Kafka with Docker

### Prerequisites

- **Docker** installed ([Get Docker](https://docs.docker.com/get-docker/))
- **Docker Compose** (included with Docker Desktop)

### Option 1: Single Docker Command

**Run Kafka container:**
```bash
docker run -d \
  --name kafka \
  -p 9092:9092 \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_LISTENERS=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@localhost:9093 \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1 \
  -e KAFKA_LOG_DIRS=/tmp/kraft-combined-logs \
  -e CLUSTER_ID=MkU3OEVBNTcwNTJENDM2Qk \
  apache/kafka:latest
```

### Option 2: Docker Compose (Recommended)

**Create `docker-compose.yml`:**
```yaml
version: '3.8'

services:
  kafka:
    image: apache/kafka:latest
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_LOG_DIRS: /tmp/kraft-combined-logs
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk
    volumes:
      - kafka-data:/tmp/kraft-combined-logs

volumes:
  kafka-data:
```

**Start Kafka:**
```bash
docker-compose up -d
```

**View logs:**
```bash
docker-compose logs -f kafka
```

**Stop Kafka:**
```bash
docker-compose down
```

**Remove volumes (reset):**
```bash
docker-compose down -v
```

### Option 3: Multi-Broker Cluster with Docker Compose

**Create `docker-compose-cluster.yml`:**
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
    volumes:
      - kafka-1-data:/tmp/kraft-logs

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
    volumes:
      - kafka-2-data:/tmp/kraft-logs

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
    volumes:
      - kafka-3-data:/tmp/kraft-logs

volumes:
  kafka-1-data:
  kafka-2-data:
  kafka-3-data:
```

**Start cluster:**
```bash
docker-compose -f docker-compose-cluster.yml up -d
```

---

## Kafka Configuration Files

### Key Configuration Files

#### 1. **server.properties** (Broker Configuration)

Location: `config/server.properties`

**Essential Settings:**

```properties
# Broker ID (unique per broker)
broker.id=0

# Listeners (network interfaces to bind to)
listeners=PLAINTEXT://localhost:9092

# Advertised listeners (what clients use to connect)
advertised.listeners=PLAINTEXT://localhost:9092

# Log directories (where Kafka stores data)
log.dirs=/var/lib/kafka/data

# Number of partitions for auto-created topics
num.partitions=3

# Default replication factor
default.replication.factor=1

# Minimum in-sync replicas
min.insync.replicas=1

# Log retention hours (how long to keep data)
log.retention.hours=168  # 7 days

# Log retention bytes (max size before deletion)
log.retention.bytes=1073741824  # 1GB

# Segment size
log.segment.bytes=1073741824  # 1GB

# Segment time (time before rolling segment)
log.segment.ms=604800000  # 7 days
```

#### 2. **kraft/server.properties** (KRaft Mode)

**Additional KRaft Settings:**

```properties
# Process roles (broker, controller, or both)
process.roles=broker,controller

# Node ID
node.id=1

# Controller quorum voters
controller.quorum.voters=1@localhost:9093

# Listeners
listeners=PLAINTEXT://:9092,CONTROLLER://:9093

# Controller listener name
controller.listener.names=CONTROLLER

# Cluster ID (generated with kafka-storage.sh random-uuid)
cluster.id=MkU3OEVBNTcwNTJENDM2Qk
```

### Configuration Categories

#### Performance Settings
```properties
# Number of network threads handling requests
num.network.threads=3

# Number of I/O threads for disk operations
num.io.threads=8

# Socket send buffer
socket.send.buffer.bytes=102400

# Socket receive buffer
socket.receive.buffer.bytes=102400

# Maximum request size
socket.request.max.bytes=104857600
```

#### Replication Settings
```properties
# Default replication factor
default.replication.factor=3

# Minimum in-sync replicas
min.insync.replicas=2

# Replica lag time
replica.lag.time.max.ms=30000

# Replica fetch max bytes
replica.fetch.max.bytes=1048576
```

#### Retention Settings
```properties
# Time-based retention (7 days)
log.retention.hours=168

# Size-based retention (1GB)
log.retention.bytes=1073741824

# Cleanup policy (delete or compact)
log.cleanup.policy=delete

# Segment size
log.segment.bytes=1073741824

# Segment time
log.segment.ms=604800000
```

---

## Starting and Stopping Kafka

### Local Installation

#### Start Kafka (KRaft Mode)
```bash
# Terminal 1: Start Kafka
bin/kafka-server-start.sh config/kraft/server.properties
```

#### Start in Background
```bash
bin/kafka-server-start.sh -daemon config/kraft/server.properties
```

#### Check if Running
```bash
ps aux | grep kafka
```

#### Stop Kafka
```bash
bin/kafka-server-stop.sh
```

#### Force Stop (if process hangs)
```bash
pkill -9 -f kafka
```

### Docker

#### Start
```bash
docker-compose up -d
```

#### Stop
```bash
docker-compose down
```

#### Restart
```bash
docker-compose restart
```

#### View Logs
```bash
docker-compose logs -f kafka
```

#### Access Kafka Container
```bash
docker exec -it kafka bash
```

---

## Basic CLI Tools

All CLI tools are located in the `bin/` directory.

### 1. kafka-topics.sh

**Create a topic:**
```bash
kafka-topics.sh --create \
  --topic my-topic \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1
```

**List topics:**
```bash
kafka-topics.sh --list --bootstrap-server localhost:9092
```

**Describe a topic:**
```bash
kafka-topics.sh --describe \
  --topic my-topic \
  --bootstrap-server localhost:9092
```

**Output:**
```
Topic: my-topic	TopicId: abc123	PartitionCount: 3	ReplicationFactor: 1
	Topic: my-topic	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: my-topic	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
	Topic: my-topic	Partition: 2	Leader: 1	Replicas: 1	Isr: 1
```

**Delete a topic:**
```bash
kafka-topics.sh --delete \
  --topic my-topic \
  --bootstrap-server localhost:9092
```

**Alter topic (increase partitions):**
```bash
kafka-topics.sh --alter \
  --topic my-topic \
  --partitions 5 \
  --bootstrap-server localhost:9092
```

### 2. kafka-console-producer.sh

**Produce messages:**
```bash
kafka-console-producer.sh \
  --topic my-topic \
  --bootstrap-server localhost:9092
```

**Type messages (each line is a message):**
```
> Hello Kafka
> This is message 2
> Another message
```

**Produce with keys:**
```bash
kafka-console-producer.sh \
  --topic my-topic \
  --property "parse.key=true" \
  --property "key.separator=:" \
  --bootstrap-server localhost:9092
```

**Type messages:**
```
> user1:Hello from user1
> user2:Hello from user2
> user1:Another message from user1
```

### 3. kafka-console-consumer.sh

**Consume messages (from now on):**
```bash
kafka-console-consumer.sh \
  --topic my-topic \
  --bootstrap-server localhost:9092
```

**Consume from beginning:**
```bash
kafka-console-consumer.sh \
  --topic my-topic \
  --from-beginning \
  --bootstrap-server localhost:9092
```

**Consume with keys:**
```bash
kafka-console-consumer.sh \
  --topic my-topic \
  --from-beginning \
  --property print.key=true \
  --property key.separator=":" \
  --bootstrap-server localhost:9092
```

**Consume in a consumer group:**
```bash
kafka-console-consumer.sh \
  --topic my-topic \
  --group my-consumer-group \
  --bootstrap-server localhost:9092
```

### 4. kafka-consumer-groups.sh

**List consumer groups:**
```bash
kafka-consumer-groups.sh --list \
  --bootstrap-server localhost:9092
```

**Describe consumer group:**
```bash
kafka-consumer-groups.sh --describe \
  --group my-consumer-group \
  --bootstrap-server localhost:9092
```

**Output:**
```
GROUP             TOPIC      PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
my-consumer-group my-topic   0          100             100             0
my-consumer-group my-topic   1          90              95              5
my-consumer-group my-topic   2          80              80              0
```

**Reset offsets:**
```bash
kafka-consumer-groups.sh --reset-offsets \
  --group my-consumer-group \
  --topic my-topic \
  --to-earliest \
  --execute \
  --bootstrap-server localhost:9092
```

### 5. kafka-configs.sh

**Describe broker config:**
```bash
kafka-configs.sh --describe \
  --broker 0 \
  --bootstrap-server localhost:9092
```

**Alter topic config:**
```bash
kafka-configs.sh --alter \
  --topic my-topic \
  --add-config retention.ms=86400000 \
  --bootstrap-server localhost:9092
```

### 6. kafka-broker-api-versions.sh

**Check API versions:**
```bash
kafka-broker-api-versions.sh \
  --bootstrap-server localhost:9092
```

---

## Kafka UI Tools

### 1. Kafka UI (Open Source)

**GitHub:** [provectus/kafka-ui](https://github.com/provectus/kafka-ui)

**Run with Docker:**
```bash
docker run -d \
  --name kafka-ui \
  -p 8080:8080 \
  -e KAFKA_CLUSTERS_0_NAME=local \
  -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=host.docker.internal:9092 \
  provectuslabs/kafka-ui:latest
```

**Access:** http://localhost:8080

**Features:**
- Browse topics and messages
- View consumer groups and lag
- Manage topics and configurations
- Monitor cluster health
- Multi-cluster support

### 2. Conduktor (Commercial)

**Website:** [conduktor.io](https://www.conduktor.io/)

**Features:**
- Desktop application (Windows, Mac, Linux)
- Topic and message management
- Schema registry integration
- Consumer group monitoring
- Kafka Connect management
- ksqlDB support
- Free tier available

### 3. Offset Explorer (formerly Kafka Tool)

**Website:** [kafkatool.com](https://www.kafkatool.com/)

**Features:**
- Desktop application
- Browse topics and messages
- View consumer offsets
- Topic configuration
- Simple and free

### 4. AKHQ (formerly KafkaHQ)

**GitHub:** [tchiotludo/akhq](https://github.com/tchiotludo/akhq)

**Features:**
- Web-based UI
- Topic and message management
- Consumer group monitoring
- Schema registry support
- LDAP/OAuth authentication

---

## Verification and Troubleshooting

### Verification Checklist

✅ **Check Java version:**
```bash
java -version  # Should be 11 or higher
```

✅ **Check Kafka is running:**
```bash
ps aux | grep kafka  # Local
docker ps  # Docker
```

✅ **Test connectivity:**
```bash
telnet localhost 9092
```

✅ **Create test topic:**
```bash
kafka-topics.sh --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

✅ **Produce test message:**
```bash
echo "test message" | kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
```

✅ **Consume test message:**
```bash
kafka-console-consumer.sh --topic test --from-beginning --bootstrap-server localhost:9092 --max-messages 1
```

### Common Issues

#### Issue 1: Connection Refused

**Error:**
```
org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
```

**Solutions:**
- Check Kafka is running: `ps aux | grep kafka`
- Verify port is open: `netstat -an | grep 9092`
- Check firewall rules
- Verify `advertised.listeners` configuration

#### Issue 2: Insufficient Java Version

**Error:**
```
Exception in thread "main" java.lang.UnsupportedClassVersionError
```

**Solution:**
- Install Java 11 or later
- Set JAVA_HOME correctly

#### Issue 3: Port Already in Use

**Error:**
```
java.net.BindException: Address already in use
```

**Solution:**
```bash
# Find process using port 9092
lsof -i :9092
# Kill process
kill -9 <PID>
```

#### Issue 4: Out of Memory

**Error:**
```
java.lang.OutOfMemoryError: Java heap space
```

**Solution:**
Set heap size in `kafka-server-start.sh` or environment:
```bash
export KAFKA_HEAP_OPTS="-Xmx512M -Xms512M"
```

#### Issue 5: No Space Left on Device

**Error:**
```
java.io.IOException: No space left on device
```

**Solution:**
- Clean up old logs
- Increase disk space
- Reduce retention settings

### Useful Diagnostic Commands

**Check Kafka logs:**
```bash
# Local
tail -f /opt/kafka/logs/server.log

# Docker
docker logs -f kafka
```

**Check disk usage:**
```bash
du -sh /var/lib/kafka/data/*
```

**Check network connections:**
```bash
netstat -an | grep 9092
```

**Check broker metadata:**
```bash
kafka-metadata-shell.sh --snapshot /tmp/kraft-combined-logs/__cluster_metadata-0
```

---

## Summary

In this module, you learned:

1. **Installation Options**: Local, Docker, and cloud-based Kafka setups

2. **Local Installation**: How to install Kafka with Java and configure KRaft mode

3. **Docker Setup**: Running Kafka with Docker and Docker Compose for quick setup

4. **Configuration**: Understanding key Kafka configuration properties

5. **Starting/Stopping**: Managing Kafka services in different environments

6. **CLI Tools**: Using kafka-topics, kafka-console-producer, kafka-console-consumer, and more

7. **UI Tools**: Exploring Kafka UI, Conduktor, and other management interfaces

8. **Troubleshooting**: Common issues and how to resolve them

---

## Key Takeaways

✅ **Docker is quickest** - Get started in minutes with Docker Compose

✅ **KRaft simplifies setup** - No separate ZooKeeper installation needed

✅ **CLI tools are powerful** - Master them for production operations

✅ **UI tools aid learning** - Visual tools help understand Kafka concepts

✅ **Configuration matters** - Proper config prevents many issues

✅ **Start simple** - Single broker for learning, multi-broker for production

---

## What's Next?

Now that you have Kafka up and running, you're ready to build producers!

The next module will cover:
- Creating producers in Java, Python, and Node.js
- Message serialization
- Partitioning strategies
- Reliability configurations
- Error handling

**Continue to [Module 4: Kafka Producers](module-04-producers.md)**

---

## Additional Resources

- [Kafka Quickstart Guide](https://kafka.apache.org/quickstart)
- [Kafka Configuration Reference](https://kafka.apache.org/documentation/#configuration)
- [Docker Images for Kafka](https://hub.docker.com/r/apache/kafka)
- [KRaft Mode Documentation](https://kafka.apache.org/documentation/#kraft)
- [Kafka Operations Guide](https://kafka.apache.org/documentation/#operations)

---

**[📝 Practice Exercises](exercise/module-03-exercises.md)** | **[📚 Back to Course Home](README.md)**
