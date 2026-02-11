# Module 9: Practical Implementation

## 📚 Learning Objectives

By the end of this module, you will understand:

- Complete EQXJS application development lifecycle
- Real-world implementation patterns and best practices
- Testing strategies for EQXJS applications
- CI/CD pipeline configuration and deployment automation
- Performance testing and optimization techniques
- Integration with external systems and APIs

## 🧭 Visual Flow (Mermaid)

```mermaid
%%{init: {'theme': 'forest'}}%%
flowchart LR
  Code["Implement features<br/>(modules/controllers/services)"] --> Unit[Unit tests]
  Unit --> IT[Integration/E2E tests]
  IT --> Build["Build artifact<br/>(tsc/nest build)"]
  Build --> Image["Containerize<br/>(Docker image)"]
  Image --> Deploy["Deploy<br/>(staging/prod)"]
  Deploy --> Obs["Observe<br/>(logs/health/metrics)"]
  Obs --> Code
```

---

## 🏗️ 9.1 Complete Application Architecture

### Enterprise E-commerce Platform Implementation

Let's build a comprehensive e-commerce platform using the EQXJS Framework:

```typescript
// src/app.module.ts
import { Module } from "@nestjs/common";
import { FrameworkModule } from "@corp-ais/eqxjs-stub";
import {
  ProductsModule,
  OrdersModule,
  UsersModule,
  PaymentModule,
  InventoryModule,
  NotificationModule,
  AnalyticsModule,
} from "./modules";

@Module({
  imports: [
    FrameworkModule.register({
      configPath: "config",
      zone: process.env.NODE_ENV || "development",
    }),

    // Core business modules
    ProductsModule,
    OrdersModule,
    UsersModule,

    // Supporting modules
    PaymentModule,
    InventoryModule,
    NotificationModule,
    AnalyticsModule,
  ],
})
export class AppModule {}
```

### Products Module Implementation

```typescript
// src/modules/products/products.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([Product, Category, Inventory]),
    CacheModule.register({
      ttl: 300, // 5 minutes
      max: 1000,
    }),
  ],
  controllers: [ProductsController],
  providers: [
    ProductsService,
    ProductsRepository,
    InventoryService,
    CacheService,
  ],
  exports: [ProductsService],
})
export class ProductsModule {}

// src/modules/products/entities/product.entity.ts
@Entity("products")
export class Product {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column({ length: 255 })
  name: string;

  @Column("text")
  description: string;

  @Column("decimal", { precision: 10, scale: 2 })
  price: number;

  @Column({ length: 100 })
  sku: string;

  @ManyToOne(() => Category, (category) => category.products)
  category: Category;

  @OneToMany(() => Inventory, (inventory) => inventory.product)
  inventory: Inventory[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @Column({ default: true })
  isActive: boolean;
}

// src/modules/products/products.controller.ts
@Controller("products")
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  constructor(
    private readonly productsService: ProductsService,
    private readonly metricsService: MetricsService,
  ) {}

  @Get()
  @Entrypoint({
    name: "list-products",
    description: "Retrieve paginated list of products",
    version: "1.0.0",
    tags: ["products", "catalog"],
  })
  @ValidateSchema({
    schema: ProductQuerySchema,
    transform: true,
  })
  @CacheTTL(300)
  async findAll(@Query() query: ProductQueryDto) {
    const startTime = Date.now();

    try {
      const result = await this.productsService.findAll(query);

      this.metricsService.recordBusinessEvent("product_list", "success");
      return result;
    } catch (error) {
      this.metricsService.recordBusinessEvent("product_list", "failure");
      throw error;
    } finally {
      const duration = Date.now() - startTime;
      this.metricsService.recordPerformance("product_list", duration);
    }
  }

  @Get(":id")
  @Entrypoint({
    name: "get-product",
    description: "Get product details by ID",
    version: "1.0.0",
    tags: ["products", "details"],
  })
  @ConsumerMasking({
    enabled: true,
    consumerTypes: ["guest", "basic-user"],
    rules: [
      {
        field: "supplierInfo",
        strategy: "remove",
      },
      {
        field: "costPrice",
        strategy: "remove",
      },
    ],
  })
  async findOne(@Param("id") id: string) {
    return this.productsService.findOne(id);
  }

  @Post()
  @Entrypoint({
    name: "create-product",
    description: "Create a new product",
    version: "1.0.0",
    authentication: { required: true, roles: ["admin", "product-manager"] },
    tags: ["products", "admin"],
  })
  @ValidateSchema({
    schema: CreateProductSchema,
    transform: true,
  })
  @Roles(["admin", "product-manager"])
  async create(@Body() createProductDto: CreateProductDto) {
    const result = await this.productsService.create(createProductDto);
    this.metricsService.recordBusinessEvent("product_created", "success");
    return result;
  }

  @Put(":id")
  @Entrypoint({
    name: "update-product",
    description: "Update product information",
    version: "1.0.0",
    authentication: { required: true, roles: ["admin", "product-manager"] },
  })
  @ValidateSchema({
    schema: UpdateProductSchema,
    transform: true,
  })
  async update(
    @Param("id") id: string,
    @Body() updateProductDto: UpdateProductDto,
  ) {
    return this.productsService.update(id, updateProductDto);
  }
}
```

