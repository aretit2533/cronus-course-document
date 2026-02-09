# Module 7 Exercises: Additional Fundamentals

## Overview
These exercises cover middleware, pipes, guards, interceptors, and custom decorators - the building blocks of request lifecycle management.

---

## Exercise 1: Logging Middleware with Request Duration

### Objective
Create middleware that logs all requests with execution time.

### Instructions

#### Step 1: Create Logger Middleware
```bash
nest new middleware-app
cd middleware-app

# Generate middleware
nest g middleware common/logger --flat
```

**`src/common/logger.middleware.ts`:**
```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const startTime = Date.now();
    const { method, originalUrl, ip } = req;

    // Log request
    console.log(`[${new Date().toISOString()}] ${method} ${originalUrl} - ${ip}`);

    // Capture response finish event
    res.on('finish', () => {
      const duration = Date.now() - startTime;
      const { statusCode } = res;
      
      console.log(
        `[${new Date().toISOString()}] ${method} ${originalUrl} - ${statusCode} - ${duration}ms`,
      );
    });

    next();
  }
}
```

#### Step 2: Apply Middleware
**Update `src/app.module.ts`:**
```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/logger.middleware';

@Module({
  // ... your modules
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*'); // Apply to all routes
  }
}
```

#### Step 3: Advanced - Route-Specific Middleware
```typescript
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes(
        { path: 'users', method: RequestMethod.ALL },
        { path: 'products', method: RequestMethod.GET },
      );
      
    // Or exclude specific routes
    consumer
      .apply(LoggerMiddleware)
      .exclude(
        { path: 'health', method: RequestMethod.GET },
        'auth/login',
      )
      .forRoutes('*');
  }
}
```

#### Step 4: Create Request Context Middleware
**`src/common/request-context.middleware.ts`:**
```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { randomUUID } from 'crypto';

@Injectable()
export class RequestContextMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Add unique request ID
    const requestId = randomUUID();
    req['requestId'] = requestId;
    res.setHeader('X-Request-ID', requestId);

    // Add user context (from auth header for example)
    const authHeader = req.headers.authorization;
    if (authHeader) {
      // Parse token and add user info to request
      req['user'] = { id: 1, username: 'john' }; // Simplified
    }

    next();
  }
}
```

### Tasks to Complete
- [ ] Create logger middleware
- [ ] Track request duration
- [ ] Apply to all routes
- [ ] Test with various endpoints
- [ ] Add request context middleware
- [ ] Verify logging output

### Expected Outcome
- All requests logged with timing
- Request context available
- Understanding of middleware execution

### Time Estimate
30-45 minutes

---

## Exercise 2: Validation Pipes

### Objective
Implement validation using pipes and class-validator.

### Instructions

#### Step 1: Install Dependencies
```bash
npm install class-validator class-transformer
```

#### Step 2: Create DTOs with Validation
**`src/users/dto/create-user.dto.ts`:**
```typescript
import {
  IsString,
  IsEmail,
  IsInt,
  Min,
  Max,
  MinLength,
  MaxLength,
  IsNotEmpty,
  Matches,
  IsOptional,
} from 'class-validator';

export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  @MaxLength(20)
  @Matches(/^[a-zA-Z0-9_]+$/, {
    message: 'Username can only contain letters, numbers, and underscores',
  })
  username: string;

  @IsEmail()
  @IsNotEmpty()
  email: string;

  @IsString()
  @MinLength(8)
  @MaxLength(100)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;

  @IsString()
  @IsNotEmpty()
  @MinLength(2)
  firstName: string;

  @IsString()
  @IsNotEmpty()
  @MinLength(2)
  lastName: string;

  @IsInt()
  @Min(13)
  @Max(120)
  age: number;

  @IsString()
  @IsOptional()
  @MaxLength(500)
  bio?: string;
}
```

#### Step 3: Enable Global Validation Pipe
**Update `src/main.ts`:**
```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // Strip non-whitelisted properties
      forbidNonWhitelisted: true, // Throw error for non-whitelisted properties
      transform: true, // Auto-transform payloads to DTO types
      transformOptions: {
        enableImplicitConversion: true, // Convert types automatically
      },
    }),
  );
  
  await app.listen(3000);
}
bootstrap();
```

#### Step 4: Test Validation
```bash
# Valid request
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "email": "john@example.com",
    "password": "Password123",
    "firstName": "John",
    "lastName": "Doe",
    "age": 25
  }'

# Invalid - short username
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "jd",
    "email": "john@example.com",
    "password": "Password123",
    "firstName": "John",
    "lastName": "Doe",
    "age": 25
  }'

# Expected error response:
{
  "statusCode": 400,
  "message": [
    "username must be longer than or equal to 3 characters"
  ],
  "error": "Bad Request"
}
```

#### Step 5: Create Custom Validation Pipe
**`src/common/pipes/parse-positive-int.pipe.ts`:**
```typescript
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParsePositiveIntPipe implements PipeTransform<string, number> {
  transform(value: string): number {
    const val = parseInt(value, 10);
    
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed: must be a number');
    }
    
    if (val <= 0) {
      throw new BadRequestException('Validation failed: must be positive');
    }
    
    return val;
  }
}
```

