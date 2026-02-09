# Module 3 Exercises: Controllers

## Overview
These exercises focus on building controllers, handling HTTP requests, routing, and working with different types of request data.

---

## Exercise 1: Build a Complete Users Controller

### Objective
Create a fully functional UsersController with all CRUD operations.

### Instructions

#### Step 1: Generate Controller
```bash
# Create new project or use existing
nest new users-api
cd users-api

# Generate controller
nest generate controller users --no-spec
```

#### Step 2: Create User Interface
**Create `src/users/interfaces/user.interface.ts`:**
```typescript
export interface User {
  id: number;
  username: string;
  email: string;
  firstName: string;
  lastName: string;
  age: number;
  createdAt: Date;
}
```

#### Step 3: Implement CRUD Routes
**Update `src/users/users.controller.ts`:**
```typescript
import { Controller, Get, Post, Put, Delete, Body, Param, Query, HttpCode, HttpStatus } from '@nestjs/common';
import { User } from './interfaces/user.interface';

@Controller('users')
export class UsersController {
  private users: User[] = [
    {
      id: 1,
      username: 'john_doe',
      email: 'john@example.com',
      firstName: 'John',
      lastName: 'Doe',
      age: 30,
      createdAt: new Date('2024-01-01'),
    },
    {
      id: 2,
      username: 'jane_smith',
      email: 'jane@example.com',
      firstName: 'Jane',
      lastName: 'Smith',
      age: 25,
      createdAt: new Date('2024-01-02'),
    },
  ];

  // GET /users - Get all users
  @Get()
  findAll(): User[] {
    return this.users;
  }

  // GET /users/:id - Get user by ID
  @Get(':id')
  findOne(@Param('id') id: string): User {
    const user = this.users.find(u => u.id === parseInt(id));
    if (!user) {
      throw new Error('User not found');
    }
    return user;
  }

  // POST /users - Create new user
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: Partial<User>): User {
    const newUser: User = {
      id: Math.max(...this.users.map(u => u.id), 0) + 1,
      username: createUserDto.username,
      email: createUserDto.email,
      firstName: createUserDto.firstName,
      lastName: createUserDto.lastName,
      age: createUserDto.age,
      createdAt: new Date(),
    };
    this.users.push(newUser);
    return newUser;
  }

  // PUT /users/:id - Update user
  @Put(':id')
  update(@Param('id') id: string, @Body() updateUserDto: Partial<User>): User {
    const index = this.users.findIndex(u => u.id === parseInt(id));
    if (index === -1) {
      throw new Error('User not found');
    }
    this.users[index] = { ...this.users[index], ...updateUserDto };
    return this.users[index];
  }

  // DELETE /users/:id - Delete user
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id') id: string): void {
    const index = this.users.findIndex(u => u.id === parseInt(id));
    if (index === -1) {
      throw new Error('User not found');
    }
    this.users.splice(index, 1);
  }
}
```

#### Step 4: Test All Endpoints

**Using curl:**
```bash
# Get all users
curl http://localhost:3000/users

# Get specific user
curl http://localhost:3000/users/1

# Create user
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice_wonder",
    "email": "alice@example.com",
    "firstName": "Alice",
    "lastName": "Wonder",
    "age": 28
  }'

# Update user
curl -X PUT http://localhost:3000/users/1 \
  -H "Content-Type: application/json" \
  -d '{
    "age": 31
  }'

# Delete user
curl -X DELETE http://localhost:3000/users/1
```

**Or create test file `test-api.http`:**
```http
### Get all users
GET http://localhost:3000/users

### Get user by ID
GET http://localhost:3000/users/1

### Create new user
POST http://localhost:3000/users
Content-Type: application/json

{
  "username": "bob_builder",
  "email": "bob@example.com",
  "firstName": "Bob",
  "lastName": "Builder",
  "age": 35
}

### Update user
PUT http://localhost:3000/users/1
Content-Type: application/json

{
  "age": 32
}

### Delete user
DELETE http://localhost:3000/users/3
```

### Tasks to Complete
- [ ] Generate users controller
- [ ] Implement all CRUD operations
- [ ] Test GET all users
- [ ] Test GET single user
- [ ] Test POST create user
- [ ] Test PUT update user
- [ ] Test DELETE user
- [ ] Handle not found cases

### Expected Outcome
- Fully functional CRUD controller
- Understanding of HTTP methods
- Experience with route parameters
- Practice with request bodies

### Time Estimate
45-60 minutes

---

## Exercise 2: Query Parameters and Filtering

### Objective
Implement filtering, pagination, and sorting using query parameters.

### Instructions

