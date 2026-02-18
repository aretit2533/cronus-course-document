# Quick Start: Minimal EQXJS App (`@eqxjs-stub`)

**Quick navigation:**

- Course outline: `course-outline.md`
- Module 2 (full setup walkthrough): `module-02-getting-started.md`

This Quick Start boots a minimal NestJS app wired to `@eqxjs-stub` using the actual API: `FrameworkModule.register({ configPath, zone })`.

---

## 1) Create a new NestJS project

```bash
npx @nestjs/cli new eqxjs-quickstart
cd eqxjs-quickstart
```

## 2) Install `@eqxjs-stub`

```bash
npm install @eqxjs-stub
```

## 3) Add a minimal EQXJS config

Create `config/development.config.yaml`:

```yaml
app:
  component-name: "eqxjs-quickstart"

log:
  level: "debug"

# Required by @eqxjs/utils (UtilService.randomNanoId)
nanoid:
  alphanum: "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
  len: 21

server:
  timeout: 30000
```

## 4) Register `FrameworkModule`

Update `src/app.module.ts`:

```ts
import { Module } from "@nestjs/common";
import { FrameworkModule } from "@eqxjs-stub";

@Module({
  imports: [
    FrameworkModule.register({
      configPath: "config",
      zone: process.env.NODE_ENV || "development",
    }),
  ],
})
export class AppModule {}
```

## 5) Enable shutdown hooks (recommended)

Update `src/main.ts`:

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableShutdownHooks();
  await app.listen(process.env.PORT || 3000);
}

bootstrap();
```

## 6) Run

```bash
npm run start:dev
```

### Sanity checks

If the app doesn’t start, double-check:

- `config/development.config.yaml` exists (and matches the `zone`)
- `FrameworkModule.register({ configPath, zone })` points at the right `configPath`
- Your Node.js version is 18+

If the app starts successfully, you’re ready to continue with Module 2 and the exercises.
