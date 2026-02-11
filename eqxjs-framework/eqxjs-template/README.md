# EQXJS Template Message REST API Course

> Build a NestJS domain service that exposes REST APIs and publishes/consumes Kafka events using EQXJS.

---

## About This Course

This short course explains how the `esb-dos-template-message-restapi` project is structured, configured, and extended. You will follow the REST and Kafka flows end-to-end and learn where to plug in new features.

**Course Level**: Beginner
**Estimated Duration**: 6-8 hours
**Prerequisites**: Basic TypeScript, NestJS fundamentals, and Kafka basics

---

## Learning Objectives

By the end of this course, you will be able to:

- Understand the project architecture and module layout
- Configure environments and service dependencies
- Trace the REST request flow through managers and services
- Trace Kafka consumer/producer flows end-to-end
- Extend repositories, DTOs, and topic configuration safely

---

## Quick Start

1. Install dependencies:

```bash
npm install
```

2. Choose a configuration file by setting `ZONE`:

```bash
# Example
export ZONE=local
```

3. Set required runtime variables (example values shown):

```bash
export BROKERS=localhost:9092
export DATABASE_URL=mongodb://localhost:27017
export DATABASE_NAME=example
export DATABASE_AUTH_TYPE=none
export DATABASE_CERT=
export RETRYKAFKACOUNTMAX=3
```

4. Start the service:

```bash
npm run start:local
```

---

## Course Modules

| Module | Topic                         | Content                                      | Exercises                                    | Duration  |
| ------ | ----------------------------- | -------------------------------------------- | -------------------------------------------- | --------- |
| 1      | Introduction and Architecture | [Module 1](module-01-introduction.md)        | [Exercises](exercise/module-01-exercises.md) | 1.5 hours |
| 2      | Setup and Configuration       | [Module 2](module-02-setup-configuration.md) | [Exercises](exercise/module-02-exercises.md) | 2 hours   |
| 3      | REST API Flow                 | [Module 3](module-03-rest-api-flow.md)       | [Exercises](exercise/module-03-exercises.md) | 1.5 hours |
| 4      | Kafka and Data Flow           | [Module 4](module-04-kafka-data-flow.md)     | [Exercises](exercise/module-04-exercises.md) | 1.5 hours |

---

## Course Outline

See the full outline in [course-outline.md](course-outline.md).

---

## Exercises

All exercises are in [exercise/README.md](exercise/README.md).
