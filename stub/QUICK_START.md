# Framework Module Quick Start Guide (@eqxjs/stub)

## Overview

This guide helps you quickly get started with the @corp-ais/eqxjs-stub Framework Module in your NestJS application.

## Installation

```bash
npm install @corp-ais/eqxjs-stub
```

## Basic Setup

### 1. Create Configuration File

Create a configuration directory and add your environment-specific config:

```bash
mkdir config
touch config/development.config.yaml
```

**config/development.config.yaml:**

```yaml
app:
  name: "MyApplication"
  version: "1.0.0"
  component-name: "my-service" # Required: Used for global.targetDomain

log:
  level: "debug"

security:
  enabled: true

database:
  host: "localhost"
  port: 27017
  name: "myapp"
```

### 2. Register Framework Module

**app.module.ts:**

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

### 3. Setup Application Bootstrap

**main.ts:**

```typescript
import { NestFactory } from '@nestjs/core';
import { GracefulShutdownService } from '@corp-ais/eqxjs-stub';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Setup graceful shutdown
  const gracefulShutdown = app.get(GracefulShutdownService);
  gracefulShutdown.setup(app);

  // Ensure shutdown hooks are enabled
  app.enableShutdownHooks();

  await app.listen(3000);
  console.log(`Application running on: ${await app.getUrl()}`);
}

bootstrap().catch(err => {
  console.error('Failed to start application:', err);
  process.exit(1);
});
```

## Common Usage Patterns

### Using Entry Point Decorators

```typescript
import { Injectable } from '@nestjs/common';
import { Payload } from '@nestjs/microservices';
import { EntryPoint, ConsumerMasking } from '@corp-ais/eqxjs-stub';

@Injectable()
export class UserService {
  @EntryPoint('USER_CREATED')
  @ConsumerMasking(['password', 'email'])
  async handleUserCreated(@Payload() data: CreateUserDto) {
    console.log('Processing user created event:', data);
    // Add your business logic here
  }
}
```

### Adding Request Interceptors

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { AppInterceptor, HttpInterceptor } from '@corp-ais/eqxjs-stub';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: AppInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: HttpInterceptor,
    },
  ],
})
export class AppModule {}
```

### Using Framework Services

```typescript
import { Injectable } from '@nestjs/common';
import {
  FrameworkUtilService,
  GracefulShutdownService,
} from '@corp-ais/eqxjs-stub';

@Injectable()
export class MyService {
  constructor(
    private readonly frameworkUtil: FrameworkUtilService,
    private readonly gracefulShutdown: GracefulShutdownService,
  ) {
    // Register cleanup task
    this.gracefulShutdown.addCleanupTask(async () => {
      console.log('Cleaning up resources...');
      // Add your cleanup logic here
    });
  }

  async performBusinessLogic() {
    // Use framework utilities
    // Implementation here
  }
}
```
  ) {
    // Register cleanup task
    this.gracefulShutdown.addCleanupTask(async () => {
      console.log("Cleaning up resources...");
      // Add your cleanup logic here
    });
  }
}
```

## Environment-Specific Configuration

### Multiple Environments

Create separate config files for each environment:

```bash
config/
├── development.config.yaml
├── staging.config.yaml
├── production.config.yaml
└── test.config.yaml
```

## Environment-Specific Configuration

### Multiple Environments

Create separate config files for each environment:

```bash
config/
├── development.config.yaml
├── staging.config.yaml
├── production.config.yaml
└── test.config.yaml
```

### Dynamic Zone Selection

```typescript
import { Module } from '@nestjs/common';
import { FrameworkModule } from '@corp-ais/eqxjs-stub';

const environment = process.env.NODE_ENV || 'development';
const configPath = process.env.CONFIG_PATH || './config';

@Module({
  imports: [
    FrameworkModule.register({
      configPath,
      zone: environment
    })
  ]
})
export class AppModule {}
```

## Configuration Reference

### Required Configuration Sections

```yaml
app:
  component-name: "service-name"  # REQUIRED: Sets global.targetDomain
  name: "Application Name"
  version: "1.0.0"
```

### Optional but Recommended Sections

```yaml
log:
  level: "info"                    # debug, info, warn, error
  detail:
    level: "debug"
    enable-file-logging: true
    enable-console-logging: true

security:
  enabled: true
  cors:
    enabled: true
    origins: ["http://localhost:3000"]

database:
  type: "mongodb"
  host: "localhost"  
  port: 27017
  name: "mydb"

http:
  timeout: 30000
  retries: 3

health:
  enabled: true
  interval: 30000
```

## Environment Variables

Use environment variables for sensitive data:

```yaml
database:
  username: "${DB_USERNAME}"
  password: "${DB_PASSWORD}"
  host: "${DB_HOST:localhost}"  # localhost as default
```

Set in your environment:

```bash
export DB_USERNAME=myuser
export DB_PASSWORD=mypassword
export DB_HOST=prod-db.example.com
```

## Testing Setup

### Test Configuration

**config/test.config.yaml:**

```yaml
app:
  component-name: "test-service"
  name: "Test Application"

log:
  level: "error"  # Minimize test output
  
database:
  host: "localhost"
  port: 27017
  name: "test-db"
```

### Test Module

**test/app.module.ts:**

```typescript
import { Module } from '@nestjs/common';
import { FrameworkModule } from '@corp-ais/eqxjs-stub';

@Module({
  imports: [
    FrameworkModule.register({
      configPath: './config',
      zone: 'test',
    }),
  ],
})
export class TestAppModule {}
```

## Troubleshooting

### Common Issues

#### Configuration File Not Found

```
Error: ENOENT: no such file or directory, open './config/development.config.yaml'
```

**Solution:** Ensure the config file exists at the correct path with proper naming convention.

#### Missing component-name

```
Error: global.targetDomain is undefined
```

**Solution:** Add `app.component-name` to your configuration file:

```yaml
app:
  component-name: "my-service"
```

#### Module Import Issues

```
Error: Nest can't resolve dependencies of FrameworkModule
```

**Solution:** Ensure all @corp-ais/eqxjs-\* dependencies are installed:

```bash
npm install @corp-ais/eqxjs-commander @corp-ais/eqxjs-logger # etc.
```

## Next Steps

1. **Read Full Documentation**: [Framework Module Documentation](./FRAMEWORK_MODULE_DOCUMENTATION.md)
2. **API Reference**: [Complete API Reference](./API_REFERENCE.md)
3. **Training Materials**: See training documentation in the project
4. **Examples**: Check the example applications in the workspace

## Support

- **GitHub Issues**: [Report issues](https://github.com/corp-ais/cronus-eqxjs-common-library-stub/issues)
- **Documentation**: [Full documentation](https://github.com/corp-ais/cronus-eqxjs-common-library-stub#readme)

---

That's it! You now have the EQXJS Framework Module configured and ready to use in your NestJS application.
