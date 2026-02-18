# EQXJS Swagger Codegen Course Outline

## Course Overview

This course teaches how to use `@eqxjs/swagger-codegen` to generate production-ready NestJS server code and TypeScript client SDKs from Swagger/OpenAPI specifications.

Core outcomes include:

- faster API scaffolding from existing specs
- consistent DTOs and contracts across teams
- better maintainability with generated modules/controllers/services/clients

## Target Audience

- NestJS backend developers
- API platform and integration teams
- Developers maintaining Swagger/OpenAPI contracts

## Prerequisites

- Basic TypeScript and NestJS knowledge
- Basic Swagger/OpenAPI understanding (2.0 and 3.0)
- Familiarity with CLI commands and project structure

## Course Duration

~7–9 hours (1 day workshop) + optional project extension.

---

## Module 1: Introduction to Swagger Code Generation

**Duration:** 45–60 minutes

**Topics:**

- What `@eqxjs/swagger-codegen` is and where it fits in the EQXJS ecosystem
- Supported formats: Swagger/OpenAPI, JSON/YAML
- Generation modes: `server`, `client`, `both`, `dtos`
- Generated artifacts overview (DTOs, services, controllers, modules, client services, tests)
- Default behavior: tests and validation enabled by default

**Learning Objectives:**

- Explain the value of contract-first code generation
- Choose the correct generation mode for backend, frontend, or full-stack workflows

---

## Module 2: Installation and CLI Setup

**Duration:** 45–75 minutes

**Topics:**

- Install options: global install, `npx`, and local source usage
- Command structure: `generate -i <input> -o <output>`
- Core options:
  - `--mode` (server, client, both, dtos)
  - `--no-test` (skip test generation, tests enabled by default)
  - `--no-validate` (skip validation decorators, enabled by default)
- Default behavior: tests and validation are generated automatically
- Project setup best practices for generated output directories

**Learning Objectives:**

- Run generation from different environments (local/global/CI)
- Configure CLI options for team conventions

---

## Module 3: Working with Swagger 2.0 and OpenAPI 3.0

**Duration:** 60–90 minutes

**Topics:**

- Input file differences between Swagger 2.0 and OpenAPI 3.0
- JSON vs YAML format handling
- Schema mapping to TypeScript types
- Path/query/body parameter generation behavior

**Learning Objectives:**

- Prepare compatible specs for reliable generation
- Diagnose and fix common schema issues before generation

---

## Module 4: Server Code Generation (NestJS)

**Duration:** 75–105 minutes

**Topics:**

- Generated server structure by tags/resources
- DTO generation with `@ApiProperty` and validation decorators
- Controller and service scaffolding patterns
- Module generation and dependency wiring
- `app.module.ts` auto-creation and auto-update behavior (preserves custom imports)
- `index.ts` Swagger setup helper with DocumentBuilder configuration

**Learning Objectives:**

- Navigate and extend generated NestJS files safely
- Integrate generated modules into an existing NestJS application

---

## Module 5: Client SDK Generation (TypeScript + Axios)

**Duration:** 60–90 minutes

**Topics:**

- Client mode output structure
- Typed request/response handling
- URL/path parameter substitution and query/body support
- Axios instance configuration (headers, auth, interceptors)

**Learning Objectives:**

- Generate a strongly-typed client SDK from API specs
- Consume generated clients from frontend or service-to-service integrations

---

## Module 6: Validation and Testing Flags

**Duration:** 60–90 minutes

**Topics:**

- Default behavior: validation and tests enabled automatically
- Using `--no-validate` to skip class-validator decorators
- Using `--no-test` to skip test file generation
- Generated validators: `@IsString()`, `@IsEmail()`, `@IsUUID()`, `@Min()`, `@Max()`, `@ValidateNested()`, etc.
- Generated test scaffolds for controllers, services, and DTOs

**Learning Objectives:**

- Understand default validation and testing behavior
- Control generation output with `--no-validate` and `--no-test` flags
- Leverage generated tests as scaffolds for comprehensive test coverage

---

## Module 7: DTO-Only and Shared Contract Patterns

**Duration:** 45–75 minutes

**Topics:**

- `--mode dtos` for generating only Data Transfer Objects
- Using generated DTOs in monorepos and multi-service systems
- Combining `--mode dtos` with `--no-validate` and `--no-test` for plain DTOs
- Versioning strategies for API contracts
- Change management when specs evolve

**Learning Objectives:**

- Design a reusable contract package strategy
- Minimize contract drift across producers and consumers

---

## Module 8: Production Workflow and CI/CD Integration

**Duration:** 60–90 minutes

**Topics:**

- Regeneration strategy in pull request workflows
- Keeping generated code deterministic and reviewable
- Lint/build/test steps after generation
- Common pitfalls and recovery patterns

**Learning Objectives:**

- Implement a reliable automation pipeline around code generation
- Establish team rules for generated vs manually maintained code

---

## Hands-On Labs (Suggested)

1. Generate server code from Swagger 2.0 JSON and run NestJS bootstrap.
2. Generate client SDK from OpenAPI 3.0 YAML and call mock endpoints.
3. Verify default DTO validation behavior with class-validator decorators.
4. Run generated test suites and extend them with custom scenarios.
5. Generate `--mode dtos` output and consume it from a second project.
6. Use `--no-test --no-validate` to generate plain DTOs without extras.

---

## Final Capstone (Optional)

Generate a full backend + client stack from a real API specification using `--mode both`, then integrate it into an existing NestJS project with a CI step that verifies generated artifacts are up to date.