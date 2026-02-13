# Module 8: Testing and Best Practices

## 📚 Learning Objectives

By the end of this module, you will:

- Write comprehensive unit tests with Jest
- Implement integration tests
- Test Kafka producers and consumers
- Follow code quality standards
- Understand deployment strategies
- Implement monitoring and observability
- Apply production best practices

---

## 8.1 Unit Testing

### 8.1.1 Testing Services

```typescript
// example.service.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { ExampleService } from "./example.service";
import { ExampleMongoRepository } from "../repositories/implements/example.mongo.repository";
import { EventProducerService } from "../producer/event.producer.service";
import { ExampleApiService } from "../external-services/example-api.service";
import {
  ConfigService,
  CustomLoggerService,
  MessageContextService,
} from "@eqxjs/stub";

describe("ExampleService", () => {
  let service: ExampleService;
  let repository: ExampleMongoRepository;
  let producerService: EventProducerService;
  let apiService: ExampleApiService;
  let messageContext: MessageContextService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ExampleService,
        {
          provide: ExampleMongoRepository,
          useValue: {
            findManyExample: jest.fn(),
            createExample: jest.fn(),
          },
        },
        {
          provide: EventProducerService,
          useValue: {
            publisher: jest.fn(),
          },
        },
        {
          provide: ExampleApiService,
          useValue: {
            getExample: jest.fn(),
          },
        },
        {
          provide: MessageContextService,
          useValue: {
            updateMessageProperties: jest.fn(),
            cloneContextMessage: jest.fn(),
            deleteContextMessage: jest.fn(),
          },
        },
        {
          provide: CustomLoggerService,
          useValue: {
            info: jest.fn(),
            error: jest.fn(),
            warn: jest.fn(),
          },
        },
        {
          provide: ConfigService,
          useValue: {
            get: jest.fn().mockReturnValue("test-topic"),
          },
        },
      ],
    }).compile();

    service = module.get<ExampleService>(ExampleService);
    repository = module.get<ExampleMongoRepository>(ExampleMongoRepository);
    producerService = module.get<EventProducerService>(EventProducerService);
    apiService = module.get<ExampleApiService>(ExampleApiService);
    messageContext = module.get<MessageContextService>(MessageContextService);
  });

  it("should be defined", () => {
    expect(service).toBeDefined();
  });

  describe("example", () => {
    it("should process request successfully", async () => {
      // Arrange
      const request = { id: "123", name: "Test" };
      const apiData = [{ id: "1", value: "api-data" }];
      const dbData = [{ id: "2", value: "db-data" }];
      const insertedData = { id: "3", ...request };
      const contextEvent = {
        header: { identity: { correlationId: "abc-123" } },
        body: dbData,
      };

      jest.spyOn(apiService, "getExample").mockResolvedValue(apiData);
      jest.spyOn(repository, "findManyExample").mockResolvedValue(dbData);
      jest.spyOn(repository, "createExample").mockResolvedValue(insertedData);
      jest
        .spyOn(messageContext, "cloneContextMessage")
        .mockReturnValue(contextEvent);

      // Act
      const result = await service.example(request);

      // Assert
      expect(messageContext.updateMessageProperties).toHaveBeenCalled();
      expect(apiService.getExample).toHaveBeenCalledWith({});
      expect(repository.findManyExample).toHaveBeenCalledWith({});
      expect(repository.createExample).toHaveBeenCalledWith(request);
      expect(messageContext.cloneContextMessage).toHaveBeenCalledWith(dbData);
      expect(producerService.publisher).toHaveBeenCalled();
      expect(result).toEqual(contextEvent);
    });

    it("should handle errors gracefully", async () => {
      // Arrange
      const request = { id: "123", name: "Test" };
      const error = new Error("Database error");

      jest.spyOn(apiService, "getExample").mockResolvedValue([]);
      jest.spyOn(repository, "findManyExample").mockRejectedValue(error);

      // Act & Assert
      await expect(service.example(request)).rejects.toThrow("Database error");
    });
  });
});
```

### 8.1.2 Testing Repositories

