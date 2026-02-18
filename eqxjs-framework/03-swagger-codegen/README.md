# EQXJS Swagger Codegen Course

This course teaches how to use `@eqxjs/swagger-codegen` to generate NestJS server code and TypeScript client SDKs from Swagger/OpenAPI contracts.

## 📑 Table of Contents

- [🎯 Course Overview](#-course-overview)
- [👥 Target Audience](#-target-audience)
- [📋 Prerequisites](#-prerequisites)
- [🚀 Quick Start](#-quick-start)
- [📚 Course Modules](#-course-modules)
- [📋 Course Outline](course-outline.md)

---

## 🎯 Course Overview

Key outcomes:

- generate server scaffolding (DTOs, controllers, services, modules)
- generate typed client SDKs (Axios + DTOs)
- enforce validation and improve test readiness
- standardize API contracts across teams

---

## 👥 Target Audience

- NestJS backend developers
- API platform and integration teams
- Developers maintaining Swagger/OpenAPI contracts

---

## 📋 Prerequisites

- Node.js 22+
- TypeScript and NestJS fundamentals
- Basic understanding of Swagger/OpenAPI

---

## 🚀 Quick Start

```bash
# One-off usage with default settings (includes tests & validation)
npx @eqxjs/swagger-codegen generate -i ./swagger.json -o ./generated

# Generate both server + client
npx @eqxjs/swagger-codegen generate -i ./openapi.yaml -o ./generated --mode both

# Generate only DTOs without tests or validation
npx @eqxjs/swagger-codegen generate -i ./swagger.json -o ./types --mode dtos --no-test --no-validate
```

---

## 📚 Course Modules

| Module | Topic | Content | Duration |
| ------ | ----- | ------- | -------- |
| 1 | Introduction to Swagger Code Generation | [Module 1](module-01-introduction.md) | 45–60m |
| 2 | Installation and CLI Setup | [Module 2](module-02-installation-cli-setup.md) | 45–75m |
| 3 | Working with Swagger 2.0 and OpenAPI 3.0 | [Module 3](module-03-swagger-openapi-formats.md) | 60–90m |
| 4 | Server Code Generation (NestJS) | [Module 4](module-04-server-code-generation.md) | 75–105m |
| 5 | Client SDK Generation (TypeScript + Axios) | [Module 5](module-05-client-sdk-generation.md) | 60–90m |
| 6 | Validation and Testing Flags | [Module 6](module-06-validation-testing.md) | 60–90m |
| 7 | DTO-Only and Shared Contract Patterns | [Module 7](module-07-dto-only-contracts.md) | 45–75m |
| 8 | Production Workflow and CI/CD Integration | [Module 8](module-08-production-cicd.md) | 60–90m |

---

## 🔗 Reference

- Package: `@eqxjs/swagger-codegen`
- NPM README: generation modes, command flags, and examples
