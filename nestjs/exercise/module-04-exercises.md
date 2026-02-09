# Module 4 Exercises: Providers and Dependency Injection

## Overview
These exercises focus on creating services, implementing dependency injection, understanding provider scopes, and building layered architectures.

---

## Exercise 1: Create a Complete User Service

### Objective
Build a UserService with full CRUD operations and inject it into a controller.

### Instructions

#### Step 1: Generate Service
```bash
# Create new project or use existing
nest new users-service-app
cd users-service-app

# Generate service
nest generate service users --no-spec

# Generate controller
nest generate controller users --no-spec
```

#### Step 2: Create User Interface and DTO
**Create `src/users/interfaces/user.interface.ts`:**
```typescript
export interface User {
  id: number;
  username: string;
  email: string;
  firstName: string;
  lastName: string;
  age: number;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

**Create `src/users/dto/create-user.dto.ts`:**
```typescript
export class CreateUserDto {
  username: string;
  email: string;
  firstName: string;
  lastName: string;
  age: number;
}
```

**Create `src/users/dto/update-user.dto.ts`:**
```typescript
export class UpdateUserDto {
  username?: string;
  email?: string;
  firstName?: string;
  lastName?: string;
  age?: number;
  isActive?: boolean;
}
```

#### Step 3: Implement UserService
**Update `src/users/users.service.ts`:**
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { User } from './interfaces/user.interface';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Injectable()
export class UsersService {
  private users: User[] = [
    {
      id: 1,
      username: 'john_doe',
      email: 'john@example.com',
      firstName: 'John',
      lastName: 'Doe',
      age: 30,
      isActive: true,
      createdAt: new Date('2024-01-01'),
      updatedAt: new Date('2024-01-01'),
    },
    {
      id: 2,
      username: 'jane_smith',
      email: 'jane@example.com',
      firstName: 'Jane',
      lastName: 'Smith',
      age: 25,
      isActive: true,
      createdAt: new Date('2024-01-02'),
      updatedAt: new Date('2024-01-02'),
    },
  ];

  /**
   * Get all users
   */
  findAll(): User[] {
    return this.users;
  }

  /**
   * Find user by ID
   */
  findOne(id: number): User {
    const user = this.users.find(u => u.id === id);
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }

  /**
   * Find user by email
   */
  findByEmail(email: string): User | undefined {
    return this.users.find(u => u.email === email);
  }

  /**
   * Find user by username
   */
  findByUsername(username: string): User | undefined {
    return this.users.find(u => u.username === username);
  }

  /**
   * Create new user
   */
  create(createUserDto: CreateUserDto): User {
    // Check if email already exists
    if (this.findByEmail(createUserDto.email)) {
      throw new Error('Email already exists');
    }

    // Check if username already exists
    if (this.findByUsername(createUserDto.username)) {
      throw new Error('Username already exists');
    }

    const newUser: User = {
      id: Math.max(...this.users.map(u => u.id), 0) + 1,
      ...createUserDto,
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    this.users.push(newUser);
    return newUser;
  }

  /**
   * Update user
   */
  update(id: number, updateUserDto: UpdateUserDto): User {
    const user = this.findOne(id);
    
    // Check email uniqueness if email is being updated
    if (updateUserDto.email && updateUserDto.email !== user.email) {
      if (this.findByEmail(updateUserDto.email)) {
        throw new Error('Email already exists');
      }
    }

    // Check username uniqueness if username is being updated
    if (updateUserDto.username && updateUserDto.username !== user.username) {
      if (this.findByUsername(updateUserDto.username)) {
        throw new Error('Username already exists');
      }
    }

    const index = this.users.findIndex(u => u.id === id);
    this.users[index] = {
      ...user,
      ...updateUserDto,
      updatedAt: new Date(),
    };

    return this.users[index];
  }

  /**
   * Delete user
   */
  remove(id: number): void {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    this.users.splice(index, 1);
  }

  /**
   * Soft delete - deactivate user
   */
  deactivate(id: number): User {
    return this.update(id, { isActive: false });
  }

  /**
   * Reactivate user
   */
  activate(id: number): User {
    return this.update(id, { isActive: true });
  }

  /**
   * Get active users only
   */
  findAllActive(): User[] {
    return this.users.filter(u => u.isActive);
  }

  /**
   * Count total users
   */
  count(): number {
    return this.users.length;
  }

  /**
   * Count active users
   */
  countActive(): number {
    return this.users.filter(u => u.isActive).length;
  }
}
```

