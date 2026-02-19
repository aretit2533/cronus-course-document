# @eqxjs/transporter-http

A powerful HTTP transport layer for NestJS applications that provides enhanced Axios functionality with automatic retry logic, comprehensive error handling, and customizable request/response processing.

## Installation

```bash
npm install @eqxjs/transporter-http
```

## Description

This module serves as the entry point for the HTTP client library, providing:

- Enhanced HTTP client service with retry capabilities
- Comprehensive error handling for HTTP operations
- Response processing utilities
- Custom Axios service with advanced configuration options
- HTTP exception classes for better error management

## Usage

### Basic Setup

```typescript
import { Module } from '@nestjs/common';
import { HttpClientModule } from '@eqxjs/transporter-http';

@Module({
  imports: [
    HttpClientModule.register()
  ]
})
export class AppModule {}
```

### Using the Custom Axios Service

```typescript
import { Injectable } from '@nestjs/common';
import { CustomAxiosService } from '@eqxjs/transporter-http';

@Injectable()
export class ApiService {
  constructor(private readonly httpClient: CustomAxiosService) {}

  async getData() {
    // Simple HTTP request
    const response = await this.httpClient.request({
      method: 'GET',
      url: 'https://api.example.com/data',
      headers: {
        'Content-Type': 'application/json'
      }
    });

    return response.data;
  }

  async getDataWithRetry() {
    // HTTP request with retry logic
    const response = await this.httpClient.request({
      method: 'GET',
      url: 'https://api.example.com/data',
      timeout: 5000
    }, 'GET_USER_DATA', {
      retries: 3,
      retryOnHttpStatus: 500,
      retryDelay: () => 1000, // 1 second delay
      retryCondition: (error) => {
        // Retry on server errors (5xx) or network errors
        return error.response?.status >= 500 || !error.response;
      },
      onRetry: (error) => {
        console.log(`Retrying request due to: ${error.message}`);
      }
    });

    return response.data;
  }
}
```

### Error Handling

```typescript
import { Injectable } from '@nestjs/common';
import { CustomAxiosService, UnExpectedHttpError } from '@eqxjs/transporter-http';

@Injectable()
export class SafeApiService {
  constructor(private readonly httpClient: CustomAxiosService) {}

  async safeRequest() {
    try {
      const response = await this.httpClient.request({
        method: 'POST',
        url: 'https://api.example.com/users',
        data: { name: 'John Doe' }
      });

      return response.data;
    } catch (error) {
      if (error instanceof UnExpectedHttpError) {
        console.error('Unexpected HTTP error:', error.message);
        console.error('Error code:', error.code);
      } else {
        console.error('Other error:', error);
      }
      throw error;
    }
  }
}
```

## API Reference

### Classes

#### HttpClientModule

Main module class that provides HTTP client functionality for NestJS applications.

**Methods:**
- `static register(): DynamicModule` - Registers the HttpClientModule and returns a configured dynamic module

**Example:**
```typescript
@Module({
  imports: [HttpClientModule.register()]
})
export class MyModule {}
```

#### CustomAxiosService

Enhanced Axios service that provides HTTP request capabilities with retry logic and error handling.

**Constructor:**
```typescript
constructor(
  httpService: HttpService,
  utilService: HttpHelperService
)
```

**Methods:**

##### request(axiosConfig: AxiosRequestConfig, cmd?: string, retryOption?: retryOptionType): Promise<CustomAxiosResponse>

Sends an HTTP request with optional retry configuration.

**Parameters:**
- `axiosConfig: AxiosRequestConfig` - The Axios request configuration object
- `cmd?: string` - Optional command string for logging/tracking purposes
- `retryOption?: retryOptionType` - Optional retry configuration

**Returns:** `Promise<CustomAxiosResponse>` - Promise resolving to the HTTP response

**Throws:** `UnExpectedHttpError` - When request fails and no response is received

**Example:**
```typescript
// Basic request
const response = await customAxios.request({
  method: 'GET',
  url: 'https://api.example.com/users'
});

// Request with retry
const responseWithRetry = await customAxios.request({
  method: 'POST',
  url: 'https://api.example.com/users',
  data: userData
}, 'CREATE_USER', {
  retries: 3,
  retryOnHttpStatus: 500,
  retryDelay: () => 2000
});
```

#### HttpHelperService

Service providing helper methods for processing HTTP responses.

**Methods:**

##### responseServiceApi(response): CustomAxiosResponse

Processes the response from service API and extracts relevant information.

**Parameters:**
- `response` - The response object from the service API containing:
  - `status: number` - HTTP status code
  - `headers: object` - Response headers
  - `data: any` - Response data payload
  - `maxRetry: number` - Maximum retry attempts allowed
  - `retryCount: number` - Current retry attempts made

**Returns:** `CustomAxiosResponse` - Processed response object

**Example:**
```typescript
const processedResponse = httpHelper.responseServiceApi(rawResponse);
console.log(`Status: ${processedResponse.status}`);
console.log(`Retry attempts: ${processedResponse.retryCount}/${processedResponse.maxRetry}`);
```

### Types and Interfaces

#### retryOptionType

Configuration options for HTTP request retry behavior.