### Service Implementation with Domain Logic

```typescript
// src/modules/products/products.service.ts
@Injectable()
export class ProductsService {
  constructor(
    private readonly productsRepository: ProductsRepository,
    private readonly inventoryService: InventoryService,
    private readonly cacheService: CacheService,
    private readonly eventEmitter: EventEmitter2,
    @Inject("LOGGER") private readonly logger: Logger,
  ) {}

  async findAll(query: ProductQueryDto): Promise<PaginatedResponse<Product>> {
    const cacheKey = `products:list:${JSON.stringify(query)}`;

    // Try to get from cache first
    const cached =
      await this.cacheService.get<PaginatedResponse<Product>>(cacheKey);
    if (cached) {
      this.logger.debug("Products retrieved from cache", { query });
      return cached;
    }

    const {
      page = 1,
      limit = 20,
      category,
      priceMin,
      priceMax,
      search,
    } = query;

    const queryBuilder = this.productsRepository
      .createQueryBuilder("product")
      .leftJoinAndSelect("product.category", "category")
      .leftJoinAndSelect("product.inventory", "inventory")
      .where("product.isActive = :isActive", { isActive: true });

    // Apply filters
    if (category) {
      queryBuilder.andWhere("category.slug = :category", { category });
    }

    if (priceMin) {
      queryBuilder.andWhere("product.price >= :priceMin", { priceMin });
    }

    if (priceMax) {
      queryBuilder.andWhere("product.price <= :priceMax", { priceMax });
    }

    if (search) {
      queryBuilder.andWhere(
        "(product.name ILIKE :search OR product.description ILIKE :search)",
        { search: `%${search}%` },
      );
    }

    // Apply pagination
    const [products, total] = await queryBuilder
      .orderBy("product.createdAt", "DESC")
      .skip((page - 1) * limit)
      .take(limit)
      .getManyAndCount();

    const result: PaginatedResponse<Product> = {
      data: products,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
      },
    };

    // Cache the result
    await this.cacheService.set(cacheKey, result, 300); // 5 minutes

    this.logger.info("Products retrieved from database", {
      count: products.length,
      total,
      filters: { category, priceMin, priceMax, search },
    });

    return result;
  }

  async findOne(id: string): Promise<Product> {
    const cacheKey = `product:${id}`;

    const cached = await this.cacheService.get<Product>(cacheKey);
    if (cached) {
      return cached;
    }

    const product = await this.productsRepository.findOne({
      where: { id, isActive: true },
      relations: ["category", "inventory"],
    });

    if (!product) {
      throw new NotFoundException(`Product with ID ${id} not found`);
    }

    // Cache the product
    await this.cacheService.set(cacheKey, product, 600); // 10 minutes

    return product;
  }

  async create(createProductDto: CreateProductDto): Promise<Product> {
    const { categoryId, ...productData } = createProductDto;

    // Check if SKU already exists
    const existingProduct = await this.productsRepository.findOne({
      where: { sku: productData.sku },
    });

    if (existingProduct) {
      throw new ConflictException(
        `Product with SKU ${productData.sku} already exists`,
      );
    }

    const category = await this.findCategoryOrThrow(categoryId);

    const product = this.productsRepository.create({
      ...productData,
      category,
    });

    const savedProduct = await this.productsRepository.save(product);

    // Create initial inventory record
    await this.inventoryService.create({
      productId: savedProduct.id,
      quantity: 0,
      lowStockThreshold: 10,
      location: "main-warehouse",
    });

    // Emit event for other services
    this.eventEmitter.emit("product.created", {
      productId: savedProduct.id,
      sku: savedProduct.sku,
      name: savedProduct.name,
    });

    this.logger.info("Product created successfully", {
      productId: savedProduct.id,
      sku: savedProduct.sku,
      name: savedProduct.name,
    });

    return savedProduct;
  }

  async update(
    id: string,
    updateProductDto: UpdateProductDto,
  ): Promise<Product> {
    const product = await this.findOne(id);

    if (updateProductDto.sku && updateProductDto.sku !== product.sku) {
      const existingProduct = await this.productsRepository.findOne({
        where: { sku: updateProductDto.sku },
      });

      if (existingProduct) {
        throw new ConflictException(
          `Product with SKU ${updateProductDto.sku} already exists`,
        );
      }
    }

    // Update product
    Object.assign(product, updateProductDto);
    const updatedProduct = await this.productsRepository.save(product);

    // Invalidate cache
    await this.cacheService.del(`product:${id}`);
    await this.cacheService.clear("products:list:*");

    // Emit update event
    this.eventEmitter.emit("product.updated", {
      productId: id,
      changes: updateProductDto,
    });

    this.logger.info("Product updated successfully", {
      productId: id,
      changes: Object.keys(updateProductDto),
    });

    return updatedProduct;
  }

  private async findCategoryOrThrow(categoryId: string) {
    const category = await this.categoryRepository.findOne({
      where: { id: categoryId },
    });

    if (!category) {
      throw new NotFoundException(`Category with ID ${categoryId} not found`);
    }

    return category;
  }
}
```

