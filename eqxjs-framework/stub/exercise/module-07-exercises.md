# Module 7 Exercises: Decorators & Validation

## 📚 Exercise Overview

These exercises focus on creating and using decorators for enterprise concerns (entrypoints, masking, rate limiting) and implementing Joi-based validation + transformation.

### 🎯 Learning Objectives

- Build custom decorators using `SetMetadata` and `applyDecorators`
- Read metadata using `Reflector`
- Implement validation with Joi schemas and consistent error responses
- Implement output masking based on consumer type
- Compose decorators into reusable “endpoint presets”

### ⏱️ Estimated Time: 2.5 hours

---

## 🏁 Exercise 7.1: Build a Custom `@Entrypoint()` Decorator (Quick Start)

### Objective

Create a decorator that stores entrypoint metadata on a route handler.

### Instructions

1. Create `src/decorators/entrypoint.decorator.ts`.
2. Define `EntrypointOptions` (name, version, tags, authRequired).
3. Implement `Entrypoint(options)` using `SetMetadata("eqxjs:entrypoint", options)`.

### 📝 Tasks

- [ ] Decorator compiles
- [ ] Metadata key is consistent across app
- [ ] You can attach it to a controller method

---

## 🔧 Exercise 7.2: Create an Entrypoint Registry (Hands-On)

### Objective

Collect all entrypoints and expose them as a discovery endpoint.

### Instructions

1. Create a service `EntrypointRegistry` that stores:

- entrypoint name
- controller + method
- tags
- version

2. Create an interceptor that:

- reads entrypoint metadata via `Reflector`
- registers access events (timestamp, requestId, userId)

3. Expose `GET /_meta/entrypoints` returning all known entrypoints.

### 📝 Tasks

- [ ] Endpoint lists entrypoints
- [ ] Access events are recorded per request
- [ ] Registry does not grow unbounded (add a max history size)

---

## 🚀 Exercise 7.3: Joi Validation Decorator + Interceptor (Hands-On)

### Objective

Implement `@ValidateSchema({ schema })` and an interceptor that validates request body.

### Instructions

1. Create `src/decorators/validate-schema.decorator.ts`.
2. Store metadata key `eqxjs:validation-schema`.
3. Create `SchemaValidationInterceptor`:

- validate `request.body`
- on error throw `BadRequestException` with `{ errors: [...] }`
- optionally support `stripUnknown` and `abortEarly`

### 📝 Tasks

- [ ] Invalid payload returns 400 with field-level errors
- [ ] Valid payload passes through
- [ ] Validation options affect behavior

---

## 🛡️ Exercise 7.4: Consumer Masking Decorator + Interceptor (Challenge)

### Objective

Mask sensitive fields in responses based on consumer type.

### Instructions

1. Create `@ConsumerMasking({ rules, consumerTypes })`.
2. Determine consumer type from `x-consumer-type` header.
3. Implement masking strategies:

- `partial` (show first N chars and last 2)
- `remove` (omit field)
- `hash` (sha256)
- `replace` (fixed string)

4. Apply masking only for configured consumer types.

### 📝 Tasks

- [ ] Masking applies for matching consumers
- [ ] Masking does not apply for non-matching consumers
- [ ] Deep field paths (e.g., `profile.email`) are supported

---

## 🔄 Exercise 7.5: Transform Input Data (Hands-On)

### Objective

Implement a transformation decorator that normalizes request input.

### Instructions

1. Create `@TransformData({ rules, direction })`.
2. Implement built-in transformers:

- `trim`, `toLowerCase`, `toNumber`

3. Apply transformations before validation in the request pipeline.

### 📝 Tasks

- [ ] Input is normalized before validation
- [ ] Transformation is applied only for configured fields

---

## 🧩 Exercise 7.6: Compose Decorators into an `@ApiEndpoint()` Preset (Challenge)

### Objective

Create a reusable decorator preset that bundles:

- `@Entrypoint()`
- `@ValidateSchema()`
- `@ConsumerMasking()`

### Instructions

1. Create `ApiEndpoint(options)` that applies decorators conditionally.
2. Use it on at least 2 routes.

### 📝 Tasks

- [ ] Preset reduces boilerplate
- [ ] Options are flexible
- [ ] Routes behave the same as applying decorators individually

---

## 🧪 Exercise 7.7: Unit Tests for Decorators + Interceptors (Project)

### Objective

Add test coverage for metadata and interceptor behavior.

### Instructions

1. Test that decorators set metadata keys correctly.
2. Test interceptors:

- validation fail/success
- masking applied/not applied
- preset composes behaviors

### 📝 Tasks

- [ ] Tests validate metadata presence using `Reflect.getMetadata` or `Reflector`
- [ ] Interceptor tests cover at least 6 scenarios total

---

## ✅ Wrap-up

When you finish, you’ll have a production-friendly decorator toolkit: discovery-ready entrypoints, robust validation, safe data masking, and reusable endpoint presets.
