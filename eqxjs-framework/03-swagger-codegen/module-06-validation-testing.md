# Module 6: Validation and Testing Flags

## 📚 Learning Objectives

By the end of this module, you will be able to:

- Generate DTO validation decorators from schema metadata
- Generate test scaffolds for key artifacts
- Combine flags to strengthen generated quality baselines

---

## 6.1 `--with-validate`

Enable validation decorator generation:

```bash
eqxjs-swagger-codegen generate -i ./swagger.json -o ./generated --with-validate
```

Common generated decorators include:

- `@IsString()`, `@IsNumber()`, `@IsInt()`, `@IsBoolean()`
- `@IsEmail()`, `@IsUUID()`, `@IsUrl()`
- `@IsEnum()`
- `@IsArray()` and nested validation helpers

---

## 6.2 `--with-test`

Generate `.spec.ts` scaffolds:

```bash
eqxjs-swagger-codegen generate -i ./swagger.json -o ./generated --with-test
```

Typical output includes tests for:

- controllers
- services
- DTOs

---

## 6.3 Combine flags

```bash
eqxjs-swagger-codegen generate \
  -i ./swagger.json \
  -o ./generated \
  --with-validate \
  --with-test
```

This creates a stronger default baseline for runtime validation + test coverage.

---

## 6.4 Practical guidance

- Keep validation in DTO layer for contract consistency
- Treat generated tests as scaffolds and extend scenario coverage manually
- Run lint/build/test in CI after every regeneration

---

## ✅ Summary

- `--with-validate` turns schema hints into runtime DTO validation.
- `--with-test` accelerates initial testing setup.

Next: [Module 7: DTO-Only and Shared Contract Patterns](module-07-dto-only-contracts.md)
