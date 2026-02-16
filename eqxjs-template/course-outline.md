# EQXJS Template Training Course

## Enterprise NestJS Microservices Development with Event-Driven Architecture

---

## 🎯 Course Overview

This comprehensive training course teaches developers how to build enterprise-grade microservices using the **EQXJS Template** - a battle-tested NestJS template that combines REST API and Event-Driven Architecture with Kafka messaging.

### What You'll Learn

- Enterprise microservices architecture patterns
- Event-driven development with Kafka
- EQXJS framework library usage
- Database integration with MongoDB
- REST API and event consumer/producer implementation
- Testing strategies for microservices
- Production deployment best practices

### Prerequisites

- Node.js (>= 18.x)
- TypeScript fundamentals
- Basic NestJS knowledge
- Understanding of REST APIs
- Basic Kafka concepts (helpful but not required)

---

## 📖 Course Modules

| Module   | Topic                                    | Duration | Content                                             | Exercises                                    |
| -------- | ---------------------------------------- | -------- | --------------------------------------------------- | -------------------------------------------- |
| Module 1 | Introduction to EQXJS Framework          | 2 hours  | [View Content](module-01-introduction.md)           | [Exercises](exercise/module-01-exercises.md) |
| Module 2 | Getting Started with EQXJS Template      | 2 hours  | [View Content](module-02-getting-started.md)        | [Exercises](exercise/module-02-exercises.md) |
| Module 3 | Template Architecture Deep Dive          | 3 hours  | [View Content](module-03-architecture.md)           | [Exercises](exercise/module-03-exercises.md) |
| Module 4 | Controllers and Managers                 | 3 hours  | [View Content](module-04-controllers-managers.md)   | [Exercises](exercise/module-04-exercises.md) |
| Module 5 | Services and Repositories                | 3 hours  | [View Content](module-05-services-repositories.md)  | [Exercises](exercise/module-05-exercises.md) |
| Module 6 | Event-Driven Architecture with Kafka     | 4 hours  | [View Content](module-06-kafka-events.md)           | [Exercises](exercise/module-06-exercises.md) |
| Module 7 | External Services & Database Integration | 3 hours  | [View Content](module-07-integrations.md)           | [Exercises](exercise/module-07-exercises.md) |
| Module 8 | Testing and Best Practices               | 3 hours  | [View Content](module-08-testing-best-practices.md) | [Exercises](exercise/module-08-exercises.md) |

### 💡 [Complete Exercise Guide](exercise/README.md)

Access comprehensive hands-on exercises for all modules with solutions.

**Total Duration:** ~23 hours

---

## Module 1: Introduction to EQXJS Framework

📚 **[View Module 1: Introduction to EQXJS Framework](module-01-introduction.md)**

### 1.1 What is EQXJS Framework?

- Overview of EQXJS ecosystem
- Framework philosophy and design principles
- Core modules and libraries
- Template-based development approach

### 1.2 EQXJS Components

- `@eqxjs/stub` - Main framework module
- `@eqxjs/custom-kafka-server` - Kafka integration
- `@eqxjs/azure-manage-identity` - Azure integration
- Companion libraries and utilities

### 1.3 Framework Features

- Graceful shutdown handling
- Advanced logging with context
- Message context management
- HTTP and REST interceptors
- Custom decorators and validators
- Exception handling framework

### 1.4 When to Use EQXJS Template

- Enterprise microservices
- Event-driven applications
- Kafka-based messaging systems
- REST + Event hybrid architecture
- Regulated industries requiring audit trails

---

## Module 2: Getting Started with EQXJS Template

📚 **[View Module 2: Getting Started](module-02-getting-started.md)**

### 2.1 Environment Setup

- Prerequisites installation
- Node.js and npm/yarn setup
- Docker for local development
- IDE configuration (VS Code)
- Required global packages

### 2.2 Template Installation

- Cloning the template repository
- Installing dependencies
- Understanding project structure
- Configuration files overview
- Environment-specific configs

### 2.3 Project Structure

```
cronus-eqxjs-template/
├── assets/config/          # Environment configurations
├── scripts/                # Helper scripts (Docker, etc.)
├── src/
│   ├── app.module.ts      # Root module
│   ├── main.ts            # Application bootstrap
│   ├── database/          # Database module & utilities
│   └── example/           # Example implementation
│       ├── consumer/      # Event consumers
│       ├── producer/      # Event producers
│       ├── manager/       # Business logic orchestration
│       ├── service/       # Business services
│       ├── repositories/  # Data access layer
│       ├── external-services/ # API integrations
│       └── dtos/          # Data transfer objects
└── package.json
```

### 2.4 Running the Application

- Local development with hot-reload
- Starting Kafka with Docker Compose
- Starting MongoDB with Docker Compose
- Environment variable configuration
- Testing the example endpoints

### 2.5 Configuration Management

- Environment-based config files
- YAML configuration structure
- Zone-based configuration (local, dev, staging, prod)
- Secure configuration practices

