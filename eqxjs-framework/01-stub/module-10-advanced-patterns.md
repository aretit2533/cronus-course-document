# Module 10: Advanced Patterns & Integration

## 📚 Learning Objectives

By the end of this module, you will understand:

- Advanced enterprise architecture patterns with EQXJS
- Microservices communication and integration strategies
- Event-driven architecture and message queue implementation
- Advanced caching and performance optimization patterns
- Multi-tenant architecture and domain-driven design
- Cloud-native deployment and scaling strategies

## 🧭 Visual Flow (Mermaid)

```mermaid
flowchart TB
  UI["Adapters<br/>(HTTP controllers / consumers)"] --> App["Application layer<br/>(use cases)"]
  App --> Domain["Domain model<br/>(entities/VOs)"]
  App --> Ports["Ports<br/>(interfaces)"]
  Ports --> Infra["Adapters<br/>(DB/HTTP/MQ clients)"]
  Infra --> Ext[External systems]
```

---

## 🏛️ 10.1 Advanced Enterprise Architecture Patterns

### Hexagonal Architecture with EQXJS

The EQXJS Framework supports hexagonal (ports and adapters) architecture for maintainable enterprise applications:

```typescript
// src/core/domain/ports/user.repository.interface.ts
export interface UserRepositoryPort {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
  findByRole(role: UserRole): Promise<User[]>;
}

// src/core/domain/ports/payment.gateway.interface.ts
export interface PaymentGatewayPort {
  processPayment(payment: PaymentRequest): Promise<PaymentResult>;
  refundPayment(transactionId: string, amount: number): Promise<RefundResult>;
  getTransactionStatus(transactionId: string): Promise<TransactionStatus>;
}

// src/core/domain/services/order.domain-service.ts
@Injectable()
export class OrderDomainService {
  constructor(
    @Inject("UserRepositoryPort")
    private readonly userRepository: UserRepositoryPort,
    @Inject("PaymentGatewayPort")
    private readonly paymentGateway: PaymentGatewayPort,
    @Inject("InventoryServicePort")
    private readonly inventoryService: InventoryServicePort,
    private readonly eventBus: EventBus,
  ) {}

  async processOrder(orderRequest: CreateOrderRequest): Promise<OrderResult> {
    // Domain business logic
    const user = await this.userRepository.findById(orderRequest.userId);
    if (!user) {
      throw new DomainException("User not found");
    }

    // Validate order
    const orderValidation = await this.validateOrder(orderRequest);
    if (!orderValidation.isValid) {
      throw new DomainException(orderValidation.errors.join(", "));
    }

    // Check inventory
    const inventoryCheck = await this.inventoryService.checkAvailability(
      orderRequest.items.map((item) => ({
        productId: item.productId,
        quantity: item.quantity,
      })),
    );

    if (!inventoryCheck.available) {
      throw new DomainException("Insufficient inventory");
    }

    // Create order
    const order = Order.create({
      id: OrderId.generate(),
      userId: UserId.fromString(orderRequest.userId),
      items: orderRequest.items.map((item) => OrderItem.create(item)),
      totalAmount: this.calculateTotal(orderRequest.items),
      status: OrderStatus.PENDING,
    });

    // Reserve inventory
    await this.inventoryService.reserve(
      order.items.map((item) => ({
        productId: item.productId.value,
        quantity: item.quantity,
        reservationId: order.id.value,
      })),
    );

    // Process payment
    const paymentResult = await this.paymentGateway.processPayment({
      amount: order.totalAmount.value,
      currency: order.totalAmount.currency,
      customerId: user.id.value,
      orderId: order.id.value,
      paymentMethod: orderRequest.paymentMethod,
    });

    if (!paymentResult.success) {
      // Rollback inventory reservation
      await this.inventoryService.cancelReservation(order.id.value);
      throw new DomainException(
        `Payment failed: ${paymentResult.errorMessage}`,
      );
    }

    // Update order status
    order.confirmPayment(paymentResult.transactionId);

    // Emit domain event
    this.eventBus.publish(
      new OrderConfirmedEvent({
        orderId: order.id.value,
        userId: user.id.value,
        totalAmount: order.totalAmount.value,
        items: order.items.map((item) => item.toDTO()),
        confirmedAt: new Date(),
      }),
    );

    return {
      success: true,
      orderId: order.id.value,
      transactionId: paymentResult.transactionId,
      estimatedDelivery: this.calculateEstimatedDelivery(order),
    };
  }

  private async validateOrder(
    request: CreateOrderRequest,
  ): Promise<ValidationResult> {
    const errors: string[] = [];

    if (!request.items || request.items.length === 0) {
      errors.push("Order must contain at least one item");
    }

    for (const item of request.items) {
      if (item.quantity <= 0) {
        errors.push(`Invalid quantity for item ${item.productId}`);
      }

      if (item.unitPrice <= 0) {
        errors.push(`Invalid price for item ${item.productId}`);
      }
    }

    const totalAmount = this.calculateTotal(request.items);
    if (totalAmount <= 0) {
      errors.push("Order total must be greater than zero");
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }
}
```

