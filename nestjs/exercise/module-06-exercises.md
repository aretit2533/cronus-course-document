# Module 6 Exercises: Core Fundamentals

## Overview
These exercises cover exception handling, testing, application lifecycle, and implementing SOLID principles in practice.

---

## Exercise 1: Custom Exception Filter

### Objective
Create a custom exception filter to handle errors consistently across your application.

### Instructions

#### Step 1: Create Exception Filter
```bash
nest new exception-handling-app
cd exception-handling-app

# Create filter
nest g filter common/http-exception --flat
```

#### Step 2: Implement Custom Exception Filter
**`src/common/http-exception.filter.ts`:**
```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const message =
      exception instanceof HttpException
        ? exception.message
        : 'Internal server error';

    const errorResponse = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: message,
      ...(exception instanceof HttpException && {
        details: exception.getResponse(),
      }),
    };

    // Log the error
    console.error('Exception caught:', {
      ...errorResponse,
      stack: exception instanceof Error ? exception.stack : undefined,
    });

    response.status(status).json(errorResponse);
  }
}
```

#### Step 3: Create Custom Exceptions
**Create `src/common/exceptions/business.exception.ts`:**
```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

export class BusinessException extends HttpException {
  constructor(message: string) {
    super(
      {
        statusCode: HttpStatus.BAD_REQUEST,
        message,
        error: 'Business Rule Violation',
      },
      HttpStatus.BAD_REQUEST,
    );
  }
}

export class NotFoundException extends HttpException {
  constructor(resource: string, id: number | string) {
    super(
      {
        statusCode: HttpStatus.NOT_FOUND,
        message: `${resource} with ID ${id} not found`,
        error: 'Not Found',
      },
      HttpStatus.NOT_FOUND,
    );
  }
}

export class ValidationException extends HttpException {
  constructor(errors: Record<string, any>) {
    super(
      {
        statusCode: HttpStatus.UNPROCESSABLE_ENTITY,
        message: 'Validation failed',
        errors,
      },
      HttpStatus.UNPROCESSABLE_ENTITY,
    );
  }
}
```

#### Step 4: Apply Filter Globally
**Update `src/main.ts`:**
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { HttpExceptionFilter } from './common/http-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply exception filter globally
  app.useGlobalFilters(new HttpExceptionFilter());
  
  await app.listen(3000);
}
bootstrap();
```

#### Step 5: Test Custom Exceptions
**Create `src/users/users.service.ts`:**
```typescript
import { Injectable } from '@nestjs/common';
import { NotFoundException, BusinessException } from '../common/exceptions/business.exception';

@Injectable()
export class UsersService {
  private users = [
    { id: 1, name: 'John', email: 'john@example.com' },
  ];

  findOne(id: number) {
    const user = this.users.find(u => u.id === id);
    if (!user) {
      throw new NotFoundException('User', id);
    }
    return user;
  }

  create(email: string) {
    if (this.users.find(u => u.email === email)) {
      throw new BusinessException(`User with email ${email} already exists`);
    }
    const user = { id: Date.now(), name: 'New User', email };
    this.users.push(user);
    return user;
  }
}
```

#### Step 6: Test Error Responses
```bash
# Test not found
curl http://localhost:3000/users/999

# Expected response:
{
  "statusCode": 404,
  "timestamp": "2024-01-01T10:00:00.000Z",
  "path": "/users/999",
  "method": "GET",
  "message": "User with ID 999 not found",
  "details": {
    "statusCode": 404,
    "message": "User with ID 999 not found",
    "error": "Not Found"
  }
}
```

### Tasks to Complete
- [ ] Create custom exception filter
- [ ] Implement custom exception classes
- [ ] Apply filter globally
- [ ] Test different exception types
- [ ] Verify error responses are consistent

### Expected Outcome
- Consistent error handling
- Detailed error information
- Professional error responses

### Time Estimate
45-60 minutes

---

## Exercise 2: Unit Testing Services

### Objective
Write comprehensive unit tests for a service with mocked dependencies.

### Instructions

#### Step 1: Create Service to Test
**`src/products/products.service.ts`:**
```typescript
import { Injectable } from '@nestjs/common';
import { ProductsRepository } from './products.repository';

export interface Product {
  id: number;
  name: string;
  price: number;
  stock: number;
}

@Injectable()
export class ProductsService {
  constructor(private readonly repository: ProductsRepository) {}

