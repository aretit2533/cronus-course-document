# Module 5: Modules

## 5.1 Introduction to Modules

### What Are Modules?

A module is a class annotated with the `@Module()` decorator. The `@Module()` decorator provides metadata that NestJS uses to organize the application structure.

```
┌─────────────────────────────┐
│       AppModule (Root)      │
└──────────────┬──────────────┘
               │
       ┌───────┴───────┐
       │               │
┌──────▼──────┐ ┌─────▼──────┐
│ UsersModule │ │ CatsModule │
└──────┬──────┘ └─────┬──────┘
       │               │
  ┌────┴────┐     ┌───┴────┐
  │ Feature │     │Feature │
  └─────────┘     └────────┘
```

### Module Decorator

```typescript
import { Module } from '@nestjs/common';

@Module({
  imports: [],      // Other modules
  controllers: [],  // Controllers in this module
  providers: [],    // Providers/services
  exports: [],      // Providers to export
})
export class AppModule {}
```

### Purpose of Modules

1. ✅ **Organization**: Group related functionality
2. ✅ **Encapsulation**: Hiding implementation details
3. ✅ **Reusability**: Share modules across features
4. ✅ **Dependency Management**: Control what's available where
5. ✅ **Separation of Concerns**: Clear boundaries

---

## 5.2 Feature Modules

### Creating a Feature Module

Feature modules organize code relevant for a specific feature.

#### Example: CatsModule

```typescript
// cats/cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

### Project Structure with Modules

```
src/
├── app.module.ts
├── main.ts
├── cats/
│   ├── cats.module.ts
│   ├── cats.controller.ts
│   ├── cats.service.ts
│   ├── dto/
│   │   ├── create-cat.dto.ts
│   │   └── update-cat.dto.ts
│   └── interfaces/
│       └── cat.interface.ts
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── dto/
└── common/
    ├── common.module.ts
    └── services/
```

### Importing Feature Modules

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';
import { UsersModule } from './users/users.module';

@Module({
  imports: [
    CatsModule,
    UsersModule,
  ],
})
export class AppModule {}
```

### Generate Module with CLI

```bash
# Generate a module
nest g module cats

# Generate module with full resource
nest g resource cats

# Generate in subdirectory
nest g module modules/cats
```

---

## 5.3 Shared Modules

### Making Providers Available

By default, modules encapsulate their providers. To share them:

```typescript
// cats/cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],  // ← Export for other modules
})
export class CatsModule {}
```

### Using Exported Providers

```typescript
// dogs/dogs.module.ts
import { Module } from '@nestjs/common';
import { CatsModule } from '../cats/cats.module';  // Import module
import { DogsController } from './dogs.controller';
import { DogsService } from './dogs.service';

@Module({
  imports: [CatsModule],  // Import CatsModule
  controllers: [DogsController],
  providers: [DogsService],
})
export class DogsModule {}
```

```typescript
// dogs/dogs.service.ts
import { Injectable } from '@nestjs/common';
import { CatsService } from '../cats/cats.service';  // Can now inject

@Injectable()
export class DogsService {
  constructor(private catsService: CatsService) {}  // ← Inject CatsService
  
  findAllAnimals() {
    const cats = this.catsService.findAll();
    // Use cats data
  }
}
```

### Singleton Pattern

**Important**: Modules are singletons by default. The same instance of a provider is shared across all modules that import it.

```typescript
// CatsService is the SAME instance in both modules
@Module({
  imports: [CatsModule],  // Shares CatsService instance
})
export class DogsModule {}

@Module({
  imports: [CatsModule],  // Same CatsService instance
})
export class BirdsModule {}
```

---

## 5.4 Module Re-exporting

### Re-exporting Imported Modules

You can re-export modules to simplify imports:

