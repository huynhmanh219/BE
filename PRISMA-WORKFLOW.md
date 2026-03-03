# 🔄 PRISMA WORKFLOW - HIỂU TOÀN BỘ LUỒNG

## 📖 MỤC LỤC

1. [Overview - Big Picture](#overview)
2. [Setup Prisma từ đầu](#setup)
3. [Migration Workflow](#migration)
4. [Prisma Client](#client)
5. [Tích hợp NestJS](#nestjs)
6. [Sử dụng trong Code](#usage)
7. [Workflow khi có thay đổi](#changes)

---

## 🎯 OVERVIEW - BIG PICTURE {#overview}

### **Prisma là gì?**

**Prisma** = ORM (Object-Relational Mapping)
- Cầu nối giữa **TypeScript code** và **Database**
- Thay vì viết SQL thuần → Dùng TypeScript methods
- Type-safe, auto-complete, dễ maintain

### **So sánh với cách cũ:**

**KHÔNG dùng Prisma (SQL thuần):**
```typescript
const result = await db.query(
  'SELECT * FROM users WHERE email = $1',
  ['test@example.com']
);
// result là any → Không có type safety!
```

**DÙNG Prisma:**
```typescript
const user = await prisma.user.findUnique({
  where: { email: 'test@example.com' }
});
// user có type User → Auto-complete, type-safe!
```

---

### **3 Components chính của Prisma:**

```
┌─────────────────────────────────────────┐
│         1. PRISMA SCHEMA                │
│    (prisma/schema.prisma)               │
│    Define models, database structure    │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         2. PRISMA CLI                   │
│    Commands: migrate, generate          │
│    Tạo migrations, generate client      │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         3. PRISMA CLIENT                │
│    (Generated TypeScript code)          │
│    Query methods để dùng trong app      │
└─────────────────────────────────────────┘
```

---

## 🚀 SETUP PRISMA TỪ ĐẦU {#setup}

### **Bước 1: Install Prisma**

```bash
# Install Prisma CLI (dev dependency)
npm install -D prisma

# Install Prisma Client (runtime dependency)
npm install @prisma/client
```

**Giải thích:**
- `prisma` (CLI): Dùng để generate code, chạy migrations
- `@prisma/client`: Library dùng trong code để query database

---

### **Bước 2: Initialize Prisma**

```bash
npx prisma init
```

**Lệnh này tạo:**
```
project/
├── prisma/
│   └── schema.prisma    # ← Main config file
└── .env                 # ← Database URL
```

**File `prisma/schema.prisma` (initial):**
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

**File `.env` (initial):**
```env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

---

### **Bước 3: Config Database**

**Option 1: PostgreSQL (Production)**
```env
DATABASE_URL="postgresql://postgres:postgres123@localhost:5432/tarot_db"
```

**Option 2: SQLite (Development)**
```env
DATABASE_URL="file:./dev.db"
```

Update `schema.prisma`:
```prisma
datasource db {
  provider = "postgresql"  # hoặc "sqlite"
  url      = env("DATABASE_URL")
}
```

---

### **Bước 4: Define Models**

Thêm models vào `schema.prisma`:

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  password  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // Relations
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  published Boolean  @default(false)
  authorId  Int
  
  // Relations
  author    User     @relation(fields: [authorId], references: [id])
}
```

**Giải thích annotations:**
- `@id` - Primary key
- `@default(autoincrement())` - Auto tăng
- `@unique` - Giá trị không trùng lặp
- `@default(now())` - Timestamp hiện tại
- `@updatedAt` - Tự động update khi record thay đổi
- `@relation` - Foreign key relationship

---

## 🔄 MIGRATION WORKFLOW {#migration}

### **Migration là gì?**

**Migration** = File SQL để thay đổi database schema
- Track history của database changes
- Giống Git cho database
- Có thể rollback nếu cần

---

### **Workflow Migration:**

```
┌──────────────────────┐
│ 1. Edit schema.prisma│
│    Add/Change models │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────┐
│ 2. Run: npx prisma migrate   │
│    dev --name description    │
└──────────┬───────────────────┘
           │
           ├─────────────────────────────┐
           │                             │
           ▼                             ▼
┌──────────────────────┐    ┌──────────────────────┐
│ 3a. Generate SQL     │    │ 3b. Apply to DB      │
│     migration file   │    │     Execute SQL      │
└──────────────────────┘    └──────────────────────┘
           │                             │
           └─────────────┬───────────────┘
                         │
                         ▼
           ┌──────────────────────────┐
           │ 4. Generate Prisma Client│
           │    TypeScript types      │
           └──────────────────────────┘
```

---

### **Các lệnh Migration:**

#### **1. Create Migration (Development)**
```bash
npx prisma migrate dev --name init
```

**Làm gì:**
1. Compare `schema.prisma` với database hiện tại
2. Tạo SQL migration file
3. Apply migration vào database
4. Generate Prisma Client mới

**Kết quả:**
```
prisma/migrations/
└── 20260302074333_init/
    └── migration.sql      # SQL đã chạy
```

---

#### **2. Apply Migration (Production)**
```bash
npx prisma migrate deploy
```

**Làm gì:**
- Apply pending migrations
- KHÔNG tạo migrations mới
- Dùng trong CI/CD, production

---

#### **3. Reset Database**
```bash
npx prisma migrate reset
```

**Làm gì:**
- Xóa tất cả data
- Drop database
- Re-run tất cả migrations
- ⚠️ CẢNH BÁO: Mất hết data!

---

#### **4. Check Migration Status**
```bash
npx prisma migrate status
```

**Làm gì:**
- Xem migrations nào đã apply
- Migrations nào pending

---

### **Migration File Format:**

```sql
-- CreateTable
CREATE TABLE "User" (
    "id" SERIAL NOT NULL,
    "email" TEXT NOT NULL,
    "name" TEXT NOT NULL,
    
    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");
```

**Prisma tự generate SQL này từ schema.prisma!**

---

## 🎨 PRISMA CLIENT {#client}

### **Prisma Client là gì?**

**Prisma Client** = Auto-generated TypeScript code
- Dựa trên `schema.prisma`
- Có tất cả query methods
- Type-safe, auto-complete

---

### **Generate Prisma Client:**

```bash
npx prisma generate
```

**Làm gì:**
1. Đọc `schema.prisma`
2. Generate TypeScript code
3. Lưu vào `node_modules/@prisma/client` (mặc định)
4. Hoặc custom path (như project này: `src/generated/prisma`)

---

### **Custom Output Path:**

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../src/generated/prisma"  # ← Custom path
}
```

**Lợi ích:**
- Source control (commit vào Git)
- Dễ debug
- Tách biệt với node_modules

---

### **Generated Code Structure:**

```
src/generated/prisma/
├── index.d.ts         # TypeScript types
├── index.js           # JavaScript code
├── schema.prisma      # Copy of schema
└── ...                # Other files
```

**File `index.d.ts` chứa:**
```typescript
export class PrismaClient {
  user: {
    create(data): Promise<User>;
    findMany(): Promise<User[]>;
    findUnique(where): Promise<User | null>;
    update(where, data): Promise<User>;
    delete(where): Promise<User>;
    // ... many more methods
  }
}

export type User = {
  id: number;
  email: string;
  name: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}
```

---

## 🏗️ TÍCH HỢP NESTJS {#nestjs}

### **Architecture Overview:**

```
NestJS App
│
├── PrismaModule          # Quản lý Prisma
│   └── PrismaService     # Connect & Disconnect DB
│
├── UsersModule
│   ├── UsersController   # HTTP Endpoints
│   └── UsersService      # Business Logic
│       └── uses PrismaService  # ← Query database
│
└── ProductsModule
    ├── ProductsController
    └── ProductsService
        └── uses PrismaService  # ← Query database
```

---

### **Bước 1: Tạo PrismaService**

**File:** `src/prisma/prisma.service.ts`

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '../generated/prisma';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  // Khi module khởi động
  async onModuleInit() {
    await this.$connect();  // Connect database
    console.log('Database connected!');
  }

  // Khi app shutdown
  async onModuleDestroy() {
    await this.$disconnect();  // Disconnect database
    console.log('Database disconnected!');
  }
}
```

**Giải thích:**
- `extends PrismaClient` → Kế thừa tất cả query methods
- `OnModuleInit` → Lifecycle hook khi module start
- `$connect()` → Mở connection pool
- `$disconnect()` → Đóng connections (graceful shutdown)

---

### **Bước 2: Tạo PrismaModule**

**File:** `src/prisma/prisma.module.ts`

```typescript
import { Module, Global } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()  // ← Optional: Make available globally
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

**Giải thích:**
- `@Global()` → Không cần import PrismaModule ở mọi module
- `providers` → Declare service
- `exports` → Cho phép modules khác dùng

---

### **Bước 3: Import vào AppModule**

**File:** `src/app.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from './prisma/prisma.module';
import { UsersModule } from './users/users.module';

@Module({
  imports: [
    PrismaModule,  // ← Import ở đây
    UsersModule,
  ],
})
export class AppModule {}
```

---

### **Bước 4: Sử dụng trong Service**

**File:** `src/users/users.service.ts`

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  // Inject PrismaService
  constructor(private prisma: PrismaService) {}

  // Create user
  async create(dto: CreateUserDto) {
    return this.prisma.user.create({
      data: dto,
    });
  }

  // Get all users
  async findAll() {
    return this.prisma.user.findMany();
  }

  // Get user by ID
  async findOne(id: number) {
    return this.prisma.user.findUnique({
      where: { id },
    });
  }

  // Update user
  async update(id: number, dto: UpdateUserDto) {
    return this.prisma.user.update({
      where: { id },
      data: dto,
    });
  }

  // Delete user
  async delete(id: number) {
    return this.prisma.user.delete({
      where: { id },
    });
  }
}
```

---

## 💻 SỬ DỤNG TRONG CODE {#usage}

### **Query Methods:**

#### **1. CREATE**
```typescript
// Create single
const user = await prisma.user.create({
  data: {
    email: 'test@example.com',
    name: 'Test User',
    password: 'hashed_password',
  },
});

// Create many
const users = await prisma.user.createMany({
  data: [
    { email: 'user1@test.com', name: 'User 1', password: 'pass1' },
    { email: 'user2@test.com', name: 'User 2', password: 'pass2' },
  ],
});
```

---

#### **2. READ**
```typescript
// Find all
const users = await prisma.user.findMany();

// Find with filter
const users = await prisma.user.findMany({
  where: {
    name: { contains: 'John' },  // LIKE '%John%'
  },
});

// Find unique (by unique field)
const user = await prisma.user.findUnique({
  where: { email: 'test@example.com' },
});

// Find first
const user = await prisma.user.findFirst({
  where: { name: 'John' },
});

// Pagination
const users = await prisma.user.findMany({
  skip: 10,   // Offset
  take: 10,   // Limit
});

// Sorting
const users = await prisma.user.findMany({
  orderBy: {
    createdAt: 'desc',  // Newest first
  },
});

// Select specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    name: true,
    // password: false (excluded)
  },
});

// Include relations
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: true,  // Include user's posts
  },
});
```

---

#### **3. UPDATE**
```typescript
// Update one
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'New Name',
  },
});

// Update many
const result = await prisma.user.updateMany({
  where: {
    email: { endsWith: '@test.com' },
  },
  data: {
    name: 'Test User',
  },
});

// Upsert (update if exists, create if not)
const user = await prisma.user.upsert({
  where: { email: 'test@example.com' },
  update: { name: 'Updated Name' },
  create: {
    email: 'test@example.com',
    name: 'New User',
    password: 'password',
  },
});
```

---

#### **4. DELETE**
```typescript
// Delete one
const user = await prisma.user.delete({
  where: { id: 1 },
});

// Delete many
const result = await prisma.user.deleteMany({
  where: {
    email: { endsWith: '@test.com' },
  },
});
```

---

#### **5. AGGREGATIONS**
```typescript
// Count
const count = await prisma.user.count();

// Count with filter
const count = await prisma.user.count({
  where: {
    email: { contains: '@gmail.com' },
  },
});

// Aggregate
const result = await prisma.user.aggregate({
  _avg: { age: true },
  _max: { age: true },
  _min: { age: true },
  _sum: { age: true },
});
```

---

#### **6. TRANSACTIONS**
```typescript
// Sequential operations
const [deletePosts, deleteUser] = await prisma.$transaction([
  prisma.post.deleteMany({ where: { authorId: 1 } }),
  prisma.user.delete({ where: { id: 1 } }),
]);

// Interactive transaction
const result = await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: 'test@test.com', name: 'Test', password: 'pass' },
  });
  
  const post = await tx.post.create({
    data: { title: 'First Post', content: 'Hello', authorId: user.id },
  });
  
  return { user, post };
});
```

---

## 🔄 WORKFLOW KHI CÓ THAY ĐỔI {#changes}

### **Scenario: Thêm field mới vào User**

**Bước 1: Update `schema.prisma`**
```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  password  String
  phone     String?  // ← Thêm field mới (optional)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

---

**Bước 2: Create Migration**
```bash
npx prisma migrate dev --name add_phone_to_user
```

**Output:**
```
✔ Generated migration: 20260302080000_add_phone_to_user
✔ Applied migration
✔ Generated Prisma Client
```

**File migration.sql:**
```sql
-- AlterTable
ALTER TABLE "User" ADD COLUMN "phone" TEXT;
```

---

**Bước 3: Update DTOs (NestJS)**
```typescript
// create-user.dto.ts
export class CreateUserDto {
  email: string;
  name: string;
  password: string;
  phone?: string;  // ← Thêm field
}
```

---

**Bước 4: Sử dụng ngay!**
```typescript
const user = await prisma.user.create({
  data: {
    email: 'test@test.com',
    name: 'Test',
    password: 'pass',
    phone: '0123456789',  // ← Type-safe!
  },
});
```

---

### **Scenario: Tạo table mới (Product)**

**Bước 1: Thêm model vào `schema.prisma`**
```prisma
model Product {
  id          Int      @id @default(autoincrement())
  name        String
  description String
  price       Decimal
  stock       Int
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

**Bước 2: Migration**
```bash
npx prisma migrate dev --name create_product
```

**Bước 3: Generate Resource**
```bash
nest g resource products
```

**Bước 4: Dùng trong code**
```typescript
// products.service.ts
async findAll() {
  return this.prisma.product.findMany();
}
```

---

## 📊 SUMMARY - TÓM TẮT TOÀN BỘ WORKFLOW

### **Lần đầu setup:**
```bash
1. npm install -D prisma @prisma/client
2. npx prisma init
3. Edit .env (DATABASE_URL)
4. Edit schema.prisma (define models)
5. npx prisma migrate dev --name init
6. Create PrismaService & PrismaModule
7. Import vào AppModule
8. Sử dụng trong services
```

### **Khi thay đổi schema:**
```bash
1. Edit schema.prisma
2. npx prisma migrate dev --name description
3. Update DTOs (if needed)
4. Code tự động có types mới!
```

### **Commands thường dùng:**
```bash
npx prisma migrate dev       # Create & apply migration (dev)
npx prisma generate          # Generate Prisma Client
npx prisma studio            # GUI để xem/edit data
npx prisma migrate reset     # Reset database (xóa data!)
npx prisma db push           # Push schema without migration (prototype)
npx prisma db pull           # Pull schema from existing DB
```

---

## 🎯 BEST PRACTICES

### 1. **Luôn commit migrations vào Git**
```
✅ prisma/migrations/
✅ prisma/schema.prisma
❌ src/generated/prisma/  (nếu generate vào src)
```

### 2. **Không edit migration files manually**
- Migration files là auto-generated
- Edit schema.prisma rồi generate lại

### 3. **Test migrations trước khi deploy**
```bash
# Development
npx prisma migrate dev

# Test
npx prisma migrate deploy --preview-feature

# Production
npx prisma migrate deploy
```

### 4. **Backup database trước major changes**
```bash
pg_dump tarot_db > backup.sql
```

### 5. **Use transactions cho related operations**
```typescript
// BAD: 2 separate operations
await prisma.user.delete({ where: { id: 1 } });
await prisma.post.deleteMany({ where: { authorId: 1 } });

// GOOD: Transaction
await prisma.$transaction([
  prisma.post.deleteMany({ where: { authorId: 1 } }),
  prisma.user.delete({ where: { id: 1 } }),
]);
```

---

## ❓ FAQ

**Q: Khi nào dùng `migrate dev` vs `db push`?**
A: 
- `migrate dev`: Development, cần track history
- `db push`: Prototyping nhanh, không cần migrations

**Q: Prisma Client ở đâu?**
A: Mặc định: `node_modules/@prisma/client`
    Custom: `src/generated/prisma` (trong project này)

**Q: Làm sao rollback migration?**
A: 
```bash
# Xem migrations
npx prisma migrate status

# Reset về trạng thái trước
npx prisma migrate reset
# Sau đó remove migration folder và migrate lại
```

**Q: TypeScript types không update?**
A: Chạy `npx prisma generate` lại

**Q: Database schema và Prisma schema không sync?**
A: 
```bash
# Pull từ database về schema
npx prisma db pull

# Push từ schema lên database
npx prisma db push
```

---

**BẠN ĐÃ HIỂU TOÀN BỘ LUỒNG PRISMA RỒI!** 🎉

Đọc xong document này, bạn nên hiểu:
- ✅ Prisma setup như thế nào
- ✅ Migration workflow
- ✅ Prisma Client là gì và cách hoạt động
- ✅ Tích hợp vào NestJS ra sao
- ✅ Cách query database
- ✅ Workflow khi thay đổi schema

**Có câu hỏi gì không rõ không?** 😊
