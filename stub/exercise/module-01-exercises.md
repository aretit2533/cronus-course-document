# Module 1 Exercises: EQXJS Ecosystem Foundation

## Exercise 1.1: Identify Framework Modules

**Goal:** Map each EQXJS module to its responsibility.

**Tasks**

1. From the documentation index, list the modules re-exported by the framework.
2. For each module, write 1-2 sentences describing its role.
3. Pick 2 modules and describe how they work together in a typical service.

**Deliverable**

- A short table: `module` | `responsibility` | `example usage`

---

## Exercise 1.2: Bootstrap Checklist

**Goal:** Produce a production-ready bootstrap checklist.

**Tasks**

1. List the steps required to initialize an EQXJS-based NestJS app.
2. Include graceful shutdown requirements.
3. Include configuration validation requirements.

**Deliverable**

- A checklist (10-15 bullets) that you can reuse in projects.

---

## Exercise 1.3: Configuration Map

**Goal:** Define a normalized config shape.

**Tasks**

1. Propose a YAML config structure that includes: `app`, `http`, `logging`, `security`, and one dependency.
2. State which values should come from env vars (secrets).

**Deliverable**

- Sample YAML + a short note on env var usage.

---

## Exercise 1.4: Logging Baseline

**Goal:** Define logging fields used across services.

**Tasks**

1. Define a standard log payload schema (fields + types).
2. Describe how correlation IDs should be attached.

**Deliverable**

- A JSON example of a log entry for a successful request and a failed dependency call.
