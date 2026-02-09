# Module 1 Exercises: Introduction to NestJS

## Overview
These exercises will help you understand NestJS concepts, philosophy, and architecture through research and conceptual analysis.

---

## Exercise 1: NestJS vs Other Frameworks Comparison

### Objective
Compare NestJS with other Node.js frameworks to understand its unique value proposition.

### Instructions

1. **Research Phase**
   - Read about Express.js, Fastify, Koa, and Hapi
   - Understand their architecture and design patterns
   - Note their strengths and weaknesses

2. **Create Comparison Document**
   Create a markdown file `framework-comparison.md` with the following structure:
   
   ```markdown
   # Node.js Framework Comparison
   
   ## Express.js
   - Architecture:
   - Strengths:
   - Weaknesses:
   - Best Use Cases:
   
   ## NestJS
   - Architecture:
   - Strengths:
   - Weaknesses:
   - Best Use Cases:
   
   ## Comparison Table
   | Feature | Express | NestJS | Fastify | Koa |
   |---------|---------|--------|---------|-----|
   | TypeScript Support | | | | |
   | Built-in Structure | | | | |
   | DI Container | | | | |
   | Testing Support | | | | |
   | Learning Curve | | | | |
   
   ## Conclusion
   When to choose NestJS vs alternatives
   ```

3. **Analysis Questions**
   Answer these questions in your document:
   - Why does NestJS exist when Express is already popular?
   - What problems does NestJS solve that Express doesn't?
   - When would you NOT choose NestJS?

### Expected Outcome
- A comprehensive comparison document
- Understanding of NestJS's unique positioning
- Clear criteria for when to use NestJS

### Time Estimate
45-60 minutes

---

## Exercise 2: SOLID Principles in Action

### Objective
Understand and apply SOLID principles with practical examples.

### Instructions

1. **Create Example Files**
   Create a folder `solid-examples/` with the following files:
   - `single-responsibility.ts`
   - `open-closed.ts`
   - `liskov-substitution.ts`
   - `interface-segregation.ts`
   - `dependency-inversion.ts`

2. **For Each Principle**
   Write two code examples:
   - ❌ **Bad Example**: Violates the principle
   - ✅ **Good Example**: Follows the principle
   
   Example for Single Responsibility:
   
   ```typescript
   // ❌ BAD: UserService has multiple responsibilities
   class UserService {
     createUser(data) { /* ... */ }
     sendEmail(email, message) { /* ... */ }
     logActivity(message) { /* ... */ }
     validateUser(data) { /* ... */ }
   }
   
   // ✅ GOOD: Each class has one responsibility
   class UserService {
     constructor(
       private emailService: EmailService,
       private logger: LoggerService,
       private validator: ValidationService
     ) {}
     
     createUser(data) {
       this.validator.validate(data);
       const user = /* create user */;
       this.emailService.sendWelcome(user.email);
       this.logger.log('User created');
       return user;
     }
   }
   
   class EmailService {
     sendWelcome(email: string) { /* ... */ }
   }
   
   class LoggerService {
     log(message: string) { /* ... */ }
   }
   
   class ValidationService {
     validate(data: any) { /* ... */ }
   }
   ```

3. **Write Explanations**
   For each example, add comments explaining:
   - Why the bad example violates the principle
   - How the good example follows the principle
   - Real-world benefits of the good approach

### Expected Outcome
- 5 files with clear examples of each SOLID principle
- Deep understanding of why SOLID matters
- Ability to recognize and fix violations

### Time Estimate
90-120 minutes

---

## Exercise 3: Architecture Design Document

### Objective
Design the architecture for a real-world NestJS application.

### Instructions

1. **Choose an Application**
   Pick one of these scenarios:
   - E-commerce platform
   - Social media application
   - Task management system
   - Learning management system

2. **Create Architecture Document**
   Create `architecture-design.md` with:
   
   ```markdown
   # [Application Name] Architecture
   
   ## 1. Application Overview
   - Purpose
   - Key Features
   - User Types
   
   ## 2. Module Structure
   ```
   app.module
   ├── auth.module
   ├── users.module
   ├── products.module
   └── ...
   ```
   
   ## 3. Module Details
   ### AuthModule
   - **Responsibilities**: Authentication, authorization
   - **Controllers**: AuthController
   - **Providers**: AuthService, JwtStrategy
   - **Dependencies**: UsersModule
   
   ### UsersModule
   - **Responsibilities**: User management
   - **Controllers**: UsersController
   - **Providers**: UsersService, UsersRepository
   - **Exports**: UsersService (for AuthModule)
   
   [Continue for each module...]
   
   ## 4. Data Flow Diagrams
   [Create Mermaid diagrams]
   
   ## 5. Design Decisions
   - Why this module structure?
   - How does it follow NestJS best practices?
   - How does it implement SOLID principles?
   
   ## 6. Scalability Considerations
   - How will this scale?
   - What are potential bottlenecks?
   - Future extension points
   ```

3. **Create Mermaid Diagrams**
   Include at least:
   - Module dependency diagram
   - Request flow diagram
   - Authentication flow diagram

### Expected Outcome
- Complete architecture document
- Clear module boundaries
- Understanding of dependency management
- Practical application of NestJS philosophy

### Time Estimate
120-180 minutes

---

## Exercise 4: OOP vs FP in NestJS