### Domain-Driven Design Implementation

```typescript
// src/core/domain/value-objects/money.value-object.ts
export class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string,
  ) {
    if (amount < 0) {
      throw new DomainException("Money amount cannot be negative");
    }
    if (!currency || currency.length !== 3) {
      throw new DomainException("Invalid currency code");
    }
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new DomainException("Cannot add money with different currencies");
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }

  toString(): string {
    return `${this.amount} ${this.currency}`;
  }

  static zero(currency: string): Money {
    return new Money(0, currency);
  }
}

// src/core/domain/entities/order.entity.ts
export class Order extends AggregateRoot<OrderId> {
  private constructor(
    id: OrderId,
    private readonly _userId: UserId,
    private readonly _items: OrderItem[],
    private _totalAmount: Money,
    private _status: OrderStatus,
    private _createdAt: Date,
    private _paymentTransactionId?: string,
  ) {
    super(id);
  }

  static create(props: {
    id: OrderId;
    userId: UserId;
    items: OrderItem[];
    totalAmount: Money;
    status: OrderStatus;
  }): Order {
    const order = new Order(
      props.id,
      props.userId,
      props.items,
      props.totalAmount,
      props.status,
      new Date(),
    );

    // Domain event
    order.addDomainEvent(
      new OrderCreatedEvent({
        orderId: order.id.value,
        userId: order.userId.value,
        totalAmount: order.totalAmount.amount,
        createdAt: order.createdAt,
      }),
    );

    return order;
  }

  get userId(): UserId {
    return this._userId;
  }
  get items(): readonly OrderItem[] {
    return [...this._items];
  }
  get totalAmount(): Money {
    return this._totalAmount;
  }
  get status(): OrderStatus {
    return this._status;
  }
  get createdAt(): Date {
    return this._createdAt;
  }
  get paymentTransactionId(): string | undefined {
    return this._paymentTransactionId;
  }

  confirmPayment(transactionId: string): void {
    if (this._status !== OrderStatus.PENDING) {
      throw new DomainException("Can only confirm payment for pending orders");
    }

    this._paymentTransactionId = transactionId;
    this._status = OrderStatus.CONFIRMED;

    this.addDomainEvent(
      new OrderConfirmedEvent({
        orderId: this.id.value,
        transactionId,
        confirmedAt: new Date(),
      }),
    );
  }

  ship(trackingNumber: string): void {
    if (this._status !== OrderStatus.CONFIRMED) {
      throw new DomainException("Can only ship confirmed orders");
    }

    this._status = OrderStatus.SHIPPED;

    this.addDomainEvent(
      new OrderShippedEvent({
        orderId: this.id.value,
        trackingNumber,
        shippedAt: new Date(),
      }),
    );
  }

  cancel(reason: string): void {
    if (
      this._status === OrderStatus.SHIPPED ||
      this._status === OrderStatus.DELIVERED
    ) {
      throw new DomainException("Cannot cancel shipped or delivered orders");
    }

    this._status = OrderStatus.CANCELLED;

    this.addDomainEvent(
      new OrderCancelledEvent({
        orderId: this.id.value,
        reason,
        cancelledAt: new Date(),
      }),
    );
  }
}
```