---

## 🧪 9.2 Comprehensive Testing Strategy

### Unit Testing with Jest

```typescript
// src/modules/products/products.service.spec.ts
describe("ProductsService", () => {
  let service: ProductsService;
  let repository: Repository<Product>;
  let cacheService: CacheService;
  let eventEmitter: EventEmitter2;

  const mockProduct: Product = {
    id: "test-id",
    name: "Test Product",
    description: "Test Description",
    price: 29.99,
    sku: "TEST-001",
    category: { id: "cat-1", name: "Category 1" } as any,
    inventory: [],
    createdAt: new Date(),
    updatedAt: new Date(),
    isActive: true,
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ProductsService,
        {
          provide: getRepositoryToken(Product),
          useClass: Repository,
        },
        {
          provide: CacheService,
          useValue: {
            get: jest.fn(),
            set: jest.fn(),
            del: jest.fn(),
            clear: jest.fn(),
          },
        },
        {
          provide: EventEmitter2,
          useValue: {
            emit: jest.fn(),
          },
        },
        {
          provide: "LOGGER",
          useValue: {
            info: jest.fn(),
            debug: jest.fn(),
            warn: jest.fn(),
            error: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<ProductsService>(ProductsService);
    repository = module.get<Repository<Product>>(getRepositoryToken(Product));
    cacheService = module.get<CacheService>(CacheService);
    eventEmitter = module.get<EventEmitter2>(EventEmitter2);
  });

  describe("findOne", () => {
    it("should return cached product if available", async () => {
      const productId = "test-id";
      jest.spyOn(cacheService, "get").mockResolvedValue(mockProduct);

      const result = await service.findOne(productId);

      expect(cacheService.get).toHaveBeenCalledWith(`product:${productId}`);
      expect(repository.findOne).not.toHaveBeenCalled();
      expect(result).toEqual(mockProduct);
    });

    it("should fetch from database if not cached", async () => {
      const productId = "test-id";
      jest.spyOn(cacheService, "get").mockResolvedValue(null);
      jest.spyOn(repository, "findOne").mockResolvedValue(mockProduct);

      const result = await service.findOne(productId);

      expect(repository.findOne).toHaveBeenCalledWith({
        where: { id: productId, isActive: true },
        relations: ["category", "inventory"],
      });
      expect(cacheService.set).toHaveBeenCalledWith(
        `product:${productId}`,
        mockProduct,
        600,
      );
      expect(result).toEqual(mockProduct);
    });

    it("should throw NotFoundException if product not found", async () => {
      const productId = "non-existent";
      jest.spyOn(cacheService, "get").mockResolvedValue(null);
      jest.spyOn(repository, "findOne").mockResolvedValue(null);

      await expect(service.findOne(productId)).rejects.toThrow(
        NotFoundException,
      );
    });
  });

  describe("create", () => {
    const createProductDto: CreateProductDto = {
      name: "New Product",
      description: "New Description",
      price: 39.99,
      sku: "NEW-001",
      categoryId: "cat-1",
    };

    it("should create product successfully", async () => {
      jest.spyOn(repository, "findOne").mockResolvedValue(null); // SKU not exists
      jest
        .spyOn(service as any, "findCategoryOrThrow")
        .mockResolvedValue({ id: "cat-1" });
      jest.spyOn(repository, "create").mockReturnValue(mockProduct);
      jest.spyOn(repository, "save").mockResolvedValue(mockProduct);

      const result = await service.create(createProductDto);

      expect(repository.save).toHaveBeenCalled();
      expect(eventEmitter.emit).toHaveBeenCalledWith("product.created", {
        productId: mockProduct.id,
        sku: mockProduct.sku,
        name: mockProduct.name,
      });
      expect(result).toEqual(mockProduct);
    });

    it("should throw ConflictException if SKU already exists", async () => {
      jest.spyOn(repository, "findOne").mockResolvedValue(mockProduct);

      await expect(service.create(createProductDto)).rejects.toThrow(
        ConflictException,
      );
    });
  });
});
```