### Objective
Understand when to use Object-Oriented vs Functional Programming approaches in NestJS.

### Instructions

1. **Create Comparison File**
   Create `oop-vs-fp.md` with examples

2. **Implement User Management**
   Show both approaches:
   
   **OOP Approach:**
   ```typescript
   // user.entity.ts
   export class User {
     constructor(
       public id: string,
       public name: string,
       public email: string
     ) {}
     
     updateName(name: string): User {
       this.name = name;
       return this;
     }
     
     isAdmin(): boolean {
       return this.email.endsWith('@admin.com');
     }
   }
   
   // user.service.ts
   @Injectable()
   export class UserService {
     private users: User[] = [];
     
     createUser(name: string, email: string): User {
       const user = new User(uuid(), name, email);
       this.users.push(user);
       return user;
     }
   }
   ```
   
   **FP Approach:**
   ```typescript
   // user.types.ts
   export interface User {
     id: string;
     name: string;
     email: string;
   }
   
   // user.functions.ts
   export const createUser = (name: string, email: string): User => ({
     id: uuid(),
     name,
     email
   });
   
   export const updateUserName = (user: User, name: string): User => ({
     ...user,
     name
   });
   
   export const isAdmin = (user: User): boolean =>
     user.email.endsWith('@admin.com');
   
   // user.service.ts
   @Injectable()
   export class UserService {
     private users: User[] = [];
     
     createUser(name: string, email: string): User {
       const user = createUser(name, email);
       this.users = [...this.users, user];
       return user;
     }
   }
   ```

3. **Write Analysis**
   Compare both approaches:
   - Pros and cons of each
   - When to use which
   - How NestJS supports both
   - Hybrid approach possibilities

### Expected Outcome
- Working examples of both paradigms
- Understanding when to use each
- Ability to mix approaches effectively

### Time Estimate
60-90 minutes

---

## Exercise 5: Platform Comparison

### Objective
Understand Express vs Fastify platforms in NestJS.

### Instructions

1. **Create Two Projects**
   ```bash
   # Express-based (default)
   nest new nestjs-express
   
   # Fastify-based
   nest new nestjs-fastify
   cd nestjs-fastify
   npm i @nestjs/platform-fastify
   ```

2. **Modify Fastify Bootstrap**
   Update `main.ts`:
   ```typescript
   import { NestFactory } from '@nestjs/core';
   import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
   import { AppModule } from './app.module';
   
   async function bootstrap() {
     const app = await NestFactory.create<NestFastifyApplication>(
       AppModule,
       new FastifyAdapter()
     );
     await app.listen(3000);
   }
   bootstrap();
   ```

3. **Benchmark Performance**
   Create a simple endpoint and use `autocannon` or `wrk`:
   ```bash
   npm i -D autocannon
   
   # Benchmark Express
   autocannon -c 100 -d 40 http://localhost:3000
   
   # Benchmark Fastify
   autocannon -c 100 -d 40 http://localhost:3001
   ```

4. **Document Results**
   Create `platform-comparison.md`:
   - Performance metrics
   - Code differences
   - Developer experience
   - When to choose each

### Expected Outcome
- Hands-on experience with both platforms
- Performance comparison data
- Understanding of trade-offs

### Time Estimate
45-60 minutes

---

## Challenge Exercise: Mini Framework Analysis

### Objective
Deep dive into understanding how NestJS works internally.

### Instructions

1. **Explore NestJS Source Code**
   - Clone NestJS repository
   - Navigate through `@nestjs/core` package
   - Understand key decorators implementation

2. **Document Findings**
   Create `framework-internals.md`:
   - How does `@Controller()` work?
   - How does dependency injection work?
   - How are modules bootstrapped?
   - What role does `reflect-metadata` play?

3. **Create Mini Version**
   Try to create a simplified version of core NestJS features:
   ```typescript
   // mini-nest.ts
   function Controller(path: string) {
     return function(target: any) {
       Reflect.defineMetadata('path', path, target);
     };
   }
   
   function Get(path: string) {
     return function(target: any, propertyKey: string) {
       // Store route metadata
     };
   }
   
   // Continue building...
   ```

### Expected Outcome
- Deep understanding of NestJS internals
- Appreciation for framework complexity
- Better debugging skills

### Time Estimate
180+ minutes

---

## Submission Checklist

- [ ] Framework comparison document completed
- [ ] All 5 SOLID principle examples created
- [ ] Architecture design document with diagrams
- [ ] OOP vs FP comparison with working code
- [ ] Platform comparison with benchmarks
- [ ] (Optional) Framework internals analysis

---

## Additional Challenges

1. **Research Angular Architecture**
   - Study Angular's architecture
   - Identify similarities with NestJS
   - Document the shared concepts

2. **Design Patterns Study**
   - Research common design patterns
   - Identify which patterns NestJS uses
   - Create examples of each

3. **TypeScript Deep Dive**
   - Study decorators in depth
   - Understand metadata reflection
   - Learn about experimental features

---

## Resources

- [NestJS Documentation](https://docs.nestjs.com/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [TypeScript Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [Angular Architecture](https://angular.dev/guide/architecture)
- [Design Patterns](https://refactoring.guru/design-patterns)

---

## Next Steps

Once completed, move to [Module 2 Exercises](module-02-exercises.md) to start building with NestJS!
