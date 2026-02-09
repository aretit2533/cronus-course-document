# NestJS Course Outline
## Section: Introduction, Overview & Fundamentals

---

## 📖 Course Modules

| Module | Topic | Content | Exercises |
|--------|-------|---------|-----------|
| Module 1 | Introduction to NestJS | [View Content](module-01-introduction.md) | [Exercises](exercise/module-01-exercises.md) |
| Module 2 | Getting Started | [View Content](module-02-getting-started.md) | [Exercises](exercise/module-02-exercises.md) |
| Module 3 | Controllers | [View Content](module-03-controllers.md) | [Exercises](exercise/module-03-exercises.md) |
| Module 4 | Providers | [View Content](module-04-providers.md) | [Exercises](exercise/module-04-exercises.md) |
| Module 5 | Modules | [View Content](module-05-modules.md) | [Exercises](exercise/module-05-exercises.md) |
| Module 6 | Core Fundamentals | [View Content](module-06-core-fundamentals.md) | [Exercises](exercise/module-06-exercises.md) |
| Module 7 | Additional Fundamentals | [View Content](module-07-additional-fundamentals.md) | [Exercises](exercise/module-07-exercises.md) |
| Module 8 | Practical Application | [View Content](module-08-practical-application.md) | [Exercises](exercise/module-08-exercises.md) |

### 💡 [Complete Exercise Guide](exercise/README.md)
Access comprehensive hands-on exercises for all modules.

---

## Module 1: Introduction to NestJS

📚 **[View Module 1: Introduction to NestJS](module-01-introduction.md)**

### 1.1 What is NestJS?
- Overview of NestJS framework
- Node.js server-side applications
- Progressive JavaScript framework
- Built with TypeScript support

### 1.2 Philosophy & Architecture
- Out-of-the-box application architecture
- Highly testable and scalable applications
- Loosely coupled, maintainable code
- Angular-inspired architecture
- SOLID principles

### 1.3 Core Concepts
- OOP (Object Oriented Programming)
- FP (Functional Programming)
- FRP (Functional Reactive Programming)
- Dependency Injection pattern

### 1.4 Platform Support
- Express (default HTTP framework)
- Fastify (alternative HTTP framework)
- Platform-agnostic design
- HTTP server abstraction layer

---

## Module 2: Getting Started

📚 **[View Module 2: Getting Started](module-02-getting-started.md)**

### 2.1 Prerequisites
- Node.js (version >= 20)
- npm or yarn package manager
- TypeScript basics
- JavaScript ES6+ knowledge

### 2.2 Installation & Setup
- Installing NestJS CLI globally
- Creating a new project with CLI
- Project scaffolding
- Understanding the project structure
- TypeScript strict mode

### 2.3 Project Structure
```
src/
├── app.controller.ts       # Basic controller with single route
├── app.controller.spec.ts  # Unit tests for controller
├── app.module.ts           # Root module of application
├── app.service.ts          # Basic service
└── main.ts                 # Entry file (bootstrap)
```

### 2.4 Running the Application
- Starting the development server
- Using the watch mode (`start:dev`)
- Building for production
- Using SWC builder for faster builds
- Default port configuration

### 2.5 Development Tools
- ESLint configuration
- Prettier formatting
- Linting and auto-fixing
- Code formatting standards

---

## Module 3: Controllers

📚 **[View Module 3: Controllers](module-03-controllers.md)**

### 3.1 Introduction to Controllers
- What are controllers?
- Handling incoming requests
- Sending responses to clients
- Routing mechanism
- Controller decorator `@Controller()`

### 3.2 Routing Basics
- Defining routes
- Route path prefixes
- HTTP method decorators
  - `@Get()`
  - `@Post()`
  - `@Put()`
  - `@Delete()`
  - `@Patch()`
  - `@Options()`
  - `@Head()`
  - `@All()`

### 3.3 Request Handling
- Request object access
- Request decorators:
  - `@Req()` / `@Request()`
  - `@Res()` / `@Response()`
  - `@Next()`
  - `@Session()`
  - `@Param(key?: string)`
  - `@Body(key?: string)`
  - `@Query(key?: string)`
  - `@Headers(name?: string)`
  - `@Ip()`
  - `@HostParam()`

