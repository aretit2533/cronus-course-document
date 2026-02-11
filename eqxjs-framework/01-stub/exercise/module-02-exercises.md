# Module 2 Exercises: Getting Started & Setup

## 📚 Exercise Overview

These exercises will guide you through setting up a complete EQXJS Framework development environment and creating your first applications.

### 🎯 Learning Objectives

- Set up a complete EQXJS development environment
- Create and configure a new EQXJS project
- Implement basic application features
- Configure environment-specific settings
- Test and validate your setup

### ⏱️ Estimated Time: 1.5 hours

---

## 🏁 Exercise 2.1: Development Environment Setup (Quick Start)

### Objective

Set up a complete development environment for EQXJS Framework development.

### Instructions

1. **Verify Prerequisites:**

   ```bash
   # Check Node.js (should be 18+)
   node --version

   # Check npm
   npm --version

   # Install TypeScript globally if needed
   npm install -g typescript
   npx tsc --version
   ```

2. **Create Project Structure:**

   ```bash
   mkdir eqxjs-learning-project
   cd eqxjs-learning-project
   npm init -y
   ```

3. **Install EQXJS Framework:**

   ```bash
   # Install main framework
   npm install @corp-ais/eqxjs-stub

   # Install NestJS peer dependencies
   npm install @nestjs/common @nestjs/core @nestjs/platform-express reflect-metadata rxjs

   # Install development dependencies
   npm install -D @nestjs/cli typescript @types/node ts-node
   ```

4. **Create TypeScript Configuration:**

   ```bash
   npx tsc --init
   ```

   Update `tsconfig.json` with EQXJS-optimized settings:

   ```json
   {
     "compilerOptions": {
       "module": "commonjs",
       "declaration": true,
       "removeComments": true,
       "emitDecoratorMetadata": true,
       "experimentalDecorators": true,
       "allowSyntheticDefaultImports": true,
       "target": "ES2020",
       "sourceMap": true,
       "outDir": "./dist",
       "baseUrl": "./",
       "incremental": true,
       "skipLibCheck": true,
       "strictNullChecks": false,
       "noImplicitAny": false,
       "strictBindCallApply": false,
       "forceConsistentCasingInFileNames": false,
       "noFallthroughCasesInSwitch": false
     }
   }
   ```

### 📝 Tasks

- [ ] Verify all prerequisites are met
- [ ] Create project and install dependencies
- [ ] Configure TypeScript with correct settings
- [ ] Verify installation by checking package versions

### ✅ Validation

Run this command to verify your setup:

```bash
node -e "console.log('EQXJS Version:', require('@corp-ais/eqxjs-stub/package.json').version)"
```

---

## 🔧 Exercise 2.2: Basic Application Structure (Hands-On)

### Objective

Create a complete basic EQXJS application with proper structure.

### Instructions

1. **Create Directory Structure:**

   ```bash
   mkdir -p src/{controllers,services,modules,config}
   mkdir -p config test docs
   ```

2. **Create Main Application Module:**

   Create `src/app.module.ts`:

   ```typescript
   import { Module } from "@nestjs/common";
   import { FrameworkModule } from "@corp-ais/eqxjs-stub";
   import { AppController } from "./controllers/app.controller";
   import { AppService } from "./services/app.service";

   @Module({
     imports: [
       FrameworkModule.register({
         configPath: "./config",
         zone: process.env.NODE_ENV || "development",
       }),
     ],
     controllers: [AppController],
     providers: [AppService],
   })
   export class AppModule {}
   ```

3. **Create Application Service:**

   Create `src/services/app.service.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";

   @Injectable()
   export class AppService {
     getHello(): object {
       return {
         message: "Hello from EQXJS Framework!",
         timestamp: new Date().toISOString(),
         environment: process.env.NODE_ENV || "development",
         version: "1.0.0",
       };
     }

     getStatus(): object {
       return {
         status: "running",
         uptime: process.uptime(),
         memory: process.memoryUsage(),
         platform: process.platform,
         nodeVersion: process.version,
       };
     }
   }
   ```

4. **Create Application Controller:**

   Create `src/controllers/app.controller.ts`:

   ```typescript
   import { Controller, Get } from "@nestjs/common";
   import { AppService } from "../services/app.service";

   @Controller()
   export class AppController {
     constructor(private readonly appService: AppService) {}

     @Get()
     getHello(): object {
       return this.appService.getHello();
     }

     @Get("status")
     getStatus(): object {
       return this.appService.getStatus();
     }
   }
   ```

