# Module 3: Template Architecture Deep Dive

## 📚 Learning Objectives

By the end of this module, you will:

- Understand the layered architecture pattern used in EQXJS Template
- Master the request/response flow for both REST and events
- Learn the Manager, Service, and Repository patterns
- Understand dependency injection and module organization
- Know when to use each layer appropriately

---

## 3.1 Architecture Overview

### Layered Architecture Pattern

The EQXJS Template follows a **layered architecture** that promotes separation of concerns, testability, and maintainability.

```mermaid
graph TB
    subgraph "Presentation Layer"
        A1[REST Controllers]
        A2[Event Consumers]
    end

    subgraph "Orchestration Layer"
        B1[Managers]
    end

    subgraph "Business Logic Layer"
        C1[Services]
        C2[Event Producers]
    end

    subgraph "Data Access Layer"
        D1[Repositories]
        D2[External Service Clients]
    end

    subgraph "Infrastructure Layer"
        E1[(MongoDB)]
        E2[Kafka Topics]
        E3[External APIs]
    end

    A1 --> B1
    A2 --> B1
    B1 --> C1
    B1 --> C2
    C1 --> D1
    C1 --> D2
    C2 --> E2
    D1 --> E1
    D2 --> E3

    style B1 fill:#4CAF50
    style C1 fill:#2196F3
    style D1 fill:#FF9800
```

### Layer Responsibilities

| Layer              | Purpose                         | Examples                                      |
| ------------------ | ------------------------------- | --------------------------------------------- |
| **Presentation**   | Handle incoming requests/events | `RestController`, `EventConsumerController`   |
| **Orchestration**  | Coordinate business operations  | `ExampleManager`, `ExampleManagerRest`        |
| **Business Logic** | Implement business rules        | `ExampleService`, `EventProducerService`      |
| **Data Access**    | Abstract data operations        | `ExampleMongoRepository`, `ExampleApiService` |
| **Infrastructure** | External systems                | MongoDB, Kafka, External APIs                 |

---

## 3.2 Request Flow Diagrams

### REST API Request Flow

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Interceptor as AppInterceptor
    participant Controller as REST Controller
    participant Manager as Manager Layer
    participant Service as Service Layer
    participant Repository as Repository Layer
    participant DB as MongoDB
    participant Producer
    participant Kafka

    Client->>Controller: POST /api/example
    activate Controller
    Note over Controller: @UseInterceptors(AppInterceptor)
    Controller->>Interceptor: Before request
    activate Interceptor
    Interceptor->>Interceptor: Log request<br/>Start timer<br/>Setup context
    Interceptor-->>Controller: Continue
    deactivate Interceptor

    Controller->>Manager: handleRestRequest(data)
    activate Manager

    Manager->>Service: example(request)
    activate Service
    Note over Service: Business logic<br/>processing

    Service->>Repository: createExample(data)
    activate Repository
    Repository->>DB: insert()
    DB-->>Repository: document
    deactivate Repository

    Service->>Repository: findManyExample({})
    activate Repository
    Repository->>DB: find()
    DB-->>Repository: documents[]
    deactivate Repository

    Service->>Producer: publisher(event, topic)
    activate Producer
    Producer->>Kafka: send(message)
    Kafka-->>Producer: ack
    Note over Producer: Log produced event
    deactivate Producer

    Service-->>Manager: result
    deactivate Service

    Manager-->>Controller: response
    deactivate Manager

    Controller->>Interceptor: After response
    activate Interceptor
    Interceptor->>Interceptor: Log response<br/>Calculate duration<br/>Clean context
    Interceptor-->>Controller: Continue
    deactivate Interceptor

    Controller-->>Client: 200 OK + data
    deactivate Controller