---

## 🔗 10.2 Microservices Communication Patterns

### Service-to-Service Communication

```typescript
// src/infrastructure/http/services/catalog.service.client.ts
@Injectable()
export class CatalogServiceClient {
  private readonly httpClient: AxiosInstance;
  private readonly circuitBreaker: CircuitBreaker;

  constructor(
    @Inject("HTTP_CLIENT") httpClient: AxiosInstance,
    private readonly circuitBreakerService: CircuitBreakerService,
    private readonly tracingService: TracingService,
  ) {
    this.httpClient = httpClient;
    this.circuitBreaker =
      this.circuitBreakerService.getCircuitBreaker("catalog-service");
  }

  async getProduct(productId: string): Promise<ProductResponse> {
    return this.tracingService.withSpan(
      "catalog-service.get-product",
      async (span) => {
        span.setAttribute("product.id", productId);

        return this.circuitBreaker.fire(async () => {
          const response = await this.httpClient.get(`/products/${productId}`);

          if (response.status !== 200) {
            throw new ServiceException(
              `Catalog service error: ${response.statusText}`,
            );
          }

          return response.data;
        });
      },
    );
  }

  async searchProducts(
    query: ProductSearchQuery,
  ): Promise<ProductSearchResponse> {
    return this.tracingService.withSpan(
      "catalog-service.search-products",
      async (span) => {
        this.tracingService.addSpanAttributes(span, {
          "search.query": query.term,
          "search.category": query.category,
          "search.page": query.page,
        });

        return this.circuitBreaker.fire(async () => {
          const response = await this.httpClient.post(
            "/products/search",
            query,
          );
          return response.data;
        });
      },
    );
  }

  async checkInventory(productIds: string[]): Promise<InventoryResponse[]> {
    const batchSize = 50; // Prevent large payloads
    const batches = this.chunk(productIds, batchSize);
    const results: InventoryResponse[] = [];

    for (const batch of batches) {
      const batchResult = await this.circuitBreaker.fire(async () => {
        const response = await this.httpClient.post("/inventory/check", {
          productIds: batch,
        });
        return response.data;
      });

      results.push(...batchResult);
    }

    return results;
  }

  private chunk<T>(array: T[], size: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }
}

// Service registry pattern
@Injectable()
export class ServiceRegistry {
  private readonly services = new Map<string, ServiceEndpoint>();
  private readonly healthChecks = new Map<string, HealthCheckFunction>();

  constructor(private readonly configService: ConfigService) {
    this.initializeServiceEndpoints();
    this.startHealthChecking();
  }

  private initializeServiceEndpoints(): void {
    const serviceConfig = this.configService.get("services");

    for (const [serviceName, config] of Object.entries(serviceConfig)) {
      this.services.set(serviceName, {
        name: serviceName,
        baseUrl: config.baseUrl,
        version: config.version,
        healthEndpoint: `${config.baseUrl}/health`,
        timeout: config.timeout || 5000,
        retries: config.retries || 3,
      });
    }
  }

  getService(serviceName: string): ServiceEndpoint | null {
    return this.services.get(serviceName) || null;
  }

  async isServiceHealthy(serviceName: string): Promise<boolean> {
    const service = this.services.get(serviceName);
    if (!service) {
      return false;
    }

    try {
      const response = await fetch(service.healthEndpoint, {
        method: "GET",
        timeout: service.timeout,
      });

      return response.status === 200;
    } catch (error) {
      return false;
    }
  }

  private startHealthChecking(): void {
    setInterval(async () => {
      for (const [serviceName] of this.services) {
        const isHealthy = await this.isServiceHealthy(serviceName);
        // Update service health status
        // Could emit events or update metrics here
      }
    }, 30000); // Check every 30 seconds
  }
}
```

