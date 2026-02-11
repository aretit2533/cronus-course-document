# Module 4 Exercises: Health Checks & Monitoring

## 📚 Exercise Overview

These exercises focus on implementing comprehensive health monitoring systems using EQXJS Framework's built-in health check capabilities.

### 🎯 Learning Objectives

- Implement built-in health indicators
- Create custom health indicators
- Set up health monitoring endpoints
- Integrate with monitoring systems
- Apply health check best practices

### ⏱️ Estimated Time: 2.5 hours

---

## 🏁 Exercise 4.1: Built-in Health Indicators (Quick Start)

### Objective

Set up and configure EQXJS built-in health indicators.

### Instructions

1. **Install Health Check Dependencies:**

   ```bash
   npm install @nestjs/terminus
   ```

2. **Create Health Module:**

   Create `src/health/health.module.ts`:

   ```typescript
   import { Module } from "@nestjs/common";
   import { TerminusModule } from "@nestjs/terminus";
   import { HealthController } from "./health.controller";
   import { SelfIndicator } from "./indicators/self.indicator";

   @Module({
     imports: [TerminusModule],
     controllers: [HealthController],
     providers: [SelfIndicator],
   })
   export class HealthModule {}
   ```

3. **Implement Self Health Indicator:**

   Create `src/health/indicators/self.indicator.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";
   import {
     HealthIndicator,
     HealthIndicatorResult,
     HealthCheckError,
   } from "@nestjs/terminus";

   @Injectable()
   export class SelfIndicator extends HealthIndicator {
     check(key: string): HealthIndicatorResult {
       const isHealthy = true; // Application is running

       const result = this.getStatus(key, isHealthy, {
         uptime: process.uptime(),
         memory: process.memoryUsage(),
         pid: process.pid,
         platform: process.platform,
         nodeVersion: process.version,
         timestamp: new Date().toISOString(),
       });

       if (isHealthy) {
         return result;
       }

       throw new HealthCheckError("Self check failed", result);
     }
   }
   ```

4. **Create Basic Health Controller:**

   Create `src/health/health.controller.ts`:

   ```typescript
   import { Controller, Get } from "@nestjs/common";
   import { HealthCheck, HealthCheckService } from "@nestjs/terminus";
   import { SelfIndicator } from "./indicators/self.indicator";

   @Controller("health")
   export class HealthController {
     constructor(
       private readonly healthCheckService: HealthCheckService,
       private readonly selfIndicator: SelfIndicator,
     ) {}

     @Get()
     @HealthCheck()
     check() {
       return this.healthCheckService.check([
         () => this.selfIndicator.check("self"),
       ]);
     }

     @Get("live")
     @HealthCheck()
     checkLiveness() {
       return this.healthCheckService.check([
         () => this.selfIndicator.check("self"),
       ]);
     }
   }
   ```

5. **Update App Module:**

   ```typescript
   import { Module } from "@nestjs/common";
   import { FrameworkModule } from "@corp-ais/eqxjs-stub";
   import { HealthModule } from "./health/health.module";
   import { AppController } from "./app.controller";

   @Module({
     imports: [
       FrameworkModule.register({
         configPath: "./config",
         zone: process.env.NODE_ENV || "development",
         enableHealthCheck: true,
       }),
       HealthModule,
     ],
     controllers: [AppController],
   })
   export class AppModule {}
   ```

### 📝 Tasks

- [ ] Install and configure health check dependencies
- [ ] Implement self health indicator
- [ ] Create health controller with basic endpoints
- [ ] Test health endpoints and verify responses

---

## 🔧 Exercise 4.2: Custom Health Indicators (Hands-On)

### Objective

Create custom health indicators for business-specific health checks.

### Instructions

1. **Create Database Health Indicator:**

   Create `src/health/indicators/database.indicator.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";
   import {
     HealthIndicator,
     HealthIndicatorResult,
     HealthCheckError,
   } from "@nestjs/terminus";

   @Injectable()
   export class DatabaseIndicator extends HealthIndicator {
     async check(key: string): Promise<HealthIndicatorResult> {
       try {
         const startTime = Date.now();

         // Simulate database connection check
         await this.performDatabasePing();

         const responseTime = Date.now() - startTime;

         const result = this.getStatus(key, true, {
           status: "connected",
           responseTime,
           host: "localhost",
           port: 27017,
           database: "demo_db",
           message: "Database is healthy",
         });

         return result;
       } catch (error) {
         const result = this.getStatus(key, false, {
           status: "disconnected",
           error: error.message,
           message: "Database connection failed",
         });

         throw new HealthCheckError("Database Health Check failed", result);
       }
     }

     private async performDatabasePing(): Promise<void> {
       // Simulate async database ping
       return new Promise((resolve, reject) => {
         setTimeout(() => {
           // 90% success rate for demonstration
           if (Math.random() > 0.1) {
             resolve();
           } else {
             reject(new Error("Connection timeout"));
           }
         }, Math.random() * 100);
       });
     }
   }
   ```

