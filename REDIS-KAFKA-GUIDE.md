# 🚀 REDIS & KAFKA - KHI NÀO CẦN VÀ CÁCH SỬ DỤNG

## 📖 MỤC LỤC

1. [Redis là gì? Khi nào dùng?](#redis)
2. [Kafka là gì? Khi nào dùng?](#kafka)
3. [So sánh Redis vs Kafka](#compare)
4. [Use Cases cho Tarot E-commerce](#usecases)
5. [Setup Guide](#setup)
6. [Practice Roadmap](#practice)

---

## 🔴 REDIS - IN-MEMORY DATA STORE {#redis}

### **Redis là gì?**

**Redis** = Remote Dictionary Server
- In-memory key-value database
- CỰC KỲ NHANH (microseconds response time)
- Data lưu trong RAM (không phải disk)
- Support nhiều data structures

**Key Features:**
- ⚡ Speed: 100,000+ operations/second
- 💾 Persistence: Có thể save to disk
- 🔄 Pub/Sub: Message broadcasting
- ⏰ TTL: Auto-expire keys
- 🔐 Atomic operations

---

### **Khi nào CẦN dùng Redis?**

#### **1. CACHING (80% use case)**

**Vấn đề:**
```
User request → NestJS → PostgreSQL → Return
                         ↑ Slow (50-100ms)
                         
Nếu 1000 users cùng request → 1000 queries → Database overload!
```

**Giải pháp:**
```
Lần 1: User → NestJS → PostgreSQL → Cache vào Redis → Return
Lần 2: User → NestJS → Redis → Return (5ms - NHANH GẤP 10 LẦN!)
                        ↑ Fast!
```

**Use Cases cho Tarot website:**

**a. Cache Product List**
```typescript
// Không có cache (chậm)
async getProducts() {
  return this.prisma.product.findMany(); // 50ms mỗi request
}

// Có cache (nhanh)
async getProducts() {
  // Check Redis first
  const cached = await this.redis.get('products:all');
  if (cached) return JSON.parse(cached); // 5ms - NHANH!
  
  // Nếu không có cache, query DB
  const products = await this.prisma.product.findMany();
  
  // Cache vào Redis (expire sau 5 phút)
  await this.redis.setex('products:all', 300, JSON.stringify(products));
  
  return products;
}
```

**b. Cache User Session**
```typescript
// Lưu session info
await redis.set(`session:${userId}`, JSON.stringify(userData), 'EX', 3600);

// Get session (không query DB!)
const session = await redis.get(`session:${userId}`);
```

**c. Cache Featured Products (Landing Page)**
```typescript
// Featured products ít thay đổi → cache 1 giờ
const key = 'landing:featured-products';
const cached = await redis.get(key);
if (cached) return JSON.parse(cached);

const products = await prisma.product.findMany({
  where: { isFeatured: true },
  take: 5,
});

await redis.setex(key, 3600, JSON.stringify(products));
return products;
```

---

#### **2. RATE LIMITING**

**Vấn đề:**
User spam API → Server overload

**Giải pháp:**
```typescript
const key = `rate-limit:${userId}:${endpoint}`;
const count = await redis.incr(key);

if (count === 1) {
  await redis.expire(key, 60); // Expire sau 60s
}

if (count > 10) {
  throw new TooManyRequestsException('Max 10 requests per minute');
}
```

---

#### **3. SESSION MANAGEMENT**

```typescript
// Lưu JWT session
await redis.setex(`jwt:${token}`, 604800, userId); // 7 days

// Logout - invalidate token
await redis.del(`jwt:${token}`);
```

---

#### **4. REAL-TIME FEATURES**

**a. View Count**
```typescript
// Increment view count
await redis.incr(`product:${productId}:views`);

// Get views
const views = await redis.get(`product:${productId}:views`);
```

**b. Cart (Guest Users)**
```typescript
// Save cart cho guest (chưa login)
const cartKey = `cart:guest:${sessionId}`;
await redis.setex(cartKey, 86400, JSON.stringify(cartItems)); // 1 day
```

**c. Trending Products (Real-time)**
```typescript
// Sorted Set - track popular products
await redis.zincrby('trending:products', 1, productId);

// Get top 10 trending
const trending = await redis.zrevrange('trending:products', 0, 9);
```

---

#### **5. JOB QUEUE (với Bull)**

```typescript
// Add job vào queue
await emailQueue.add('welcome-email', {
  email: user.email,
  name: user.name,
});

// Process jobs
emailQueue.process('welcome-email', async (job) => {
  await sendEmail(job.data);
});
```

---

### **Khi NÀO CHƯA CẦN Redis?**

❌ **Không cần nếu:**
- Traffic thấp (<100 users/day)
- Chưa có performance issues
- Database queries đủ nhanh
- Chưa có real-time features

✅ **Nên dùng khi:**
- Traffic cao (>1000 users/day)
- Queries chậm (>100ms)
- Cần caching
- Có real-time features
- Cần rate limiting

---

## 🟢 KAFKA - EVENT STREAMING PLATFORM {#kafka}

### **Kafka là gì?**

**Apache Kafka** = Distributed Event Streaming Platform
- Xử lý millions events/second
- Message queue phân tán
- Real-time data pipelines
- Event-driven architecture

**Key Concepts:**
- **Producer**: Gửi messages (events)
- **Consumer**: Nhận messages
- **Topic**: Channel/category của messages
- **Partition**: Chia topic thành nhiều phần (parallelism)
- **Broker**: Kafka server instance

---

### **Khi nào CẦN dùng Kafka?**

#### **1. EVENT-DRIVEN ARCHITECTURE**

**Vấn đề với Monolith:**
```
User đặt hàng
  ↓
OrderService làm TẤT CẢ:
  - Tạo order
  - Giảm stock
  - Send email
  - Update analytics
  - Create invoice
  - Notify shipping
  ↓ CHẬM! (5-10 seconds)
```

**Giải pháp với Kafka:**
```
User đặt hàng
  ↓
OrderService:
  - Tạo order
  - Publish event: "ORDER_CREATED"
  ↓ NHANH! (100ms) - Return ngay
  
Kafka Topic: "order-events"
  ↓
  ├─> EmailService: Send confirmation email
  ├─> InventoryService: Giảm stock
  ├─> AnalyticsService: Update stats
  ├─> InvoiceService: Create invoice
  └─> ShippingService: Create shipping label
  
(Chạy async, không block user!)
```

---

#### **2. MICROSERVICES COMMUNICATION**

**Architecture:**
```
┌──────────────┐      ┌──────────────┐
│ Order Service│──┐   │ Email Service│
└──────────────┘  │   └──────────────┘
                  │
┌──────────────┐  │   ┌──────────────┐
│Product Service──┼───│  KAFKA       │
└──────────────┘  │   └──────────────┘
                  │
┌──────────────┐  │   ┌──────────────┐
│Payment Service──┘   │Analytics Svc │
└──────────────┘      └──────────────┘
```

**Events flow:**
```
ProductService: Product.Created → Kafka
  ↓
AnalyticsService: Update product count
SearchService: Index new product
RecommendationService: Update recommendations
```

---

#### **3. DATA PIPELINE & ANALYTICS**

```
E-commerce Events
  ├─> User.Registered
  ├─> Product.Viewed
  ├─> Product.AddedToCart
  ├─> Order.Created
  └─> Payment.Completed
      ↓
Kafka Topics
      ↓
  ├─> Analytics Service (Real-time dashboard)
  ├─> Data Warehouse (BigQuery, Snowflake)
  ├─> Recommendation Engine
  └─> Marketing Automation
```

---

#### **4. AUDIT LOGS**

```typescript
// Every action publish event
await kafka.send({
  topic: 'audit-logs',
  messages: [{
    key: userId,
    value: JSON.stringify({
      action: 'USER_UPDATED',
      userId,
      changes: { name: 'old' → 'new' },
      timestamp: new Date(),
      ip: request.ip,
    }),
  }],
});

// Consumer lưu vào database riêng
// Không ảnh hưởng main application performance
```

---

### **Khi NÀO CHƯA CẦN Kafka?**

❌ **Không cần nếu:**
- Monolith application (1 service)
- Traffic thấp
- Không có microservices
- Không cần event sourcing
- Team nhỏ (1-2 người)

✅ **Nên dùng khi:**
- Microservices architecture (>3 services)
- Traffic cao (>10,000 users/day)
- Cần asynchronous processing
- Event-driven architecture
- Scale horizontally

---

## ⚖️ SO SÁNH REDIS VS KAFKA {#compare}

| Feature | Redis | Kafka |
|---------|-------|-------|
| **Mục đích chính** | Caching, Session | Event Streaming |
| **Speed** | Cực nhanh (μs) | Nhanh (ms) |
| **Data Persistence** | Optional | Yes (log-based) |
| **Message Order** | Không đảm bảo | Đảm bảo (trong partition) |
| **Retention** | TTL-based | Time/Size-based |
| **Complexity** | Đơn giản | Phức tạp |
| **Use Cases** | Cache, Session, Queue | Event streaming, Microservices |
| **Learning Curve** | Dễ (1-2 ngày) | Khó (1-2 tuần) |
| **Setup** | Đơn giản | Phức tạp (cần Zookeeper) |
| **Khi nào dùng** | Ngay từ đầu | Khi scale lớn |

---

## 🎯 USE CASES CHO TAROT E-COMMERCE {#usecases}

### **PHASE 1: MVP (Không cần Redis/Kafka)**

**Tech Stack:**
- NestJS + PostgreSQL
- Simple, monolith
- Traffic: <1000 users/day

**Features:**
- ✅ Users CRUD
- ✅ Products CRUD
- ✅ Orders CRUD
- ✅ Basic authentication

---

### **PHASE 2: Thêm Redis (Khi có 1000-10000 users/day)**

#### **Use Case 1: Cache Featured Products**

**Nghiệp vụ:**
- Landing page có 5 featured products
- Products này ít thay đổi
- Nhiều users xem cùng lúc

**Implementation:**
```typescript
@Injectable()
export class ProductsService {
  constructor(
    private prisma: PrismaService,
    private redis: RedisService,
  ) {}

  async getFeaturedProducts() {
    const cacheKey = 'featured-products';
    
    // Check cache
    const cached = await this.redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    // Query DB
    const products = await this.prisma.product.findMany({
      where: { isFeatured: true },
      take: 5,
    });
    
    // Cache 1 giờ
    await this.redis.setex(cacheKey, 3600, JSON.stringify(products));
    
    return products;
  }
}
```

**Lợi ích:**
- Response time: 100ms → 5ms (nhanh gấp 20 lần)
- Giảm load database 95%

---

#### **Use Case 2: Shopping Cart cho Guest Users**

**Nghiệp vụ:**
- User chưa login nhưng muốn add to cart
- Cart phải persist khi refresh page
- Không muốn lưu vào database (chưa có user)

**Implementation:**
```typescript
// Add to cart
const sessionId = req.cookies.sessionId;
const cartKey = `cart:guest:${sessionId}`;

const cart = await redis.get(cartKey) || '[]';
const items = JSON.parse(cart);
items.push({ productId, quantity });

await redis.setex(cartKey, 86400, JSON.stringify(items)); // 24h
```

**Lợi ích:**
- Không spam database với temporary carts
- Fast access
- Auto-cleanup (expire sau 24h)

---

#### **Use Case 3: Rate Limiting API**

**Nghiệp vụ:**
- Prevent spam/abuse
- Limit 100 requests/minute per user

**Implementation:**
```typescript
const key = `rate-limit:${userId}:${endpoint}`;
const requests = await redis.incr(key);

if (requests === 1) {
  await redis.expire(key, 60);
}

if (requests > 100) {
  throw new TooManyRequestsException();
}
```

---

#### **Use Case 4: Session Store (JWT Alternative)**

**Nghiệp vụ:**
- Lưu user session
- Fast logout (invalidate session)
- Track active users

**Implementation:**
```typescript
// Login - create session
const sessionId = uuid();
await redis.setex(
  `session:${sessionId}`,
  604800, // 7 days
  JSON.stringify({ userId, email, role })
);

// Get session
const session = await redis.get(`session:${sessionId}`);

// Logout - invalidate
await redis.del(`session:${sessionId}`);
```

---

#### **Use Case 5: Bull Queue (Background Jobs)**

**Nghiệp vụ:**
- Send email không block request
- Process payment asynchronously
- Generate reports in background

**Implementation:**
```typescript
// Add job to queue (Redis-backed)
await this.emailQueue.add('welcome-email', {
  userId: user.id,
  email: user.email,
});

// Return ngay - không đợi email send
return { success: true };

// Worker process job
@Process('welcome-email')
async sendWelcomeEmail(job: Job) {
  await this.emailService.send(job.data);
}
```

---

### **Khi nào thêm Redis vào Tarot Project?**

**Thêm Redis KHI:**
- ✅ Traffic >1000 users/day
- ✅ Database queries chậm
- ✅ Có features cần real-time (cart, views)
- ✅ Cần background jobs (email, reports)
- ✅ Cần rate limiting

**Priority thêm vào:**
1. **Phase 2.1:** Cache featured products (landing page)
2. **Phase 2.2:** Guest cart với Redis
3. **Phase 2.3:** Bull queue cho emails
4. **Phase 2.4:** Rate limiting
5. **Phase 2.5:** Session store

---

## 🟠 KAFKA - EVENT STREAMING {#kafka}

### **Kafka là gì?**

**Apache Kafka** = Distributed Event Streaming Platform
- Message queue phân tán
- Handle millions messages/second
- Persistent, replicated, fault-tolerant
- Event sourcing & CQRS

**Key Concepts:**
- **Event**: Something happened (OrderCreated, UserRegistered)
- **Producer**: Service gửi events
- **Consumer**: Service nhận events
- **Topic**: Category of events
- **Partition**: Parallel processing

---

### **Khi nào CẦN dùng Kafka?**

#### **1. MICROSERVICES ARCHITECTURE**

**Monolith (Hiện tại):**
```
┌─────────────────────────────────┐
│   NestJS Monolith               │
│   ├─ Users Module               │
│   ├─ Products Module            │
│   ├─ Orders Module              │
│   └─ Payments Module            │
└─────────────────────────────────┘
       ↓
   PostgreSQL
```

**Microservices (Sau này với Kafka):**
```
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Users    │    │ Products │    │ Orders   │
│ Service  │    │ Service  │    │ Service  │
└────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │
     └───────────────┼───────────────┘
                     │
              ┌──────▼──────┐
              │    KAFKA    │
              └──────┬──────┘
                     │
     ┌───────────────┼───────────────┐
     │               │               │
┌────▼─────┐    ┌───▼──────┐    ┌──▼───────┐
│ Email    │    │ Analytics│    │ Shipping │
│ Service  │    │ Service  │    │ Service  │
└──────────┘    └──────────┘    └──────────┘
```

---

#### **2. ORDER PROCESSING WORKFLOW**

**Nghiệp vụ:**
```
User đặt hàng
  ↓
1. Create order (Order Service)
2. Giảm stock (Inventory Service)
3. Send email (Email Service)
4. Create invoice (Invoice Service)
5. Notify shipping (Shipping Service)
6. Update analytics (Analytics Service)
```

**Với Kafka:**
```typescript
// Order Service - Publish event
@Post('orders')
async createOrder(dto: CreateOrderDto) {
  const order = await this.prisma.order.create({ data: dto });
  
  // Publish event
  await this.kafka.emit('order.created', {
    orderId: order.id,
    userId: order.userId,
    items: order.items,
    total: order.total,
  });
  
  // Return ngay - không đợi services khác!
  return order;
}

// Email Service - Subscribe to event
@EventPattern('order.created')
async handleOrderCreated(data: OrderCreatedEvent) {
  await this.sendOrderConfirmationEmail(data);
}

// Inventory Service - Subscribe to same event
@EventPattern('order.created')
async handleOrderCreated(data: OrderCreatedEvent) {
  await this.decreaseStock(data.items);
}
```

**Lợi ích:**
- ✅ Loosely coupled (services độc lập)
- ✅ Scalable (thêm consumer dễ dàng)
- ✅ Reliable (retry nếu fail)
- ✅ Fast response (async processing)

---

#### **3. EVENT SOURCING**

**Nghiệp vụ:**
Lưu lại TẤT CẢ events xảy ra trong hệ thống

**Events:**
```
- OrderCreated
- OrderStatusChanged: PENDING → CONFIRMED
- OrderStatusChanged: CONFIRMED → PROCESSING
- OrderStatusChanged: PROCESSING → SHIPPING
- PaymentProcessed
- OrderStatusChanged: SHIPPING → DELIVERED
```

**Lợi ích:**
- Audit trail đầy đủ
- Có thể replay events
- Analytics chính xác
- Debug dễ dàng

---

#### **4. DATA SYNCHRONIZATION**

**Nghiệp vụ:**
Sync data giữa nhiều systems

```
E-commerce (NestJS)
  ↓ Order Created
Kafka
  ↓
├─> Accounting System (Update revenue)
├─> CRM (Update customer data)
├─> Warehouse System (Update inventory)
└─> Analytics Platform (Update metrics)
```

---

### **Khi NÀO CHƯA CẦN Kafka?**

❌ **Không cần nếu:**
- Monolith application (1 service)
- Traffic thấp (<10,000 users/day)
- Team nhỏ (1-3 người)
- Đơn giản, không phức tạp
- Chưa cần event sourcing

✅ **Nên dùng khi:**
- Microservices architecture (>3 services)
- Traffic rất cao (>100,000 events/day)
- Cần event sourcing
- Cần data pipeline
- Team lớn, nhiều services

---

## 📊 DECISION MATRIX {#compare}

### **Cho Tarot E-commerce Project:**

| Phase | Users/Day | Tech Stack | Lý do |
|-------|-----------|------------|-------|
| **MVP** | 0-1K | NestJS + PostgreSQL | Đơn giản, nhanh launch |
| **Growth** | 1K-10K | + Redis | Cache, session, queue |
| **Scale** | 10K-100K | + Bull Queue | Background jobs |
| **Enterprise** | 100K+ | + Kafka + Microservices | Event-driven, scale |

---

## 🛠️ SETUP GUIDE {#setup}

### **REDIS SETUP**

#### **1. Docker Compose**

**File:** `docker-compose.yml`

Thêm Redis service:
```yaml
services:
  postgres:
    # ... existing config
  
  redis:
    image: redis:7-alpine
    container_name: tarot-redis
    restart: always
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

volumes:
  postgres_data:
  redis_data:
```

---

#### **2. Install Dependencies**

```bash
npm install ioredis
npm install @nestjs/cache-manager cache-manager
npm install cache-manager-ioredis-yet
```

---

#### **3. Create RedisModule**

**File:** `src/redis/redis.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-ioredis-yet';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: async () => ({
        store: await redisStore({
          host: 'localhost',
          port: 6379,
        }),
      }),
    }),
  ],
})
export class RedisModule {}
```

---

#### **4. Use in Service**

```typescript
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';

@Injectable()
export class ProductsService {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {}

  async getFeatured() {
    const cached = await this.cache.get('featured-products');
    if (cached) return cached;
    
    const products = await this.prisma.product.findMany({...});
    await this.cache.set('featured-products', products, 3600);
    
    return products;
  }
}
```

---

### **KAFKA SETUP**

#### **1. Docker Compose**

```yaml
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
  
  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - '9092:9092'
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
```

---

#### **2. Install Dependencies**

```bash
npm install @nestjs/microservices kafkajs
```

---

#### **3. Create Kafka Module**

```typescript
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'KAFKA_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            brokers: ['localhost:9092'],
          },
        },
      },
    ]),
  ],
})
export class KafkaModule {}
```

---

#### **4. Publish Event**

```typescript
@Injectable()
export class OrdersService {
  constructor(
    @Inject('KAFKA_SERVICE') private kafka: ClientKafka,
  ) {}

  async create(dto: CreateOrderDto) {
    const order = await this.prisma.order.create({ data: dto });
    
    // Publish event
    this.kafka.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
    });
    
    return order;
  }
}
```

---

#### **5. Subscribe to Event**

```typescript
@Controller()
export class EmailController {
  @EventPattern('order.created')
  async handleOrderCreated(data: OrderCreatedEvent) {
    await this.emailService.sendOrderConfirmation(data);
  }
}
```

---

## 🎓 PRACTICE ROADMAP {#practice}

### **BEGINNER (Đang ở đây)**

**Focus:** NestJS + PostgreSQL + Prisma
- [ ] Users CRUD
- [ ] Products CRUD
- [ ] Orders CRUD
- [ ] Authentication (JWT)
- [ ] File upload

**Timeline:** 2-4 tuần

---

### **INTERMEDIATE (Sau 1 tháng)**

**Focus:** + Redis

**Practice Tasks:**

#### **Week 1: Redis Basics**
- [ ] Setup Redis với Docker
- [ ] Install ioredis
- [ ] Practice Redis commands (SET, GET, DEL)
- [ ] Understand TTL

#### **Week 2: Caching**
- [ ] Cache featured products
- [ ] Cache product details
- [ ] Implement cache invalidation
- [ ] Measure performance improvement

#### **Week 3: Advanced Redis**
- [ ] Guest cart với Redis
- [ ] Rate limiting
- [ ] View counter
- [ ] Trending products (Sorted Sets)

#### **Week 4: Bull Queue**
- [ ] Setup Bull
- [ ] Email queue
- [ ] Image processing queue
- [ ] Scheduled jobs (cron)

**Timeline:** 1 tháng

---

### **ADVANCED (Sau 2-3 tháng)**

**Focus:** + Kafka + Microservices

**Practice Tasks:**

#### **Month 1: Kafka Basics**
- [ ] Setup Kafka với Docker
- [ ] Understand topics, partitions
- [ ] Producer/Consumer basics
- [ ] Practice với kafkajs library

#### **Month 2: Event-Driven Architecture**
- [ ] Design events (OrderCreated, etc.)
- [ ] Implement event publishing
- [ ] Multiple consumers cho 1 event
- [ ] Error handling & retry

#### **Month 3: Microservices**
- [ ] Split monolith thành services
- [ ] Service-to-service communication
- [ ] Distributed tracing
- [ ] Monitoring & alerting

**Timeline:** 3 tháng

---

## 🎯 KHI NÀO HỌC GÌ?

### **BÂY GIỜ (Month 1-2):**
**Focus:** NestJS + PostgreSQL + Prisma
```
✅ Học cái này NGAY:
- NestJS fundamentals
- Prisma ORM
- RESTful APIs
- Authentication
- File upload
- Testing

❌ CHƯA cần:
- Redis
- Kafka
- Microservices
```

**Lý do:**
- Master basics trước
- Xây dựng MVP hoàn chỉnh
- Launch product sớm
- Thu thập feedback

---

### **SAU NÀY (Month 3-4):**
**Focus:** + Redis
```
✅ Học khi:
- MVP đã launch
- Có users thật
- Thấy performance bottlenecks
- Cần optimize

❌ CHƯA cần Kafka
```

**Lý do:**
- Giải quyết vấn đề thực tế
- Optimization có target rõ ràng
- Vẫn là monolith (đơn giản)

---

### **TƯƠNG LAI (Month 6+):**
**Focus:** + Kafka
```
✅ Học khi:
- Traffic rất cao
- Cần scale nhiều services
- Team lớn hơn
- Architecture phức tạp

🎯 Lúc này bạn đã:
- Expert NestJS
- Expert Prisma
- Có experience với Redis
- Hiểu microservices concepts
```

---

## 📋 REDIS PRACTICE GUIDE (Khi sẵn sàng)

### **Bài tập 1: Setup Redis**
```bash
# Update docker-compose.yml
# Start Redis
docker-compose up -d

# Install packages
npm install ioredis @nestjs/cache-manager
```

### **Bài tập 2: Simple Cache**
```typescript
// Cache "Hello World" response
const cached = await redis.get('hello');
if (cached) return cached;

const message = 'Hello World!';
await redis.setex('hello', 60, message);
return message;
```

### **Bài tập 3: Cache Product List**
- Implement caching cho `GET /products`
- Set TTL 5 minutes
- Measure performance (before/after)

### **Bài tập 4: Cache Invalidation**
```typescript
// Khi tạo/update/delete product
async create(dto) {
  const product = await this.prisma.product.create({...});
  
  // Invalidate cache
  await this.redis.del('products:all');
  
  return product;
}
```

### **Bài tập 5: Bull Queue**
- Setup Bull
- Create email queue
- Send welcome email async
- Monitor jobs

---

## 📋 KAFKA PRACTICE GUIDE (Tương lai xa)

### **Bài tập 1: Setup Kafka**
```bash
# docker-compose.yml với Zookeeper + Kafka
docker-compose up -d
```

### **Bài tập 2: Simple Producer/Consumer**
- Producer: Gửi message "Hello Kafka"
- Consumer: Nhận và log message

### **Bài tập 3: Order Events**
- Event: OrderCreated
- Consumers:
  - Email Service
  - Inventory Service
  - Analytics Service

### **Bài tập 4: Event Sourcing**
- Lưu tất cả order events
- Rebuild order state từ events
- Audit trail

---

## 💡 RECOMMENDATIONS

### **Cho Tarot Project (6 tháng đầu):**

**Month 1-2: Foundation**
```
✅ NestJS + PostgreSQL + Prisma
✅ Users, Products, Orders CRUD
✅ Authentication (JWT)
✅ File upload
❌ KHÔNG cần Redis/Kafka
```

**Month 3-4: Optimization**
```
✅ Add Redis cho:
   - Cache featured products
   - Guest cart
   - Session store
✅ Bull Queue cho:
   - Email sending
   - Image processing
❌ CHƯA cần Kafka
```

**Month 5-6: Scale**
```
✅ Optimize Redis usage
✅ Add more queues
✅ Performance monitoring
❌ CHƯA cần Kafka (trừ khi traffic >100K/day)
```

**Year 2: Microservices**
```
✅ Consider Kafka khi:
   - Traffic >100K users/day
   - Team >5 developers
   - Cần split services
   - Architecture phức tạp
```

---

## 🎯 TÓM TẮT

### **Redis:**
- 🟢 **Độ ưu tiên:** HIGH (thêm sớm khi có traffic)
- 📅 **Khi nào:** Month 3-4
- 📚 **Khó:** Dễ học (3-5 ngày)
- 💰 **ROI:** Cao (performance boost lớn)

### **Kafka:**
- 🟡 **Độ ưu tiên:** MEDIUM-LOW (chỉ khi thật sự cần)
- 📅 **Khi nào:** Month 6+ hoặc Year 2
- 📚 **Khó:** Khó (2-3 tuần)
- 💰 **ROI:** Cao nhưng chỉ khi scale lớn

---

## ✅ ACTION PLAN

### **BÂY GIỜ (Tuần này):**
1. ✅ Hoàn thiện Users Module
2. ✅ Add error handling
3. ✅ Test toàn diện
4. ✅ Master Prisma queries

### **TUẦN TỚI:**
1. Authentication Module (JWT, bcrypt)
2. Products Module với relationships
3. File upload

### **THÁNG TỚI:**
1. Orders Module
2. Cart Module
3. Payment integration

### **QUÝ TỚI:**
1. Consider Redis (nếu có traffic)
2. Bull Queue cho emails
3. Performance optimization

### **NĂM TỚI:**
1. Consider Kafka (nếu thật sự cần)
2. Microservices architecture
3. Event sourcing

---

## 🎓 LEARNING PRIORITY

```
Priority 1 (NOW): ████████████████████ 100%
- NestJS
- TypeScript
- Prisma
- PostgreSQL
- REST APIs

Priority 2 (Month 3): ████████░░░░░░░░░░ 40%
- Redis basics
- Caching strategies
- Bull Queue

Priority 3 (Month 6+): ████░░░░░░░░░░░░░░ 20%
- Advanced Redis
- Pub/Sub

Priority 4 (Year 2): ██░░░░░░░░░░░░░░░░ 10%
- Kafka
- Microservices
- Event Sourcing
```

---

## 💬 KẾT LUẬN

### **Cho bạn bây giờ:**

**ĐỪNG lo về Redis/Kafka!** 🙅‍♂️

**Focus vào:**
1. ✅ Master NestJS basics
2. ✅ Master Prisma ORM
3. ✅ Build complete CRUD
4. ✅ Authentication
5. ✅ Launch MVP

**Redis/Kafka là advanced topics - học SAU KHI:**
- Đã thành thạo NestJS
- Đã có product running
- Có vấn đề cụ thể cần giải quyết

**Step by step, không rush!** 🐢

---

**BẠN HIỂU RỒI CHỨ?**

Redis/Kafka là powerful tools nhưng **CHƯA CẦN NGAY BÂY GIỜ**.

Focus vào:
1. Hoàn thành Users Module (tasks trong PRACTICE-GUIDE.md)
2. Làm Authentication
3. Làm Products Module
4. Launch MVP

Sau đó sẽ học Redis/Kafka khi thật sự cần! 😊

**Bạn có câu hỏi gì về Redis/Kafka không?** Hoặc **tiếp tục làm Users Module?** 🚀