```typescript
// example.mongo.repository.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { ExampleMongoRepository } from "./example.mongo.repository";
import { Db, Collection } from "mongodb";
import { CustomLoggerService } from "@eqxjs/stub";
import { DATABASE_CONNECTION } from "../../../database/database.module";

describe("ExampleMongoRepository", () => {
  let repository: ExampleMongoRepository;
  let mockDb: Partial<Db>;
  let mockCollection: Partial<Collection>;

  beforeEach(async () => {
    mockCollection = {
      insertOne: jest.fn(),
      find: jest.fn().mockReturnValue({
        sort: jest.fn().mockReturnValue({
          toArray: jest.fn(),
        }),
      }),
      findOne: jest.fn(),
      findOneAndUpdate: jest.fn(),
      deleteOne: jest.fn(),
    };

    mockDb = {
      collection: jest.fn().mockReturnValue(mockCollection),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ExampleMongoRepository,
        {
          provide: DATABASE_CONNECTION,
          useValue: mockDb,
        },
        {
          provide: CustomLoggerService,
          useValue: {
            info: jest.fn(),
            error: jest.fn(),
          },
        },
      ],
    }).compile();

    repository = module.get<ExampleMongoRepository>(ExampleMongoRepository);
  });

  describe("createExample", () => {
    it("should create a document successfully", async () => {
      // Arrange
      const data = { name: "Test", value: "Value" };
      const insertResult = {
        insertedId: "mock-id-123",
      };

      (mockCollection.insertOne as jest.Mock).mockResolvedValue(insertResult);

      // Act
      const result = await repository.createExample(data);

      // Assert
      expect(mockCollection.insertOne).toHaveBeenCalledWith(
        expect.objectContaining({
          ...data,
          createdAt: expect.any(Date),
          updatedAt: expect.any(Date),
        }),
      );
      expect(result).toEqual({
        id: "mock-id-123",
        ...data,
      });
    });
  });

  describe("findManyExample", () => {
    it("should find documents with filter", async () => {
      // Arrange
      const filter = { status: "active" };
      const documents = [
        { _id: "id1", name: "Doc 1", status: "active" },
        { _id: "id2", name: "Doc 2", status: "active" },
      ];

      const mockFind = mockCollection.find as jest.Mock;
      mockFind().sort().toArray.mockResolvedValue(documents);

      // Act
      const result = await repository.findManyExample(filter);

      // Assert
      expect(mockCollection.find).toHaveBeenCalledWith(filter);
      expect(result).toHaveLength(2);
      expect(result[0]).toHaveProperty("id", "id1");
    });
  });
});
```

### 8.1.3 Testing Event Producers

```typescript
// event.producer.service.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { EventProducerService } from "./event.producer.service";
import { CustomServerKafka } from "@eqxjs/custom-kafka-server";
import {
  ConfigService,
  CustomLoggerService,
  MessageContextService,
} from "@eqxjs/stub";

jest.mock("@eqxjs/custom-kafka-server");

describe("EventProducerService", () => {
  let service: EventProducerService;
  let mockProducer: any;

  beforeEach(async () => {
    mockProducer = {
      send: jest.fn().mockResolvedValue([
        {
          topicName: "test-topic",
          partition: 0,
          errorCode: 0,
        },
      ]),
    };

    (CustomServerKafka.getProducer as jest.Mock).mockReturnValue(mockProducer);

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        EventProducerService,
        {
          provide: MessageContextService,
          useValue: {
            deleteContextMessage: jest.fn(),
          },
        },
        {
          provide: CustomLoggerService,
          useValue: {
            info: jest.fn(),
            error: jest.fn(),
            warn: jest.fn(),
            setDependencyMetadata: jest.fn(),
          },
        },
        {
          provide: ConfigService,
          useValue: {
            get: jest.fn().mockReturnValue("test-service"),
          },
        },
      ],
    }).compile();

    service = module.get<EventProducerService>(EventProducerService);
  });

  describe("publisher", () => {
    it("should publish event successfully", async () => {
      // Arrange
      const data = {
        header: {
          identity: { correlationId: "abc-123" },
          timestamp: "2024-01-01T00:00:00Z",
          source: "test-service",
        },
        body: { test: "data" },
      };
      const topic = "test.topic.v1";

      // Act
      await service.publisher(data, topic);

      // Assert
      expect(mockProducer.send).toHaveBeenCalledWith({
        topic,
        messages: [
          {
            key: undefined,
            value: JSON.stringify(data),
          },
        ],
      });
    });

    it("should retry on failure", async () => {
      // Arrange
      const data = {
        header: {
          identity: { correlationId: "abc-123" },
        },
        body: { test: "data" },
      };
      const topic = "test.topic.v1";

      process.env.RETRYKAFKACOUNTMAX = "2";

      mockProducer.send
        .mockRejectedValueOnce(new Error("Network error"))
        .mockResolvedValueOnce([{ partition: 0 }]);

      // Act
      await service.publisher(data, topic);

      // Assert
      expect(mockProducer.send).toHaveBeenCalledTimes(2);
    });
  });
});
```

