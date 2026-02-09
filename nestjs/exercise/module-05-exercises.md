# Module 5 Exercises: Modules

## Overview
These exercises focus on organizing your application with modules, creating feature modules, shared modules, and dynamic modules.

---

## Exercise 1: Create Feature Modules

### Objective
Organize a monolithic application into feature modules (Users, Auth, Products, Orders).

### Instructions

#### Step 1: Generate Feature Modules
```bash
# Create new project
nest new modular-app
cd modular-app

# Generate modules
nest g module users
nest g module auth
nest g module products
nest g module orders

# Generate controllers and services for each
nest g controller users
nest g service users
nest g controller products
nest g service products
nest g controller orders
nest g service orders
```

#### Step 2: Create Users Module
**`src/users/users.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Export for other modules
})
export class UsersModule {}
```

**`src/users/users.service.ts`:**
```typescript
import { Injectable } from '@nestjs/common';

export interface User {
  id: number;
  username: string;
  email: string;
  password: string;
}

@Injectable()
export class UsersService {
  private users: User[] = [
    { id: 1, username: 'john', email: 'john@example.com', password: 'hashed' },
  ];

  findAll(): User[] {
    return this.users;
  }

  findOne(id: number): User | undefined {
    return this.users.find(u => u.id === id);
  }

  findByUsername(username: string): User | undefined {
    return this.users.find(u => u.username === username);
  }

  findByEmail(email: string): User | undefined {
    return this.users.find(u => u.email === email);
  }

  create(user: Omit<User, 'id'>): User {
    const newUser = { id: Date.now(), ...user };
    this.users.push(newUser);
    return newUser;
  }
}
```

#### Step 3: Create Auth Module (depends on Users)
**`src/auth/auth.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module'; // Import UsersModule

@Module({
  imports: [UsersModule], // Import to use UsersService
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

**`src/auth/auth.service.ts`:**
```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}

  async validateUser(username: string, password: string): Promise<any> {
    const user = this.usersService.findByUsername(username);
    if (user && user.password === password) {
      const { password, ...result } = user;
      return result;
    }
    throw new UnauthorizedException('Invalid credentials');
  }

  async login(username: string, password: string) {
    const user = await this.validateUser(username, password);
    return {
      access_token: 'fake-jwt-token',
      user,
    };
  }

  async register(username: string, email: string, password: string) {
    return this.usersService.create({ username, email, password });
  }
}
```

**`src/auth/auth.controller.ts`:**
```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { AuthService } from './auth.service';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  async login(@Body() body: { username: string; password: string }) {
    return this.authService.login(body.username, body.password);
  }

  @Post('register')
  async register(@Body() body: { username: string; email: string; password: string }) {
    return this.authService.register(body.username, body.email, body.password);
  }
}
```

#### Step 4: Create Products Module
**Complete implementation with ProductsService and ProductsController**

#### Step 5: Create Orders Module (depends on Products & Users)
**`src/orders/orders.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { OrdersController } from './orders.controller';
import { OrdersService } from './orders.service';
import { ProductsModule } from '../products/products.module';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [ProductsModule, UsersModule],
  controllers: [OrdersController],
  providers: [OrdersService],
})
export class OrdersModule {}
```

#### Step 6: Update App Module
**`src/app.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { AuthModule } from './auth/auth.module';
import { ProductsModule } from './products/products.module';
import { OrdersModule } from './orders/orders.module';

@Module({
  imports: [
    UsersModule,
    AuthModule,
    ProductsModule,
    OrdersModule,
  ],
})
export class AppModule {}
```

### Tasks to Complete
- [ ] Create 4 feature modules
- [ ] Implement dependencies between modules
- [ ] Export services that other modules need
- [ ] Test cross-module dependencies
- [ ] Verify module encapsulation

### Expected Outcome
- Well-organized application structure
- Clear module boundaries
- working dependencies

### Time Estimate
90-120 minutes

---

## Exercise 2: Create Shared Module

### Objective
Create a SharedModule with common services used across the application.

### Instructions

#### Step 1: Generate Shared Module
```bash
nest g module shared
nest g service shared/logger --flat
nest g service shared/config --flat
nest g service shared/validation --flat
```

#### Step 2: Create Logger Service
**`src/shared/logger.service.ts`:**
```typescript
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.TRANSIENT })
export class LoggerService {
  private context: string;

  setContext(context: string) {
    this.context = context;
  }

  log(message: string) {
    console.log(`[${this.context}] ${new Date().toISOString()} - LOG: ${message}`);
  }

  error(message: string, trace?: string) {
    console.error(`[${this.context}] ${new Date().toISOString()} - ERROR: ${message}`);
    if (trace) {
      console.error(trace);
    }
  }

