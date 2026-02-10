# Module 8 Exercises: Configuration Management and Commander

## Exercise 8.1: YAML Config Design

**Goal:** Create a multi-environment config layout.

**Tasks**

1. Draft `default.yml` and `dev.yml` overrides.
2. Identify secrets and map them to env vars.

**Deliverable**

- Two YAML samples + env var list.

---

## Exercise 8.2: Config Validation

**Goal:** Write a Joi schema for the config.

**Tasks**

1. Validate ports, base URLs, and required fields.
2. Include defaults where appropriate.

**Deliverable**

- Joi schema + examples of invalid configs.

---

## Exercise 8.3: Command Pattern

**Goal:** Design an operational command.

**Tasks**

1. Define a command (e.g., backfill, cleanup, reindex).
2. Define input, validations, and safe/dry-run mode.

**Deliverable**

- Command spec + example execution logs.
