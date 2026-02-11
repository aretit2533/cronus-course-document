# Learning Hub - Backend Technologies

Welcome to the comprehensive learning hub for modern backend technologies! This repository contains complete courses on **NestJS** and **Apache Kafka**.

---

## 📚 Available Courses

| Course                                          | Description                           | Modules    | Duration    | Status      |
| ----------------------------------------------- | ------------------------------------- | ---------- | ----------- | ----------- |
| **[NestJS](#-nestjs-fundamentals-course)**      | Progressive Node.js framework         | 8 modules  | 20-30 hours | ✅ Complete |
| **[Apache Kafka](#-apache-kafka-course)**       | Distributed event streaming platform  | 9 modules  | 26 hours    | ✅ Complete |
| **[EQXJS Framework](#-eqxjs-framework-course)** | Enterprise NestJS framework ecosystem | 10 modules | 30-40 hours | ✅ Complete |

---

## 🎯 NestJS Fundamentals Course

> Build scalable and maintainable Node.js applications with TypeScript

### 📖 Quick Access

**[📋 Complete NestJS Course](nestjs/README.md)** | **[📚 Course Outline](nestjs/course-outline.md)** | **[💡 All Exercises](nestjs/exercise/README.md)**

### Course Modules

| Module       | Topic                                                                  | Content                                             | Exercises                                           | Duration |
| ------------ | ---------------------------------------------------------------------- | --------------------------------------------------- | --------------------------------------------------- | -------- |
| **Module 1** | [Introduction to NestJS](nestjs/module-01-introduction.md)             | [View](nestjs/module-01-introduction.md)            | [Exercises](nestjs/exercise/module-01-exercises.md) | 2 hours  |
| **Module 2** | [Getting Started](nestjs/module-02-getting-started.md)                 | [View](nestjs/module-02-getting-started.md)         | [Exercises](nestjs/exercise/module-02-exercises.md) | 2 hours  |
| **Module 3** | [Controllers](nestjs/module-03-controllers.md)                         | [View](nestjs/module-03-controllers.md)             | [Exercises](nestjs/exercise/module-03-exercises.md) | 3 hours  |
| **Module 4** | [Providers](nestjs/module-04-providers.md)                             | [View](nestjs/module-04-providers.md)               | [Exercises](nestjs/exercise/module-04-exercises.md) | 3 hours  |
| **Module 5** | [Modules](nestjs/module-05-modules.md)                                 | [View](nestjs/module-05-modules.md)                 | [Exercises](nestjs/exercise/module-05-exercises.md) | 3 hours  |
| **Module 6** | [Core Fundamentals](nestjs/module-06-core-fundamentals.md)             | [View](nestjs/module-06-core-fundamentals.md)       | [Exercises](nestjs/exercise/module-06-exercises.md) | 3 hours  |
| **Module 7** | [Additional Fundamentals](nestjs/module-07-additional-fundamentals.md) | [View](nestjs/module-07-additional-fundamentals.md) | [Exercises](nestjs/exercise/module-07-exercises.md) | 4 hours  |
| **Module 8** | [Practical Application](nestjs/module-08-practical-application.md)     | [View](nestjs/module-08-practical-application.md)   | [Exercises](nestjs/exercise/module-08-exercises.md) | 5 hours  |

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

| Module       | Topic                                                                               | Content                                    | Exercises                                          | Duration |
| ------------ | ----------------------------------------------------------------------------------- | ------------------------------------------ | -------------------------------------------------- | -------- |
| **Module 1** | [Introduction to Apache Kafka](kafka/module-01-introduction.md)                     | [View](kafka/module-01-introduction.md)    | [Exercises](kafka/exercise/module-01-exercises.md) | 2 hours  |
| **Module 2** | [Kafka Architecture and Core Concepts](kafka/module-02-architecture.md)             | [View](kafka/module-02-architecture.md)    | [Exercises](kafka/exercise/module-02-exercises.md) | 3 hours  |
| **Module 3** | [Setting Up Kafka](kafka/module-03-setup.md)                                        | [View](kafka/module-03-setup.md)           | [Exercises](kafka/exercise/module-03-exercises.md) | 2 hours  |
| **Module 4** | [Kafka Producers](kafka/module-04-producers.md)                                     | [View](kafka/module-04-producers.md)       | [Exercises](kafka/exercise/module-04-exercises.md) | 3 hours  |
| **Module 5** | [Kafka Consumers](kafka/module-05-consumers.md)                                     | [View](kafka/module-05-consumers.md)       | [Exercises](kafka/exercise/module-05-exercises.md) | 3 hours  |
| **Module 6** | [Topics, Partitions, and Data Management](kafka/module-06-data-management.md)       | [View](kafka/module-06-data-management.md) | [Exercises](kafka/exercise/module-06-exercises.md) | 2 hours  |
| **Module 7** | [Kafka Connect and Kafka Streams](kafka/module-07-connect-streams.md)               | [View](kafka/module-07-connect-streams.md) | [Exercises](kafka/exercise/module-07-exercises.md) | 4 hours  |
| **Module 8** | [Advanced Topics and Best Practices](kafka/module-08-advanced.md)                   | [View](kafka/module-08-advanced.md)        | [Exercises](kafka/exercise/module-08-exercises.md) | 3 hours  |
| **Module 9** | [Practical Application - Building a Real-Time System](kafka/module-09-practical.md) | [View](kafka/module-09-practical.md)       | [Exercises](kafka/exercise/module-09-exercises.md) | 4 hours  |

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

### 📖 Quick Access

**[📋 Complete EQXJS Course](eqxjs-framework/README.md)** | **[📚 Course Outline](eqxjs-framework/course-outline.md)** | **[⚡ Quick Start Guide](eqxjs-framework/QUICK_START.md)** | **[💡 All Exercises](eqxjs-framework/exercise/README.md)**

### Additional Resources

**[📖 API Reference](../cronus-eqxjs-common-library-stub/docs/API_REFERENCE.md)** | **[⚙️ Framework Module Documentation](../cronus-eqxjs-common-library-stub/docs/FRAMEWORK_MODULE_DOCUMENTATION.md)** | **[⚡ Quick Start (Library)](../cronus-eqxjs-common-library-stub/docs/QUICK_START.md)**

### Course Modules

| Module        | Topic                                                                                              | Content                                                          | Exercises                                         | Duration  |
| ------------- | -------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------- | --------- |
| **Module 1**  | [Introduction to EQXJS Framework](eqxjs-framework/module-01-introduction.md)                | [View](eqxjs-framework/module-01-introduction.md)                | [Exercises](eqxjs-framework/exercise/module-01-exercises.md) | 3-4 hours |
| **Module 2**  | [Getting Started & Setup](eqxjs-framework/module-02-getting-started.md)                     | [View](eqxjs-framework/module-02-getting-started.md)             | [Exercises](eqxjs-framework/exercise/module-02-exercises.md) | 3-4 hours |
| **Module 3**  | [Framework Module Configuration](eqxjs-framework/module-03-framework-configuration.md)     | [View](eqxjs-framework/module-03-framework-configuration.md)     | [Exercises](eqxjs-framework/exercise/module-03-exercises.md) | 3-4 hours |
| **Module 4**  | [Health Checks & Monitoring](eqxjs-framework/module-04-health-monitoring.md)               | [View](eqxjs-framework/module-04-health-monitoring.md)           | [Exercises](eqxjs-framework/exercise/module-04-exercises.md) | 3-4 hours |
| **Module 5**  | [Interceptors & HTTP Handling](eqxjs-framework/module-05-interceptors-http.md)             | [View](eqxjs-framework/module-05-interceptors-http.md)           | [Exercises](eqxjs-framework/exercise/module-05-exercises.md) | 3-4 hours |
| **Module 6**  | [Context Management & Domain Services](eqxjs-framework/module-06-context-domain.md)        | [View](eqxjs-framework/module-06-context-domain.md)              | [Exercises](eqxjs-framework/exercise/module-06-exercises.md) | 3-4 hours |
| **Module 7**  | [Decorators & Validation](eqxjs-framework/module-07-decorators-validation.md)              | [View](eqxjs-framework/module-07-decorators-validation.md)        | [Exercises](eqxjs-framework/exercise/module-07-exercises.md) | 3-4 hours |
| **Module 8**  | [Graceful Shutdown & Production](eqxjs-framework/module-08-shutdown-production.md)         | [View](eqxjs-framework/module-08-shutdown-production.md)         | TBD | 3-4 hours |
| **Module 9**  | [Practical Implementation](eqxjs-framework/module-09-practical-implementation.md)          | [View](eqxjs-framework/module-09-practical-implementation.md)    | TBD | 3-4 hours |
| **Module 10** | [Advanced Patterns & Integration](eqxjs-framework/module-10-advanced-patterns.md)          | [View](eqxjs-framework/module-10-advanced-patterns.md)           | TBD | 3-4 hours |

### 📋 What You'll Learn

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
- ✅ Command pattern and CLI tool development
- ✅ Framework utilities and type-safe constants
- ✅ Domain-driven design and event-driven architecture
- ✅ Distributed system patterns and performance optimization

### 🚀 Quick Start

```bash
# Install EQXJS Framework
npm install @corp-ais/eqxjs-stub

# Create configuration
mkdir config
touch config/development.config.yaml

# Add to your NestJS app
import { FrameworkModule } from '@corp-ais/eqxjs-stub';
```

**[👉 Start EQXJS Course](eqxjs-framework/README.md)**

---

## 🎓 Learning Paths

### Path 1: Backend Developer Track

**Recommended Order**: NestJS → EQXJS → Kafka

Perfect for building modern microservices and event-driven applications:

1. **[NestJS Fundamentals](nestjs/README.md)** - Build RESTful APIs
2. **[EQXJS Framework](eqxjs-framework/README.md)** - Enterprise patterns and utilities
3. **[Apache Kafka](kafka/README.md)** - Add event streaming capabilities

### Path 2: Enterprise Developer Track

**Recommended Order**: NestJS → EQXJS

Focus on enterprise-grade application development:

1. **[NestJS Fundamentals](nestjs/README.md)** - Core framework concepts
2. **[EQXJS Framework](eqxjs-framework/README.md)** - Production-ready patterns

### Path 3: Data Engineer Track

**Recommended Order**: Kafka → NestJS

Focus on data streaming and integration:

1. **[Apache Kafka](kafka/README.md)** - Master event streaming
2. **[NestJS Fundamentals](nestjs/README.md)** - Build data processing services

### Path 4: Full-Stack Journey

**Combined Learning**: All courses

Comprehensive understanding of modern backend architectures.

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

---

## 📊 Course Statistics

### Combined Learning

- **Total Modules**: 27 modules
- **Total Duration**: 76-96 hours
- **Exercises**: 150+ hands-on exercises
- **Projects**: 3 complete applications
- **Difficulty**: Beginner to Advanced

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
/opt/workspace/learning/
├── README.md                          # This file - Main entry point
│
├── nestjs/                            # NestJS Course
│   ├── README.md                      # NestJS course home
│   ├── course-outline.md              # Detailed curriculum
│   ├── module-01-introduction.md      # Module 1: Introduction
│   ├── module-02-getting-started.md   # Module 2: Getting Started
│   ├── module-03-controllers.md       # Module 3: Controllers
│   ├── module-04-providers.md         # Module 4: Providers
│   ├── module-05-modules.md           # Module 5: Modules
│   ├── module-06-core-fundamentals.md # Module 6: Core Fundamentals
│   ├── module-07-additional-fundamentals.md  # Module 7: Advanced
│   ├── module-08-practical-application.md    # Module 8: Practical
│   └── exercise/                      # NestJS exercises
│       ├── README.md                  # Exercise index
│       └── module-*-exercises.md      # Module exercises
│
├── kafka/                             # Apache Kafka Course
│   ├── README.md                      # Kafka course home
│   ├── course-outline.md              # Detailed curriculum
│   ├── module-01-introduction.md      # Module 1: Introduction
│   ├── module-02-architecture.md      # Module 2: Architecture
│   ├── module-03-setup.md             # Module 3: Setup
│   ├── module-04-producers.md         # Module 4: Producers
│   ├── module-05-consumers.md         # Module 5: Consumers
│   ├── module-06-data-management.md   # Module 6: Data Management
│   ├── module-07-connect-streams.md   # Module 7: Connect & Streams
│   ├── module-08-advanced.md          # Module 8: Advanced Topics
│   ├── module-09-practical.md         # Module 9: Practical App
│   └── exercise/                      # Kafka exercises
│       ├── README.md                  # Exercise index
│       └── module-*-exercises.md      # Module exercises
│
└── eqxjs-framework/                   # EQXJS Framework Course
  ├── README.md                      # EQXJS course home
  ├── QUICK_START.md                 # Quick start guide
  ├── course-outline.md              # Detailed curriculum
  ├── module-01-introduction.md      # Module 1
  ├── module-02-getting-started.md   # Module 2
  ├── module-03-framework-configuration.md # Module 3
  ├── module-04-health-monitoring.md # Module 4
  ├── module-05-interceptors-http.md # Module 5
  ├── module-06-context-domain.md    # Module 6
  ├── module-07-decorators-validation.md # Module 7
  ├── module-08-shutdown-production.md # Module 8
  ├── module-09-practical-implementation.md # Module 9
  ├── module-10-advanced-patterns.md # Module 10
  └── exercise/                      # EQXJS exercises
    ├── README.md                  # Exercise index
    └── module-*-exercises.md      # Module exercises
```

---

## 🚀 Getting Started

### Step 1: Choose Your Course

- **New to backend?** Start with [NestJS](nestjs/README.md)
- **Enterprise development?** Try [EQXJS Framework](eqxjs-framework/README.md)
- **Building data pipelines?** Start with [Kafka](kafka/README.md)
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

Navigate to your course and begin with Module 1!

---

## 💡 Study Tips

### Effective Learning Strategies

1. **Follow Module Order**: Each module builds on previous concepts
2. **Complete All Exercises**: Hands-on practice is essential
3. **Build Projects**: Apply concepts in real applications
4. **Take Notes**: Document your learning journey
5. **Review Regularly**: Revisit complex topics
6. **Join Communities**: Ask questions and help others

### Time Management

- **Daily Practice**: Dedicate 1-2 hours per day
- **Weekend Projects**: Build applications on weekends
- **Weekly Reviews**: Recap what you learned each week
- **Monthly Goals**: Complete 2-3 modules per month

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

### EQXJS Framework Course Progress

- [ ] Module 1: EQXJS Ecosystem Foundation
- [ ] Module 2: Advanced Decorators and Interceptors
- [ ] Module 3: Health Checks and Service Management
- [ ] Module 4: Security and Exception Handling
- [ ] Module 5: Data Processing and Pipes
- [ ] Module 6: Logging and Monitoring Systems
- [ ] Module 7: Transport and HTTP Integration
- [ ] Module 8: Configuration Management and Commander
- [ ] Module 9: Utilities and Framework Constants
- [ ] Module 10: Advanced Enterprise Patterns

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

### Course Outlines

- **[📋 NestJS Outline →](nestjs/course-outline.md)**
- **[📋 EQXJS Outline →](eqxjs-framework/course-outline.md)**
- **[⚡ EQXJS Quick Start →](eqxjs-framework/QUICK_START.md)**
- **[📋 Kafka Outline →](kafka/course-outline.md)**

### Exercises

- **[💡 NestJS Exercises →](nestjs/exercise/README.md)**
- **[💡 EQXJS Exercises →](eqxjs-framework/exercise/README.md)**
- **[💡 Kafka Exercises →](kafka/exercise/README.md)**

---

<div align="center">

**Happy Learning! 🚀**

_Master modern backend technologies and build amazing applications_

**Last Updated**: February 10, 2026

</div>
