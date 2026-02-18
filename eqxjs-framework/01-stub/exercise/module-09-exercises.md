# Module 11 Exercises: Logging System & Best Practices

## � Ecosystem Component

**Working with: `@eqxjs-logger`**

Part of the EQXJS Ecosystem for comprehensive logging infrastructure.

## �📚 Exercise Overview

These exercises focus on implementing effective logging throughout an EQXJS application, from basic setup to advanced patterns and production best practices.

### 🎯 Learning Objectives

- Configure LoggerService in modules
- Inject LoggerService into services and controllers
- Implement appropriate log levels
- Structure logging with context and metadata
- Handle sensitive data in logs
- Set up file-based logging
- Integrate external logging services

### ⏱️ Estimated Time: 2 hours

---

## 🏁 Exercise 11.1: Basic Logger Setup (Getting Started)

### Objective

Set up LoggerService and log basic application events.

### Instructions

1. **Verify LoggerService is available:**

   The LoggerService is automatically provided by `FrameworkModule`. No additional installation needed.

2. **Create a service with logging:**

   Create `src/app.service.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";
   import { LoggerService } from "@eqxjs-stub";

   @Injectable()
   export class AppService {
     constructor(private readonly logger: LoggerService) {}

     getHello(): string {
       this.logger.info("getHello() called");
       return "Hello from EQXJS Logger Module!";
     }

     getStatus(): object {
       this.logger.debug("Checking application status");

       const status = {
         timestamp: new Date().toISOString(),
         uptime: process.uptime(),
         memory: process.memoryUsage(),
       };

       this.logger.info("Application status retrieved", status);
       return status;
     }
   }
   ```

3. **Update app.module.ts:**

   ```typescript
   import { Module } from "@nestjs/common";
   import { FrameworkModule } from "@eqxjs-stub";
   import { AppService } from "./app.service";
   import { AppController } from "./app.controller";

   @Module({
     imports: [
       FrameworkModule.register({
         configPath: "config",
         zone: process.env.NODE_ENV || "development",
       }),
     ],
     controllers: [AppController],
     providers: [AppService],
   })
   export class AppModule {}
   ```

4. **Create app.controller.ts:**

   ```typescript
   import { Controller, Get } from "@nestjs/common";
   import { AppService } from "./app.service";

   @Controller()
   export class AppController {
     constructor(private readonly appService: AppService) {}

     @Get()
     getHello(): string {
       return this.appService.getHello();
     }

     @Get("status")
     getStatus(): object {
       return this.appService.getStatus();
     }
   }
   ```

5. **Update configuration:**

   Ensure `config/development.config.yaml` has:

   ```yaml
   log:
     level: "debug"
     format: "json"
   ```

6. **Run and verify:**

   ```bash
   npm run start:dev
   ```

   Make requests and observe logs:

   ```bash
   curl http://localhost:3000
   curl http://localhost:3000/status
   ```

### Expected Output

You should see JSON-formatted logs like:

```json
{
  "timestamp": "2024-02-18T10:30:45.123Z",
  "level": "info",
  "message": "getHello() called"
}
{
  "timestamp": "2024-02-18T10:30:46.456Z",
  "level": "info",
  "message": "Application status retrieved",
  "uptime": 5.234
}
```

---

## 🏁 Exercise 11.2: Structured Logging with Context

### Objective

Add contextual information to logs for better traceability.

### Instructions

