# Module 8: Practical Application

## 8.1 Building a Complete CRUD Application

### Project Overview

We'll build a **Task Management API** with full CRUD operations, validation, error handling, and best practices.

### Features
- ✅ Create tasks
- ✅ List all tasks
- ✅ Get single task
- ✅ Update tasks
- ✅ Delete tasks
- ✅ Validation
- ✅ Error handling
- ✅ Filtering & pagination

---

## 8.2 Project Setup

### Initialize Project

```bash
nest new task-manager
cd task-manager
```

### Install Dependencies

```bash
npm install class-validator class-transformer
npm install --save-dev @types/express
```

### Project Structure

```
src/
├── app.module.ts
├── main.ts
├── tasks/
│   ├── tasks.module.ts
│   ├── tasks.controller.ts
│   ├── tasks.service.ts
│   ├── tasks.repository.ts
│   ├── dto/
│   │   ├── create-task.dto.ts
│   │   ├── update-task.dto.ts
│   │   └── filter-task.dto.ts
│   ├── entities/
│   │   └── task.entity.ts
│   └── enums/
│       └── task-status.enum.ts
└── common/
    ├── filters/
    │   └── http-exception.filter.ts
    └── interceptors/
        └── transform.interceptor.ts
```

---

## 8.3 Implementation

### Step 1: Create Task Status Enum

```typescript
// tasks/enums/task-status.enum.ts
export enum TaskStatus {
  OPEN = 'OPEN',
  IN_PROGRESS = 'IN_PROGRESS',
  DONE = 'DONE',
}
```

### Step 2: Create Task Entity

```typescript
// tasks/entities/task.entity.ts
import { TaskStatus } from '../enums/task-status.enum';

export class Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
  createdAt: Date;
  updatedAt: Date;
}
```

### Step 3: Create DTOs

```typescript
// tasks/dto/create-task.dto.ts
import { IsNotEmpty, IsString, MinLength, MaxLength } from 'class-validator';

export class CreateTaskDto {
  @IsNotEmpty()
  @IsString()
  @MinLength(3)
  @MaxLength(100)
  title: string;

  @IsNotEmpty()
  @IsString()
  @MinLength(10)
  @MaxLength(500)
  description: string;
}
```

```typescript
// tasks/dto/update-task.dto.ts
import { IsEnum, IsOptional, IsString, MinLength, MaxLength } from 'class-validator';
import { TaskStatus } from '../enums/task-status.enum';

export class UpdateTaskDto {
  @IsOptional()
  @IsString()
  @MinLength(3)
  @MaxLength(100)
  title?: string;

  @IsOptional()
  @IsString()
  @MinLength(10)
  @MaxLength(500)
  description?: string;

  @IsOptional()
  @IsEnum(TaskStatus)
  status?: TaskStatus;
}
```

```typescript
// tasks/dto/filter-task.dto.ts
import { IsEnum, IsOptional, IsString } from 'class-validator';
import { TaskStatus } from '../enums/task-status.enum';

export class FilterTaskDto {
  @IsOptional()
  @IsEnum(TaskStatus)
  status?: TaskStatus;

  @IsOptional()
  @IsString()
  search?: string;
}
```

### Step 4: Create Repository

```typescript
// tasks/tasks.repository.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { Task } from './entities/task.entity';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';
import { TaskStatus } from './enums/task-status.enum';
import { v4 as uuid } from 'uuid';

@Injectable()
export class TasksRepository {
  private tasks: Task[] = [];

  create(createTaskDto: CreateTaskDto): Task {
    const task: Task = {
      id: uuid(),
      ...createTaskDto,
      status: TaskStatus.OPEN,
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    this.tasks.push(task);
    return task;
  }

  findAll(): Task[] {
    return this.tasks;
  }

  findById(id: string): Task | undefined {
    return this.tasks.find(task => task.id === id);
  }

  findByStatus(status: TaskStatus): Task[] {
    return this.tasks.filter(task => task.status === status);
  }

  search(search: string): Task[] {
    return this.tasks.filter(
      task =>
        task.title.toLowerCase().includes(search.toLowerCase()) ||
        task.description.toLowerCase().includes(search.toLowerCase()),
    );
  }

  update(id: string, updateTaskDto: UpdateTaskDto): Task | undefined {
    const task = this.findById(id);
    
    if (!task) {
      return undefined;
    }

    Object.assign(task, {
      ...updateTaskDto,
      updatedAt: new Date(),
    });

    return task;
  }

  delete(id: string): boolean {
    const index = this.tasks.findIndex(task => task.id === id);
    
    if (index === -1) {
      return false;
    }

    this.tasks.splice(index, 1);
    return true;
  }
}
```