### Integration Testing

```typescript
// test/products.e2e-spec.ts
describe("ProductsController (e2e)", () => {
  let app: INestApplication;
  let httpService: HttpService;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [
        AppModule,
        TypeOrmModule.forRoot({
          type: "postgres",
          host: process.env.TEST_DB_HOST || "localhost",
          port: parseInt(process.env.TEST_DB_PORT, 10) || 5432,
          username: process.env.TEST_DB_USER || "test",
          password: process.env.TEST_DB_PASS || "test",
          database: process.env.TEST_DB_NAME || "ecommerce_test",
          autoLoadEntities: true,
          synchronize: true,
        }),
      ],
    }).compile();

    app = moduleFixture.createNestApplication();

    // Configure test app
    app.useGlobalPipes(new ValidationPipe());
    app.useGlobalFilters(new AllExceptionsFilter());

    await app.init();
    httpService = app.get<HttpService>(HttpService);
  });

  afterAll(async () => {
    await app.close();
  });

  describe("/products (GET)", () => {
    it("should return paginated products", () => {
      return request(app.getHttpServer())
        .get("/products")
        .query({ page: 1, limit: 10 })
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty("data");
          expect(res.body).toHaveProperty("meta");
          expect(Array.isArray(res.body.data)).toBe(true);
          expect(res.body.meta).toHaveProperty("total");
          expect(res.body.meta).toHaveProperty("page");
          expect(res.body.meta).toHaveProperty("limit");
        });
    });

    it("should filter products by category", () => {
      return request(app.getHttpServer())
        .get("/products")
        .query({ category: "electronics" })
        .expect(200)
        .expect((res) => {
          res.body.data.forEach((product) => {
            expect(product.category.slug).toBe("electronics");
          });
        });
    });

    it("should filter products by price range", () => {
      return request(app.getHttpServer())
        .get("/products")
        .query({ priceMin: 10, priceMax: 50 })
        .expect(200)
        .expect((res) => {
          res.body.data.forEach((product) => {
            expect(product.price).toBeGreaterThanOrEqual(10);
            expect(product.price).toBeLessThanOrEqual(50);
          });
        });
    });
  });

  describe("/products/:id (GET)", () => {
    it("should return product details", async () => {
      // First create a product
      const createResponse = await request(app.getHttpServer())
        .post("/products")
        .send({
          name: "Test Product",
          description: "Test Description",
          price: 29.99,
          sku: "TEST-001",
          categoryId: "category-id",
        })
        .set("Authorization", "Bearer admin-token");

      const productId = createResponse.body.id;

      return request(app.getHttpServer())
        .get(`/products/${productId}`)
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty("id", productId);
          expect(res.body).toHaveProperty("name", "Test Product");
          expect(res.body).toHaveProperty("price", 29.99);
        });
    });

    it("should return 404 for non-existent product", () => {
      return request(app.getHttpServer())
        .get("/products/non-existent-id")
        .expect(404);
    });
  });

  describe("/products (POST)", () => {
    it("should create product with admin role", () => {
      return request(app.getHttpServer())
        .post("/products")
        .send({
          name: "New Product",
          description: "Product Description",
          price: 49.99,
          sku: "NEW-001",
          categoryId: "category-id",
        })
        .set("Authorization", "Bearer admin-token")
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty("id");
          expect(res.body).toHaveProperty("name", "New Product");
          expect(res.body).toHaveProperty("sku", "NEW-001");
        });
    });

    it("should return 401 without authentication", () => {
      return request(app.getHttpServer())
        .post("/products")
        .send({
          name: "Unauthorized Product",
          price: 19.99,
        })
        .expect(401);
    });

    it("should return 400 for invalid data", () => {
      return request(app.getHttpServer())
        .post("/products")
        .send({
          name: "", // Invalid empty name
          price: -10, // Invalid negative price
        })
        .set("Authorization", "Bearer admin-token")
        .expect(400);
    });
  });
});
```