### 3.4 Route Parameters
- Dynamic route parameters
- Parameter tokens in paths
- Using `@Param()` decorator
- Accessing route parameters
- Parameter order best practices

### 3.5 Request Payloads
- Handling POST requests
- Data Transfer Objects (DTOs)
- Using `@Body()` decorator
- Creating DTOs with classes
- TypeScript interfaces vs classes
- Validation and pipes

### 3.6 Query Parameters
- Extracting query parameters
- Using `@Query()` decorator
- Handling complex query strings
- Nested objects and arrays
- Query string parsers (Express vs Fastify)

### 3.7 Response Handling
- Standard response approach (recommended)
- Automatic JSON serialization
- Library-specific response objects
- Using `@Res()` with passthrough option

### 3.8 Advanced Routing
- Route wildcards and patterns
- Named wildcards
- Status codes with `@HttpCode()`
- Custom response headers with `@Header()`
- Redirects with `@Redirect()`
- Sub-domain routing with host option

### 3.9 Asynchronous Operations
- Async/await support
- Returning Promises
- RxJS Observable streams
- Automatic subscription handling

### 3.10 Controller Registration
- Registering controllers in modules
- Using `@Module()` decorator
- Controllers array in module metadata

### 3.11 CLI Tools
- Generating controllers: `nest g controller [name]`
- CRUD generator: `nest g resource [name]`
- Built-in validation support

---

## Module 4: Providers

📚 **[View Module 4: Providers](module-04-providers.md)**

### 4.1 Introduction to Providers
- What are providers?
- Core concept in NestJS
- Services, repositories, factories, helpers
- Dependency injection principle
- `@Injectable()` decorator

### 4.2 Services
- Creating a service class
- Service responsibilities
- Business logic separation
- Data storage and retrieval
- Using CLI: `nest g service [name]`

### 4.3 Dependency Injection
- Understanding DI pattern
- Constructor-based injection
- Type-based dependency resolution
- IoC (Inversion of Control) container
- Singleton pattern by default

### 4.4 Provider Scopes
- Application-scoped providers (default)
- Request-scoped providers
- Transient providers
- Lifecycle management
- Injection scopes chapter reference

### 4.5 Custom Providers
- Plain value providers
- Class providers
- Factory providers (sync/async)
- Custom tokens
- useValue, useClass, useFactory

### 4.6 Optional Providers
- `@Optional()` decorator
- Handling missing dependencies
- Default values
- Configuration use cases

### 4.7 Property-based Injection
- Alternative to constructor injection
- Using `@Inject()` decorator
- Property-level injection
- When to use property injection

### 4.8 Provider Registration
- Registering providers in modules
- Providers array in `@Module()`
- Making providers available
- Module dependency resolution

### 4.9 Manual Instantiation
- Retrieving existing instances
- Dynamic provider instantiation
- Module reference usage
- Standalone applications
- Bootstrap function providers

---

## Module 5: Modules

📚 **[View Module 5: Modules](module-05-modules.md)**

### 5.1 Introduction to Modules
- What are modules?
- Organizing application structure
- `@Module()` decorator
- Module metadata

### 5.2 Feature Modules
- Creating feature modules
- Grouping related functionality
- Encapsulation
- Module CLI: `nest g module [name]`

### 5.3 Shared Modules
- Exporting providers
- Importing modules
- Module reusability
- Singleton providers across modules

### 5.4 Module Re-exporting
- Re-exporting imported modules
- Creating module facades
- Simplifying imports

### 5.5 Global Modules
- `@Global()` decorator
- Available everywhere
- Use cases and best practices

### 5.6 Dynamic Modules
- Runtime module configuration
- Configurable providers
- `forRoot()` and `forFeature()` patterns
- Returning DynamicModule

---

## Module 6: Core Fundamentals

📚 **[View Module 6: Core Fundamentals](module-06-core-fundamentals.md)**

### 6.1 Application Bootstrap
- NestFactory class
- Creating application instance
- `INestApplication` interface
- Application lifecycle
- Listening for HTTP requests