  async findAll(): Promise<Product[]> {
    return this.repository.findAll();
  }

  async findOne(id: number): Promise<Product> {
    const product = await this.repository.findById(id);
    if (!product) {
      throw new Error('Product not found');
    }
    return product;
  }

  async create(name: string, price: number, stock: number): Promise<Product> {
    if (price < 0) {
      throw new Error('Price cannot be negative');
    }
    if (stock < 0) {
      throw new Error('Stock cannot be negative');
    }

    const product = {
      id: Date.now(),
      name,
      price,
      stock,
    };

    return this.repository.save(product);
  }

  async updateStock(id: number, quantity: number): Promise<Product> {
    const product = await this.findOne(id);
    
    const newStock = product.stock + quantity;
    if (newStock < 0) {
      throw new Error('Insufficient stock');
    }

    product.stock = newStock;
    return this.repository.update(id, product);
  }

  async calculateTotal(productId: number, quantity: number): Promise<number> {
    const product = await this.findOne(productId);
    return product.price * quantity;
  }
}
```

#### Step 2: Create Unit Test
**`src/products/products.service.spec.ts`:**
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { ProductsService } from './products.service';
import { ProductsRepository } from './products.repository';

describe('ProductsService', () => {
  let service: ProductsService;
  let repository: ProductsRepository;

  const mockRepository = {
    findAll: jest.fn(),
    findById: jest.fn(),
    save: jest.fn(),
    update: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ProductsService,
        {
          provide: ProductsRepository,
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<ProductsService>(ProductsService);
    repository = module.get<ProductsRepository>(ProductsRepository);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('findAll', () => {
    it('should return an array of products', async () => {
      const products = [
        { id: 1, name: 'Product 1', price: 10, stock: 5 },
        { id: 2, name: 'Product 2', price: 20, stock: 10 },
      ];
      mockRepository.findAll.mockResolvedValue(products);

      const result = await service.findAll();

      expect(result).toEqual(products);
      expect(mockRepository.findAll).toHaveBeenCalledTimes(1);
    });

    it('should return empty array when no products', async () => {
      mockRepository.findAll.mockResolvedValue([]);

      const result = await service.findAll();

      expect(result).toEqual([]);
      expect(mockRepository.findAll).toHaveBeenCalledTimes(1);
    });
  });

  describe('findOne', () => {
    it('should return a single product', async () => {
      const product = { id: 1, name: 'Product 1', price: 10, stock: 5 };
      mockRepository.findById.mockResolvedValue(product);

      const result = await service.findOne(1);

      expect(result).toEqual(product);
      expect(mockRepository.findById).toHaveBeenCalledWith(1);
    });

    it('should throw error when product not found', async () => {
      mockRepository.findById.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow('Product not found');
      expect(mockRepository.findById).toHaveBeenCalledWith(999);
    });
  });

  describe('create', () => {
    it('should create a new product', async () => {
      const newProduct = { id: 1, name: 'New Product', price: 15, stock: 3 };
      mockRepository.save.mockResolvedValue(newProduct);

      const result = await service.create('New Product', 15, 3);

      expect(result).toEqual(newProduct);
      expect(mockRepository.save).toHaveBeenCalledWith(
        expect.objectContaining({
          name: 'New Product',
          price: 15,
          stock: 3,
        }),
      );
    });

    it('should throw error for negative price', async () => {
      await expect(service.create('Product', -10, 5)).rejects.toThrow(
        'Price cannot be negative',
      );
      expect(mockRepository.save).not.toHaveBeenCalled();
    });

    it('should throw error for negative stock', async () => {
      await expect(service.create('Product', 10, -5)).rejects.toThrow(
        'Stock cannot be negative',
      );
      expect(mockRepository.save).not.toHaveBeenCalled();
    });
  });

  describe('updateStock', () => {
    it('should increase stock', async () => {
      const product = { id: 1, name: 'Product', price: 10, stock: 5 };
      const updated = { ...product, stock: 8 };
      mockRepository.findById.mockResolvedValue(product);
      mockRepository.update.mockResolvedValue(updated);

      const result = await service.updateStock(1, 3);

      expect(result.stock).toBe(8);
      expect(mockRepository.update).toHaveBeenCalledWith(
        1,
        expect.objectContaining({ stock: 8 }),
      );
    });

    it('should decrease stock', async () => {
      const product = { id: 1, name: 'Product', price: 10, stock: 5 };
      const updated = { ...product, stock: 3 };
      mockRepository.findById.mockResolvedValue(product);
      mockRepository.update.mockResolvedValue(updated);

      const result = await service.updateStock(1, -2);

      expect(result.stock).toBe(3);
    });

    it('should throw error when stock would be negative', async () => {
      const product = { id: 1, name: 'Product', price: 10, stock: 5 };
      mockRepository.findById.mockResolvedValue(product);

      await expect(service.updateStock(1, -10)).rejects.toThrow(
        'Insufficient stock',
      );
      expect(mockRepository.update).not.toHaveBeenCalled();
    });
  });

  describe('calculateTotal', () => {
    it('should calculate total price correctly', async () => {
      const product = { id: 1, name: 'Product', price: 10, stock: 5 };
      mockRepository.findById.mockResolvedValue(product);

      const result = await service.calculateTotal(1, 3);

      expect(result).toBe(30);
    });
  });
});
```