### Performance Testing

```typescript
// test/performance/products-load.test.ts
import { performance } from 'perf_hooks';

describe('Products Performance Tests', () => {
  let app: INestApplication;

  beforeAll(async () => {
    // Setup test app
  });

  describe('Product Listing Performance', () => {
    it('should handle high concurrent requests', async () => {
      const concurrentRequests = 100;
      const requests: Promise<any>[] = [];

      const startTime = performance.now();

      for (let i = 0; i < concurrentRequests; i++) {
        requests.push(
          request(app.getHttpServer())
            .get('/products')
            .query({ page: Math.floor(Math.random() * 10) + 1 })
            .expect(200)
        );
      }

      const responses = await Promise.all(requests);
      const endTime = performance.now();
      const totalTime = endTime - startTime;

      console.log(`${concurrentRequests} concurrent requests completed in ${totalTime.toFixed(2)}ms`);
      console.log(`Average response time: ${(totalTime / concurrentRequests).toFixed(2)}ms`);

      expect(responses.length).toBe(concurrentRequests);
      expect(totalTime).toBeLessThan(5000); // Should complete within 5 seconds
    });

    it('should maintain performance with large datasets', async () => {
      // Create large dataset
      const batchSize = 1000;
      await this.createProductBatch(batchSize);

      const startTime = performance.now();

      const response = await request(app.getHttpServer())
        .get('/products')
        .query({ limit: 100 })
        .expect(200);

      const endTime = performance.now();
      const responseTime = endTime - startTime;

      console.log(`Query with ${batchSize} products completed in ${responseTime.toFixed(2)}ms`);

      expect(response.body.data.length).toBe(100);
      expect(responseTime).toBeLessThan(500); // Should complete within 500ms
    });
  });

  describe('Cache Performance', () => {
    it('should serve cached responses faster than database queries', async () => {
      const productId = 'test-product-id';

      // First request (database query)
      const dbStartTime = performance.now();
      await request(app.getHttpServer())
        .get(`/products/${productId}`)
        .expect(200);
      const dbEndTime = performance.now();
      const dbTime = dbEndTime - dbStartTime;

      // Second request (cached)
      const cacheStartTime = performance.now();
      await request(app.getHttpServer())
        .get(`/products/${productId}`)
        .expect(200);
      const cacheEndTime = performance.now();
      const cacheTime = cacheEndTime - cacheStartTime;

      console.log(`Database query time: ${dbTime.toFixed(2)}ms`);
      console.log(`Cache query time: ${cacheTime.toFixed(2)}ms`);

      expect(cacheTime).toBeLessThan(dbTime * 0.5); // Cache should be at least 50% faster
    });
  });

  private async createProductBatch(count: number) {
    const batch: any[] = [];
    for (let i = 0; i < count; i++) {
      batch.push({
        name: `Product ${i}`,
        description: `Description for product ${i}`,
        price: Math.random() * 100,
        sku: `BATCH-${i.toString().padStart(6, '0')}`,
        categoryId: 'default-category'
      });
    }

    // Batch insert products
    const repository = app.get<Repository<Product>>(getRepositoryToken(Product));
    await repository.save(batch);
  }
});
```