2. **Create External Service Health Indicator:**

   Create `src/health/indicators/external-service.indicator.ts`:

   ```typescript
   import { Injectable, HttpService } from "@nestjs/common";
   import {
     HealthIndicator,
     HealthIndicatorResult,
     HealthCheckError,
   } from "@nestjs/terminus";

   @Injectable()
   export class ExternalServiceIndicator extends HealthIndicator {
     constructor(private readonly httpService: HttpService) {
       super();
     }

     async check(key: string, url: string): Promise<HealthIndicatorResult> {
       try {
         const startTime = Date.now();

         const response = await this.httpService
           .get(url, {
             timeout: 5000,
             headers: {
               "User-Agent": "EQXJS-Health-Check/1.0",
             },
           })
           .toPromise();

         const responseTime = Date.now() - startTime;

         const result = this.getStatus(key, true, {
           url,
           statusCode: response.status,
           responseTime,
           message: "External service is reachable",
         });

         return result;
       } catch (error) {
         const result = this.getStatus(key, false, {
           url,
           error: error.message,
           statusCode: error.response?.status,
           message: "External service is unreachable",
         });

         throw new HealthCheckError(
           "External Service Health Check failed",
           result,
         );
       }
     }
   }
   ```

3. **Create Business Logic Health Indicator:**

   Create `src/health/indicators/business-logic.indicator.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";
   import {
     HealthIndicator,
     HealthIndicatorResult,
     HealthCheckError,
   } from "@nestjs/terminus";

   @Injectable()
   export class BusinessLogicIndicator extends HealthIndicator {
     async check(key: string): Promise<HealthIndicatorResult> {
       try {
         // Check various business logic health metrics
         const metrics = await this.gatherBusinessMetrics();

         const isHealthy = this.evaluateBusinessHealth(metrics);

         const result = this.getStatus(key, isHealthy, {
           ...metrics,
           healthScore: this.calculateHealthScore(metrics),
           message: isHealthy
             ? "Business logic is healthy"
             : "Business logic has issues",
         });

         if (isHealthy) {
           return result;
         } else {
           throw new HealthCheckError(
             "Business Logic Health Check failed",
             result,
           );
         }
       } catch (error) {
         const result = this.getStatus(key, false, {
           error: error.message,
           message: "Business logic health check failed",
         });

         throw new HealthCheckError(
           "Business Logic Health Check failed",
           result,
         );
       }
     }

     private async gatherBusinessMetrics(): Promise<any> {
       // Simulate gathering business metrics
       await new Promise((resolve) => setTimeout(resolve, 50));

       return {
         queueLength: Math.floor(Math.random() * 100),
         processingRate: Math.floor(Math.random() * 1000) + 500,
         errorRate: Math.random() * 0.1,
         activeUsers: Math.floor(Math.random() * 10000) + 1000,
         systemLoad: Math.random(),
       };
     }

     private evaluateBusinessHealth(metrics: any): boolean {
       return (
         metrics.queueLength < 50 &&
         metrics.errorRate < 0.05 &&
         metrics.systemLoad < 0.8
       );
     }

     private calculateHealthScore(metrics: any): number {
       let score = 100;

       if (metrics.queueLength > 25) score -= 10;
       if (metrics.errorRate > 0.02) score -= 20;
       if (metrics.systemLoad > 0.6) score -= 15;

       return Math.max(0, score);
     }
   }
   ```

4. **Update Health Module:**

   ```typescript
   import { Module, HttpModule } from "@nestjs/common";
   import { TerminusModule } from "@nestjs/terminus";
   import { HealthController } from "./health.controller";
   import { SelfIndicator } from "./indicators/self.indicator";
   import { DatabaseIndicator } from "./indicators/database.indicator";
   import { ExternalServiceIndicator } from "./indicators/external-service.indicator";
   import { BusinessLogicIndicator } from "./indicators/business-logic.indicator";

   @Module({
     imports: [TerminusModule, HttpModule],
     controllers: [HealthController],
     providers: [
       SelfIndicator,
       DatabaseIndicator,
       ExternalServiceIndicator,
       BusinessLogicIndicator,
     ],
   })
   export class HealthModule {}
   ```

### 📝 Tasks

- [ ] Create database health indicator with connection testing
- [ ] Implement external service health indicator
- [ ] Build business logic health indicator with metrics
- [ ] Test all custom indicators