#### Step 1: Add Query Parameter Handling
**Update `findAll()` method:**
```typescript
import { Controller, Get, Query } from '@nestjs/common';

@Controller('users')
export class UsersController {
  // ... existing code ...

  @Get()
  findAll(
    @Query('search') search?: string,
    @Query('age') age?: string,
    @Query('sort') sort?: 'asc' | 'desc',
    @Query('page') page?: string,
    @Query('limit') limit?: string,
  ): { data: User[]; total: number; page: number; limit: number } {
    let filteredUsers = [...this.users];

    // Filter by search term (username or email)
    if (search) {
      filteredUsers = filteredUsers.filter(
        user =>
          user.username.toLowerCase().includes(search.toLowerCase()) ||
          user.email.toLowerCase().includes(search.toLowerCase()),
      );
    }

    // Filter by age
    if (age) {
      filteredUsers = filteredUsers.filter(user => user.age === parseInt(age));
    }

    // Sort by username
    if (sort) {
      filteredUsers.sort((a, b) => {
        if (sort === 'asc') {
          return a.username.localeCompare(b.username);
        }
        return b.username.localeCompare(a.username);
      });
    }

    // Pagination
    const pageNum = parseInt(page) || 1;
    const limitNum = parseInt(limit) || 10;
    const startIndex = (pageNum - 1) * limitNum;
    const endIndex = startIndex + limitNum;
    const paginatedUsers = filteredUsers.slice(startIndex, endIndex);

    return {
      data: paginatedUsers,
      total: filteredUsers.length,
      page: pageNum,
      limit: limitNum,
    };
  }
}
```

#### Step 2: Test Query Parameters
```bash
# Search by username
curl "http://localhost:3000/users?search=john"

# Filter by age
curl "http://localhost:3000/users?age=30"

# Sort ascending
curl "http://localhost:3000/users?sort=asc"

# Sort descending
curl "http://localhost:3000/users?sort=desc"

# Pagination
curl "http://localhost:3000/users?page=1&limit=5"

# Combine multiple filters
curl "http://localhost:3000/users?search=doe&sort=asc&page=1&limit=10"
```

#### Step 3: Create DTO for Query Parameters
**Create `src/users/dto/find-users-query.dto.ts`:**
```typescript
export class FindUsersQueryDto {
  search?: string;
  age?: number;
  sort?: 'asc' | 'desc';
  page?: number;
  limit?: number;
}
```

**Update controller:**
```typescript
@Get()
findAll(@Query() query: FindUsersQueryDto) {
  // Use query.search, query.age, etc.
}
```

### Tasks to Complete
- [ ] Implement search functionality
- [ ] Add age filtering
- [ ] Implement sorting
- [ ] Add pagination
- [ ] Create query DTO
- [ ] Test all combinations
- [ ] Handle edge cases (invalid page, negative limits)

### Expected Outcome
- Working search and filter
- Proper pagination
- Clean query handling
- Understanding of query parameters

### Time Estimate
45-60 minutes

---

## Exercise 3: Nested Routes and Sub-resources

### Objective
Implement nested routes for user posts (e.g., `/users/:userId/posts`).

### Instructions

#### Step 1: Create Post Interface
**Create `src/users/interfaces/post.interface.ts`:**
```typescript
export interface Post {
  id: number;
  userId: number;
  title: string;
  content: string;
  createdAt: Date;
}
```

#### Step 2: Add Posts Management
**Update `users.controller.ts`:**
```typescript
import { Post } from './interfaces/post.interface';

@Controller('users')
export class UsersController {
  private users: User[] = [/* ... */];
  private posts: Post[] = [
    {
      id: 1,
      userId: 1,
      title: 'First Post',
      content: 'This is my first post',
      createdAt: new Date('2024-01-15'),
    },
    {
      id: 2,
      userId: 1,
      title: 'Second Post',
      content: 'This is my second post',
      createdAt: new Date('2024-01-16'),
    },
    {
      id: 3,
      userId: 2,
      title: 'Jane\'s Post',
      content: 'Hello from Jane',
      createdAt: new Date('2024-01-17'),
    },
  ];

  // GET /users/:userId/posts - Get all posts by user
  @Get(':userId/posts')
  findUserPosts(@Param('userId') userId: string): Post[] {
    return this.posts.filter(post => post.userId === parseInt(userId));
  }

  // GET /users/:userId/posts/:postId - Get specific post
  @Get(':userId/posts/:postId')
  findUserPost(
    @Param('userId') userId: string,
    @Param('postId') postId: string,
  ): Post {
    const post = this.posts.find(
      p => p.userId === parseInt(userId) && p.id === parseInt(postId),
    );
    if (!post) {
      throw new Error('Post not found');
    }
    return post;
  }

  // POST /users/:userId/posts - Create post for user
  @Post(':userId/posts')
  @HttpCode(HttpStatus.CREATED)
  createUserPost(
    @Param('userId') userId: string,
    @Body() createPostDto: { title: string; content: string },
  ): Post {
    const newPost: Post = {
      id: Math.max(...this.posts.map(p => p.id), 0) + 1,
      userId: parseInt(userId),
      title: createPostDto.title,
      content: createPostDto.content,
      createdAt: new Date(),
    };
    this.posts.push(newPost);
    return newPost;
  }

  // DELETE /users/:userId/posts/:postId - Delete post
  @Delete(':userId/posts/:postId')
  @HttpCode(HttpStatus.NO_CONTENT)
  deleteUserPost(
    @Param('userId') userId: string,
    @Param('postId') postId: string,
  ): void {
    const index = this.posts.findIndex(
      p => p.userId === parseInt(userId) && p.id === parseInt(postId),
    );
    if (index === -1) {
      throw new Error('Post not found');
    }
    this.posts.splice(index, 1);
  }
}
```

