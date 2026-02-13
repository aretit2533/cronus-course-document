# Module 3: CustomServerKafka Architecture

## 📚 Learning Objectives

By the end of this module, you will be able to:

- Trace the `listen()` → `start()` → `bindEvents()` lifecycle
- Understand the role of the Kafka Admin client
- Explain static accessors and member assignment tracking

---

## 3.1 Startup sequence

At a high level:

1. `listen()` creates consumer client and producer client
2. `start()` connects admin, producer, consumer
3. `bindEvents()` subscribes to topics and starts `consumer.run()`

---

## 3.2 Admin client and topic visibility

The server uses Kafka Admin to:

- `listTopics()` (when monitoring is enabled)
- validate that the message patterns map to existing topics

This helps catch typos/missing topics early.

---

## 3.3 Static accessors

The strategy stores connected KafkaJS instances in static fields:

- `CustomServerKafka.getConsumer()`
- `CustomServerKafka.getProducer()`

Use these carefully: treat them as low-level building blocks.

---

## 3.4 Group join and member assignment

On `consumer.group_join`, the strategy stores the assignment:

- `CustomServerKafka.memberAssignment`

This is later used by the heap guardrail to pause/resume partitions.

---

## 🧭 Visual Flow (Mermaid)

```mermaid
flowchart TD
  L[listen()] --> C1[createClient()<br/>(consumer client)]
  L --> C2[createProducerClient()]
  L --> S[start()]
  S --> A[admin.connect()]
  S --> P[producer.connect()]
  S --> C[consumer.connect()]
  S --> B[bindEvents()]
  B --> T[retrieveTopics()]
  T --> Sub[subscribe(topics)]
  Sub --> Run[consumer.run(eachMessage)]
  Run --> Done[Ready]
```

---

## ✅ Summary

- Admin is used for topic visibility/monitoring.
- Member assignment is captured for safe pause/resume.

Next: [Module 4: Resilience & Recovery](module-04-resilience-recovery.md)