### Message Queue Integration

```typescript
// src/infrastructure/messaging/rabbitmq.service.ts
@Injectable()
export class RabbitMQService implements MessageQueueService {
  private connection: Connection;
  private channels = new Map<string, Channel>();
  private readonly logger = new Logger(RabbitMQService.name);

  constructor(private readonly configService: ConfigService) {}

  async connect(): Promise<void> {
    try {
      const connectionUrl = this.configService.get<string>("RABBITMQ_URL");
      this.connection = await amqp.connect(connectionUrl);

      this.connection.on("error", (error) => {
        this.logger.error("RabbitMQ connection error", error);
      });

      this.connection.on("close", () => {
        this.logger.warn("RabbitMQ connection closed");
        // Implement reconnection logic
      });

      this.logger.log("Connected to RabbitMQ");
    } catch (error) {
      this.logger.error("Failed to connect to RabbitMQ", error);
      throw error;
    }
  }

  async createChannel(channelId: string): Promise<Channel> {
    if (this.channels.has(channelId)) {
      return this.channels.get(channelId)!;
    }

    const channel = await this.connection.createChannel();
    await channel.prefetch(10); // QoS

    this.channels.set(channelId, channel);
    return channel;
  }

  async publishEvent(event: DomainEvent): Promise<void> {
    const channel = await this.createChannel("events");
    const exchangeName = "domain.events";
    const routingKey = event.eventType;

    await channel.assertExchange(exchangeName, "topic", { durable: true });

    const message = JSON.stringify({
      eventId: event.eventId,
      eventType: event.eventType,
      aggregateId: event.aggregateId,
      data: event.data,
      metadata: {
        timestamp: event.occurredOn,
        version: event.version,
        correlationId: event.correlationId,
      },
    });

    await channel.publish(exchangeName, routingKey, Buffer.from(message), {
      persistent: true,
      messageId: event.eventId,
      timestamp: Date.now(),
      headers: {
        eventType: event.eventType,
        version: event.version,
      },
    });

    this.logger.debug(`Published event: ${event.eventType}`, {
      eventId: event.eventId,
      aggregateId: event.aggregateId,
    });
  }

  async subscribeToEvents(
    eventTypes: string[],
    handler: EventHandler,
    options: SubscriptionOptions = {},
  ): Promise<void> {
    const channel = await this.createChannel("subscription");
    const exchangeName = "domain.events";
    const queueName = options.queueName || `${handler.constructor.name}.queue`;

    await channel.assertExchange(exchangeName, "topic", { durable: true });
    await channel.assertQueue(queueName, {
      durable: true,
      arguments: {
        "x-dead-letter-exchange": `${exchangeName}.dlx`,
        "x-dead-letter-routing-key": "failed",
      },
    });

    for (const eventType of eventTypes) {
      await channel.bindQueue(queueName, exchangeName, eventType);
    }

    await channel.consume(queueName, async (message) => {
      if (!message) return;

      try {
        const content = JSON.parse(message.content.toString());
        const event = this.deserializeEvent(content);

        await handler.handle(event);
        channel.ack(message);

        this.logger.debug(`Processed event: ${event.eventType}`, {
          eventId: event.eventId,
          handler: handler.constructor.name,
        });
      } catch (error) {
        this.logger.error("Error processing event", error);

        // Implement retry logic
        const retryCount =
          (message.properties.headers?.["x-retry-count"] || 0) + 1;
        const maxRetries = options.maxRetries || 3;

        if (retryCount <= maxRetries) {
          await this.requeueMessage(message, retryCount);
        } else {
          channel.nack(message, false, false); // Send to DLQ
        }
      }
    });
  }

  private async requeueMessage(
    message: Message,
    retryCount: number,
  ): Promise<void> {
    // Implement exponential backoff
    const delay = Math.min(1000 * Math.pow(2, retryCount), 30000);

    setTimeout(async () => {
      const channel = await this.createChannel("retry");
      await channel.publish("", message.fields.routingKey, message.content, {
        ...message.properties,
        headers: {
          ...message.properties.headers,
          "x-retry-count": retryCount,
        },
      });
    }, delay);
  }
}

// Event handlers with EQXJS integration
@Injectable()
export class OrderEventHandler {
  constructor(
    private readonly emailService: EmailService,
    private readonly inventoryService: InventoryService,
    private readonly analyticsService: AnalyticsService,
  ) {}

  @EventPattern("order.confirmed")
  async handleOrderConfirmed(event: OrderConfirmedEvent): Promise<void> {
    await Promise.all([
      this.sendOrderConfirmationEmail(event),
      this.updateInventoryLevels(event),
      this.trackOrderAnalytics(event),
    ]);
  }

  @EventPattern("order.shipped")
  async handleOrderShipped(event: OrderShippedEvent): Promise<void> {
    await this.sendShippingNotification(event);
  }

  @EventPattern("order.cancelled")
  async handleOrderCancelled(event: OrderCancelledEvent): Promise<void> {
    await Promise.all([
      this.releaseInventoryReservation(event),
      this.processRefund(event),
      this.sendCancellationEmail(event),
    ]);
  }

  private async sendOrderConfirmationEmail(
    event: OrderConfirmedEvent,
  ): Promise<void> {
    // Implementation
  }

  private async updateInventoryLevels(
    event: OrderConfirmedEvent,
  ): Promise<void> {
    // Implementation
  }

  private async trackOrderAnalytics(event: OrderConfirmedEvent): Promise<void> {
    // Implementation
  }
}
```

