# Module 3 Exercises: Health Checks and Service Management

## Exercise 3.1: Health Endpoint Spec

**Goal:** Define liveness/readiness/startup probes.

**Tasks**

1. Define the response shapes for each probe.
2. Define what dependencies belong in readiness.

**Deliverable**

- A brief spec including response fields and status rules.

---

## Exercise 3.2: Dependency Health Matrix

**Goal:** Build a matrix of dependencies and signals.

**Tasks**

1. Pick 3 dependencies (DB, Kafka, external API).
2. For each, define checks, timeouts, and failure thresholds.

**Deliverable**

- A table: dependency | check | timeout | failure behavior.

---

## Exercise 3.3: Graceful Shutdown Plan

**Goal:** Create a shutdown plan for a service.

**Tasks**

1. List resources that need cleanup.
2. Describe ordering (stop receiving traffic, drain, close).

**Deliverable**

- Step-by-step shutdown sequence.
