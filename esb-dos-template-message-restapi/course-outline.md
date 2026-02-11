# EQXJS Template Message REST API Course Outline

---

## Course Modules

| Module   | Topic                         | Content                                          | Exercises                                    |
| -------- | ----------------------------- | ------------------------------------------------ | -------------------------------------------- |
| Module 1 | Introduction and Architecture | [View Content](module-01-introduction.md)        | [Exercises](exercise/module-01-exercises.md) |
| Module 2 | Setup and Configuration       | [View Content](module-02-setup-configuration.md) | [Exercises](exercise/module-02-exercises.md) |
| Module 3 | REST API Flow                 | [View Content](module-03-rest-api-flow.md)       | [Exercises](exercise/module-03-exercises.md) |
| Module 4 | Kafka and Data Flow           | [View Content](module-04-kafka-data-flow.md)     | [Exercises](exercise/module-04-exercises.md) |

---

## Module 1: Introduction and Architecture

- What the template provides (REST + Kafka + MongoDB)
- Key dependencies (EQXJS Stub, Custom Kafka Server)
- High-level module layout
- Request and event flow overview

---

## Module 2: Setup and Configuration

- Environment selection with `ZONE`
- YAML configuration structure
- Kafka configuration and credentials
- MongoDB connection setup
- Runtime environment variables

---

## Module 3: REST API Flow

- `RestController` and `HttpEntryPoint`
- Manager pattern for request handling
- Service orchestration (API call, DB access, producer)
- Logging and message context usage

---

## Module 4: Kafka and Data Flow

- Kafka consumer entry points
- DTO mapping and sanitization
- Producer retries and logging
- Repository and database access patterns

---

## Exercise Index

- [All Exercises](exercise/README.md)