---

## 🏗️ 10.3 Multi-Tenant Architecture

### Tenant Context Management

```typescript
// src/core/multi-tenancy/tenant.context.ts
@Injectable()
export class TenantContext {
  private readonly tenantId$ = new AsyncLocalStorage<string>();

  getTenantId(): string | undefined {
    return this.tenantId$.getStore();
  }

  run<T>(tenantId: string, callback: () => T): T {
    return this.tenantId$.run(tenantId, callback);
  }

  async runAsync<T>(tenantId: string, callback: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.tenantId$.run(tenantId, () => {
        callback().then(resolve).catch(reject);
      });
    });
  }
}

@Injectable()
export class TenantInterceptor implements NestInterceptor {
  constructor(
    private readonly tenantContext: TenantContext,
    private readonly tenantService: TenantService,
  ) {}

  async intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const tenantId = this.extractTenantId(request);

    if (!tenantId) {
      throw new BadRequestException("Tenant ID is required");
    }

    // Validate tenant
    const tenant = await this.tenantService.findById(tenantId);
    if (!tenant || !tenant.isActive) {
      throw new UnauthorizedException("Invalid or inactive tenant");
    }

    // Set tenant context
    return new Observable((observer) => {
      this.tenantContext.run(tenantId, () => {
        next.handle().subscribe(observer);
      });
    });
  }

  private extractTenantId(request: any): string | null {
    // Multiple extraction strategies

    // 1. From subdomain
    const host = request.get("host");
    if (host) {
      const subdomain = host.split(".")[0];
      if (subdomain && subdomain !== "www" && subdomain !== "api") {
        return subdomain;
      }
    }

    // 2. From header
    const headerTenantId = request.get("X-Tenant-ID");
    if (headerTenantId) {
      return headerTenantId;
    }

    // 3. From JWT token
    const user = request.user;
    if (user && user.tenantId) {
      return user.tenantId;
    }

    // 4. From path parameter
    const pathTenantId = request.params?.tenantId;
    if (pathTenantId) {
      return pathTenantId;
    }

    return null;
  }
}

// Tenant-aware repository pattern
@Injectable()
export class TenantAwareRepository<T extends TenantEntity> {
  constructor(
    private readonly repository: Repository<T>,
    private readonly tenantContext: TenantContext,
  ) {}

  async findAll(options?: FindManyOptions<T>): Promise<T[]> {
    const tenantId = this.requireTenantId();

    return this.repository.find({
      ...options,
      where: {
        ...options?.where,
        tenantId,
      },
    });
  }

  async findOne(id: string): Promise<T | null> {
    const tenantId = this.requireTenantId();

    return this.repository.findOne({
      where: {
        id,
        tenantId,
      } as any,
    });
  }

  async save(entity: T): Promise<T> {
    const tenantId = this.requireTenantId();
    entity.tenantId = tenantId;

    return this.repository.save(entity);
  }

  async delete(id: string): Promise<void> {
    const tenantId = this.requireTenantId();

    await this.repository.delete({
      id,
      tenantId,
    } as any);
  }

  private requireTenantId(): string {
    const tenantId = this.tenantContext.getTenantId();
    if (!tenantId) {
      throw new Error("Tenant context is required");
    }
    return tenantId;
  }
}
```

