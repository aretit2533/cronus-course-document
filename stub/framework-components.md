# Framework Components Reference

## Overview

The @corp-ais/eqxjs-stub framework provides a comprehensive set of components designed for building enterprise-grade NestJS applications. This document serves as a complete reference for all available components, their functionality, and usage patterns.

## 📋 Table of Contents

- [Core Module](#core-module)
- [Services](#services)
- [Decorators](#decorators)
- [Interceptors](#interceptors)
- [Utilities](#utilities)
- [EQXJS Ecosystem Modules](#eqxjs-ecosystem-modules)
- [Integration Patterns](#integration-patterns)
- [Best Practices](#best-practices)

---

## Core Module

### FrameworkModule

The central module that bootstraps the entire EQXJS framework ecosystem and provides core functionality.

#### Features
- **Automatic Module Registration**: Registers all 8 core EQXJS modules
- **Configuration Management**: YAML-based configuration with environment support
- **Dependency Injection**: Framework-specific DI container setup
- **Lifecycle Management**: Application startup and shutdown coordination
- **Context Management**: Domain-based service organization

#### Usage
```typescript
import { FrameworkModule } from '@corp-ais/eqxjs-stub';

@Module({
  imports: [
    FrameworkModule.forRoot({
      configPath: './config',
      environment: process.env.NODE_ENV || 'development'
    })
  ]
})
export class AppModule {}
```

#### Configuration Options
- `configPath`: Path to configuration directory
- `environment`: Environment name (development, staging, production)
- `globalPrefix`: Global API prefix
- `enableCors`: CORS configuration
- `logLevel`: Logging level configuration

### Configuration Management

Advanced YAML-based configuration system with environment-specific overrides and validation.

#### Features
- **Multi-Environment Support**: Separate configs for dev/staging/prod
- **Schema Validation**: Joi schema validation for configuration
- **Environment Variables**: Runtime environment variable injection
- **Hot Reloading**: Configuration reload without restart (development)
- **Encryption Support**: Encrypted configuration values

#### Configuration Structure
```yaml
app:
  name: "MyApplication"
  version: "1.0.0"
  component-name: "my-service"
  port: 3000

database:
  host: "${DB_HOST:localhost}"
  port: ${DB_PORT:5432}
  name: "${DB_NAME:myapp}"

logging:
  level: "info"
  format: "json"
  masking:
    enabled: true
    fields: ["password", "email", "ssn"]
```

---

## Services

### GracefulShutdownService

Manages application lifecycle with graceful startup and shutdown procedures.

#### Features
- **Signal Handling**: SIGTERM and SIGINT signal processing
- **Resource Cleanup**: Database connections, file handles, timers
- **Timeout Management**: Configurable shutdown timeout
- **Health Status**: Application readiness and liveness indicators
- **Custom Cleanup**: Register custom cleanup functions

#### Usage
```typescript
import { GracefulShutdownService } from '@corp-ais/eqxjs-stub';

@Injectable()
export class MyService {
  constructor(private shutdownService: GracefulShutdownService) {}

  async onModuleInit() {
    // Register cleanup function
    this.shutdownService.registerCleanup(async () => {
      await this.cleanup();
    });
  }

  private async cleanup() {
    // Custom cleanup logic
    console.log('Cleaning up resources...');
  }
}
```

#### Configuration
```yaml
gracefulShutdown:
  timeout: 30000  # 30 seconds
  signals: ["SIGTERM", "SIGINT"]
  forceExit: true
```

### FrameworkUtilService

Provides framework-specific utility functions and common operations.

#### Features
- **Message Context**: Request correlation and context management
- **Data Transformation**: Common data transformation utilities
- **Validation Helpers**: Schema validation and data sanitization
- **Crypto Utilities**: Hashing, encryption, and secure random generation
- **Date/Time Utilities**: Timezone handling and date formatting

#### Usage
```typescript
import { FrameworkUtilService } from '@corp-ais/eqxjs-stub';

@Injectable()
export class MyService {
  constructor(private utilService: FrameworkUtilService) {}

  async processRequest(data: any) {
    // Generate correlation ID
    const correlationId = this.utilService.generateCorrelationId();
    
    // Transform data
    const transformed = this.utilService.transformData(data);
    
    // Validate schema
    const validated = await this.utilService.validateSchema(transformed, schema);
    
    return { correlationId, data: validated };
  }
}
```

---

## Decorators

### @EntryPoint

Marks methods as event entry points for event-driven architecture patterns.

#### Features
- **Event Binding**: Automatic event listener registration
- **Message Routing**: Route events to appropriate handlers
- **Error Handling**: Built-in error handling and retry logic
- **Metrics Collection**: Automatic metrics for event processing
- **Logging Integration**: Structured logging for events

#### Usage
```typescript
import { EntryPoint } from '@corp-ais/eqxjs-stub';

@Controller()
export class EventController {
  @EntryPoint('user.created', { 
    retry: { attempts: 3, delay: 1000 },
    timeout: 5000 
  })
  async handleUserCreated(event: UserCreatedEvent) {
    // Handle user creation event
    console.log('User created:', event.userId);
  }
}
```

#### Options
- `retry`: Retry configuration (attempts, delay, backoff)
- `timeout`: Event processing timeout
- `deadLetter`: Dead letter queue configuration
- `batch`: Batch processing options

### @ConsumerMasking

Provides secure logging by masking sensitive fields in log output.

#### Features
- **Field Masking**: Mask specific fields in objects
- **Pattern Matching**: Regex patterns for dynamic field detection
- **Custom Maskers**: Custom masking functions
- **Deep Object Support**: Nested object and array masking
- **Performance Optimized**: Minimal overhead for production

#### Usage
```typescript
import { ConsumerMasking } from '@corp-ais/eqxjs-stub';

@Controller()
export class UserController {
  @ConsumerMasking(['password', 'email', 'ssn'])
  @Post('/users')
  async createUser(@Body() userData: CreateUserDto) {
    // Logs will mask password, email, and ssn fields
    return await this.userService.create(userData);
  }
}
```

#### Advanced Configuration
```typescript
@ConsumerMasking({
  fields: ['password', 'creditCard'],
  patterns: [/.*token.*/i, /.*key.*/i],
  maskWith: '***MASKED***',
  preserveLength: true
})
```

### @DisableConsumerLogging

Controls logging behavior for specific methods or classes.

#### Features
- **Selective Logging**: Disable logging for sensitive operations
- **Conditional Disabling**: Environment-based logging control
- **Performance**: Reduce logging overhead for high-frequency operations
- **Security**: Prevent logging of sensitive operations

#### Usage
```typescript
import { DisableConsumerLogging } from '@corp-ais/eqxjs-stub';

@Controller()
export class AuthController {
  @DisableConsumerLogging(true)
  @Post('/login')
  async login(@Body() credentials: LoginDto) {
    // This method will not generate consumer logs
    return await this.authService.login(credentials);
  }

  @DisableConsumerLogging(process.env.NODE_ENV === 'production')
  @Get('/debug')
  async debug() {
    // Logging disabled only in production
    return this.diagnosticService.getDebugInfo();
  }
}
```

### @SetMessageMode

Configures message processing modes for different types of operations.

#### Features
- **Processing Modes**: Synchronous, asynchronous, batch processing
- **Message Types**: Different handling for different message types
- **Performance Optimization**: Optimized processing based on message characteristics
- **Resource Management**: Efficient resource allocation per mode

#### Usage
```typescript
import { SetMessageMode, MessageTypeEnum } from '@corp-ais/eqxjs-stub';

@Controller()
export class MessageController {
  @SetMessageMode(MessageTypeEnum.ASYNC)
  @Post('/async-process')
  async processAsync(@Body() message: AsyncMessage) {
    // Processed asynchronously
    return await this.messageService.processAsync(message);
  }

  @SetMessageMode(MessageTypeEnum.BATCH)
  @Post('/batch-process')
  async processBatch(@Body() messages: BatchMessage[]) {
    // Processed in batches
    return await this.messageService.processBatch(messages);
  }
}
```

#### Message Types
- `SYNC`: Synchronous processing
- `ASYNC`: Asynchronous processing
- `BATCH`: Batch processing
- `STREAM`: Stream processing
- `EVENT`: Event-driven processing

---

## Interceptors

### AppInterceptor

Global application interceptor for request/response handling and cross-cutting concerns.

#### Features
- **Request/Response Logging**: Comprehensive request/response logging
- **Performance Metrics**: Request duration and performance monitoring
- **Error Handling**: Global error handling and transformation
- **Context Management**: Request context setup and cleanup
- **Security Headers**: Automatic security header injection

#### Usage
```typescript
import { AppInterceptor } from '@corp-ais/eqxjs-stub';

@UseInterceptors(AppInterceptor)
@Controller()
export class ApiController {
  // All methods will use AppInterceptor
}

// Or globally
app.useGlobalInterceptors(new AppInterceptor());
```

#### Configuration
```yaml
interceptors:
  app:
    logging:
      requests: true
      responses: true
      errors: true
    metrics:
      enabled: true
      histogram: true
    security:
      headers: true
      cors: true
```

### HttpInterceptor

Specialized interceptor for HTTP client requests and external API calls.

#### Features
- **Request/Response Transformation**: Automatic data transformation
- **Retry Logic**: Configurable retry mechanisms
- **Circuit Breaker**: Circuit breaker pattern implementation
- **Caching**: Response caching for GET requests
- **Rate Limiting**: Client-side rate limiting

#### Usage
```typescript
import { HttpInterceptor } from '@corp-ais/eqxjs-stub';

@Injectable()
export class ExternalApiService {
  constructor(
    @Inject(HttpService) private httpService: HttpService
  ) {
    // Configure interceptor
    this.httpService.axiosRef.interceptors.request.use(
      HttpInterceptor.requestInterceptor()
    );
    this.httpService.axiosRef.interceptors.response.use(
      HttpInterceptor.responseInterceptor()
    );
  }
}
```

#### Configuration
```yaml
http:
  interceptor:
    retry:
      attempts: 3
      delay: 1000
      exponentialBackoff: true
    circuitBreaker:
      enabled: true
      threshold: 5
      timeout: 30000
    cache:
      enabled: true
      ttl: 300000  # 5 minutes
```

### RestInterceptor

REST API specific interceptor for standardized API responses and error handling.

#### Features
- **Response Standardization**: Consistent API response format
- **Error Transformation**: Standard error response format
- **Pagination Support**: Automatic pagination handling
- **Content Negotiation**: Accept header processing
- **API Versioning**: Version header handling

#### Usage
```typescript
import { RestInterceptor } from '@corp-ais/eqxjs-stub';

@UseInterceptors(RestInterceptor)
@Controller('api/v1')
export class RestApiController {
  @Get('/users')
  async getUsers(@Query() query: GetUsersQuery) {
    const users = await this.userService.findAll(query);
    // RestInterceptor will standardize the response
    return users;
  }
}
```

#### Standard Response Format
```typescript
{
  success: true,
  data: any,
  message?: string,
  pagination?: {
    page: number,
    limit: number,
    total: number,
    totalPages: number
  },
  meta?: {
    timestamp: string,
    correlationId: string,
    version: string
  }
}
```

---

## Utilities

### Health Utilities

Comprehensive health check utilities for application monitoring.

#### Features
- **Built-in Checks**: Database, Redis, HTTP services, disk space
- **Custom Checks**: Register custom health indicators
- **Health Endpoints**: Standard health check endpoints
- **Monitoring Integration**: Prometheus metrics integration
- **Alerting**: Health status change notifications

#### Usage
```typescript
import { HealthUtilities, HealthIndicator } from '@corp-ais/eqxjs-stub';

@Injectable()
export class CustomHealthIndicator extends HealthIndicator {
  key = 'custom-service';

  async check(): Promise<HealthCheckResult> {
    const isHealthy = await this.checkCustomService();
    
    return this.getStatus(this.key, isHealthy, {
      message: isHealthy ? 'Service is healthy' : 'Service is down'
    });
  }
  
  private async checkCustomService(): Promise<boolean> {
    // Custom health check logic
    return true;
  }
}
```

#### Available Health Checks
- **Database**: Connection and query performance
- **Redis**: Connection and response time
- **HTTP Services**: External service availability
- **Disk Space**: Available disk space
- **Memory**: Memory usage and availability
- **CPU**: CPU usage monitoring

### Database Utilities

Database management and optimization utilities.

#### Features
- **Connection Management**: Connection pooling and lifecycle
- **Migration Support**: Database migration utilities
- **Query Optimization**: Query performance monitoring
- **Transaction Management**: Distributed transaction support
- **Schema Validation**: Database schema validation

#### Usage
```typescript
import { DatabaseUtilities } from '@corp-ais/eqxjs-stub';

@Injectable()
export class DataService {
  constructor(private dbUtils: DatabaseUtilities) {}

  async performComplexOperation(data: any) {
    return await this.dbUtils.withTransaction(async (manager) => {
      const result1 = await manager.save(EntityA, data.entityA);
      const result2 = await manager.save(EntityB, data.entityB);
      return { result1, result2 };
    });
  }
}
```

### Validation Utilities

Joi schema validation utilities with enhanced error handling.

#### Features
- **Schema Registry**: Centralized schema management
- **Custom Validators**: Domain-specific validation rules
- **Error Formatting**: User-friendly validation errors
- **Async Validation**: Support for async validation rules
- **Conditional Validation**: Context-aware validation

#### Usage
```typescript
import { ValidationUtilities, JoiSchema } from '@corp-ais/eqxjs-stub';

const userSchema = JoiSchema.object({
  email: JoiSchema.string().email().required(),
  password: JoiSchema.string().min(8).pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  age: JoiSchema.number().min(18).max(120)
});

@Injectable()
export class UserService {
  constructor(private validator: ValidationUtilities) {}

  async createUser(userData: any) {
    const validatedData = await this.validator.validate(userData, userSchema);
    return await this.userRepository.save(validatedData);
  }
}
```

---

## EQXJS Ecosystem Modules

### Configuration & Commands

#### @corp-ais/eqxjs-commander
- Configuration management and validation
- Command-line interface utilities
- Environment variable processing
- Configuration schema validation

#### @corp-ais/eqxjs-decorator
- Custom decorator utilities
- Metadata management
- Reflection helpers
- Decorator composition patterns

### Transport & Communication

#### @corp-ais/eqxjs-transporter-http
- HTTP transport layer
- RESTful client capabilities
- Request/response transformation
- Connection pooling and management

### Logging & Monitoring

#### @corp-ais/eqxjs-logger
- Structured logging with JSON format
- Log level management
- Field masking and sanitization
- Performance logging and metrics

### Data Processing

#### @corp-ais/eqxjs-pipes
- Data transformation pipelines
- Validation pipes with Joi integration
- Custom pipe development utilities
- Async pipe processing

#### @corp-ais/eqxjs-utils
- Common utility functions
- Message context management
- Date/time utilities
- Data manipulation helpers

### Security & Error Handling

#### @corp-ais/eqxjs-security
- Authentication and authorization
- Security utilities and helpers
- Input sanitization
- Security header management

#### @corp-ais/eqxjs-exception
- Exception handling framework
- Custom error types
- Error transformation and formatting
- Global exception filters

---

## Integration Patterns

### Microservices Architecture

```typescript
@Module({
  imports: [
    FrameworkModule.forRoot({
      configPath: './config',
      microservice: true
    }),
    // Other modules
  ],
  providers: [
    {
      provide: 'MICROSERVICE_OPTIONS',
      useValue: {
        transport: Transport.RMQ,
        options: { urls: ['amqp://localhost:5672'] }
      }
    }
  ]
})
export class MicroserviceModule {}
```

### Event-Driven Architecture

```typescript
@Injectable()
export class EventDrivenService {
  @EntryPoint('order.created', { retry: { attempts: 3 } })
  async handleOrderCreated(event: OrderCreatedEvent) {
    await this.processOrder(event);
  }

  @EntryPoint('payment.completed')
  async handlePaymentCompleted(event: PaymentEvent) {
    await this.fulfillOrder(event.orderId);
  }
}
```

### REST API with Full Observability

```typescript
@UseInterceptors(RestInterceptor, AppInterceptor)
@ConsumerMasking(['password', 'creditCard'])
@Controller('api/v1/users')
export class UserApiController {
  @Post()
  @SetMessageMode(MessageTypeEnum.ASYNC)
  async createUser(@Body() userData: CreateUserDto) {
    return await this.userService.create(userData);
  }

  @Get(':id')
  @DisableConsumerLogging(false)
  async getUser(@Param('id') id: string) {
    return await this.userService.findById(id);
  }
}
```

---

## Best Practices

### Configuration Management

1. **Environment Separation**: Use separate config files for each environment
2. **Secret Management**: Use environment variables for secrets
3. **Validation**: Always validate configuration with Joi schemas
4. **Documentation**: Document all configuration options

### Service Design

1. **Single Responsibility**: Each service should have a single, well-defined purpose
2. **Dependency Injection**: Use DI for all dependencies
3. **Error Handling**: Implement comprehensive error handling
4. **Testing**: Write unit and integration tests for all services

### Logging and Monitoring

1. **Structured Logging**: Use JSON format for all logs
2. **Field Masking**: Always mask sensitive information
3. **Correlation IDs**: Use correlation IDs for request tracing
4. **Health Checks**: Implement comprehensive health checks

### Security

1. **Input Validation**: Validate all inputs with Joi schemas
2. **Output Sanitization**: Sanitize all outputs
3. **Authentication**: Implement proper authentication
4. **Authorization**: Use role-based access control

### Performance

1. **Connection Pooling**: Use connection pooling for databases
2. **Caching**: Implement appropriate caching strategies
3. **Async Processing**: Use async processing for heavy operations
4. **Resource Management**: Properly manage resources and cleanup

---

## Troubleshooting

### Common Issues

#### Configuration Not Loading
- Check file paths and permissions
- Verify YAML syntax
- Ensure environment variables are set

#### Health Checks Failing
- Check database connections
- Verify external service availability
- Review health check timeouts

#### Performance Issues
- Review connection pool settings
- Check for memory leaks
- Monitor CPU and memory usage

#### Logging Issues
- Verify log level configuration
- Check masking configuration
- Review log format settings

### Debug Mode

Enable debug mode for detailed logging:

```yaml
logging:
  level: "debug"
  debug:
    enabled: true
    components: ["framework", "interceptors", "health"]
```

### Support Resources

- **GitHub Issues**: Report bugs and feature requests
- **Documentation**: Comprehensive documentation and examples
- **Training**: Complete training materials available
- **Community**: Active developer community and support

---

**This completes the Framework Components Reference. For detailed API documentation, see [API_REFERENCE.md](API_REFERENCE.md).**