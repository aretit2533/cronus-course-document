# @eqxjs/decorator

A collection of useful NestJS decorators that provide common functionality for parameter extraction, data transformation, logging metadata, and RPC context handling.

## Installation

```bash
npm install @eqxjs/decorator
```

## Description

This module re-exports all decorators from the decorator directory, providing:

- Logging metadata decorators for event tracking
- Data transformation decorators for cleaning and parsing
- Parameter extraction decorators for RPC contexts
- Session management decorators
- String manipulation utilities for parameter handling

These decorators are designed to simplify common tasks in NestJS applications, especially those dealing with RPC communications, data sanitization, and logging.

## Usage

### Basic Setup

```typescript
import { Controller, Post } from '@nestjs/common';
import { 
  Logs, 
  RemoveAtSymbol, 
  TrimAtSign, 
  GetSessionIdDecorator, 
  ToObjectDecorator 
} from '@eqxjs/decorator';

@Controller('api')
export class ApiController {
  @Post('process')
  @Logs('DATA_PROCESSING')
  processData(
    @RemoveAtSymbol() cleanData: any,
    @GetSessionIdDecorator() sessionId: string,
    @ToObjectDecorator() parsedBody: any
  ) {
    return {
      sessionId,
      cleanData,
      parsedBody
    };
  }
}
```

## API Reference

### Decorators

#### @Logs(event: string)

A decorator function that sets metadata for logging purposes.

**Parameters:**
- `event: string` - The event name to be logged

**Usage:**
```typescript
@Controller('users')
export class UsersController {
  @Post()
  @Logs('USER_CREATION')
  createUser(@Body() userData: CreateUserDto) {
    // The 'USER_CREATION' event will be available as metadata
    return this.userService.create(userData);
  }

  @Get(':id')
  @Logs('USER_RETRIEVAL')
  getUser(@Param('id') id: string) {
    return this.userService.findById(id);
  }
}
```

**Integration with Interceptors:**
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const reflector = new Reflector();
    const event = reflector.get<string>('event', context.getHandler());
    
    if (event) {
      console.log(`Processing event: ${event}`);
    }
    
    return next.handle();
  }
}
```

#### @RemoveAtSymbol()

A custom decorator that removes the '@' symbol from JSON keys in the request data.

**Parameters:**
- `targets: any` - Target object (not used in this implementation)
- `ctx: ExecutionContext` - Execution context providing access to request data

**Returns:** Modified data with '@' symbols removed from JSON keys

**Usage:**
```typescript
@Controller('data')
export class DataController {
  @Post('clean')
  cleanData(@RemoveAtSymbol() cleanedData: any) {
    // Input: { "@userId": "123", "@email": "test@example.com", "name": "John" }
    // Output: { "userId": "123", "email": "test@example.com", "name": "John" }
    return cleanedData;
  }
}
```

**Recursive Processing:**
The decorator recursively processes nested objects and arrays:

```typescript
// Input data:
{
  "@user": {
    "@id": "123",
    "@profile": {
      "@name": "John",
      "@settings": ["@theme", "@language"]
    }
  }
}

// Output data:
{
  "user": {
    "id": "123",
    "profile": {
      "name": "John",
      "settings": ["theme", "language"]
    }
  }
}
```

#### @TrimAtSign(targets: Array<string>)

A custom parameter decorator that trims the '@' character from specified target keys in the request data.

**Parameters:**
- `targets: Array<string>` - Array of strings representing the keys that need '@' character removed

**Returns:** Modified request data with '@' character removed from specified keys

**Usage:**
```typescript
@Controller('users')
export class UsersController {
  @Post('update')
  updateUser(@TrimAtSign(['@userId', '@email']) userData: any) {
    // Input: { "@userId": "123", "@email": "user@example.com", "name": "John" }
    // Output: { "userId": "123", "email": "user@example.com", "name": "John" }
    return this.userService.update(userData);
  }

  @Post('search')
  searchUsers(@TrimAtSign(['@query', '@filters']) searchData: any) {
    // Only specified keys will have '@' removed
    return this.userService.search(searchData);
  }
}
```

#### @GetSessionIdDecorator()

Custom decorator to extract the session ID from the request header in RPC context.

**Parameters:**
- `targets: any` - Target object (not used in this implementation)
- `ctx: ExecutionContext` - Execution context providing access to request data

**Returns:** Session ID from request header

**Usage:**
```typescript
@Controller('session')
export class SessionController {
  @Post('validate')
  validateSession(@GetSessionIdDecorator() sessionId: string) {
    console.log(`Validating session: ${sessionId}`);
    return this.sessionService.validate(sessionId);
  }

  @Get('info')
  getSessionInfo(@GetSessionIdDecorator() sessionId: string) {
    return {
      sessionId,
      isValid: this.sessionService.isValid(sessionId),
      expiry: this.sessionService.getExpiry(sessionId)
    };
  }
}
```

**Expected Request Structure:**
```json
{
  "header": {
    "session": "abc123def456",
    "timestamp": "2023-01-01T00:00:00Z"
  },
  "body": {
    "data": "..."
  }
}
```

#### @ToObjectDecorator()

A decorator that extracts and parses the body of an RPC request, converting JSON strings to objects.

**Parameters:**
- `targets: any` - Target parameters for the decorator
- `ctx: ExecutionContext` - Execution context of the RPC request

**Returns:** Parsed body of the RPC request (object if JSON string, original value otherwise)

**Usage:**
```typescript
@Controller('data')
export class DataController {
  @Post('parse')
  parseData(@ToObjectDecorator() parsedBody: any) {
    // If body is JSON string: '{"name": "John", "age": 30}'
    // Returns: { name: "John", age: 30 }
    
    // If body is already an object: { name: "John", age: 30 }
    // Returns: { name: "John", age: 30 }
    
    // If body is invalid JSON string: 'invalid json'
    // Returns: null
    
    return {
      processed: true,
      data: parsedBody
    };
  }

