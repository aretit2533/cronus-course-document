# EQXJS Custom Kafka Server Course Outline

## Course Overview

This course focuses on how to use `@eqxjs/custom-kafka-server` to run Kafka consumers/producers in NestJS microservices with stronger operational behavior:

- consumer crash recovery
- topic monitoring + validation
- memory guardrails via pause/resume

## Target Audience

- NestJS microservices developers
- Developers who operate Kafka consumers in production
- Platform teams providing a shared Kafka runtime

## Prerequisites

- NestJS microservices basics
- Kafka basics (topics/partitions/groups)
- Familiarity with environment variables

---

## Module 1: Introduction & Features

**Topics:**

- What `CustomServerKafka` is (extends `ServerKafka`)
- Why custom strategy vs `Transport.KAFKA`
- Feature map: recovery, topic monitor, heap guard, separate producer

**Learning Objectives:**

- Explain what problems the library solves
- Identify which features to enable/disable per environment

---

## Module 2: Setup & NestJS Integration

**Topics:**

- Installing the package and required deps
- Creating a Nest microservice with `strategy: new CustomServerKafka(...)`
- Options mapping: `client`, `consumer`, `producer`, `subscribe`, `run`
- Basic consumer + producer usage patterns

**Learning Objectives:**

- Boot a microservice using `CustomServerKafka`
- Know where to configure brokers, groupId, and subscription behavior

---

## Module 3: CustomServerKafka Architecture

**Topics:**

- Lifecycle: `listen()` → `start()` → `bindEvents()`
- Admin client usage
- Static accessors: `CustomServerKafka.getConsumer()` / `getProducer()`
- Group join event and `memberAssignment`

**Learning Objectives:**

- Trace the startup sequence
- Understand how topics are derived from message patterns

---

## Module 4: Resilience & Recovery

**Topics:**

- `consumer.crash` event handling
- `consumer.retry.restartOnFailure` behavior
- Recreate consumer flow + concurrency guard (`underRestartConsumer`)
- Producer recreation path

**Learning Objectives:**

- Explain what triggers a recreate
- Validate recovery with logs and simple fault injection

---

## Module 5: Topic Monitoring & Subscription

**Topics:**

- Topic retrieval sources: registered patterns vs admin listTopics
- Behavior when topics are missing
- Monitoring loop and interval
- Env toggles:
  - `MONITOR_TOPICS`
  - `KAFKA_MONITOR_INTERVAL`

**Learning Objectives:**

- Decide when to enable monitoring
- Tune interval and handle dynamic topic changes

---

## Module 6: Memory Guardrails & Production Tuning

**Topics:****

- Heap usage check: pause/resume
- Env toggles:
  - `KAFKA_HEAP_USED_SIZE_CHECK_ENABLED`
  - `KAFKA_HEAP_USED_SIZE_PERCENT`
  - `KAFKA_CONSUMER_PAUSE_TIME_MS`
- Operational guidance: logging, safe defaults, failure modes

**Learning Objectives:**

- Use pause/resume guardrails responsibly
- Understand tradeoffs (latency vs stability)

---

## Hands-On Exercises

See the full index: [exercise/README.md](exercise/README.md)