### Database Isolation Strategies

```typescript
// src/infrastructure/database/tenant-database.service.ts
@Injectable()
export class TenantDatabaseService {
  private readonly connections = new Map<string, Connection>();
  private readonly logger = new Logger(TenantDatabaseService.name);

  constructor(
    private readonly configService: ConfigService,
    private readonly tenantService: TenantService,
  ) {}

  async getConnection(tenantId: string): Promise<Connection> {
    // Return cached connection if available
    if (this.connections.has(tenantId)) {
      const connection = this.connections.get(tenantId)!;
      if (connection.isConnected) {
        return connection;
      } else {
        // Remove stale connection
        this.connections.delete(tenantId);
      }
    }

    // Create new connection for tenant
    const tenant = await this.tenantService.findById(tenantId);
    if (!tenant) {
      throw new NotFoundException(`Tenant ${tenantId} not found`);
    }

    const connectionOptions = this.buildConnectionOptions(tenant);
    const connection = await createConnection(connectionOptions);

    this.connections.set(tenantId, connection);
    this.logger.log(`Created database connection for tenant: ${tenantId}`);

    return connection;
  }

  private buildConnectionOptions(tenant: Tenant): ConnectionOptions {
    const isolationStrategy = this.configService.get<string>(
      "TENANT_ISOLATION_STRATEGY",
    );

    switch (isolationStrategy) {
      case "DATABASE_PER_TENANT":
        return this.buildDatabasePerTenantOptions(tenant);

      case "SCHEMA_PER_TENANT":
        return this.buildSchemaPerTenantOptions(tenant);

      default:
        throw new Error(`Unsupported isolation strategy: ${isolationStrategy}`);
    }
  }

  private buildDatabasePerTenantOptions(tenant: Tenant): ConnectionOptions {
    return {
      name: `tenant_${tenant.id}`,
      type: "postgres",
      host: this.configService.get<string>("DB_HOST"),
      port: this.configService.get<number>("DB_PORT"),
      username: this.configService.get<string>("DB_USERNAME"),
      password: this.configService.get<string>("DB_PASSWORD"),
      database: `tenant_${tenant.id}`,
      entities: [
        /* tenant entities */
      ],
      migrations: [
        /* tenant migrations */
      ],
      synchronize: false,
      migrationsRun: true,
      ssl: this.configService.get<boolean>("DB_SSL"),
      pool: {
        min: 1,
        max: 10,
        idle: 300000,
      },
    };
  }

  private buildSchemaPerTenantOptions(tenant: Tenant): ConnectionOptions {
    return {
      name: `tenant_${tenant.id}`,
      type: "postgres",
      host: this.configService.get<string>("DB_HOST"),
      port: this.configService.get<number>("DB_PORT"),
      username: this.configService.get<string>("DB_USERNAME"),
      password: this.configService.get<string>("DB_PASSWORD"),
      database: this.configService.get<string>("DB_NAME"),
      schema: `tenant_${tenant.id}`,
      entities: [
        /* tenant entities */
      ],
      migrations: [
        /* tenant migrations */
      ],
      synchronize: false,
      migrationsRun: true,
    };
  }

  async closeTenantConnections(): Promise<void> {
    const closePromises: Promise<void>[] = [];

    for (const [tenantId, connection] of this.connections.entries()) {
      if (connection.isConnected) {
        closePromises.push(
          connection.close().then(() => {
            this.logger.log(`Closed connection for tenant: ${tenantId}`);
          }),
        );
      }
    }

    await Promise.all(closePromises);
    this.connections.clear();
  }
}
```

