# Module 7 Exercises: Transport and HTTP Integration

## Exercise 7.1: Client Contract

**Goal:** Define a type-safe client contract.

**Tasks**

1. Choose an external endpoint.
2. Define request DTO and response DTO.
3. Define standard error mapping.

**Deliverable**

- DTO definitions + error normalization rules.

---

## Exercise 7.2: Resilience Policy

**Goal:** Define timeouts and retries.

**Tasks**

1. Define timeout per endpoint.
2. Define retry policy for GET vs POST.

**Deliverable**

- Resilience table: endpoint | timeout | retries | notes.

---

## Exercise 7.3: Interceptor Order

**Goal:** Specify interceptor ordering.

**Tasks**

1. Define interceptors for headers, auth, logging, errors.
2. Explain the order.

**Deliverable**

- Ordered list + rationale.
