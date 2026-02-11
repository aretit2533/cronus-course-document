# Module 2: Setup and Configuration

> Configure environments, Kafka, and MongoDB for local development.

---

## Learning Goals

- Select the correct configuration file
- Set required environment variables
- Start the service locally

---

## 2.1 Environment Selection with `ZONE`

Configuration is loaded from `assets/config/<zone>.config.yaml`. The `ZONE` value is sanitized before use in `src/app.module.ts` and `src/main.ts`.

Example:

```bash
export ZONE=local
```

---

## 2.2 YAML Configuration Structure

Each YAML file contains:

- `app`: component metadata and collection name
- `log`: detail and summary logging settings
- `kafka`: client and consumer configuration
- `topics`: consume and produce topic names
- `api-services`: outbound HTTP integrations

Example file: `assets/config/local.config.yaml`.

---

## 2.3 Required Environment Variables

The service expects the following environment variables at runtime:

- `ZONE`: configuration selector
- `BROKERS`: Kafka broker list (comma-separated)
- `API_KEY` / `API_SECRET`: Kafka SASL credentials (non-local)
- `DATABASE_URL`: MongoDB base URL
- `DATABASE_NAME`: MongoDB database name
- `DATABASE_AUTH_TYPE`: auth mode for MongoDB
- `DATABASE_CERT`: CA or client cert path if needed
- `RETRYKAFKACOUNTMAX`: producer retry limit

---

## 2.4 Start the Service

```bash
npm install
npm run start:local
```

---

## 2.5 Common Setup Errors

- `ZONE` does not match a YAML file in `assets/config/`
- Kafka brokers unreachable or SASL credentials missing
- MongoDB connection string is invalid or blocked

---

## Next Module

Continue to [Module 3: REST API Flow](module-03-rest-api-flow.md).
