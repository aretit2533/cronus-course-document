# Module 3 Exercises: Framework Module Configuration

## 📚 Exercise Overview

These exercises focus on mastering EQXJS Framework configuration patterns, zone-based settings, and advanced configuration techniques.

### 🎯 Learning Objectives

- Master FrameworkModule configuration options
- Implement zone-based configuration management
- Create custom configuration providers
- Validate configuration schemas
- Apply configuration best practices

### ⏱️ Estimated Time: 2 hours

---

## 🏁 Exercise 3.1: Basic Framework Configuration (Quick Start)

### Objective

Set up basic FrameworkModule configuration with different zones.

### Instructions

1. **Create Multi-Zone Configuration:**

   Create `config/base.config.yaml`:

   ```yaml
   app:
     name: "EQXJS Configuration Demo"
     version: "1.0.0"
     description: "Base configuration"

   logging:
     timestamp: true
     context: true

   server:
     timeout: 30000
     cors:
       enabled: true
   ```

   Create `config/development.config.yaml`:

   ```yaml
   app:
     debug: true

   server:
     port: 3000
     host: "localhost"
     cors:
       origins: ["http://localhost:3000", "http://localhost:8080"]

   logging:
     level: "debug"
     format: "pretty"

   database:
     type: "mongodb"
     host: "localhost"
     port: 27017
     database: "demo_dev"

   targetDomain: "dev.demo.local"
   ```

2. **Create Configuration Module:**

   Create `src/config/config.module.ts`:

   ```typescript
   import { Module, Global } from "@nestjs/common";
   import { ConfigService } from "./config.service";

   @Global()
   @Module({
     providers: [ConfigService],
     exports: [ConfigService],
   })
   export class ConfigModule {}
   ```

   Create `src/config/config.service.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";

   @Injectable()
   export class ConfigService {
     getEnvironment(): string {
       return process.env.NODE_ENV || "development";
     }

     getPort(): number {
       return parseInt(process.env.PORT) || 3000;
     }

     getTargetDomain(): string {
       return global.targetDomain || "localhost";
     }

     isDevelopment(): boolean {
       return this.getEnvironment() === "development";
     }

     isProduction(): boolean {
       return this.getEnvironment() === "production";
     }

     getDatabaseConfig(): any {
       return {
         type: process.env.DB_TYPE || "mongodb",
         host: process.env.DB_HOST || "localhost",
         port: parseInt(process.env.DB_PORT) || 27017,
         database: process.env.DB_NAME || "demo_dev",
       };
     }
   }
   ```

3. **Update App Module:**

   ```typescript
   import { Module } from "@nestjs/common";
   import { FrameworkModule } from "@eqxjs-stub";
   import { ConfigModule } from "./config/config.module";
   import { AppController } from "./app.controller";
   import { AppService } from "./app.service";

   @Module({
     imports: [
       FrameworkModule.register({
         configPath: "./config",
         zone: process.env.NODE_ENV || "development",
         enableHealthCheck: true,
         enableGracefulShutdown: true,
       }),
       ConfigModule,
     ],
     controllers: [AppController],
     providers: [AppService],
   })
   export class AppModule {}
   ```

### 📝 Tasks

- [ ] Create base and zone-specific configuration files
- [ ] Implement ConfigService with environment methods
- [ ] Configure FrameworkModule with all options
- [ ] Test configuration loading in different zones

---

## 🔧 Exercise 3.2: Advanced Configuration Patterns (Hands-On)

### Objective

Implement advanced configuration patterns including validation, secrets, and dynamic updates.

### Instructions