---

## Module 3: Template Architecture Deep Dive

📚 **[View Module 3: Template Architecture](module-03-architecture.md)**

### 3.1 Architecture Overview

- Layered architecture pattern
- Separation of concerns
- Dependency injection strategy
- Module organization

### 3.2 Request Flow

- REST API request flow
- Kafka event consumption flow
- Event production flow
- Error handling flow

### 3.3 Core Patterns

- Manager pattern for orchestration
- Repository pattern for data access
- Service pattern for business logic
- DTO pattern for data validation

### 3.4 Framework Integration

- FrameworkModule registration
- Graceful shutdown setup
- Message context flow
- Logging infrastructure

---

## Module 4: Controllers and Managers

📚 **[View Module 4: Controllers and Managers](module-04-controllers-managers.md)**

### 4.1 Event Consumer Controllers

- `@EntryPoint()` decorator
- Kafka message handling
- Context extraction
- Error handling in consumers
- Consumer interceptors

### 4.2 REST Controllers

- REST endpoint implementation
- Request/response handling
- Validation with DTOs
- HTTP interceptors

### 4.3 Manager Layer

- Manager responsibilities
- Orchestrating multiple services
- Transaction management
- Manager vs Service distinction

### 4.4 Decorators and Pipes

- `@ToObjectDecorator()`
- `RemoveAtSymbolPipe`
- Custom decorators
- Data transformation

---

## Module 5: Services and Repositories

📚 **[View Module 5: Services and Repositories](module-05-services-repositories.md)**

### 5.1 Service Layer Design

- Business logic implementation
- Service dependencies
- Message context service usage
- Configuration service integration

### 5.2 Repository Pattern

- Repository interfaces
- MongoDB repository implementation
- Data access abstraction
- Query building and filtering

### 5.3 Database Integration

- MongoDB connection setup
- Database module configuration
- Collection management
- Connection pooling

### 5.4 Data Models

- Interface definitions
- Schema design
- Type safety with TypeScript
- Data validation

---

## Module 6: Event-Driven Architecture with Kafka

📚 **[View Module 6: Event-Driven Architecture](module-06-kafka-events.md)**

### 6.1 Kafka Integration

- CustomServerKafka setup
- Producer configuration
- Consumer group setup
- Topic management

### 6.2 Event Consumers

- Subscribing to topics
- Message deserialization
- Consumer patterns
- Error handling and retry

### 6.3 Event Producers

- Publishing events
- Message serialization
- Producer patterns
- Delivery guarantees

### 6.4 Message Context

- Message correlation
- Context propagation
- Tracing and debugging
- Logging with context

### 6.5 Advanced Patterns

- Multiple producers
- Batch processing
- Dead letter queues
- Event versioning

---

## Module 7: External Services & Database Integration

📚 **[View Module 7: External Services & Database](module-07-integrations.md)**

### 7.1 External API Integration

- HTTP client setup with Axios
- Service client implementation
- Error handling
- Retry strategies

### 7.2 API Service Pattern

- Structured API services
- Request/response typing
- Authentication handling
- Rate limiting

### 7.3 Database Operations

- CRUD operations
- Query optimization
- Indexing strategies
- Transaction management

### 7.4 Integration Testing

- Mocking external services
- Database test utilities
- Integration test patterns

---

## Module 8: Testing and Best Practices

📚 **[View Module 8: Testing and Best Practices](module-08-testing-best-practices.md)**

### 8.1 Unit Testing

- Jest configuration
- Service unit tests
- Repository unit tests
- Mocking dependencies

### 8.2 Integration Testing

- End-to-end testing
- Database testing
- Kafka testing
- API testing

### 8.3 Code Quality

- ESLint configuration
- Prettier formatting
- Code review guidelines
- Documentation standards

### 8.4 Production Best Practices

- Error handling strategies
- Logging best practices
- Performance optimization
- Security considerations
- Monitoring and observability

### 8.5 Deployment

- Docker containerization
- Environment configuration
- CI/CD pipelines
- Health checks
- Graceful shutdown

---

## 🛠️ Practical Projects

Throughout the course, you'll build:

1. **User Management Service** - REST API with database integration
2. **Order Processing Service** - Event-driven order processing with Kafka
3. **Notification Service** - Multi-channel notification system
4. **Payment Gateway Integration** - External service integration project

---

## 📚 Additional Resources

- [EQXJS Framework Documentation](../eqxjs-framework/README.md)
- [NestJS Official Documentation](https://docs.nestjs.com)
- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

## 🎓 Certification

Upon completion of all modules and exercises, participants will receive:

- EQXJS Framework Developer Certificate
- Code review from senior developers
- Portfolio project showcase

---

## 📞 Support

- Technical questions: Use GitHub Discussions
- Bug reports: Create GitHub Issues
- Direct support: Contact training team

---

**Last Updated:** February 2026  
**Version:** 1.0.0  
**Maintained by:** EQXJS Team
