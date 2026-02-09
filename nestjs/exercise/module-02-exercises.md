# Module 2 Exercises: Getting Started

## Overview
These hands-on exercises will help you set up your development environment and create your first NestJS applications.

---

## Exercise 1: Create Your First NestJS Application

### Objective
Successfully install NestJS CLI and create a working NestJS project.

### Prerequisites
- Node.js >= 20.0.0 installed
- npm or yarn package manager
- Code editor (VS Code recommended)

### Instructions

#### Step 1: Install NestJS CLI
```bash
# Install globally
npm install -g @nestjs/cli

# Verify installation
nest --version
```

**Expected Output:**
```
10.x.x
```

#### Step 2: Create New Project
```bash
# Create project
nest new my-first-nest-app

# Choose package manager (npm recommended for beginners)
# Wait for installation to complete
```

#### Step 3: Explore Generated Files
Navigate to the project and verify the structure:
```bash
cd my-first-nest-app
ls -la src/
```

**Expected files in `src/`:**
- `main.ts` - Entry point
- `app.module.ts` - Root module
- `app.controller.ts` - Controller
- `app.controller.spec.ts` - Controller tests
- `app.service.ts` - Service

#### Step 4: Run the Application
```bash
# Development mode with hot reload
npm run start:dev
```

**Expected Output:**
```
[Nest] 12345  - 01/01/2024, 10:00:00 AM   LOG [NestFactory] Starting Nest application...
[Nest] 12345  - 01/01/2024, 10:00:00 AM   LOG [InstanceLoader] AppModule dependencies initialized
[Nest] 12345  - 01/01/2024, 10:00:00 AM   LOG [RoutesResolver] AppController {/}:
[Nest] 12345  - 01/01/2024, 10:00:00 AM   LOG [RouterExplorer] Mapped {/, GET} route
[Nest] 12345  - 01/01/2024, 10:00:00 AM   LOG [NestApplication] Nest application successfully started
```

#### Step 5: Test the Application
Open browser or use curl:
```bash
curl http://localhost:3000
```

**Expected Response:**
```
Hello World!
```

#### Step 6: Modify the Response
Edit `src/app.service.ts`:
```typescript
@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello NestJS! My first application is running!';
  }
}
```

Save the file and watch the hot reload in action.

#### Step 7: Test Your Change
```bash
curl http://localhost:3000
```

**Expected Response:**
```
Hello NestJS! My first application is running!
```

### Tasks to Complete
- [ ] Install NestJS CLI
- [ ] Create new project
- [ ] Run application successfully
- [ ] Verify "Hello World" response
- [ ] Modify the message
- [ ] See hot reload work
- [ ] Stop the application (Ctrl+C)

### Expected Outcome
- Working NestJS application
- Understanding of project creation
- Experience with hot reload
- Confidence in basic setup

### Time Estimate
20-30 minutes

### Troubleshooting

**Issue: Port 3000 already in use**
```bash
# Find and kill process
lsof -i :3000
kill -9 <PID>

# Or run on different port
PORT=3001 npm run start:dev
```

**Issue: Command not found: nest**
```bash
# Reinstall CLI
npm uninstall -g @nestjs/cli
npm install -g @nestjs/cli

# Or use npx
npx @nestjs/cli new my-app
```

---

## Exercise 2: Explore Project Structure

### Objective
Understand every file in the NestJS project structure and their purposes.

### Instructions

#### Step 1: Document Each File
Create a file `project-structure-notes.md` documenting each file:

```markdown
# Project Structure Analysis

## Root Directory

### package.json
- **Purpose**: Project metadata and dependencies
- **Key Dependencies**:
  - @nestjs/core: Core framework
  - @nestjs/common: Common utilities and decorators
  - rxjs: Reactive programming
  - reflect-metadata: Decorator metadata
- **Key Scripts**:
  - start:dev: Development with watch mode
  - build: Compile TypeScript
  - test: Run unit tests

### tsconfig.json
- **Purpose**: TypeScript compiler configuration
- **Key Settings**:
  - experimentalDecorators: Enable decorators
  - emitDecoratorMetadata: Metadata for DI
  - target: ES2021 JavaScript output

### nest-cli.json
- **Purpose**: NestJS CLI configuration
- **Key Settings**:
  - sourceRoot: src directory
  - collection: @nestjs/schematics

## Source Directory (src/)

### main.ts
[Continue documenting...]
```

#### Step 2: Trace Request Flow
Document the complete flow of a request:

1. **Request arrives** at `http://localhost:3000`
2. **main.ts** - Application bootstrap
   ```typescript
   NestFactory.create(AppModule)
   ```
3. **app.module.ts** - Module registers controller
   ```typescript
   controllers: [AppController]
   ```
4. **app.controller.ts** - Route handler
   ```typescript
   @Get() → calls service
   ```
5. **app.service.ts** - Business logic
   ```typescript
   getHello() → returns string
   ```
6. **Response sent** back to client

#### Step 3: Modify and Observe
Make these changes and note what happens:

1. **Change port in main.ts**
   ```typescript
   await app.listen(4000);
   ```

2. **Add route prefix in app.controller.ts**
   ```typescript
   @Controller('api')
   ```

3. **Add new method in app.service.ts**
   ```typescript
   getVersion(): string {
     return 'v1.0.0';
   }
   ```

4. **Add new route in app.controller.ts**
   ```typescript
   @Get('version')
   getVersion(): string {
     return this.appService.getVersion();
   }
   ```

#### Step 4: Run Tests
```bash
# Unit tests
npm run test

# Test coverage
npm run test:cov

# E2E tests
npm run test:e2e
```

Document the test results and understand what's being tested.

### Tasks to Complete
- [ ] Document all configuration files
- [ ] Trace complete request flow
- [ ] Make modifications and test
- [ ] Run all test suites
- [ ] Understand each component's role

### Expected Outcome
- Deep understanding of project structure
- Ability to navigate codebase confidently
- Knowledge of request flow
- Understanding of testing setup

### Time Estimate
45-60 minutes

---

## Exercise 3: Multiple Installation Methods

### Objective
Practice all three installation methods and compare them.

### Instructions

#### Method 1: NestJS CLI (Already Done)
✅ Completed in Exercise 1

#### Method 2: Git Clone Starter
```bash
# TypeScript starter
git clone https://github.com/nestjs/typescript-starter.git nest-git-clone
cd nest-git-clone
npm install
npm run start:dev
```

**Compare with CLI method:**
- What's different?
- What's the same?
- When would you use this?

#### Method 3: Manual Setup
```bash
# Create project manually
mkdir nest-manual-setup
cd nest-manual-setup
npm init -y
```

**Install dependencies:**
```bash
npm install @nestjs/core @nestjs/common @nestjs/platform-express reflect-metadata rxjs
npm install -D @nestjs/cli @nestjs/schematics typescript @types/node ts-node
```

**Create tsconfig.json:**
```json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2021",
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

**Create src/main.ts:**
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log(`Application is running on: ${await app.getUrl()}`);
}
bootstrap();
```

**Create src/app.module.ts:**
```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**Create src/app.controller.ts:**
```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

**Create src/app.service.ts:**
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello from manual setup!';
  }
}
```

**Add scripts to package.json:**
```json
{
  "scripts": {
    "start": "ts-node src/main.ts",
    "start:dev": "nest start --watch",
    "build": "nest build",
    "test": "jest"
  }
}
```

**Run the application:**
```bash
npm run start
```

#### Step 4: Create Comparison Document
Create `installation-methods-comparison.md`:

```markdown
# Installation Methods Comparison

## CLI Method
**Pros:**
- Fastest setup
- Complete configuration
- Includes test setup
- Best practices out of the box