```typescript
// common/common.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from './logger/logger.module';
import { ConfigModule } from './config/config.module';
import { DatabaseModule } from './database/database.module';

@Module({
  imports: [
    LoggerModule,
    ConfigModule,
    DatabaseModule,
  ],
  exports: [
    LoggerModule,    // Re-export
    ConfigModule,    // Re-export
    DatabaseModule,  // Re-export
  ],
})
export class CommonModule {}
```

### Benefits of Re-exporting

```typescript
// Without re-exporting
@Module({
  imports: [
    LoggerModule,
    ConfigModule,
    DatabaseModule,
  ],
})
export class UsersModule {}

// With re-exporting (cleaner)
@Module({
  imports: [CommonModule],  // Gets all three modules
})
export class UsersModule {}
```

### Core Module Pattern

```typescript
// core/core.module.ts
import { Module, Global } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { ConfigModule } from './config/config.module';
import { LoggerModule } from './logger/logger.module';

@Global()  // Available everywhere
@Module({
  imports: [
    DatabaseModule,
    ConfigModule,
    LoggerModule,
  ],
  exports: [
    DatabaseModule,
    ConfigModule,
    LoggerModule,
  ],
})
export class CoreModule {}
```

---

## 5.5 Global Modules

### The @Global() Decorator

Global modules make their providers available everywhere without explicit imports.

```typescript
// database/database.module.ts
import { Module, Global } from '@nestjs/common';
import { DatabaseService } from './database.service';

@Global()  // ← Makes module global
@Module({
  providers: [DatabaseService],
  exports: [DatabaseService],
})
export class DatabaseModule {}
```

### Using Global Modules

```typescript
// app.module.ts - Register once
@Module({
  imports: [DatabaseModule],  // Import once in root
})
export class AppModule {}

// users/users.service.ts - Use anywhere
@Injectable()
export class UsersService {
  // No need to import DatabaseModule in UsersModule
  constructor(private databaseService: DatabaseService) {}
}
```

### When to Use Global Modules

✅ **Good Use Cases**:
- Database connections
- Configuration services
- Logger services
- Cache managers
- Authentication services

❌ **Avoid for**:
- Feature-specific services
- Business logic services
- Most regular services

### Best Practices

```typescript
// ✅ Good: Core services as global
@Global()
@Module({
  providers: [LoggerService, ConfigService],
  exports: [LoggerService, ConfigService],
})
export class CoreModule {}

// ❌ Bad: Feature services as global
@Global()  // Don't do this!
@Module({
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

---

## 5.6 Dynamic Modules

### What Are Dynamic Modules?

Dynamic modules allow you to customize module configuration at runtime.

### Basic Dynamic Module

```typescript
// config/config.module.ts
import { Module, DynamicModule } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}
```

### Using Dynamic Modules

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      folder: './config',
      format: 'json',
    }),
  ],
})
export class AppModule {}
```

### forRoot() vs forFeature()

#### forRoot() - Application-Wide Configuration

```typescript
@Module({})
export class DatabaseModule {
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'DATABASE_CONNECTION',
          useValue: createConnection(options),
        },
      ],
      exports: ['DATABASE_CONNECTION'],
    };
  }
}

// Usage in root module
@Module({
  imports: [
    DatabaseModule.forRoot({
      host: 'localhost',
      port: 5432,
    }),
  ],
})
export class AppModule {}
```

#### forFeature() - Feature-Specific Configuration

```typescript
@Module({})
export class DatabaseModule {
  static forFeature(entities: Entity[]): DynamicModule {
    const providers = entities.map(entity => ({
      provide: `${entity.name}_REPOSITORY`,
      useFactory: (connection: Connection) => {
        return connection.getRepository(entity);
      },
      inject: ['DATABASE_CONNECTION'],
    }));

    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}

// Usage in feature modules
@Module({
  imports: [
    DatabaseModule.forFeature([User, Post]),
  ],
})
export class UsersModule {}
```

### Complete Dynamic Module Example