#### Step 4: Inject Service into Controller
**Update `src/users/users.controller.ts`:**
```typescript
import { 
  Controller, 
  Get, 
  Post, 
  Put, 
  Delete, 
  Patch,
  Body, 
  Param, 
  HttpCode, 
  HttpStatus,
  ParseIntPipe 
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { User } from './interfaces/user.interface';

@Controller('users')
export class UsersController {
  // Dependency injection via constructor
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll(): User[] {
    return this.usersService.findAll();
  }

  @Get('active')
  findAllActive(): User[] {
    return this.usersService.findAllActive();
  }

  @Get('stats')
  getStats() {
    return {
      total: this.usersService.count(),
      active: this.usersService.countActive(),
    };
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number): User {
    return this.usersService.findOne(id);
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto): User {
    return this.usersService.create(createUserDto);
  }

  @Put(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateUserDto: UpdateUserDto,
  ): User {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number): void {
    this.usersService.remove(id);
  }

  @Patch(':id/deactivate')
  deactivate(@Param('id', ParseIntPipe) id: number): User {
    return this.usersService.deactivate(id);
  }

  @Patch(':id/activate')
  activate(@Param('id', ParseIntPipe) id: number): User {
    return this.usersService.activate(id);
  }
}
```

#### Step 5: Register Service in Module
**Update `src/users/users.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Export for other modules
})
export class UsersModule {}
```

#### Step 6: Test the Service
```bash
# Run application
npm run start:dev

# Test endpoints
curl http://localhost:3000/users
curl http://localhost:3000/users/1
curl http://localhost:3000/users/active
curl http://localhost:3000/users/stats
```

### Tasks to Complete
- [ ] Generate service and controller
- [ ] Create interfaces and DTOs
- [ ] Implement full CRUD in service
- [ ] Inject service into controller
- [ ] Register in module
- [ ] Test all operations
- [ ] Verify dependency injection works

### Expected Outcome
- Working service with business logic
- Proper dependency injection
- Separation of concerns
- Controller delegates to service

### Time Estimate
60-90 minutes

---

## Exercise 2: Implement Repository Pattern

### Objective
Separate data access logic from business logic using the Repository pattern.

### Instructions

#### Step 1: Create Repository
**Create `src/users/users.repository.ts`:**
```typescript
import { Injectable } from '@nestjs/common';
import { User } from './interfaces/user.interface';

@Injectable()
export class UsersRepository {
  private users: User[] = [
    {
      id: 1,
      username: 'john_doe',
      email: 'john@example.com',
      firstName: 'John',
      lastName: 'Doe',
      age: 30,
      isActive: true,
      createdAt: new Date('2024-01-01'),
      updatedAt: new Date('2024-01-01'),
    },
  ];

  /**
   * Find all users
   */
  findAll(): User[] {
    return [...this.users]; // Return copy
  }

  /**
   * Find user by ID
   */
  findById(id: number): User | undefined {
    return this.users.find(u => u.id === id);
  }

  /**
   * Find user by email
   */
  findByEmail(email: string): User | undefined {
    return this.users.find(u => u.email === email);
  }

  /**
   * Find user by username
   */
  findByUsername(username: string): User | undefined {
    return this.users.find(u => u.username === username);
  }

  /**
   * Find users by criteria
   */
  findBy(criteria: Partial<User>): User[] {
    return this.users.filter(user => {
      return Object.keys(criteria).every(key => {
        return user[key] === criteria[key];
      });
    });
  }

  /**
   * Save new user
   */
  save(user: User): User {
    this.users.push(user);
    return user;
  }

  /**
   * Update user
   */
  update(id: number, user: User): User {
    const index = this.users.findIndex(u => u.id === id);
    if (index !== -1) {
      this.users[index] = user;
      return user;
    }
    return undefined;
  }

  /**
   * Delete user
   */
  delete(id: number): boolean {
    const index = this.users.findIndex(u => u.id === id);
    if (index !== -1) {
      this.users.splice(index, 1);
      return true;
    }
    return false;
  }

  /**
   * Count users
   */
  count(criteria?: Partial<User>): number {
    if (criteria) {
      return this.findBy(criteria).length;
    }
    return this.users.length;
  }

  /**
   * Check if user exists
   */
  exists(id: number): boolean {
    return this.findById(id) !== undefined;
  }
}
```