  warn(message: string) {
    console.warn(`[${this.context}] ${new Date().toISOString()} - WARN: ${message}`);
  }

  debug(message: string) {
    console.debug(`[${this.context}] ${new Date().toISOString()} - DEBUG: ${message}`);
  }
}
```

#### Step 3: Create Config Service
**`src/shared/config.service.ts`:**
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class ConfigService {
  get(key: string): string | undefined {
    return process.env[key];
  }

  getOrThrow(key: string): string {
    const value = this.get(key);
    if (!value) {
      throw new Error(`Configuration key "${key}" is not set`);
    }
    return value;
  }

  getNumber(key: string, defaultValue: number): number {
    const value = this.get(key);
    return value ? parseInt(value, 10) : defaultValue;
  }

  getBoolean(key: string, defaultValue: boolean): boolean {
    const value = this.get(key);
    return value ? value === 'true' : defaultValue;
  }
}
```

#### Step 4: Create Shared Module
**`src/shared/shared.module.ts`:**
```typescript
import { Global, Module } from '@nestjs/common';
import { LoggerService } from './logger.service';
import { ConfigService } from './config.service';
import { ValidationService } from './validation.service';

@Global() // Makes module globally available
@Module({
  providers: [LoggerService, ConfigService, ValidationService],
  exports: [LoggerService, ConfigService, ValidationService],
})
export class SharedModule {}
```

#### Step 5: Use in Feature Modules
**`src/users/users.service.ts`:**
```typescript
import { Injectable } from '@nestjs/common';
import { LoggerService } from '../shared/logger.service';

@Injectable()
export class UsersService {
  constructor(private logger: LoggerService) {
    this.logger.setContext('UsersService');
  }

  findAll() {
    this.logger.log('Finding all users');
    return [];
  }
}
```

#### Step 6: Import Shared Module Once
**`src/app.module.ts`:**
```typescript
@Module({
  imports: [
    SharedModule, // Import once, available everywhere
    UsersModule,
    AuthModule,
    ProductsModule,
    OrdersModule,
  ],
})
export class AppModule {}
```

### Tasks to Complete
- [ ] Create SharedModule with common services
- [ ] Make it @Global()
- [ ] Use shared services in feature modules
- [ ] Test that services are accessible
- [ ] Verify singleton behavior

### Expected Outcome
- Reusable common services
- No duplicate code
- Global availability

### Time Estimate
45-60 minutes

---

## Exercise 3: Create Dynamic Module

### Objective
Build a CacheModule that can be configured differently for each feature.

### Instructions

#### Step 1: Create Cache Module
```bash
nest g module cache
nest g service cache
```

#### Step 2: Implement Dynamic Module
**`src/cache/cache.module.ts`:**
```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { CacheService } from './cache.service';

export interface CacheModuleOptions {
  ttl: number; // Time to live in seconds
  max: number; // Maximum items
  prefix?: string;
}

@Module({})
export class CacheModule {
  static forRoot(options: CacheModuleOptions): DynamicModule {
    return {
      module: CacheModule,
      providers: [
        {
          provide: 'CACHE_OPTIONS',
          useValue: options,
        },
        CacheService,
      ],
      exports: [CacheService],
      global: true,
    };
  }

  static forFeature(prefix: string): DynamicModule {
    return {
      module: CacheModule,
      providers: [
        {
          provide: 'CACHE_PREFIX',
          useValue: prefix,
        },
        CacheService,
      ],
      exports: [CacheService],
    };
  }
}
```

**`src/cache/cache.service.ts`:**
```typescript
import { Injectable, Inject, Optional } from '@nestjs/common';
import { CacheModuleOptions } from './cache.module';

@Injectable()
export class CacheService {
  private cache = new Map<string, { value: any; expiry: number }>();

  constructor(
    @Inject('CACHE_OPTIONS') private options: CacheModuleOptions,
    @Optional() @Inject('CACHE_PREFIX') private prefix?: string,
  ) {}

  private getKey(key: string): string {
    const finalPrefix = this.prefix || this.options.prefix || '';
    return finalPrefix ? `${finalPrefix}:${key}` : key;
  }

  set(key: string, value: any): void {
    const fullKey = this.getKey(key);
    const expiry = Date.now() + this.options.ttl * 1000;
    
    // Check max size
    if (this.cache.size >= this.options.max) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    this.cache.set(fullKey, { value, expiry });
  }

  get(key: string): any | null {
    const fullKey = this.getKey(key);
    const item = this.cache.get(fullKey);

    if (!item) {
      return null;
    }

    // Check expiry
    if (Date.now() > item.expiry) {
      this.cache.delete(fullKey);
      return null;
    }

    return item.value;
  }

  delete(key: string): void {
    const fullKey = this.getKey(key);
    this.cache.delete(fullKey);
  }

  clear(): void {
    this.cache.clear();
  }

  has(key: string): boolean {
    return this.get(key) !== null;
  }
}
```