#### Step 3: Run Tests
```bash
# Run all tests
npm run test

# Run with coverage
npm run test:cov

# Run in watch mode
npm run test:watch

# Run specific file
npm run test products.service
```

### Tasks to Complete
- [ ] Create service with business logic
- [ ] Write unit tests with mocks
- [ ] Test happy paths
- [ ] Test error cases
- [ ] Achieve high test coverage
- [ ] Run tests successfully

### Expected Outcome
- Comprehensive test coverage
- All tests passing
- Understanding of mocking

### Time Estimate
60-90 minutes

---

## Exercise 3: E2E Testing

### Objective
Write end-to-end tests for your API endpoints.

### Instructions

#### Step 1: Set Up E2E Test
**`test/users.e2e-spec.ts`:**
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, HttpStatus } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from './../src/app.module';

describe('Users (e2e)', () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterEach(async () => {
    await app.close();
  });

  describe('/users (GET)', () => {
    it('should return an array of users', () => {
      return request(app.getHttpServer())
        .get('/users')
        .expect(HttpStatus.OK)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });
  });

  describe('/users (POST)', () => {
    it('should create a new user', () => {
      const newUser = {
        username: 'testuser',
        email: 'test@example.com',
        password: 'password123',
      };

      return request(app.getHttpServer())
        .post('/users')
        .send(newUser)
        .expect(HttpStatus.CREATED)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.username).toBe(newUser.username);
          expect(res.body.email).toBe(newUser.email);
        });
    });

    it('should return 400 for invalid data', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({ username: '' })
        .expect(HttpStatus.BAD_REQUEST);
    });
  });

  describe('/users/:id (GET)', () => {
    it('should return a single user', async () => {
      // First create a user
      const createRes = await request(app.getHttpServer())
        .post('/users')
        .send({
          username: 'getuser',
          email: 'get@example.com',
          password: 'pass',
        });

      const userId = createRes.body.id;

      // Then fetch it
      return request(app.getHttpServer())
        .get(`/users/${userId}`)
        .expect(HttpStatus.OK)
        .expect((res) => {
          expect(res.body.id).toBe(userId);
          expect(res.body.username).toBe('getuser');
        });
    });

    it('should return 404 for non-existent user', () => {
      return request(app.getHttpServer())
        .get('/users/99999')
        .expect(HttpStatus.NOT_FOUND);
    });
  });

  describe('/users/:id (PUT)', () => {
    it('should update a user', async () => {
      // Create user
      const createRes = await request(app.getHttpServer())
        .post('/users')
        .send({
          username: 'updateuser',
          email: 'update@example.com',
          password: 'pass',
        });

      const userId = createRes.body.id;

      // Update user
      return request(app.getHttpServer())
        .put(`/users/${userId}`)
        .send({ username: 'updateduser' })
        .expect(HttpStatus.OK)
        .expect((res) => {
          expect(res.body.username).toBe('updateduser');
        });
    });
  });

  describe('/users/:id (DELETE)', () => {
    it('should delete a user', async () => {
      // Create user
      const createRes = await request(app.getHttpServer())
        .post('/users')
        .send({
          username: 'deleteuser',
          email: 'delete@example.com',
          password: 'pass',
        });

      const userId = createRes.body.id;

      // Delete user
      await request(app.getHttpServer())
        .delete(`/users/${userId}`)
        .expect(HttpStatus.NO_CONTENT);

      // Verify deletion
      return request(app.getHttpServer())
        .get(`/users/${userId}`)
        .expect(HttpStatus.NOT_FOUND);
    });
  });
});
```

#### Step 2: Run E2E Tests
```bash
# Run E2E tests
npm run test:e2e