### Step 5: Create Service

```typescript
// tasks/tasks.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { TasksRepository } from './tasks.repository';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';
import { FilterTaskDto } from './dto/filter-task.dto';
import { Task } from './entities/task.entity';

@Injectable()
export class TasksService {
  constructor(private readonly tasksRepository: TasksRepository) {}

  create(createTaskDto: CreateTaskDto): Task {
    return this.tasksRepository.create(createTaskDto);
  }

  findAll(filterDto: FilterTaskDto): Task[] {
    const { status, search } = filterDto;

    let tasks = this.tasksRepository.findAll();

    if (status) {
      tasks = tasks.filter(task => task.status === status);
    }

    if (search) {
      tasks = tasks.filter(
        task =>
          task.title.toLowerCase().includes(search.toLowerCase()) ||
          task.description.toLowerCase().includes(search.toLowerCase()),
      );
    }

    return tasks;
  }

  findOne(id: string): Task {
    const task = this.tasksRepository.findById(id);

    if (!task) {
      throw new NotFoundException(`Task with ID "${id}" not found`);
    }

    return task;
  }

  update(id: string, updateTaskDto: UpdateTaskDto): Task {
    const task = this.tasksRepository.update(id, updateTaskDto);

    if (!task) {
      throw new NotFoundException(`Task with ID "${id}" not found`);
    }

    return task;
  }

  remove(id: string): void {
    const deleted = this.tasksRepository.delete(id);

    if (!deleted) {
      throw new NotFoundException(`Task with ID "${id}" not found`);
    }
  }

  updateStatus(id: string, status: TaskStatus): Task {
    return this.update(id, { status });
  }
}
```

### Step 6: Create Controller

```typescript
// tasks/tasks.controller.ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  Query,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';
import { TasksService } from './tasks.service';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';
import { FilterTaskDto } from './dto/filter-task.dto';
import { Task } from './entities/task.entity';
import { TaskStatus } from './enums/task-status.enum';

@Controller('tasks')
export class TasksController {
  constructor(private readonly tasksService: TasksService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createTaskDto: CreateTaskDto): Task {
    return this.tasksService.create(createTaskDto);
  }

  @Get()
  findAll(@Query() filterDto: FilterTaskDto): Task[] {
    return this.tasksService.findAll(filterDto);
  }

  @Get(':id')
  findOne(@Param('id') id: string): Task {
    return this.tasksService.findOne(id);
  }

  @Patch(':id')
  update(
    @Param('id') id: string,
    @Body() updateTaskDto: UpdateTaskDto,
  ): Task {
    return this.tasksService.update(id, updateTaskDto);
  }

  @Patch(':id/status')
  updateStatus(
    @Param('id') id: string,
    @Body('status') status: TaskStatus,
  ): Task {
    return this.tasksService.updateStatus(id, status);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id') id: string): void {
    this.tasksService.remove(id);
  }
}
```

### Step 7: Create Module

```typescript
// tasks/tasks.module.ts
import { Module } from '@nestjs/common';
import { TasksController } from './tasks.controller';
import { TasksService } from './tasks.service';
import { TasksRepository } from './tasks.repository';

@Module({
  controllers: [TasksController],
  providers: [TasksService, TasksRepository],
  exports: [TasksService],
})
export class TasksModule {}
```

### Step 8: Exception Filter

```typescript
// common/filters/http-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
  Logger,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(HttpExceptionFilter.name);

  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();

    const errorResponse = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: 
        typeof exceptionResponse === 'string'
          ? exceptionResponse
          : (exceptionResponse as any).message || 'Internal server error',
    };

    this.logger.error(
      `${request.method} ${request.url}`,
      JSON.stringify(errorResponse),
    );

    response.status(status).json(errorResponse);
  }
}
```

### Step 9: Transform Interceptor

```typescript
// common/interceptors/transform.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
  statusCode: number;
  timestamp: string;
}

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, Response<T>>
{
  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        data,
        statusCode: context.switchToHttp().getResponse().statusCode,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
```

### Step 10: Update App Module

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TasksModule } from './tasks/tasks.module';

