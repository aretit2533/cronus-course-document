# Documentation Index - @corp-ais/eqxjs-stub

Welcome to the comprehensive documentation for the EQXJS Framework Module. This documentation provides everything you need to understand, implement, and maintain applications using the @corp-ais/eqxjs-stub framework.

## 📚 Documentation Overview

### Getting Started

- **[Quick Start Guide](QUICK_START.md)** - Get up and running in minutes
- **[Learning Hub Overview](../README.md)** - Repository overview and course navigation

### Core Documentation

- **[Framework Module Documentation](FRAMEWORK_MODULE_DOCUMENTATION.md)** - Comprehensive technical documentation
- **[Framework Components Reference](framework-components.md)** - Complete component catalog and usage guide
- **[API Reference](API_REFERENCE.md)** - Complete API documentation and usage examples

### Course Materials

- **[Course Outline](course-outline.md)** - Full EQXJS course outline (Modules 1-10)
- **[Exercise Guide](exercise/README.md)** - Hands-on exercises for all modules

### Framework Components

#### Core Module

- [FrameworkModule](FRAMEWORK_MODULE_DOCUMENTATION.md#frameworkmodule) - Main framework bootstrapping
- [Configuration Management](FRAMEWORK_MODULE_DOCUMENTATION.md#configuration) - YAML-based config system

#### Services

- [GracefulShutdownService](API_REFERENCE.md#gracefulshutdownservice) - Application lifecycle management
- [FrameworkUtilService](API_REFERENCE.md#frameworkutilservice) - Framework utilities

#### Decorators

- [@EntryPoint](API_REFERENCE.md#entrypointevent-string-options-entrypointoptiondto) - Event entry points
- [@ConsumerMasking](API_REFERENCE.md#consumermaskingfields-string) - Log field masking
- [@DisableConsumerLogging](API_REFERENCE.md#disableconsumerloggingdisable-boolean) - Logging control
- [@SetMessageMode](API_REFERENCE.md#setmessagemodemessagetypeenum) - Message processing modes

#### Interceptors

- [AppInterceptor](API_REFERENCE.md#appinterceptor) - Global request/response handling
- [HttpInterceptor](API_REFERENCE.md#httpinterceptor) - HTTP client interception
- [RestInterceptor](API_REFERENCE.md#restinterceptor) - REST API processing

#### Utilities

- [Health Utilities](API_REFERENCE.md#health-utilities) - Application health monitoring
- [Database Utilities](API_REFERENCE.md#database-utilities) - Database management
- [Validation Utilities](API_REFERENCE.md#joi-schema-validators) - Joi schema validation

### EQXJS Ecosystem Integration

The framework integrates and re-exports the following EQXJS modules:

#### Configuration & Commands

- **@corp-ais/eqxjs-commander** - Configuration management and command handling
- **@corp-ais/eqxjs-decorator** - Custom decorators and metadata utilities

#### Transport & Communication

- **@corp-ais/eqxjs-transporter-http** - HTTP transport layer and client capabilities

#### Logging & Monitoring

- **@corp-ais/eqxjs-logger** - Comprehensive logging with masking and formatting

#### Data Processing

- **@corp-ais/eqxjs-pipes** - Data transformation and validation pipes
- **@corp-ais/eqxjs-utils** - Utility functions, message context, and common constants

#### Security & Error Handling

- **@corp-ais/eqxjs-security** - Security utilities, authentication, and validation
- **@corp-ais/eqxjs-exception** - Exception handling framework and custom errors

## 🚀 Quick Navigation

### For New Users

1. Start with [Quick Start Guide](QUICK_START.md)
2. Review [Framework Module Documentation](FRAMEWORK_MODULE_DOCUMENTATION.md#overview)
3. Follow [Basic Application Setup](QUICK_START.md#basic-setup)

### For Developers

1. Read [Framework Module Documentation](FRAMEWORK_MODULE_DOCUMENTATION.md)
2. Explore [Framework Components Reference](framework-components.md) for component details
3. Reference [API Documentation](API_REFERENCE.md) for detailed usage
4. Complete [module exercises](exercise/README.md)

### For Architects

1. Study [Architecture](FRAMEWORK_MODULE_DOCUMENTATION.md#architecture) section
2. Review [Integration Patterns](FRAMEWORK_MODULE_DOCUMENTATION.md#integration-patterns)
3. Examine [Advanced Usage](FRAMEWORK_MODULE_DOCUMENTATION.md#advanced-usage)

### For Operations

1. Understanding [Configuration](FRAMEWORK_MODULE_DOCUMENTATION.md#configuration)
2. Learn [Graceful Shutdown](API_REFERENCE.md#gracefulshutdownservice) management
3. Setup [Health Monitoring](API_REFERENCE.md#health-utilities)

## 🔍 Find What You Need

### By Topic

#### Configuration

- [Basic Configuration](QUICK_START.md#create-configuration-file)
- [Advanced Configuration](FRAMEWORK_MODULE_DOCUMENTATION.md#configuration)
- [Environment Variables](API_REFERENCE.md#environment-configuration-examples)

#### Integration

- [Basic Setup](QUICK_START.md#register-framework-module)
- [Integration Patterns](FRAMEWORK_MODULE_DOCUMENTATION.md#integration-patterns)
- [Custom Modules](FRAMEWORK_MODULE_DOCUMENTATION.md#advanced-usage)

#### Development

- [Entry Points](API_REFERENCE.md#decorators)
- [Interceptors](API_REFERENCE.md#interceptors)
- [Services](API_REFERENCE.md#services)

#### Operations

- [Health Checks](API_REFERENCE.md#health-utilities)
- [Graceful Shutdown](FRAMEWORK_MODULE_DOCUMENTATION.md#graceful-shutdown-with-custom-cleanup)
- [Troubleshooting](FRAMEWORK_MODULE_DOCUMENTATION.md#troubleshooting)

### By Component Type

#### Framework Overview

- [Framework Components Reference](framework-components.md) - Complete component catalog and usage patterns

#### Modules

- [FrameworkModule](FRAMEWORK_MODULE_DOCUMENTATION.md#frameworkmodule)
- [Integrated Modules](API_REFERENCE.md#registered-modules)

#### Services & Providers

- [All Services](API_REFERENCE.md#services)
- [Utilities](API_REFERENCE.md#utilities)

#### Decorators & Interceptors

- [All Decorators](API_REFERENCE.md#decorators)
- [All Interceptors](API_REFERENCE.md#interceptors)

#### Data Types

- [DTOs & Interfaces](API_REFERENCE.md#dtos-and-interfaces)
- [Validation Schemas](API_REFERENCE.md#joi-schema-validators)

## 🎯 Use Cases

### Microservices

- [Basic Microservice Setup](QUICK_START.md#basic-setup)
- [Event-Driven Architecture](API_REFERENCE.md#entrypointevent-string-options-entrypointoptiondto)
- [Health Monitoring](API_REFERENCE.md#health-utilities)

### REST APIs

- [REST API Setup](API_REFERENCE.md#usage-patterns)
- [Request Interception](API_REFERENCE.md#restinterceptor)
- [Logging & Masking](API_REFERENCE.md#consumermaskingfields-string)

### Enterprise Applications

- [Multi-Environment Config](FRAMEWORK_MODULE_DOCUMENTATION.md#zone-based-environments)
- [Security Integration](API_REFERENCE.md#corp-aiseqxjs-security)
- [Graceful Shutdown](FRAMEWORK_MODULE_DOCUMENTATION.md#graceful-shutdown-with-custom-cleanup)

## 📖 Learning Path

### Beginner (0-2 weeks)

1. Complete [Quick Start Guide](QUICK_START.md)
2. Build a simple application following the guide
3. Understand [basic configuration](QUICK_START.md#create-configuration-file)

### Intermediate (2-4 weeks)

1. Study [Framework Module Documentation](FRAMEWORK_MODULE_DOCUMENTATION.md)
2. Learn [decorators and interceptors](API_REFERENCE.md#decorators)
3. Complete [module exercises](exercise/README.md)

### Advanced (1-2 months)

1. Master [advanced usage patterns](FRAMEWORK_MODULE_DOCUMENTATION.md#advanced-usage)
2. Understand [ecosystem integration](API_REFERENCE.md#re-exported-modules)
3. Review the [course outline](course-outline.md) and complete remaining modules/exercises

## 🔧 Tools & Resources

### Development Tools

- **TypeDoc Configuration**: Not included in this documentation workspace
- **Build Scripts**: Not included in this documentation workspace
- **Testing Setup**: [Test configurations](QUICK_START.md#testing-setup)

### Templates & Examples

- **Templates**: Not included in this documentation workspace
- **Configuration Examples**: [Environment configs](API_REFERENCE.md#environment-configuration-examples)

### Support Resources

- **GitHub Repository**: [cronus-eqxjs-common-library-stub](https://github.com/corp-ais/cronus-eqxjs-common-library-stub)
- **Issue Tracker**: [Report bugs and requests](https://github.com/corp-ais/cronus-eqxjs-common-library-stub/issues)
- **Training Materials**: Complete training package included

## 🆕 Version Information

**Current Version**: 3.2.6
**Last Updated**: February 2026
**Compatibility**: NestJS 11+, TypeScript 5+

### Recent Changes

- Enhanced framework module documentation
- Improved TypeDoc configuration
- Added comprehensive API reference
- Updated training materials

---

## Getting Help

If you can't find what you're looking for:

1. **Search the documentation** - Use your browser's search function
2. **Check the [FAQ](FRAMEWORK_MODULE_DOCUMENTATION.md#troubleshooting)** - Common issues and solutions
3. **Review [examples](API_REFERENCE.md#usage-patterns)** - Real-world usage patterns
4. **Open an issue** - [GitHub Issues](https://github.com/corp-ais/cronus-eqxjs-common-library-stub/issues)

**Happy coding with EQXJS! 🚀**