---

## 8.2 Integration Testing

### 8.2.1 Testing with Real MongoDB

```typescript
// example.service.integration.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { MongoMemoryServer } from "mongodb-memory-server";
import { MongoClient, Db } from "mongodb";
import { ExampleService } from "./example.service";
import { ExampleMongoRepository } from "../repositories/implements/example.mongo.repository";
import { DATABASE_CONNECTION } from "../../../database/database.module";

describe("ExampleService Integration", () => {
  let mongoServer: MongoMemoryServer;
  let mongoClient: MongoClient;
  let db: Db;
  let service: ExampleService;
  let module: TestingModule;

  beforeAll(async () => {
    // Start in-memory MongoDB
    mongoServer = await MongoMemoryServer.create();
    const mongoUri = mongoServer.getUri();
    mongoClient = await MongoClient.connect(mongoUri);
    db = mongoClient.db("test-db");
  });

  afterAll(async () => {
    await mongoClient.close();
    await mongoServer.stop();
  });

  beforeEach(async () => {
    module = await Test.createTestingModule({
      providers: [
        ExampleService,
        ExampleMongoRepository,
        {
          provide: DATABASE_CONNECTION,
          useValue: db,
        },
        // ... other providers
      ],
    }).compile();

    service = module.get<ExampleService>(ExampleService);
  });

  afterEach(async () => {
    // Clean up collections
    const collections = await db.collections();
    for (const collection of collections) {
      await collection.deleteMany({});
    }
  });

  it("should create and retrieve data from database", async () => {
    // Arrange
    const data = { name: "Integration Test", value: "Test Data" };

    // Act
    const created = await service.createData(data);
    const retrieved = await service.findData(created.id);

    // Assert
    expect(retrieved).toMatchObject(data);
    expect(retrieved.id).toBeDefined();
    expect(retrieved.createdAt).toBeDefined();
  });
});
```

### 8.2.2 E2E Testing

```typescript
// app.e2e.spec.ts
import { Test, TestingModule } from "@nestjs/testing";
import { INestApplication } from "@nestjs/common";
import * as request from "supertest";
import { AppModule } from "../src/app.module";

describe("AppController (e2e)", () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  describe("/api/examples (POST)", () => {
    it("should create a new example", () => {
      return request(app.getHttpServer())
        .post("/api/examples")
        .send({
          name: "Test Example",
          value: "Test Value",
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty("id");
          expect(res.body.name).toBe("Test Example");
        });
    });

    it("should return 400 for invalid data", () => {
      return request(app.getHttpServer())
        .post("/api/examples")
        .send({
          invalid: "data",
        })
        .expect(400);
    });
  });

  describe("/api/examples (GET)", () => {
    it("should return all examples", () => {
      return request(app.getHttpServer())
        .get("/api/examples")
        .expect(200)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });
  });
});
```

---

## 8.3 Code Quality Standards

### 8.3.1 ESLint Configuration

```json
// .eslintrc.json
{
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "project": "tsconfig.json",
    "sourceType": "module"
  },
  "plugins": ["@typescript-eslint/eslint-plugin"],
  "extends": [
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended"
  ],
  "root": true,
  "env": {
    "node": true,
    "jest": true
  },
  "ignorePatterns": [".eslintrc.js"],
  "rules": {
    "@typescript-eslint/interface-name-prefix": "off",
    "@typescript-eslint/explicit-function-return-type": "off",
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/no-unused-vars": [
      "error",
      {
        "argsIgnorePattern": "^_"
      }
    ],
    "no-console": [
      "warn",
      {
        "allow": ["warn", "error"]
      }
    ]
  }
}
```

### 8.3.2 Prettier Configuration

