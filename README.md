# Learning Hub - Backend Technologies

Welcome to the comprehensive learning hub for modern backend technologies! This repository contains complete courses on **NestJS**, **Apache Kafka**, **EQXJS Framework**, and the **EQXJS Template**.

---

## 📚 Available Courses

| Course                                          | Description                           | Modules    | Status      |
| ----------------------------------------------- | ------------------------------------- | ---------- | ----------- |
| **[NestJS](#-nestjs-fundamentals-course)**      | Progressive Node.js framework         | 8 modules  | ✅ Complete |
| **[Apache Kafka](#-apache-kafka-course)**       | Distributed event streaming platform  | 9 modules  | ✅ Complete |
| **[EQXJS Framework](#-eqxjs-framework-course)** | Enterprise NestJS framework ecosystem | 16 modules | ✅ Complete |
| **[EQXJS Template](#-eqxjs-template-course)**   | REST + Kafka domain service template  | 8 modules  | ✅ Complete |

---

## 🎯 NestJS Fundamentals Course

> Build scalable and maintainable Node.js applications with TypeScript

### 📖 Quick Access

**[📋 Complete NestJS Course](nestjs/README.md)** | **[📚 Course Outline](nestjs/course-outline.md)** | **[💡 All Exercises](nestjs/exercise/README.md)**

### Course Modules

| Module       | Topic                                                                  | Content                                             | Exercises                                           |
| ------------ | ---------------------------------------------------------------------- | --------------------------------------------------- | --------------------------------------------------- |
| **Module 1** | [Introduction to NestJS](nestjs/module-01-introduction.md)             | [View](nestjs/module-01-introduction.md)            | [Exercises](nestjs/exercise/module-01-exercises.md) |
| **Module 2** | [Getting Started](nestjs/module-02-getting-started.md)                 | [View](nestjs/module-02-getting-started.md)         | [Exercises](nestjs/exercise/module-02-exercises.md) |
| **Module 3** | [Controllers](nestjs/module-03-controllers.md)                         | [View](nestjs/module-03-controllers.md)             | [Exercises](nestjs/exercise/module-03-exercises.md) |
| **Module 4** | [Providers](nestjs/module-04-providers.md)                             | [View](nestjs/module-04-providers.md)               | [Exercises](nestjs/exercise/module-04-exercises.md) |
| **Module 5** | [Modules](nestjs/module-05-modules.md)                                 | [View](nestjs/module-05-modules.md)                 | [Exercises](nestjs/exercise/module-05-exercises.md) |
| **Module 6** | [Core Fundamentals](nestjs/module-06-core-fundamentals.md)             | [View](nestjs/module-06-core-fundamentals.md)       | [Exercises](nestjs/exercise/module-06-exercises.md) |
| **Module 7** | [Additional Fundamentals](nestjs/module-07-additional-fundamentals.md) | [View](nestjs/module-07-additional-fundamentals.md) | [Exercises](nestjs/exercise/module-07-exercises.md) |
| **Module 8** | [Practical Application](nestjs/module-08-practical-application.md)     | [View](nestjs/module-08-practical-application.md)   | [Exercises](nestjs/exercise/module-08-exercises.md) |

### 📋 What You'll Learn

- ✅ NestJS architecture and dependency injection
- ✅ Building RESTful APIs with TypeScript
- ✅ Controllers, Providers, and Modules
- ✅ Middleware, Pipes, Guards, and Interceptors
- ✅ Exception handling and validation
- ✅ Testing (Unit & E2E)
- ✅ SOLID principles and best practices
- ✅ Production-ready application development

### 🚀 Quick Start

```bash
# Install NestJS CLI
npm i -g @nestjs/cli

# Create a new project
nest new my-project

# Start development
cd my-project
npm run start:dev
```

**[👉 Start NestJS Course](nestjs/README.md)**

---

## 🎯 Apache Kafka Course

> Master distributed event streaming and real-time data processing

### 📖 Quick Access

**[📋 Complete Kafka Course](kafka/README.md)** | **[📚 Course Outline](kafka/course-outline.md)** | **[💡 All Exercises](kafka/exercise/README.md)**

### Course Modules

| Module       | Topic                                                                               | Content                                    | Exercises                                          |
| ------------ | ----------------------------------------------------------------------------------- | ------------------------------------------ | -------------------------------------------------- |
| **Module 1** | [Introduction to Apache Kafka](kafka/module-01-introduction.md)                     | [View](kafka/module-01-introduction.md)    | [Exercises](kafka/exercise/module-01-exercises.md) |
| **Module 2** | [Kafka Architecture and Core Concepts](kafka/module-02-architecture.md)             | [View](kafka/module-02-architecture.md)    | [Exercises](kafka/exercise/module-02-exercises.md) |
| **Module 3** | [Setting Up Kafka](kafka/module-03-setup.md)                                        | [View](kafka/module-03-setup.md)           | [Exercises](kafka/exercise/module-03-exercises.md) |
| **Module 4** | [Kafka Producers](kafka/module-04-producers.md)                                     | [View](kafka/module-04-producers.md)       | [Exercises](kafka/exercise/module-04-exercises.md) |
| **Module 5** | [Kafka Consumers](kafka/module-05-consumers.md)                                     | [View](kafka/module-05-consumers.md)       | [Exercises](kafka/exercise/module-05-exercises.md) |
| **Module 6** | [Topics, Partitions, and Data Management](kafka/module-06-data-management.md)       | [View](kafka/module-06-data-management.md) | [Exercises](kafka/exercise/module-06-exercises.md) |
| **Module 7** | [Kafka Connect and Kafka Streams](kafka/module-07-connect-streams.md)               | [View](kafka/module-07-connect-streams.md) | [Exercises](kafka/exercise/module-07-exercises.md) |
| **Module 8** | [Advanced Topics and Best Practices](kafka/module-08-advanced.md)                   | [View](kafka/module-08-advanced.md)        | [Exercises](kafka/exercise/module-08-exercises.md) |
| **Module 9** | [Practical Application - Building a Real-Time System](kafka/module-09-practical.md) | [View](kafka/module-09-practical.md)       | [Exercises](kafka/exercise/module-09-exercises.md) |

### 📋 What You'll Learn

- ✅ Event streaming and event-driven architecture
- ✅ Kafka architecture (brokers, topics, partitions)
- ✅ Building reliable producers and consumers
- ✅ Data management and replication
- ✅ Kafka Connect and Kafka Streams
- ✅ Security and monitoring
- ✅ Performance optimization
- ✅ Real-time data processing systems

### 🚀 Quick Start

```bash
# Using Docker
docker-compose up -d

# Create a topic
kafka-topics.sh --create \
  --topic my-topic \
  --bootstrap-server localhost:9092

# Produce messages
echo "Hello, Kafka!" | kafka-console-producer.sh \
  --topic my-topic \
  --bootstrap-server localhost:9092

# Consume messages
kafka-console-consumer.sh \
  --topic my-topic \
  --from-beginning \
  --bootstrap-server localhost:9092
```

**[👉 Start Kafka Course](kafka/README.md)**

---

## 🎯 EQXJS Framework Course

> Master enterprise-grade NestJS application development with the EQXJS ecosystem

### 📖 Available Courses

| Course                                                                                    | Description                                                                                     | Modules | Quick Links                                                                                                                                                                       |
| ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **[EQXJS Stub Framework](eqxjs-framework/README.md)**                                     | Complete enterprise NestJS framework with decorators, interceptors, health monitoring, and more | 10      | [📚 Outline](eqxjs-framework/01-stub/course-outline.md) \| [⚡ Quick Start](eqxjs-framework/01-stub/QUICK_START.md) \| [💡 Exercises](eqxjs-framework/01-stub/exercise/README.md) |
| **[EQXJS Custom Kafka Server](eqxjs-framework/02-custom-kafka-server/course-outline.md)** | Kafka integration, consumer/producer management, resilience patterns                            | 6       | [📚 Outline](eqxjs-framework/02-custom-kafka-server/course-outline.md) \| [💡 Exercises](eqxjs-framework/02-custom-kafka-server/exercise/README.md)                               |
| **[EQXJS Swagger Codegen](eqxjs-framework/03-swagger-codegen/course-outline.md)**         | Swagger/OpenAPI contract-first code generation for NestJS server + TypeScript client SDKs      | 8       | [📚 Outline](eqxjs-framework/03-swagger-codegen/course-outline.md) \| [📖 Course](eqxjs-framework/03-swagger-codegen/README.md)                                                |

---

### 📚 Sub-Course 1: EQXJS Stub Framework (10 Modules)

| Module        | Topic                                                                                          | Content                                                               | Exercises                                                            |
| ------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **Module 1**  | [Introduction to EQXJS Framework](eqxjs-framework/01-stub/module-01-introduction.md)           | [View](eqxjs-framework/01-stub/module-01-introduction.md)             | [Exercises](eqxjs-framework/01-stub/exercise/module-01-exercises.md) |
| **Module 2**  | [Getting Started & Setup](eqxjs-framework/01-stub/module-02-getting-started.md)                | [View](eqxjs-framework/01-stub/module-02-getting-started.md)          | [Exercises](eqxjs-framework/01-stub/exercise/module-02-exercises.md) |
| **Module 3**  | [Framework Module Configuration](eqxjs-framework/01-stub/module-03-framework-configuration.md) | [View](eqxjs-framework/01-stub/module-03-framework-configuration.md)  | [Exercises](eqxjs-framework/01-stub/exercise/module-03-exercises.md) |
| **Module 4**  | [Health Checks & Monitoring](eqxjs-framework/01-stub/module-04-health-monitoring.md)           | [View](eqxjs-framework/01-stub/module-04-health-monitoring.md)        | [Exercises](eqxjs-framework/01-stub/exercise/module-04-exercises.md) |
| **Module 5**  | [Interceptors & HTTP Handling](eqxjs-framework/01-stub/module-05-interceptors-http.md)         | [View](eqxjs-framework/01-stub/module-05-interceptors-http.md)        | [Exercises](eqxjs-framework/01-stub/exercise/module-05-exercises.md) |
| **Module 6**  | [Context Management & Domain Services](eqxjs-framework/01-stub/module-06-context-domain.md)    | [View](eqxjs-framework/01-stub/module-06-context-domain.md)           | [Exercises](eqxjs-framework/01-stub/exercise/module-06-exercises.md) |
| **Module 7**  | [Decorators & Validation](eqxjs-framework/01-stub/module-07-decorators-validation.md)          | [View](eqxjs-framework/01-stub/module-07-decorators-validation.md)    | [Exercises](eqxjs-framework/01-stub/exercise/module-07-exercises.md) |
| **Module 8**  | [Graceful Shutdown & Production](eqxjs-framework/01-stub/module-08-shutdown-production.md)     | [View](eqxjs-framework/01-stub/module-08-shutdown-production.md)      | Coming Soon                                                          |
| **Module 9**  | [Practical Implementation](eqxjs-framework/01-stub/module-09-practical-implementation.md)      | [View](eqxjs-framework/01-stub/module-09-practical-implementation.md) | Coming Soon                                                          |
| **Module 10** | [Advanced Patterns & Integration](eqxjs-framework/01-stub/module-10-advanced-patterns.md)      | [View](eqxjs-framework/01-stub/module-10-advanced-patterns.md)        | Coming Soon                                                          |

---

### 📚 Sub-Course 2: EQXJS Custom Kafka Server (6 Modules)

| Module       | Topic                                                                                                       | Content                                                                         | Exercises                                                                           |
| ------------ | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Module 1** | [Introduction & Features](eqxjs-framework/02-custom-kafka-server/module-01-introduction.md)                 | [View](eqxjs-framework/02-custom-kafka-server/module-01-introduction.md)        | [Exercises](eqxjs-framework/02-custom-kafka-server/exercise/module-01-exercises.md) |
| **Module 2** | [Setup & Integration](eqxjs-framework/02-custom-kafka-server/module-02-setup-integration.md)                | [View](eqxjs-framework/02-custom-kafka-server/module-02-setup-integration.md)   | [Exercises](eqxjs-framework/02-custom-kafka-server/exercise/module-02-exercises.md) |
| **Module 3** | [Architecture & Design](eqxjs-framework/02-custom-kafka-server/module-03-architecture.md)                   | [View](eqxjs-framework/02-custom-kafka-server/module-03-architecture.md)        | [Exercises](eqxjs-framework/02-custom-kafka-server/exercise/module-03-exercises.md) |
| **Module 4** | [Resilience & Recovery](eqxjs-framework/02-custom-kafka-server/module-04-resilience-recovery.md)            | [View](eqxjs-framework/02-custom-kafka-server/module-04-resilience-recovery.md) | [Exercises](eqxjs-framework/02-custom-kafka-server/exercise/module-04-exercises.md) |
| **Module 5** | [Topic Monitoring](eqxjs-framework/02-custom-kafka-server/module-05-topic-monitoring.md)                    | [View](eqxjs-framework/02-custom-kafka-server/module-05-topic-monitoring.md)    | [Exercises](eqxjs-framework/02-custom-kafka-server/exercise/module-05-exercises.md) |
| **Module 6** | [Memory & Production Best Practices](eqxjs-framework/02-custom-kafka-server/module-06-memory-production.md) | [View](eqxjs-framework/02-custom-kafka-server/module-06-memory-production.md)   | [Exercises](eqxjs-framework/02-custom-kafka-server/exercise/module-06-exercises.md) |

---

### 📚 Sub-Course 3: EQXJS Swagger Codegen (8 Modules)

| Module       | Topic                                                                                                               | Content                                                                             | Duration   |
| ------------ | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------- |
| **Module 1** | [Introduction to Swagger Code Generation](eqxjs-framework/03-swagger-codegen/module-01-introduction.md)            | [View](eqxjs-framework/03-swagger-codegen/module-01-introduction.md)               | 45-60 min  |
| **Module 2** | [Installation and CLI Setup](eqxjs-framework/03-swagger-codegen/module-02-installation-cli-setup.md)               | [View](eqxjs-framework/03-swagger-codegen/module-02-installation-cli-setup.md)     | 45-75 min  |
| **Module 3** | [Working with Swagger 2.0 and OpenAPI 3.0](eqxjs-framework/03-swagger-codegen/module-03-swagger-openapi-formats.md) | [View](eqxjs-framework/03-swagger-codegen/module-03-swagger-openapi-formats.md)    | 60-90 min  |
| **Module 4** | [Server Code Generation (NestJS)](eqxjs-framework/03-swagger-codegen/module-04-server-code-generation.md)          | [View](eqxjs-framework/03-swagger-codegen/module-04-server-code-generation.md)     | 75-105 min |
| **Module 5** | [Client SDK Generation (TypeScript + Axios)](eqxjs-framework/03-swagger-codegen/module-05-client-sdk-generation.md) | [View](eqxjs-framework/03-swagger-codegen/module-05-client-sdk-generation.md)      | 60-90 min  |
| **Module 6** | [Validation and Testing Flags](eqxjs-framework/03-swagger-codegen/module-06-validation-testing.md)                 | [View](eqxjs-framework/03-swagger-codegen/module-06-validation-testing.md)         | 60-90 min  |
| **Module 7** | [DTO-Only and Shared Contract Patterns](eqxjs-framework/03-swagger-codegen/module-07-dto-only-contracts.md)        | [View](eqxjs-framework/03-swagger-codegen/module-07-dto-only-contracts.md)         | 45-75 min  |
| **Module 8** | [Production Workflow and CI/CD Integration](eqxjs-framework/03-swagger-codegen/module-08-production-cicd.md)       | [View](eqxjs-framework/03-swagger-codegen/module-08-production-cicd.md)            | 60-90 min  |

---

### 🎓 Learning Outcomes

**Stub Framework Learning Path:**

### 📋 What You'll Learn

**EQXJS Stub Framework:**

- ✅ EQXJS framework architecture and 8+ core modules
- ✅ Enterprise configuration management (YAML-based)
- ✅ Custom decorators and metadata-driven development
- ✅ Advanced interceptor patterns and data masking
- ✅ Production-ready health monitoring systems
- ✅ JWT authentication and role-based authorization
- ✅ Graceful shutdown and lifecycle management
- ✅ Data processing pipes and validation patterns
- ✅ Structured logging with privacy compliance
- ✅ HTTP transport and service communication

**EQXJS Custom Kafka Server:**

- ✅ Kafka consumer/producer integration with NestJS
- ✅ Consumer crash recovery and resilience patterns
- ✅ Topic validation and monitoring strategies
- ✅ Memory management with pause/resume mechanisms
- ✅ Production-ready Kafka deployment patterns
- ✅ Event-driven architecture best practices

**EQXJS Swagger Codegen:**

- ✅ Contract-first API development with Swagger/OpenAPI
- ✅ NestJS server scaffolding generation (DTOs, controllers, services, modules)
- ✅ TypeScript Axios client SDK generation
- ✅ DTO validation decorators and test scaffold generation
- ✅ DTO-only contract sharing strategies for monorepos and multi-service systems
- ✅ CI/CD patterns for deterministic regeneration workflows

### 🚀 Quick Start

**Stub Framework:**

```bash
# Install EQXJS Framework
npm install @corp-ais/eqxjs-stub

# Create configuration
mkdir config
touch config/development.config.yaml

# Add to your NestJS app
import { FrameworkModule } from '@corp-ais/eqxjs-stub';
```

**Custom Kafka Server:**

```bash
# Install Custom Kafka Server
npm install @corp-ais/eqxjs-custom-kafka-server

# Configure Kafka transport
import { CustomServerKafka } from '@corp-ais/eqxjs-custom-kafka-server';
```

**Swagger Codegen:**

```bash
# Generate NestJS server scaffolding from OpenAPI spec (tests & validation included by default)
npx @eqxjs/swagger-codegen generate -i ./openapi.yaml -o ./generated

# Generate both server + client
npx @eqxjs/swagger-codegen generate -i ./openapi.yaml -o ./generated --mode both

# Generate only DTOs for shared contracts
npx @eqxjs/swagger-codegen generate -i ./openapi.yaml -o ./types --mode dtos
```

**[👉 Start EQXJS Framework Course](eqxjs-framework/README.md)**

---

## 🎯 EQXJS Template Course

> Build a NestJS domain service that handles REST APIs and Kafka events using EQXJS.

### 📖 Quick Access

**[📋 Complete Course](eqxjs-template/README.md)** | **[📚 Course Outline](eqxjs-template/course-outline.md)** | **[💡 All Exercises](eqxjs-template/exercise/README.md)**

### Course Modules

| Module       | Topic                                    | Content                                                    | Exercises                                       |
| ------------ | ---------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------- |
| **Module 1** | Introduction to EQXJS Framework          | [View](eqxjs-template/module-01-introduction.md)           | [Exercises](eqxjs-template/exercise/README.md) |
| **Module 2** | Getting Started with EQXJS Template      | [View](eqxjs-template/module-02-getting-started.md)        | [Exercises](eqxjs-template/exercise/README.md) |
| **Module 3** | Template Architecture Deep Dive          | [View](eqxjs-template/module-03-architecture.md)           | [Exercises](eqxjs-template/exercise/README.md) |
| **Module 4** | Controllers and Managers                 | [View](eqxjs-template/module-04-controllers-managers.md)   | [Exercises](eqxjs-template/exercise/README.md) |
| **Module 5** | Services and Repositories                | [View](eqxjs-template/module-05-services-repositories.md)  | [Exercises](eqxjs-template/exercise/README.md) |
| **Module 6** | Event-Driven Architecture with Kafka     | [View](eqxjs-template/module-06-kafka-events.md)           | [Exercises](eqxjs-template/exercise/README.md) |
| **Module 7** | External Services & Database Integration | [View](eqxjs-template/module-07-integrations.md)           | [Exercises](eqxjs-template/exercise/README.md) |
| **Module 8** | Testing and Best Practices               | [View](eqxjs-template/module-08-testing-best-practices.md) | [Exercises](eqxjs-template/exercise/README.md) |

### 📋 What You'll Learn

- ✅ Enterprise microservices architecture patterns
- ✅ Event-driven development with Apache Kafka
- ✅ EQXJS framework library usage and best practices
- ✅ MongoDB integration and database patterns
- ✅ REST API and event consumer/producer implementation
- ✅ Layered architecture (Controllers, Managers, Services, Repositories)
- ✅ Testing strategies for microservices (Unit & Integration)
- ✅ Production deployment and monitoring
- ✅ Code quality and best practices

### 🚀 Quick Start

```bash
npm install
export ZONE=local
export BROKERS=localhost:9092
npm run start:local
```

**[👉 Start EQXJS Template Course](eqxjs-template/README.md)**

---

## 🎓 Learning Paths

### Path 1: Backend Developer Track

Build modern microservices and event-driven applications:

1. **[NestJS Fundamentals](nestjs/README.md)** - Build RESTful APIs
2. **[EQXJS Framework](eqxjs-framework/README.md)** - Enterprise patterns and utilities
3. **[Apache Kafka](kafka/README.md)** - Event streaming capabilities

### Path 2: Enterprise Developer Track

Focus on enterprise-grade application development:

1. **[NestJS Fundamentals](nestjs/README.md)** - Core framework concepts
2. **[EQXJS Framework](eqxjs-framework/README.md)** - Production-ready patterns

### Path 3: Data Engineer Track

Focus on data streaming and integration:

1. **[Apache Kafka](kafka/README.md)** - Master event streaming
2. **[NestJS Fundamentals](nestjs/README.md)** - Build data processing services

### Path 4: Full-Stack Journey

Comprehensive understanding of modern backend architectures with all courses.

---

## 🛠️ Technology Stack

### NestJS Course

- **Runtime**: Node.js 20+
- **Language**: TypeScript
- **Framework**: NestJS 10.x
- **Testing**: Jest
- **Tools**: NestJS CLI, ESLint, Prettier

### EQXJS Framework Course

- **Runtime**: Node.js 20+
- **Language**: TypeScript
- **Framework**: NestJS 11+, EQXJS 3.2.6+
- **Core Modules**: 8 integrated EQXJS modules
- **Tools**: TypeDoc, Joi Validation, JWT

### Kafka Course

- **Platform**: Apache Kafka 3.x
- **Languages**: Java, Python, Node.js (TypeScript)
- **Container**: Docker & Docker Compose
- **Tools**: Kafka CLI, Schema Registry, Kafka UI
- **Monitoring**: Prometheus, Grafana

### EQXJS Template Course

- **Runtime**: Node.js 18+
- **Language**: TypeScript
- **Framework**: NestJS 10.x, EQXJS Stub 2.4.0
- **Messaging**: Kafka (CustomServerKafka)
- **Database**: MongoDB

---

## 📊 Course Statistics

### Combined Learning

- **Total Modules**: 41 modules
- **Exercises**: 200+ hands-on exercises
- **Projects**: 4+ complete applications
- **Difficulty Levels**: Beginner to Advanced

### Prerequisites

- Node.js experience
- JavaScript/TypeScript basics
- Understanding of HTTP/REST
- Familiarity with command-line tools
- Basic distributed systems concepts

---

## 🎯 Learning Outcomes

After completing all courses, you will be able to:

### Application Development

- ✅ Build scalable RESTful APIs with NestJS
- ✅ Implement dependency injection and IoC patterns
- ✅ Design modular, maintainable applications
- ✅ Write comprehensive tests (Unit & E2E)

### Enterprise Patterns

- ✅ Master EQXJS framework ecosystem
- ✅ Implement custom decorators and interceptors
- ✅ Build production-ready health monitoring
- ✅ JWT authentication and authorization

### Event-Driven Architecture

- ✅ Design event-driven systems
- ✅ Implement reliable message producers and consumers
- ✅ Build real-time data pipelines
- ✅ Handle data streaming at scale

### Production Skills

- ✅ Deploy and monitor applications
- ✅ Implement security best practices
- ✅ Optimize performance and reliability
- ✅ Handle errors and exceptions gracefully

---

## 📁 Repository Structure

```
/opt/code/dos/cronus-course-document/
├── README.md                                 # This file - Main entry point
│
├── nestjs/                                   # NestJS Course
│   ├── README.md                             # NestJS course home
│   ├── course-outline.md                     # Detailed curriculum
│   ├── module-01-introduction.md             # Module 1: Introduction
│   ├── module-02-getting-started.md          # Module 2: Getting Started
│   ├── module-03-controllers.md              # Module 3: Controllers
│   ├── module-04-providers.md                # Module 4: Providers
│   ├── module-05-modules.md                  # Module 5: Modules
│   ├── module-06-core-fundamentals.md        # Module 6: Core Fundamentals
│   ├── module-07-additional-fundamentals.md  # Module 7: Advanced
│   ├── module-08-practical-application.md    # Module 8: Practical
│   └── exercise/                             # NestJS exercises
│       ├── README.md                         # Exercise index
│       └── module-*-exercises.md             # Module exercises
│
├── kafka/                                    # Apache Kafka Course
│   ├── README.md                             # Kafka course home
│   ├── course-outline.md                     # Detailed curriculum
│   ├── module-01-introduction.md             # Module 1: Introduction
│   ├── module-02-architecture.md             # Module 2: Architecture
│   ├── module-03-setup.md                    # Module 3: Setup
│   ├── module-04-producers.md                # Module 4: Producers
│   ├── module-05-consumers.md                # Module 5: Consumers
│   ├── module-06-data-management.md          # Module 6: Data Management
│   ├── module-07-connect-streams.md          # Module 7: Connect & Streams
│   ├── module-08-advanced.md                 # Module 8: Advanced Topics
│   ├── module-09-practical.md                # Module 9: Practical App
│   └── exercise/                             # Kafka exercises
│       ├── README.md                         # Exercise index
│       └── module-*-exercises.md             # Module exercises
│
├── eqxjs-framework/                          # EQXJS Framework Courses
│   ├── README.md                             # EQXJS courses hub
│   ├── 01-stub/                              # EQXJS Stub Framework
│   │   ├── README.md                         # Stub course home
│   │   ├── QUICK_START.md                    # Quick start guide
│   │   ├── course-outline.md                 # Detailed curriculum
│   │   ├── module-01-introduction.md         # Module 1
│   │   ├── module-02-getting-started.md      # Module 2
│   │   ├── module-03-framework-configuration.md # Module 3
│   │   ├── module-04-health-monitoring.md    # Module 4
│   │   ├── module-05-interceptors-http.md    # Module 5
│   │   ├── module-06-context-domain.md       # Module 6
│   │   ├── module-07-decorators-validation.md # Module 7
│   │   ├── module-08-shutdown-production.md  # Module 8
│   │   ├── module-09-practical-implementation.md # Module 9
│   │   ├── module-10-advanced-patterns.md    # Module 10
│   │   └── exercise/                         # EQXJS exercises
│   │       ├── README.md                     # Exercise index
│   │       └── module-*-exercises.md         # Module exercises
│   │
│   └── 02-custom-kafka-server/               # EQXJS Kafka Integration
│       ├── README.md                         # Kafka integration home
│       ├── course-outline.md                 # Detailed curriculum
│       ├── module-01-introduction.md         # Module 1
│       ├── module-02-setup-integration.md    # Module 2
│       ├── module-03-architecture.md         # Module 3
│       ├── module-04-resilience-recovery.md  # Module 4
│       ├── module-05-topic-monitoring.md     # Module 5
│       ├── module-06-memory-production.md    # Module 6
│       └── exercise/                         # Kafka integration exercises
│           ├── README.md                     # Exercise index
│           └── module-*-exercises.md         # Module exercises
│
├── eqxjs-template/                           # EQXJS Template Course
│   ├── README.md                             # Course home
│   ├── course-outline.md                     # Detailed curriculum
│   ├── module-01-introduction.md             # Module 1
│   ├── module-02-getting-started.md          # Module 2
│   ├── module-03-architecture.md             # Module 3
│   ├── module-04-controllers-managers.md     # Module 4
│   ├── module-05-services-repositories.md    # Module 5
│   ├── module-06-kafka-events.md             # Module 6
│   ├── module-07-integrations.md             # Module 7
│   ├── module-08-testing-best-practices.md   # Module 8
│   └── exercise/                             # Course exercises
│       ├── README.md                         # Exercise index
│       └── module-*-exercises.md             # Module exercises
```

---

## 🚀 Getting Started

### Step 1: Choose Your Course

- **New to backend?** Start with [NestJS](nestjs/README.md)
- **Enterprise development?** Try [EQXJS Framework](eqxjs-framework/README.md)
- **Building data pipelines?** Start with [Kafka](kafka/README.md)
- **Template-based development?** Try [EQXJS Template](eqxjs-template/README.md)
- **Want them all?** Follow a [Learning Path](#-learning-paths)

### Step 2: Set Up Your Environment

Install required tools for your chosen course:

**For NestJS:**

```bash
node --version  # Should be >= 20
npm i -g @nestjs/cli
```

**For EQXJS Framework:**

```bash
node --version  # Should be >= 20
npm install @corp-ais/eqxjs-stub
```

**For Kafka:**

```bash
docker --version
docker-compose --version
```

### Step 3: Start Learning

Navigate to your course and begin with Module 1:

- **NestJS** → Start at [nestjs/](nestjs/)
- **EQXJS Framework** → Start at [eqxjs-framework/01-stub/](eqxjs-framework/01-stub/)
- **Kafka** → Start at [kafka/](kafka/)
- **EQXJS Template** → Start at [eqxjs-template/](eqxjs-template/)

---

## 💡 Study Tips

### Effective Learning Strategies

1. **Follow Module Order**: Each module builds on previous concepts
2. **Complete All Exercises**: Hands-on practice is essential
3. **Build Projects**: Apply concepts in real applications
4. **Take Notes**: Document your learning journey
5. **Review Regularly**: Revisit complex topics
6. **Join Communities**: Ask questions and help others

### Consistent Practice

- **Regular Practice**: Dedicate consistent time daily
- **Projects**: Build applications to apply your learning
- **Regular Reviews**: Recap what you learned regularly
- **Set Goals**: Complete modules at your own pace

### Best Practices

- ✅ Code along with examples
- ✅ Experiment with variations
- ✅ Read official documentation
- ✅ Write tests for your code
- ✅ Review solutions carefully
- ✅ Debug errors independently

---

## 🔗 Additional Resources

### Documentation

- [NestJS Official Docs](https://docs.nestjs.com/)
- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Node.js Docs](https://nodejs.org/docs/)

### Community

- [NestJS Discord](https://discord.gg/G7Qnnhy)
- [Kafka Slack](https://kafka.apache.org/contact)
- [Stack Overflow](https://stackoverflow.com/)
- [Reddit r/NestJS](https://www.reddit.com/r/Nestjs/)

### Video Resources

- [NestJS Official Courses](https://courses.nestjs.com/)
- [Kafka Summit](https://www.kafka-summit.org/)
- [YouTube Tutorials](https://www.youtube.com/)

### Books

- "Building Microservices with Node.js" by Daniel Kapexhiu
- "Kafka: The Definitive Guide" by Neha Narkhede
- "Designing Data-Intensive Applications" by Martin Kleppmann

---

## 📊 Progress Tracking

### NestJS Course Progress

- [ ] Module 1: Introduction to NestJS
- [ ] Module 2: Getting Started
- [ ] Module 3: Controllers
- [ ] Module 4: Providers
- [ ] Module 5: Modules
- [ ] Module 6: Core Fundamentals
- [ ] Module 7: Additional Fundamentals
- [ ] Module 8: Practical Application

### EQXJS Framework Stub Course Progress

- [ ] Module 1: Introduction to EQXJS Framework
- [ ] Module 2: Getting Started & Setup
- [ ] Module 3: Framework Module Configuration
- [ ] Module 4: Health Checks & Monitoring
- [ ] Module 5: Interceptors & HTTP Handling
- [ ] Module 6: Context Management & Domain Services
- [ ] Module 7: Decorators & Validation
- [ ] Module 8: Graceful Shutdown & Production Best Practices
- [ ] Module 9: Practical Implementation
- [ ] Module 10: Advanced Patterns & Integration

### Custom Kafka Server Course Progress

- [ ] Module 1: Introduction & Features
- [ ] Module 2: Setup & NestJS Integration
- [ ] Module 3: CustomServerKafka Architecture
- [ ] Module 4: Resilience & Recovery
- [ ] Module 5: Topic Monitoring & Subscription
- [ ] Module 6: Memory Management & Production Best Practices

### Kafka Course Progress

- [ ] Module 1: Introduction to Apache Kafka
- [ ] Module 2: Kafka Architecture and Core Concepts
- [ ] Module 3: Setting Up Kafka
- [ ] Module 4: Kafka Producers
- [ ] Module 5: Kafka Consumers
- [ ] Module 6: Topics, Partitions, and Data Management
- [ ] Module 7: Kafka Connect and Kafka Streams
- [ ] Module 8: Advanced Topics and Best Practices
- [ ] Module 9: Practical Application - Building a Real-Time System

### EQXJS Template Course Progress

- [ ] Module 1: Introduction to EQXJS Framework
- [ ] Module 2: Getting Started with EQXJS Template
- [ ] Module 3: Template Architecture Deep Dive
- [ ] Module 4: Controllers and Managers
- [ ] Module 5: Services and Repositories
- [ ] Module 6: Event-Driven Architecture with Kafka
- [ ] Module 7: External Services & Database Integration
- [ ] Module 8: Testing and Best Practices

---

## 🎖️ Next Steps After Completion

### Career Development

- Build portfolio projects
- Contribute to open source
- Share knowledge through blogs/talks
- Join tech communities

### Advanced Topics

- **NestJS**: GraphQL, WebSockets, Microservices
- **EQXJS**: Advanced interceptors, Custom modules, Integration patterns
- **Kafka**: ksqlDB, Schema Registry, MQTT
- **Architecture**: CQRS, Event Sourcing, Saga Pattern
- **DevOps**: Kubernetes, CI/CD, Monitoring

---

## 📧 Support

### How to Get Help

1. Review the module content thoroughly
2. Check the exercises and solutions
3. Consult official documentation
4. Search community forums
5. Ask specific questions with context

### Contributing

Found an issue or have suggestions? Contributions are welcome:

- Report issues with clear descriptions
- Suggest improvements with examples
- Share additional resources
- Help others in the community

---

## 📜 License

This educational content is provided for learning purposes.

- **NestJS** is [MIT licensed](https://github.com/nestjs/nest/blob/master/LICENSE)
- **EQXJS Framework** is proprietary (version 3.2.6)
- **Apache Kafka** is [Apache 2.0 licensed](https://github.com/apache/kafka/blob/trunk/LICENSE)

---

## 🌟 Quick Links

### Start Learning

- **[📘 NestJS Course →](nestjs/README.md)**
- **[📙 EQXJS Framework Course →](eqxjs-framework/README.md)**
- **[📕 Kafka Course →](kafka/README.md)**
- **[📗 EQXJS Template Course →](eqxjs-template/README.md)**

### Course Outlines

- **[📋 NestJS Outline →](nestjs/course-outline.md)**
- **[📋 EQXJS Stub Outline →](eqxjs-framework/01-stub/course-outline.md)**
- **[⚡ EQXJS Quick Start →](eqxjs-framework/01-stub/QUICK_START.md)**
- **[📋 EQXJS Kafka Outline →](eqxjs-framework/02-custom-kafka-server/course-outline.md)**
- **[📋 Kafka Outline →](kafka/course-outline.md)**
- **[📋 EQXJS Template Outline →](eqxjs-template/course-outline.md)**

### Exercises

- **[💡 NestJS Exercises →](nestjs/exercise/README.md)**
- **[💡 EQXJS Stub Exercises →](eqxjs-framework/01-stub/exercise/README.md)**
- **[💡 EQXJS Kafka Exercises →](eqxjs-framework/02-custom-kafka-server/exercise/README.md)**
- **[💡 Kafka Exercises →](kafka/exercise/README.md)**
- **[💡 EQXJS Template Exercises →](eqxjs-template/exercise/README.md)**

---

<div align="center">

**Happy Learning! 🚀**

_Master modern backend technologies and build amazing applications_

**Last Updated**: February 18, 2026

</div>
