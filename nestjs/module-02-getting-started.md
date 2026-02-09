# Module 2: Getting Started

## 2.1 Prerequisites

### Required Knowledge
Before starting with NestJS, you should be comfortable with:

#### 1. JavaScript Fundamentals
- ES6+ features (arrow functions, destructuring, spread operator)
- Promises and async/await
- Classes and object-oriented concepts
- Modules (import/export)

#### 2. TypeScript Basics
- Type annotations
- Interfaces and types
- Generics
- Decorators
- tsconfig.json configuration

#### 3. Node.js
- npm/yarn package managers
- Node.js runtime basics
- Understanding of package.json
- Basic command-line usage

### System Requirements

**Node.js**
- Version: **>= 20.0.0** (recommended latest LTS)
- Check version: `node --version`
- Download: https://nodejs.org/

**Package Manager**
- npm (comes with Node.js)
- Or yarn: `npm install -g yarn`
- Or pnpm: `npm install -g pnpm`

**Code Editor**
- VS Code (recommended)
- WebStorm
- Or any TypeScript-enabled editor

### Recommended VS Code Extensions
```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "firsttris.vscode-jest-runner"
  ]
}
```

---

## 2.2 Installation & Setup

### Method 1: Using NestJS CLI (Recommended)

#### Step 1: Install NestJS CLI Globally
```bash
npm i -g @nestjs/cli
```

Verify installation:
```bash
nest --version
```

#### Step 2: Create New Project
```bash
nest new project-name
```

You'll be prompted to choose a package manager:
```
? Which package manager would you ❤️  to use?
❯ npm
  yarn
  pnpm
```

#### Step 3: Navigate to Project
```bash
cd project-name
```

#### Optional: Strict Mode
For stricter TypeScript checking:
```bash
nest new project-name --strict
```

### Method 2: Git Clone Starter

**TypeScript Starter**
```bash
git clone https://github.com/nestjs/typescript-starter.git project
cd project
npm install
```

**Without Git History (using degit)**
```bash
npx degit nestjs/typescript-starter project
cd project
npm install
```

**JavaScript Starter**
```bash
git clone https://github.com/nestjs/javascript-starter.git project
cd project
npm install
```

### Method 3: Manual Setup

#### Step 1: Initialize npm Project
```bash
mkdir my-nest-app
cd my-nest-app
npm init -y
```

#### Step 2: Install Core Dependencies
```bash
npm install @nestjs/core @nestjs/common @nestjs/platform-express reflect-metadata rxjs
```

#### Step 3: Install Dev Dependencies
```bash
npm install -D @nestjs/cli @nestjs/schematics typescript @types/node
```

#### Step 4: Create tsconfig.json
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

---

## 2.3 Project Structure

### Generated Project Structure
```
my-nest-app/
├── node_modules/
├── src/
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test/
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── .eslintrc.js
├── .prettierrc
├── nest-cli.json
├── package.json
├── tsconfig.build.json
├── tsconfig.json
└── README.md
```

### Core Files Explained

#### `src/main.ts` - Application Entry Point
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

**Purpose**: 
- Bootstrap the application
- Create NestJS instance
- Start HTTP server

#### `src/app.module.ts` - Root Module
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

**Purpose**:
- Root module of the application
- Imports other modules
- Declares controllers and providers

#### `src/app.controller.ts` - Basic Controller
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

**Purpose**:
- Handle HTTP requests
- Define routes
- Use services

#### `src/app.service.ts` - Basic Service
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

**Purpose**:
- Business logic
- Data operations
- Reusable functionality

#### `src/app.controller.spec.ts` - Unit Tests
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { AppController } from './app.controller';
import { AppService } from './app.service';

describe('AppController', () => {
  let appController: AppController;

  beforeEach(async () => {
    const app: TestingModule = await Test.createTestingModule({
      controllers: [AppController],
      providers: [AppService],
    }).compile();

    appController = app.get<AppController>(AppController);
  });

  describe('root', () => {
    it('should return "Hello World!"', () => {
      expect(appController.getHello()).toBe('Hello World!');
    });
  });
});
```

**Purpose**:
- Unit testing
- Test isolation
- Mocking dependencies

### Configuration Files

#### `package.json` - Scripts
```json
{
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  }
}
```

#### `nest-cli.json` - NestJS Configuration
```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true
  }
}
```

#### `.eslintrc.js` - Linting Rules
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
    '@typescript-eslint/no-explicit-any': 'off',
  },
};
```

---

## 2.4 Running the Application

### Development Mode

#### Standard Start
```bash
npm run start
```
- Compiles TypeScript
- Starts the application
- Runs on port 3000 (default)

#### Watch Mode (Recommended for Development)
```bash
npm run start:dev
```
- Watches for file changes
- Auto-recompiles
- Auto-restarts server
- Hot reload