1. **Create Configuration Schema:**

   Create `src/config/config.schema.ts`:

   ```typescript
   import * as Joi from "joi";

   export const configSchema = Joi.object({
     app: Joi.object({
       name: Joi.string().required(),
       version: Joi.string()
         .pattern(/^\d+\.\d+\.\d+$/)
         .required(),
       description: Joi.string().required(),
       debug: Joi.boolean().default(false),
     }).required(),

     server: Joi.object({
       port: Joi.number().port().required(),
       host: Joi.string().required(),
       timeout: Joi.number().positive().default(30000),
       cors: Joi.object({
         enabled: Joi.boolean().default(true),
         origins: Joi.array().items(Joi.string()).default(["*"]),
         methods: Joi.array()
           .items(Joi.string())
           .default(["GET", "POST", "PUT", "DELETE"]),
       }),
     }).required(),

     logging: Joi.object({
       level: Joi.string().valid("error", "warn", "info", "debug").required(),
       format: Joi.string().valid("json", "pretty").required(),
       timestamp: Joi.boolean().default(true),
       context: Joi.boolean().default(true),
     }).required(),

     database: Joi.object({
       type: Joi.string().valid("mongodb", "postgresql", "mysql").required(),
       host: Joi.string().required(),
       port: Joi.number().port().required(),
       database: Joi.string().required(),
       username: Joi.string(),
       password: Joi.string(),
       ssl: Joi.boolean().default(false),
     }).required(),

     security: Joi.object({
       jwt: Joi.object({
         secret: Joi.string().min(32).required(),
         expiresIn: Joi.string().required(),
       }),
       encryption: Joi.object({
         algorithm: Joi.string().default("aes-256-gcm"),
         key: Joi.string().required(),
       }),
     }),

     targetDomain: Joi.string().domain().required(),
   });
   ```

2. **Create Configuration Validator Service:**

   Create `src/config/config-validator.service.ts`:

   ```typescript
   import { Injectable, BadRequestException } from "@nestjs/common";
   import { configSchema } from "./config.schema";

   export interface ConfigValidationResult {
     isValid: boolean;
     errors?: string[];
     config?: any;
   }

   @Injectable()
   export class ConfigValidatorService {
     validateConfiguration(config: any): ConfigValidationResult {
       const { error, value } = configSchema.validate(config, {
         allowUnknown: false,
         abortEarly: false,
       });

       if (error) {
         return {
           isValid: false,
           errors: error.details.map((detail) => detail.message),
         };
       }

       return {
         isValid: true,
         config: value,
       };
     }

     validateAndThrow(config: any): any {
       const result = this.validateConfiguration(config);

       if (!result.isValid) {
         throw new BadRequestException({
           message: "Configuration validation failed",
           errors: result.errors,
         });
       }

       return result.config;
     }
   }
   ```

3. **Create Environment Variable Handler:**

   Create `src/config/env-handler.service.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";

   @Injectable()
   export class EnvHandlerService {
     substituteEnvironmentVariables(config: any): any {
       const configStr = JSON.stringify(config);
       const substitutedStr = configStr.replace(
         /\$\{([^}]+)\}/g,
         (match, envVar) => {
           const value = process.env[envVar];
           if (value === undefined) {
             throw new Error(`Environment variable ${envVar} is not defined`);
           }
           return value;
         },
       );

       return JSON.parse(substitutedStr);
     }

     validateRequiredEnvVars(requiredVars: string[]): void {
       const missingVars = requiredVars.filter(
         (varName) => !process.env[varName],
       );

       if (missingVars.length > 0) {
         throw new Error(
           `Missing required environment variables: ${missingVars.join(", ")}`,
         );
       }
     }

     getEnvVar(name: string, defaultValue?: string): string {
       const value = process.env[name];
       if (value === undefined) {
         if (defaultValue !== undefined) {
           return defaultValue;
         }
         throw new Error(`Environment variable ${name} is required`);
       }
       return value;
     }
   }
   ```

4. **Create Production Configuration:**

   Create `config/production.config.yaml`:

   ```yaml
   app:
     debug: false

   server:
     port: 8080
     host: "0.0.0.0"
     timeout: 15000
     cors:
       origins: ["https://myapp.com"]

   logging:
     level: "warn"
     format: "json"

   database:
     type: "mongodb"
     host: "${DB_HOST}"
     port: ${DB_PORT}
     database: "${DB_NAME}"
     username: "${DB_USERNAME}"
     password: "${DB_PASSWORD}"
     ssl: true

   security:
     jwt:
       secret: "${JWT_SECRET}"
       expiresIn: "2h"
     encryption:
       key: "${ENCRYPTION_KEY}"

   targetDomain: "api.myapp.com"
   ```