  @Post('flexible')
  handleFlexibleInput(@ToObjectDecorator() data: any) {
    if (data === null) {
      throw new BadRequestException('Invalid JSON format');
    }
    
    return this.dataService.process(data);
  }
}
```

**Error Handling:**
The decorator safely handles JSON parsing errors:

```typescript
// Valid JSON string input
body: '{"user": "john", "role": "admin"}'
// Output: { user: "john", role: "admin" }

// Invalid JSON string input
body: '{"user": "john", "role":}'
// Output: null

// Non-string input
body: { user: "john", role: "admin" }
// Output: { user: "john", role: "admin" }
```

## Advanced Usage

### Combining Multiple Decorators

```typescript
@Controller('advanced')
export class AdvancedController {
  @Post('process')
  @Logs('ADVANCED_PROCESSING')
  processAdvancedData(
    @RemoveAtSymbol() cleanData: any,
    @GetSessionIdDecorator() sessionId: string,
    @ToObjectDecorator() parsedBody: any,
    @TrimAtSign(['@requestId', '@timestamp']) metadata: any
  ) {
    return {
      sessionId,
      cleanData,
      parsedBody,
      metadata,
      timestamp: new Date().toISOString()
    };
  }
}
```

### Custom Validation with Decorators

```typescript
@Controller('validation')
export class ValidationController {
  @Post('submit')
  @Logs('DATA_SUBMISSION')
  submitData(
    @ToObjectDecorator() data: any,
    @GetSessionIdDecorator() sessionId: string
  ) {
    if (!data) {
      throw new BadRequestException('Invalid data format');
    }

    if (!sessionId) {
      throw new UnauthorizedException('Session ID required');
    }

    return this.dataService.submit(data, sessionId);
  }
}
```

### Error Handling

```typescript
@Controller('safe')
export class SafeController {
  @Post('process')
  safeProcess(
    @RemoveAtSymbol() data: any,
    @ToObjectDecorator() body: any
  ) {
    try {
      // Handle potential null values from ToObjectDecorator
      if (body === null) {
        return { error: 'Invalid JSON in request body' };
      }

      // Process cleaned data
      const result = this.processData(data, body);
      return { success: true, result };
    } catch (error) {
      return { 
        error: 'Processing failed', 
        message: error.message 
      };
    }
  }
}
```

### Integration with Guards and Interceptors

```typescript
@Injectable()
export class EventLoggingInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const event = this.reflector.get<string>('event', context.getHandler());
    
    if (event) {
      const startTime = Date.now();
      
      return next.handle().pipe(
        tap(() => {
          const duration = Date.now() - startTime;
          console.log(`Event '${event}' completed in ${duration}ms`);
        })
      );
    }
    
    return next.handle();
  }
}

// Usage
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: EventLoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

### RPC Context Handling

These decorators are specifically designed for RPC contexts. Ensure your application is configured for RPC communication:

```typescript
// Expected RPC message structure
{
  "header": {
    "session": "session-id-123",
    "messageType": "REQUEST",
    "timestamp": "2023-01-01T00:00:00Z"
  },
  "body": {
    "@userId": "123",
    "@email": "user@example.com",
    "data": '{"key": "value"}'
  }
}
```

## Features

### Automatic Data Cleaning
- Remove '@' symbols from JSON keys automatically
- Selective key cleaning with TrimAtSign
- Recursive processing for nested objects

### Session Management
- Extract session IDs from RPC headers
- Integrate with authentication and authorization systems
- Support for custom session validation

### Flexible Data Parsing
- Safe JSON string parsing with error handling
- Automatic type detection and conversion
- Null return for invalid JSON (no exceptions thrown)

### Event Logging Integration
- Metadata-based event tracking
- Integration with logging systems and interceptors
- Support for audit trails and monitoring

## Best Practices

### 1. Use Appropriate Decorators
Choose the right decorator for your use case:
- Use `@RemoveAtSymbol()` for complete '@' symbol removal
- Use `@TrimAtSign()` for selective key cleaning
- Use `@ToObjectDecorator()` for safe JSON parsing

### 2. Error Handling
Always handle potential null values from `@ToObjectDecorator()`:

```typescript
@Post('safe')
safeEndpoint(@ToObjectDecorator() data: any) {
  if (data === null) {
    throw new BadRequestException('Invalid JSON format');
  }
  // Process data...
}
```

### 3. Logging Integration
Use `@Logs()` decorator with interceptors for comprehensive event tracking:

```typescript
@Logs('CRITICAL_OPERATION')
@Post('critical')
criticalOperation(@Body() data: any) {
  // Critical business logic
}
```

### 4. Session Validation
Always validate session IDs extracted with `@GetSessionIdDecorator()`:

```typescript
@Post('secured')
securedEndpoint(@GetSessionIdDecorator() sessionId: string) {
  if (!this.sessionService.isValid(sessionId)) {
    throw new UnauthorizedException('Invalid session');
  }
  // Proceed with authenticated operation
}
```

## Dependencies

This library requires:
- `@nestjs/common` - For NestJS decorators and context handling

## TypeScript Support

Full TypeScript support with proper type definitions for all decorators and their parameters.

## License

ISC