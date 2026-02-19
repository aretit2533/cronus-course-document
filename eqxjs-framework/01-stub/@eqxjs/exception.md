# @eqxjs/exception

A comprehensive exception handling library for NestJS applications that provides structured error classes with severity levels, result types, and automatic error event message generation.

## Installation

```bash
npm install @eqxjs/exception
```

## Description

This module provides a comprehensive set of exception classes and utilities for error handling in enterprise applications. It includes:

- Custom exception classes with specific error codes and HTTP status codes
- Severity levels for error classification and prioritization
- Result types for categorizing errors by source and nature
- Automatic error event message generation with domain context
- Global Error prototype extension for consistent error formatting

## Usage

### Basic Setup

```typescript
import { Module } from '@nestjs/common';
import { 
  MissingOrIncorrectParameterError, 
  AccessDeniedError,
  ResultType,
  Severity 
} from '@eqxjs/exception';

@Injectable()
export class UserService {
  findUser(id: string) {
    if (!id) {
      throw new MissingOrIncorrectParameterError();
    }
    
    if (!this.hasPermission(id)) {
      throw new AccessDeniedError();
    }
    
    return this.userRepository.findById(id);
  }
}
```

### Error Event Message Generation

```typescript
import { AccessDeniedError } from '@eqxjs/exception';

try {
  // Some operation that might fail
  throw new AccessDeniedError();
} catch (error) {
  // Convert error to event message format
  const eventMessage = error.toErrorEventMessage('USER_SERVICE');
  
  console.log(eventMessage);
  // Output: {
  //   code: "40100",
  //   reason: "[USER_SERVICE] Access Denied",
  //   status: "401"
  // }
}
```

### Exception Filtering

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Response } from 'express';
import { AccessDeniedError, ResultType, Severity } from '@eqxjs/exception';

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();

    // Handle custom application exceptions
    if (exception instanceof AccessDeniedError) {
      response.status(exception.status).json({
        error: {
          code: exception.code,
          message: exception.message,
          resultType: exception.resultType,
          severity: exception.severity,
          timestamp: new Date().toISOString()
        }
      });
    } else {
      // Handle other exceptions
      response.status(500).json({
        error: {
          message: 'Internal Server Error',
          timestamp: new Date().toISOString()
        }
      });
    }
  }
}
```

## API Reference

### Exception Classes

#### MissingOrIncorrectParameterError

Error for missing or incorrect parameters in requests.

**Properties:**
- `code: 40000`
- `status: 400`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MINOR_ISSUE`
- `message: "Missing or incorrect parameter"`

**Usage:**
```typescript
if (!userId) {
  throw new MissingOrIncorrectParameterError();
}
```

#### RequestIncorrectFormatError

Error for requests with incorrect format.

**Properties:**
- `code: 40001`
- `status: 400`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MINOR_ISSUE`
- `message: "Request incorrect format"`

**Usage:**
```typescript
if (!this.validateRequestFormat(data)) {
  throw new RequestIncorrectFormatError();
}
```

#### NoDocumentHasBeenEditedError

Error when an operation expects a document to have been edited but none has been.

**Properties:**
- `code: 40002`
- `status: 400`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MINOR_ISSUE`
- `message: "No document has been edited"`

**Usage:**
```typescript
if (!hasChanges) {
  throw new NoDocumentHasBeenEditedError();
}
```

#### UnableToAccessApiError

Error when an external API cannot be accessed.

**Properties:**
- `code: 40003`
- `status: 400`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MAJOR_ISSUE`
- `message: "Unable to access service"`

**Usage:**
```typescript
try {
  await this.externalApiService.call();
} catch (error) {
  throw new UnableToAccessApiError(error.stack);
}
```

#### CannotProcessIncomingRequestError

Error when the server cannot process an incoming request.

**Properties:**
- `code: 40004`
- `status: 400`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MAJOR_ISSUE`
- `message: "Unable to process the request from the client"`

**Usage:**
```typescript
if (this.serverOverloaded()) {
  throw new CannotProcessIncomingRequestError();
}
```

#### PayloadTooLargeError

Error when request payload exceeds size limits.

**Properties:**
- `code: 41300`
- `status: 413`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MINOR_ISSUE`
- `message: "Payload Too Large"`

**Usage:**
```typescript
if (requestSize > MAX_PAYLOAD_SIZE) {
  throw new PayloadTooLargeError();
}
```

#### TooManyRequestError

Error for rate limiting when too many requests are made.

**Properties:**
- `code: 42900`
- `status: 429`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MAJOR_ISSUE`
- `message: "Too Many Request"`

**Usage:**
```typescript
if (this.rateLimiter.isExceeded(clientId)) {
  throw new TooManyRequestError();
}
```

#### AccessDeniedError

Error for unauthorized access attempts.

**Properties:**
- `code: 40100`
- `status: 401`
- `resultType: ResultType.CLIENT_ERROR`
- `severity: Severity.MAJOR_ISSUE`
- `message: "Access Denied"`

