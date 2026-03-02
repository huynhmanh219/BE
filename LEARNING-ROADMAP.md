# 🎓 NESTJS LEARNING ROADMAP - TAROT E-COMMERCE API

## 🎯 MỤC TIÊU CUỐI CÙNG

Xây dựng REST API hoàn chỉnh cho trang web Tarot & Phong Thủy với:
- Authentication & Authorization (JWT)
- CRUD cho Products, Users, Orders
- File upload (images)
- Payment integration
- Email notifications
- Database với TypeORM/Prisma
- Validation & Error handling
- Testing

---

## 📋 LỘ TRÌNH HỌC (10 GIAI ĐOẠN)

### **GIAI ĐOẠN 1: FOUNDATIONS (1-2 ngày)**

#### Mục tiêu:
- Hiểu cấu trúc NestJS project
- Làm quen với Decorators, Modules, Controllers, Services
- Tạo API endpoint đầu tiên

#### Bài tập:
1. **Setup project (Đã có sẵn)**
   ```bash
   cd BE
   npm install
   npm run start:dev
   ```

2. **Tạo Health Check endpoint**
   - File: `src/app.controller.ts`
   - Endpoint: `GET /health`
   - Response: `{ status: 'ok', timestamp: '...' }`

3. **Tạo module đầu tiên: HelloModule**
   ```bash
   nest g module hello
   nest g controller hello
   nest g service hello
   ```
   - Endpoint: `GET /hello`
   - Response: `{ message: 'Hello Tarot API!' }`

#### Học:
- Controllers: Nhận request, trả response
- Services: Business logic
- Modules: Nhóm các features lại
- Dependency Injection