```typescript
// logger/logger.module.ts
import { Module, DynamicModule, Provider } from '@nestjs/common';
import { LoggerService } from './logger.service';

interface LoggerModuleOptions {
  level: 'debug' | 'info' | 'warn' | 'error';
  prefix?: string;
}

@Module({})
export class LoggerModule {
  static forRoot(options: LoggerModuleOptions): DynamicModule {
    const loggerProvider: Provider = {
      provide: 'LOGGER_OPTIONS',
      useValue: options,
    };

    return {
      module: LoggerModule,
      providers: [loggerProvider, LoggerService],
      exports: [LoggerService],
      global: true,  // Optional: make it global
    };
  }

  static forFeature(prefix: string): DynamicModule {
    return {
      module: LoggerModule,
      providers: [
        {
          provide: 'LOGGER_PREFIX',
          useValue: prefix,
        },
      ],
    };
  }
}
```

### Async Dynamic Modules

For configuration that requires async operations:

```typescript
@Module({})
export class ConfigModule {
  static forRootAsync(options: ConfigModuleAsyncOptions): DynamicModule {
    return {
      module: ConfigModule,
      imports: options.imports || [],
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useFactory: options.useFactory,
          inject: options.inject || [],
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}

// Usage
@Module({
  imports: [
    ConfigModule.forRootAsync({
      imports: [HttpModule],
      useFactory: async (httpService: HttpService) => {
        const config = await httpService.get('/config').toPromise();
        return config.data;
      },
      inject: [HttpService],
    }),
  ],
})
export class AppModule {}
```

---

## 5.7 Module Metadata

### @Module() Decorator Properties

```typescript
@Module({
  imports: [/*...*/],      // Modules to import
  controllers: [/*...*/],  // Controllers in this module
  providers: [/*...*/],    // Providers in this module
  exports: [/*...*/],      // Providers/modules to export
})
```

### Imports

```typescript
@Module({
  imports: [
    UsersModule,
    CatsModule,
    ConfigModule.forRoot(),
  ],
})
export class AppModule {}
```

### Controllers

```typescript
@Module({
  controllers: [
    AppController,
    UsersController,
    CatsController,
  ],
})
export class AppModule {}
```

### Providers

```typescript
@Module({
  providers: [
    AppService,
    UsersService,
    {
      provide: 'DATABASE',
      useFactory: () => createConnection(),
    },
  ],
})
export class AppModule {}
```

### Exports

```typescript
@Module({
  providers: [UsersService, InternalService],
  exports: [UsersService],  // Only UsersService is available to other modules
})
export class UsersModule {}
```

---

## 5.8 Module Best Practices

### 1. One Feature Per Module

```typescript
// ✅ Good
@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
})
export class UsersModule {}

// ❌ Bad
@Module({
  controllers: [UsersController, CatsController, DogsController],
  providers: [UsersService, CatsService, DogsService],
})
export class AnimalsModule {}  // Too many responsibilities
```

### 2. Clear Module Dependencies

```typescript
// ✅ Good: Clear what module needs
@Module({
  imports: [DatabaseModule, ConfigModule],
  providers: [UsersService],
})
export class UsersModule {}

// ❌ Bad: Hidden dependencies
@Module({
  providers: [UsersService],  // Needs DatabaseModule but not imported
})
export class UsersModule {}
```

### 3. Export Only What's Needed

```typescript
// ✅ Good: Export public API only
@Module({
  providers: [UsersService, InternalHelper, PrivateService],
  exports: [UsersService],  // Only public service
})
export class UsersModule {}

// ❌ Bad: Export everything
@Module({
  providers: [UsersService, InternalHelper, PrivateService],
  exports: [UsersService, InternalHelper, PrivateService],
})
export class UsersModule {}
```

### 4. Module Directory Structure

```
src/
├── app.module.ts
├── main.ts
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── users.repository.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   ├── entities/
│   │   └── user.entity.ts
│   └── interfaces/
│       └── user.interface.ts
└── shared/
    ├── shared.module.ts
    └── services/
```