---

## 🚀 Exercise 4.3: Monitoring Integration (Challenge)

### Objective

Integrate health checks with monitoring systems and implement alerting.

### Instructions

1. **Create Health Metrics Service:**

   Create `src/health/health-metrics.service.ts`:

   ```typescript
   import { Injectable, Logger } from "@nestjs/common";
   import { HealthCheckResult } from "@nestjs/terminus";

   @Injectable()
   export class HealthMetricsService {
     private readonly logger = new Logger(HealthMetricsService.name);
     private healthHistory: HealthCheckResult[] = [];
     private readonly maxHistorySize = 100;

     recordHealthCheck(result: HealthCheckResult, endpoint: string): void {
       // Record health check result
       this.healthHistory.push({
         ...result,
         timestamp: new Date().toISOString(),
         endpoint,
       } as any);

       // Maintain history size
       if (this.healthHistory.length > this.maxHistorySize) {
         this.healthHistory.shift();
       }

       // Log health check
       this.logHealthCheck(result, endpoint);

       // Generate alerts if needed
       this.checkForAlerts(result, endpoint);
     }

     getHealthMetrics(): any {
       const recent = this.healthHistory.slice(-10);

       return {
         totalChecks: this.healthHistory.length,
         recentChecks: recent.length,
         successRate: this.calculateSuccessRate(),
         failureRate: this.calculateFailureRate(),
         averageResponseTime: this.calculateAverageResponseTime(),
         recentResults: recent,
         alerts: this.getActiveAlerts(),
       };
     }

     private logHealthCheck(result: HealthCheckResult, endpoint: string): void {
       const logData = {
         endpoint,
         status: result.status,
         timestamp: new Date().toISOString(),
         successfulChecks: Object.keys(result.info || {}),
         failedChecks: Object.keys(result.error || {}),
       };

       if (result.status === "ok") {
         this.logger.log("Health check passed", logData);
       } else {
         this.logger.error("Health check failed", logData);
       }
     }

     private checkForAlerts(result: HealthCheckResult, endpoint: string): void {
       if (result.status !== "ok") {
         this.generateAlert("HEALTH_CHECK_FAILED", {
           endpoint,
           failedChecks: Object.keys(result.error || {}),
           timestamp: new Date().toISOString(),
         });
       }

       // Check for performance degradation
       const recentFailures = this.getRecentFailures(5);
       if (recentFailures.length >= 3) {
         this.generateAlert("PERFORMANCE_DEGRADATION", {
           endpoint,
           recentFailures: recentFailures.length,
           timestamp: new Date().toISOString(),
         });
       }
     }

     private generateAlert(type: string, data: any): void {
       this.logger.warn(`ALERT: ${type}`, data);

       // Here you could integrate with external alerting systems
       // this.notificationService.sendAlert(type, data);
       // this.slackService.postMessage(type, data);
       // this.emailService.sendAlert(type, data);
     }

     private calculateSuccessRate(): number {
       if (this.healthHistory.length === 0) return 100;

       const successful = this.healthHistory.filter(
         (h) => h.status === "ok",
       ).length;
       return (successful / this.healthHistory.length) * 100;
     }

     private calculateFailureRate(): number {
       return 100 - this.calculateSuccessRate();
     }

     private calculateAverageResponseTime(): number {
       const responseTimes = this.healthHistory
         .map((h) => (h as any).responseTime)
         .filter((rt) => typeof rt === "number");

       if (responseTimes.length === 0) return 0;

       return (
         responseTimes.reduce((sum, rt) => sum + rt, 0) / responseTimes.length
       );
     }

     private getRecentFailures(count: number): HealthCheckResult[] {
       return this.healthHistory.slice(-count).filter((h) => h.status !== "ok");
     }

     private getActiveAlerts(): any[] {
       // In a real implementation, you'd track active alerts
       return [];
     }
   }
   ```

