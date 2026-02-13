# Module 5: Interceptors & HTTP Handling

## 📚 Learning Objectives

By the end of this module, you will understand:

- EQXJS interceptor architecture and patterns
- HTTP interceptor for request/response transformation
- REST interceptor for RESTful API enhancements
- Legacy HTTP support and migration strategies
- Error handling and response formatting
- Performance optimization with interceptors

## 🧭 Visual Flow (Mermaid)

```mermaid
---
config:
  theme: dark
  themeVariables:
    lineColor: '#7c7a9c'
    arrowheadColor: '#7c7a9c'
    primaryTextColor: '#2f2d3a'
    primaryColor: '#f7e9f3'
    secondaryColor: '#e8f2fb'
    tertiaryColor: '#f7f4e8'
    background: '#ffffff'
    clusterBkg: '#f0f7ff'
    clusterBorder: '#c9d7f2'
  look: classic
---
sequenceDiagram
  autonumber
  participant C as Client
  participant N as Nest Router
  participant LI as LegacyHttpInterceptor
  participant HI as HttpInterceptor
  participant RI as RestInterceptor
  participant Ctrl as Controller
  participant S as Service

  C->>N: HTTP Request
  N->>LI: intercept()

  alt Legacy request detected
    LI->>LI: transform request (url/body/headers)
    LI->>HI: next.handle()
  else Normal request
    LI->>HI: next.handle()
  end

  HI->>HI: preprocess (requestId, timestamp, sanitize)
  HI->>HI: set response headers (security/version)
  HI->>RI: next.handle()

  RI->>RI: apply REST conventions (status/ETag/Location)
  RI->>Ctrl: invoke route handler
  Ctrl->>S: business logic

  alt Success
    S-->>Ctrl: data
    Ctrl-->>RI: data
    RI-->>HI: REST response formatting
    HI-->>N: envelope + logging
    N-->>C: HTTP Response
  else Error
    S--x Ctrl: throw error
    Ctrl--x RI: propagate
    RI--x HI: propagate
    HI-->>N: map error -> standard response
    N-->>C: HTTP Error Response
  end
```

---

## 🔄 5.1 Interceptor Architecture

### NestJS Interceptor Foundation

EQXJS interceptors build upon NestJS's interceptor pattern, providing enterprise-grade enhancements:

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { tap, map, catchError } from "rxjs/operators";

@Injectable()
export abstract class BaseEqxjsInterceptor implements NestInterceptor {
  abstract intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<any>;

  protected getRequest(context: ExecutionContext) {
    return context.switchToHttp().getRequest();
  }

  protected getResponse(context: ExecutionContext) {
    return context.switchToHttp().getResponse();
  }

  protected isHttpContext(context: ExecutionContext): boolean {
    return context.getType() === "http";
  }
}
```

### EQXJS Interceptor Hierarchy

```
BaseEqxjsInterceptor (Abstract)
├── HttpInterceptor
│   ├── Request transformation
│   ├── Response transformation
│   └── Error handling
├── RestInterceptor
│   ├── RESTful patterns
│   ├── Resource-based routing
│   └── Content negotiation
├── LegacyHttpInterceptor
│   ├── Backward compatibility
│   └── Legacy system integration
└── AppInterceptor (Custom)
    ├── Application-specific logic
    └── Business rule enforcement
