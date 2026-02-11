# Module 6 Exercises: Context Management & Domain Services

## 📚 Exercise Overview

These exercises focus on building Domain Service Contexts: isolating domain services, managing lifecycle hooks, and enabling safe cross-domain communication.

### 🎯 Learning Objectives

- Design domain boundaries and dependencies
- Implement a simple Domain Service Context registry
- Add lifecycle hooks (bootstrap/ready/shutdown)
- Implement cross-domain communication via an event bus
- Apply advanced patterns (multi-tenant contexts, health per domain)

### ⏱️ Estimated Time: 2.5 hours

---

## 🏁 Exercise 6.1: Define Domain Boundaries (Quick Start)

### Objective

Create a domain map and dependency diagram for a sample system.

### Scenario

You are building an internal platform with domains:

- `identity` (users, roles)
- `catalog` (products)
- `orders` (checkout)
- `payments` (payment processing)

### Instructions

1. Create `docs/domain-map.md`.
2. For each domain, list:
   - primary responsibilities
   - exported capabilities (APIs/services)
   - dependencies on other domains
3. Add a dependency rule: **dependencies must be one-directional** and avoid cycles.

### 📝 Tasks

- [ ] Domain map exists and is readable
- [ ] Dependencies are explicit
- [ ] You identified at least 2 potential cycle risks and how to avoid them

---

## 🔧 Exercise 6.2: Minimal Context Registry (Hands-On)

### Objective

Implement a simple registry that resolves services by domain.

### Instructions

1. Create a `DomainContextRegistry` service with APIs:

- `register(domain, context)`
- `get(domain)`
- `listDomains()`

2. Create two contexts (`catalog`, `orders`) exposing one service each.

Example interface:

```ts
export interface DomainServiceContext {
  getDomain(): string;
  get<T>(token: any): T;
}
```

### 📝 Tasks

- [ ] You can register both contexts
- [ ] You can resolve a service from each context
- [ ] Unknown domain returns a safe error

---

## 🚀 Exercise 6.3: Lifecycle Hooks (Hands-On)

### Objective

Add lifecycle management to contexts.

### Instructions

1. Extend your context implementation to accept lifecycle hooks:

- `onBootstrap()`
- `onReady()`
- `onShutdown()`

2. Ensure the registry can call:

- `bootstrapAll()`
- `readyAll()`
- `shutdownAll()`

3. Add timeouts per hook (e.g., 10s) so shutdown never blocks forever.

### 📝 Tasks

- [ ] Hook execution order is deterministic
- [ ] A failing hook is logged but does not prevent other domains from running hooks
- [ ] Shutdown hooks are always attempted

---

## 📡 Exercise 6.4: Cross-Domain Communication via Event Bus (Challenge)

### Objective

Implement a lightweight in-process event bus for cross-domain messaging.

### Instructions

1. Create an `EventBus` with:

- `publish(eventType, payload)`
- `subscribe(eventType, handler)`

2. Implement events:

- `catalog.productCreated`
- `orders.orderPlaced`

3. Wire behaviors:

- When a product is created in `catalog`, `orders` updates its “search index cache” (mocked in memory).

### 📝 Tasks

- [ ] `orders` receives events from `catalog`
- [ ] Handlers are isolated (a failing handler does not crash the publisher)
- [ ] Event payload includes `timestamp` and `correlationId`

---

## 🧪 Exercise 6.5: Domain Health per Context (Hands-On)

### Objective

Expose health signals per domain context.

### Instructions

1. Add a `health()` method to each context that returns:

```json
{ "domain": "catalog", "status": "up", "checks": { "cache": "up" } }
```

2. Implement a simple aggregator endpoint `GET /health/domains` returning all domain health results.

### 📝 Tasks

- [ ] Each domain can report health independently
- [ ] Aggregated health response contains all domains
- [ ] A single degraded domain is clearly visible in the output

---

## 🏆 Exercise 6.6: Multi-Tenant Context Isolation (Project)

### Objective

Simulate multi-tenant contexts where each tenant gets an isolated domain context instance.

### Instructions

1. Define tenants: `tenantA`, `tenantB`.
2. Implement a `TenantContextManager` that:

- creates per-tenant domain contexts
- routes requests based on header `x-tenant-id`

3. Demonstrate isolation:

- Creating a product in `tenantA` should not affect `tenantB`.

### 📝 Tasks

- [ ] Requests with different tenant headers do not share state
- [ ] Events are tenant-scoped
- [ ] Health endpoint can show per-tenant domain status

---

## ✅ Wrap-up

When you finish, you should have a practical understanding of domain isolation, lifecycle management, and safe cross-domain communication patterns that scale to enterprise needs.
