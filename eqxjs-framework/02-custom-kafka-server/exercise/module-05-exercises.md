# Module 5 Exercises: Topic Monitoring & Subscription

## 🎯 Goals

- Understand the monitoring feature
- Practice toggling monitoring

## ✅ Exercise 1: Disable monitoring

1. Set `MONITOR_TOPICS=false`.
2. Start the service.
3. Confirm it subscribes directly to registered patterns.

## ✅ Exercise 2: Monitoring interval

1. Enable monitoring.
2. Set `KAFKA_MONITOR_INTERVAL=10000`.
3. Observe periodic monitor logs.

## 🧩 Challenge

Create a new topic in Kafka while the service is running and add a new `@MessagePattern` handler, then restart the service and verify it discovers the new topic.

## ✔️ Verification

- You can describe when monitoring helps and when it’s overhead.
