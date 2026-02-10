# Module 5: Data Processing and Pipes

## EQXJS Framework - Advanced Data Transformation Patterns

---

## Learning Objectives

After completing this module, you will be able to:

- Understand EQXJS pipe architecture and advanced data processing patterns
- Implement validation pipes using Joi schemas with type-safe patterns
- Create transformation pipes for various data formats and encodings
- Build async data processing pipelines with error handling
- Apply advanced pipeline patterns for complex data workflows

---

## Overview

This module focuses on the **@corp-ais/eqxjs-pipes** module, which provides powerful data transformation and validation capabilities. You'll learn how to create sophisticated data processing pipelines that handle complex business requirements while maintaining performance and reliability.

### Key Topics Covered

- **Pipe Architecture**: Advanced NestJS pipe patterns with EQXJS integration
- **Validation Pipes**: Type-safe validation using Joi schemas
- **Transformation Pipes**: Data format conversion and manipulation
- **Async Processing**: Stream processing and batch operations
- **Pipeline Patterns**: Multi-stage processing and error recovery

## Exercises

- [Module 5 Exercises](exercise/module-05-exercises.md)

---

## 5.1 Pipe Architecture and Patterns

### Understanding EQXJS Pipe Integration

The EQXJS framework extends NestJS's native pipe system with enterprise-grade features:

```typescript
import { PipeTransform, Injectable, ArgumentMetadata } from "@nestjs/common";
import { EqxjsPipe, ValidationResult } from "@corp-ais/eqxjs-pipes";

@Injectable()
export class CustomDataPipe implements PipeTransform, EqxjsPipe {
  async transform(value: any, metadata: ArgumentMetadata): Promise<any> {
    // EQXJS pipe processing logic
    return this.processWithEqxjs(value, metadata);
  }

  private async processWithEqxjs(
    value: any,
    metadata: ArgumentMetadata,
  ): Promise<ValidationResult> {
    // Enhanced processing with framework utilities
    return {
      isValid: true,
      transformedValue: value,
      metadata: metadata,
    };
  }
}
```

### Advanced Pipe Composition

Create reusable pipe chains for complex data processing:

```typescript
import { Injectable } from "@nestjs/common";
import { PipelineBuilder, DataProcessor } from "@corp-ais/eqxjs-pipes";

@Injectable()
export class DataProcessingService {
  private readonly pipeline: DataProcessor;

  constructor() {
    this.pipeline = new PipelineBuilder()
      .addValidation("request-validation")
      .addTransformation("data-normalization")
      .addSanitization("security-cleanup")
      .addSerialization("response-format")
      .build();
  }

  async processData(rawData: unknown): Promise<ProcessedData> {
    return await this.pipeline.execute(rawData);
  }
}
```

### Performance Optimization Strategies

Optimize pipe performance for high-throughput applications:

```typescript
import { Injectable, Logger } from "@nestjs/common";
import { PerformancePipe, CacheStrategy } from "@corp-ais/eqxjs-pipes";

@Injectable()
export class OptimizedValidationPipe extends PerformancePipe {
  private readonly logger = new Logger(OptimizedValidationPipe.name);
  private readonly cache = new Map<string, ValidationResult>();

  protected getCacheStrategy(): CacheStrategy {
    return {
      enabled: true,
      ttl: 300000, // 5 minutes
      maxSize: 1000,
    };
  }

  async transform(value: any, metadata: ArgumentMetadata): Promise<any> {
    const cacheKey = this.generateCacheKey(value, metadata);

    // Check cache first
    if (this.cache.has(cacheKey)) {
      this.logger.debug(`Cache hit for validation: ${cacheKey}`);
      return this.cache.get(cacheKey)?.transformedValue;
    }

    // Process and cache result
    const result = await this.performValidation(value, metadata);
    this.cache.set(cacheKey, result);

    return result.transformedValue;
  }
}
```

---

## 5.2 Validation Pipe Implementation

### Joi Schema Integration

Implement robust validation using Joi schemas with EQXJS:

```typescript
import { Injectable, BadRequestException } from "@nestjs/common";
import { JoiValidationPipe } from "@corp-ais/eqxjs-pipes";
import * as Joi from "joi";

// User registration DTO with Joi schema
export class CreateUserDto {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  dateOfBirth?: Date;
}

export const CreateUserSchema = Joi.object({
  email: Joi.string().email({ minDomainSegments: 2 }).required().messages({
    "string.email": "Please provide a valid email address",
    "any.required": "Email is required",
  }),

  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
    .required()
    .messages({
      "string.pattern.base":
        "Password must contain at least one uppercase letter, lowercase letter, digit, and special character",
    }),

  firstName: Joi.string().trim().min(2).max(50).required(),

  lastName: Joi.string().trim().min(2).max(50).required(),

  dateOfBirth: Joi.date().iso().max("now").optional(),
});

@Injectable()
export class UserValidationPipe extends JoiValidationPipe {
  constructor() {
    super(CreateUserSchema);
  }

  protected override async handleValidationError(
    error: Joi.ValidationError,
  ): Promise<never> {
    const errorMessages = error.details.map((detail) => ({
      field: detail.path.join("."),
      message: detail.message,
      value: detail.context?.value,
    }));

    throw new BadRequestException({
      message: "Validation failed",
      errors: errorMessages,
      timestamp: new Date().toISOString(),
    });
  }
}
```