**Cons:**
- Requires global CLI installation
- Less control over initial setup

**Best For:**
- New projects
- Standard applications
- Quick prototyping

## Git Clone Method
**Pros:**
- No CLI required
- Clean git history
- Official template

**Cons:**
- Must manually remove git history
- Less convenient than CLI

**Best For:**
- No global npm packages
- Template customization
- Offline development

## Manual Method
**Pros:**
- Complete control
- Understanding every piece
- Custom configuration

**Cons:**
- Time-consuming
- Error-prone
- Must know all dependencies

**Best For:**
- Learning purposes
- Custom setups
- Integration into existing projects
```

### Tasks to Complete
- [ ] Try all three installation methods
- [ ] Run each project successfully
- [ ] Compare generated structures
- [ ] Document pros and cons
- [ ] Decide preference for future use

### Expected Outcome
- Experience with all setup methods
- Understanding of when to use each
- Independence in project creation

### Time Estimate
60-90 minutes

---

## Exercise 4: Development Workflow Setup

### Objective
Set up a productive development environment with proper tooling.

### Instructions

#### Step 1: Install VS Code Extensions
```bash
# Create .vscode/extensions.json
mkdir .vscode
```

**Create `.vscode/extensions.json`:**
```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "firsttris.vscode-jest-runner",
    "christian-kohler.npm-intellisense",
    "christian-kohler.path-intellisense",
    "usernamehw.errorlens"
  ]
}
```

#### Step 2: Configure Prettier
Create `.prettierrc`:
```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "semi": true
}
```

#### Step 3: Configure ESLint
Update `.eslintrc.js`:
```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: 'tsconfig.json',
    tsconfigRootDir: __dirname,
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint/eslint-plugin'],
  extends: [
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
  ],
  root: true,
  env: {
    node: true,
    jest: true,
  },
  ignorePatterns: ['.eslintrc.js'],
  rules: {
    '@typescript-eslint/interface-name-prefix': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
  },
};
```

#### Step 4: Add Custom Scripts
Update `package.json` scripts:
```json
{
  "scripts": {
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "format:check": "prettier --check \"src/**/*.ts\" \"test/**/*.ts\"",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "lint:check": "eslint \"{src,apps,libs,test}/**/*.ts\"",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  }
}
```

#### Step 5: Set Up Git Hooks (Optional)
```bash
npm install -D husky lint-staged

# Initialize husky
npx husky init
```

**Create `.husky/pre-commit`:**
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint:check
npm run format:check
npm run test
```

**Add to package.json:**
```json
{
  "lint-staged": {
    "*.ts": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

#### Step 6: Create .env Setup
```bash
npm install @nestjs/config
```

**Create `.env`:**
```
PORT=3000
NODE_ENV=development
```

**Create `.env.example`:**
```
PORT=3000
NODE_ENV=development
```

**Update `main.ts`:**
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const port = process.env.PORT || 3000;
  await app.listen(port);
  
  console.log(`🚀 Application is running on: http://localhost:${port}`);
}
bootstrap();
```

#### Step 7: Create README.md
Document your project:
```markdown
# My NestJS Application

## Description
[Your project description]

## Installation
\`\`\`bash
npm install
\`\`\`

## Running the app
\`\`\`bash
# development
npm run start:dev

# production mode
npm run start:prod
\`\`\`

## Test
\`\`\`bash
# unit tests
npm run test

# e2e tests
npm run test:e2e

# test coverage
npm run test:cov
\`\`\`
```

### Tasks to Complete
- [ ] Install VS Code extensions
- [ ] Configure Prettier and ESLint
- [ ] Add custom npm scripts
- [ ] Set up environment variables
- [ ] Test all scripts work
- [ ] Create comprehensive README
- [ ] (Optional) Set up Git hooks

### Expected Outcome
- Professional development environment
- Automated code formatting
- Consistent code style
- Clear documentation

