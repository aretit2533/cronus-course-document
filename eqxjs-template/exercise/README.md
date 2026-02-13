# EQXJS Template - Comprehensive Exercises

Welcome to the hands-on exercises for the EQXJS Template training course! These exercises are designed to reinforce your learning and provide practical experience with the template.

---

## 📋 Table of Contents

- [Module 1: Introduction](#module-1-introduction-exercises)
- [Module 2: Getting Started](#module-2-getting-started-exercises)
- [Module 3: Architecture](#module-3-architecture-exercises)
- [Module 4: Controllers & Managers](#module-4-controllers--managers-exercises)
- [Module 5: Services & Repositories](#module-5-services--repositories-exercises)
- [Module 6: Event-Driven with Kafka](#module-6-event-driven-exercises)
- [Module 7: External Services & Database](#module-7-integrations-exercises)
- [Module 8: Testing & Best Practices](#module-8-testing-exercises)
- [Final Project](#final-project)

---

## Module 1: Introduction Exercises

### Exercise 1.1: Understanding EQXJS Components

**Objective:** Identify and understand EQXJS framework components

**Tasks:**

1. List all EQXJS modules used in the template
2. Explain the purpose of each module
3. Draw a diagram showing how components interact

**Expected Outcome:**

- Clear understanding of framework components
- Visual representation of component relationships

---

### Exercise 1.2: Message Context Flow

**Objective:** Understand message context propagation

**Tasks:**

1. Trace a request through the template
2. Identify where message context is created
3. Identify where context is cloned
4. Identify where context is deleted

**Expected Outcome:**

- Understanding of context lifecycle
- Knowledge of when to use context methods

---

## Module 2: Getting Started Exercises

### Exercise 2.1: Initial Setup

**Objective:** Set up development environment

**Tasks:**

1. Clone the template repository
2. Install all dependencies
3. Configure local environment variables
4. Start Kafka and MongoDB with Docker
5. Run the application successfully

**Expected Outcome:**

- Working development environment
- Application running on localhost:3080

**Verification:**

```bash
curl http://localhost:3080/api/example
```

---

### Exercise 2.2: Create a Custom Endpoint

**Objective:** Add a new REST endpoint

**Tasks:**

1. Create a new GET endpoint `/api/status`
2. Return service name, version, and uptime
3. Add logging to the endpoint
4. Test with curl or Postman

**Example Response:**

```json
{
  "service": "my-service",
  "version": "1.0.0",
  "uptime": "2 hours",
  "status": "healthy",
  "timestamp": "2026-02-13T10:00:00.000Z"
}
```

**Verification:**

```bash
curl http://localhost:3080/api/status
```

---

### Exercise 2.3: Configuration Management

**Objective:** Work with environment-based configuration

**Tasks:**

1. Add a new configuration value to `local.config.yaml`
2. Access the value in a service using ConfigService
3. Create different values for dev and prod environments
4. Test switching between environments

**Expected Outcome:**

- Understanding of configuration management
- Ability to use ConfigService

---

## Module 3: Architecture Exercises

### Exercise 3.1: Layer Identification

**Objective:** Identify architectural layers in existing code

**Tasks:**

1. Review the example module
2. Identify which files belong to which layer
3. Create a table mapping files to layers
4. Explain the responsibility of each layer

**Expected Outcome:**

- Clear understanding of layered architecture
- Ability to identify layer responsibilities

---

### Exercise 3.2: Design a New Feature

**Objective:** Apply architectural patterns to a new feature

**Scenario:** Design a "Product Management" feature

**Tasks:**

1. Design the layer structure (Controller, Manager, Service, Repository)
2. Define the interfaces and DTOs
3. Draw the interaction flow
4. List the dependencies for each component

**Expected Outcome:**

- Architectural design document
- Component interaction diagram

---

## Module 4: Controllers & Managers Exercises

### Exercise 4.1: Create Event Consumer

**Objective:** Implement a Kafka event consumer

**Tasks:**

1. Create a new consumer for topic `user.registered.v1`
2. Parse the incoming event
3. Log the event details
4. Delegate to manager

**Event Structure:**

```json
{
  "header": {
    "identity": {
      "correlationId": "abc-123",
      "session": "session-xyz"
    },
    "timestamp": "2026-02-13T10:00:00.000Z",
    "source": "auth-service"
  },
  "body": {
    "userId": "user-123",
    "email": "user@example.com",
    "registeredAt": "2026-02-13T10:00:00.000Z"
  }
}
```

**Expected Outcome:**

- Working event consumer
- Proper logging
- Error handling

---

### Exercise 4.2: Create REST Controller with Validation

**Objective:** Implement REST endpoint with validation

**Tasks:**

1. Create POST `/api/products` endpoint
2. Create DTO with validations:
   - name (required, min 3 chars)
   - price (required, positive number)
   - category (required, enum)
   - description (optional, max 500 chars)
3. Return validation errors for invalid input
4. Test with valid and invalid data

**Expected Outcome:**

- Validated endpoint
- Proper error responses

---

### Exercise 4.3: Manager Orchestration

**Objective:** Create a manager that coordinates multiple services

**Scenario:** Order creation manager

**Tasks:**

1. Create `OrderManager`
2. Coordinate these operations:
   - Validate product availability
   - Calculate total price
   - Create order record
   - Publish order created event
3. Handle errors at each step
4. Implement rollback on failure

**Expected Outcome:**

- Working orchestration
- Transaction-like behavior
- Proper error handling

---

## Module 5: Services & Repositories Exercises

### Exercise 5.1: Implement Service Layer

**Objective:** Create a service with business logic

**Tasks:**

1. Create `ProductService`
2. Implement business logic:
   - Validate product data
   - Check for duplicate products
   - Calculate discounted price
3. Use repository for data access
4. Add comprehensive logging

**Expected Outcome:**

- Service with business logic
- Repository integration
- Logging at key points

---

### Exercise 5.2: MongoDB Repository

**Objective:** Implement a complete repository

**Tasks:**

1. Create `ProductMongoRepository`
2. Implement interface:
   ```typescript
   interface ProductRepositoryInterface {
     create(product: Product): Promise<Product>;
     findById(id: string): Promise<Product>;
     findAll(filter: any): Promise<Product[]>;
     update(id: string, data: Partial<Product>): Promise<Product>;
     delete(id: string): Promise<boolean>;
     search(term: string): Promise<Product[]>;
   }
   ```
3. Add pagination to `findAll`
4. Implement text search
5. Add proper error handling

**Expected Outcome:**

- Complete repository implementation
- Pagination support
- Search functionality

---

### Exercise 5.3: External API Client

**Objective:** Create an external service client

**Scenario:** Call external pricing API

**Tasks:**

1. Create `PricingApiService`
2. Implement methods:
   - `getPrice(productId: string)`
   - `getBulkPrices(productIds: string[])`
3. Add retry logic
4. Add timeout handling
5. Add logging for API calls

**Expected Outcome:**

- Robust API client
- Retry mechanism
- Proper error handling

---

## Module 6: Event-Driven Exercises

### Exercise 6.1: Event Producer

**Objective:** Implement event production

**Tasks:**

1. Create event producer for `product.created.v1`
2. Include proper message context
3. Add logging for produced events
4. Test event production

**Event Structure:**

```json
{
  "header": {
    "identity": { "correlationId": "...", "session": "..." },
    "timestamp": "...",
    "source": "product-service"
  },
  "body": {
    "productId": "prod-123",
    "name": "Product Name",
    "price": 99.99,
    "category": "electronics"
  }
}
```

**Expected Outcome:**

- Working event producer
- Proper event structure
- Context propagation

---

### Exercise 6.2: Multiple Event Production

**Objective:** Publish to multiple topics

**Scenario:** Order placement triggers multiple events

**Tasks:**

1. When order is created, publish:
   - `order.created.v1`
   - `inventory.reserved.v1`
   - `notification.send.v1`
2. Use `MultipleProducer` pattern
3. Handle failure in any publication

**Expected Outcome:**

- Multiple events published
- Proper error handling
- All-or-nothing publishing

---

### Exercise 6.3: Event Versioning

**Objective:** Handle multiple event versions

**Tasks:**

1. Create consumers for both v1 and v2 of an event
2. Transform v1 events to v2 format
3. Process both versions uniformly
4. Log the version being processed

**Expected Outcome:**

- Backward compatibility
- Version transformation
- Unified processing

---

## Module 7: Integrations Exercises

### Exercise 7.1: Advanced MongoDB Operations

**Objective:** Implement complex database operations

**Tasks:**

1. Create aggregation pipeline for sales analytics:
   - Group by category
   - Calculate total sales
   - Calculate average price
   - Sort by total sales
2. Implement transaction for order processing
3. Create geospatial query for nearby stores

**Expected Outcome:**

- Working aggregations
- Transaction support
- Geospatial queries

---

### Exercise 7.2: API Integration with Circuit Breaker

**Objective:** Implement resilient API client

**Tasks:**

1. Create API client with circuit breaker
2. Configure:
   - Failure threshold: 5
   - Reset timeout: 60 seconds
3. Test circuit breaker:
   - Make API fail 5 times
   - Verify circuit opens
   - Wait for reset
   - Verify circuit closes

**Expected Outcome:**

- Circuit breaker implementation
- Proper state management
- Resilient API calls

---

### Exercise 7.3: Caching Implementation

**Objective:** Add caching layer

**Tasks:**

1. Implement caching for product queries
2. Set TTL to 5 minutes
3. Invalidate cache on updates
4. Implement cache-aside pattern

**Expected Outcome:**

- Working cache
- Proper invalidation
- Performance improvement

---

## Module 8: Testing Exercises

### Exercise 8.1: Unit Tests

**Objective:** Write comprehensive unit tests

**Tasks:**

1. Write tests for `ProductService`:
   - Test create method
   - Test validation logic
   - Test error handling
   - Achieve >80% coverage
2. Mock all dependencies
3. Test edge cases

**Expected Outcome:**

- Comprehensive test suite
- > 80% code coverage
- All edge cases covered

---

### Exercise 8.2: Integration Tests

**Objective:** Write integration tests

**Tasks:**

1. Write integration test for order flow:
   - Create order
   - Verify database record
   - Verify event published
2. Use MongoDB memory server
3. Mock Kafka

**Expected Outcome:**

- End-to-end test
- Real database operations
- Event verification

---

### Exercise 8.3: E2E API Tests

**Objective:** Test REST endpoints

**Tasks:**

1. Write E2E tests for product API:
   - Test POST /api/products
   - Test GET /api/products
   - Test GET /api/products/:id
   - Test PUT /api/products/:id
   - Test DELETE /api/products/:id
2. Test validation errors
3. Test edge cases

**Expected Outcome:**

- Complete API test suite
- All endpoints tested
- Error cases covered

---

## Final Project

### Project: Build a Complete Microservice

**Objective:** Apply all learned concepts to build a complete service

**Scenario:** Build an "Order Management Service"

### Requirements:

#### 1. REST API Endpoints

- `POST /api/orders` - Create order
- `GET /api/orders` - List orders (with pagination)
- `GET /api/orders/:id` - Get order details
- `PUT /api/orders/:id/status` - Update order status
- `DELETE /api/orders/:id` - Cancel order

#### 2. Event Consumers

- Listen to `payment.completed.v1`
- Listen to `inventory.updated.v1`
- Listen to `shipping.updated.v1`

#### 3. Event Producers

- Publish `order.created.v1`
- Publish `order.updated.v1`
- Publish `order.cancelled.v1`

#### 4. Business Logic

- Validate order items
- Calculate totals with tax
- Apply discount codes
- Check inventory availability
- Process payment
- Update order status

#### 5. Data Persistence

- Store orders in MongoDB
- Store order history
- Track status changes

#### 6. External Integrations

- Call inventory service API
- Call payment service API
- Call shipping service API

#### 7. Error Handling

- Implement retry logic
- Implement circuit breaker
- Implement dead letter queue

#### 8. Testing

- Unit tests (>80% coverage)
- Integration tests
- E2E tests

#### 9. Monitoring

- Add health checks
- Add Prometheus metrics
- Implement structured logging

#### 10. Deployment

- Create Dockerfile
- Create docker-compose.yml
- Create CI/CD pipeline

### Deliverables:

1. **Source Code**
   - Clean, well-documented code
   - Following EQXJS patterns
   - Proper layer separation

2. **Documentation**
   - README with setup instructions
   - API documentation
   - Architecture diagram
   - Sequence diagrams

3. **Tests**
   - Comprehensive test suite
   - > 80% code coverage
   - Test documentation

4. **Deployment**
   - Working Docker setup
   - CI/CD pipeline
   - Deployment guide

### Evaluation Criteria:

- **Architecture (25%)**: Proper layer separation, pattern usage
- **Code Quality (25%)**: Clean code, documentation, best practices
- **Functionality (25%)**: All features working correctly
- **Testing (15%)**: Comprehensive tests, good coverage
- **Deployment (10%)**: Working Docker and CI/CD

### Timeline:

- **Week 1:** Design and API implementation
- **Week 2:** Event handling and business logic
- **Week 3:** External integrations and error handling
- **Week 4:** Testing and documentation
- **Week 5:** Deployment and finalization

---

## 📚 Resources

### Code Examples

Each exercise has reference implementations in the `solutions/` directory.

### Documentation

- [EQXJS Framework Docs](../eqxjs-framework/README.md)
- [NestJS Documentation](https://docs.nestjs.com)
- [MongoDB Documentation](https://docs.mongodb.com)
- [Kafka Documentation](https://kafka.apache.org/documentation/)

### Support

- Ask questions in GitHub Discussions
- Review solution examples
- Request code review from instructors

---

## 🎯 Completion Checklist

Track your progress:

- [ ] Module 1 Exercises Complete
- [ ] Module 2 Exercises Complete
- [ ] Module 3 Exercises Complete
- [ ] Module 4 Exercises Complete
- [ ] Module 5 Exercises Complete
- [ ] Module 6 Exercises Complete
- [ ] Module 7 Exercises Complete
- [ ] Module 8 Exercises Complete
- [ ] Final Project Complete
- [ ] Code Review Passed
- [ ] Certification Earned

---

## 🏆 Certification

Upon completion of all exercises and the final project:

1. Submit your code for review
2. Present your final project
3. Pass the code review
4. Receive your EQXJS Developer Certificate

---

**Good luck with your exercises!** 🚀

**[← Back to Course Outline](../course-outline.md)**
