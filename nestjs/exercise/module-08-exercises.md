# Module 8 Exercises: Practical Application

## Overview
Build a complete, production-ready Task Management API applying all concepts learned throughout the course.

---

## Final Project: Task Management API

### Project Overview
Build a comprehensive Task Management system with full CRUD operations, authentication, validation, error handling, and testing.

---

## Phase 1: Project Setup and Structure

### Objective
Set up a well-structured NestJS project with all necessary modules.

### Instructions

#### Step 1: Initialize Project
```bash
# Create project
nest new task-management-api --strict
cd task-management-api

# Install dependencies
npm install @nestjs/config @nestjs/jwt @nestjs/passport passport passport-jwt
npm install class-validator class-transformer
npm install bcrypt
npm install -D @types/passport-jwt @types/bcrypt
```

#### Step 2: Create Project Structure
```bash
# Generate modules
nest g module auth
nest g module users
nest g module tasks
nest g module shared

# Generate controllers
nest g controller auth --no-spec
nest g controller users --no-spec
nest g controller tasks --no-spec

# Generate services
nest g service auth --no-spec
nest g service users --no-spec
nest g service tasks --no-spec
```

#### Step 3: Project Structure
```
src/
├── auth/
│   ├── guards/
│   │   └── jwt-auth.guard.ts
│   ├── strategies/
│   │   └── jwt.strategy.ts
│   ├── dto/
│   │   ├── login.dto.ts
│   │   └── register.dto.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── auth.module.ts
├── users/
│   ├── entities/
│   │   └── user.entity.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── users.module.ts
├── tasks/
│   ├── entities/
│   │   └── task.entity.ts
│   ├── dto/
│   │   ├── create-task.dto.ts
│   │   ├── update-task.dto.ts
│   │   └── filter-task.dto.ts
│   ├── enums/
│   │   └── task-status.enum.ts
│   ├── tasks.controller.ts
│   ├── tasks.service.ts
│   └── tasks.module.ts
├── shared/
│   ├── filters/
│   │   └── http-exception.filter.ts
│   ├── interceptors/
│   │   ├── transform.interceptor.ts
│   │   └── logging.interceptor.ts
│   ├── decorators/
│   │   ├── user.decorator.ts
│   │   └── public.decorator.ts
│   └── shared.module.ts
├── config/
│   └── configuration.ts
├── app.module.ts
└── main.ts
```

### Tasks to Complete
- [ ] Create project with CLI
- [ ] Generate all modules
- [ ] Set up folder structure
- [ ] Install dependencies

### Time Estimate
15-20 minutes

---

## Phase 2: User Management Implementation

### Objective
Implement user registration and authentication.

### Instructions

#### Step 1: Create User Entity
**`src/users/entities/user.entity.ts`:**
```typescript
export class User {
  id: number;
  username: string;
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;

  constructor(partial: Partial<User>) {
    Object.assign(this, partial);
  }
}
```

#### Step 2: Create User DTOs
**`src/users/dto/create-user.dto.ts`:**
```typescript
import {
  IsString,
  IsEmail,
  IsNotEmpty,
  MinLength,
  MaxLength,
  Matches,
} from 'class-validator';

export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  @MaxLength(20)
  username: string;

  @IsEmail()
  @IsNotEmpty()
  email: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;

  @IsString()
  @IsNotEmpty()
  firstName: string;

  @IsString()
  @IsNotEmpty()
  lastName: string;
}
```