### Time Estimate
45-60 minutes

---

## Exercise 5: Create Multi-Environment Setup

### Objective
Set up configuration for different environments (development, staging, production).

### Instructions

#### Step 1: Install Dependencies
```bash
npm install @nestjs/config
npm install -D cross-env
```

#### Step 2: Create Environment Files
```bash
# .env.development
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost/nestapp_dev
LOG_LEVEL=debug

# .env.staging
PORT=3000
NODE_ENV=staging
DATABASE_URL=mongodb://staging-server/nestapp
LOG_LEVEL=info

# .env.production
PORT=8080
NODE_ENV=production
DATABASE_URL=mongodb://prod-server/nestapp
LOG_LEVEL=error
```

#### Step 3: Create Config Module
**Create `src/config/configuration.ts`:**
```typescript
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: process.env.NODE_ENV || 'development',
  database: {
    url: process.env.DATABASE_URL,
  },
  logging: {
    level: process.env.LOG_LEVEL || 'info',
  },
});
```

#### Step 4: Update App Module
```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import configuration from './config/configuration';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [configuration],
      envFilePath: `.env.${process.env.NODE_ENV || 'development'}`,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

#### Step 5: Use Configuration
**Update `main.ts`:**
```typescript
import { NestFactory } from '@nestjs/core';
import { ConfigService } from '@nestjs/config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const configService = app.get(ConfigService);
  const port = configService.get<number>('port');
  const env = configService.get<string>('environment');
  
  await app.listen(port);
  console.log(`🚀 Application running in ${env} mode on port ${port}`);
}
bootstrap();
```

#### Step 6: Update Scripts
**Add to `package.json`:**
```json
{
  "scripts": {
    "start:dev": "cross-env NODE_ENV=development nest start --watch",
    "start:staging": "cross-env NODE_ENV=staging nest start",
    "start:prod": "cross-env NODE_ENV=production node dist/main",
    "build:staging": "cross-env NODE_ENV=staging nest build",
    "build:prod": "cross-env NODE_ENV=production nest build"
  }
}
```

#### Step 7: Test Each Environment
```bash
# Development
npm run start:dev

# Staging
npm run start:staging

# Production build
npm run build:prod
npm run start:prod
```

### Tasks to Complete
- [ ] Create environment files
- [ ] Set up ConfigModule
- [ ] Use configuration in code
- [ ] Test each environment
- [ ] Document environment setup

### Expected Outcome
- Multi-environment configuration
- Environment-specific settings
- Production-ready setup

### Time Estimate
30-45 minutes

---

## Challenge Exercise: Custom Project Template

### Objective
Create your own project template with custom configuration.

### Instructions

1. **Create New Project**
   ```bash
   nest new my-template --strict
   ```

2. **Add Custom Features**
   - Winston logger
   - Helmet security
   - Rate limiting
   - Compression
   - Custom exception filters
   - Health check endpoint

3. **Document Everything**
   - Comprehensive README
   - Contributing guidelines
   - Setup instructions

4. **Create Script**
   Write a script to bootstrap new projects from your template

### Time Estimate
120+ minutes

---

## Submission Checklist

- [ ] First NestJS app created and running
- [ ] Project structure documented
- [ ] All installation methods tested
- [ ] Development workflow configured
- [ ] Multi-environment setup working
- [ ] All scripts tested and documented

---

## Additional Resources

- [NestJS CLI Documentation](https://docs.nestjs.com/cli/overview)
- [VS Code Tips](https://code.visualstudio.com/docs/typescript/typescript-tutorial)
- [TypeScript Configuration](https://www.typescriptlang.org/tsconfig)
- [ESLint Rules](https://eslint.org/docs/rules/)
- [Prettier Options](https://prettier.io/docs/en/options.html)

---

## Next Steps

Proceed to [Module 3 Exercises](module-03-exercises.md) to learn about controllers and routing!
