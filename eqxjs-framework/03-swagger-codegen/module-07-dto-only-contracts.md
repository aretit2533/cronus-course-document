# Module 7: DTO-Only and Shared Contract Patterns

## 📚 Learning Objectives

By the end of this module, you will be able to:

- Use `--only-dtos` for shared model generation
- Design contract-sharing patterns in monorepos and multi-service systems
- Apply safe versioning strategies for evolving API contracts

---

## 7.1 Generate only DTOs

```bash
eqxjs-swagger-codegen generate -i ./swagger.json -o ./dtos --only-dtos
```

Optional validation support:

```bash
eqxjs-swagger-codegen generate -i ./swagger.json -o ./dtos --only-dtos --with-validate
```

---

## 7.2 When DTO-only is useful

- shared contract package across services
- frontend/backend type synchronization
- API gateway and downstream service alignment
- platform teams enforcing standard schemas

---

## 7.3 Monorepo pattern

Example approach:

- `packages/contracts` stores generated DTOs
- backend services import DTOs from `contracts`
- frontend clients use same DTOs for typed payload handling

Regenerate contracts from spec in a dedicated pipeline step.

---

## 7.4 Versioning and change management

Recommended practices:

- semantic versioning for generated contract package
- changelog for breaking and non-breaking schema updates
- CI checks to detect uncommitted generated changes

---

## ✅ Summary

- `--only-dtos` is ideal for contract sharing at scale.
- Strong versioning discipline prevents drift across consumers.

Next: [Module 8: Production Workflow and CI/CD Integration](module-08-production-cicd.md)