---

## 🔄 9.3 CI/CD Pipeline Configuration

### GitHub Actions Workflow

```yaml
# .github/workflows/ci-cd.yml
name: EQXJS E-commerce CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: "18"
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Test Suite
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: ecommerce_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:6
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run unit tests
        run: npm run test:unit
        env:
          NODE_ENV: test

      - name: Run integration tests
        run: npm run test:e2e
        env:
          NODE_ENV: test
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/ecommerce_test
          REDIS_URL: redis://localhost:6379

      - name: Run performance tests
        run: npm run test:performance
        env:
          NODE_ENV: test
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/ecommerce_test

      - name: Generate coverage report
        run: npm run test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: test

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run security audit
        run: npm audit --audit-level moderate

      - name: Run vulnerability scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=medium

      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/nodejs

  build:
    name: Build and Push Image
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'

    environment:
      name: staging
      url: https://staging.ecommerce.company.com

    steps:
      - name: Deploy to staging
        uses: azure/webapps-deploy@v2
        with:
          app-name: ecommerce-staging
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE_STAGING }}
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:develop

      - name: Run smoke tests
        run: |
          curl -f https://staging.ecommerce.company.com/health || exit 1
          curl -f https://staging.ecommerce.company.com/metrics || exit 1

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'

    environment:
      name: production
      url: https://ecommerce.company.com

    steps:
      - name: Deploy to production
        uses: azure/webapps-deploy@v2
        with:
          app-name: ecommerce-production
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE_PRODUCTION }}
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

      - name: Run production smoke tests
        run: |
          curl -f https://ecommerce.company.com/health || exit 1
          curl -f https://ecommerce.company.com/metrics || exit 1

      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: "#deployments"
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Docker Configuration

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:18-alpine AS runtime

# Create app directory
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nestjs -u 1001

# Copy built application
COPY --from=builder /app/node_modules ./node_modules
COPY --chown=nestjs:nodejs . .

# Build application
RUN npm run build

# Remove dev dependencies
RUN npm prune --production

# Set security-related environment variables
ENV NODE_ENV=production
ENV NPM_CONFIG_UPDATE_NOTIFIER=false
ENV NPM_CONFIG_FUND=false

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Switch to non-root user
USER nestjs

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "dist/main.js"]
```

---

## 📈 9.4 Monitoring and Observability Implementation

### Application Metrics Dashboard

```typescript
// src/modules/monitoring/monitoring.controller.ts
@Controller("monitoring")
export class MonitoringController {
  constructor(
    private readonly metricsService: MetricsService,
    private readonly healthService: HealthService,
  ) {}

  @Get("health")
  @HealthCheck()
  async getHealth() {
    return this.healthService.check([
      () => this.healthService.checkDatabase("database"),
      () => this.healthService.checkRedis("redis"),
      () => this.healthService.checkMemoryHeap("memory_heap", 150),
      () =>
        this.healthService.checkDiskSpace("disk", {
          path: "/",
          threshold: 0.8,
        }),
    ]);
  }

  @Get("metrics")
  async getMetrics() {
    return this.metricsService.getPrometheusMetrics();
  }

  @Get("dashboard")
  async getDashboard() {
    const [systemHealth, businessMetrics, performanceMetrics, errorMetrics] =
      await Promise.all([
        this.getSystemHealth(),
        this.getBusinessMetrics(),
        this.getPerformanceMetrics(),
        this.getErrorMetrics(),
      ]);

    return {
      timestamp: new Date().toISOString(),
      system: systemHealth,
      business: businessMetrics,
      performance: performanceMetrics,
      errors: errorMetrics,
    };
  }

  private async getSystemHealth() {
    const memoryUsage = process.memoryUsage();
    const cpuUsage = process.cpuUsage();

    return {
      uptime: process.uptime(),
      memory: {
        used: Math.round(memoryUsage.heapUsed / 1024 / 1024),
        total: Math.round(memoryUsage.heapTotal / 1024 / 1024),
        external: Math.round(memoryUsage.external / 1024 / 1024),
      },
      cpu: {
        user: cpuUsage.user,
        system: cpuUsage.system,
      },
      nodeVersion: process.version,
      platform: process.platform,
    };
  }

  private async getBusinessMetrics() {
    return {
      totalProducts: await this.metricsService.getCounter("products_total"),
      totalOrders: await this.metricsService.getCounter("orders_total"),
      activeUsers: await this.metricsService.getGauge("active_users"),
      revenue: await this.metricsService.getCounter("revenue_total"),
      conversionRate: await this.calculateConversionRate(),
    };
  }

  private async getPerformanceMetrics() {
    return {
      averageResponseTime: await this.metricsService.getHistogramValue(
        "http_request_duration",
        "mean",
      ),
      throughput: await this.metricsService.getCounter(
        "http_requests_per_second",
      ),
      errorRate: await this.calculateErrorRate(),
      cacheHitRate: await this.calculateCacheHitRate(),
    };
  }

  private async getErrorMetrics() {
    return {
      totalErrors: await this.metricsService.getCounter("errors_total"),
      errorsByType: await this.metricsService.getCounterByLabels(
        "errors_total",
        "error_type",
      ),
      criticalErrors: await this.metricsService.getCounter(
        "critical_errors_total",
      ),
      lastError: await this.getLastErrorInfo(),
    };
  }
}
```

