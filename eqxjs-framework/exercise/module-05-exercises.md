# Module 5 Exercises: Interceptors & HTTP Handling

## 📚 Exercise Overview

These exercises focus on implementing EQXJS-style interceptors for HTTP request/response transformation, error handling, and performance/compatibility concerns.

### 🎯 Learning Objectives

- Implement and compose NestJS interceptors using RxJS operators
- Standardize API responses and error envelopes
- Add request correlation IDs and structured logging
- Handle legacy response formats and migration strategies
- Apply performance and safety best practices in interceptors

### ⏱️ Estimated Time: 3 hours

---

## 🏁 Exercise 5.1: Request ID & Timing Interceptor (Quick Start)

### Objective

Create a reusable interceptor that ensures every request has a correlation ID and logs request duration.

### Instructions

1. **Create an interceptor file** `src/interceptors/request-context.interceptor.ts`.
2. Implement:
   - If header `x-request-id` is missing, generate one.
   - Attach the request id to response header `x-request-id`.
   - Measure duration and log `{ requestId, method, path, statusCode, durationMs }`.

Example skeleton:

```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
  Logger,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { tap } from "rxjs/operators";

@Injectable()
export class RequestContextInterceptor implements NestInterceptor {
  private readonly logger = new Logger(RequestContextInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const http = context.switchToHttp();
    const request = http.getRequest();
    const response = http.getResponse();

    const requestId =
      request.headers["x-request-id"] || this.generateRequestId();
    request.headers["x-request-id"] = requestId;
    response.setHeader("x-request-id", requestId);

    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        const durationMs = Date.now() - start;
        this.logger.log({
          requestId,
          method: request.method,
          path: request.url,
          statusCode: response.statusCode,
          durationMs,
        });
      }),
    );
  }

  private generateRequestId(): string {
    return `${Date.now()}-${Math.random().toString(16).slice(2)}`;
  }
}
```

3. Register it globally (recommended) or per-controller.

### 📝 Tasks

- [ ] Interceptor generates `x-request-id` when missing
- [ ] Response includes the same `x-request-id`
- [ ] Logs duration and status code for every request

### 🎯 Expected Output

- Every HTTP response contains `x-request-id`.
- Logs include request timing and correlation id.

---

## 🔧 Exercise 5.2: Standard Response Envelope Interceptor (Hands-On)

### Objective

Wrap successful responses into a consistent envelope.

Target envelope:

```json
{
  "success": true,
  "data": {},
  "meta": {
    "timestamp": "...",
    "requestId": "..."
  }
}
```

### Instructions

1. Create `src/interceptors/response-envelope.interceptor.ts`.
2. Read `x-request-id` from the request.
3. Use RxJS `map()` to wrap the response.
4. Do **not** wrap if the handler already returns an object with `success` field (idempotency).

### 📝 Tasks

- [ ] All normal responses become `{ success: true, data, meta }`
- [ ] Responses that already include `success` are not double-wrapped
- [ ] Envelope includes ISO timestamp and requestId

### 🎯 Expected Output

- `GET /ping` returning `"pong"` becomes:

```json
{
  "success": true,
  "data": "pong",
  "meta": { "timestamp": "...", "requestId": "..." }
}
```

---

## 🚀 Exercise 5.3: Error Envelope + Exception Mapping (Hands-On)

### Objective

Standardize error responses using an interceptor (or filter) that returns safe, predictable error shapes.

Target envelope:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "...",
    "details": []
  },
  "meta": { "timestamp": "...", "requestId": "..." }
}
```

### Instructions

1. Choose one approach:
   - Interceptor using `catchError()`
   - Global `ExceptionFilter` (often cleaner)
2. Map common errors:
   - `BadRequestException` → `VALIDATION_ERROR`
   - `UnauthorizedException` → `UNAUTHORIZED`
   - `ForbiddenException` → `FORBIDDEN`
   - `NotFoundException` → `NOT_FOUND`
   - Default → `INTERNAL_ERROR`
3. Include requestId and timestamp.
4. Ensure internal errors do not leak stack traces in production.

### 📝 Tasks

- [ ] Error responses match the target envelope
- [ ] HTTP status codes remain correct
- [ ] Production mode hides sensitive details

---

## 📦 Exercise 5.4: REST Enhancements (Hands-On)

### Objective

Implement REST-friendly conventions:

- Pagination meta for list endpoints
- Content negotiation for JSON only (reject others)

### Instructions

1. Create an interceptor that, when the handler returns `{ items, total }`, converts it into:

```json
{
  "success": true,
  "data": { "items": [], "total": 0 },
  "meta": { "page": 1, "limit": 20, "totalPages": 0 }
}
```

2. Read pagination query params `page`, `limit`.
3. If `Accept` header is present and does not include `application/json`, return `406 Not Acceptable`.

### 📝 Tasks

- [ ] Pagination meta is computed correctly
- [ ] Non-JSON `Accept` is rejected with 406

---

## 🧩 Exercise 5.5: Legacy HTTP Compatibility Layer (Challenge)

### Objective

Support a legacy response format while allowing new clients to adopt the new envelope.

### Instructions

1. Implement a strategy:
   - If header `x-api-version: legacy`, respond in legacy format
   - Otherwise, respond in new envelope
2. Define a legacy format (example):

```json
{ "status": "OK", "result": {} }
```

3. Make sure errors also follow a legacy shape when legacy mode is enabled.

### 📝 Tasks

- [ ] New clients get modern envelope
- [ ] Legacy clients get legacy envelope
- [ ] Both modes include `x-request-id`

---

## 🧪 Exercise 5.6: Testing Interceptors (Hands-On)

### Objective

Write unit tests that validate the behavior of your interceptors.

### Instructions

1. Add Jest tests for:
   - `x-request-id` generation and propagation
   - response envelope wrapping
   - error envelope mapping
2. Mock `ExecutionContext` and `CallHandler`.

### 📝 Tasks

- [ ] Tests cover at least 3 scenarios per interceptor
- [ ] Tests assert both response body and headers where relevant

---

## 🔍 Exercise 5.7: Performance & Safety Review (Challenge)

### Objective

Audit your interceptors for common performance and reliability pitfalls.

### Checklist

- [ ] Avoid deep cloning large payloads by default
- [ ] Avoid logging full request/response bodies unless explicitly enabled
- [ ] Ensure interceptors are idempotent and order-safe
- [ ] Ensure interceptors do not swallow errors
- [ ] Ensure timeouts/cancellation behavior is acceptable

Deliverable: a short `INTERCEPTOR_REVIEW.md` describing what you changed.

---

## 🏆 Exercise 5.8: End-to-End Demo Endpoint (Project)

### Objective

Create a small controller to demonstrate all interceptor capabilities.

### Requirements

Create endpoints:

- `GET /demo/ping` → returns `"pong"`
- `GET /demo/items?page=1&limit=10` → returns `{ items, total }`
- `GET /demo/error` → throws a `BadRequestException`

### 📝 Tasks

- [ ] All endpoints return standardized envelopes
- [ ] Request ID and timing logs appear
- [ ] Legacy mode works when `x-api-version: legacy` is set

---

## ✅ Wrap-up

When you finish, you should have a working interceptor stack that:

- standardizes success and error responses
- provides correlation IDs and request timing
- supports legacy response compatibility
- is covered by unit tests
