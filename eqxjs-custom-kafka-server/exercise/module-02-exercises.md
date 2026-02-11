# Module 2 Exercises: Setup & NestJS Integration

## 🎯 Goals

- Boot a Nest microservice with `CustomServerKafka`
- Register at least one `@MessagePattern` topic

## ✅ Exercise 1: Create a demo service

1. Create a new Nest project (or reuse an existing one):

```bash
nest new demo-custom-kafka
cd demo-custom-kafka
npm install @nestjs/microservices kafkajs @eqxjs/custom-kafka-server
```

2. Add a controller with `@MessagePattern('orders.created')`.

3. In `main.ts`, create a microservice using:

- `strategy: new CustomServerKafka(consumerOptions)`

4. Run the service and confirm it connects without throwing.

## ✅ Exercise 2: Produce a message

Use the KafkaJS producer returned by:

- `CustomServerKafka.getProducer()`

Send one message to `orders.created`.

## 🧩 Challenge

Add separate producer options (different clientId) by passing the second parameter to the constructor.

## ✔️ Verification

- Service starts successfully.
- A message is produced to `orders.created`.