#### Step 2: Refactor Service to Use Repository
**Update `src/users/users.service.ts`:**
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { User } from './interfaces/user.interface';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { UsersRepository } from './users.repository';

@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  findAll(): User[] {
    return this.usersRepository.findAll();
  }

  findOne(id: number): User {
    const user = this.usersRepository.findById(id);
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }

  create(createUserDto: CreateUserDto): User {
    // Business logic: Check uniqueness
    if (this.usersRepository.findByEmail(createUserDto.email)) {
      throw new Error('Email already exists');
    }

    if (this.usersRepository.findByUsername(createUserDto.username)) {
      throw new Error('Username already exists');
    }

    // Business logic: Generate ID
    const users = this.usersRepository.findAll();
    const newId = Math.max(...users.map(u => u.id), 0) + 1;

    // Business logic: Set defaults
    const newUser: User = {
      id: newId,
      ...createUserDto,
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    // Data access: Save
    return this.usersRepository.save(newUser);
  }

  update(id: number, updateUserDto: UpdateUserDto): User {
    const user = this.findOne(id);

    // Business logic: Check uniqueness
    if (updateUserDto.email && updateUserDto.email !== user.email) {
      if (this.usersRepository.findByEmail(updateUserDto.email)) {
        throw new Error('Email already exists');
      }
    }

    if (updateUserDto.username && updateUserDto.username !== user.username) {
      if (this.usersRepository.findByUsername(updateUserDto.username)) {
        throw new Error('Username already exists');
      }
    }

    // Business logic: Merge data
    const updatedUser: User = {
      ...user,
      ...updateUserDto,
      updatedAt: new Date(),
    };

    // Data access: Update
    return this.usersRepository.update(id, updatedUser);
  }

  remove(id: number): void {
    if (!this.usersRepository.exists(id)) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    this.usersRepository.delete(id);
  }

  deactivate(id: number): User {
    return this.update(id, { isActive: false });
  }

  activate(id: number): User {
    return this.update(id, { isActive: true });
  }

  findAllActive(): User[] {
    return this.usersRepository.findBy({ isActive: true });
  }

  count(): number {
    return this.usersRepository.count();
  }

  countActive(): number {
    return this.usersRepository.count({ isActive: true });
  }
}
```

#### Step 3: Register Repository in Module
**Update `src/users/users.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';

@Module({
  controllers: [UsersController],
  providers: [
    UsersService,
    UsersRepository, // Add repository
  ],
  exports: [UsersService],
})
export class UsersModule {}
```

### Tasks to Complete
- [ ] Create repository with data access methods
- [ ] Refactor service to use repository
- [ ] Inject repository into service
- [ ] Test all operations still work
- [ ] Verify separation of concerns

### Expected Outcome
- Clear separation: Controller → Service → Repository
- Service handles business logic
- Repository handles data access
- Proper dependency injection chain

### Time Estimate
45-60 minutes

---

## Exercise 3: Custom Providers

### Objective
Learn different ways to register providers using factory patterns.

### Instructions

#### Step 1: Value Provider
**Create `src/config/config.constants.ts`:**
```typescript
export const CONFIG_OPTIONS = {
  appName: 'Users API',
  version: '1.0.0',
  environment: 'development',
  port: 3000,
};
```

**Register as value provider in `app.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { CONFIG_OPTIONS } from './config/config.constants';

@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: CONFIG_OPTIONS,
    },
  ],
})
export class AppModule {}
```

**Inject in controller:**
```typescript
import { Controller, Get, Inject } from '@nestjs/common';