**Usage:**
```typescript
if (!this.authService.hasPermission(user, resource)) {
  throw new AccessDeniedError();
}
```

### Enums

#### ResultType

Categorizes errors by their source and nature.

```typescript
export enum ResultType {
  HEALTHY = 'HEALTHY',           // Successful operations
  CLIENT_ERROR = 'CLIENT_ERROR', // Errors caused by client
  SYSTEM_ERROR = 'SYSTEM_ERROR', // Errors caused by system
  BUSINESS_ERROR = 'BUSINESS_ERROR' // Errors in business logic
}
```

**Usage:**
```typescript
// Check error category
if (error.resultType === ResultType.CLIENT_ERROR) {
  // Handle client errors differently
  logClientError(error);
} else if (error.resultType === ResultType.SYSTEM_ERROR) {
  // Handle system errors
  alertOperations(error);
}
```

#### Severity

Defines the severity level of errors for prioritization.

```typescript
export enum Severity {
  NORMAL = 'NORMAL',                   // Normal, non-urgent issue
  NOTICE = 'NOTICE',                   // Issue to note, not urgent
  MINOR_ISSUE = 'MINOR_ISSUE',         // Minor issue, may need attention
  MAJOR_ISSUE = 'MAJOR_ISSUE',         // Major issue, needs prompt attention
  CRITICAL_ISSUE = 'CRITICAL_ISSUE',   // Critical, needs immediate attention
  SYSTEM_DOWN = 'SYSTEM_DOWN'          // System down, urgent resolution needed
}
```

**Usage:**
```typescript
// Prioritize error handling based on severity
switch (error.severity) {
  case Severity.CRITICAL_ISSUE:
  case Severity.SYSTEM_DOWN:
    await this.alertManager.sendImmediateAlert(error);
    break;
  case Severity.MAJOR_ISSUE:
    await this.alertManager.sendUrgentAlert(error);
    break;
  case Severity.MINOR_ISSUE:
    this.logger.warn(error.message);
    break;
}
```

### Global Error Extension

#### Error.prototype.toErrorEventMessage(domain?: string)

Extended Error prototype method for consistent error event message formatting.

**Parameters:**
- `domain?: string` - Optional domain context for the error

**Returns:** Object containing formatted error information:
```typescript
{
  code: string;    // Error code as string
  reason: string;  // Formatted reason with domain context
  status: string;  // HTTP status code as string
}
```

**Usage:**
```typescript
try {
  throw new AccessDeniedError();
} catch (error) {
  const eventMessage = error.toErrorEventMessage('AUTH_SERVICE');
  // Returns: {
  //   code: "40100",
  //   reason: "[AUTH_SERVICE] Access Denied", 
  //   status: "401"
  // }
}
```

## Advanced Usage

### Custom Exception Classes

Create custom exceptions by extending the base Error class:

```typescript
import { ResultType, Severity } from '@eqxjs/exception';

export class CustomBusinessError extends Error {
  public code: number;
  public status: number;
  public resultType: ResultType;
  public severity: Severity;

  constructor(message?: string, stack?: string) {
    super(message || 'Custom business logic error');
    this.name = this.constructor.name;
    this.code = 50001;
    this.status = 500;
    this.stack = stack;
    this.resultType = ResultType.BUSINESS_ERROR;
    this.severity = Severity.MAJOR_ISSUE;
  }
}
```

### Error Monitoring and Alerting

```typescript
import { Injectable } from '@nestjs/common';
import { Severity, ResultType } from '@eqxjs/exception';

@Injectable()
export class ErrorMonitoringService {
  handleError(error: any, context?: string) {
    const eventMessage = error.toErrorEventMessage(context);
    
    // Log error details
    this.logger.error({
      ...eventMessage,
      severity: error.severity,
      resultType: error.resultType,
      stack: error.stack
    });
    
    // Send alerts based on severity
    this.sendAlertsBasedOnSeverity(error);
    
    // Update metrics
    this.updateErrorMetrics(error);
  }

  private sendAlertsBasedOnSeverity(error: any) {
    switch (error.severity) {
      case Severity.CRITICAL_ISSUE:
      case Severity.SYSTEM_DOWN:
        this.sendImmediateAlert(error);
        break;
      case Severity.MAJOR_ISSUE:
        this.sendUrgentAlert(error);
        break;
      case Severity.MINOR_ISSUE:
      case Severity.NOTICE:
        this.logForReview(error);
        break;
    }
  }
}
```

### Error Response Standardization