#### Step 3: Use forRoot in App Module
**`src/app.module.ts`:**
```typescript
import { CacheModule } from './cache/cache.module';

@Module({
  imports: [
    CacheModule.forRoot({
      ttl: 60, // 60 seconds
      max: 100, // 100 items max
      prefix: 'app',
    }),
    UsersModule,
    ProductsModule,
  ],
})
export class AppModule {}
```

#### Step 4: Use forFeature in Feature Modules
**`src/users/users.module.ts`:**
```typescript
import { CacheModule } from '../cache/cache.module';

@Module({
  imports: [CacheModule.forFeature('users')],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

**Use in service:**
```typescript
@Injectable()
export class UsersService {
  constructor(private cacheService: CacheService) {}

  findOne(id: number) {
    const cacheKey = `user:${id}`;
    
    // Try cache first
    const cached = this.cacheService.get(cacheKey);
    if (cached) {
      return cached;
    }

    // Fetch from database
    const user = { id, username: 'john' };
    
    // Store in cache
    this.cacheService.set(cacheKey, user);
    
    return user;
  }
}
```

### Tasks to Complete
- [ ] Create CacheModule with forRoot
- [ ] Implement forFeature method
- [ ] Configure globally with forRoot
- [ ] Use forFeature in modules
- [ ] Test caching functionality

### Expected Outcome
- Configurable dynamic module
- Both global and feature-specific configuration
- Working cache implementation

### Time Estimate
60-90 minutes

---

## Exercise 4: Refactor Monolithic Module

### Objective
Take a large monolithic module and break it into organized feature modules.

### Instructions

#### Given: Monolithic Structure
```typescript
// app.module.ts - Everything in one module
@Module({
  controllers: [
    UsersController,
    ProductsController,
    OrdersController,
    AuthController,
    PaymentsController,
    ShippingController,
  ],
  providers: [
    UsersService,
    ProductsService,
    OrdersService,
    AuthService,
    PaymentsService,
    ShippingService,
    EmailService,
    LoggerService,
  ],
})
export class AppModule {}
```

#### Task: Refactor Into
```
src/
├── modules/
│   ├── users/
│   │   ├── users.module.ts
│   │   ├── users.controller.ts
│   │   └── users.service.ts
│   ├── products/
│   ├── orders/
│   ├── auth/
│   ├── payments/
│   └── shipping/
├── shared/
│   ├── shared.module.ts
│   ├── email.service.ts
│   └── logger.service.ts
└── app.module.ts
```

### Steps
1. Create feature modules for each domain
2. Move controllers and services to respective modules
3. Create SharedModule for common services
4. Set up module dependencies
5. Update imports in AppModule
6. Test all functionality still works

### Expected Outcome
- Clean modular structure
- Clear module boundaries
- Proper dependency management
- Maintainable codebase

### Time Estimate
90-120 minutes

---

## Challenge Exercise: Plugin System

### Objective
Create a plugin system using dynamic modules where plugins can be loaded at runtime.

### Requirements
1. Base PluginModule interface
2. Dynamic plugin registration
3. Plugin discovery mechanism
4. Plugin lifecycle hooks
5. Plugin configuration

### Example Plugins
- LoggingPlugin
- CachePlugin
- MetricsPlugin
- SecurityPlugin

### Time Estimate
180+ minutes

---

## Submission Checklist

- [ ] Feature modules created and organized
- [ ] Shared module with global services
- [ ] Dynamic module with forRoot/forFeature
- [ ] Monolithic app refactored
- [ ] Module dependencies working correctly
- [ ] All code tested and documented

---

## Best Practices Summary

1. ✅ **One feature per module**
2. ✅ **Export only what's needed**
3. ✅ **Use @Global() sparingly**
4. ✅ **Shared utilities in SharedModule**
5. ✅ **Dynamic modules for configuration**
6. ✅ **Clear module naming convention**
7. ✅ **Document module dependencies**

---

## Additional Resources

- [NestJS Modules Documentation](https://docs.nestjs.com/modules)
- [Dynamic Modules](https://docs.nestjs.com/fundamentals/dynamic-modules)
- [Module Reference](https://docs.nestjs.com/fundamentals/module-ref)
- [Circular Dependency](https://docs.nestjs.com/fundamentals/circular-dependency)

---

## Next Steps

Continue to [Module 6 Exercises](module-06-exercises.md) to learn about core fundamentals!