```json
// .prettierrc
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 80,
  "tabWidth": 2,
  "semi": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### 8.3.3 Pre-commit Hooks

```json
// package.json
{
  "scripts": {
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "test": "jest",
    "test:cov": "jest --coverage",
    "prepare": "husky install"
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.ts": [
      "eslint --fix",
      "prettier --write",
      "jest --bail --findRelatedTests"
    ]
  }
}
```

---

## 8.4 Production Best Practices

### 8.4.1 Error Handling

```typescript
// global-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from "@nestjs/common";
import { CustomLoggerService } from "@eqxjs/stub";

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(private logger: CustomLoggerService) {}

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const message =
      exception instanceof HttpException
        ? exception.getResponse()
        : "Internal server error";

    this.logger.error("Unhandled exception", {
      statusCode: status,
      message,
      path: request.url,
      method: request.method,
      stack: exception instanceof Error ? exception.stack : undefined,
    });

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message,
    });
  }
}
```

### 8.4.2 Health Checks

```typescript
// health.controller.ts
import { Controller, Get } from "@nestjs/common";
import {
  HealthCheck,
  HealthCheckService,
  MongooseHealthIndicator,
} from "@nestjs/terminus";

@Controller("health")
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: MongooseHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([() => this.db.pingCheck("mongodb")]);
  }

  @Get("liveness")
  liveness() {
    return { status: "ok", timestamp: new Date().toISOString() };
  }

  @Get("readiness")
  @HealthCheck()
  readiness() {
    return this.health.check([
      () => this.db.pingCheck("mongodb"),
      // Add Kafka health check
      // Add other dependencies
    ]);
  }
}
```

### 8.4.3 Monitoring with Prometheus

```typescript
// metrics.module.ts
import { Module } from "@nestjs/common";
import { PrometheusModule } from "@willsoto/nestjs-prometheus";

@Module({
  imports: [
    PrometheusModule.register({
      path: "/metrics",
      defaultMetrics: {
        enabled: true,
      },
    }),
  ],
})
export class MetricsModule {}

// Using metrics in service
import { Injectable } from "@nestjs/common";
import { Counter, Histogram } from "prom-client";
import { InjectMetric } from "@willsoto/nestjs-prometheus";

@Injectable()
export class UserService {
  constructor(
    @InjectMetric("user_operations_total")
    private operationsCounter: Counter,
    @InjectMetric("user_operation_duration_seconds")
    private operationDuration: Histogram,
  ) {}

  async createUser(data: any) {
    const end = this.operationDuration.startTimer();

    try {
      const user = await this.repository.create(data);
      this.operationsCounter.inc({ operation: "create", status: "success" });
      return user;
    } catch (error) {
      this.operationsCounter.inc({ operation: "create", status: "error" });
      throw error;
    } finally {
      end({ operation: "create" });
    }
  }
}
```

---

## 8.5 Deployment

### 8.5.1 Docker Configuration

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./
COPY --from=builder /app/assets ./assets

ENV NODE_ENV=production

EXPOSE 3080

CMD ["node", "dist/main.js"]
```

### 8.5.2 Docker Compose

```yaml
# docker-compose.yml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "3080:3080"
    environment:
      - ZONE=production
      - BROKERS=kafka:9092
      - MONGODB_URI=mongodb://mongo:27017
    depends_on:
      - kafka
      - mongodb
    restart: unless-stopped

  mongodb:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    restart: unless-stopped

  kafka:
    image: confluentinc/cp-kafka:latest
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
    restart: unless-stopped

volumes:
  mongo-data:
```

### 8.5.3 CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm run test:cov
        env:
          ZONE: test

      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t myapp:latest .

      - name: Push to registry
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push myapp:latest
```

---

## 📝 Summary

In this module, you learned:

- ✅ Writing comprehensive unit tests with Jest
- ✅ Integration and E2E testing strategies
- ✅ Code quality standards with ESLint and Prettier
- ✅ Production error handling and monitoring
- ✅ Health checks and observability
- ✅ Docker containerization and deployment
- ✅ CI/CD pipeline setup

---

## 🎓 Course Completion

Congratulations! You have completed the EQXJS Template training course.

### Next Steps:

1. Complete all exercises
2. Build a sample project using the template
3. Review the code with senior developers
4. Apply learnings to production projects

---

**[← Previous: Module 7](module-07-integrations.md)** | **[Back to Course Outline](course-outline.md)** | **[View Exercises →](exercise/README.md)**
