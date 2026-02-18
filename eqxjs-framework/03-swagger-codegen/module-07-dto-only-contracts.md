# Module 7: DTO-Only and Shared Contract Patterns

## 📚 Learning Objectives

By the end of this module, you will be able to:

- Use `--mode dtos` for shared model generation
- Design contract-sharing patterns in monorepos and multi-service systems
- Apply safe versioning strategies for evolving API contracts

---

## 7.1 Generate only DTOs

Use `--mode dtos` to generate only Data Transfer Objects without services, controllers, or modules:

```bash
# Generate DTOs with validation and tests (default behavior)
eqxjs-swagger-codegen generate -i ./swagger.json -o ./dtos --mode dtos
```

**Output structure:**

```
dtos/
├── dtos/
│   ├── user.dto.ts
│   ├── create-user-dto.dto.ts
│   └── post.dto.ts
├── tests/
│   └── dtos/
│       ├── user.dto.spec.ts
│       ├── create-user-dto.dto.spec.ts
│       └── post.dto.spec.ts
└── index.ts  # Barrel exports
```

---

## 7.2 Control validation and tests

Since v1.0.0+, validation and tests are enabled by default. Use flags to control output:

```bash
# DTOs without validation decorators
eqxjs-swagger-codegen generate -i ./swagger.json -o ./dtos --mode dtos --no-validate

# DTOs without test files
eqxjs-swagger-codegen generate -i ./swagger.json -o ./dtos --mode dtos --no-test

# Plain DTOs (no validation, no tests)
eqxjs-swagger-codegen generate -i ./swagger.json -o ./dtos --mode dtos --no-validate --no-test
```

---

## 7.3 When DTO-only mode is useful

**Use `--mode dtos` when you:**

- Need a shared contract package across services
- Want frontend/backend type synchronization
- Are building API gateway with downstream service alignment
- Need platform teams to enforce standard schemas
- Want consistent type definitions for a monorepo
- Need to generate TypeScript interfaces from API specs
- Are sharing DTOs between multiple services

---

## 7.4 Monorepo pattern

Example approach:

**Project structure:**

```
my-workspace/
├── packages/
│   ├── contracts/        # Generated DTOs
│   │   ├── dtos/
│   │   └── index.ts
│   ├── backend-api/      # NestJS backend
│   │   └── src/
│   │       └── users/
│   │           └── users.service.ts
│   └── frontend-app/     # React/Vue/Angular
│       └── src/
│           └── api/
└── openapi.yaml
```

**Regeneration workflow:**

```bash
# Generate shared contracts
eqxjs-swagger-codegen generate \
  -i ./openapi.yaml \
  -o ./packages/contracts \
  --mode dtos

# Backend imports
import { CreateUserDto, User } from '@workspace/contracts';

# Frontend imports
import type { CreateUserDto, User } from '@workspace/contracts';
```

Backend services and frontend clients import DTOs from the `contracts` package. Regenerate contracts from spec in a dedicated CI/CD pipeline step.

---

## 7.5 Versioning and change management

Recommended practices:

**Semantic versioning for contract package:**

- Major version: Breaking schema changes (removed fields, type changes)
- Minor version: Non-breaking additions (new optional fields, new DTOs)
- Patch version: Documentation or non-functional updates

**Changelog tracking:**

- Document all schema changes with migration notes
- Highlight breaking vs non-breaking changes
- Provide upgrade guidance for consumers

**CI checks:**

- Detect uncommitted generated changes
- Fail builds if specs and generated code are out of sync
- Run contract tests across all consumers

**Example CI workflow:**

```bash
#!/bin/bash
# ci-check-contracts.sh

# Regenerate from spec
eqxjs-swagger-codegen generate -i ./openapi.yaml -o ./packages/contracts --mode dtos

# Check for uncommitted changes
git diff --exit-code ./packages/contracts || {
  echo "Error: Generated contracts are out of sync with openapi.yaml"
  echo "Run: npm run generate:contracts"
  exit 1
}
```

---

## ✅ Summary

- `--mode dtos` is ideal for contract sharing at scale
- Use `--no-validate` and `--no-test` for minimal plain DTOs
- Strong versioning discipline prevents drift across consumers
- CI/CD integration ensures contracts stay synchronized

Next: [Module 8: Production Workflow and CI/CD Integration](module-08-production-cicd.md)