1. **Create a user service with structured logging:**

   Create `src/users/users.service.ts`:

   ```typescript
   import { Injectable, BadRequestException } from "@nestjs/common";
   import { LoggerService } from "@eqxjs-stub";

   interface User {
     id: string;
     email: string;
     name: string;
     role: string;
   }

   @Injectable()
   export class UsersService {
     private users: Map<string, User> = new Map();

     constructor(private readonly logger: LoggerService) {
       this.logger.info("UsersService initialized");
     }

     create(email: string, name: string): User {
       const context = {
         action: "create",
         email,
         name,
         timestamp: new Date().toISOString(),
       };

       this.logger.debug("Creating new user", context);

       const id = `user-${Date.now()}`;
       const user: User = { id, email, name, role: "user" };

       this.users.set(id, user);

       this.logger.info("User created successfully", {
         ...context,
         userId: id,
         role: user.role,
       });

       return user;
     }

     findById(id: string): User {
       this.logger.debug("Searching for user", { userId: id });

       const user = this.users.get(id);

       if (!user) {
         this.logger.warn("User not found", { userId: id });
         throw new BadRequestException(`User ${id} not found`);
       }

       this.logger.info("User found", {
         userId: id,
         email: user.email,
         role: user.role,
       });

       return user;
     }

     update(id: string, updates: Partial<User>): User {
       const context = {
         action: "update",
         userId: id,
         fieldsUpdating: Object.keys(updates),
         timestamp: new Date().toISOString(),
       };

       this.logger.debug("Updating user", context);

       const user = this.users.get(id);
       if (!user) {
         this.logger.error("Cannot update: user not found", {
           ...context,
           error: "NOT_FOUND",
         });
         throw new BadRequestException(`User ${id} not found`);
       }

       const updated = { ...user, ...updates };
       this.users.set(id, updated);

       this.logger.info("User updated successfully", {
         ...context,
         updatedFields: Object.keys(updates),
       });

       return updated;
     }

     delete(id: string): void {
       this.logger.debug("Deleting user", { userId: id });

       if (!this.users.has(id)) {
         this.logger.error("Cannot delete: user not found", {
           userId: id,
           action: "delete",
         });
         throw new BadRequestException(`User ${id} not found`);
       }

       this.users.delete(id);

       this.logger.info("User deleted successfully", {
         userId: id,
         timestamp: new Date().toISOString(),
       });
     }
   }
   ```

2. **Create users controller:**

   Create `src/users/users.controller.ts`:

   ```typescript
   import {
     Controller,
     Get,
     Post,
     Patch,
     Delete,
     Body,
     Param,
   } from "@nestjs/common";
   import { UsersService } from "./users.service";

   @Controller("users")
   export class UsersController {
     constructor(private readonly usersService: UsersService) {}

     @Post()
     create(@Body() body: { email: string; name: string }) {
       return this.usersService.create(body.email, body.name);
     }

     @Get(":id")
     findById(@Param("id") id: string) {
       return this.usersService.findById(id);
     }

     @Patch(":id")
     update(@Param("id") id: string, @Body() updates: Partial<any>) {
       return this.usersService.update(id, updates);
     }

     @Delete(":id")
     delete(@Param("id") id: string) {
       this.usersService.delete(id);
       return { deleted: true };
     }
   }
   ```

3. **Create users module:**

   Create `src/users/users.module.ts`:

   ```typescript
   import { Module } from "@nestjs/common";
   import { UsersService } from "./users.service";
   import { UsersController } from "./users.controller";

   @Module({
     controllers: [UsersController],
     providers: [UsersService],
   })
   export class UsersModule {}
   ```

4. **Update app.module.ts:**

   ```typescript
   import { Module } from "@nestjs/common";
   import { FrameworkModule } from "@eqxjs-stub";
   import { UsersModule } from "./users/users.module";

   @Module({
     imports: [
       FrameworkModule.register({
         configPath: "config",
         zone: process.env.NODE_ENV || "development",
       }),
       UsersModule,
     ],
   })
   export class AppModule {}
   ```

5. **Test the application:**

   ```bash
   npm run start:dev

   # Create user
   curl -X POST http://localhost:3000/users \
     -H "Content-Type: application/json" \
     -d '{"email":"john@example.com","name":"John Doe"}'

   # Get user (use ID from previous response)
   curl http://localhost:3000/users/user-1708259000000

   # Update user
   curl -X PATCH http://localhost:3000/users/user-1708259000000 \
     -H "Content-Type: application/json" \
     -d '{"name":"Jane Doe"}'

   # Delete user
   curl -X DELETE http://localhost:3000/users/user-1708259000000
   ```

### Expected Logs

Logs should include contextual information:

```json
{
  "level": "info",
  "message": "User created successfully",
  "userId": "user-1708259000000",
  "email": "john@example.com",
  "name": "John Doe",
  "role": "user"
}
```

---

## 🏁 Exercise 11.3: Error Logging and Stack Traces

### Objective

Properly log errors with stack traces for debugging.

### Instructions