# Run specific e2e test
npm run test:e2e -- users.e2e-spec
```

### Tasks to Complete
- [ ] Write E2E tests for all endpoints
- [ ] Test complete request/response cycles
- [ ] Test error scenarios
- [ ] Test data persistence
- [ ] All E2E tests passing

### Expected Outcome
- Complete API testing
- Confidence in endpoint behavior
- Integration testing coverage

### Time Estimate
60-90 minutes

---

## Exercise 4: Configuration Module with Validation

### Objective
Set up configuration management with validation.

### Instructions

#### Step 1: Install Dependencies
```bash
npm install @nestjs/config joi
```

#### Step 2: Create Configuration Schema
**`src/config/configuration.ts`:**
```typescript
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  database: {
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USER || 'postgres',
    password: process.env.DATABASE_PASSWORD,
    database: process.env.DATABASE_NAME || 'nestjs_app',
  },
  jwt: {
    secret: process.env.JWT_SECRET || 'supersecret',
    expiresIn: process.env.JWT_EXPIRES_IN || '1d',
  },
  email: {
    host: process.env.EMAIL_HOST,
    port: parseInt(process.env.EMAIL_PORT, 10) || 587,
    user: process.env.EMAIL_USER,
    password: process.env.EMAIL_PASSWORD,
  },
});
```

#### Step 3: Create Validation Schema
**`src/config/config.validation.ts`:**
```typescript
import * as Joi from 'joi';

export const configValidationSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .default('development'),
  PORT: Joi.number().default(3000),
  DATABASE_HOST: Joi.string().required(),
  DATABASE_PORT: Joi.number().default(5432),
  DATABASE_USER: Joi.string().required(),
  DATABASE_PASSWORD: Joi.string().required(),
  DATABASE_NAME: Joi.string().required(),
  JWT_SECRET: Joi.string().min(32).required(),
  JWT_EXPIRES_IN: Joi.string().default('1d'),
  EMAIL_HOST: Joi.string(),
  EMAIL_PORT: Joi.number().default(587),
  EMAIL_USER: Joi.string().when('EMAIL_HOST', {
    is: Joi.exist(),
    then: Joi.required(),
  }),
  EMAIL_PASSWORD: Joi.string().when('EMAIL_HOST', {
    is: Joi.exist(),
    then: Joi.required(),
  }),
});
```

#### Step 4: Configure Module
**Update `src/app.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import configuration from './config/configuration';
import { configValidationSchema } from './config/config.validation';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [configuration],
      validationSchema: configValidationSchema,
      validationOptions: {
        abortEarly: false, // Show all errors
      },
    }),
  ],
})
export class AppModule {}
```

#### Step 5: Use Configuration
```typescript
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {
    const host = this.configService.get<string>('database.host');
    const port = this.configService.get<number>('database.port');
    console.log(`Connecting to ${host}:${port}`);
  }
}
```

### Tasks to Complete
- [ ] Set up ConfigModule
- [ ] Create configuration schema
- [ ] Add validation
- [ ] Test with valid config
- [ ] Test with invalid config
- [ ] Use config in services

### Expected Outcome
- Validated configuration
- Type-safe config access
- Early error detection

### Time Estimate
30-45 minutes

---

## Exercise 5: Implement SOLID Principles

### Objective
Refactor code to follow SOLID principles.

### Instructions

Review and refactor code examples applying each SOLID principle. Create before/after examples demonstrating improvements.

**Focus on:**
- Single Responsibility
- Dependency Inversion
- Interface Segregation
- Open/Closed principle

### Time Estimate
60-90 minutes

---

## Submission Checklist

- [ ] Custom exception filter implemented
- [ ] Unit tests with high coverage
- [ ] E2E tests for API
- [ ] Configuration with validation
- [ ] SOLID principles applied
- [ ] All tests passing

---

## Additional Resources

- [NestJS Exception Filters](https://docs.nestjs.com/exception-filters)
- [NestJS Testing](https://docs.nestjs.com/fundamentals/testing)
- [Configuration](https://docs.nestjs.com/techniques/configuration)
- [Jest Documentation](https://jestjs.io/docs/getting-started)

---

## Next Steps

Continue to [Module 7 Exercises](module-07-exercises.md) for middleware, pipes, guards, and interceptors!