@Controller()
export class AppController {
  constructor(@Inject('CONFIG') private config: any) {}

  @Get('config')
  getConfig() {
    return this.config;
  }
}
```

#### Step 2: Factory Provider
**Create `src/common/providers/logger.factory.ts`:**
```typescript
export interface Logger {
  log(message: string): void;
  error(message: string): void;
  warn(message: string): void;
}

export class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }

  error(message: string): void {
    console.error(`[ERROR] ${message}`);
  }

  warn(message: string): void {
    console.warn(`[WARN] ${message}`);
  }
}

export class FileLogger implements Logger {
  log(message: string): void {
    // Write to file
    console.log(`[FILE LOG] ${message}`);
  }

  error(message: string): void {
    console.error(`[FILE ERROR] ${message}`);
  }

  warn(message: string): void {
    console.warn(`[FILE WARN] ${message}`);
  }
}

export function loggerFactory(env: string): Logger {
  if (env === 'production') {
    return new FileLogger();
  }
  return new ConsoleLogger();
}
```

**Register factory provider:**
```typescript
import { Module } from '@nestjs/common';
import { loggerFactory } from './common/providers/logger.factory';

@Module({
  providers: [
    {
      provide: 'Logger',
      useFactory: () => {
        const env = process.env.NODE_ENV || 'development';
        return loggerFactory(env);
      },
    },
  ],
})
export class AppModule {}
```

**Use in service:**
```typescript
import { Injectable, Inject } from '@nestjs/common';
import { Logger } from './common/providers/logger.factory';

@Injectable()
export class UsersService {
  constructor(
    @Inject('Logger') private logger: Logger,
    private usersRepository: UsersRepository,
  ) {}

  create(createUserDto: CreateUserDto): User {
    this.logger.log(`Creating user: ${createUserDto.username}`);
    // ... rest of the code
  }
}
```

#### Step 3: Async Factory Provider
**Create `src/config/database.config.ts`:**
```typescript
export interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
}

export async function createDatabaseConfig(): Promise<DatabaseConfig> {
  // Simulate async operation (e.g., fetch from remote config)
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        host: 'localhost',
        port: 5432,
        database: 'nestjs_app',
      });
    }, 1000);
  });
}
```

**Register async factory:**
```typescript
import { Module } from '@nestjs/common';
import { createDatabaseConfig } from './config/database.config';

@Module({
  providers: [
    {
      provide: 'DATABASE_CONFIG',
      useFactory: async () => {
        return await createDatabaseConfig();
      },
    },
  ],
})
export class AppModule {}
```

#### Step 4: Class Provider
**Create alternative service implementation:**
```typescript
// users-mock.service.ts
import { Injectable } from '@nestjs/common';
import { UsersService } from './users.service';

@Injectable()
export class UsersMockService extends UsersService {
  // Override methods for testing
  findAll() {
    return [
      {
        id: 1,
        username: 'mock_user',
        email: 'mock@example.com',
        firstName: 'Mock',
        lastName: 'User',
        age: 99,
        isActive: true,
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    ];
  }
}
```

**Register with useClass:**
```typescript
@Module({
  providers: [
    {
      provide: UsersService,
      useClass: process.env.NODE_ENV === 'test' 
        ? UsersMockService 
        : UsersService,
    },
    UsersRepository,
  ],
})
export class UsersModule {}
```

### Tasks to Complete
- [ ] Create value provider
- [ ] Implement factory provider
- [ ] Create async factory provider
- [ ] Try class provider
- [ ] Inject custom providers
- [ ] Test different scenarios

### Expected Outcome
- Understanding of provider types
- Ability to choose right provider type
- Flexible configuration options

### Time Estimate
60-90 minutes

---

## Exercise 4: Multi-Layer Architecture

### Objective
Build a complete multi-layer architecture: Controller → Service → Repository → Entity.

### Instructions

#### Step 1: Project Structure
```
src/
├── products/
│   ├── controllers/
│   │   └── products.controller.ts
│   ├── services/
│   │   └── products.service.ts
│   ├── repositories/
│   │   └── products.repository.ts
│   ├── entities/
│   │   └── product.entity.ts
│   ├── dto/
│   │   ├── create-product.dto.ts
│   │   └── update-product.dto.ts
│   ├── interfaces/
│   │   └── product.interface.ts
│   └── products.module.ts
└── common/
    ├── interfaces/
    │   └── repository.interface.ts
    └── exceptions/
        └── business.exception.ts
```

#### Step 2: Create Base Repository Interface
**Create `src/common/interfaces/repository.interface.ts`:**
```typescript
export interface IRepository<T> {
  findAll(): T[];
  findById(id: number): T | undefined;
  save(entity: T): T;
  update(id: number, entity: T): T | undefined;
  delete(id: number): boolean;
  count(): number;
}
```

#### Step 3: Create Product Entity
**Create `src/products/entities/product.entity.ts`:**
```typescript
export class Product {
  id: number;
  name: string;
  description: string;
  price: number;
  stock: number;
  category: string;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;