**Use in controller:**
```typescript
@Get(':id')
findOne(@Param('id', ParsePositiveIntPipe) id: number) {
  return this.usersService.findOne(id);
}
```

#### Step 6: Advanced - Custom Validator
**`src/common/validators/is-unique.validator.ts`:**
```typescript
import {
  registerDecorator,
  ValidationOptions,
  ValidatorConstraint,
  ValidatorConstraintInterface,
  ValidationArguments,
} from 'class-validator';
import { Injectable } from '@nestjs/common';

@ValidatorConstraint({ async: true })
@Injectable()
export class IsUniqueConstraint implements ValidatorConstraintInterface {
  async validate(value: any, args: ValidationArguments) {
    const [relatedProperty] = args.constraints;
    // Check database for uniqueness
    // This is a simplified example
    return value !== 'taken'; // Returns false if validation fails
  }

  defaultMessage(args: ValidationArguments) {
    return `${args.property} is already taken`;
  }
}

export function IsUnique(property: string, validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [property],
      validator: IsUniqueConstraint,
    });
  };
}
```

**Use in DTO:**
```typescript
export class CreateUserDto {
  @IsString()
  @IsUnique('username')
  username: string;
}
```

### Tasks to Complete
- [ ] Create DTOs with validation decorators
- [ ] Enable global validation pipe
- [ ] Test valid and invalid inputs
- [ ] Create custom validation pipe
- [ ] Implement custom validator
- [ ] Verify all validation works

### Expected Outcome
- Automatic request validation
- Clear error messages
- Type transformations working

### Time Estimate
60-90 minutes

---

## Exercise 3: Authentication Guard with JWT

### Objective
Implement an authentication guard using JWT tokens.

### Instructions

#### Step 1: Install Dependencies
```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install -D @types/passport-jwt
```

#### Step 2: Create Auth Guard
**`src/common/guards/auth.guard.ts`:**
```typescript
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  UnauthorizedException,
} from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { Request } from 'express';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest<Request>();
    const token = this.extractTokenFromHeader(request);

    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      const payload = await this.jwtService.verifyAsync(token, {
        secret: process.env.JWT_SECRET || 'your-secret-key',
      });
      
      // Attach user to request
      request['user'] = payload;
    } catch {
      throw new UnauthorizedException('Invalid token');
    }

    return true;
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}
```

#### Step 3: Create Roles Guard
**`src/common/guards/roles.guard.ts`:**
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );

    if (!requiredRoles) {
      return true; // No roles required
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    if (!user) {
      return false;
    }

    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}
```

#### Step 4: Create Roles Decorator
**`src/common/decorators/roles.decorator.ts`:**
```typescript
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

#### Step 5: Use Guards in Controller
```typescript
import { Controller, Get, Post, UseGuards } from '@nestjs/common';
import { AuthGuard } from '../common/guards/auth.guard';
import { RolesGuard } from '../common/guards/roles.guard';
import { Roles } from '../common/decorators/roles.decorator';

@Controller('users')
@UseGuards(AuthGuard, RolesGuard)
export class UsersController {
  @Get()
  @Roles('admin', 'user')
  findAll() {
    return [];
  }

  @Get('profile')
  getProfile() {
    // Only authentication required, any role
    return { message: 'Profile' };
  }

  @Post()
  @Roles('admin')
  create() {
    // Only admin can create
    return { message: 'Created' };
  }
}
```

#### Step 6: Set Up JWT Module
**Create `src/auth/auth.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';

@Module({
  imports: [
    JwtModule.register({
      global: true,
      secret: process.env.JWT_SECRET || 'your-secret-key',
      signOptions: { expiresIn: '1d' },
    }),
  ],
  providers: [AuthService],
  controllers: [AuthController],
})
export class AuthModule {}
```

### Tasks to Complete
- [ ] Create AuthGuard
- [ ] Create RolesGuard
- [ ] Implement JWT verification
- [ ] Create Roles decorator
- [ ] Protect routes with guards
- [ ] Test authentication flow

### Expected Outcome
- Protected routes
- Role-based access control
- JWT authentication working

### Time Estimate
60-90 minutes

---

## Exercise 4: Response Transformation Interceptor

### Objective
Create interceptors to transform responses and add metadata.

### Instructions

#### Step 1: Create Transform Interceptor
**`src/common/interceptors/transform.interceptor.ts`:**
```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  success: boolean;
  data: T;
  timestamp: string;
  path: string;
}

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, Response<T>>
{
  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<Response<T>> {
    const request = context.switchToHttp().getRequest();

    return next.handle().pipe(
      map((data) => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
        path: request.url,
      })),
    );
  }
}
```

#### Step 2: Create Logging Interceptor
**`src/common/interceptors/logging.interceptor.ts`:**
```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;

    console.log(`[REQUEST] ${method} ${url}`);

    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        const delay = Date.now() - now;
        console.log(
          `[RESPONSE] ${method} ${url} ${response.statusCode} - ${delay}ms`,
        );
      }),
    );
  }
}
```

