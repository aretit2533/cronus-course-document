# Module 8 Exercises: Production Best Practices

## Overview

These exercises focus on production readiness, operational safety, and shutdown behavior.

## Exercises

1. **Graceful Shutdown Hooks**
   - Add shutdown hooks that close HTTP servers, Kafka consumers, and database clients.
   - Validate that in-flight requests are handled before exit.

2. **Health and Readiness Probes**
   - Implement readiness checks that reflect downstream dependencies.
   - Add liveness checks with sensible timeouts.

3. **Configuration Hardening**
   - Validate required environment variables at startup.
   - Fail fast with clear error messages when config is invalid.

4. **Structured Logging**
   - Enforce consistent log format for requests, errors, and background jobs.
   - Mask sensitive fields in logs.

## Deliverables

- Updated service with production-safe shutdown and checks.
- Evidence of readiness and liveness endpoints behaving correctly.