#### Debug Mode
```bash
npm run start:debug
```
- Starts with debugger attached
- Debug port: 9229
- Use with VS Code debugger

#### Using SWC Builder (Faster)
```bash
npm run start -- -b swc
```
- Up to 20x faster builds
- Recommended for large projects

**Install SWC**:
```bash
npm i -D @swc/cli @swc/core
```

### Production Mode

#### Build
```bash
npm run build
```
- Compiles to `dist/` folder
- Optimized for production
- No source maps

#### Run Production Build
```bash
npm run start:prod
```
- Runs compiled JavaScript
- No TypeScript compilation
- Better performance

### Verify Running Application

**Test in Browser**
```
http://localhost:3000
```

**Test with cURL**
```bash
curl http://localhost:3000
```

**Expected Response**
```
Hello World!
```

### Port Configuration

**Environment Variable**
```bash
PORT=4000 npm run start:dev
```

**In code (main.ts)**
```typescript
await app.listen(process.env.PORT ?? 3000);
```

**Using .env file**
```bash
# .env
PORT=4000
```

---

## 2.5 Development Tools

### ESLint (Linting)

**Purpose**: Find and fix code problems

**Run Linting**
```bash
npm run lint
```

**Auto-fix Issues**
```bash
npm run lint -- --fix
```

**Configuration**: `.eslintrc.js`

**Example Rules**
```javascript
rules: {
  '@typescript-eslint/no-unused-vars': 'error',
  '@typescript-eslint/no-explicit-any': 'warn',
  'prettier/prettier': 'error',
}
```

### Prettier (Formatting)

**Purpose**: Code formatting

**Run Formatting**
```bash
npm run format
```

**Configuration**: `.prettierrc`
```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 80,
  "tabWidth": 2,
  "semi": true
}
```

**VS Code Integration**
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

### Git Hooks with Husky (Optional)

**Install**
```bash
npm install -D husky lint-staged
npx husky install
```

**Pre-commit Hook**
```bash
npx husky add .husky/pre-commit "npm run lint-staged"
```

**package.json Configuration**
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

### Testing Setup

**Unit Tests**
```bash
npm run test
```

**Watch Mode**
```bash
npm run test:watch
```

**Coverage**
```bash
npm run test:cov
```

**E2E Tests**
```bash
npm run test:e2e
```

---

## Hands-On Exercise

### Exercise 1: Create Your First NestJS App

1. Install NestJS CLI
2. Create a new project called "my-first-nest-app"
3. Run the application in watch mode
4. Verify it's working at http://localhost:3000
5. Modify the message in `app.service.ts`
6. See hot reload in action

### Exercise 2: Explore Project Structure

1. Open each file in `src/` directory
2. Read and understand the code
3. Try to trace the request flow from main.ts to service
4. Run the unit tests
5. Check the test coverage

### Exercise 3: Customize Configuration

1. Change the default port to 4000
2. Modify ESLint rules
3. Change Prettier settings
4. Create a custom npm script

---

## Common Issues & Solutions

### Issue 1: Port Already in Use
```
Error: listen EADDRINUSE: address already in use :::3000
```

**Solution**:
```bash
# Find process using port
lsof -i :3000
# Kill process
kill -9 <PID>
# Or use different port
PORT=4000 npm run start:dev
```

### Issue 2: TypeScript Errors
```
Cannot find module '@nestjs/core'
```

**Solution**:
```bash
npm install
# Or reinstall
rm -rf node_modules package-lock.json
npm install
```

### Issue 3: ESLint/Prettier Conflicts
**Solution**: Ensure correct order in `.eslintrc.js`:
```javascript
extends: [
  'plugin:@typescript-eslint/recommended',
  'plugin:prettier/recommended', // Must be last
],
```

---

## Summary

### What We Learned
1. ✅ Prerequisites for NestJS development
2. ✅ Three methods to create a NestJS project
3. ✅ Understanding project structure
4. ✅ Running application in different modes
5. ✅ Using development tools (ESLint, Prettier)

### Key Commands
```bash
# Create project
nest new project-name

# Development
npm run start:dev

# Build
npm run build

# Production
npm run start:prod

# Test
npm run test

# Lint & Format
npm run lint
npm run format
```

### Next Steps
In Module 3, we'll dive deep into Controllers and learn how to handle HTTP requests!

---

## Additional Resources
- [NestJS CLI Documentation](https://docs.nestjs.com/cli/overview)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

---

## 📚 Course Navigation

⬅️ **[Previous: Module 1 - Introduction to NestJS](module-01-introduction.md)**

🏠 **[Back to Course Outline](course-outline.md)**

➡️ **[Next: Module 3 - Controllers](module-03-controllers.md)**