### 6.2 Platform Selection
- Express vs Fastify
- Platform-specific APIs
- `NestExpressApplication`
- `NestFastifyApplication`
- Adapter pattern

### 6.3 Exception Handling
- Built-in exception filters
- HTTP exceptions
- Standard HTTP error responses
- Custom exceptions
- Exception filters chapter

### 6.4 State Management
- Shared resources across requests
- Singleton pattern safety
- Request/response model
- Node.js single-threaded architecture
- Edge cases for request-based lifetimes

### 6.5 Development Best Practices
- SOLID principles
- Code organization
- Modular architecture
- Testing strategies
- Documentation

### 6.6 Testing Fundamentals
- Unit testing controllers
- Testing services
- Mocking dependencies
- Test file structure
- Jest integration

---

## Module 7: Additional Fundamentals

📚 **[View Module 7: Additional Fundamentals](module-07-additional-fundamentals.md)**

### 7.1 Middleware
- Request/response pipeline
- Middleware functions
- Class-based middleware
- Functional middleware
- Applying middleware

### 7.2 Pipes
- Data transformation
- Validation
- Built-in pipes
- Custom pipes
- ValidationPipe

### 7.3 Guards
- Authorization and authentication
- Route protection
- Execution context
- Return values
- Guard order

### 7.4 Interceptors
- Request/response interception
- Response transformation
- Side effects
- Aspect-oriented programming
- RxJS operators

### 7.5 Custom Decorators
- Creating custom decorators
- Parameter decorators
- Combining decorators
- Metadata reflection
- Use cases

---

## Module 8: Practical Application

📚 **[View Module 8: Practical Application](module-08-practical-application.md)**

### 8.1 Building a CRUD Application
- Project setup
- Creating resources
- Implementing CRUD operations
- DTOs and interfaces
- Service layer logic

### 8.2 Project Structure Best Practices
- Feature-based organization
- Module directory structure
- File naming conventions
- Import organization

### 8.3 Development Workflow
- Hot reload with watch mode
- Debugging applications
- Environment configuration
- Logging strategies

### 8.4 CLI Usage
- Common CLI commands
- Generating resources
- Schematic options
- Workspace management

---

## Learning Objectives

By the end of this section, students will be able to:

1. ✅ Understand NestJS philosophy and architecture
2. ✅ Set up and configure NestJS projects
3. ✅ Create and manage controllers with various routing patterns
4. ✅ Implement providers and services with dependency injection
5. ✅ Organize code using modules
6. ✅ Handle requests, responses, and errors effectively
7. ✅ Apply best practices for scalable applications
8. ✅ Use NestJS CLI for rapid development
9. ✅ Implement basic CRUD operations
10. ✅ Understand the request/response lifecycle

---

## Hands-on Labs

### Lab 1: Hello NestJS
- Install NestJS CLI
- Create first project
- Run and test the application

### Lab 2: Building a Cats API
- Create controllers and routes
- Implement services
- Add DTOs and interfaces
- Test with HTTP client

### Lab 3: Module Organization
- Create feature modules
- Implement dependency injection
- Share providers across modules

### Lab 4: Advanced Routing
- Dynamic routes with parameters
- Query parameter handling
- Request payload validation

### Lab 5: Complete CRUD Application
- Build full resource
- Implement all HTTP methods
- Add error handling
- Test the complete API

---

## Resources

- **Official Documentation**: https://docs.nestjs.com/
- **GitHub Repository**: https://github.com/nestjs/nest
- **Discord Community**: https://discord.gg/G7Qnnhy
- **Stack Overflow**: Tag: `nestjs`
- **Official Courses**: https://courses.nestjs.com/

---

## Assessment

- Quiz on NestJS fundamentals
- Practical coding exercises
- Mini-project: Build a RESTful API
- Code review and best practices evaluation

---

## Next Steps

After completing this section, students should be prepared to:
- Explore advanced NestJS features
- Learn about database integration
- Implement authentication and authorization
- Deploy NestJS applications
- Work with WebSockets and microservices