#### Tài liệu:
- [NestJS Docs - Controllers](https://docs.nestjs.com/controllers)
- [NestJS Docs - Providers](https://docs.nestjs.com/providers)

---

### **GIAI ĐOẠN 2: DATABASE SETUP (1-2 ngày)**

#### Mục tiêu:
- Kết nối PostgreSQL/MySQL
- Hiểu ORM (TypeORM hoặc Prisma)
- Tạo entities/models đầu tiên

#### Bài tập:
1. **Cài đặt database**
   ```bash
   # Option 1: PostgreSQL với TypeORM
   npm install @nestjs/typeorm typeorm pg
   
   # Option 2: Prisma (recommend cho beginner)
   npm install @prisma/client
   npm install -D prisma
   ```

2. **Setup Prisma (Recommend)**
   ```bash
   npx prisma init
   ```
   - Tạo file `schema.prisma`
   - Define model `User` đơn giản
   - Chạy migration đầu tiên

3. **Hoặc setup TypeORM**
   - Config database trong `app.module.ts`
   - Tạo entity `User`
   - Sync database

#### Học:
- Database connections
- ORM concepts
- Migrations
- Entities/Models

#### Database Schema đề xuất:
```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
}
```

---

### **GIAI ĐOẠN 3: CRUD BASICS (2-3 ngày)**

#### Mục tiêu:
- Tạo CRUD đầy đủ cho một resource
- Hiểu DTOs (Data Transfer Objects)
- Validation với class-validator

#### Bài tập:
1. **Tạo Users Module với CRUD**
   ```bash
   nest g resource users
   ```
   Choose:
   - REST API
   - Generate CRUD entry points? Yes

2. **Implement các endpoint:**
   - `POST /users` - Tạo user mới
   - `GET /users` - Lấy danh sách users
   - `GET /users/:id` - Lấy user theo ID
   - `PATCH /users/:id` - Update user
   - `DELETE /users/:id` - Xóa user

3. **Tạo DTOs với validation**
   ```typescript
   // create-user.dto.ts
   export class CreateUserDto {
     @IsEmail()
     email: string;
     
     @IsString()
     @MinLength(2)
     name: string;
   }
   ```

4. **Install validation**
   ```bash
   npm install class-validator class-transformer
   ```

5. **Enable global validation pipe**
   ```typescript
   // main.ts
   app.useGlobalPipes(new ValidationPipe());
   ```

#### Học:
- DTOs vs Entities
- Validation decorators
- Error handling
- Status codes

#### Test với:
- Postman hoặc Insomnia
- Thunder Client (VS Code extension)

---

### **GIAI ĐOẠN 4: AUTHENTICATION (3-4 ngày)**

#### Mục tiêu:
- Implement JWT authentication
- Hash passwords
- Protected routes
- Guards

#### Bài tập:
1. **Install dependencies**
   ```bash
   npm install @nestjs/jwt @nestjs/passport passport passport-jwt
   npm install bcrypt
   npm install -D @types/bcrypt @types/passport-jwt
   ```

2. **Tạo Auth Module**
   ```bash
   nest g module auth
   nest g service auth
   nest g controller auth
   ```

3. **Implement endpoints:**
   - `POST /auth/register` - Đăng ký
   - `POST /auth/login` - Đăng nhập
   - `GET /auth/profile` - Lấy thông tin user (protected)

4. **Hash password với bcrypt**
   ```typescript
   import * as bcrypt from 'bcrypt';
   
   const hashedPassword = await bcrypt.hash(password, 10);
   const isMatch = await bcrypt.compare(password, user.password);
   ```

5. **Generate JWT token**
   ```typescript
   const payload = { email: user.email, sub: user.id };
   const token = this.jwtService.sign(payload);
   ```

6. **Tạo JWT Strategy và Guard**
   - File: `jwt.strategy.ts`
   - File: `jwt-auth.guard.ts`
   - Protect routes với `@UseGuards(JwtAuthGuard)`

#### Học:
- JWT tokens
- Password hashing
- Guards và Strategies
- Request context
- Decorators

#### Security:
- NEVER lưu password plain text
- ALWAYS hash passwords
- Use environment variables cho JWT_SECRET

---

### **GIAI ĐOẠN 5: PRODUCTS MODULE (2-3 ngày)**

#### Mục tiêu:
- Tạo CRUD cho Products
- Relationships (Category, Images)
- Query filters & pagination

#### Bài tập:
1. **Define Prisma Schema**
   ```prisma
   model Product {
     id          Int      @id @default(autoincrement())
     name        String
     description String
     price       Decimal
     stock       Int
     category    Category @relation(fields: [categoryId], references: [id])
     categoryId  Int
     images      ProductImage[]
     createdAt   DateTime @default(now())
   }
   
   model Category {
     id       Int       @id @default(autoincrement())
     name     String    @unique
     products Product[]
   }
   
   model ProductImage {
     id        Int     @id @default(autoincrement())
     url       String
     product   Product @relation(fields: [productId], references: [id])
     productId Int
   }
   ```

2. **Generate Resources**
   ```bash
   nest g resource products
   nest g resource categories
   ```

3. **Implement Features:**
   - CRUD cho Products
   - Filter by category
   - Search by name
   - Pagination (skip, take)
   - Sort (newest, price asc/desc)

4. **DTOs:**
   ```typescript
   class CreateProductDto {
     name: string;
     description: string;
     price: number;
     stock: number;
     categoryId: number;
   }
   
   class QueryProductsDto {
     category?: string;
     search?: string;
     page?: number;
     limit?: number;
     sortBy?: 'price' | 'createdAt';
     order?: 'asc' | 'desc';
   }
   ```

#### Học:
- Database relations (1:N, N:M)
- Query builders
- Pagination
- Filtering & Sorting

---

### **GIAI ĐOẠN 6: FILE UPLOAD (1-2 ngày)**

#### Mục tiêu:
- Upload hình ảnh sản phẩm
- Validate file types
- Store files (local hoặc cloud)

#### Bài tập:
1. **Install multer**
   ```bash
   npm install @nestjs/platform-express multer
   npm install -D @types/multer
   ```

2. **Create upload endpoint**
   ```typescript
   @Post('upload')
   @UseInterceptors(FileInterceptor('file'))
   uploadFile(@UploadedFile() file: Express.Multer.File) {
     return { filename: file.filename };
   }
   ```

3. **Validate file**
   - Max size: 5MB
   - Allowed types: jpg, png, webp
   - Custom pipe validation

4. **Store files**
   - Option 1: Local (public/uploads/)
   - Option 2: Cloudinary
   - Option 3: AWS S3

#### Học:
- File uploads
- Multer configuration
- Static file serving
- File validation

---

### **GIAI ĐOẠN 7: ORDERS & CART (3-4 ngày)**

#### Mục tiêu:
- Cart system (persistent)
- Order creation
- Order status workflow
- Inventory management

#### Bài tập:
1. **Define Schema**
   ```prisma
   model Cart {
     id     Int        @id @default(autoincrement())
     user   User       @relation(fields: [userId], references: [id])
     userId Int        @unique
     items  CartItem[]
   }
   
   model CartItem {
     id        Int     @id @default(autoincrement())
     cart      Cart    @relation(fields: [cartId], references: [id])
     cartId    Int
     product   Product @relation(fields: [productId], references: [id])
     productId Int
     quantity  Int
   }
   
   model Order {
     id        Int         @id @default(autoincrement())
     user      User        @relation(fields: [userId], references: [id])
     userId    Int
     items     OrderItem[]
     total     Decimal
     status    OrderStatus
     createdAt DateTime    @default(now())
   }
   
   enum OrderStatus {
     PENDING
     CONFIRMED
     PROCESSING
     SHIPPING
     DELIVERED
     CANCELLED
   }
   ```

2. **Cart endpoints:**
   - `POST /cart/items` - Add to cart
   - `GET /cart` - Get cart
   - `PATCH /cart/items/:id` - Update quantity
   - `DELETE /cart/items/:id` - Remove item
   - `DELETE /cart` - Clear cart

3. **Order endpoints:**
   - `POST /orders` - Create from cart
   - `GET /orders` - User's orders
   - `GET /orders/:id` - Order details
   - `PATCH /orders/:id/status` - Update status (admin)
   - `DELETE /orders/:id` - Cancel order

4. **Business Logic:**
   - Check stock before adding to cart
   - Decrease stock when order confirmed
   - Restore stock when order cancelled
   - Calculate total price
   - Prevent negative stock

#### Học:
- Complex business logic
- Transactions
- State machines
- Race conditions

---

### **GIAI ĐOẠN 8: ADVANCED FEATURES (2-3 ngày)**

#### Mục tiêu:
- Role-based access control (RBAC)
- Email notifications
- Logging
- Environment configs

#### Bài tập:
1. **RBAC:**
   ```typescript
   enum UserRole {
     CUSTOMER = 'customer',
     ADMIN = 'admin',
     STAFF = 'staff',
   }
   
   @Roles('admin')
   @UseGuards(JwtAuthGuard, RolesGuard)
   @Delete('products/:id')
   deleteProduct() {}
   ```

2. **Email Service:**
   ```bash
   npm install @nestjs-modules/mailer nodemailer
   ```
   - Send welcome email on register
   - Order confirmation email
   - Password reset email

3. **Config Module:**
   ```bash
   npm install @nestjs/config
   ```
   - Use .env for secrets
   - Validate env variables

4. **Logging:**
   - Use NestJS built-in Logger
   - Log important events
   - Error tracking

#### Học:
- Guards composition
- Email templates
- Configuration management
- Logging best practices

---

### **GIAI ĐOẠN 9: TESTING (2-3 ngày)**

#### Mục tiêu:
- Unit tests
- Integration tests
- E2E tests

#### Bài tập:
1. **Unit Tests:**
   ```bash
   npm test
   ```
   - Test services
   - Mock dependencies
   - Test business logic

2. **E2E Tests:**
   ```bash
   npm run test:e2e
   ```
   - Test complete flows
   - Test authentication
   - Test CRUD operations

3. **Coverage:**
   ```bash
   npm run test:cov
   ```
   - Aim for >80% coverage

#### Học:
- Jest framework
- Mocking
- Test-driven development
- Coverage reports

---

### **GIAI ĐOẠN 10: DEPLOYMENT (1-2 ngày)**

#### Mục tiêu:
- Deploy to production
- Docker
- CI/CD

#### Bài tập:
1. **Dockerfile:**
   ```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build
   CMD ["npm", "run", "start:prod"]
   ```

2. **Deploy options:**
   - Railway
   - Render
   - Heroku
   - AWS/DigitalOcean

#### Học:
- Docker basics
- Environment variables
- Production best practices

---

## 🎯 CHIẾN LƯỢC HỌC

### Mỗi giai đoạn:
1. **ĐỌC** - Đọc docs NestJS về topic
2. **XEM** - Xem video tutorial (nếu cần)
3. **CODE** - Tự code theo bài tập
4. **TEST** - Test với Postman
5. **DEBUG** - Fix bugs (học nhiều nhất ở đây!)
6. **REVIEW** - Review code, refactor

### Khi gặp khó khăn:
1. Đọc lại docs
2. Google error message
3. Hỏi ChatGPT/mình
4. Check GitHub issues
5. Join Discord NestJS

---

## 📚 TÀI LIỆU THAM KHẢO

### Official:
- [NestJS Docs](https://docs.nestjs.com)
- [Prisma Docs](https://www.prisma.io/docs)
- [TypeORM Docs](https://typeorm.io)

### Courses:
- [NestJS Zero to Hero](https://www.udemy.com/course/nestjs-zero-to-hero/)
- [FreeCodeCamp NestJS Course](https://www.youtube.com/watch?v=GHTA143_b-s)

### Tools:
- Postman - API testing
- DBeaver - Database GUI
- Docker Desktop - Containers

---

## 🏁 BƯỚC TIẾP THEO

**BẠN SẴN SÀNG BẮT ĐẦU?**

Mình suggest bắt đầu từ **GIAI ĐOẠN 1** - tạo endpoint đơn giản.

Khi bạn làm xong mỗi giai đoạn, hãy:
1. Show code cho mình review
2. Mình sẽ cho feedback
3. Suggest improvements
4. Chuyển sang giai đoạn tiếp theo

**LƯU Ý:**
- Đừng rush! Mỗi giai đoạn học kỹ
- Code nhiều > đọc nhiều
- Debug là cách học tốt nhất
- Hỏi khi stuck >30 phút

Ready? Let's start! 🚀