### Custom Validation Rules

Create domain-specific validation rules:

```typescript
import { Injectable } from "@nestjs/common";
import { CustomValidatorPipe, ValidationContext } from "@corp-ais/eqxjs-pipes";

interface BusinessRuleContext extends ValidationContext {
  userId: string;
  organizationId: string;
}

@Injectable()
export class BusinessRuleValidationPipe extends CustomValidatorPipe<BusinessRuleContext> {
  async validateBusinessRules(
    value: any,
    context: BusinessRuleContext,
  ): Promise<ValidationResult> {
    const results: ValidationIssue[] = [];

    // Check user permissions
    const hasPermission = await this.validateUserPermission(
      context.userId,
      context.organizationId,
      value.action,
    );

    if (!hasPermission) {
      results.push({
        field: "action",
        code: "INSUFFICIENT_PERMISSIONS",
        message: "User does not have permission to perform this action",
      });
    }

    // Check business constraints
    const isWithinLimits = await this.validateBusinessLimits(
      value,
      context.organizationId,
    );

    if (!isWithinLimits) {
      results.push({
        field: "amount",
        code: "EXCEEDS_BUSINESS_LIMITS",
        message: "Request exceeds organizational limits",
      });
    }

    return {
      isValid: results.length === 0,
      issues: results,
      transformedValue: value,
    };
  }

  private async validateUserPermission(
    userId: string,
    organizationId: string,
    action: string,
  ): Promise<boolean> {
    // Implement permission checking logic
    return true; // Placeholder
  }

  private async validateBusinessLimits(
    value: any,
    organizationId: string,
  ): Promise<boolean> {
    // Implement business rule checking
    return true; // Placeholder
  }
}
```

### Type-Safe Validation Patterns

Implement type-safe validation with comprehensive error handling:

```typescript
import { Injectable } from "@nestjs/common";
import { TypeSafeValidationPipe, ValidatedType } from "@corp-ais/eqxjs-pipes";

interface UserProfile {
  id: string;
  email: string;
  profile: {
    firstName: string;
    lastName: string;
    preferences: UserPreferences;
  };
}

interface UserPreferences {
  language: "en" | "es" | "fr";
  timezone: string;
  notifications: NotificationSettings;
}

@Injectable()
export class TypeSafeUserPipe extends TypeSafeValidationPipe<UserProfile> {
  protected getValidationSchema(): Joi.ObjectSchema<UserProfile> {
    return Joi.object<UserProfile>({
      id: Joi.string().uuid().required(),
      email: Joi.string().email().required(),
      profile: Joi.object({
        firstName: Joi.string().min(2).max(50).required(),
        lastName: Joi.string().min(2).max(50).required(),
        preferences: Joi.object({
          language: Joi.string().valid("en", "es", "fr").required(),
          timezone: Joi.string().required(),
          notifications: Joi.object({
            email: Joi.boolean().default(true),
            push: Joi.boolean().default(false),
            sms: Joi.boolean().default(false),
          }).required(),
        }).required(),
      }).required(),
    });
  }

  protected override transformValue(
    validatedValue: UserProfile,
  ): ValidatedType<UserProfile> {
    return {
      ...validatedValue,
      profile: {
        ...validatedValue.profile,
        fullName: `${validatedValue.profile.firstName} ${validatedValue.profile.lastName}`,
        preferences: {
          ...validatedValue.profile.preferences,
          formattedTimezone: this.formatTimezone(
            validatedValue.profile.preferences.timezone,
          ),
        },
      },
    };
  }

  private formatTimezone(timezone: string): string {
    return (
      Intl.DateTimeFormat("en-US", { timeZoneName: "long" })
        .formatToParts(new Date())
        .find((part) => part.type === "timeZoneName")?.value || timezone
    );
  }
}
```

---

## 5.3 Transformation Pipes

### Data Format Transformation

Handle multiple data formats with dedicated transformation pipes:

```typescript
import { Injectable } from "@nestjs/common";
import { DataTransformationPipe, SupportedFormat } from "@corp-ais/eqxjs-pipes";

export interface TransformationConfig {
  inputFormat: SupportedFormat;
  outputFormat: SupportedFormat;
  options?: TransformationOptions;
}

@Injectable()
export class MultiFormatTransformationPipe extends DataTransformationPipe {
  private readonly transformers = new Map<string, DataTransformer>();

  constructor() {
    super();
    this.initializeTransformers();
  }

  async transform(
    value: any,
    metadata: ArgumentMetadata & { config?: TransformationConfig },
  ): Promise<any> {
    const config = metadata.config;
    if (!config) {
      return value;
    }

    const transformerKey = `${config.inputFormat}-to-${config.outputFormat}`;
    const transformer = this.transformers.get(transformerKey);

    if (!transformer) {
      throw new Error(`No transformer found for ${transformerKey}`);
    }

    return await transformer.transform(value, config.options);
  }

  private initializeTransformers(): void {
    // JSON to XML transformation
    this.transformers.set("json-to-xml", {
      transform: async (data: any, options?: any) => {
        const xmlOptions = {
          xmldec: { version: "1.0", encoding: "UTF-8" },
          renderOpts: { pretty: true },
          ...options,
        };
        return this.jsonToXml(data, xmlOptions);
      },
    });

    // CSV to JSON transformation
    this.transformers.set("csv-to-json", {
      transform: async (data: string, options?: any) => {
        return this.csvToJson(data, options);
      },
    });

    // XML to JSON transformation
    this.transformers.set("xml-to-json", {
      transform: async (data: string, options?: any) => {
        return this.xmlToJson(data, options);
      },
    });
  }

  private async jsonToXml(data: any, options: any): Promise<string> {
    // Intentionally omitted: XML serialization typically requires a dedicated XML library.
    // Keep the transformation boundary here and plug in your organization's preferred implementation.
    throw new Error("JSON->XML transformation not implemented");
  }

  private async csvToJson(csvData: string, options: any): Promise<any[]> {
    // Intentionally omitted: CSV parsing typically requires a dedicated CSV parser.
    // Keep this as a well-defined transformation stage rather than embedding parsing details in controllers.
    throw new Error("CSV->JSON transformation not implemented");
  }

  private async xmlToJson(xmlData: string, options: any): Promise<any> {
    // Intentionally omitted: XML parsing typically requires a dedicated XML parser.
    throw new Error("XML->JSON transformation not implemented");
  }
}
```

### Date/Time and Locale Transformation

Handle date/time formatting and internationalization:

```typescript
import { Injectable } from "@nestjs/common";
import { LocalizationPipe, LocaleContext } from "@corp-ais/eqxjs-pipes";

export interface LocalizationOptions {
  locale: string;
  timezone: string;
  currency: string;
  dateFormat: "short" | "medium" | "long" | "full";
  numberFormat: "decimal" | "percent" | "currency";
}

@Injectable()
export class InternationalizationPipe extends LocalizationPipe {
  async transform(
    value: any,
    metadata: ArgumentMetadata & { locale?: LocalizationOptions },
  ): Promise<any> {
    const options = metadata.locale;
    if (!options || !value) {
      return value;
    }

    return this.localizeObject(value, options);
  }

  private localizeObject(obj: any, options: LocalizationOptions): any {
    if (obj === null || obj === undefined) {
      return obj;
    }

    if (Array.isArray(obj)) {
      return obj.map((item) => this.localizeObject(item, options));
    }

    if (typeof obj === "object" && obj instanceof Date) {
      return this.formatDate(obj, options);
    }

    if (typeof obj === "number") {
      return this.formatNumber(obj, options);
    }

    if (typeof obj === "object") {
      const localized: any = {};
      for (const [key, value] of Object.entries(obj)) {
        localized[key] = this.localizeObject(value, options);
      }
      return localized;
    }

    return obj;
  }

  private formatDate(date: Date, options: LocalizationOptions): string {
    return new Intl.DateTimeFormat(options.locale, {
      dateStyle: options.dateFormat,
      timeZone: options.timezone,
    }).format(date);
  }

  private formatNumber(num: number, options: LocalizationOptions): string {
    switch (options.numberFormat) {
      case "currency":
        return new Intl.NumberFormat(options.locale, {
          style: "currency",
          currency: options.currency,
        }).format(num);
      case "percent":
        return new Intl.NumberFormat(options.locale, {
          style: "percent",
        }).format(num);
      default:
        return new Intl.NumberFormat(options.locale).format(num);
    }
  }
}
```

### String Manipulation and Encoding

Handle text processing and encoding transformations:

```typescript
import { Injectable } from "@nestjs/common";
import {
  StringTransformationPipe,
  EncodingOptions,
} from "@corp-ais/eqxjs-pipes";

export interface StringProcessingOptions {
  encoding?: "utf8" | "base64" | "hex";
  normalization?: "NFC" | "NFD" | "NFKC" | "NFKD";
  case?: "upper" | "lower" | "title" | "camel" | "snake" | "kebab";
  trim?: boolean;
  sanitize?: boolean;
}

@Injectable()
export class StringProcessingPipe extends StringTransformationPipe {
  async transform(
    value: any,
    metadata: ArgumentMetadata & { stringOptions?: StringProcessingOptions },
  ): Promise<any> {
    const options = metadata.stringOptions;
    if (!options || typeof value !== "string") {
      return value;
    }

    let processed = value;

    // Apply encoding transformation
    if (options.encoding) {
      processed = this.handleEncoding(processed, options.encoding);
    }

    // Apply normalization
    if (options.normalization) {
      processed = processed.normalize(options.normalization);
    }

    // Apply case transformation
    if (options.case) {
      processed = this.transformCase(processed, options.case);
    }

    // Apply trimming
    if (options.trim) {
      processed = processed.trim();
    }

    // Apply sanitization
    if (options.sanitize) {
      processed = this.sanitizeString(processed);
    }

    return processed;
  }

  private handleEncoding(
    value: string,
    encoding: "utf8" | "base64" | "hex",
  ): string {
    switch (encoding) {
      case "base64":
        return Buffer.from(value, "utf8").toString("base64");
      case "hex":
        return Buffer.from(value, "utf8").toString("hex");
      case "utf8":
      default:
        return value;
    }
  }

  private transformCase(
    value: string,
    caseType: StringProcessingOptions["case"],
  ): string {
    switch (caseType) {
      case "upper":
        return value.toUpperCase();
      case "lower":
        return value.toLowerCase();
      case "title":
        return value.replace(
          /\w\S*/g,
          (txt) => txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase(),
        );
      case "camel":
        return value.replace(/[-_\s]+(.)?/g, (_, c) =>
          c ? c.toUpperCase() : "",
        );
      case "snake":
        return value
          .replace(/([A-Z])/g, "_$1")
          .toLowerCase()
          .replace(/^_/, "");
      case "kebab":
        return value
          .replace(/([A-Z])/g, "-$1")
          .toLowerCase()
          .replace(/^-/, "");
      default:
        return value;
    }
  }

  private sanitizeString(value: string): string {
    // Remove potentially harmful characters and escape HTML
    return value
      .replace(/[<>]/g, "") // Remove angle brackets
      .replace(/['"]/g, "") // Remove quotes
      .replace(/javascript:/gi, "") // Remove javascript: protocol
      .replace(/on\w+=/gi, "") // Remove event handlers
      .trim();
  }
}
```

---

## 5.4 Async Data Processing

### Stream Processing Patterns

Implement efficient stream processing for large datasets:

```typescript
import { Injectable, Logger } from "@nestjs/common";
import { Transform, Readable, Writable } from "stream";
import { AsyncDataProcessor, StreamConfig } from "@corp-ais/eqxjs-pipes";

export interface StreamProcessingConfig extends StreamConfig {
  batchSize: number;
  concurrency: number;
  bufferTimeout: number;
}

@Injectable()
export class StreamDataProcessor extends AsyncDataProcessor {
  private readonly logger = new Logger(StreamDataProcessor.name);

  async processDataStream<T, R>(
    inputStream: Readable,
    processor: (batch: T[]) => Promise<R[]>,
    config: StreamProcessingConfig,
  ): Promise<Writable> {
    const outputStream = new Writable({ objectMode: true });
    const batchProcessor = this.createBatchTransform(processor, config);

    // Create processing pipeline
    const pipeline = inputStream
      .pipe(this.createJSONParser())
      .pipe(this.createBatchAccumulator(config.batchSize, config.bufferTimeout))
      .pipe(batchProcessor)
      .pipe(this.createResultSerializer())
      .pipe(outputStream);

    // Handle errors and monitoring
    pipeline.on("error", (error) => {
      this.logger.error(
        `Stream processing error: ${error.message}`,
        error.stack,
      );
    });

    pipeline.on("finish", () => {
      this.logger.log("Stream processing completed successfully");
    });

    return outputStream;
  }

  private createBatchTransform<T, R>(
    processor: (batch: T[]) => Promise<R[]>,
    config: StreamProcessingConfig,
  ): Transform {
    return new Transform({
      objectMode: true,
      async transform(batch: T[], encoding, callback) {
        try {
          const results = await processor(batch);
          callback(null, results);
        } catch (error) {
          callback(error);
        }
      },
    });
  }

  private createBatchAccumulator(
    batchSize: number,
    timeout: number,
  ): Transform {
    let batch: any[] = [];
    let timer: NodeJS.Timeout | null = null;

    const flushBatch = (transform: Transform) => {
      if (batch.length > 0) {
        transform.push(batch);
        batch = [];
      }
      if (timer) {
        clearTimeout(timer);
        timer = null;
      }
    };

    return new Transform({
      objectMode: true,
      transform(chunk, encoding, callback) {
        batch.push(chunk);

        // Flush when batch is full
        if (batch.length >= batchSize) {
          flushBatch(this);
        } else {
          // Set timeout to flush incomplete batches
          if (!timer) {
            timer = setTimeout(() => flushBatch(this), timeout);
          }
        }

        callback();
      },
      flush(callback) {
        flushBatch(this);
        callback();
      },
    });
  }

  private createJSONParser(): Transform {
    return new Transform({
      readableObjectMode: true,
      writableObjectMode: false,
      transform(chunk, encoding, callback) {
        try {
          const lines = chunk
            .toString()
            .split("\n")
            .filter((line) => line.trim());
          for (const line of lines) {
            this.push(JSON.parse(line));
          }
          callback();
        } catch (error) {
          callback(error);
        }
      },
    });
  }

  private createResultSerializer(): Transform {
    return new Transform({
      readableObjectMode: false,
      writableObjectMode: true,
      transform(chunk, encoding, callback) {
        try {
          const serialized = JSON.stringify(chunk) + "\n";
          callback(null, serialized);
        } catch (error) {
          callback(error);
        }
      },
    });
  }
}
```