5. **Create Main Entry Point:**

   Create `src/main.ts`:

   ```typescript
   import { NestFactory } from "@nestjs/core";
   import { AppModule } from "./app.module";

   async function bootstrap() {
     const app = await NestFactory.create(AppModule);

     // Enable graceful shutdown
     app.enableShutdownHooks();

     // Set global prefix
     app.setGlobalPrefix("api/v1");

     const port = process.env.PORT || 3000;
     await app.listen(port);

     console.log(`🚀 Application running on: http://localhost:${port}`);
     console.log(`📋 Health check: http://localhost:${port}/api/v1/health`);
   }

   bootstrap();
   ```

### 📝 Tasks

- [ ] Create proper directory structure
- [ ] Implement AppModule with FrameworkModule
- [ ] Create AppService with business logic
- [ ] Create AppController with endpoints
- [ ] Create main entry point

### ✅ Validation

Build and test your application:

```bash
npx tsc
node dist/main.js
```

---

## 🚀 Exercise 2.3: Environment Configuration (Challenge)

### Objective

Implement comprehensive environment configuration for development, staging, and production.

### Instructions

1. **Create Environment Configuration Files:**

   Create `config/development.config.yaml`:

   ```yaml
   app:
     name: "EQXJS Learning Project"
     version: "1.0.0"
     description: "Development environment"
     debug: true

   server:
     port: 3000
     host: "localhost"
     cors:
       enabled: true
       origins: ["http://localhost:3000", "http://localhost:8080"]

   logging:
     level: "debug"
     format: "pretty"
     timestamp: true

   database:
     type: "mongodb"
     host: "localhost"
     port: 27017
     database: "eqxjs_learning_dev"

   redis:
     host: "localhost"
     port: 6379
     database: 0

   security:
     jwt:
       secret: "dev-secret-key"
       expiresIn: "1d"
     api:
       rateLimit:
         windowMs: 900000
         max: 1000

   targetDomain: "dev.learning.local"
   ```

2. **Create Configuration Service:**

   Create `src/services/config.service.ts`:

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

     isDevelopment(): boolean {
       return this.getEnvironment() === "development";
     }

     isProduction(): boolean {
       return this.getEnvironment() === "production";
     }

     getTargetDomain(): string {
       return global.targetDomain || "localhost";
     }
   }
   ```

3. **Update Package Scripts:**

   Update `package.json` scripts:

   ```json
   {
     "scripts": {
       "build": "tsc",
       "start": "node dist/main.js",
       "start:dev": "NODE_ENV=development npx ts-node src/main.ts",
       "start:staging": "NODE_ENV=staging node dist/main.js",
       "start:prod": "NODE_ENV=production node dist/main.js",
       "start:watch": "NODE_ENV=development npx ts-node --watch src/main.ts"
     }
   }
   ```

### 📝 Tasks

- [ ] Create configuration files for all environments
- [ ] Implement configuration service
- [ ] Add environment-specific scripts
- [ ] Test each environment configuration

---

## 📦 Exercise 2.4: Complete Application with Testing (Project)

### Objective

Build a complete EQXJS application with API endpoints, testing, and documentation.

### Instructions

1. **Install Testing Dependencies:**

   ```bash
   npm install -D jest @types/jest ts-jest supertest @types/supertest
   ```

2. **Create Users Management Feature:**

   Create `src/modules/users.module.ts`:

   ```typescript
   import { Module } from "@nestjs/common";
   import { UsersController } from "../controllers/users.controller";
   import { UsersService } from "../services/users.service";

   @Module({
     controllers: [UsersController],
     providers: [UsersService],
     exports: [UsersService],
   })
   export class UsersModule {}
   ```

3. **Create Tests:**

   Create `src/services/users.service.spec.ts`:

   ```typescript
   import { Test, TestingModule } from "@nestjs/testing";
   import { UsersService } from "./users.service";
   import { NotFoundException } from "@nestjs/common";

   describe("UsersService", () => {
     let service: UsersService;

     beforeEach(async () => {
       const module: TestingModule = await Test.createTestingModule({
         providers: [UsersService],
       }).compile();

       service = module.get<UsersService>(UsersService);
     });

     it("should be defined", () => {
       expect(service).toBeDefined();
     });

     it("should return all users", () => {
       const users = service.findAll();
       expect(users).toHaveLength(2);
     });
   });
   ```

### 📝 Tasks

- [ ] Install testing dependencies and configure Jest
- [ ] Create complete users management feature
- [ ] Write comprehensive tests
- [ ] Create API documentation
- [ ] Run tests and verify all pass

---

## 🎯 Exercise Completion Checklist

Before moving to Module 3, ensure you have completed:

### Exercise 2.1: Environment Setup

- [ ] Successfully installed EQXJS Framework
- [ ] Created project with proper dependencies
- [ ] Configured TypeScript correctly
- [ ] Verified installation

### Exercise 2.2: Application Structure

- [ ] Created proper directory structure
- [ ] Implemented all core files
- [ ] Built and tested application
- [ ] Verified endpoints work

### Exercise 2.3: Environment Configuration

- [ ] Created configuration files for all environments
- [ ] Implemented configuration service
- [ ] Tested different environment settings
- [ ] Verified configuration loading

### Exercise 2.4: Complete Application

- [ ] Built full user management feature
- [ ] Created comprehensive tests
- [ ] Achieved good test coverage
- [ ] Documented API endpoints

## 📚 Learning Reflection

Reflect on your experience:

1. **What was most challenging** about setting up the EQXJS project?
2. **How does the framework setup** compare to plain NestJS?
3. **What benefits** do you see from the configuration-driven approach?
4. **What questions** do you have about the framework architecture?

## ➡️ Next Steps

Congratulations! You now have a working EQXJS application. Next:

1. **Review your code** and ensure all tests pass
2. **Experiment** with different configuration options
3. **Prepare for Module 3** - Framework Module Configuration
4. **Continue to** [Module 3 Exercises](module-03-exercises.md)

---

**Excellent work on setting up your EQXJS development environment! 🎉**