```

### Kafka Event Consumption Flow

```mermaid
sequenceDiagram
    autonumber
    participant Kafka
    participant Consumer as Event Consumer
    participant Interceptor as AppInterceptor
    participant Manager
    participant Service
    participant Repository
    participant DB as MongoDB
    participant Producer

    Kafka->>Consumer: Deliver message<br/>(topic: example.topic.v1)
    activate Consumer
    Note over Consumer: @EntryPoint('topic-example')

    Consumer->>Interceptor: Before processing
    activate Interceptor
    Interceptor->>Interceptor: Extract context<br/>Setup logging<br/>Start timer
    Interceptor-->>Consumer: Continue
    deactivate Interceptor

    Consumer->>Consumer: @ToObjectDecorator()<br/>Parse message
    Consumer->>Consumer: RemoveAtSymbolPipe<br/>Transform data

    Consumer->>Manager: handleEventExample(data, context)
    activate Manager

    Manager->>Service: processEvent(data)
    activate Service

    Service->>Repository: updateExample(data)
    activate Repository
    Repository->>DB: update()
    DB-->>Repository: result
    deactivate Repository

    Service->>Producer: publisher(event, topic)
    activate Producer
    Producer->>Kafka: send(message)
    Kafka-->>Producer: ack
    deactivate Producer

    Service-->>Manager: processed
    deactivate Service

    Manager-->>Consumer: success
    deactivate Manager

    Consumer->>Interceptor: After processing
    activate Interceptor
    Interceptor->>Interceptor: Log completion<br/>Metrics<br/>Clean context
    Interceptor-->>Consumer: Done
    deactivate Interceptor

    Consumer-->>Kafka: Commit offset
    deactivate Consumer
```

---

## 3.3 Core Architectural Patterns

### 3.3.1 Manager Pattern

**Purpose:** Orchestrate complex business operations that involve multiple services or transactions.

**When to Use:**

- Multiple service calls needed
- Transaction coordination required
- Complex business workflows
- Need to aggregate data from multiple sources

**Example:**

```typescript
import { Injectable } from "@nestjs/common";
import { ExampleService } from "../service/example.service";
import { EventProducerService } from "../producer/event.producer.service";
import { CustomLoggerService, MessageContextService } from "@eqxjs/stub";

@Injectable()
export class ExampleManager {
  constructor(
    private exampleService: ExampleService,
    private producerService: EventProducerService,
    private messageContext: MessageContextService,
    private logger: CustomLoggerService,
  ) {}

  async handleEventExample(data: any, context: any) {
    try {
      // 1. Update message context
      this.messageContext.updateMessageProperties();

      // 2. Coordinate multiple operations
      const result = await this.exampleService.processData(data);

      // 3. Trigger follow-up events if needed
      if (result.shouldNotify) {
        await this.producerService.publisher(
          this.messageContext.cloneContextMessage(result),
          "notification.topic",
        );
      }

      // 4. Return orchestrated result
      return result;
    } catch (error) {
      this.logger.error("Error in handleEventExample", error);
      throw error;
    }
  }
}
```

### 3.3.2 Service Pattern

**Purpose:** Implement business logic and rules.

**When to Use:**

- Business logic implementation
- Data validation and transformation
- Coordinating repository calls
- Implementing domain rules

**Example:**

```typescript
import { Injectable } from "@nestjs/common";
import { ExampleMongoRepository } from "../repositories/implements/example.mongo.repository";
import { ConfigService, CustomLoggerService } from "@eqxjs/stub";

@Injectable()
export class ExampleService {
  constructor(
    private repository: ExampleMongoRepository,
    private logger: CustomLoggerService,
    private configService: ConfigService,
  ) {}

  async processData(data: any): Promise<any> {
    // 1. Validate business rules
    if (!this.validateBusinessRules(data)) {
      throw new Error("Business rule violation");
    }

    // 2. Transform data
    const transformed = this.transformData(data);

    // 3. Persist data
    const saved = await this.repository.createExample(transformed);

    // 4. Apply business logic
    const result = this.applyBusinessLogic(saved);

    return result;
  }

  private validateBusinessRules(data: any): boolean {
    // Business validation logic
    return true;
  }

  private transformData(data: any): any {
    // Data transformation logic
    return data;
  }

  private applyBusinessLogic(data: any): any {
    // Business logic
    return data;
  }
}
```

### 3.3.3 Repository Pattern

**Purpose:** Abstract data access operations.

**When to Use:**

- Database operations
- External service calls
- Data source abstraction
- Query building

**Example:**

```typescript
// Interface
export interface ExampleMongoRepositoryInterface {
  createExample(data: any): Promise<any>;
  findManyExample(filter: any): Promise<any[]>;
  findOneExample(id: string): Promise<any>;
  updateExample(id: string, data: any): Promise<any>;
  deleteExample(id: string): Promise<boolean>;
}

// Implementation
import { Injectable, Scope } from "@nestjs/common";
import { Db } from "mongodb";
import { InjectConnection } from "../../../database/database.module";

@Injectable({ scope: Scope.REQUEST })
export class ExampleMongoRepository implements ExampleMongoRepositoryInterface {
  private collection: any;

  constructor(@InjectConnection() private db: Db) {
    this.collection = this.db.collection("examples");
  }