#### Step 3: Test Nested Routes
```bash
# Get all posts by user 1
curl http://localhost:3000/users/1/posts

# Get specific post
curl http://localhost:3000/users/1/posts/1

# Create post for user
curl -X POST http://localhost:3000/users/1/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "New Post",
    "content": "This is a new post content"
  }'

# Delete post
curl -X DELETE http://localhost:3000/users/1/posts/1
```

### Tasks to Complete
- [ ] Create Post interface
- [ ] Implement GET user posts
- [ ] Implement GET single post
- [ ] Implement POST create post
- [ ] Implement DELETE post
- [ ] Test all nested routes
- [ ] Verify userId validation

### Expected Outcome
- Working nested routes
- Understanding of route parameters
- Practice with sub-resources

### Time Estimate
30-45 minutes

---

## Exercise 4: Request and Response Customization

### Objective
Learn to work with headers, status codes, and response transformation.

### Instructions

#### Step 1: Add Custom Headers
**Create `src/users/users.controller.ts` methods:**
```typescript
import { 
  Controller, 
  Get, 
  Post, 
  Header, 
  Headers, 
  Req, 
  Res,
  HttpStatus 
} from '@nestjs/common';
import { Request, Response } from 'express';

@Controller('users')
export class UsersController {
  // Custom response headers
  @Get('export')
  @Header('Content-Type', 'text/csv')
  @Header('Content-Disposition', 'attachment; filename="users.csv"')
  exportUsers(): string {
    const csv = this.users
      .map(u => `${u.id},${u.username},${u.email},${u.age}`)
      .join('\n');
    return `id,username,email,age\n${csv}`;
  }

  // Read request headers
  @Get('headers')
  getHeaders(@Headers() headers: Record<string, string>): any {
    return {
      userAgent: headers['user-agent'],
      host: headers['host'],
      contentType: headers['content-type'],
    };
  }

  // Custom status codes
  @Post('verify')
  @HttpCode(HttpStatus.ACCEPTED)
  verifyUser(@Body() data: { email: string }): any {
    return {
      message: 'Verification email sent',
      email: data.email,
      status: 'pending',
    };
  }

  // Using Response object directly
  @Get('custom-response')
  customResponse(@Res() res: Response): void {
    res
      .status(HttpStatus.OK)
      .header('X-Custom-Header', 'CustomValue')
      .json({
        message: 'Custom response',
        timestamp: new Date().toISOString(),
      });
  }

  // Using Request object
  @Get('request-info')
  getRequestInfo(@Req() req: Request): any {
    return {
      method: req.method,
      url: req.url,
      ip: req.ip,
      userAgent: req.get('user-agent'),
      query: req.query,
      params: req.params,
    };
  }
}
```

#### Step 2: Test Custom Responses
```bash
# Export CSV
curl http://localhost:3000/users/export

# Get headers info
curl http://localhost:3000/users/headers

# Verify user (202 Accepted)
curl -X POST http://localhost:3000/users/verify \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com"}'

# Custom response
curl -i http://localhost:3000/users/custom-response

# Request info
curl http://localhost:3000/users/request-info
```

### Tasks to Complete
- [ ] Implement CSV export with headers
- [ ] Read and return request headers
- [ ] Use custom status codes
- [ ] Work with Response object
- [ ] Work with Request object
- [ ] Test all endpoints

### Expected Outcome
- Understanding of HTTP headers
- Status code management
- Request/Response object usage

### Time Estimate
30-45 minutes

---

## Exercise 5: DTOs and Data Validation Setup

### Objective
Create Data Transfer Objects (DTOs) for type safety and prepare for validation.

### Instructions

#### Step 1: Install Validation Dependencies
```bash
npm install class-validator class-transformer
```

#### Step 2: Create DTOs
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
}
```

**Create `src/users/dto/create-post.dto.ts`:**
```typescript
export class CreatePostDto {
  title: string;
  content: string;
}
```

#### Step 3: Use DTOs in Controller
```typescript
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { CreatePostDto } from './dto/create-post.dto';