### 📝 Tasks

- [ ] Create comprehensive configuration schema
- [ ] Implement configuration validation service
- [ ] Create environment variable substitution
- [ ] Add production configuration with secrets
- [ ] Test validation with valid and invalid configs

---

## 🚀 Exercise 3.3: Custom Configuration Provider (Challenge)

### Objective

Create a custom configuration provider that loads configuration from multiple sources.

### Instructions

1. **Create Multi-Source Configuration Provider:**

   Create `src/config/multi-source-config.provider.ts`:

   ```typescript
   import { Injectable, Logger } from "@nestjs/common";
   import * as fs from "fs";
   import * as yaml from "js-yaml";
   import * as path from "path";

   export interface ConfigSource {
     name: string;
     priority: number;
     loader: () => Promise<any>;
   }

   @Injectable()
   export class MultiSourceConfigProvider {
     private readonly logger = new Logger(MultiSourceConfigProvider.name);
     private sources: ConfigSource[] = [];

     addSource(source: ConfigSource): void {
       this.sources.push(source);
       this.sources.sort((a, b) => b.priority - a.priority);
     }

     async loadConfiguration(): Promise<any> {
       let mergedConfig = {};

       for (const source of this.sources) {
         try {
           this.logger.debug(`Loading configuration from ${source.name}`);
           const config = await source.loader();

           if (config) {
             mergedConfig = this.mergeDeep(mergedConfig, config);
             this.logger.debug(
               `Successfully loaded configuration from ${source.name}`,
             );
           }
         } catch (error) {
           this.logger.warn(
             `Failed to load configuration from ${source.name}: ${error.message}`,
           );
         }
       }

       return mergedConfig;
     }

     private mergeDeep(target: any, source: any): any {
       const output = Object.assign({}, target);

       if (this.isObject(target) && this.isObject(source)) {
         Object.keys(source).forEach((key) => {
           if (this.isObject(source[key])) {
             if (!(key in target)) {
               Object.assign(output, { [key]: source[key] });
             } else {
               output[key] = this.mergeDeep(target[key], source[key]);
             }
           } else {
             Object.assign(output, { [key]: source[key] });
           }
         });
       }

       return output;
     }

     private isObject(item: any): boolean {
       return item && typeof item === "object" && !Array.isArray(item);
     }
   }
   ```

2. **Create Configuration Sources:**

   Create `src/config/sources/yaml-source.ts`:

   ```typescript
   import * as fs from "fs";
   import * as yaml from "js-yaml";
   import * as path from "path";
   import { ConfigSource } from "../multi-source-config.provider";

   export function createYamlSource(
     configPath: string,
     zone: string,
   ): ConfigSource {
     return {
       name: "YAML Files",
       priority: 1,
       loader: async () => {
         const files = [
           `${configPath}/base.config.yaml`,
           `${configPath}/${zone}.config.yaml`,
           `${configPath}/local.config.yaml`,
         ];

         let config = {};

         for (const file of files) {
           if (fs.existsSync(file)) {
             const fileContent = fs.readFileSync(file, "utf8");
             const yamlConfig = yaml.load(fileContent) as any;
             config = { ...config, ...yamlConfig };
           }
         }

         return config;
       },
     };
   }
   ```

   Create `src/config/sources/env-source.ts`:

   ```typescript
   import { ConfigSource } from "../multi-source-config.provider";

   export function createEnvSource(): ConfigSource {
     return {
       name: "Environment Variables",
       priority: 2,
       loader: async () => {
         return {
           app: {
             debug: process.env.APP_DEBUG === "true",
           },
           server: {
             port: process.env.PORT ? parseInt(process.env.PORT) : undefined,
             host: process.env.HOST,
           },
           database: {
             host: process.env.DB_HOST,
             port: process.env.DB_PORT
               ? parseInt(process.env.DB_PORT)
               : undefined,
             database: process.env.DB_NAME,
             username: process.env.DB_USERNAME,
             password: process.env.DB_PASSWORD,
           },
           targetDomain: process.env.TARGET_DOMAIN,
         };
       },
     };
   }
   ```

