# @eqxjs/pipes

A collection of useful NestJS transformation pipes for data processing and validation.

## Installation

```bash
npm install @eqxjs/pipes
```

## Description

This library provides transformation pipes specifically designed for NestJS applications. It includes pipes for data transformation, formatting, and validation that can be used in controllers, route handlers, and other parts of your NestJS application.

## Usage

### Basic Setup

```typescript
import { Module } from '@nestjs/common';
import { RemoveAtSymbolPipe, PlainToClassExcludePipe } from '@eqxjs/pipes';

@Module({
  providers: [RemoveAtSymbolPipe, PlainToClassExcludePipe],
  exports: [RemoveAtSymbolPipe, PlainToClassExcludePipe]
})
export class PipesModule {}
```

### Using Pipes in Controllers

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { RemoveAtSymbolPipe, PlainToClassExcludePipe } from '@eqxjs/pipes';

@Controller('users')
export class UsersController {
  @Post()
  createUser(
    @Body(new PlainToClassExcludePipe()) userData: CreateUserDto,
    @Body('email', new RemoveAtSymbolPipe()) email: string
  ) {
    // userData will be transformed to CreateUserDto instance, excluding extraneous values
    // email will have '@' symbols removed
    return this.usersService.create(userData);
  }
}
```

## API Reference

### Pipes

#### RemoveAtSymbolPipe

A pipe that removes the '@' symbol from the given value.

**Implementation:** `PipeTransform`

**Methods:**
- `transform(value: any, metadata: ArgumentMetadata)` - Transforms the input value into an instance of the specified metadata type

**Usage:**
```typescript
// In a controller
@Post()
createUser(@Body('username', new RemoveAtSymbolPipe()) username: string) {
  // username will have '@' symbols removed
}

// Example: "@john.doe" becomes "john.doe"
```

**Parameters:**
- `value: any` - The value to be transformed
- `metadata: ArgumentMetadata` - The metadata containing the target metatype

**Returns:**
- Transformed value as an instance of the specified metatype

#### PlainToClassExcludePipe

A pipe that transforms a plain object to an instance of a class, excluding extraneous values.

This pipe uses the `plainToClass` function from the `class-transformer` library to convert a plain JavaScript object into an instance of the specified class type, while excluding any properties that are not defined in the class.

**Implementation:** `PipeTransform`

**Methods:**
- `transform(value: any, metadata: ArgumentMetadata)` - Transforms the input plain object to an instance of the specified class, excluding extraneous values

**Usage:**
```typescript
// Define a DTO with class-transformer decorators
export class CreateUserDto {
  @Expose()
  name: string;

  @Expose()
  email: string;

  // age property will be excluded if not decorated with @Expose()
}

// In a controller
@Post()
createUser(@Body(new PlainToClassExcludePipe()) userData: CreateUserDto) {
  // userData will be transformed to CreateUserDto instance
  // Only properties decorated with @Expose() will be included
  // Extraneous properties in the request body will be excluded
}
```

**Parameters:**
- `value: any` - The plain object to be transformed
- `metadata: ArgumentMetadata` - The metadata containing the target class type

**Returns:**
- The transformed instance of the specified class with extraneous values excluded

**Features:**
- Uses `class-transformer` with `excludeExtraneousValues: true` option
- Automatically filters out properties not defined in the target class
- Helps with data validation and security by preventing unwanted properties
- Works seamlessly with class-transformer decorators like `@Expose()`

## Advanced Usage

### Combining Pipes

```typescript
@Controller('api')
export class ApiController {
  @Post('users')
  createUser(
    @Body(new ValidationPipe(), new PlainToClassExcludePipe()) 
    userData: CreateUserDto
  ) {
    // First validates, then transforms with excluded extraneous values
  }

  @Get('user/:email')
  getUserByEmail(
    @Param('email', new RemoveAtSymbolPipe()) 
    email: string
  ) {
    // Removes @ symbols from email parameter
  }
}
```

### Custom Pipe Configuration

```typescript
// You can extend the pipes for custom behavior
@Injectable()
export class CustomRemoveAtSymbolPipe extends RemoveAtSymbolPipe {
  transform(value: any, metadata: ArgumentMetadata) {
    const result = super.transform(value, metadata);
    // Add custom logic here
    return result;
  }
}
```

## Dependencies

This library depends on:
- `@nestjs/common` - For NestJS pipe interfaces and decorators
- `class-transformer` - For object transformation capabilities

## TypeScript Support

This library is written in TypeScript and includes full type definitions. All pipes are properly typed and provide IntelliSense support in compatible IDEs.

## Examples

### Example 1: Data Sanitization

```typescript
@Controller('auth')
export class AuthController {
  @Post('login')
  async login(
    @Body('username', new RemoveAtSymbolPipe()) username: string,
    @Body(new PlainToClassExcludePipe()) loginData: LoginDto
  ) {
    // username: "@user123" -> "user123"
    // loginData: only properties defined in LoginDto are included
    return this.authService.login(username, loginData.password);
  }
}
```

### Example 2: Secure Data Transfer

```typescript
export class UpdateProfileDto {
  @Expose()
  @IsString()
  name: string;

  @Expose()
  @IsEmail()
  email: string;

  // Internal properties like 'id', 'createdAt' won't be accepted
  // even if sent in the request body
}

@Controller('profile')
export class ProfileController {
  @Put('update')
  updateProfile(
    @Body(new PlainToClassExcludePipe()) profileData: UpdateProfileDto
  ) {
    // Only name and email will be processed
    // Any other properties in the request body will be ignored
  }
}
```

## License

ISC