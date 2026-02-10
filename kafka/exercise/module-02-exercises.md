# Module 2 Exercises: Kafka Architecture and Core Concepts

## Exercise 1: Understanding Kafka Cluster Architecture (20 minutes)

**Objective:** Understand how Kafka brokers work together

**Tasks:**
1. Draw a Kafka cluster with 3 brokers
2. Show a topic with:
   - Name: `orders`
   - 6 partitions
   - Replication factor: 3
3. Mark which broker is the leader for each partition
4. Show where replicas are placed

**Questions:**
1. If Broker 2 fails, what happens?
2. How does Kafka ensure no data loss?
3. What is the maximum number of broker failures the cluster can tolerate?

**Verification:**
- Each partition should have 1 leader and 2 followers
- Leaders should be distributed across brokers
- Answer: Can tolerate 2 broker failures (RF=3, need 1 replica available)

---

## Exercise 2: Partition Assignment Simulation (30 minutes)

**Objective:** Understand how partitions are assigned to consumers

**Scenario:**
- Topic: `events` with 12 partitions (P0-P11)
- Consumer group: `analytics-group`

**Tasks:**

Simulate partition assignment for these cases:

**Case 1:** 3 consumers in the group
```
Consumer A gets: _____
Consumer B gets: _____
Consumer C gets: _____
```

**Case 2:** 4 consumers in the group
```
Consumer A gets: _____
Consumer B gets: _____
Consumer C gets: _____
Consumer D gets: _____
```

**Case 3:** 6 consumers in the group
```
Consumer A-F get: _____
```

**Case 4:** 15 consumers in the group
```
What happens? _____
```

**Questions:**
1. Which assignment is most balanced?
2. How many consumers would you recommend for optimal performance?
3. What happens when you add a 7th consumer?

---

## Exercise 3: Offset Management Deep Dive (25 minutes)

**Objective:** Master offset tracking and management

**Scenario Setup:**
```
Topic: user-events (3 partitions)
Consumer Group: processor-group
```

**Initial State:**
```
Partition 0: Offset 100 (last committed)
Partition 1: Offset 250 (last committed)
Partition 2: Offset 175 (last committed)
```

**Tasks:**

1. Consumer processes these records:
   ```
   P0: 101-110 (10 records)
   P1: 251-260 (10 records)
   P2: 176-185 (10 records)
   ```
   If auto-commit is enabled with interval 5s, what are the committed offsets?

2. Consumer crashes after processing records 101-105 in P0, but before commit. After restart, which records are reprocessed?

3. Design an "at-least-once" commit strategy (pseudocode)

4. Design an "exactly-once" commit strategy (pseudocode)

**Solution Template:**
```typescript
// At-least-once
while (true) {
    const records = await consumer.poll();
    for (const record of records) {
        await process(record);  // Step 1: Process
    }
    await consumer.commitSync();  // Step 2: Commit
}

// Exactly-once (your solution)
// ...
```

---

## Exercise 4: Replication and ISR Scenarios (30 minutes)

**Objective:** Understand replication and In-Sync Replicas

**Given:**
- Topic: `transactions`
- 1 partition
- Replication factor: 3
- min.insync.replicas: 2
- acks: all

**Scenario 1: Normal Operation**
```
Leader: Broker 1 (offset 1000)
Follower: Broker 2 (offset 1000)
Follower: Broker 3 (offset 1000)
ISR: [1, 2, 3]
```

Producer sends a message with `acks=all`. What happens?

**Scenario 2: Follower Lag**
```
Leader: Broker 1 (offset 1500)
Follower: Broker 2 (offset 1500)
Follower: Broker 3 (offset 1400)  // Lagging
ISR: [1, 2]
```

Producer sends with `acks=all`. What happens?

**Scenario 3: Leader Failure**
```
Leader: Broker 1 (OFFLINE)
Follower: Broker 2 (offset 2000, in-sync)
Follower: Broker 3 (offset 2000, in-sync)
ISR: [2, 3]
```

What happens next? Who becomes the new leader?

**Scenario 4: Not Enough ISR**
```
Leader: Broker 1 (offset 3000)
Follower: Broker 2 (OFFLINE)
Follower: Broker 3 (OFFLINE)
ISR: [1]
min.insync.replicas: 2
```

Producer sends with `acks=all`. What happens?

**Answer these:**
1. What does ISR mean?
2. Why is min.insync.replicas important?
3. When would you set min.insync.replicas to 1 vs 2?

---

## Exercise 5: Log Segments and Retention (20 minutes)

**Objective:** Understand how Kafka stores data on disk

**Tasks:**

1. Draw the log segment structure for a partition:
   ```
   /kafka-logs/my-topic-0/
     00000000000000000000.log
     00000000000000000000.index
     00000000000000001000.log
     00000000000000001000.index
     00000000000000002000.log
     00000000000000002000.index (active)
   ```