@Module({
  imports: [TasksModule],
})
export class AppModule {}
```

### Step 11: Configure Main.ts

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';
import { HttpExceptionFilter } from './common/filters/http-exception.filter';
import { TransformInterceptor } from './common/interceptors/transform.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Global prefix
  app.setGlobalPrefix('api');

  // Global validation pipe
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  // Global exception filter
  app.useGlobalFilters(new HttpExceptionFilter());

  // Global interceptor
  app.useGlobalInterceptors(new TransformInterceptor());

  // Enable CORS
  app.enableCors();

  const port = process.env.PORT || 3000;
  await app.listen(port);

  console.log(`Application is running on: http://localhost:${port}/api`);
}
bootstrap();
```

---

## 8.4 Testing the API

### Using cURL

```bash
# Create a task
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Learn NestJS",
    "description": "Complete the NestJS fundamentals course"
  }'

# Get all tasks
curl http://localhost:3000/api/tasks

# Get task by ID
curl http://localhost:3000/api/tasks/{id}

# Filter tasks by status
curl http://localhost:3000/api/tasks?status=OPEN

# Search tasks
curl http://localhost:3000/api/tasks?search=NestJS

# Update task
curl -X PATCH http://localhost:3000/api/tasks/{id} \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated title",
    "status": "IN_PROGRESS"
  }'

# Update task status
curl -X PATCH http://localhost:3000/api/tasks/{id}/status \
  -H "Content-Type: application/json" \
  -d '{
    "status": "DONE"
  }'

# Delete task
curl -X DELETE http://localhost:3000/api/tasks/{id}
```

### Using Postman

1. Import collection
2. Create requests for each endpoint
3. Test validation errors
4. Test error handling

---

## 8.5 Adding Tests

### Unit Tests for Service

```typescript
// tasks/tasks.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { NotFoundException } from '@nestjs/common';
import { TasksService } from './tasks.service';
import { TasksRepository } from './tasks.repository';
import { TaskStatus } from './enums/task-status.enum';

describe('TasksService', () => {
  let service: TasksService;
  let repository: TasksRepository;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [TasksService, TasksRepository],
    }).compile();

    service = module.get<TasksService>(TasksService);
    repository = module.get<TasksRepository>(TasksRepository);
  });

  describe('create', () => {
    it('should create a task', () => {
      const createTaskDto = {
        title: 'Test Task',
        description: 'Test Description',
      };

      const result = service.create(createTaskDto);

      expect(result).toHaveProperty('id');
      expect(result.title).toBe(createTaskDto.title);
      expect(result.status).toBe(TaskStatus.OPEN);
    });
  });

  describe('findOne', () => {
    it('should return a task', () => {
      const task = service.create({
        title: 'Test Task',
        description: 'Test Description',
      });

      const result = service.findOne(task.id);

      expect(result).toEqual(task);
    });

    it('should throw NotFoundException', () => {
      expect(() => service.findOne('non-existent-id')).toThrow(
        NotFoundException,
      );
    });
  });

  describe('update', () => {
    it('should update a task', () => {
      const task = service.create({
        title: 'Test Task',
        description: 'Test Description',
      });

      const updated = service.update(task.id, {
        title: 'Updated Title',
      });

      expect(updated.title).toBe('Updated Title');
      expect(updated.id).toBe(task.id);
    });
  });

  describe('remove', () => {
    it('should remove a task', () => {
      const task = service.create({
        title: 'Test Task',
        description: 'Test Description',
      });

      service.remove(task.id);

      expect(() => service.findOne(task.id)).toThrow(NotFoundException);
    });
  });
});
```

### E2E Tests

```typescript
// test/tasks.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from './../src/app.module';