```

---

## 🌐 5.2 HTTP Interceptor

### Core HTTP Interceptor Implementation

```typescript
import {
  Injectable,
  Logger,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { tap, map, catchError } from "rxjs/operators";
import { BaseEqxjsInterceptor } from "./base-eqxjs.interceptor";

@Injectable()
export class HttpInterceptor extends BaseEqxjsInterceptor {
  private readonly logger = new Logger(HttpInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    if (!this.isHttpContext(context)) {
      return next.handle();
    }

    const request = this.getRequest(context);
    const response = this.getResponse(context);
    const startTime = Date.now();

    // Pre-processing
    this.preprocessRequest(request);
    this.setResponseHeaders(response);

    return next.handle().pipe(
      map((data) => this.transformResponse(data, context)),
      tap(() => this.logRequestComplete(request, startTime)),
      catchError((error) => this.handleError(error, context)),
    );
  }

  private preprocessRequest(request: any): void {
    // Add request ID for tracing
    if (!request.headers["x-request-id"]) {
      request.headers["x-request-id"] = this.generateRequestId();
    }

    // Add timestamp
    request.timestamp = Date.now();

    // Sanitize query parameters
    if (request.query) {
      request.query = this.sanitizeQueryParams(request.query);
    }

    // Log incoming request
    this.logger.debug(`Incoming ${request.method} ${request.url}`, {
      requestId: request.headers["x-request-id"],
      userAgent: request.headers["user-agent"],
      ip: request.ip,
      body: this.sanitizeRequestBody(request.body),
    });
  }

  private setResponseHeaders(response: any): void {
    // Security headers
    response.setHeader("X-Content-Type-Options", "nosniff");
    response.setHeader("X-Frame-Options", "DENY");
    response.setHeader("X-XSS-Protection", "1; mode=block");

    // API version header
    response.setHeader("X-API-Version", "1.0.0");

    // CORS headers (if not handled by framework)
    if (!response.getHeader("Access-Control-Allow-Origin")) {
      response.setHeader("Access-Control-Allow-Origin", "*");
    }
  }

  private transformResponse(data: any, context: ExecutionContext): any {
    const request = this.getRequest(context);
    const requestId = request.headers["x-request-id"];

    // Add metadata to response
    const transformedResponse = {
      success: true,
      requestId,
      timestamp: new Date().toISOString(),
      data,
      meta: {
        processingTime: Date.now() - request.timestamp,
        version: "1.0.0",
      },
    };

    return transformedResponse;
  }

  private handleError(
    error: any,
    context: ExecutionContext,
  ): Observable<never> {
    const request = this.getRequest(context);
    const requestId = request.headers["x-request-id"];

    this.logger.error(`Request ${requestId} failed`, {
      error: error.message,
      stack: error.stack,
      method: request.method,
      url: request.url,
    });

    // Transform error response
    const errorResponse = {
      success: false,
      requestId,
      timestamp: new Date().toISOString(),
      error: {
        message: error.message,
        code: error.status || 500,
        type: error.constructor.name,
      },
      meta: {
        processingTime: Date.now() - request.timestamp,
      },
    };

    throw errorResponse;
  }

  private generateRequestId(): string {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private sanitizeQueryParams(query: any): any {
    // Remove potentially harmful query parameters
    const sanitized = { ...query };
    delete sanitized.password;
    delete sanitized.secret;
    delete sanitized.token;
    return sanitized;
  }

  private sanitizeRequestBody(body: any): any {
    if (!body) return body;

    const sanitized = { ...body };
    delete sanitized.password;
    delete sanitized.secret;
    delete sanitized.token;
    return sanitized;
  }

  private logRequestComplete(request: any, startTime: number): void {
    const processingTime = Date.now() - startTime;

    this.logger.log(`${request.method} ${request.url} completed`, {
      requestId: request.headers["x-request-id"],
      processingTime,
      status: "success",
    });
  }
}
```

---

## 🚀 5.3 REST Interceptor

### RESTful API Enhancement Interceptor

```typescript
import {
  Injectable,
  Logger,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";
import { BaseEqxjsInterceptor } from "./base-eqxjs.interceptor";

@Injectable()
export class RestInterceptor extends BaseEqxjsInterceptor {
  private readonly logger = new Logger(RestInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    if (!this.isHttpContext(context)) {
      return next.handle();
    }

    const request = this.getRequest(context);
    const response = this.getResponse(context);

    // Apply RESTful conventions
    this.applyRestfulConventions(request, response);

    return next
      .handle()
      .pipe(map((data) => this.formatRestResponse(data, request, response)));
  }

  private applyRestfulConventions(request: any, response: any): void {
    // Set appropriate status codes for different methods
    switch (request.method) {
      case "POST":
        if (!response.statusCode || response.statusCode === 200) {
          response.statusCode = 201; // Created
        }
        break;
      case "PUT":
      case "PATCH":
        if (!response.statusCode || response.statusCode === 200) {
          response.statusCode = 200; // OK
        }
        break;
      case "DELETE":
        if (!response.statusCode || response.statusCode === 200) {
          response.statusCode = 204; // No Content
        }
        break;
    }

    // Add REST-specific headers
    if (request.method === "POST" && request.body?.id) {
      const resourceLocation = `${request.protocol}://${request.get("host")}${request.path}/${request.body.id}`;
      response.setHeader("Location", resourceLocation);
    }

    // Enable ETag for caching
    if (["GET", "HEAD"].includes(request.method)) {
      const etag = this.generateETag(request);
      response.setHeader("ETag", etag);

      // Handle conditional requests
      if (request.headers["if-none-match"] === etag) {
        response.statusCode = 304; // Not Modified
      }
    }
  }

  private formatRestResponse(data: any, request: any, response: any): any {
    // Handle different REST response patterns
    switch (request.method) {
      case "GET":
        return this.formatGetResponse(data, request);
      case "POST":
        return this.formatCreateResponse(data, request);
      case "PUT":
      case "PATCH":
        return this.formatUpdateResponse(data, request);
      case "DELETE":
        return this.formatDeleteResponse(data, request);
      default:
        return data;
    }
  }

  private formatGetResponse(data: any, request: any): any {
    // Handle collections vs single resources
    if (Array.isArray(data)) {
      return {
        data,
        pagination: this.extractPaginationInfo(request, data),
        links: this.generateHATEOASLinks(request, data),
        meta: {
          total: data.length,
          timestamp: new Date().toISOString(),
        },
      };
    } else {
      return {
        data,
        links: this.generateResourceLinks(request, data),
        meta: {
          timestamp: new Date().toISOString(),
        },
      };
    }
  }

  private formatCreateResponse(data: any, request: any): any {
    return {
      data,
      message: "Resource created successfully",
      links: {
        self: `${request.protocol}://${request.get("host")}${request.path}/${data.id}`,
        collection: `${request.protocol}://${request.get("host")}${request.path}`,
      },
      meta: {
        created: new Date().toISOString(),
      },
    };
  }

  private formatUpdateResponse(data: any, request: any): any {
    return {
      data,
      message: "Resource updated successfully",
      links: {
        self: `${request.protocol}://${request.get("host")}${request.url}`,
        collection: `${request.protocol}://${request.get("host")}${request.path}`,
      },
      meta: {
        updated: new Date().toISOString(),
      },
    };
  }

  private formatDeleteResponse(data: any, request: any): any {
    // DELETE typically returns empty body with 204 status
    if (data === null || data === undefined) {
      return null;
    }

    return {
      message: "Resource deleted successfully",
      meta: {
        deleted: new Date().toISOString(),
      },
    };
  }

  private extractPaginationInfo(request: any, data: any[]): any {
    const page = parseInt(request.query.page) || 1;
    const limit = parseInt(request.query.limit) || 10;
    const offset = (page - 1) * limit;

    return {
      page,
      limit,
      offset,
      total: data.length,
      hasNext: offset + limit < data.length,
      hasPrevious: page > 1,
    };
  }

  private generateHATEOASLinks(request: any, data: any[]): any {
    const baseUrl = `${request.protocol}://${request.get("host")}${request.path}`;
    const page = parseInt(request.query.page) || 1;
    const limit = parseInt(request.query.limit) || 10;

    const links: any = {
      self: request.url,
      first: `${baseUrl}?page=1&limit=${limit}`,
    };

    if (page > 1) {
      links.prev = `${baseUrl}?page=${page - 1}&limit=${limit}`;
    }

    if (data.length === limit) {
      links.next = `${baseUrl}?page=${page + 1}&limit=${limit}`;
    }

    return links;
  }

  private generateResourceLinks(request: any, data: any): any {
    const baseUrl = `${request.protocol}://${request.get("host")}`;

    return {
      self: request.url,
      collection: `${baseUrl}${request.path}`,
    };
  }

  private generateETag(request: any): string {
    // Simple ETag generation based on URL and timestamp
    const content = `${request.url}-${Date.now()}`;
    return `\"${Buffer.from(content).toString("base64")}\"`;
  }
}
```

---

## 🔄 5.4 Legacy HTTP Support

### Legacy HTTP Interceptor for Backward Compatibility

```typescript
import {
  Injectable,
  Logger,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map, tap } from "rxjs/operators";
import { BaseEqxjsInterceptor } from "./base-eqxjs.interceptor";

@Injectable()
export class LegacyHttpInterceptor extends BaseEqxjsInterceptor {
  private readonly logger = new Logger(LegacyHttpInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    if (!this.isHttpContext(context)) {
      return next.handle();
    }

    const request = this.getRequest(context);
    const response = this.getResponse(context);

    // Check if request requires legacy handling
    if (this.isLegacyRequest(request)) {
      return this.handleLegacyRequest(context, next);
    }

    return next.handle();
  }

  private isLegacyRequest(request: any): boolean {
    // Detect legacy requests by various criteria
    return (
      request.headers["x-legacy-api"] === "true" ||
      request.headers["user-agent"]?.includes("LegacyClient") ||
      request.url.startsWith("/api/v1/legacy") ||
      request.headers["content-type"] === "application/x-www-form-urlencoded"
    );
  }

  private handleLegacyRequest(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<any> {
    const request = this.getRequest(context);
    const response = this.getResponse(context);

    this.logger.debug(
      `Handling legacy request: ${request.method} ${request.url}`,
      {
        userAgent: request.headers["user-agent"],
        contentType: request.headers["content-type"],
      },
    );

    // Transform legacy request format
    this.transformLegacyRequest(request);

    // Set legacy response headers
    this.setLegacyResponseHeaders(response);

    return next.handle().pipe(
      map((data) => this.transformLegacyResponse(data, request)),
      tap(() => this.logLegacyRequestComplete(request)),
    );
  }

  private transformLegacyRequest(request: any): void {
    // Transform legacy URL patterns
    if (request.url.includes("/legacy/")) {
      request.url = request.url.replace("/legacy/", "/v2/");
    }

    // Transform form data to JSON
    if (
      request.headers["content-type"] === "application/x-www-form-urlencoded" &&
      request.body
    ) {
      request.body = this.formDataToJson(request.body);
      request.headers["content-type"] = "application/json";
    }

    // Add compatibility headers
    request.headers["x-api-version"] = "legacy";
    request.headers["x-legacy-support"] = "true";
  }

  private setLegacyResponseHeaders(response: any): void {
    // Legacy-specific headers
    response.setHeader("X-Legacy-API", "true");
    response.setHeader("X-API-Version", "legacy");

    // Disable modern security headers for compatibility
    response.removeHeader("X-Content-Type-Options");
    response.removeHeader("X-Frame-Options");

    // Set old-style cache headers
    response.setHeader("Pragma", "no-cache");
    response.setHeader("Expires", "-1");
  }

  private transformLegacyResponse(data: any, request: any): any {
    // Legacy systems expect different response formats
    if (request.headers["x-legacy-format"] === "xml") {
      return this.jsonToXml(data);
    }

    if (request.headers["x-legacy-format"] === "simple") {
      // Simple format without metadata
      return data.data || data;
    }

    // Default legacy JSON format
    return {
      status: "success",
      result: data,
      timestamp: Date.now(), // Unix timestamp for legacy compatibility
      version: "legacy",
    };
  }

  private formDataToJson(formData: string): any {
    const params = new URLSearchParams(formData);
    const json: any = {};

    for (const [key, value] of params) {
      // Handle nested objects (user[name] -> user.name)
      if (key.includes("[") && key.includes("]")) {
        const match = key.match(/^([^[]+)\[([^\]]+)\]$/);
        if (match) {
          const [, parentKey, childKey] = match;
          if (!json[parentKey]) {
            json[parentKey] = {};
          }
          json[parentKey][childKey] = value;
        }
      } else {
        json[key] = value;
      }
    }

    return json;
  }

  private jsonToXml(data: any): string {
    // Simple JSON to XML conversion
    const xmlHeader = '<?xml version=\"1.0\" encoding=\"UTF-8\"?>';
    const xmlBody = this.objectToXml(data, "response");
    return `${xmlHeader}\n${xmlBody}`;
  }

  private objectToXml(obj: any, rootName: string): string {
    if (typeof obj !== "object" || obj === null) {
      return `<${rootName}>${obj}</${rootName}>`;
    }

    const xmlParts = [`<${rootName}>`];

    for (const [key, value] of Object.entries(obj)) {
      if (Array.isArray(value)) {
        xmlParts.push(`<${key}>`);
        value.forEach((item) => {
          xmlParts.push(this.objectToXml(item, "item"));
        });
        xmlParts.push(`</${key}>`);
      } else if (typeof value === "object" && value !== null) {
        xmlParts.push(this.objectToXml(value, key));
      } else {
        xmlParts.push(`<${key}>${value}</${key}>`);
      }
    }

    xmlParts.push(`</${rootName}>`);
    return xmlParts.join("");
  }

  private logLegacyRequestComplete(request: any): void {
    this.logger.warn(
      `Legacy request completed: ${request.method} ${request.url}`,
      {
        userAgent: request.headers["user-agent"],
        migrationRecommended: true,
      },
    );
  }
}
```

---

## 🎯 Summary

In this module, we've covered:

✅ **Interceptor Architecture**: EQXJS interceptor patterns and hierarchy  
✅ **HTTP Interceptor**: Request/response transformation and error handling  
✅ **REST Interceptor**: RESTful API enhancements and HATEOAS support  
✅ **Legacy HTTP Support**: Backward compatibility and migration strategies  
✅ **Response Formatting**: Consistent API response structures

### Key Takeaways

1. **Interceptors provide centralized** request/response processing
2. **HTTP Interceptor handles** common concerns like logging and security
3. **REST Interceptor enforces** RESTful conventions automatically
4. **Legacy support enables** smooth migration from old systems
5. **Response formatting ensures** consistent API experiences

---

## 🎓 Knowledge Check

Before proceeding to Module 6, ensure you understand:

- [ ] EQXJS interceptor architecture and patterns
- [ ] HTTP interceptor capabilities and configuration
- [ ] RESTful enhancements provided by REST interceptor
- [ ] Legacy HTTP support for backward compatibility
- [ ] Error handling and response transformation techniques

---

## ➡️ Next Steps

👉 **Continue to [Module 6: Context Management & Domain Services](module-06-context-domain.md)**

📝 **Complete the exercises**: [Module 5 Exercises](exercise/module-05-exercises.md)

---

## 📚 Additional Resources

- [NestJS Interceptors](https://docs.nestjs.com/interceptors)
- [RESTful API Design](https://restfulapi.net/)
- [HATEOAS Principles](https://restfulapi.net/hateoas/)
- [HTTP Status Codes](https://httpstatuses.com/)
