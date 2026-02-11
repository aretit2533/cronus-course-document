# Quick Start: Minimal EQXJS App (`@corp-ais/eqxjs-stub`)

This Quick Start boots a minimal NestJS app wired to `@corp-ais/eqxjs-stub` using the actual API: `FrameworkModule.register({ configPath, zone })`.

---

## 1) Create a new NestJS project

```bash
npx @nestjs/cli new eqxjs-quickstart
cd eqxjs-quickstart
```

## 2) Install `@corp-ais/eqxjs-stub`

```bash
npm install @corp-ais/eqxjs-stub
```

## 3) Add a minimal EQXJS config

Create `config/development.config.yaml`:

```yaml
app:
  component-name: "eqxjs-quickstart"

logging:
  level: "debug"

server:
  timeout: 30000
```

## 4) Register `FrameworkModule`

Update `src/app.module.ts`:

```ts
import { Module } from "@nestjs/common";
import { FrameworkModule } from "@corp-ais/eqxjs-stub";

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

If the app starts successfully, you’re ready to continue with Module 2 and the exercises.