  constructor(partial: Partial<Product>) {
    Object.assign(this, partial);
  }

  // Business methods
  isInStock(): boolean {
    return this.stock > 0 && this.isActive;
  }

  canPurchase(quantity: number): boolean {
    return this.isInStock() && this.stock >= quantity;
  }

  decreaseStock(quantity: number): void {
    if (!this.canPurchase(quantity)) {
      throw new Error('Insufficient stock');
    }
    this.stock -= quantity;
    this.updatedAt = new Date();
  }

  increaseStock(quantity: number): void {
    this.stock += quantity;
    this.updatedAt = new Date();
  }
}
```

####Step 4: Create Repository
**Create `src/products/repositories/products.repository.ts`:**
```typescript
import { Injectable } from '@nestjs/common';
import { IRepository } from '../../common/interfaces/repository.interface';
import { Product } from '../entities/product.entity';

@Injectable()
export class ProductsRepository implements IRepository<Product> {
  private products: Product[] = [
    new Product({
      id: 1,
      name: 'Laptop',
      description: 'High performance laptop',
      price: 999.99,
      stock: 10,
      category: 'Electronics',
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    }),
  ];

  findAll(): Product[] {
    return [...this.products];
  }

  findById(id: number): Product | undefined {
    return this.products.find(p => p.id === id);
  }

  findByCategory(category: string): Product[] {
    return this.products.filter(p => p.category === category);
  }

  findBy(criteria: Partial<Product>): Product[] {
    return this.products.filter(product => {
      return Object.keys(criteria).every(key => {
        return product[key] === criteria[key];
      });
    });
  }

  save(product: Product): Product {
    this.products.push(product);
    return product;
  }

  update(id: number, product: Product): Product | undefined {
    const index = this.products.findIndex(p => p.id === id);
    if (index !== -1) {
      this.products[index] = product;
      return product;
    }
    return undefined;
  }

  delete(id: number): boolean {
    const index = this.products.findIndex(p => p.id === id);
    if (index !== -1) {
      this.products.splice(index, 1);
      return true;
    }
    return false;
  }

