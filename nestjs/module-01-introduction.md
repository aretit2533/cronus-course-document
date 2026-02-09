# Module 1: Introduction to NestJS

## 1.1 What is NestJS?

### Overview
NestJS is a progressive Node.js framework for building efficient, reliable, and scalable server-side applications. It uses modern JavaScript, is built with TypeScript (while still supporting pure JavaScript), and combines elements of Object-Oriented Programming (OOP), Functional Programming (FP), and Functional Reactive Programming (FRP).

### Key Characteristics
- **Progressive**: Adopts the latest JavaScript features and best practices
- **TypeScript First**: Built with TypeScript but JavaScript compatible
- **Node.js Framework**: Runs on the Node.js runtime
- **Server-Side Focus**: Designed for backend applications

### Under the Hood
NestJS leverages robust HTTP server frameworks:
- **Express** (default): Battle-tested, minimalist web framework
- **Fastify** (optional): High-performance, low-overhead alternative

### Key Benefits
1. **Abstraction Layer**: Provides a level of abstraction above Express/Fastify
2. **Direct API Access**: Still exposes underlying platform APIs
3. **Third-Party Compatibility**: Works with the vast ecosystem of Node.js modules
4. **Flexibility**: Freedom to use platform-specific features when needed

---

## 1.2 Philosophy & Architecture

### The Problem NestJS Solves
While JavaScript has become the universal language of the web, and Node.js offers countless libraries and tools, there was a critical gap: **architecture**. Most Node.js libraries don't solve the fundamental challenge of structuring large-scale applications.

### NestJS Solution
NestJS provides an **out-of-the-box application architecture** that enables:
- ✅ **Highly Testable** applications
- ✅ **Scalable** systems
- ✅ **Loosely Coupled** components
- ✅ **Easily Maintainable** codebases

### Angular Inspiration
The architecture is heavily inspired by Angular, bringing enterprise-grade patterns to Node.js:
- Modular structure
- Dependency injection
- Decorators for metadata
- Clear separation of concerns

### Architectural Principles

#### 1. Modularity
```
Application
├── UserModule
│   ├── UserController
│   ├── UserService
│   └── UserRepository
├── AuthModule
│   ├── AuthController
│   ├── AuthService
│   └── JwtStrategy
└── ProductModule
    ├── ProductController
    ├── ProductService
    └── ProductRepository
```

#### 2. Separation of Concerns
- **Controllers**: Handle HTTP requests and responses
- **Providers/Services**: Contain business logic
- **Modules**: Organize and encapsulate features
- **Middleware/Guards/Interceptors**: Cross-cutting concerns

#### 3. Dependency Injection
```typescript
@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}
}
```

### Design Philosophy Goals
1. **Developer Experience**: Intuitive, productive development
2. **Enterprise-Ready**: Suitable for large-scale applications
3. **Best Practices**: Enforces good architectural patterns
4. **Flexibility**: Doesn't lock you into specific implementations

---

## 1.3 Core Concepts

### Object-Oriented Programming (OOP)
NestJS embraces OOP principles:

**Classes as Building Blocks**
```typescript
export class CatsController {
  constructor(private catsService: CatsService) {}
}
```

**Encapsulation**
- Private methods and properties
- Clear interfaces
- Information hiding

**Inheritance**
```typescript
export class AdminController extends BaseController {
  // Inherits base functionality
}
```

**Polymorphism**
- Interface implementations
- Abstract classes
- Method overriding

### Functional Programming (FP)
NestJS supports functional patterns:

**Pure Functions**
```typescript
export const calculateTotal = (items: Item[]): number => {
  return items.reduce((sum, item) => sum + item.price, 0);
};
```

**Immutability**
```typescript
const updatedUser = { ...user, name: 'New Name' };
```

**Higher-Order Functions**
```typescript
const processData = (transformer: (data: any) => any) => {
  return (data: any) => transformer(data);
};
```

### Functional Reactive Programming (FRP)
NestJS integrates RxJS for reactive programming:

**Observable Streams**
```typescript
@Get()
findAll(): Observable<Cat[]> {
  return of([]);
}
```

**Operators**
```typescript
return this.httpService.get('/api/data').pipe(
  map(response => response.data),
  catchError(error => of([]))
);
```

### SOLID Principles

#### S - Single Responsibility Principle
Each class should have one reason to change.
```typescript
// Good: Service handles only business logic
@Injectable()
export class UserService {
  async createUser(userData: CreateUserDto): Promise<User> {
    // Only user creation logic
  }
}
```

