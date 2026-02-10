# Module 2 Exercises: Advanced Decorators and Interceptors

## Exercise 2.1: Decorator Design

**Goal:** Design a custom decorator and define metadata.

**Tasks**

1. Define a decorator use case (e.g., `@Audit()` or `@RateLimit()`).
2. Specify the metadata fields the decorator should store.
3. Explain where and when the metadata is read.

**Deliverable**

- A short spec: name, inputs, metadata shape, runtime behavior.

---

## Exercise 2.2: Interceptor Chain

**Goal:** Define an interceptor chain for request/response handling.

**Tasks**

1. Create an ordered list of interceptors for: correlation headers, logging, error normalization.
2. Explain why order matters.

**Deliverable**

- Interceptor chain diagram (bullets are fine) + rationale.

---

## Exercise 2.3: Masking Strategy

**Goal:** Decide which fields must be masked in logs.

**Tasks**

1. List at least 10 sensitive keys.
2. Define 2 masking rules: partial reveal and full redaction.

**Deliverable**

- Masking policy + examples of before/after.
