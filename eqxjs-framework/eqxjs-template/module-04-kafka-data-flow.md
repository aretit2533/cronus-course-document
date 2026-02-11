# Module 4: Kafka and Data Flow

> Trace Kafka consumption and production, plus database access patterns.

---

## Learning Goals

- Understand how Kafka handlers are wired
- Learn how DTOs and pipes map payloads
- Follow producer retries and logging
- See how MongoDB access is structured

---

## 4.1 Kafka Consumer Entry Point

`EventConsumerController` uses:

- `@EntryPoint(topicConsume.exampleTopic)` for topic mapping
- `RemoveAtSymbolPipe` to normalize payload keys
- `ToObjectDecorator` to map payloads into `TopicConsumerEvent`

The topic name is loaded from YAML and exposed by `src/example/utils/utils-consume-produce.ts`.

---

## 4.2 Kafka Production

`EventProducerService`:

- Builds a KafkaJS `send` payload
- Logs producing actions with masking for sensitive fields
- Retries based on `RETRYKAFKACOUNTMAX`
- Uses `CustomServerKafka.getProducer()` for the shared producer

---

## 4.3 MongoDB Repository

`ExampleMogoRepository` wraps MongoDB access and adds structured logging:

- `findManyExample()` reads from the configured collection
- `createExample()` shows insert operations and write concern usage

The database provider lives in `src/database/database.module.ts` and injects a `MongoProperty` with `db` and `connection`.

---

## Knowledge Check

- Which config key controls the consumed topic name?
- Where is the producer retry limit read from?
- What provider token is used to inject MongoDB?

---

## Course Wrap-Up

You now have a full picture of REST, Kafka, and data flows. Continue with the exercises to practice extending the template.