### Custom Alerts System

```typescript
// src/modules/monitoring/alerts.service.ts
@Injectable()
export class AlertsService {
  private readonly logger = new Logger(AlertsService.name);
  private alertRules: AlertRule[] = [];

  constructor(
    private readonly metricsService: MetricsService,
    private readonly notificationService: NotificationService,
  ) {
    this.initializeDefaultAlerts();
  }

  private initializeDefaultAlerts() {
    this.alertRules = [
      {
        name: "high_memory_usage",
        condition: () => this.checkMemoryUsage(),
        threshold: 85,
        severity: "warning",
        cooldown: 300000, // 5 minutes
        description: "Memory usage is above 85%",
      },
      {
        name: "high_error_rate",
        condition: () => this.checkErrorRate(),
        threshold: 5,
        severity: "critical",
        cooldown: 60000, // 1 minute
        description: "Error rate is above 5%",
      },
      {
        name: "slow_response_time",
        condition: () => this.checkResponseTime(),
        threshold: 2000,
        severity: "warning",
        cooldown: 180000, // 3 minutes
        description: "Average response time is above 2 seconds",
      },
      {
        name: "database_connection_failure",
        condition: () => this.checkDatabaseHealth(),
        threshold: 1,
        severity: "critical",
        cooldown: 30000, // 30 seconds
        description: "Database connection is failing",
      },
    ];

    // Start monitoring
    this.startAlertMonitoring();
  }

  private startAlertMonitoring() {
    setInterval(async () => {
      for (const rule of this.alertRules) {
        try {
          await this.evaluateAlertRule(rule);
        } catch (error) {
          this.logger.error(
            `Failed to evaluate alert rule ${rule.name}`,
            error,
          );
        }
      }
    }, 30000); // Check every 30 seconds
  }

  private async evaluateAlertRule(rule: AlertRule) {
    const currentValue = await rule.condition();
    const isTriggered = currentValue > rule.threshold;

    if (isTriggered && this.shouldTriggerAlert(rule)) {
      await this.triggerAlert(rule, currentValue);
    }
  }

  private shouldTriggerAlert(rule: AlertRule): boolean {
    const lastTriggered = this.getLastTriggeredTime(rule.name);
    const now = Date.now();

    return !lastTriggered || now - lastTriggered > rule.cooldown;
  }

  private async triggerAlert(rule: AlertRule, currentValue: number) {
    const alert: Alert = {
      id: uuidv4(),
      name: rule.name,
      severity: rule.severity,
      description: rule.description,
      currentValue,
      threshold: rule.threshold,
      triggeredAt: new Date(),
      status: "active",
    };

    this.logger.warn(`Alert triggered: ${rule.name}`, {
      severity: rule.severity,
      currentValue,
      threshold: rule.threshold,
    });

    // Send notifications
    await this.sendAlertNotifications(alert);

    // Store alert
    await this.storeAlert(alert);

    // Update last triggered time
    this.setLastTriggeredTime(rule.name, Date.now());
  }

  private async sendAlertNotifications(alert: Alert) {
    const channels = this.getNotificationChannels(alert.severity);

    for (const channel of channels) {
      try {
        await this.notificationService.send(channel, {
          title: `🚨 Alert: ${alert.name}`,
          message: `${alert.description}\nCurrent value: ${alert.currentValue}\nThreshold: ${alert.threshold}`,
          severity: alert.severity,
          timestamp: alert.triggeredAt,
        });
      } catch (error) {
        this.logger.error(`Failed to send alert to ${channel}`, error);
      }
    }
  }

  private getNotificationChannels(severity: AlertSeverity): string[] {
    switch (severity) {
      case "critical":
        return ["slack", "email", "sms"];
      case "warning":
        return ["slack", "email"];
      case "info":
        return ["slack"];
      default:
        return ["slack"];
    }
  }

  // Alert condition methods
  private async checkMemoryUsage(): Promise<number> {
    const memoryUsage = process.memoryUsage();
    return (memoryUsage.heapUsed / memoryUsage.heapTotal) * 100;
  }

  private async checkErrorRate(): Promise<number> {
    const totalRequests = await this.metricsService.getCounter(
      "http_requests_total",
    );
    const errorRequests =
      await this.metricsService.getCounter("http_errors_total");

    return totalRequests > 0 ? (errorRequests / totalRequests) * 100 : 0;
  }

  private async checkResponseTime(): Promise<number> {
    return await this.metricsService.getHistogramValue(
      "http_request_duration",
      "mean",
    );
  }

  private async checkDatabaseHealth(): Promise<number> {
    try {
      // Simple database health check
      const result = await this.healthService.checkDatabase();
      return result.status === "up" ? 0 : 1;
    } catch (error) {
      return 1;
    }
  }
}

interface AlertRule {
  name: string;
  condition: () => Promise<number>;
  threshold: number;
  severity: AlertSeverity;
  cooldown: number;
  description: string;
}

interface Alert {
  id: string;
  name: string;
  severity: AlertSeverity;
  description: string;
  currentValue: number;
  threshold: number;
  triggeredAt: Date;
  status: "active" | "resolved";
}

type AlertSeverity = "info" | "warning" | "critical";
```

