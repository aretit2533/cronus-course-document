# Module 3: REST API Flow

> Follow an HTTP request from controller to service to response.

---

## Learning Goals

- Identify the REST entry point
- Understand manager and service responsibilities
- See how outbound HTTP calls and DB access are orchestrated

---

## 3.1 REST Entry Point

`src/example/consumer/rest.controller.ts` defines the REST endpoint:

- `@HttpEntryPoint` marks the handler as an EQXJS REST entry
- `RestInterceptor` captures common logging and context
- The controller delegates to `ExampleManagerRest`

---

## 3.2 Manager Responsibilities

`ExampleManagerRest` handles:

- Message context initialization
- Inbound and outbound logging
- Response shaping and status codes
- Summary log flushing

This keeps the controller thin and focused on transport details.

---

## 3.3 Service Workflow

`ExampleService.example()` orchestrates the work:

1. Update the message context
2. Call the external API via `ExampleApiService`
3. Read and write to MongoDB through `ExampleMogoRepository`
4. Produce a Kafka event using `EventProducerService`

---

## 3.4 External API Integration

`ExampleApiService` uses `CustomAxiosService` and logs dependency metadata for observability. Configuration is pulled from `api-services` in YAML.

---

## Knowledge Check

- Which class writes the HTTP response payload?
- Where does the external API URL come from?
- Which service publishes the Kafka event?

---

## Next Module

Continue to [Module 4: Kafka and Data Flow](module-04-kafka-data-flow.md).