2. **Create Enhanced Health Controller:**

   Update `src/health/health.controller.ts`:

   ```typescript
   import { Controller, Get, Query } from "@nestjs/common";
   import {
     HealthCheck,
     HealthCheckService,
     HealthCheckResult,
   } from "@nestjs/terminus";
   import { SelfIndicator } from "./indicators/self.indicator";
   import { DatabaseIndicator } from "./indicators/database.indicator";
   import { ExternalServiceIndicator } from "./indicators/external-service.indicator";
   import { BusinessLogicIndicator } from "./indicators/business-logic.indicator";
   import { HealthMetricsService } from "./health-metrics.service";

   @Controller("health")
   export class HealthController {
     constructor(
       private readonly healthCheckService: HealthCheckService,
       private readonly selfIndicator: SelfIndicator,
       private readonly databaseIndicator: DatabaseIndicator,
       private readonly externalServiceIndicator: ExternalServiceIndicator,
       private readonly businessLogicIndicator: BusinessLogicIndicator,
       private readonly healthMetricsService: HealthMetricsService,
     ) {}

     @Get()
     @HealthCheck()
     async check() {
       const result = await this.healthCheckService.check([
         () => this.selfIndicator.check("self"),
         () => this.databaseIndicator.check("database"),
         () => this.businessLogicIndicator.check("business-logic"),
       ]);

       this.healthMetricsService.recordHealthCheck(result, "/health");
       return result;
     }

     @Get("live")
     @HealthCheck()
     async checkLiveness() {
       const result = await this.healthCheckService.check([
         () => this.selfIndicator.check("self"),
       ]);

       this.healthMetricsService.recordHealthCheck(result, "/health/live");
       return result;
     }

     @Get("ready")
     @HealthCheck()
     async checkReadiness() {
       const result = await this.healthCheckService.check([
         () => this.selfIndicator.check("self"),
         () => this.databaseIndicator.check("database"),
       ]);

       this.healthMetricsService.recordHealthCheck(result, "/health/ready");
       return result;
     }

     @Get("detailed")
     @HealthCheck()
     async checkDetailed(@Query("external_url") externalUrl?: string) {
       const checks = [
         () => this.selfIndicator.check("self"),
         () => this.databaseIndicator.check("database"),
         () => this.businessLogicIndicator.check("business-logic"),
       ];

       if (externalUrl) {
         checks.push(() =>
           this.externalServiceIndicator.check("external-service", externalUrl),
         );
       }

       const result = await this.healthCheckService.check(checks);

       this.healthMetricsService.recordHealthCheck(result, "/health/detailed");
       return result;
     }

     @Get("metrics")
     getHealthMetrics() {
       return this.healthMetricsService.getHealthMetrics();
     }
   }
   ```

3. **Create Health Dashboard Data:**

   Create `src/health/health-dashboard.service.ts`:

   ```typescript
   import { Injectable } from "@nestjs/common";
   import { HealthMetricsService } from "./health-metrics.service";

   @Injectable()
   export class HealthDashboardService {
     constructor(private readonly healthMetricsService: HealthMetricsService) {}

     getDashboardData(): any {
       const metrics = this.healthMetricsService.getHealthMetrics();

       return {
         overview: {
           status: this.getOverallStatus(metrics),
           uptime: process.uptime(),
           version: process.env.npm_package_version || "1.0.0",
           environment: process.env.NODE_ENV || "development",
         },
         metrics: {
           totalChecks: metrics.totalChecks,
           successRate: metrics.successRate,
           failureRate: metrics.failureRate,
           averageResponseTime: metrics.averageResponseTime,
         },
         system: {
           memory: process.memoryUsage(),
           platform: process.platform,
           nodeVersion: process.version,
           pid: process.pid,
         },
         recent: metrics.recentResults.slice(-5),
         alerts: metrics.alerts,
       };
     }

     private getOverallStatus(metrics: any): string {
       if (metrics.successRate >= 95) return "healthy";
       if (metrics.successRate >= 80) return "warning";
       return "critical";
     }
   }
   ```

### 📝 Tasks

- [ ] Implement health metrics tracking service
- [ ] Create enhanced health controller with all endpoints
- [ ] Build health dashboard service
- [ ] Test monitoring integration
- [ ] Verify alert generation for failures

---

## 🎯 Exercise Completion Checklist

### Exercise 4.1: Built-in Health Indicators

- [ ] Configured health check dependencies
- [ ] Implemented self health indicator
- [ ] Created basic health endpoints
- [ ] Tested health check responses

### Exercise 4.2: Custom Health Indicators

- [ ] Built database health indicator
- [ ] Created external service health indicator
- [ ] Implemented business logic health indicator
- [ ] Tested all custom indicators

### Exercise 4.3: Monitoring Integration

- [ ] Implemented health metrics tracking
- [ ] Enhanced health controller with monitoring
- [ ] Created health dashboard service
- [ ] Verified monitoring and alerting

## 📚 Learning Reflection

After completing these exercises:

1. **How important** are health checks in production systems?
2. **What custom indicators** would be useful for your applications?
3. **How would you integrate** health checks with your monitoring stack?
4. **What alerting strategies** work best for different failure types?

## ➡️ Next Steps

Great job on implementing comprehensive health monitoring!

👉 **Continue to** [Module 5 Exercises](module-05-exercises.md)

---

**Excellent work on mastering health checks and monitoring! 🎉**
