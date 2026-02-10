# Module 7: Transport and HTTP Integration

## EQXJS Framework - HTTP Client, Interceptors, and Service Communication

---

## 🎯 Learning Objectives

After completing this module, you will be able to:

- Understand EQXJS HTTP transport architecture and integration points
- Build type-safe HTTP clients with consistent error handling
- Apply interceptor chains for outbound requests and inbound responses
- Implement resilience patterns (timeouts, retries, circuit breakers)
- Integrate service-to-service communication patterns in microservices

---

## 📚 Module Overview

Enterprise services rarely run alone. They depend on external APIs, internal microservices, and shared platforms. This module teaches how to design robust HTTP communication using EQXJS transport patterns (HTTP transporter, interceptors, standard errors, and logging).

Key themes:

- **Consistency**: standard request/response handling
- **Safety**: authentication, masking, and validation
- **Reliability**: retries, timeouts, fallbacks
- **Observability**: correlation IDs and structured logs

---

## 7.1 HTTP Transport Architecture

### Common components

A typical EQXJS HTTP integration stack includes:

- HTTP client provider (configured once)
- Interceptors for:
  - headers/correlation
  - auth
  - logging/masking
  - error normalization
- DTO validation (incoming + outgoing)

### Recommended request lifecycle

1. Build request DTO
2. Validate DTO
3. Add context headers (`x-correlation-id`, `x-request-id`)
4. Send request with timeout
5. Normalize response errors
6. Log summary + metrics

---

## 7.2 API Client Development

### Type-safe client interface

```typescript
export interface HttpRequestOptions {
  timeoutMs?: number;
  headers?: Record<string, string>;
}

export interface HttpResult<T> {
  status: number;
  data: T;
  headers: Record<string, string | string[]>;
}

export interface ExternalUserResponse {
  id: string;
  email: string;
  status: "active" | "inactive";
}

export class UsersApiClient {
  constructor(
    private readonly baseUrl: string,
    private readonly http: { get: Function; post: Function },
  ) {}

  async getUserById(
    userId: string,
    opts: HttpRequestOptions = {},
  ): Promise<HttpResult<ExternalUserResponse>> {
    const url = `${this.baseUrl}/users/${encodeURIComponent(userId)}`;
    return await this.http.get(url, {
      timeoutMs: opts.timeoutMs ?? 5000,
      headers: {
        ...(opts.headers ?? {}),
      },
    });
  }
}
```

### Standardizing errors

Create normalized error shapes that every client uses:

```typescript
export interface HttpErrorDetails {
  code: string;
  message: string;
  status?: number;
  retriable?: boolean;
  cause?: unknown;
}

export function normalizeHttpError(err: any): HttpErrorDetails {
  // Example rules. Adapt to your actual HTTP library.
  if (err?.code === "ECONNABORTED" || err?.message?.includes("timeout")) {
    return {
      code: "HTTP_TIMEOUT",
      message: "HTTP request timed out",
      retriable: true,
      cause: err,
    };
  }
  if (typeof err?.status === "number") {
    const retriable = err.status >= 500;
    return {
      code: "HTTP_RESPONSE_ERROR",
      message: "HTTP response error",
      status: err.status,
      retriable,
      cause: err,
    };
  }
  return {
    code: "HTTP_UNKNOWN",
    message: "Unknown HTTP error",
    retriable: false,
    cause: err,
  };
}
```

---

## 7.3 Interceptors for Outbound Requests

### Header propagation interceptor

```typescript
export interface ContextHeaderProvider {
  getCorrelationId(): string | undefined;
  getRequestId(): string | undefined;
}

export function buildContextHeaders(
  ctx: ContextHeaderProvider,
): Record<string, string> {
  const headers: Record<string, string> = {};
  const correlationId = ctx.getCorrelationId();
  const requestId = ctx.getRequestId();
  if (correlationId) headers["x-correlation-id"] = correlationId;
  if (requestId) headers["x-request-id"] = requestId;
  return headers;
}
```

### Authentication interceptor

```typescript
export interface AuthTokenProvider {
  getAccessToken(): Promise<string>;
}

export async function buildAuthHeader(
  provider: AuthTokenProvider,
): Promise<Record<string, string>> {
  const token = await provider.getAccessToken();
  return { authorization: `Bearer ${token}` };
}
```

### Logging interceptor (safe-by-default)

Only log summary fields unless explicitly in debug mode; always mask sensitive headers.

---

## 7.4 Resilience Patterns

### Timeouts

Define service-level timeouts and enforce them consistently.

- Short timeouts for interactive requests
- Longer timeouts for batch/async tasks

### Retries

Retry only when safe:

- network errors
- `5xx`
- idempotent operations (GET)

Avoid retrying:

- non-idempotent POST without idempotency keys
- `4xx` validation errors

### Circuit breaker

Use circuit breakers when dependencies are flaky:

- Open circuit on repeated failures
- Half-open to probe recovery

### Backoff strategies

Use exponential backoff with jitter to avoid thundering herd.

---

## 7.5 Service-to-Service Communication Patterns

### Contract-driven integration

Use:

- DTO schemas and validation
- versioned endpoints
- backward-compatible changes

### Observability requirements

Every call should produce:

- correlation ID propagation
- latency metric
- dependency error metric

### Safe fallbacks

Define a fallback policy per dependency:

- cached response
- degraded functionality
- fail-fast with meaningful error

---

## 📝 Summary

In this module, you learned:

- How to design HTTP clients with consistent behavior
- How to apply interceptors for headers, auth, logging, and errors
- Reliability patterns to stabilize microservice communication
- Integration patterns that support operations and troubleshooting

## 🎯 Next Steps

- Complete the [Module 7 Exercises](exercise/module-07-exercises.md)
- Standardize one real API client with these patterns
- Continue to [Module 8: Configuration Management and Commander](Module-08-Configuration-Management-and-Commander.md)

---

**[⬅️ Previous: Module 6 - Logging and Monitoring Systems](Module-06-Logging-and-Monitoring-Systems.md)** | **[Next: Module 8 - Configuration Management and Commander ➡️](Module-08-Configuration-Management-and-Commander.md)**