#### O - Open/Closed Principle
Open for extension, closed for modification.
```typescript
// Use abstract classes and interfaces
export abstract class PaymentStrategy {
  abstract processPayment(amount: number): Promise<boolean>;
}
```

#### L - Liskov Substitution Principle
Subtypes must be substitutable for their base types.
```typescript
export interface Logger {
  log(message: string): void;
}

export class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(message);
  }
}
```

#### I - Interface Segregation Principle
Clients shouldn't depend on interfaces they don't use.
```typescript
// Split large interfaces into smaller ones
export interface Readable {
  read(): string;
}

export interface Writable {
  write(data: string): void;
}
```

#### D - Dependency Inversion Principle
Depend on abstractions, not concretions.
```typescript
@Injectable()
export class OrderService {
  constructor(
    private readonly paymentService: IPaymentService, // Depends on interface
  ) {}
}
```

---

## 1.4 Platform Support

### Express (Default)

**About Express**
- Well-known minimalist web framework
- Battle-tested and production-ready
- Vast community and resources
- Extensive middleware ecosystem

**When to Use Express**
- ✅ Starting a new NestJS project
- ✅ Need extensive middleware support
- ✅ Team familiar with Express
- ✅ Many third-party integrations

**Installation**
```bash
# Included by default
npm i @nestjs/platform-express
```

**Usage**
```typescript
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  await app.listen(3000);
}
```

### Fastify (Alternative)

**About Fastify**
- High-performance framework
- Low overhead
- Maximum efficiency and speed
- Modern plugin architecture

**When to Use Fastify**
- ✅ Performance-critical applications
- ✅ High throughput requirements
- ✅ Need lower resource consumption
- ✅ Working with large-scale APIs

**Installation**
```bash
npm i @nestjs/platform-fastify
```

**Usage**
```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter()
  );
  await app.listen(3000);
}
```

### Platform-Agnostic Design

**Benefits**
1. **Flexibility**: Switch platforms without major refactoring
2. **Future-Proof**: Not locked into one HTTP framework
3. **Adapter Pattern**: Clean abstraction layer

**Platform Differences**
```typescript
// Express-specific
app.use(cors());
app.set('view engine', 'pug');

// Fastify-specific
app.register(require('fastify-cors'));
app.register(require('point-of-view'), { engine: { pug: require('pug') } });
```

**Best Practice**
Use NestJS abstractions when possible to maintain platform independence:
```typescript
// Platform-agnostic approach
app.enableCors();
app.setGlobalPrefix('api');
```

### Comparison Table

| Feature | Express | Fastify |
|---------|---------|---------|
| Performance | Good | Excellent |
| Ecosystem | Massive | Growing |
| Learning Curve | Easy | Moderate |
| Middleware | Extensive | Plugin-based |
| TypeScript Support | Good | Excellent |
| Validation | Manual | Built-in |
| Default in NestJS | Yes | No |

---

## Summary

### Key Takeaways
1. ✅ NestJS is a progressive, TypeScript-based Node.js framework
2. ✅ Provides out-of-the-box architecture for scalable applications
3. ✅ Combines OOP, FP, and FRP paradigms
4. ✅ Follows SOLID principles for maintainable code
5. ✅ Platform-agnostic with Express and Fastify support

### Why Choose NestJS?
- **For Teams**: Standardized architecture and patterns
- **For Enterprises**: Scalable, testable, maintainable solutions
- **For Developers**: Productive, enjoyable development experience
- **For Projects**: Long-term maintainability and growth

### Moving Forward
In the next module, we'll set up our development environment and create our first NestJS application!

---

## Additional Resources
- [Official NestJS Website](https://nestjs.com/)
- [NestJS Documentation](https://docs.nestjs.com/)
- [TypeScript Documentation](https://www.typescriptlang.org/)
- [Angular Architecture Guide](https://angular.dev/guide/architecture)
- [SOLID Principles Explained](https://en.wikipedia.org/wiki/SOLID)

## Quiz Questions
1. What are the three programming paradigms that NestJS combines?
2. Which HTTP frameworks does NestJS support out-of-the-box?
3. What architectural framework inspired NestJS?
4. What does SOLID stand for?
5. What is the main problem that NestJS solves in the Node.js ecosystem?

---

## 📚 Course Navigation

⬅️ **[Back to Course Outline](course-outline.md)**

➡️ **[Next: Module 2 - Getting Started](module-02-getting-started.md)**