---

## ☁️ 10.4 Cloud-Native Deployment Patterns

### Kubernetes Deployment Configuration

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: eqxjs-ecommerce
  labels:
    app.kubernetes.io/name: eqxjs-ecommerce
    app.kubernetes.io/part-of: microservices-platform

---
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: eqxjs-config
  namespace: eqxjs-ecommerce
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  CACHE_TTL: "300"
  HEALTH_CHECK_INTERVAL: "30"
  database.yaml: |
    database:
      type: postgres
      host: postgres-service
      port: 5432
      ssl: true
      pool:
        min: 5
        max: 50
        idle: 300000

    redis:
      host: redis-service
      port: 6379
      db: 0
      ttl: 300

    monitoring:
      metrics:
        enabled: true
        endpoint: /metrics
      tracing:
        enabled: true
        serviceName: eqxjs-ecommerce

---
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: eqxjs-secrets
  namespace: eqxjs-ecommerce
type: Opaque
stringData:
  DATABASE_URL: "postgresql://user:password@postgres-service:5432/ecommerce"
  JWT_SECRET: "your-super-secret-jwt-key-32-chars-long"
  ENCRYPTION_KEY: "your-encryption-key-32-chars-long"
  REDIS_PASSWORD: "redis-password"
  RABBITMQ_URL: "amqp://user:pass@rabbitmq:5672"

---
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eqxjs-api
  namespace: eqxjs-ecommerce
  labels:
    app: eqxjs-api
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: eqxjs-api
      version: v1
  template:
    metadata:
      labels:
        app: eqxjs-api
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: eqxjs-service-account
      containers:
        - name: api
          image: ghcr.io/company/eqxjs-ecommerce:latest
          ports:
            - containerPort: 3000
              name: http
          env:
            - name: NODE_ENV
              valueFrom:
                configMapKeyRef:
                  name: eqxjs-config
                  key: NODE_ENV
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: eqxjs-secrets
                  key: DATABASE_URL
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: eqxjs-secrets
                  key: JWT_SECRET
          volumeMounts:
            - name: config-volume
              mountPath: /app/config
              readOnly: true
            - name: logs-volume
              mountPath: /app/logs

          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /health/ready
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 2

          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "500m"

          securityContext:
            runAsNonRoot: true
            runAsUser: 1001
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

      volumes:
        - name: config-volume
          configMap:
            name: eqxjs-config
            items:
              - key: database.yaml
                path: database.yaml
        - name: logs-volume
          emptyDir: {}

      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values: ["eqxjs-api"]
                topologyKey: kubernetes.io/hostname

---
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: eqxjs-api-service
  namespace: eqxjs-ecommerce
  labels:
    app: eqxjs-api
spec:
  selector:
    app: eqxjs-api
  ports:
    - name: http
      port: 80
      targetPort: 3000
  type: ClusterIP

---
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: eqxjs-api-hpa
  namespace: eqxjs-ecommerce
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: eqxjs-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
```

### Helm Chart Configuration

```yaml
# helm/eqxjs-framework/Chart.yaml
apiVersion: v2
name: eqxjs-framework
description: A Helm chart for EQXJS Framework applications
type: application
version: 1.0.0
appVersion: "1.0.0"

dependencies:
  - name: postgresql
    version: 11.9.13
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: 17.3.7
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled

# helm/eqxjs-framework/values.yaml
# Default values for eqxjs-framework
replicaCount: 3