---

## 🎯 Summary

In this module, we've covered:

✅ **Complete Application Architecture**: Full-scale e-commerce platform implementation  
✅ **Comprehensive Testing Strategy**: Unit, integration, and performance testing  
✅ **CI/CD Pipeline Configuration**: Automated testing, security, and deployment  
✅ **Production Deployment**: Docker containerization and multi-environment deployment  
✅ **Monitoring & Observability**: Real-time metrics, alerts, and dashboards  
✅ **Real-world Integration Patterns**: External APIs, payment processing, and notifications

### Key Takeaways

1. **Layered architecture ensures maintainability** and scalability of enterprise applications
2. **Comprehensive testing strategy** catches issues early and ensures quality
3. **Automated CI/CD pipelines** enable reliable and frequent deployments
4. **Production monitoring** provides insights into application performance and health
5. **Real-world integration patterns** address common enterprise requirements

---

## 🎓 Knowledge Check

Before proceeding to Module 10, ensure you understand:

- [ ] Complete application architecture and module organization
- [ ] Testing strategies and implementation patterns
- [ ] CI/CD pipeline configuration and deployment automation
- [ ] Production monitoring and alerting systems
- [ ] Performance optimization and scalability considerations
- [ ] Security implementation and best practices

---

## ➡️ Next Steps

👉 **Continue to [Module 10: Advanced Patterns & Integration](module-10-advanced-patterns.md)**

📝 **Complete the exercises**: [Module 9 Exercises](exercise/module-09-exercises.md)

---

## 📚 Additional Resources

- [NestJS Testing Documentation](https://docs.nestjs.com/fundamentals/testing)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Prometheus Monitoring](https://prometheus.io/docs/introduction/overview/)
- [Jest Testing Framework](https://jestjs.io/docs/getting-started)