  count(): number {
    return this.products.length;
  }
}
```

#### Step 5: Create Service
**Create `src/products/services/products.service.ts`:**
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { Product } from '../entities/product.entity';
import { ProductsRepository } from '../repositories/products.repository';
import { CreateProductDto } from '../dto/create-product.dto';
import { UpdateProductDto } from '../dto/update-product.dto';

@Injectable()
export class ProductsService {
  constructor(private readonly productsRepository: ProductsRepository) {}

  findAll(): Product[] {
    return this.productsRepository.findAll();
  }

  findOne(id: number): Product {
    const product = this.productsRepository.findById(id);
    if (!product) {
      throw new NotFoundException(`Product with ID ${id} not found`);
    }
    return product;
  }

  findByCategory(category: string): Product[] {
    return this.productsRepository.findByCategory(category);
  }

  findAvailable(): Product[] {
    return this.productsRepository.findBy({ isActive: true })
      .filter(p => p.stock > 0);
  }

  create(createProductDto: CreateProductDto): Product {
    const products = this.productsRepository.findAll();
    const newId = Math.max(...products.map(p => p.id), 0) + 1;

    const product = new Product({
      id: newId,
      ...createProductDto,
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    });

    return this.productsRepository.save(product);
  }

  update(id: number, updateProductDto: UpdateProductDto): Product {
    const product = this.findOne(id);

    const updated = new Product({
      ...product,
      ...updateProductDto,
      updatedAt: new Date(),
    });

    return this.productsRepository.update(id, updated);
  }

  remove(id: number): void {
    const product = this.findOne(id);
    this.productsRepository.delete(id);
  }

  purchase(id: number, quantity: number): Product {
    const product = this.findOne(id);
    product.decreaseStock(quantity);
    return this.productsRepository.update(id, product);
  }

  restock(id: number, quantity: number): Product {
    const product = this.findOne(id);
    product.increaseStock(quantity);
    return this.productsRepository.update(id, product);
  }
}
```

#### Step 6: Create Controller
**Create `src/products/controllers/products.controller.ts`:**
```typescript
import { 
  Controller, 
  Get, 
  Post, 
  Put, 
  Delete, 
  Body, 
  Param, 
  Query,
  ParseIntPipe,
  HttpCode,
  HttpStatus 
} from '@nestjs/common';
import { ProductsService } from '../services/products.service';
import { CreateProductDto } from '../dto/create-product.dto';
import { UpdateProductDto } from '../dto/update-product.dto';

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  findAll(@Query('category') category?: string) {
    if (category) {
      return this.productsService.findByCategory(category);
    }
    return this.productsService.findAll();
  }

  @Get('available')
  findAvailable() {
    return this.productsService.findAvailable();
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.productsService.findOne(id);
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }

  @Put(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateProductDto: UpdateProductDto,
  ) {
    return this.productsService.update(id, updateProductDto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number) {
    this.productsService.remove(id);
  }

  @Post(':id/purchase')
  purchase(
    @Param('id', ParseIntPipe) id: number,
    @Body('quantity', ParseIntPipe) quantity: number,
  ) {
    return this.productsService.purchase(id, quantity);
  }

  @Post(':id/restock')
  restock(
    @Param('id', ParseIntPipe) id: number,
    @Body('quantity', ParseIntPipe) quantity: number,
  ) {
    return this.productsService.restock(id, quantity);
  }
}
```

#### Step 7: Create Module
**Create `src/products/products.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { ProductsController } from './controllers/products.controller';
import { ProductsService } from './services/products.service';
import { ProductsRepository } from './repositories/products.repository';

@Module({
  controllers: [ProductsController],
  providers: [ProductsService, ProductsRepository],
  exports: [ProductsService],
})
export class ProductsModule {}
```

### Tasks to Complete
- [ ] Create all layers (Entity, Repository, Service, Controller)
- [ ] Implement dependency injection chain
- [ ] Add business logic in Entity
- [ ] Test complete flow
- [ ] Verify separation of concerns

### Expected Outcome
- Clean architecture
- Clear responsibilities
- Testable components
- Professional structure

### Time Estimate
90-120 minutes

---

## Challenge Exercise: E-Commerce Backend

### Objective
Build a multi-module e-commerce backend with proper DI.

### Requirements

1. **Modules:**
   - ProductsModule
   - OrdersModule
   - CustomersModule
   - InventoryModule

2. **Dependencies:**
   - OrdersService depends on ProductsService
   - OrdersService depends on CustomersService
   - InventoryModule is shared

3. **Features:**
   - Create orders with multiple products
   - Check stock availability
   - Calculate totals
   - Track order history

### Time Estimate
180+ minutes

---

## Submission Checklist

- [ ] UserService with full CRUD completed
- [ ] Repository pattern implemented
- [ ] Custom providers working
- [ ] Multi-layer architecture built
- [ ] All dependency injections working
- [ ] Code well-organized and tested

---

## Additional Resources

- [NestJS Providers Documentation](https://docs.nestjs.com/providers)
- [Dependency Injection](https://docs.nestjs.com/fundamentals/custom-providers)
- [Provider Scopes](https://docs.nestjs.com/fundamentals/injection-scopes)
- [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html)

---

## Next Steps

Continue to [Module 5 Exercises](module-05-exercises.md) to learn about modules and application structure!
