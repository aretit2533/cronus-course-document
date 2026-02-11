# Module 4 Exercises: Resilience & Recovery

## 🎯 Goals

- Trigger a consumer failure
- Verify the consumer recreation path

## ✅ Exercise 1: Fault injection

1. In your message handler for `orders.created`, intentionally throw an error for a specific payload (e.g. `id === 'boom'`).
2. Produce a message with that payload.
3. Observe logs:
   - error output
   - consumer crash handling
   - recreation attempt

## ✅ Exercise 2: Concurrency guard

1. Produce multiple failing messages quickly.
2. Observe that the restart loop does not overlap infinitely.

## 🧩 Challenge

Implement a “dead letter” topic producer in your handler for failures, so errors don’t continuously crash the consumer.

## ✔️ Verification

- You can capture log evidence of: `consumer.crash` → recreate.