#### Step 3: Implement Users Service
**`src/users/users.service.ts`:**
```typescript
import { Injectable, ConflictException, NotFoundException } from '@nestjs/common';
import * as bcrypt from 'bcrypt';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  private users: User[] = [];
  private idCounter = 1;

  async create(createUserDto: CreateUserDto): Promise<User> {
    // Check if username exists
    if (this.users.find(u => u.username === createUserDto.username)) {
      throw new ConflictException('Username already exists');
    }

    // Check if email exists
    if (this.users.find(u => u.email === createUserDto.email)) {
      throw new ConflictException('Email already registered');
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(createUserDto.password, 10);

    const user = new User({
      id: this.idCounter++,
      ...createUserDto,
      password: hashedPassword,
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    });

    this.users.push(user);

    // Return user without password
    const { password, ...result } = user;
    return result as User;
  }

  async findByUsername(username: string): Promise<User | undefined> {
    return this.users.find(u => u.username === username);
  }

  async findByEmail(email: string): Promise<User | undefined> {
    return this.users.find(u => u.email === email);
  }

  async findOne(id: number): Promise<User> {
    const user = this.users.find(u => u.id === id);
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    const { password, ...result } = user;
    return result as User;
  }

  async findAll(): Promise<User[]> {
    return this.users.map(({ password, ...user }) => user as User);
  }
}
```

### Tasks to Complete
- [ ] Create User entity
- [ ] Implement DTOs with validation
- [ ] Build UsersService with password hashing
- [ ] Test user creation

### Time Estimate
30-45 minutes

---

## Phase 3: Authentication Implementation

### Objective
Implement JWT-based authentication.

### Instructions

#### Step 1: Create Auth DTOs
**`src/auth/dto/login.dto.ts`:**
```typescript
import { IsString, IsNotEmpty } from 'class-validator';

export class LoginDto {
  @IsString()
  @IsNotEmpty()
  username: string;

  @IsString()
  @IsNotEmpty()
  password: string;
}
```

#### Step 2: Implement Auth Service
**`src/auth/auth.service.ts`:**
```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';
import { LoginDto } from './dto/login.dto';
import { CreateUserDto } from '../users/dto/create-user.dto';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async register(createUserDto: CreateUserDto) {
    const user = await this.usersService.create(createUserDto);
    return user;
  }

  async login(loginDto: LoginDto) {
    const user = await this.usersService.findByUsername(loginDto.username);

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(loginDto.password, user.password);

    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const payload = { sub: user.id, username: user.username };
    const access_token = await this.jwtService.signAsync(payload);

    return {
      access_token,
      user: {
        id: user.id,
        username: user.username,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
      },
    };
  }

  async validateUser(userId: number) {
    return this.usersService.findOne(userId);
  }
}
```

#### Step 3: Create JWT Strategy
**`src/auth/strategies/jwt.strategy.ts`:**
```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';
import { AuthService } from '../auth.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private configService: ConfigService,
    private authService: AuthService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_SECRET') || 'your-secret-key',
    });
  }

  async validate(payload: any) {
    const user = await this.authService.validateUser(payload.sub);
    if (!user) {
      throw new UnauthorizedException();
    }
    return user;
  }
}
```

#### Step 4: Create Auth Guard
**`src/auth/guards/jwt-auth.guard.ts`:**
```typescript
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../../shared/decorators/public.decorator';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true;
    }

    return super.canActivate(context);
  }
}
```

#### Step 5: Configure Auth Module
**`src/auth/auth.module.ts`:**
```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { JwtStrategy } from './strategies/jwt.strategy';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET') || 'your-secret-key',
        signOptions: { expiresIn: '1d' },
      }),
      inject: [ConfigService],
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

#### Step 6: Create Auth Controller
**`src/auth/auth.controller.ts`:**
```typescript
import { Controller, Post, Body, HttpCode, HttpStatus } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LoginDto } from './dto/login.dto';
import { CreateUserDto } from '../users/dto/create-user.dto';
import { Public } from '../shared/decorators/public.decorator';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Public()
  @Post('register')
  @HttpCode(HttpStatus.CREATED)
  async register(@Body() createUserDto: CreateUserDto) {
    return this.authService.register(createUserDto);
  }

  @Public()
  @Post('login')
  @HttpCode(HttpStatus.OK)
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }
}
```

### Tasks to Complete
- [ ] Implement JWT strategy
- [ ] Create auth service
- [ ] Build auth guard
- [ ] Test registration
- [ ] Test login
- [ ] Verify JWT tokens

### Time Estimate
45-60 minutes

---

## Phase 4: Task Management Implementation

### Objective
Build complete task CRUD with status management and filtering.

### Instructions

#### Step 1: Create Task Enums and Entity
**`src/tasks/enums/task-status.enum.ts`:**
```typescript
export enum TaskStatus {
  OPEN = 'OPEN',
  IN_PROGRESS = 'IN_PROGRESS',
  IN_REVIEW = 'IN_REVIEW',
  COMPLETED = 'COMPLETED',
}
```

**`src/tasks/entities/task.entity.ts`:**
```typescript
import { TaskStatus } from '../enums/task-status.enum';