3. **Create Enhanced Configuration Service:**

   Update `src/config/config.service.ts`:

   ```typescript
   import { Injectable, OnModuleInit, Logger } from "@nestjs/common";
   import { BehaviorSubject, Observable } from "rxjs";
   import { MultiSourceConfigProvider } from "./multi-source-config.provider";
   import { ConfigValidatorService } from "./config-validator.service";
   import { EnvHandlerService } from "./env-handler.service";
   import { createYamlSource } from "./sources/yaml-source";
   import { createEnvSource } from "./sources/env-source";

   @Injectable()
   export class ConfigService implements OnModuleInit {
     private readonly logger = new Logger(ConfigService.name);
     private config: any;
     private configSubject = new BehaviorSubject<any>(null);

     constructor(
       private readonly multiSourceProvider: MultiSourceConfigProvider,
       private readonly validator: ConfigValidatorService,
       private readonly envHandler: EnvHandlerService,
     ) {}

     async onModuleInit() {
       await this.loadConfiguration();
     }

     private async loadConfiguration(): Promise<void> {
       try {
         // Setup configuration sources
         this.multiSourceProvider.addSource(
           createYamlSource("./config", process.env.NODE_ENV || "development"),
         );
         this.multiSourceProvider.addSource(createEnvSource());

         // Load and merge configuration from all sources
         let config = await this.multiSourceProvider.loadConfiguration();

         // Substitute environment variables
         config = this.envHandler.substituteEnvironmentVariables(config);

         // Validate configuration
         config = this.validator.validateAndThrow(config);

         // Store final configuration
         this.config = config;
         this.configSubject.next(config);

         this.logger.log("Configuration loaded and validated successfully");
       } catch (error) {
         this.logger.error("Failed to load configuration", error);
         throw error;
       }
     }

     get<T = any>(path: string, defaultValue?: T): T {
       const keys = path.split(".");
       let value = this.config;

       for (const key of keys) {
         if (value && typeof value === "object" && key in value) {
           value = value[key];
         } else {
           return defaultValue;
         }
       }

       return value;
     }

     getConfigObservable(): Observable<any> {
       return this.configSubject.asObservable();
     }

     async reloadConfiguration(): Promise<void> {
       await this.loadConfiguration();
     }
   }
   ```

### 📝 Tasks

- [ ] Implement multi-source configuration provider
- [ ] Create YAML and environment variable sources
- [ ] Build enhanced configuration service
- [ ] Test configuration loading from multiple sources
- [ ] Verify configuration merging and priority

---

## 🎯 Exercise Completion Checklist

### Exercise 3.1: Basic Framework Configuration

- [ ] Created multi-zone configuration files
- [ ] Implemented basic ConfigService
- [ ] Configured FrameworkModule with all options
- [ ] Tested zone-based configuration loading

### Exercise 3.2: Advanced Configuration Patterns

- [ ] Built comprehensive configuration schema
- [ ] Implemented configuration validation
- [ ] Created environment variable substitution
- [ ] Added production configuration with secrets

### Exercise 3.3: Custom Configuration Provider

- [ ] Built multi-source configuration provider
- [ ] Created configuration source implementations
- [ ] Enhanced configuration service with observables
- [ ] Tested complex configuration scenarios

## 📚 Learning Reflection

After completing these exercises:

1. **What benefits** do you see from zone-based configuration?
2. **How important** is configuration validation in production?
3. **What challenges** might arise with multi-source configuration?
4. **How would you handle** sensitive configuration in different environments?

## ➡️ Next Steps

Excellent work on mastering EQXJS configuration! Continue to:

👉 **[Module 4 Exercises](module-04-exercises.md)** - Health Checks & Monitoring

---

**Outstanding job mastering configuration management! 🎉**