2. Configure retention for these requirements:
   - **Scenario A:** IoT sensor data, keep 3 days
   - **Scenario B:** Financial transactions, keep 7 years
   - **Scenario C:** User sessions, keep 100 GB max
   - **Scenario D:** Cache/config data, keep latest value only

**Configuration Required:**
```properties
# Scenario A
retention.ms=?
retention.bytes=?

# Scenario B
retention.ms=?

# Scenario C
retention.bytes=?

# Scenario D
cleanup.policy=?
```

3. Calculate storage requirements:
   - 1000 messages/sec
   - Average message size: 2 KB
   - Retention: 7 days
   - Replication factor: 3
   
   Total storage needed: _____ GB

---

## Exercise 6: Controller and Metadata Management (15 minutes)

**Objective:** Understand the role of the controller

**Questions:**

1. What is the Kafka controller?
2. How is the controller elected?
3. List 3 responsibilities of the controller:
   - a. _____
   - b. _____
   - c. _____

4. What happens if the controller fails?

5. In KRaft mode (without ZooKeeper), how does metadata management differ?

**Research Task:**
Compare ZooKeeper vs KRaft:
| Aspect | ZooKeeper Mode | KRaft Mode |
|--------|---------------|------------|
| Metadata storage | ? | ? |
| Setup complexity | ? | ? |
| Performance | ? | ? |
| Recommended for? | ? | ? |

---

## Challenge Exercise: Design a Multi-Datacenter Kafka Deployment (60 minutes)

**Objective:** Design a resilient, multi-datacenter Kafka architecture

**Requirements:**
- 2 datacenters (US-East, US-West)
- Each datacenter should handle local traffic
- Data should replicate across datacenters
- Survive entire datacenter failure
- Optimize for low latency

**Tasks:**

1. **Topology Design**
   - How many brokers in each DC?
   - How will you replicate data across DCs?
   - Will you use one cluster or multiple?

2. **Replication Strategy**
   - Should you set RF=6 (3 in each DC)?
   - Or use separate clusters with MirrorMaker?
   - Trade-offs?

3. **Producer Configuration**
   - Where should producers send data?
   - What `acks` setting?
   - Should producers be DC-aware?

4. **Consumer Configuration**
   - Can consumers read from any DC?
   - How to minimize cross-DC traffic?

5. **Failover Plan**
   - US-East goes down - what happens?
   - How long to recover?
   - Data loss scenario?

**Deliverable:**
- Architecture diagram
- Kafka configuration files
- Failover runbook (step-by-step)

---

## Solutions & Discussion

### Exercise 2: Partition Assignment
- Case 1 (3 consumers): Each gets 4 partitions (balanced)
- Case 2 (4 consumers): 3 consumers get 3 partitions, 1 consumer gets 3 partitions (balanced)
- Case 3 (6 consumers): Each gets 2 partitions (perfectly balanced)
- Case 4 (15 consumers): 12 consumers get 1 partition each, 3 sit idle (waste)

**Recommendation:** Number of consumers ≤ number of partitions for best utilization

### Exercise 4: ISR Scenarios
1. **Normal**: Message replicated to all 3 brokers, producer gets ACK
2. **Follower Lag**: Message replicated to Brokers 1 & 2 (ISR met), producer gets ACK
3. **Leader Failure**: Broker 2 or 3 becomes new leader (both in-sync)
4. **Not Enough ISR**: Producer request FAILS with `NotEnoughReplicasException`

### Exercise 5: Storage Calculation
```
1000 msg/sec × 2 KB × 86,400 sec/day × 7 days × 3 (RF) = 3,628,800,000 KB
≈ 3,629 GB ≈ 3.5 TB
```

Add 20% overhead: **~4.2 TB**

### Challenge Exercise: Multi-DC Considerations
**Option 1: Stretched Cluster** (One cluster across DCs)
- Pros: Simpler, exactly-once easier
- Cons: Higher latency, quorum across DCs

**Option 2: Active-Passive** (Primary + DR cluster)
- Pros: Lower latency in primary DC
- Cons: DR lag, complex failover

**Option 3: Active-Active** (2 independent clusters + MirrorMaker)
- Pros: Both DCs active, low latency
- Cons: Eventually consistent, complex

**Recommended:** Active-Active for high availability

---

## Hands-On Lab (Optional)

If you have Kafka running locally:

```bash
# Describe partition details
kafka-topics.sh --describe --topic user-events --bootstrap-server localhost:9092

# Check consumer group offsets
kafka-consumer-groups.sh --describe --group my-group --bootstrap-server localhost:9092

# Monitor under-replicated partitions
kafka-topics.sh --describe --under-replicated-partitions --bootstrap-server localhost:9092
```

---

**Time to Complete:** 3-4 hours

**[← Previous: Module 1 Exercises](module-01-exercises.md)** | **[Next: Module 3 Exercises →](module-03-exercises.md)** | **[📚 Back to Exercises Home](README.md)**