export class Task {
  id: number;
  title: string;
  description: string;
  status: TaskStatus;
  priority: 'low' | 'medium' | 'high';
  dueDate?: Date;
  userId: number; // Owner of the task
  createdAt: Date;
  updatedAt: Date;

  constructor(partial: Partial<Task>) {
    Object.assign(this, partial);
  }
}
```

#### Step 2: Create Task DTOs
**`src/tasks/dto/create-task.dto.ts`:**
```typescript
import { 
  IsString, 
  IsNotEmpty, 
  IsEnum, 
  IsOptional, 
  MinLength,
  MaxLength,
  IsDateString,
} from 'class-validator';
import { TaskStatus } from '../enums/task-status.enum';

export class CreateTaskDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  @MaxLength(100)
  title: string;

  @IsString()
  @IsOptional()
  @MaxLength(1000)
  description?: string;

  @IsEnum(['low', 'medium', 'high'])
  @IsOptional()
  priority?: 'low' | 'medium' | 'high';

  @IsDateString()
  @IsOptional()
  dueDate?: string;
}
```

**`src/tasks/dto/update-task.dto.ts`:**
```typescript
import {
  IsString,
  IsEnum,
  IsOptional,
  MinLength,
  MaxLength,
  IsDateString,
} from 'class-validator';
import { TaskStatus } from '../enums/task-status.enum';

export class UpdateTaskDto {
  @IsString()
  @IsOptional()
  @MinLength(3)
  @MaxLength(100)
  title?: string;

  @IsString()
  @IsOptional()
  @MaxLength(1000)
  description?: string;

  @IsEnum(TaskStatus)
  @IsOptional()
  status?: TaskStatus;

  @IsEnum(['low', 'medium', 'high'])
  @IsOptional()
  priority?: 'low' | 'medium' | 'high';

  @IsDateString()
  @IsOptional()
  dueDate?: string;
}
```

**`src/tasks/dto/filter-task.dto.ts`:**
```typescript
import { IsEnum, IsOptional, IsString } from 'class-validator';
import { TaskStatus } from '../enums/task-status.enum';

export class FilterTaskDto {
  @IsEnum(TaskStatus)
  @IsOptional()
  status?: TaskStatus;

  @IsEnum(['low', 'medium', 'high'])
  @IsOptional()
  priority?: 'low' | 'medium' | 'high';

  @IsString()
  @IsOptional()
  search?: string;
}
```

#### Step 3: Implement Tasks Service
**`src/tasks/tasks.service.ts`:**
```typescript
import { Injectable, NotFoundException, ForbiddenException } from '@nestjs/common';
import { Task } from './entities/task.entity';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';
import { FilterTaskDto } from './dto/filter-task.dto';
import { TaskStatus } from './enums/task-status.enum';

@Injectable()
export class TasksService {
  private tasks: Task[] = [];
  private idCounter = 1;

  create(createTaskDto: CreateTaskDto, userId: number): Task {
    const task = new Task({
      id: this.idCounter++,
      ...createTaskDto,
      status: TaskStatus.OPEN,
      priority: createTaskDto.priority || 'medium',
      dueDate: createTaskDto.dueDate ? new Date(createTaskDto.dueDate) : undefined,
      userId,
      createdAt: new Date(),
      updatedAt: new Date(),
    });

    this.tasks.push(task);
    return task;
  }

