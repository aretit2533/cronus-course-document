# Module 6 Exercises: Memory Guardrails & Production Tuning

## 🎯 Goals

- Enable heap guardrails
- Understand pause/resume impact

## ✅ Exercise 1: Enable heap guardrails

1. Set:
   - `KAFKA_HEAP_USED_SIZE_CHECK_ENABLED=true`
   - `KAFKA_HEAP_USED_SIZE_PERCENT=1` (intentionally low)
   - `KAFKA_CONSUMER_PAUSE_TIME_MS=5000`

2. Start the service and produce messages.
3. Observe logs indicating pause/resume.

## ✅ Exercise 2: Observe lag behavior

If you can measure consumer lag in your Kafka environment, observe that pausing increases lag temporarily.

## 🧩 Challenge

Tune the threshold to a realistic value (e.g., 75–85) and write down a policy for when the guardrail should be enabled.

## ✔️ Verification

- You can explain the tradeoff: stability vs throughput/latency.
