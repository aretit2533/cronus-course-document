# Module 5: Client SDK Generation (TypeScript + Axios)

## 📚 Learning Objectives

By the end of this module, you will be able to:

- Generate TypeScript client services from API specs
- Use generated clients with typed requests/responses
- Configure Axios behavior for authentication and integrations

---

## 5.1 Generate client SDK

```bash
eqxjs-swagger-codegen generate -i ./swagger.json -o ./generated --mode client
```

Combined output:

```bash
eqxjs-swagger-codegen generate -i ./swagger.json -o ./generated --mode both
```

---

## 5.2 Typical client output

```text
generated/
├── dtos/
└── users/
    └── users.client.ts
```

Each client service usually maps to resource tags/paths from the spec.

---

## 5.3 Using generated clients

```ts
import { UsersService } from "./generated/users/users.client";

const usersService = new UsersService("https://api.example.com", {
  headers: {
    Authorization: "Bearer TOKEN",
  },
});

async function run() {
  const users = await usersService.getUsers();
  console.log(users);
}
```

---

## 5.4 Client features

- automatic URL path parameter substitution
- query parameter support
- request body support for mutation endpoints
- typed DTO responses
- configurable Axios instance options

---

## ✅ Summary

- Client mode provides a typed SDK for API consumption.
- `both` mode gives server and client artifacts in one run.

Next: [Module 6: Validation and Testing Flags](module-06-validation-testing.md)