1. **Create a payment service with error handling:**

   Create `src/payments/payments.service.ts`:

   ```typescript
   import { Injectable, BadRequestException } from "@nestjs/common";
   import { LoggerService } from "@eqxjs-stub";

   interface Payment {
     id: string;
     amount: number;
     status: "pending" | "completed" | "failed";
     createdAt: Date;
   }

   @Injectable()
   export class PaymentsService {
     private payments: Map<string, Payment> = new Map();

     constructor(private readonly logger: LoggerService) {
       this.logger.info("PaymentsService initialized");
     }

     async processPayment(orderId: string, amount: number): Promise<Payment> {
       const context = {
         action: "processPayment",
         orderId,
         amount,
         timestamp: new Date().toISOString(),
       };

       this.logger.debug("Payment processing started", context);

       try {
         // Simulate validation
         if (amount <= 0) {
           throw new Error("Amount must be greater than 0");
         }

         if (amount > 100000) {
           throw new Error("Amount exceeds maximum limit");
         }

         // Simulate processing
         await this.simulatePaymentGateway(amount);

         const payment: Payment = {
           id: `pay-${Date.now()}`,
           amount,
           status: "completed",
           createdAt: new Date(),
         };

         this.payments.set(payment.id, payment);

         this.logger.info("Payment processed successfully", {
           ...context,
           paymentId: payment.id,
           status: "completed",
         });

         return payment;
       } catch (error) {
         this.logger.error("Payment processing failed", {
           ...context,
           errorMessage: error.message,
           errorCode: error.code,
           stack: error.stack,
         });

         throw new BadRequestException(`Payment failed: ${error.message}`);
       }
     }

     async refundPayment(paymentId: string): Promise<void> {
       const context = {
         action: "refund",
         paymentId,
       };

       this.logger.debug("Refund processing started", context);

       try {
         const payment = this.payments.get(paymentId);

         if (!payment) {
           throw new Error(`Payment ${paymentId} not found`);
         }

         if (payment.status !== "completed") {
           throw new Error(
             `Cannot refund payment with status: ${payment.status}`,
           );
         }

         // Simulate refund
         payment.status = "failed";

         this.logger.info("Payment refunded successfully", {
           ...context,
           amount: payment.amount,
           originalStatus: "completed",
         });
       } catch (error) {
         this.logger.error("Refund failed", {
           ...context,
           errorMessage: error.message,
           stack: error.stack,
         });

         throw new BadRequestException(`Refund failed: ${error.message}`);
       }
     }

     private async simulatePaymentGateway(amount: number): Promise<void> {
       return new Promise((resolve) => {
         setTimeout(() => resolve(), 100);
       });
     }
   }
   ```

2. **Create payment controller:**

   Create `src/payments/payments.controller.ts`:

   ```typescript
   import { Controller, Post, Body, Param } from "@nestjs/common";
   import { PaymentsService } from "./payments.service";

   @Controller("payments")
   export class PaymentsController {
     constructor(private readonly paymentsService: PaymentsService) {}

     @Post("process")
     async process(@Body() body: { orderId: string; amount: number }) {
       return this.paymentsService.processPayment(body.orderId, body.amount);
     }

     @Post("refund/:id")
     async refund(@Param("id") paymentId: string) {
       await this.paymentsService.refundPayment(paymentId);
       return { refunded: true };
     }
   }
   ```

3. **Create payments module and add to app.module:**

   ```typescript
   import { Module } from "@nestjs/common";
   import { PaymentsService } from "./payments.service";
   import { PaymentsController } from "./payments.controller";

   @Module({
     controllers: [PaymentsController],
     providers: [PaymentsService],
   })
   export class PaymentsModule {}
   ```

4. **Test error logging:**

   ```bash
   # Process valid payment
   curl -X POST http://localhost:3000/payments/process \
     -H "Content-Type: application/json" \
     -d '{"orderId":"order-123","amount":50}'

   # Trigger validation error (amount too high)
   curl -X POST http://localhost:3000/payments/process \
     -H "Content-Type: application/json" \
     -d '{"orderId":"order-124","amount":150000}'

   # Trigger validation error (invalid amount)
   curl -X POST http://localhost:3000/payments/process \
     -H "Content-Type: application/json" \
     -d '{"orderId":"order-125","amount":-10}'
   ```

### Expected Logs

Error logs should include stack traces:

```json
{
  "level": "error",
  "message": "Payment processing failed",
  "orderId": "order-124",
  "amount": 150000,
  "errorMessage": "Amount exceeds maximum limit",
  "stack": "Error: Amount exceeds maximum limit\n  at PaymentsService.processPayment ..."
}
```

---

## 🏁 Exercise 11.4: Log Level Configuration