### Message Queue Integration

Process data from message queues with error handling:

```typescript
import { Injectable, OnModuleInit, OnModuleDestroy } from "@nestjs/common";
import {
  QueueProcessor,
  QueueConfig,
  MessageHandler,
} from "@corp-ais/eqxjs-pipes";

export interface ProcessingJob<T = any> {
  id: string;
  data: T;
  attempt: number;
  maxAttempts: number;
  delay?: number;
}

@Injectable()
export class QueueDataProcessor
  extends QueueProcessor
  implements OnModuleInit, OnModuleDestroy
{
  private processors = new Map<string, MessageHandler>();
  private activeJobs = new Map<string, ProcessingJob>();

  async onModuleInit() {
    await this.initializeQueues();
    await this.startProcessing();
  }

  async onModuleDestroy() {
    await this.stopProcessing();
    await this.closeQueues();
  }

  registerProcessor<T>(
    queueName: string,
    handler: (data: T) => Promise<any>,
  ): void {
    this.processors.set(queueName, {
      handle: handler,
      retryConfig: {
        maxAttempts: 3,
        backoffType: "exponential",
        backoffSettings: {
          initial: 1000,
          max: 30000,
          multiplier: 2,
        },
      },
    });
  }

  async processJob<T>(queueName: string, job: ProcessingJob<T>): Promise<void> {
    const processor = this.processors.get(queueName);
    if (!processor) {
      throw new Error(`No processor registered for queue: ${queueName}`);
    }

    this.activeJobs.set(job.id, job);

    try {
      this.logger.log(`Processing job ${job.id} (attempt ${job.attempt})`);

      const result = await processor.handle(job.data);

      this.logger.log(`Job ${job.id} completed successfully`);
      await this.markJobComplete(job.id, result);
    } catch (error) {
      this.logger.error(`Job ${job.id} failed: ${error.message}`);
      await this.handleJobFailure(job, error, processor.retryConfig);
    } finally {
      this.activeJobs.delete(job.id);
    }
  }

  private async handleJobFailure(
    job: ProcessingJob,
    error: Error,
    retryConfig: any,
  ): Promise<void> {
    if (job.attempt >= job.maxAttempts) {
      this.logger.error(
        `Job ${job.id} failed permanently after ${job.attempt} attempts`,
      );
      await this.moveToDeadLetter(job, error);
      return;
    }

    // Calculate retry delay
    const delay = this.calculateRetryDelay(job.attempt, retryConfig);

    this.logger.warn(`Job ${job.id} will be retried in ${delay}ms`);
    await this.scheduleRetry(job, delay);
  }

  private calculateRetryDelay(attempt: number, retryConfig: any): number {
    const { initial, max, multiplier } = retryConfig.backoffSettings;

    switch (retryConfig.backoffType) {
      case "exponential":
        return Math.min(initial * Math.pow(multiplier, attempt - 1), max);
      case "linear":
        return Math.min(initial * attempt, max);
      case "fixed":
      default:
        return initial;
    }
  }

  async getActiveJobsStats(): Promise<{
    total: number;
    byQueue: Record<string, number>;
    byStatus: Record<string, number>;
  }> {
    const stats = {
      total: this.activeJobs.size,
      byQueue: {} as Record<string, number>,
      byStatus: {} as Record<string, number>,
    };

    // Count jobs by queue and status
    for (const job of this.activeJobs.values()) {
      // Implement queue and status tracking
    }

    return stats;
  }
}
```

### Batch Processing Implementation

Handle large datasets with efficient batch processing:

