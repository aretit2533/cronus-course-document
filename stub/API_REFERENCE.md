# Framework Module API Reference (@eqxjs/stub)

## Quick Reference

This document provides a complete API reference for the @corp-ais/eqxjs-stub Framework Module and all exported components.

## Table of Contents

- [Core Module](#core-module)
- [Services](#services)
- [Decorators](#decorators)
- [Interceptors](#interceptors)
- [DTOs and Interfaces](#dtos-and-interfaces)
- [Utilities](#utilities)
- [Re-exported Modules](#re-exported-modules)

## Core Module

### FrameworkModule

```typescript
@Module({})
export class FrameworkModule {
  static register(options: FrameworkOptionDto): DynamicModule;
}
```

**Description:** The main framework module that bootstraps the EQXJS ecosystem as a global dynamic module.

**Type:** `@Module()` decorator class  
**Scope:** Global

#### Static Methods

##### register(options: FrameworkOptionDto): DynamicModule

Registers and configures the framework module with integration of all EQXJS ecosystem modules.

```typescript
static register(options: FrameworkOptionDto): DynamicModule
```

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `options` | `FrameworkOptionDto` | Configuration options for framework initialization |
| `options.configPath` | `string` | Path to configuration files directory |
| `options.zone` | `string` | Environment zone identifier (development, staging, production, test) |

**Returns:** `DynamicModule` - Configured dynamic module with integrated EQXJS services

**Example:**

```typescript
import { Module } from '@nestjs/common';
import { FrameworkModule } from '@corp-ais/eqxjs-stub';

@Module({
  imports: [
    FrameworkModule.register({
      configPath: './config',
      zone: 'development'
    })
  ]
})
export class AppModule {}
```

#### Registered Modules

| Module | Package | Description |
|--------|---------|-------------|
| `DomainConfigModule` | `@corp-ais/eqxjs-commander` | Configuration management and command handling |
| `HttpClientModule` | `@corp-ais/eqxjs-transporter-http` | HTTP transport layer and client capabilities |
| `LoggerModule` | `@corp-ais/eqxjs-logger` | Comprehensive logging with masking and formatting |
| `UtilModule` | `@corp-ais/eqxjs-utils` | Utility functions and common services |
| `SecurityModule` | `@corp-ais/eqxjs-security` | Security utilities and validation |

#### Provided Services

| Service | Description |
|---------|-------------|
| `FrameworkUtilService` | Framework-specific utility operations |
| `GracefulShutdownService` | Application lifecycle and cleanup management |

---

## Services

### FrameworkUtilService

```typescript
@Injectable()
export class FrameworkUtilService {
  // Framework-specific utility methods
}
```

**Description:** Injectable service providing framework-specific utility operations.

**Decorator:** `@Injectable()`  
**Scope:** Singleton

**Usage Example:**

```typescript
import { Injectable } from '@nestjs/common';
import { FrameworkUtilService } from '@corp-ais/eqxjs-stub';

@Injectable()
export class MyService {
  constructor(
    private readonly frameworkUtil: FrameworkUtilService,
  ) {}
}
```

### GracefulShutdownService

```typescript
@Injectable()
export class GracefulShutdownService 
  implements BeforeApplicationShutdown, OnApplicationShutdown {
  
  setup(app: INestApplication): void;
  addCleanupTask(task: () => Promise<void>): void;
  shutdown(): Promise<void>;
  beforeApplicationShutdown(): Promise<void>;
  onApplicationShutdown(): Promise<void>;
}
```

**Description:** Injectable service that handles graceful application shutdown with cleanup orchestration.

**Decorator:** `@Injectable()`  
**Scope:** Singleton  
**Implements:** `BeforeApplicationShutdown`, `OnApplicationShutdown`

#### Methods

##### setup(app: INestApplication): void

Initializes graceful shutdown handling for the NestJS application.

| Parameter | Type | Description |
|-----------|------|-------------|
| `app` | `INestApplication` | NestJS application instance |

##### addCleanupTask(task: () => Promise<void>): void

Registers a cleanup task to be executed during shutdown sequence.

| Parameter | Type | Description |
|-----------|------|-------------|
| `task` | `() => Promise<void>` | Async cleanup function to execute |

##### shutdown(): Promise<void>

Manually initiates the graceful shutdown sequence.

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `gracefulShutdownTimeout` | `number` | Shutdown timeout in milliseconds |
| `circuitBreakerTimeout` | `number` | Circuit breaker timeout |
| `processingMessages` | `number` | Current processing message count |

**Usage Example:**

```typescript
import { Injectable } from '@nestjs/common';
import { GracefulShutdownService } from '@corp-ais/eqxjs-stub';

@Injectable()
export class MyService {
  constructor(
    private readonly gracefulShutdown: GracefulShutdownService,
  ) {
    this.gracefulShutdown.addCleanupTask(async () => {
      await this.closeConnections();
    });
  }
  
  private async closeConnections(): Promise<void> {
    // Cleanup logic
  }
}
```

---

## Decorators

### @EntryPoint()

```typescript
EntryPoint(event: string, options?: EntrypointOptionDto): MethodDecorator
```

**Description:** Method decorator that marks a method as an entry point for event processing. Automatically applies logging, masking, and message mode decorators.

**Type:** Method Decorator

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event` | `string` | Yes | Event name to listen for |
| `options` | `EntrypointOptionDto` | No | Optional configuration object |
| `options.manualConsumerLogging` | `boolean` | No | Enable/disable manual consumer logging |
| `options.consumerMasking` | `string[]` | No | Array of field names to mask in logs |
| `options.messageType` | `MessageTypeEnum` | No | Message processing type (defaults to M2) |

#### Automatically Applied Decorators

- `@EventPattern(event)` - NestJS microservice event pattern
- `@Logs(event)` - EQXJS logging decorator
- `@DisableConsumerLogging()` - Consumer logging control
- `@ConsumerMasking()` - Field masking for sensitive data
- `@MessageMode()` - Message processing mode

**Usage Example:**

```typescript
import { Controller } from '@nestjs/common';
import { Payload } from '@nestjs/microservices';
import { EntryPoint, MessageTypeEnum } from '@corp-ais/eqxjs-stub';

@Controller()
export class UserController {
  @EntryPoint('USER_CREATED', {
    manualConsumerLogging: false,
    consumerMasking: ['password', 'ssn'],
    messageType: MessageTypeEnum.M2
  })
  async handleUserCreated(@Payload() data: CreateUserDto) {
    // Handle user created event
  }
}
```

### @DisableConsumerLogging()

```typescript
DisableConsumerLogging(disable?: boolean): MethodDecorator
```

**Description:** Method decorator that controls consumer logging for specific methods.

**Type:** Method Decorator

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `disable` | `boolean` | No | `true` | Whether to disable consumer logging |

**Usage Example:**

```typescript
import { Controller, Get } from '@nestjs/common';
import { DisableConsumerLogging } from '@corp-ais/eqxjs-stub';

@Controller('data')
export class DataController {
  @DisableConsumerLogging()
  @Get('sensitive-data')
  getSensitiveData() {
    // No consumer logging for this endpoint
  }
}
```

### @ConsumerMasking()

```typescript
ConsumerMasking(fields: string[]): MethodDecorator
```

**Description:** Method decorator that masks specified fields in consumer logs for security purposes.

**Type:** Method Decorator

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fields` | `string[]` | Yes | Array of field names to mask in logs |

**Usage Example:**

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { ConsumerMasking } from '@corp-ais/eqxjs-stub';

@Controller('users')
export class UserController {
  @ConsumerMasking(['password', 'creditCard', 'ssn'])
  @Post()
  createUser(@Body() userData: CreateUserDto) {
    // Specified fields will be masked in logs
    return this.userService.create(userData);
  }
}
```

### @SetMessageMode()

```typescript
SetMessageMode(mode: MessageTypeEnum): MethodDecorator
```

**Description:** Method decorator that configures the message processing mode for the method.

**Type:** Method Decorator

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mode` | `MessageTypeEnum` | Yes | Message processing mode (M2, M3, etc.) |

**Usage Example:**

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { SetMessageMode, MessageTypeEnum } from '@corp-ais/eqxjs-stub';

@Controller('process')
export class ProcessController {
  @SetMessageMode(MessageTypeEnum.M3)
  @Post('async-process')
  processAsync(@Body() data: ProcessDto) {
    // Will be processed using M3 message mode
    return this.processService.handle(data);
  }
}
```

---

## Interceptors

### AppInterceptor

```typescript
@Injectable({ scope: Scope.REQUEST })
export class AppInterceptor implements NestInterceptor {
  constructor(
    private logger: CustomLoggerService,
    private requestHelper: LoggerHelperService,
    private reflector: Reflector,
    private summaryLogger: CustomSummaryLoggerService,
    private summaryFlush: FlushSummaryLog,
    private messageContext: MessageContextService,
    private frameworkUtil: FrameworkUtilService,
    private instanceDataManager: InstanceDataManagerService,
    private singletonLogger: CustomSingletonLoggerService,
    private gracefulShutdown: GracefulShutdownService,
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any>;
}
```

**Description:** Global application interceptor that handles request/response processing, logging, and framework integration.

**Decorator:** `@Injectable({ scope: Scope.REQUEST })`  
**Implements:** `NestInterceptor`  
**Scope:** Request-scoped

#### Features

- Request/response logging with masking support
- Message type validation and processing
- Graceful shutdown integration
- Error handling and transformation
- Performance monitoring

#### Methods

##### intercept(context: ExecutionContext, next: CallHandler): Observable\<any>

Intercepts and processes requests/responses with comprehensive logging and error handling.

**Registration Example:**

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { AppInterceptor } from '@corp-ais/eqxjs-stub';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: AppInterceptor,
    },
  ],
})
export class AppModule {}
```

### HttpInterceptor

```typescript
@Injectable()
export class HttpInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any>;
}
```

**Description:** Interceptor specifically designed for HTTP client communications and transport layer operations.

**Decorator:** `@Injectable()`  
**Implements:** `NestInterceptor`

### LegacyHttpInterceptor

```typescript
@Injectable()
export class LegacyHttpInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any>;
}
```

**Description:** Interceptor that provides compatibility layer for legacy HTTP communications and maintains backward compatibility.

**Decorator:** `@Injectable()`  
**Implements:** `NestInterceptor`

### RestInterceptor

```typescript
@Injectable()
export class RestInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any>;
}
```

**Description:** Interceptor designed for RESTful API request/response processing with REST-specific optimizations.

**Decorator:** `@Injectable()`  
**Implements:** `NestInterceptor`

---

## DTOs and Interfaces

### FrameworkOptionDto

```typescript
export interface FrameworkOptionDto {
  /**
   * Path to configuration files directory
   * @example './config'
   */
  configPath: string;
  
  /**
   * Environment zone identifier
   * @example 'development' | 'staging' | 'production' | 'test'
   */
  zone: string;
}
```

**Description:** Configuration options interface for framework module initialization.

### EntrypointOptionDto

```typescript
export interface EntrypointOptionDto {
  /**
   * Enable or disable manual consumer logging
   * @default false
   */
  manualConsumerLogging?: boolean;
  
  /**
   * Array of field names to mask in consumer logs
   * @example ['password', 'creditCard', 'ssn']
   */
  consumerMasking?: string[];
  
  /**
   * Message processing type
   * @default MessageTypeEnum.M2
   */
  messageType?: MessageTypeEnum;
}
```

**Description:** Configuration options interface for entry point behavior customization.

### ValidatorsSchemaDto

```typescript
export interface ValidatorsSchemaDto {
  // Schema definitions for data validation
}
```

**Description:** Schema definitions interface for data validation operations.

### Graceful Shutdown Interfaces

```typescript
export interface IGracefulShutdownCleanup {
  /**
   * Cleanup method called during graceful shutdown
   */
  cleanup(): Promise<void>;
}

export interface ICircuitBreakerListener {
  /**
   * Called when circuit breaker opens
   */
  onCircuitBreakerOpen(): void;
  
  /**
   * Called when circuit breaker closes
   */
  onCircuitBreakerClose(): void;
}

export interface ISetupFunctionParams {
  /**
   * NestJS application instance
   */
  app: INestApplication;
  
  /**
   * Optional timeout configuration in milliseconds
   */
  timeout?: number;
}
```

**Description:** Interface definitions for graceful shutdown contract and configuration.

### DomainServiceContext

```typescript
export interface DomainServiceContext {
  // Context management for domain services
}
```

**Description:** Context management interface for domain-driven service operations.

---

## Utilities

### Health Utilities

```typescript
export namespace HealthUtil {
  function checkHealth(): Promise<HealthCheckResult>;
  function getHealthStatus(): Promise<HealthStatus>;
}
```

**Description:** Health check utilities for application monitoring and status reporting.

#### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `checkHealth()` | `Promise<HealthCheckResult>` | Performs comprehensive application health check |
| `getHealthStatus()` | `Promise<HealthStatus>` | Returns current health status of the application |

**Usage Example:**

```typescript
import { HealthUtil } from '@corp-ais/eqxjs-stub';

// Check application health
const healthResult = await HealthUtil.checkHealth();
console.log('Health status:', healthResult.status);

// Get current status
const currentStatus = await HealthUtil.getHealthStatus();
```

### Database Utilities

```typescript
export namespace DbUtil {
  function getConnection(): Promise<DatabaseConnection>;
  function closeConnection(): Promise<void>;
}
```

**Description:** Database connection and management utilities for MongoDB operations.

#### Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `getConnection()` | `Promise<DatabaseConnection>` | Establishes and returns database connection |
| `closeConnection()` | `Promise<void>` | Safely closes database connection |

**Usage Example:**

```typescript
import { DbUtil } from '@corp-ais/eqxjs-stub';

// Establish connection
const connection = await DbUtil.getConnection();

// Later, close connection
await DbUtil.closeConnection();
```

### Joi Schema Validators

```typescript
export namespace JoiSchemas {
  export const M2SupportedSchemas: Joi.Schema;
  export const m3Schema: Joi.Schema;
}
```

**Description:** Pre-configured Joi validation schemas for message type validation.

#### Available Schemas

| Schema | Type | Description |
|--------|------|-------------|
| `M2SupportedSchemas` | `Joi.Schema` | Validation schemas for M2 message types |
| `m3Schema` | `Joi.Schema` | Validation schema for M3 message types |

**Usage Example:**

```typescript
import { M2SupportedSchemas, m3Schema } from '@corp-ais/eqxjs-stub';

// Validate M2 message
const { error, value } = M2SupportedSchemas.validate(messageData);
if (error) {
  throw new BadRequestException('Invalid M2 message format');
}

// Validate M3 message  
const m3Result = m3Schema.validate(m3MessageData);
if (m3Result.error) {
  throw new BadRequestException('Invalid M3 message format');
}
```

---

## Re-exported Modules

The framework re-exports all functionality from the following EQXJS ecosystem modules:

### Configuration and Commands

#### @corp-ais/eqxjs-commander

```typescript
export * from '@corp-ais/eqxjs-commander';
```

**Provides:**
- Configuration management and loading
- Command handling patterns
- Domain configuration utilities

#### @corp-ais/eqxjs-decorator

```typescript
export * from '@corp-ais/eqxjs-decorator';
```

**Provides:**
- Custom decorators for method enhancement
- Metadata reflection utilities
- Framework-specific decorator patterns

### Transport and Communication

#### @corp-ais/eqxjs-transporter-http

```typescript
export * from '@corp-ais/eqxjs-transporter-http';
```

**Provides:**
- HTTP client capabilities and configuration
- Request/response handling utilities
- Transport layer abstractions

### Logging and Monitoring

#### @corp-ais/eqxjs-logger

```typescript
export * from '@corp-ais/eqxjs-logger';
```

**Provides:**
- Comprehensive logging services
- Log formatting and structuring
- Data masking capabilities
- Summary logging functionality

### Data Processing

#### @corp-ais/eqxjs-pipes

```typescript
export * from '@corp-ais/eqxjs-pipes';
```

**Provides:**
- Data transformation pipes
- Validation pipes for request processing

#### @corp-ais/eqxjs-utils

```typescript
export * from '@corp-ais/eqxjs-utils';
```

**Provides:**
- Utility functions and helpers
- Message context management
- Instance data management
- Common enums and constants

### Security and Error Handling

#### @corp-ais/eqxjs-exception

```typescript
export * from '@corp-ais/eqxjs-exception';
```

**Provides:**
- Exception handling framework
- Error formatting and standardization
- Custom exception types

#### @corp-ais/eqxjs-security

```typescript
export * from '@corp-ais/eqxjs-security';
```

**Provides:**
- Security utilities and validation
- Authentication helper functions
- Authorization patterns

---

## Usage Patterns

### Basic Application Setup

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import {
  FrameworkModule,
  AppInterceptor,
  HttpInterceptor,
  RestInterceptor,
} from '@corp-ais/eqxjs-stub';

@Module({
  imports: [
    FrameworkModule.register({
      configPath: './config',
      zone: process.env.NODE_ENV || 'development',
    }),
  ],
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: AppInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: HttpInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: RestInterceptor,
    },
  ],
})
export class AppModule {}
```

### Service with Entry Points

```typescript
import { Injectable } from '@nestjs/common';
import { Payload } from '@nestjs/microservices';
import {
  EntryPoint,
  ConsumerMasking,
  SetMessageMode,
  MessageTypeEnum,
} from '@corp-ais/eqxjs-stub';

@Injectable()
export class UserService {
  @EntryPoint('USER_CREATED')
  @ConsumerMasking(['password', 'email']) 
  async handleUserCreated(@Payload() data: CreateUserDto) {
    // Process user creation event
    console.log('Processing user creation:', data);
  }

  @EntryPoint('USER_UPDATED', {
    messageType: MessageTypeEnum.M3,
    manualConsumerLogging: true,
  })
  @SetMessageMode(MessageTypeEnum.M3)
  async handleUserUpdated(@Payload() data: UpdateUserDto) {
    // Process user update event
    console.log('Processing user update:', data);
  }
}
```
export class UserService {
  @EntryPoint("USER_CREATED")
  @ConsumerMasking(["password", "email"])
  async handleUserCreated(@Payload() data: CreateUserDto) {
    // Process user creation
  }

  @EntryPoint("USER_UPDATED", {
    messageType: MessageTypeEnum.M3,
    manualConsumerLogging: true,
  })
  @SetMessageMode(MessageTypeEnum.M3)
  async handleUserUpdated(@Payload() data: UpdateUserDto) {
    // Process user update
  }
}
```

### Graceful Shutdown Setup

```typescript
import { NestFactory } from '@nestjs/core';
import { GracefulShutdownService } from '@corp-ais/eqxjs-stub';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Get graceful shutdown service and configure
  const gracefulShutdown = app.get(GracefulShutdownService);
  gracefulShutdown.setup(app);

  // Enable shutdown hooks
  app.enableShutdownHooks();

  await app.listen(3000);
  console.log(`Application is running on: ${await app.getUrl()}`);
}

bootstrap().catch(err => {
  console.error('Failed to start application:', err);
  process.exit(1);
});
```

### Custom Service with Framework Integration

```typescript
import { Injectable, OnModuleDestroy } from '@nestjs/common';
import {
  FrameworkUtilService,
  GracefulShutdownService,
  CustomLoggerService
} from '@corp-ais/eqxjs-stub';

@Injectable()
export class BusinessService implements OnModuleDestroy {
  constructor(
    private readonly frameworkUtil: FrameworkUtilService,
    private readonly gracefulShutdown: GracefulShutdownService,
    private readonly logger: CustomLoggerService,
  ) {
    // Register cleanup tasks
    this.gracefulShutdown.addCleanupTask(async () => {
      await this.cleanup();
    });
  }

  async onModuleDestroy() {
    this.logger.info('BusinessService is being destroyed');
  }

  private async cleanup(): Promise<void> {
    this.logger.info('Cleaning up business service resources');
    // Add cleanup logic here
  }
}
```

---

## Environment Configuration Examples

### Development Configuration

```yaml
# development.config.yaml
app:
  name: "MyApp"
  component-name: "my-service"
  version: "1.0.0"
  description: "Development environment configuration"

log:
  level: "debug"
  detail:
    level: "trace"
    enable-console-logging: true
    enable-file-logging: false

security:
  enabled: false
  cors:
    enabled: true
    origins: ["http://localhost:3000"]

database:
  host: "localhost"
  port: 27017
  name: "myapp-dev"
  auth-type: "none"

kafka:
  brokers: ["localhost:9092"]
  client-id: "my-service-dev"
  group-id: "my-service-group-dev"
```

### Production Configuration

```yaml
# production.config.yaml
app:
  name: "MyApp"
  component-name: "my-service"
  version: "1.0.0"
  description: "Production environment configuration"

log:
  level: "info"
  detail:
    level: "warn"
    enable-console-logging: false
    enable-file-logging: true
    max-files: 10
    max-file-size: "20MB"

security:
  enabled: true
  jwt:
    secret: "${JWT_SECRET}"
    expiration: "1h"
  cors:
    enabled: true
    origins: ["https://myapp.com"]
  rate-limit:
    ttl: 60
    limit: 100

database:
  host: "${DB_HOST}"
  port: 27017
  name: "${DB_NAME}"
  username: "${DB_USER}"
  password: "${DB_PASS}"
  auth-type: "credentials"

kafka:
  brokers: ["${KAFKA_BROKER_1}", "${KAFKA_BROKER_2}"]
  client-id: "my-service-prod"
  group-id: "my-service-group-prod"
  ssl:
    enabled: true
```

### Test Configuration

```yaml
# test.config.yaml
app:
  name: "MyApp"
  component-name: "my-service"
  version: "1.0.0"
  description: "Test environment configuration"

log:
  level: "error"
  detail:
    level: "error"
    enable-console-logging: false
    enable-file-logging: false

security:
  enabled: false

database:
  host: "localhost"
  port: 27017
  name: "myapp-test"
  auth-type: "none"

kafka:
  brokers: ["localhost:9092"]
  client-id: "my-service-test"
  group-id: "my-service-group-test"
```

---

For detailed implementation examples and advanced usage patterns, see the [Framework Module Documentation](./FRAMEWORK_MODULE_DOCUMENTATION.md).
