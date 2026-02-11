# Module 3 Exercises: Architecture

## 🎯 Goals

- Observe startup behavior and topic subscription
- Understand how topics are selected

## ✅ Exercise 1: Confirm topics come from patterns

1. Register two patterns:
   - `orders.created`
   - `orders.cancelled`

2. Start the service.

3. Confirm via logs that it attempts to subscribe to both topics.

## ✅ Exercise 2: Observe member assignment

1. Start two instances of the same service (same groupId).
2. Observe `consumer.group_join` logs.
3. Confirm that member assignment changes between instances.

## 🧩 Challenge

Write a small helper service that logs `CustomServerKafka.memberAssignment` every 10 seconds.

## ✔️ Verification

- You can explain the `listen()` → `start()` → `bindEvents()` chain.