```typescript
import { Injectable } from "@nestjs/common";
import {
  BatchProcessor,
  BatchConfig,
  ProcessingResult,
} from "@corp-ais/eqxjs-pipes";

export interface BatchJob<T> {
  id: string;
  items: T[];
  metadata: {
    startTime: Date;
    totalItems: number;
    batchSize: number;
  };
}

export interface BatchResult<T, R> {
  jobId: string;
  processed: number;
  failed: number;
  results: R[];
  errors: ProcessingError[];
  duration: number;
}

@Injectable()
export class EnhancedBatchProcessor<T, R> extends BatchProcessor<T, R> {
  async processBatch(
    items: T[],
    processor: (item: T) => Promise<R>,
    config: BatchConfig,
  ): Promise<BatchResult<T, R>> {
    const jobId = this.generateJobId();
    const startTime = new Date();

    const job: BatchJob<T> = {
      id: jobId,
      items,
      metadata: {
        startTime,
        totalItems: items.length,
        batchSize: config.batchSize || 100,
      },
    };

    this.logger.log(`Starting batch job ${jobId} with ${items.length} items`);

    const results: R[] = [];
    const errors: ProcessingError[] = [];
    let processed = 0;

    // Process in chunks
    const chunks = this.createChunks(items, job.metadata.batchSize);

    for (let i = 0; i < chunks.length; i++) {
      const chunk = chunks[i];

      this.logger.debug(
        `Processing chunk ${i + 1}/${chunks.length} (${chunk.length} items)`,
      );

      const chunkResults = await this.processChunk(
        chunk,
        processor,
        config,
        jobId,
      );

      results.push(...chunkResults.results);
      errors.push(...chunkResults.errors);
      processed += chunkResults.processed;

      // Progress callback
      if (config.onProgress) {
        await config.onProgress({
          jobId,
          processed,
          total: items.length,
          percentage: (processed / items.length) * 100,
        });
      }

      // Add delay between chunks if configured
      if (config.chunkDelay && i < chunks.length - 1) {
        await this.delay(config.chunkDelay);
      }
    }

    const duration = Date.now() - startTime.getTime();
    const result: BatchResult<T, R> = {
      jobId,
      processed,
      failed: errors.length,
      results,
      errors,
      duration,
    };

    this.logger.log(
      `Batch job ${jobId} completed in ${duration}ms. ` +
        `Processed: ${processed}, Failed: ${errors.length}`,
    );

    return result;
  }

  private async processChunk<T, R>(
    chunk: T[],
    processor: (item: T) => Promise<R>,
    config: BatchConfig,
    jobId: string,
  ): Promise<{
    results: R[];
    errors: ProcessingError[];
    processed: number;
  }> {
    const results: R[] = [];
    const errors: ProcessingError[] = [];

    // Process items with controlled concurrency
    const semaphore = new Semaphore(config.concurrency || 5);

    const promises = chunk.map(async (item, index) => {
      await semaphore.acquire();

      try {
        const result = await processor(item);
        results.push(result);
        return { success: true, result };
      } catch (error) {
        const processingError: ProcessingError = {
          itemIndex: index,
          error: error.message,
          item,
          jobId,
          timestamp: new Date(),
        };
        errors.push(processingError);
        return { success: false, error: processingError };
      } finally {
        semaphore.release();
      }
    });

    await Promise.all(promises);

    return {
      results,
      errors,
      processed: results.length,
    };
  }

  private createChunks<T>(items: T[], chunkSize: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < items.length; i += chunkSize) {
      chunks.push(items.slice(i, i + chunkSize));
    }
    return chunks;
  }

  private generateJobId(): string {
    return `batch_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

// Semaphore utility for controlling concurrency
class Semaphore {
  private available: number;
  private waiters: (() => void)[] = [];

  constructor(max: number) {
    this.available = max;
  }

  async acquire(): Promise<void> {
    if (this.available > 0) {
      this.available--;
      return;
    }

    return new Promise((resolve) => {
      this.waiters.push(resolve);
    });
  }

  release(): void {
    if (this.waiters.length > 0) {
      const resolve = this.waiters.shift()!;
      resolve();
    } else {
      this.available++;
    }
  }
}
```

---

## 5.5 Advanced Pipeline Patterns

### Multi-Stage Processing Pipeline

Implement complex multi-stage data processing:

```typescript
import { Injectable } from "@nestjs/common";
import {
  PipelineStage,
  MultiStagePipeline,
  StageResult,
  PipelineContext,
} from "@corp-ais/eqxjs-pipes";

export interface StageDefinition<TInput, TOutput> {
  name: string;
  stage: PipelineStage<TInput, TOutput>;
  errorHandling?: "stop" | "skip" | "retry";
  retryConfig?: {
    maxAttempts: number;
    backoffMs: number;
  };
  condition?: (input: TInput, context: PipelineContext) => boolean;
}

@Injectable()
export class DataProcessingPipeline extends MultiStagePipeline {
  private stages: StageDefinition<any, any>[] = [];
  private metrics = new Map<string, StageMetrics>();

  addStage<TInput, TOutput>(
    definition: StageDefinition<TInput, TOutput>,
  ): this {
    this.stages.push(definition);
    this.metrics.set(definition.name, {
      totalExecutions: 0,
      successfulExecutions: 0,
      failedExecutions: 0,
      totalProcessingTime: 0,
      averageProcessingTime: 0,
    });
    return this;
  }

