# Framework Module Documentation (@eqxjs/stub)

## Overview

The `FrameworkModule` is the core module of the @eqxjs/stub package, serving as the main entry point and bootstrapping mechanism for NestJS applications that utilize the EQXJS ecosystem. This module provides integrated configuration management, service initialization, and seamless integration of all EQXJS ecosystem modules.

## Table of Contents

- [Architecture](#architecture)
- [Core Features](#core-features)
- [Configuration](#configuration)
- [API Reference](#api-reference)
- [Integration Patterns](#integration-patterns)
- [Advanced Usage](#advanced-usage)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Architecture

### Module Structure

```
FrameworkModule (Dynamic Module)
├── DomainConfigModule (Global)
├── HttpClientModule (Global)
├── LoggerModule (Global)
├── UtilModule (Global)
├── SecurityModule (Global)
├── FrameworkUtilService (Provider)
└── GracefulShutdownService (Provider)
```

### Dependency Graph

The FrameworkModule creates a dependency graph that ensures proper initialization order:

1. **Configuration Loading** - YAML config files are parsed based on zone
2. **Global Domain Setting** - `global.targetDomain` is set from config
3. **Module Initialization** - All EQXJS modules are registered with config
4. **Service Provisioning** - Framework utilities and graceful shutdown services are provided

## Core Features

### 1. Configuration Management

- **Zone-based Configuration**: Supports multiple environments (development, staging, production)
- **YAML Configuration Loading**: Automatic parsing of `{zone}.config.yaml` files
- **Global Domain Registration**: Sets `global.targetDomain` for ecosystem-wide access

### 2. Module Integration

- **Unified Registration**: Single point of registration for all EQXJS modules
- **Consistent Configuration**: All modules receive the same configuration object
- **Global Scope**: Modules are registered globally for application-wide access

### 3. Service Provisioning

- **Framework Utilities**: Provides `FrameworkUtilService` for common operations
- **Graceful Shutdown**: Includes `GracefulShutdownService` for clean application termination

## Configuration

### Configuration File Structure

The framework expects YAML configuration files with the following structure:

```yaml
# {zone}.config.yaml
app:
  name: "MyApplication"
  version: "1.0.0"
  component-name: "my-service" # Used for global.targetDomain
  port: 3000
  description: "Application description"

log:
  level: "info"
  detail:
    level: "debug"
    enable-file-logging: true
    enable-console-logging: true
    max-file-size: "20MB"
    max-files: 5
  masking:
    enabled: true
    fields: ["password", "secret", "token"]

security:
  enabled: true
  jwt:
    secret: "${JWT_SECRET}"
    expiration: "1h"
  cors:
    enabled: true
    origins: ["http://localhost:3000"]
  rate-limit:
    ttl: 60
    limit: 10

database:
  type: "mongodb"
  host: "localhost"
  port: 27017
  name: "mydb"
  username: "${DB_USERNAME}"
  password: "${DB_PASSWORD}"

http:
  timeout: 30000
  retries: 3
  base-url: "https://api.example.com"

kafka:
  brokers: ["localhost:9092"]
  group-id: "my-service-group"
  client-id: "my-service"

health:
  enabled: true
  interval: 30000
  endpoints:
    - name: "database"
      url: "mongodb://localhost:27017"
    - name: "external-api"
      url: "https://api.example.com/health"
```

### Environment Variables

The framework supports environment variable substitution in configuration files:

```yaml
database:
  username: "${DB_USERNAME}"
  password: "${DB_PASSWORD}"
  host: "${DB_HOST:localhost}" # localhost as default
```

### Zone-based Environments

```
config/
├── development.config.yaml
├── staging.config.yaml
├── production.config.yaml
└── test.config.yaml
```

## API Reference

### FrameworkModule

#### Static Methods

##### `register(options: FrameworkOptionDto): DynamicModule`

Registers the framework module with the provided configuration options.

**Parameters:**

- `options: FrameworkOptionDto` - Configuration options for the framework
  - `configPath: string` - Path to configuration files directory
  - `zone: string` - Environment zone identifier

**Returns:**
`DynamicModule` - Configured dynamic module with integrated services

**Configuration Process:**

1. Loads YAML configuration from `{configPath}/{zone}.config.yaml`
2. Sets `global.targetDomain` based on `config.app.component-name`
3. Registers all EQXJS ecosystem modules with the loaded configuration
4. Provides framework-specific services

**Example:**

```typescript
@Module({
  imports: [
    FrameworkModule.register({
      configPath: "./assets/config",
      zone: process.env.NODE_ENV || "development",
    }),
  ],
})
export class AppModule {}
```

### Integrated Modules

The FrameworkModule automatically configures and provides the following modules:

#### 1. DomainConfigModule (@corp-ais/eqxjs-commander)

- **Purpose**: Configuration management and command handling
- **Configuration**: Receives `configPath` and `zone`
- **Global**: Yes

#### 2. HttpClientModule (@corp-ais/eqxjs-transporter-http)

- **Purpose**: HTTP client transport layer
- **Configuration**: Uses framework configuration
- **Global**: Yes

#### 3. LoggerModule (@corp-ais/eqxjs-logger)

- **Purpose**: Comprehensive logging capabilities
- **Configuration**: Receives full configuration object
- **Global**: Yes

#### 4. UtilModule (@corp-ais/eqxjs-utils)

- **Purpose**: Utility functions and services
- **Configuration**: Receives full configuration object
- **Global**: Yes

#### 5. SecurityModule (@corp-ais/eqxjs-security)

- **Purpose**: Security utilities and validation
- **Configuration**: Receives full configuration object
- **Global**: Yes

### Framework Services

#### FrameworkUtilService

Utility service providing framework-specific helper functions.

**Methods:**

- Configuration access utilities
- Common validation helpers
- Framework state management

#### GracefulShutdownService

Service for handling graceful application shutdown.

**Key Features:**

- Automatic cleanup registration
- Configurable shutdown timeout
- Resource cleanup orchestration
- Circuit breaker support

**Methods:**

- `setup(app: INestApplication)` - Initialize graceful shutdown
- `addCleanupTask(task: () => Promise<void>)` - Register cleanup function
- `shutdown()` - Trigger graceful shutdown

## Integration Patterns

### 1. Basic Application Setup

```typescript
import { Module } from "@nestjs/common";
import { FrameworkModule } from "@corp-ais/eqxjs-stub";

@Module({
  imports: [
    FrameworkModule.register({
      configPath: "./config",
      zone: "development",
    }),
  ],
})
export class AppModule {}
```

### 2. Multi-Environment Setup

```typescript
const environment = process.env.NODE_ENV || "development";
const configPath = process.env.CONFIG_PATH || "./assets/config";

@Module({
  imports: [
    FrameworkModule.register({
      configPath,
      zone: environment,
    }),
  ],
})
export class AppModule {}
```

### 3. With Custom Module

```typescript
@Module({
  imports: [
    FrameworkModule.register({
      configPath: "./config",
      zone: "production",
    }),
    CustomBusinessModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### 4. Testing Configuration

```typescript
// test/app.module.ts
@Module({
  imports: [
    FrameworkModule.register({
      configPath: "./test/config",
      zone: "test",
    }),
  ],
})
export class TestAppModule {}
```

## Advanced Usage

### 1. Custom Configuration Loading

```typescript
import { Module, Global } from "@nestjs/common";
import { FrameworkModule } from "@corp-ais/eqxjs-stub";
import { ConfigModule } from "@nestjs/config";

@Global()
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: [".env.local", ".env"],
    }),
    FrameworkModule.register({
      configPath: process.env.CONFIG_PATH || "./config",
      zone: process.env.APP_ZONE || "development",
    }),
  ],
})
export class CoreModule {}
```

### 2. Conditional Module Loading

```typescript
@Module({
  imports: [
    FrameworkModule.register({
      configPath: "./config",
      zone: "development",
    }),
    ...(process.env.NODE_ENV === "development"
      ? [
          // Development-only modules
        ]
      : []),
  ],
})
export class AppModule {}
```

### 3. Dynamic Configuration

```typescript
import { DynamicModule } from "@nestjs/common";
import { FrameworkModule } from "@corp-ais/eqxjs-stub";

export class AppModuleFactory {
  static create(zone: string, configPath: string): DynamicModule {
    return {
      module: AppModule,
      imports: [
        FrameworkModule.register({
          configPath,
          zone,
        }),
      ],
    };
  }
}
```

### 4. Service Access Pattern

```typescript
import { Injectable } from "@nestjs/common";
import {
  FrameworkUtilService,
  GracefulShutdownService,
} from "@corp-ais/eqxjs-stub";

@Injectable()
export class MyService {
  constructor(
    private readonly frameworkUtil: FrameworkUtilService,
    private readonly gracefulShutdown: GracefulShutdownService,
  ) {
    // Register cleanup task
    this.gracefulShutdown.addCleanupTask(async () => {
      await this.cleanup();
    });
  }

  async cleanup() {
    // Custom cleanup logic
  }
}
```

## Best Practices

### 1. Configuration Management

**✅ Do:**

- Use environment-specific configuration files
- Leverage environment variables for secrets
- Validate configuration on startup
- Use default values for optional settings

**❌ Don't:**

- Hard-code sensitive information in config files
- Use production secrets in development/test configs
- Ignore configuration validation errors

### 2. Module Registration

**✅ Do:**

- Register FrameworkModule in your root AppModule
- Use consistent naming for configuration zones
- Organize configuration files in a dedicated directory

**❌ Don't:**

- Register FrameworkModule in multiple places
- Mix configuration formats (stick to YAML)
- Override framework-provided global services

### 3. Service Usage

**✅ Do:**

- Inject framework services through constructor
- Use graceful shutdown for cleanup tasks
- Follow dependency injection patterns

**❌ Don't:**

- Access services through global variables
- Bypass the graceful shutdown mechanism
- Create circular dependencies with framework services

### 4. Testing

**✅ Do:**

- Create separate test configurations
- Mock framework services in unit tests
- Test configuration loading separately

**❌ Don't:**

- Use production configurations in tests
- Skip framework service testing
- Ignore configuration validation in tests

## Troubleshooting

### Common Issues

#### 1. Configuration File Not Found

**Error:** `Module not found: ./config/{zone}.config.yaml`

**Solution:**

- Verify the configuration file exists at the specified path
- Check the zone parameter matches the filename
- Ensure the file extension is `.config.yaml`

#### 2. Invalid Configuration Format

**Error:** `Configuration parsing failed`

**Solution:**

- Validate YAML syntax using a YAML parser
- Check for correct indentation and structure
- Verify required configuration sections exist

#### 3. Module Registration Issues

**Error:** `Cannot resolve dependencies of FrameworkModule`

**Solution:**

- Ensure all required EQXJS modules are installed
- Check package.json for missing dependencies
- Verify module import paths

#### 4. Global Domain Not Set

**Error:** `global.targetDomain is undefined`

**Solution:**

- Verify `app.component-name` exists in configuration
- Check configuration file loading order
- Ensure FrameworkModule is registered first

### Debugging

Enable debug logging to troubleshoot issues:

```yaml
# config/development.config.yaml
log:
  level: "debug"
  detail:
    level: "trace"
    enable-console-logging: true
```

### Performance Considerations

- Configuration files are loaded synchronously during module initialization
- Large configuration files may impact startup time
- Consider configuration caching for production environments

## Migration Guide

### From Previous Versions

When migrating from earlier versions of the framework:

1. **Update package references** from `eqxjs/stub` to `@corp-ais/eqxjs-stub`
2. **Review configuration structure** for any breaking changes
3. **Update import statements** to match new module exports
4. **Test graceful shutdown** functionality with new implementation

### Breaking Changes

#### Version 3.x

- Configuration file naming convention changed to `{zone}.config.yaml`
- Global domain setting moved to `app.component-name`
- Some service interfaces have been updated

## Examples Repository

For complete working examples, see the [EQXJS Examples Repository](https://github.com/corp-ais/eqxjs-examples) which includes:

- Basic application setup
- Multi-environment configurations
- Custom service integrations
- Testing patterns
- Production deployment examples

## Support

For additional support and documentation:

- **GitHub Issues**: [Report issues and bugs](https://github.com/corp-ais/cronus-eqxjs-common-library-stub/issues)
- **Documentation**: [Full API documentation](https://github.com/corp-ais/cronus-eqxjs-common-library-stub#readme)
- **Training Materials**: See the training documentation in the training package

---

## Version History

- **3.2.6**: Current version with enhanced documentation and stability improvements
- **3.2.x**: Added graceful shutdown improvements and configuration validation
- **3.1.x**: Enhanced module integration and logging capabilities
- **3.0.x**: Major rewrite with TypeScript improvements and NestJS 11 support