  findAll(userId: number, filterDto: FilterTaskDto): Task[] {
    let tasks = this.tasks.filter(task => task.userId === userId);

    if (filterDto.status) {
      tasks = tasks.filter(task => task.status === filterDto.status);
    }

    if (filterDto.priority) {
      tasks = tasks.filter(task => task.priority === filterDto.priority);
    }

    if (filterDto.search) {
      const search = filterDto.search.toLowerCase();
      tasks = tasks.filter(
        task =>
          task.title.toLowerCase().includes(search) ||
          task.description?.toLowerCase().includes(search),
      );
    }

    return tasks;
  }

  findOne(id: number, userId: number): Task {
    const task = this.tasks.find(t => t.id === id);

    if (!task) {
      throw new NotFoundException(`Task with ID ${id} not found`);
    }

    if (task.userId !== userId) {
      throw new ForbiddenException('You do not have access to this task');
    }

    return task;
  }

  update(id: number, updateTaskDto: UpdateTaskDto, userId: number): Task {
    const task = this.findOne(id, userId);

    Object.assign(task, {
      ...updateTaskDto,
      updatedAt: new Date(),
    });

    return task;
  }

  remove(id: number, userId: number): void {
    const task = this.findOne(id, userId);
    const index = this.tasks.indexOf(task);
    this.tasks.splice(index, 1);
  }

  updateStatus(id: number, status: TaskStatus, userId: number): Task {
    return this.update(id, { status }, userId);
  }

  getStatistics(userId: number) {
    const userTasks = this.tasks.filter(task => task.userId === userId);

    return {
      total: userTasks.length,
      open: userTasks.filter(t => t.status === TaskStatus.OPEN).length,
      inProgress: userTasks.filter(t => t.status === TaskStatus.IN_PROGRESS).length,
      inReview: userTasks.filter(t => t.status === TaskStatus.IN_REVIEW).length,
      completed: userTasks.filter(t => t.status === TaskStatus.COMPLETED).length,
      byPriority: {
        high: userTasks.filter(t => t.priority === 'high').length,
        medium: userTasks.filter(t => t.priority === 'medium').length,
        low: userTasks.filter(t => t.priority === 'low').length,
      },
    };
  }
}
```

#### Step 4: Create Tasks Controller
**`src/tasks/tasks.controller.ts`:**
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
  Query,
  ParseIntPipe,
  HttpCode,
  HttpStatus,
  UseGuards,
} from '@nestjs/common';
import { TasksService } from './tasks.service';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';
import { FilterTaskDto } from './dto/filter-task.dto';
import { TaskStatus } from './enums/task-status.enum';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { User } from '../shared/decorators/user.decorator';

@Controller('tasks')
@UseGuards(JwtAuthGuard)
export class TasksController {
  constructor(private readonly tasksService: TasksService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createTaskDto: CreateTaskDto, @User('id') userId: number) {
    return this.tasksService.create(createTaskDto, userId);
  }

  @Get()
  findAll(@Query() filterDto: FilterTaskDto, @User('id') userId: number) {
    return this.tasksService.findAll(userId, filterDto);
  }

  @Get('statistics')
  getStatistics(@User('id') userId: number) {
    return this.tasksService.getStatistics(userId);
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number, @User('id') userId: number) {
    return this.tasksService.findOne(id, userId);
  }

  @Put(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateTaskDto: UpdateTaskDto,
    @User('id') userId: number,
  ) {
    return this.tasksService.update(id, updateTaskDto, userId);
  }

  @Patch(':id/status')
  updateStatus(
    @Param('id', ParseIntPipe) id: number,
    @Body('status') status: TaskStatus,
    @User('id') userId: number,
  ) {
    return this.tasksService.updateStatus(id, status, userId);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number, @User('id') userId: number) {
    this.tasksService.remove(id, userId);
  }
}
```

### Tasks to Complete
- [ ] Create task entity and DTOs
- [ ] Implement task service with filtering
- [ ] Build task controller
- [ ] Test CRUD operations
- [ ] Test filtering
- [ ] Test statistics endpoint

### Time Estimate
60-90 minutes

---

## Phase 5: Global Features

### Objective
Add interceptors, filters, and configuration.