```typescript
import { Injectable } from '@nestjs/common';
import { ResultType } from '@eqxjs/exception';

@Injectable()
export class ErrorResponseService {
  formatErrorResponse(error: any, requestId?: string) {
    const eventMessage = error.toErrorEventMessage();
    
    return {
      error: {
        code: eventMessage.code,
        message: eventMessage.reason,
        status: eventMessage.status,
        resultType: error.resultType || ResultType.SYSTEM_ERROR,
        severity: error.severity,
        requestId,
        timestamp: new Date().toISOString()
      },
      success: false,
      data: null
    };
  }

  isClientError(error: any): boolean {
    return error.resultType === ResultType.CLIENT_ERROR;
  }

  isSystemError(error: any): boolean {
    return error.resultType === ResultType.SYSTEM_ERROR;
  }

  isCritical(error: any): boolean {
    return error.severity === Severity.CRITICAL_ISSUE || 
           error.severity === Severity.SYSTEM_DOWN;
  }
}
```

### Global Exception Filter

```typescript
import { 
  ExceptionFilter, 
  Catch, 
  ArgumentsHost, 
  HttpException,
  Logger
} from '@nestjs/common';
import { Request, Response } from 'express';
import { ResultType, Severity } from '@eqxjs/exception';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: any, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // Determine error details
    let status = 500;
    let code = '50000';
    let resultType = ResultType.SYSTEM_ERROR;
    let severity = Severity.MAJOR_ISSUE;

    if (exception.status && exception.code) {
      // Custom application exception
      status = exception.status;
      code = exception.code.toString();
      resultType = exception.resultType;
      severity = exception.severity;
    } else if (exception instanceof HttpException) {
      // NestJS HTTP exception
      status = exception.getStatus();
      code = status.toString();
      resultType = status < 500 ? ResultType.CLIENT_ERROR : ResultType.SYSTEM_ERROR;
      severity = status >= 500 ? Severity.MAJOR_ISSUE : Severity.MINOR_ISSUE;
    }

    // Generate error event message
    const eventMessage = exception.toErrorEventMessage ? 
      exception.toErrorEventMessage('GLOBAL') : 
      {
        code,
        reason: `[GLOBAL] ${exception.message || 'Unknown error'}`,
        status: status.toString()
      };

    // Log error
    this.logger.error({
      ...eventMessage,
      resultType,
      severity,
      path: request.url,
      method: request.method,
      stack: exception.stack
    });

    // Send response
    response.status(status).json({
      error: {
        ...eventMessage,
        resultType,
        severity,
        path: request.url,
        timestamp: new Date().toISOString()
      }
    });
  }
}
```

### Integration with Validation

```typescript
import { Injectable, BadRequestException } from '@nestjs/common';
import { MissingOrIncorrectParameterError, RequestIncorrectFormatError } from '@eqxjs/exception';

@Injectable()
export class ValidationService {
  validateUserInput(data: any) {
    if (!data) {
      throw new MissingOrIncorrectParameterError();
    }

    if (!data.email || !this.isValidEmail(data.email)) {
      throw new RequestIncorrectFormatError();
    }

    if (!data.password || data.password.length < 8) {
      throw new MissingOrIncorrectParameterError();
    }

    return true;
  }

  private isValidEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
}
```

## Error Categorization

### By HTTP Status Code
- **400 series**: Client errors (bad requests, authentication, authorization)
- **401**: Authentication errors (`AccessDeniedError`)
- **413**: Payload size errors (`PayloadTooLargeError`)
- **429**: Rate limiting errors (`TooManyRequestError`)

### By Result Type
- **CLIENT_ERROR**: Issues with client requests
- **SYSTEM_ERROR**: Internal system failures
- **BUSINESS_ERROR**: Business logic violations
- **HEALTHY**: Successful operations

### By Severity Level
- **SYSTEM_DOWN**: Immediate attention required
- **CRITICAL_ISSUE**: Urgent resolution needed
- **MAJOR_ISSUE**: Prompt attention required
- **MINOR_ISSUE**: May need attention
- **NOTICE**: Should be noted
- **NORMAL**: Non-urgent

## Best Practices

### 1. Use Specific Exception Types
Choose the most specific exception type for each error scenario:

```typescript
// Good - specific exception
if (!userId) {
  throw new MissingOrIncorrectParameterError();
}

// Avoid - generic exception
if (!userId) {
  throw new Error('Bad request');
}
```

### 2. Include Stack Traces for System Errors
Always include stack traces for debugging:

```typescript
try {
  await this.databaseService.save(data);
} catch (error) {
  throw new UnableToAccessApiError(error.stack);
}
```

### 3. Use Appropriate Severity Levels
Match severity to business impact:

```typescript
// Critical - affects all users
throw new SystemDownError();

// Major - affects specific functionality
throw new AccessDeniedError();

// Minor - affects single request
throw new MissingOrIncorrectParameterError();
```

### 4. Consistent Error Event Messages
Always use the `toErrorEventMessage()` method for logging:

```typescript
try {
  // risky operation
} catch (error) {
  const eventMessage = error.toErrorEventMessage('USER_SERVICE');
  this.logger.error(eventMessage);
}
```

## Dependencies

This library requires:
- `@nestjs/common` - For NestJS integration (optional)
- No additional runtime dependencies

## TypeScript Support

Full TypeScript support with comprehensive type definitions for all exception classes, enums, and interfaces.

## License

ISC