@Controller('users')
export class UsersController {
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto): User {
    const newUser: User = {
      id: Math.max(...this.users.map(u => u.id), 0) + 1,
      ...createUserDto,
      createdAt: new Date(),
    };
    this.users.push(newUser);
    return newUser;
  }

  @Put(':id')
  update(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): User {
    const index = this.users.findIndex(u => u.id === parseInt(id));
    if (index === -1) {
      throw new Error('User not found');
    }
    this.users[index] = { ...this.users[index], ...updateUserDto };
    return this.users[index];
  }

  @Post(':userId/posts')
  @HttpCode(HttpStatus.CREATED)
  createUserPost(
    @Param('userId') userId: string,
    @Body() createPostDto: CreatePostDto,
  ): Post {
    const newPost: Post = {
      id: Math.max(...this.posts.map(p => p.id), 0) + 1,
      userId: parseInt(userId),
      ...createPostDto,
      createdAt: new Date(),
    };
    this.posts.push(newPost);
    return newPost;
  }
}
```

#### Step 4: Add Validation Decorators
```typescript
import { IsString, IsEmail, IsInt, Min, Max, IsNotEmpty, MinLength, MaxLength } from 'class-validator';

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
  @IsNotEmpty()
  firstName: string;

  @IsString()
  @IsNotEmpty()
  lastName: string;

  @IsInt()
  @Min(1)
  @Max(150)
  age: number;
}
```

### Tasks to Complete
- [ ] Create all DTOs
- [ ] Add validation decorators
- [ ] Use DTOs in controller methods
- [ ] Test with valid data
- [ ] Prepare for validation (Module 7)

### Expected Outcome
- Type-safe request handling
- Clear data contracts
- Prepared validation decorators

### Time Estimate
30-45 minutes

---

## Challenge Exercise: Blog API with Comments

### Objective
Build a complete blog API with posts and nested comments.

### Requirements

1. **Routes:**
   - `GET /posts` - List all posts
   - `GET /posts/:id` - Get single post
   - `POST /posts` - Create post
   - `PUT /posts/:id` - Update post
   - `DELETE /posts/:id` - Delete post
   - `GET /posts/:postId/comments` - Get post comments
   - `POST /posts/:postId/comments` - Add comment
   - `DELETE /posts/:postId/comments/:commentId` - Delete comment

2. **Features:**
   - Query parameters for filtering posts (author, tag, date)
   - Pagination for posts
   - Nested comments support
   - Custom headers for rate limiting info
   - CSV export for posts

3. **Data Structures:**
   ```typescript
   interface Post {
     id: number;
     title: string;
     content: string;
     author: string;
     tags: string[];
     createdAt: Date;
     updatedAt: Date;
   }

   interface Comment {
     id: number;
     postId: number;
     author: string;
     content: string;
     createdAt: Date;
   }
   ```

### Tasks
- [ ] Design the API structure
- [ ] Create all interfaces and DTOs
- [ ] Implement all routes
- [ ] Add query parameter handling
- [ ] Test with real requests
- [ ] Document API with examples

### Time Estimate
120-180 minutes

---

## Submission Checklist

- [ ] Users CRUD controller completed
- [ ] Query parameters and filtering working
- [ ] Nested routes implemented
- [ ] Custom headers and status codes used
- [ ] DTOs created for all operations
- [ ] All endpoints tested
- [ ] Code is clean and well-organized

---

## Testing Guide

Create a `test-endpoints.sh` script:
```bash
#!/bin/bash

BASE_URL="http://localhost:3000"

echo "Testing Users API..."

# Test GET all
echo "\n1. GET /users"
curl -s $BASE_URL/users | jq

# Test GET one
echo "\n2. GET /users/1"
curl -s $BASE_URL/users/1 | jq

# Test POST
echo "\n3. POST /users"
curl -s -X POST $BASE_URL/users \
  -H "Content-Type: application/json" \
  -d '{"username":"test","email":"test@test.com","firstName":"Test","lastName":"User","age":25}' | jq

# Test PUT
echo "\n4. PUT /users/1"
curl -s -X PUT $BASE_URL/users/1 \
  -H "Content-Type: application/json" \
  -d '{"age":26}' | jq

# Test query parameters
echo "\n5. GET /users?search=john"
curl -s "$BASE_URL/users?search=john" | jq

echo "\nAll tests completed!"
```

---

## Additional Resources

- [NestJS Controllers Documentation](https://docs.nestjs.com/controllers)
- [HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [HTTP Status Codes](https://httpstatuses.com/)
- [REST API Best Practices](https://restfulapi.net/)

---

## Next Steps

Continue to [Module 4 Exercises](module-04-exercises.md) to learn about providers and dependency injection!