  async execute<TInput, TOutput>(
    initialInput: TInput,
    context?: PipelineContext,
  ): Promise<StageResult<TOutput>> {
    const pipelineContext: PipelineContext = {
      executionId: this.generateExecutionId(),
      startTime: new Date(),
      metadata: {},
      ...context,
    };

    this.logger.log(
      `Starting pipeline execution ${pipelineContext.executionId}`,
    );

    let currentInput: any = initialInput;
    const stageResults: StageResult<any>[] = [];

    for (const stageDefinition of this.stages) {
      // Check stage condition
      if (
        stageDefinition.condition &&
        !stageDefinition.condition(currentInput, pipelineContext)
      ) {
        this.logger.debug(
          `Skipping stage ${stageDefinition.name} due to condition`,
        );
        continue;
      }

      const stageResult = await this.executeStage(
        stageDefinition,
        currentInput,
        pipelineContext,
      );

      stageResults.push(stageResult);

      if (!stageResult.success) {
        if (stageDefinition.errorHandling === "stop") {
          this.logger.error(
            `Pipeline stopped at stage ${stageDefinition.name}`,
          );
          break;
        } else if (stageDefinition.errorHandling === "skip") {
          this.logger.warn(
            `Skipping stage ${stageDefinition.name} due to error`,
          );
          continue;
        }
      }

      currentInput = stageResult.output;
    }

    const finalResult: StageResult<TOutput> = {
      success: stageResults.every(
        (r) => r.success || r.stage?.errorHandling === "skip",
      ),
      output: currentInput,
      error: stageResults.find((r) => !r.success)?.error,
      executionTime: Date.now() - pipelineContext.startTime.getTime(),
      stageResults,
      context: pipelineContext,
    };

    this.logger.log(
      `Pipeline execution ${pipelineContext.executionId} completed. ` +
        `Success: ${finalResult.success}, Duration: ${finalResult.executionTime}ms`,
    );

    return finalResult;
  }

  private async executeStage<TInput, TOutput>(
    definition: StageDefinition<TInput, TOutput>,
    input: TInput,
    context: PipelineContext,
  ): Promise<StageResult<TOutput>> {
    const metrics = this.metrics.get(definition.name)!;
    const startTime = Date.now();

    metrics.totalExecutions++;

    let attempt = 1;
    const maxAttempts = definition.retryConfig?.maxAttempts || 1;

    while (attempt <= maxAttempts) {
      try {
        this.logger.debug(
          `Executing stage ${definition.name} (attempt ${attempt})`,
        );

        const output = await definition.stage.process(input, context);

        const executionTime = Date.now() - startTime;
        metrics.successfulExecutions++;
        metrics.totalProcessingTime += executionTime;
        metrics.averageProcessingTime =
          metrics.totalProcessingTime / metrics.successfulExecutions;

        return {
          success: true,
          output,
          executionTime,
          stage: definition,
          context,
        };
      } catch (error) {
        this.logger.error(
          `Stage ${definition.name} failed (attempt ${attempt}): ${error.message}`,
        );

        if (attempt === maxAttempts) {
          metrics.failedExecutions++;

          return {
            success: false,
            output: undefined as any,
            error: error.message,
            executionTime: Date.now() - startTime,
            stage: definition,
            context,
          };
        }

        // Wait before retry
        if (definition.retryConfig?.backoffMs) {
          await this.delay(definition.retryConfig.backoffMs * attempt);
        }

        attempt++;
      }
    }

    // Should never reach here, but TypeScript requires it
    throw new Error("Unexpected end of executeStage method");
  }

  getStageMetrics(): Map<string, StageMetrics> {
    return new Map(this.metrics);
  }

  getPipelineStats(): {
    totalStages: number;
    avgExecutionTime: number;
    successRate: number;
    bottlenecks: string[];
  } {
    const stats = {
      totalStages: this.stages.length,
      avgExecutionTime: 0,
      successRate: 0,
      bottlenecks: [] as string[],
    };

    let totalExecutions = 0;
    let totalSuccesses = 0;
    let totalTime = 0;

    for (const [stageName, metrics] of this.metrics.entries()) {
      totalExecutions += metrics.totalExecutions;
      totalSuccesses += metrics.successfulExecutions;
      totalTime += metrics.totalProcessingTime;

      // Identify bottlenecks (stages with above-average execution time)
      if (metrics.averageProcessingTime > 1000) {
        // 1 second threshold
        stats.bottlenecks.push(stageName);
      }
    }

    stats.avgExecutionTime = totalTime / Math.max(totalExecutions, 1);
    stats.successRate = (totalSuccesses / Math.max(totalExecutions, 1)) * 100;

    return stats;
  }