image:
  repository: ghcr.io/company/eqxjs-ecommerce
  pullPolicy: IfNotPresent
  tag: ""

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "3000"
  prometheus.io/path: "/metrics"

podSecurityContext:
  fsGroup: 1001

securityContext:
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1001

service:
  type: ClusterIP
  port: 80
  targetPort: 3000

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "1000"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
  hosts:
    - host: api.ecommerce.company.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.ecommerce.company.com

resources:
  limits:
    cpu: 500m
    memory: 1Gi
  requests:
    cpu: 250m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values: ["eqxjs-framework"]
          topologyKey: kubernetes.io/hostname

# EQXJS Framework specific configuration
eqxjs:
  environment: production
  logLevel: info

  database:
    enabled: true
    ssl: true
    pool:
      min: 5
      max: 50

  cache:
    enabled: true
    ttl: 300

  monitoring:
    enabled: true
    metrics: true
    tracing: true

# External dependencies
postgresql:
  enabled: true
  auth:
    database: ecommerce
    username: ecommerce_user
  primary:
    persistence:
      size: 20Gi
    resources:
      requests:
        memory: 512Mi
        cpu: 250m

redis:
  enabled: true
  auth:
    enabled: true
  master:
    persistence:
      size: 8Gi
    resources:
      requests:
        memory: 256Mi
        cpu: 100m
```

---

## 🎯 Summary

In this final module, we've explored:

✅ **Advanced Enterprise Patterns**: Hexagonal architecture and domain-driven design  
✅ **Microservices Integration**: Service communication and message queue patterns  
✅ **Multi-Tenant Architecture**: Context management and database isolation  
✅ **Cloud-Native Deployment**: Kubernetes and Helm chart configuration  
✅ **Advanced Caching Patterns**: Distributed caching and performance optimization  
✅ **Event-Driven Architecture**: Domain events and asynchronous processing

### Key Takeaways

1. **Hexagonal architecture promotes** maintainable and testable code through clear separation
2. **Event-driven patterns enable** loose coupling and scalable system design
3. **Multi-tenancy requires careful isolation** and context management strategies
4. **Cloud-native deployment** provides scalability and resilience for production systems
5. **Advanced patterns solve complex enterprise** requirements while maintaining code quality

---

## 🎓 Final Knowledge Check

Congratulations on completing the EQXJS Framework course! Ensure you understand:

- [ ] Advanced enterprise architecture patterns and their implementation
- [ ] Microservices communication strategies and integration patterns
- [ ] Multi-tenant architecture design and database isolation
- [ ] Cloud-native deployment with Kubernetes and container orchestration
- [ ] Event-driven architecture and message queue integration
- [ ] Performance optimization and advanced caching strategies

---

## 🏆 Course Completion

You have successfully completed the **EQXJS Framework Enterprise Development Course**!

### What You've Achieved

✨ **Framework Mastery**: Deep understanding of EQXJS architecture and patterns  
✨ **Enterprise Skills**: Advanced patterns for scalable, maintainable applications  
✨ **Production Readiness**: Deployment, monitoring, and operational excellence  
✨ **Advanced Integration**: Microservices, events, and cloud-native patterns

### Next Steps

📝 **Complete the final project**: [Final Project Assignment](exercise/final-project.md)  
🎖️ **Get certified**: Submit your project for EQXJS Framework certification  
💼 **Apply your skills**: Use EQXJS Framework in production applications  
🌟 **Share knowledge**: Contribute to the EQXJS community and documentation

---

## 📚 Additional Resources

- [EQXJS Framework Official Documentation](https://eqxjs.framework.docs)
- [Enterprise Architecture Patterns](https://martinfowler.com/eaaCatalog/)
- [Domain-Driven Design Reference](https://domainlanguage.com/ddd/)
- [Kubernetes Production Patterns](https://kubernetes.io/docs/concepts/)
- [Cloud Native Computing Foundation](https://www.cncf.io/)

---

**Thank you for completing the EQXJS Framework Enterprise Development Course!** 🎉

_Your journey to enterprise-grade application development starts now._