### 5. Shared/Common Module Pattern

```typescript
// shared/shared.module.ts
@Module({
  imports: [
    LoggerModule,
    ConfigModule,
    CacheModule,
  ],
  exports: [
    LoggerModule,
    ConfigModule,
    CacheModule,
  ],
})
export class SharedModule {}

// Use in feature modules
@Module({
  imports: [SharedModule],  // Get all shared dependencies
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

---

## 5.9 Complete Module Example

```typescript
// users/dto/create-user.dto.ts
export class CreateUserDto {
  email: string;
  password: string;
  name: string;
}

// users/entities/user.entity.ts
export class User {
  id: number;
  email: string;
  password: string;
  name: string;
  createdAt: Date;
}

// users/users.repository.ts
import { Injectable } from '@nestjs/common';
import { User } from './entities/user.entity';

@Injectable()
export class UsersRepository {
  private users: User[] = [];
  private idCounter = 1;

  create(userData: Partial<User>): User {
    const user = {
      id: this.idCounter++,
      ...userData,
      createdAt: new Date(),
    } as User;
    this.users.push(user);
    return user;
  }

  findAll(): User[] {
    return this.users;
  }

  findById(id: number): User | undefined {
    return this.users.find(user => user.id === id);
  }

  findByEmail(email: string): User | undefined {
    return this.users.find(user => user.email === email);
  }
}

// users/users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { UsersRepository } from './users.repository';
import { CreateUserDto } from './dto/create-user.dto';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {
  constructor(private usersRepository: UsersRepository) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    return this.usersRepository.create(createUserDto);
  }

  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();
  }

  async findOne(id: number): Promise<User> {
    const user = this.usersRepository.findById(id);
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }

  async findByEmail(email: string): Promise<User | undefined> {
    return this.usersRepository.findByEmail(email);
  }
}

// users/users.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }
}

// users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';

@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],  // Make available to other modules
})
export class UsersModule {}

// app.module.ts
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';

@Module({
  imports: [UsersModule],
})
export class AppModule {}
```

---

## Summary

### Key Concepts

1. ✅ **Modules**: Organize application structure
2. ✅ **Feature Modules**: Group related functionality
3. ✅ **Shared Modules**: Export providers for reuse
4. ✅ **Global Modules**: Available everywhere
5. ✅ **Dynamic Modules**: Runtime configuration
6. ✅ **Module Re-export**: Simplify imports
7. ✅ **forRoot/forFeature**: Configuration patterns

### Best Practices

- ✅ One feature per module
- ✅ Clear dependencies via imports
- ✅ Export only public API
- ✅ Use shared module for common dependencies
- ✅ Keep modules focused and cohesive
- ✅ Use dynamic modules for configuration
- ✅ Document module dependencies

---

## Exercises

### Exercise 1: Create Feature Modules
Create separate modules for:
- Users
- Auth
- Products
- Orders

### Exercise 2: Shared Module
Create a SharedModule that exports:
- LoggerService
- ConfigService
- ValidationService

### Exercise 3: Dynamic Module
Create a CacheModule with:
- forRoot() for global configuration
- forFeature() for feature-specific keys

### Exercise 4: Module Organization
Refactor a monolithic module into:
- Feature modules
- Shared modules
- Core module

---

## Additional Resources

- [NestJS Modules Documentation](https://docs.nestjs.com/modules)
- [Dynamic Modules](https://docs.nestjs.com/fundamentals/dynamic-modules)
- [Module Reference](https://docs.nestjs.com/fundamentals/module-ref)

---

## 📚 Course Navigation

⬅️ **[Previous: Module 4 - Providers](module-04-providers.md)**

🏠 **[Back to Course Outline](course-outline.md)**

➡️ **[Next: Module 6 - Core Fundamentals](module-06-core-fundamentals.md)**