#### Step 3: Create Cache Interceptor
**`src/common/interceptors/cache.interceptor.ts`:**
```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  private cache = new Map<string, any>();

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = `${request.method}:${request.url}`;

    // Return cached response if exists
    if (this.cache.has(cacheKey)) {
      console.log(`[CACHE HIT] ${cacheKey}`);
      return of(this.cache.get(cacheKey));
    }

    console.log(`[CACHE MISS] ${cacheKey}`);

    // Store response in cache
    return next.handle().pipe(
      tap((data) => {
        this.cache.set(cacheKey, data);
        
        // Clear cache after 60 seconds
        setTimeout(() => {
          this.cache.delete(cacheKey);
        }, 60000);
      }),
    );
  }
}
```

#### Step 4: Apply Interceptors
**Globally in `main.ts`:**
```typescript
import { TransformInterceptor } from './common/interceptors/transform.interceptor';
import { LoggingInterceptor } from './common/interceptors/logging.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalInterceptors(
    new LoggingInterceptor(),
    new TransformInterceptor(),
  );
  
  await app.listen(3000);
}
```

**Or on specific routes:**
```typescript
@Controller('users')
@UseInterceptors(CacheInterceptor)
export class UsersController {
  @Get()
  findAll() {
    return [];
  }
}
```

### Tasks to Complete
- [ ] Create transform interceptor
- [ ] Create logging interceptor
- [ ] Create cache interceptor
- [ ] Apply globally or per route
- [ ] Test response transformation
- [ ] Verify caching works

### Expected Outcome
- Consistent response format
- Request/response logging
- Response caching working

### Time Estimate
45-60 minutes

---

## Exercise 5: Custom Decorators

### Objective
Build custom decorators for common patterns.

### Instructions

#### Step 1: User Decorator
**`src/common/decorators/user.decorator.ts`:**
```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
```

**Usage:**
```typescript
@Get('profile')
getProfile(@User() user: any) {
  return user;
}

@Get('info')
getUserId(@User('id') userId: number) {
  return { userId };
}
```

#### Step 2: API Response Decorator
**`src/common/decorators/api-response.decorator.ts`:**
```typescript
import { applyDecorators, HttpStatus } from '@nestjs/common';
import { ApiResponse } from '@nestjs/swagger';

export function ApiSuccessResponse(description: string) {
  return applyDecorators(
    ApiResponse({
      status: HttpStatus.OK,
      description,
    }),
  );
}

export function ApiCreatedResponse(description: string) {
  return applyDecorators(
    ApiResponse({
      status: HttpStatus.CREATED,
      description,
    }),
  );
}

export function ApiErrorResponses() {
  return applyDecorators(
    ApiResponse({
      status: HttpStatus.BAD_REQUEST,
      description: 'Bad Request',
    }),
    ApiResponse({
      status: HttpStatus.UNAUTHORIZED,
      description: 'Unauthorized',
    }),
    ApiResponse({
      status: HttpStatus.INTERNAL_SERVER_ERROR,
      description: 'Internal Server Error',
    }),
  );
}
```

#### Step 3: Public Route Decorator
**`src/common/decorators/public.decorator.ts`:**
```typescript
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

**Update AuthGuard:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private reflector: Reflector,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true;
    }

    // ... rest of authentication logic
  }
}
```

**Usage:**
```typescript
@Controller('auth')
export class AuthController {
  @Public()
  @Post('login')
  login() {
    // No authentication required
  }
}
```

### Tasks to Complete
- [ ] Create User decorator
- [ ] Create API response decorators
- [ ] Create Public decorator
- [ ] Test all decorators
- [ ] Combine multiple decorators

### Expected Outcome
- Reusable custom decorators
- Cleaner controller code
- Better code organization

### Time Estimate
30-45 minutes

---

## Challenge Exercise: Complete Request Lifecycle

### Objective
Build a complete request lifecycle with all components working together.

### Components
1. Authentication middleware
2. Logging middleware
3. Validation pipe
4. Auth guard
5. Roles guard
6. Transform interceptor
7. Exception filter

### Tasks
- Set up all components
- Configure execution order
- Test complete flow
- Document behavior

### Time Estimate
120+ minutes

---

## Submission Checklist

- [ ] Logging middleware created
- [ ] Validation pipes implemented
- [ ] Authentication guard working
- [ ] Interceptors transforming responses
- [ ] Custom decorators in use
- [ ] Complete lifecycle tested

---

## Additional Resources

- [Middleware](https://docs.nestjs.com/middleware)
- [Pipes](https://docs.nestjs.com/pipes)
- [Guards](https://docs.nestjs.com/guards)
- [Interceptors](https://docs.nestjs.com/interceptors)
- [Custom Decorators](https://docs.nestjs.com/custom-decorators)

---

## Next Steps

Complete your learning with [Module 8 Exercises](module-08-exercises.md) - build a complete practical application!