  private generateExecutionId(): string {
    return `pipeline_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

interface StageMetrics {
  totalExecutions: number;
  successfulExecutions: number;
  failedExecutions: number;
  totalProcessingTime: number;
  averageProcessingTime: number;
}
```

### Conditional Pipeline Execution

Implement pipelines with dynamic routing and conditions:

```typescript
import { Injectable } from "@nestjs/common";
import {
  ConditionalPipeline,
  RouteCondition,
  PipelineBranch,
} from "@corp-ais/eqxjs-pipes";

export interface BranchDefinition<T> {
  id: string;
  condition: RouteCondition<T>;
  pipeline: PipelineStage<T, any>[];
  fallback?: boolean;
}

@Injectable()
export class RoutingPipeline<T> extends ConditionalPipeline<T> {
  private branches: BranchDefinition<T>[] = [];

  addBranch(definition: BranchDefinition<T>): this {
    this.branches.push(definition);
    return this;
  }

  async execute(input: T, context: PipelineContext): Promise<any> {
    this.logger.debug(
      `Evaluating routing conditions for ${this.branches.length} branches`,
    );

    // Evaluate conditions and find matching branch
    for (const branch of this.branches) {
      try {
        const matches = await branch.condition.evaluate(input, context);

        if (matches) {
          this.logger.log(`Routing to branch: ${branch.id}`);
          return await this.executeBranch(branch, input, context);
        }
      } catch (error) {
        this.logger.error(
          `Error evaluating condition for branch ${branch.id}: ${error.message}`,
        );
      }
    }

    // Try fallback branch
    const fallbackBranch = this.branches.find((b) => b.fallback);
    if (fallbackBranch) {
      this.logger.log(`Using fallback branch: ${fallbackBranch.id}`);
      return await this.executeBranch(fallbackBranch, input, context);
    }

    throw new Error("No matching branch found and no fallback configured");
  }

  private async executeBranch(
    branch: BranchDefinition<T>,
    input: T,
    context: PipelineContext,
  ): Promise<any> {
    let currentInput: any = input;

    for (const stage of branch.pipeline) {
      currentInput = await stage.process(currentInput, context);
    }

    return currentInput;
  }
}

// Example usage with business logic routing
@Injectable()
export class BusinessProcessingPipeline {
  private pipeline = new RoutingPipeline<BusinessData>();

  constructor() {
    this.setupBranches();
  }

  private setupBranches(): void {
    // High-priority processing branch
    this.pipeline.addBranch({
      id: "high-priority",
      condition: {
        evaluate: async (data: BusinessData) => data.priority === "high",
      },
      pipeline: [
        new ValidateHighPriorityStage(),
        new EnhancedProcessingStage(),
        new PriorityNotificationStage(),
      ],
    });

    // Standard processing branch
    this.pipeline.addBranch({
      id: "standard",
      condition: {
        evaluate: async (data: BusinessData) => data.priority === "normal",
      },
      pipeline: [
        new StandardValidationStage(),
        new StandardProcessingStage(),
        new StandardNotificationStage(),
      ],
    });

    // Batch processing branch
    this.pipeline.addBranch({
      id: "batch",
      condition: {
        evaluate: async (data: BusinessData) => data.type === "batch",
      },
      pipeline: [
        new BatchValidationStage(),
        new BatchProcessingStage(),
        new BatchReportingStage(),
      ],
    });

    // Fallback branch
    this.pipeline.addBranch({
      id: "default",
      condition: {
        evaluate: async () => true, // Always matches
      },
      pipeline: [new BasicValidationStage(), new BasicProcessingStage()],
      fallback: true,
    });
  }

  async processBusinessData(data: BusinessData): Promise<ProcessingResult> {
    const context: PipelineContext = {
      executionId: `business_${Date.now()}`,
      startTime: new Date(),
      metadata: {
        userId: data.userId,
        organizationId: data.organizationId,
      },
    };

    return await this.pipeline.execute(data, context);
  }
}
```

---

## Summary

In this module, you've learned:

1. **Advanced Pipe Architecture**: How to leverage EQXJS pipes for enterprise-grade data processing
2. **Validation Patterns**: Creating robust validation with Joi schemas and custom business rules
3. **Data Transformation**: Implementing multi-format transformation and localization
4. **Async Processing**: Building stream processing and batch processing systems
5. **Pipeline Patterns**: Creating sophisticated multi-stage processing workflows

## 📋 Key Takeaways

- **Performance First**: Always consider performance implications when designing data processing pipelines
- **Error Resilience**: Implement comprehensive error handling and recovery strategies
- **Type Safety**: Use TypeScript effectively to ensure type safety throughout the pipeline
- **Monitoring**: Include metrics and monitoring in all data processing operations
- **Configurability**: Make pipelines configurable to handle different business requirements

## Next Steps

- Complete the [Module 5 Exercises](exercise/module-05-exercises.md)
- Practice implementing custom validation pipes
- Build a complete data processing pipeline for your use case
- Review [Module 6: Logging and Monitoring Systems](Module-06-Logging-and-Monitoring-Systems.md)

---

**Previous: [Module 4 - Security and Exception Handling](Module-04-Security-and-Exception-Handling.md)** | **Next: [Module 6 - Logging and Monitoring Systems](Module-06-Logging-and-Monitoring-Systems.md)**