  async createExample(data: any): Promise<any> {
    const result = await this.collection.insertOne({
      ...data,
      createdAt: new Date(),
      updatedAt: new Date(),
    });

    return {
      id: result.insertedId,
      ...data,
    };
  }

  async findManyExample(filter: any): Promise<any[]> {
    return await this.collection.find(filter).toArray();
  }

  async findOneExample(id: string): Promise<any> {
    return await this.collection.findOne({ _id: id });
  }

  async updateExample(id: string, data: any): Promise<any> {
    const result = await this.collection.findOneAndUpdate(
      { _id: id },
      { $set: { ...data, updatedAt: new Date() } },
      { returnDocument: "after" },
    );

    return result.value;
  }

  async deleteExample(id: string): Promise<boolean> {
    const result = await this.collection.deleteOne({ _id: id });
    return result.deletedCount > 0;
  }
}
```

### 3.3.4 DTO (Data Transfer Object) Pattern

**Purpose:** Define the shape of data transferred between layers.

**When to Use:**

- API request/response validation
- Kafka message structure
- Type safety across layers
- Documentation

**Example:**

```typescript
import { IsString, IsNotEmpty, IsOptional, IsNumber } from "class-validator";

export class TopicConsumerEvent {
  @IsNotEmpty()
  header: EventHeader;

  @IsNotEmpty()
  body: EventBody;
}

export class EventHeader {
  @IsNotEmpty()
  identity: Identity;

  @IsString()
  @IsNotEmpty()
  timestamp: string;

  @IsString()
  @IsNotEmpty()
  source: string;
}

export class Identity {
  @IsString()
  @IsNotEmpty()
  correlationId: string;

  @IsString()
  @IsNotEmpty()
  session: string;

  @IsString()
  @IsOptional()
  device?: string;
}

export class EventBody {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsString()
  @IsNotEmpty()
  value: string;

  @IsNumber()
  @IsOptional()
  quantity?: number;
}
```

---

## 3.4 Dependency Injection Strategy

### Module Organization

```mermaid
---
config:
  layout: elk
---
flowchart TB
    AppModule["AppModule"] --> FrameworkModule["FrameworkModule"] & ExamplesModule["ExamplesModule"]
    ExamplesModule --> DatabaseModule["DatabaseModule"] & Controllers["Controllers<br>REST + Event"] & Managers["Managers"] & Services["Services<br>Business + Producer"] & Repositories["Repositories"]

    style FrameworkModule fill:#2196F3
    style ExamplesModule fill:#4CAF50
```

### examples.module.ts

```typescript
import { Module, Scope } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";

// Controllers
import { EventConsumerController } from "./consumer/event.consumer.controller";
import { RestController } from "./consumer/rest.controller";

// Managers
import { ExampleManager } from "./manager/example-manager";
import { ExampleManagerRest } from "./manager/example-manager-rest";

// Services
import { EventProducerService } from "./producer/event.producer.service";
import { ExampleService } from "./service/example.service";

// External Services
import { ExampleApiService } from "./external-services/example-api.service";

// Repositories
import { ExampleMogoRepository } from "./repositories/implements/example.mongo.repository";

// Modules
import { DatabaseModule } from "src/database/database.module";

@Module({
  imports: [
    DatabaseModule, // Import database connectivity
  ],
  controllers: [EventConsumerController, RestController],
  providers: [
    // Managers
    ExampleManager,
    ExampleManagerRest,

    // Services
    EventProducerService,
    ExampleService,
    ExampleApiService,

    // Repositories (Request-scoped for context isolation)
    {
      provide: ExampleMogoRepository,
      scope: Scope.REQUEST,
      useClass: ExampleMogoRepository,
    },
  ],
})
export class ExamplesModule {}
```

### Dependency Injection Best Practices

#### 1. Constructor Injection (Recommended)

```typescript
@Injectable()
export class ExampleService {
  constructor(
    private repository: ExampleMongoRepository,
    private logger: CustomLoggerService,
    private messageContext: MessageContextService,
  ) {
    // Dependencies automatically injected
  }
}
```

#### 2. Request-Scoped Providers

Use for providers that need request-specific context:

```typescript
@Injectable({ scope: Scope.REQUEST })
export class ExampleMongoRepository {
  // New instance created for each request
  // Maintains request-specific state
}
```

#### 3. Circular Dependency Resolution

Use `@Inject(forwardRef())` when needed:

```typescript
import { Inject, forwardRef } from "@nestjs/common";

