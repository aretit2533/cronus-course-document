# Module 2: Installation and CLI Setup

## 📚 Learning Objectives

By the end of this module, you will be able to:

- Install and run `@eqxjs/swagger-codegen` in multiple ways
- Use the `generate` command with required and optional flags
- Organize output folders for maintainable generated code

---

## 2.1 Installation options

### Global CLI

```bash
npm install -g @eqxjs/swagger-codegen
```

### One-off with npx

```bash
npx @eqxjs/swagger-codegen generate -i ./swagger.json -o ./generated
```

### Local source invocation

```bash
node dist/cli.js generate -i ./swagger.json -o ./generated
```

---

## 2.2 Core command

```bash
eqxjs-swagger-codegen generate -i <input-file> -o <output-dir>
```

Required option:

- `-i, --input <path>`

Common option:

- `-o, --output <path>` (default `./generated`)

---

## 2.3 Important flags

- `-m, --mode <mode>` → `server | client | both`
- `--with-test` → generate test files
- `--with-validate` → add class-validator decorators
- `--only-dtos` → generate only DTO files

Example:

```bash
eqxjs-swagger-codegen generate \
  -i ./openapi3-example.yaml \
  -o ./generated \
  --mode both \
  --with-validate \
  --with-test
```

---

## 2.4 Output folder strategy

Recommended patterns:

- Keep generated code in a dedicated directory (for example, `src/generated`)
- Avoid mixing generated and handwritten logic in the same files
- Regenerate often; customize behavior through wrappers/extension files

---

## ✅ Summary

- You can run the generator globally, via `npx`, or locally.
- CLI flags let you shape generation for different team workflows.

Next: [Module 3: Working with Swagger 2.0 and OpenAPI 3.0](module-03-swagger-openapi-formats.md)