### Objective

Configure different log levels for development and production.

### Instructions

1. **Create environment-specific configurations:**

   Create `config/development.config.yaml`:

   ```yaml
   app:
     component-name: "logger-exercise-dev"

   log:
     level: "debug" # Verbose in development
     format: "json"
     include-timestamp: true
     include-context: true
   ```

   Create `config/production.config.yaml`:

   ```yaml
   app:
     component-name: "logger-exercise-prod"

   log:
     level: "warn" # Less verbose in production
     format: "json"
     include-timestamp: true
     include-context: true
   ```

2. **Run with different environments:**

   ```bash
   # Development - see all debug logs
   NODE_ENV=development npm run start:dev

   # Production simulation - only warn and error
   NODE_ENV=production npm run start:dev
   ```

3. **Observe the difference:**

   Make the same requests in both environments and note that:
   - Development shows debug logs
   - Production shows only warn and error logs

### Expected Behavior

- `development`: All logs (debug, info, warn, error) are visible
- `production`: Only warn and error logs are visible

---

## 🏁 Exercise 11.5: Sensitive Data Masking (Challenge)

### Objective

Implement logging without leaking sensitive data.

### Instructions

1. **Create a utility for masking sensitive data:**

   Create `src/utils/log-mask.util.ts`:

   ```typescript
   export class LogMask {
     /**
      * Mask email addresses (keep first 2 chars)
      */
     static maskEmail(email: string): string {
       if (!email || !email.includes("@")) return "***@***";
       const [local, domain] = email.split("@");
       return `${local.substring(0, 2)}***@${domain}`;
     }

     /**
      * Mask full names (keep initials)
      */
     static maskName(name: string): string {
       if (!name) return "***";
       const parts = name.split(" ");
       return parts.map((p) => p[0] + "***").join(" ");
     }

     /**
      * Mask credit card numbers
      */
     static maskCCNumber(cc: string): string {
       if (!cc || cc.length < 4) return "****";
       const last4 = cc.slice(-4);
       return `****-****-****-${last4}`;
     }

     /**
      * Mask SSN/Tax ID
      */
     static maskSSN(ssn: string): string {
       if (!ssn || ssn.length < 4) return "***-**-****";
       const last4 = ssn.slice(-4);
       return `***-**-${last4}`;
     }

     /**
      * Mask API keys and tokens
      */
     static maskToken(token: string): string {
       if (!token || token.length < 4) return "***";
       const first = token.substring(0, 4);
       const last = token.substring(token.length - 4);
       return `${first}...${last}`;
     }
   }
   ```

2. **Update users service to use masking:**

   Update the logging in `src/users/users.service.ts`:

   ```typescript
   import { LogMask } from "../utils/log-mask.util";

   create(email: string, name: string): User {
     const context = {
       action: "create",
       email: LogMask.maskEmail(email),  // Masked
       name: LogMask.maskName(name),      // Masked
       timestamp: new Date().toISOString(),
     };

     this.logger.debug("Creating new user", context);

     const id = `user-${Date.now()}`;
     const user: User = { id, email, name, role: "user" };

     this.users.set(id, user);

     this.logger.info("User created successfully", {
       ...context,
       userId: id,
       role: user.role,
     });

     return user;
   }
   ```

3. **Verify masking works:**

   ```bash
   curl -X POST http://localhost:3000/users \
     -H "Content-Type: application/json" \
     -d '{"email":"john.doe@example.com","name":"John Doe"}'
   ```

   Logs should show:

   ```json
   {
     "level": "info",
     "message": "User created successfully",
     "email": "jo***@example.com",
     "name": "J*** D***",
     "userId": "user-1708259000000"
   }
   ```

---

## ✅ Validation Checklist

After completing all exercises, verify:

- [ ] LoggerService is injected in services and controllers
- [ ] Multiple log levels are used appropriately (debug, info, warn, error)
- [ ] Structured logging with context is implemented
- [ ] Error logs include stack traces
- [ ] Configuration allows environment-specific log levels
- [ ] Sensitive data is masked in logs
- [ ] Logs are readable and useful for debugging

---

## 📚 Next Steps

1. Integrate logging into your existing EQXJS application
2. Set up file-based log output in production
3. Configure integration with external logging services (Datadog, CloudWatch, etc.)
4. Implement log analysis and alerting
5. Review module-10 (Advanced Patterns) for combining logging with other features