@Injectable()
export class ServiceA {
  constructor(
    @Inject(forwardRef(() => ServiceB))
    private serviceB: ServiceB,
  ) {}
}
```

---

## 3.5 Framework Integration Points

### 3.5.1 FrameworkModule Registration

The FrameworkModule provides core functionality:

```typescript
import { Module } from "@nestjs/common";
import { FrameworkModule } from "@eqxjs/stub";
import { join } from "path";

@Module({
  imports: [
    FrameworkModule.register({
      configPath: join(process.cwd(), "assets", "config"),
      zone: process.env.ZONE || "local",
    }),
  ],
})
export class AppModule {}
```

**What FrameworkModule Provides:**

- `ConfigService` - Configuration management
- `CustomLoggerService` - Advanced logging
- `MessageContextService` - Request correlation
- `AppInterceptor` - Request/response interceptor
- Custom decorators and pipes

### 3.5.2 Message Context Flow

```mermaid
graph LR
    A[Request Arrives] --> B{Has Context?}
    B -->|No| C[Create New Context]
    B -->|Yes| D[Extract Context]
    C --> E[MessageContextService]
    D --> E
    E --> F[Store in Request Scope]
    F --> G[Available to All Layers]
    G --> H[Propagate to Events]
    H --> I[Clean Up After Response]

    style E fill:#4CAF50
```

**Usage:**

```typescript
@Injectable()
export class ExampleService {
  constructor(private messageContext: MessageContextService) {}

  async processData(data: any) {
    // Update context with additional metadata
    this.messageContext.updateMessageProperties();

    // Clone context for event publishing
    const event = this.messageContext.cloneContextMessage(data);

    // Delete context after processing
    this.messageContext.deleteContextMessage();

    return event;
  }
}
```

### 3.5.3 Logging Infrastructure

```typescript
import { CustomLoggerService, LoggerAction } from "@eqxjs/stub";

@Injectable()
export class ExampleService {
  constructor(private logger: CustomLoggerService) {}

  async processData(data: any) {
    // Log with action context
    this.logger.info(LoggerAction.PROCESSING("Processing user data"), {
      userId: data.id,
    });

    try {
      const result = await this.doWork(data);

      this.logger.info(
        LoggerAction.PROCESSED("Data processed successfully"),
        result,
      );

      return result;
    } catch (error) {
      this.logger.error(LoggerAction.FAILED("Processing failed"), error);
      throw error;
    }
  }
}
```

---

## 3.6 Error Handling Strategy

### Error Flow Architecture

```mermaid
graph TB
    A[Error Occurs] --> B{Layer?}

    B -->|Controller| C[AppInterceptor]
    B -->|Manager| D[Log & Transform]
    B -->|Service| E[Log & Throw]
    B -->|Repository| F[Log & Throw]

    C --> G[HTTP Exception Filter]
    D --> G
    E --> D
    F --> E

    G --> H{Error Type?}
    H -->|Business| I[400 Bad Request]
    H -->|Not Found| J[404 Not Found]
    H -->|System| K[500 Internal Server]

    I --> L[Log Error Details]
    J --> L
    K --> L

    L --> M[Return Error Response]

    style G fill:#FF5722
    style L fill:#FFC107
```

### Implementation

```typescript
// In Service Layer
@Injectable()
export class ExampleService {
  async processData(data: any) {
    try {
      return await this.repository.save(data);
    } catch (error) {
      this.logger.error("Failed to process data", error);
      throw error; // Let upper layers handle
    }
  }
}

// In Manager Layer
@Injectable()
export class ExampleManager {
  async handleRequest(data: any) {
    try {
      return await this.service.processData(data);
    } catch (error) {
      // Transform error if needed
      throw new BadRequestException("Invalid data provided");
    }
  }
}

// AppInterceptor handles at Controller level
@UseInterceptors(AppInterceptor)
@Controller()
export class MyController {
  // Errors are automatically logged and transformed
}
```

---

## 📝 Summary

In this module, you learned:

- ✅ Layered architecture pattern with clear separation of concerns
- ✅ Request flow for REST APIs and Kafka events
- ✅ Manager, Service, and Repository patterns
- ✅ Dependency injection strategy and module organization
- ✅ Framework integration points
- ✅ Error handling architecture

---

## 🎯 Next Steps

In [Module 4: Controllers and Managers](module-04-controllers-managers.md), you will:

- Implement REST controllers
- Create Kafka event consumers
- Build manager orchestration logic
- Use decorators and pipes effectively

---

**[← Previous: Module 2](module-02-getting-started.md)** | **[Back to Course Outline](course-outline.md)** | **[Next: Module 4 →](module-04-controllers-managers.md)**