```typescript
export type retryOptionType = {
  retries: number;                                              // Number of retry attempts
  retryOnHttpStatus?: number;                                   // HTTP status code that triggers retry
  retryDelay?: () => number;                                    // Function returning delay in milliseconds
  retryCondition?: (e: AxiosError<unknown, any>) => boolean;    // Custom retry condition function
  onRetry?: (e: AxiosError<unknown, any>) => void;             // Callback function called before each retry
}
```

**Properties:**
- `retries: number` - **Required.** Number of retry attempts to make
- `retryOnHttpStatus?: number` - **Optional.** Specific HTTP status code that should trigger a retry
- `retryDelay?: () => number` - **Optional.** Function that returns delay in milliseconds between retry attempts
- `retryCondition?: (error: AxiosError) => boolean` - **Optional.** Custom function to determine if request should be retried based on error
- `onRetry?: (error: AxiosError) => void` - **Optional.** Callback executed before each retry attempt

#### CustomAxiosResponse

Interface representing the processed HTTP response.

```typescript
interface CustomAxiosResponse {
  status: number;        // HTTP status code
  headers: object;       // Response headers
  data: any;            // Response data
  maxRetry: number;     // Maximum retry attempts configured
  retryCount: number;   // Actual retry attempts made
}
```

### Exceptions

#### UnExpectedHttpError

Custom exception class for handling unexpected HTTP errors.

**Properties:**
- `code: string` - Error code
- `message: string` - Error message
- `stack?: string` - Error stack trace

**Usage:**
```typescript
try {
  const response = await httpClient.request(config);
} catch (error) {
  if (error instanceof UnExpectedHttpError) {
    console.log('HTTP Error Code:', error.code);
    console.log('Error Message:', error.message);
  }
}
```

## Features

### Automatic Retry Logic

Configure intelligent retry behavior for failed requests:

```typescript
const retryConfig: retryOptionType = {
  retries: 5,
  retryDelay: () => Math.pow(2, retryCount) * 1000, // Exponential backoff
  retryCondition: (error) => {
    // Retry on network errors or 5xx server errors
    return !error.response || error.response.status >= 500;
  },
  onRetry: (error) => {
    console.log(`Retrying request due to ${error.message}`);
  }
};

const response = await httpClient.request(axiosConfig, 'API_CALL', retryConfig);
```

### Response Processing

Automatic response processing with metadata:

```typescript
const response = await httpClient.request(config);

console.log(`Request completed with status: ${response.status}`);
console.log(`Retry attempts: ${response.retryCount} of ${response.maxRetry}`);
console.log(`Response data:`, response.data);
```

### Error Handling

Comprehensive error handling with typed exceptions:

```typescript
try {
  const result = await httpClient.request({
    method: 'GET',
    url: 'https://unreachable-api.example.com'
  });
} catch (error) {
  if (error instanceof UnExpectedHttpError) {
    // Handle network or unexpected errors
    console.error('Network error:', error.code);
  } else {
    // Handle HTTP errors (4xx, 5xx responses)
    console.error('HTTP error:', error.response?.status);
  }
}
```

### Timeout and Cancellation

Configure request timeouts and cancellation:

```typescript
const response = await httpClient.request({
  method: 'GET',
  url: 'https://slow-api.example.com/data',
  timeout: 10000, // 10 second timeout
  signal: abortController.signal // Support for cancellation
});
```

## Advanced Usage

### Custom Retry Conditions

Implement sophisticated retry logic based on specific conditions:

```typescript
const customRetryConfig: retryOptionType = {
  retries: 3,
  retryCondition: (error) => {
    // Retry on rate limit errors
    if (error.response?.status === 429) return true;
    
    // Retry on specific server errors
    if (error.response?.status === 502 || error.response?.status === 503) return true;
    
    // Retry on timeout errors
    if (error.code === 'ECONNABORTED') return true;
    
    return false;
  },
  retryDelay: () => {
    // Custom delay logic (e.g., respect rate limit headers)
    const retryAfter = error.response?.headers['retry-after'];
    return retryAfter ? parseInt(retryAfter) * 1000 : 2000;
  },
  onRetry: (error) => {
    // Custom logging or metrics
    console.log(`Retrying after error: ${error.response?.status || error.code}`);
  }
};
```

### Request Interceptors

Combine with Axios interceptors for additional functionality:

```typescript
@Injectable()
export class InterceptedHttpService {
  constructor(private httpClient: CustomAxiosService) {
    // Add request interceptor
    this.httpClient.axiosRef.interceptors.request.use(
      config => {
        config.headers['X-Request-ID'] = generateRequestId();
        return config;
      }
    );
  }
}
```

### Integration with NestJS Modules

Full integration with NestJS dependency injection:

```typescript
@Module({
  imports: [HttpClientModule.register()],
  providers: [MyApiService],
  exports: [MyApiService]
})
export class ApiModule {}
```

## Dependencies

This library requires:
- `@nestjs/axios` - NestJS HTTP module
- `@nestjs/common` - NestJS common utilities
- `axios` - HTTP client library
- `axios-retry` - Retry functionality for Axios
- `rxjs` - Reactive extensions for JavaScript

## TypeScript Support

Full TypeScript support with comprehensive type definitions for all interfaces, classes, and service methods. Generic support for typed responses:

```typescript
interface UserData {
  id: number;
  name: string;
  email: string;
}

const response = await httpClient.request({
  method: 'GET',
  url: '/api/users/1'
});

const user: UserData = response.data;
```

## License

ISC