### Instructions

#### Step 1: Create Shared Components
**Public Decorator (`src/shared/decorators/public.decorator.ts`):**
```typescript
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

**User Decorator (`src/shared/decorators/user.decorator.ts`):**
```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
```

**Transform Interceptor (`src/shared/interceptors/transform.interceptor.ts`):**
```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  success: boolean;
  data: T;
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
      map((data) => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
```

**Exception Filter (`src/shared/filters/http-exception.filter.ts`):**
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
      success: false,
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message,
    };

    console.error('Exception:', errorResponse);

    response.status(status).json(errorResponse);
  }
}
```

#### Step 2: Configure Main.ts
**`src/main.ts`:**
```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';
import { HttpExceptionFilter } from './shared/filters/http-exception.filter';
import { TransformInterceptor } from './shared/interceptors/transform.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Global prefix
  app.setGlobalPrefix('api');

  // CORS
  app.enableCors();

  // Validation
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  // Exception filter
  app.useGlobalFilters(new HttpExceptionFilter());

  // Transform interceptor
  app.useGlobalInterceptors(new TransformInterceptor());

  const port = process.env.PORT || 3000;
  await app.listen(port);

  console.log(`🚀 Application running on: http://localhost:${port}/api`);
}
bootstrap();
```

### Tasks to Complete
- [ ] Create all shared components
- [ ] Configure global pipes, filters, interceptors
- [ ] Test error handling
- [ ] Test response transformation

### Time Estimate
30-45 minutes

---

## Phase 6: Testing

### Objective
Write comprehensive tests for the application.

### Instructions

Write unit tests for services and E2E tests for API endpoints focusing on:
- Auth flow (register, login)
- Task CRUD operations
- Access control
- Validation
- Error handling

### Time Estimate
90-120 minutes

---

## Final Checklist

- [ ] User registration and authentication working
- [ ] JWT guards protecting routes
- [ ] Task CRUD operations complete
- [ ] Filtering and search working
- [ ] Validation on all inputs
- [ ] Exception handling globally
- [ ] Response transformation
- [ ] Unit tests passing
- [ ] E2E tests passing
- [ ] API documentation (README)

---

## API Testing Guide

### Test with cURL
```bash
# Register
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "email": "john@example.com",
    "password": "Password123",
    "firstName": "John",
    "lastName": "Doe"
  }'

# Login
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "Password123"
  }'

# Create Task (use token from login)
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "title": "Complete NestJS Course",
    "description": "Finish all modules and exercises",
    "priority": "high"
  }'

# Get all tasks
curl http://localhost:3000/api/tasks \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"

# Get task statistics
curl http://localhost:3000/api/tasks/statistics \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

---

## Bonus Challenges

1. **Add Comments to Tasks**
   - Create comments module
   - Allow users to comment on tasks
   - Nest routes: `/tasks/:id/comments`

2. **Task Assignments**
   - Allow multiple users per task
   - Track task assignees
   - Filter by assigned tasks

3. **Due Date Notifications**
   - Add service to check overdue tasks
   - Implement scheduled tasks with @nestjs/schedule

4. **File Attachments**
   - Allow file uploads on tasks
   - Use multer for file handling

5. **API Documentation with Swagger**
   - Add @nestjs/swagger
   - Document all endpoints

---

## Submission

Create a comprehensive README.md with:
- Project description
- Setup instructions
- API documentation
- Architecture overview
- Testing instructions
- Technologies used

---

## Resources

- [NestJS Documentation](https://docs.nestjs.com/)
- [JWT Authentication](https://docs.nestjs.com/security/authentication)
- [Validation](https://docs.nestjs.com/techniques/validation)
- [Testing](https://docs.nestjs.com/fundamentals/testing)

---

## Congratulations!

🎉 You've completed the NestJS course! You now have:
- Deep understanding of NestJS architecture
- Practical experience building APIs
- Knowledge of best practices
- A complete portfolio project

Keep building and exploring more advanced topics!

---

**[Back to Exercise Index](README.md)**