describe('TasksController (e2e)', () => {
  let app: INestApplication;
  let taskId: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    app.useGlobalPipes(new ValidationPipe());
    await app.init();
  });

  describe('/tasks (POST)', () => {
    it('should create a task', () => {
      return request(app.getHttpServer())
        .post('/tasks')
        .send({
          title: 'Test Task',
          description: 'Test Description',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          taskId = res.body.id;
        });
    });

    it('should fail validation', () => {
      return request(app.getHttpServer())
        .post('/tasks')
        .send({
          title: 'T',  // Too short
          description: 'Test Description',
        })
        .expect(400);
    });
  });

  describe('/tasks (GET)', () => {
    it('should return all tasks', () => {
      return request(app.getHttpServer())
        .get('/tasks')
        .expect(200)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });
  });

  describe('/tasks/:id (GET)', () => {
    it('should return a task', () => {
      return request(app.getHttpServer())
        .get(`/tasks/${taskId}`)
        .expect(200)
        .expect((res) => {
          expect(res.body.id).toBe(taskId);
        });
    });

    it('should return 404', () => {
      return request(app.getHttpServer())
        .get('/tasks/non-existent-id')
        .expect(404);
    });
  });

  afterAll(async () => {
    await app.close();
  });
});
```

---

## 8.6 Best Practices Demonstrated

### 1. Layered Architecture
- ✅ Controller → Service → Repository
- ✅ Clear separation of concerns
- ✅ Easy to test and maintain

### 2. DTOs for Validation
- ✅ Input validation with class-validator
- ✅ Type safety
- ✅ Self-documenting API

### 3. Error Handling
- ✅ Custom exception filter
- ✅ Consistent error responses
- ✅ Proper HTTP status codes

### 4. Interceptors
- ✅ Response transformation
- ✅ Consistent response format
- ✅ Logging capabilities

### 5. Repository Pattern
- ✅ Data access layer abstraction
- ✅ Easy to swap implementations
- ✅ Testable

---

## 8.7 Enhancements

### Add Logging Middleware

```typescript
// middleware/logger.middleware.ts
import { Injectable, NestMiddleware, Logger } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  private logger = new Logger('HTTP');

  use(req: Request, res: Response, next: NextFunction) {
    const { method, originalUrl } = req;
    const startTime = Date.now();

    res.on('finish', () => {
      const { statusCode } = res;
      const duration = Date.now() - startTime;

      this.logger.log(
        `${method} ${originalUrl} ${statusCode} - ${duration}ms`,
      );
    });

    next();
  }
}

// Apply in AppModule
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*');
  }
}
```

### Add Pagination

```typescript
// dto/pagination.dto.ts
import { IsOptional, IsInt, Min, Max } from 'class-validator';
import { Type } from 'class-transformer';

export class PaginationDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 10;
}

// Service method
findAll(filterDto: FilterTaskDto, paginationDto: PaginationDto) {
  let tasks = this.filterTasks(filterDto);
  
  const { page, limit } = paginationDto;
  const startIndex = (page - 1) * limit;
  const endIndex = page * limit;
  
  return {
    data: tasks.slice(startIndex, endIndex),
    total: tasks.length,
    page,
    limit,
    totalPages: Math.ceil(tasks.length / limit),
  };
}
```

---

## Summary

### What We Built
1. ✅ Complete CRUD API
2. ✅ Input validation
3. ✅ Error handling
4. ✅ Response transformation
5. ✅ Filtering & search
6. ✅ Unit & E2E tests
7. ✅ Best practices

### Key Takeaways
- Layered architecture (Controller → Service → Repository)
- DTOs with validation
- Proper error handling
- Global pipes, filters, and interceptors
- Comprehensive testing
- Clean, maintainable code

---

## Next Steps

### Advanced Topics to Explore
1. **Database Integration**: TypeORM, Prisma, Mongoose
2. **Authentication**: JWT, Passport, Sessions
3. **Authorization**: Role-based access control
4. **WebSockets**: Real-time features
5. **Microservices**: Distributed systems
6. **GraphQL**: Alternative to REST
7. **Caching**: Redis integration
8. **Testing**: Advanced testing strategies
9. **Deployment**: Docker, Kubernetes, Cloud platforms
10. **Monitoring**: Logging, metrics, tracing

---

## Congratulations! 🎉

You've completed the NestJS Fundamentals course! You now have a solid foundation to build scalable, maintainable Node.js applications.

### Resources for Continued Learning
- [Official NestJS Documentation](https://docs.nestjs.com/)
- [NestJS Courses](https://courses.nestjs.com/)
- [NestJS Discord Community](https://discord.gg/G7Qnnhy)
- [GitHub Examples](https://github.com/nestjs/nest/tree/master/sample)
- [Awesome NestJS](https://github.com/nestjs/awesome-nestjs)

Keep building and happy coding! 🚀

---

## 📚 Course Navigation

⬅️ **[Previous: Module 7 - Additional Fundamentals](module-07-additional-fundamentals.md)**

🏠 **[Back to Course Outline](course-outline.md)**

✅ **Course Complete!** - Start building your own NestJS applications!
