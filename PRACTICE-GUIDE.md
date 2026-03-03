# 🎯 PRACTICE GUIDE - TỰ THỰC HÀNH TỪNG BƯỚC

## 📋 PROGRESS TRACKER

### ✅ ĐÃ HOÀN THÀNH
- [x] Setup NestJS project
- [x] Setup PostgreSQL với Docker
- [x] Setup Prisma ORM
- [x] Tạo schema với User model
- [x] Chạy migration đầu tiên
- [x] Tạo PrismaService & PrismaModule
- [x] Generate Users resource (controller, service, DTOs)
- [x] Implement UsersService với Prisma queries

### 🔄 ĐANG LÀM
- [ ] **Enable validation**
- [ ] **Test API endpoints**
- [ ] **Handle errors properly**
- [ ] **Format code**

### 📝 SẮP TỚI
- [ ] Authentication (JWT)
- [ ] Password hashing
- [ ] Products Module
- [ ] Orders Module

---

## 🎯 GIAI ĐOẠN HIỆN TẠI: HOÀN THIỆN USERS MODULE

---

## ✅ **BƯỚC 1: FORMAT CODE**

### Mục tiêu:
- Code đẹp, dễ đọc
- Tắt lỗi Prettier

### Làm gì:
```bash
npm run format
```

### Kết quả mong đợi:
- Tất cả files được format
- Spacing nhất quán
- Quotes đúng (single quotes)

---

## ✅ **BƯỚC 2: ENABLE VALIDATION**

### Mục tiêu:
- Validate request body trước khi vào service
- Tự động reject invalid data

### Làm gì:

**File:** `src/main.ts`

**Thêm import:**
```typescript
import { ValidationPipe } from '@nestjs/common';
```

**Thêm vào function bootstrap() TRƯỚC `app.listen()`:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }),
);
```

### Giải thích options:
- `whitelist: true` - Tự động xóa properties không có trong DTO
- `forbidNonWhitelisted: true` - Throw error nếu có property thừa
- `transform: true` - Tự động convert types (string "1" → number 1)

### Verify:
- File `src/main.ts` có ValidationPipe
- Có console.log message khi start server

---

## ✅ **BƯỚC 3: RESTART SERVER**

### Làm gì:
```bash
# Stop server: Ctrl+C
# Start lại:
npm run start:dev
```

### Kết quả mong đợi:
```
✅ Database connected successfully!
🚀 Server running on http://localhost:3000
```

---

## ✅ **BƯỚC 4: TEST API ENDPOINTS**

### Mục tiêu:
- Test tất cả 5 endpoints
- Verify validation hoạt động
- Check database có lưu data

### Tools:
- Postman (recommend)
- Thunder Client (VS Code extension)
- Hoặc `curl` command

---

### **TEST 1: CREATE USER**

**Request:**
```http
POST http://localhost:3000/users
Content-Type: application/json

{
  "email": "test@example.com",
  "name": "Test User",
  "password": "password123"
}
```

**Kết quả mong đợi (200 OK):**
```json
{
  "id": 1,
  "email": "test@example.com",
  "name": "Test User",
  "password": "password123",
  "createdAt": "2026-03-02T...",
  "updatedAt": "2026-03-02T..."
}
```

**Verify:**
- [ ] Response có id (auto-generated)
- [ ] Response có timestamps
- [ ] Status code 201 Created

---

### **TEST 2: VALIDATION HOẠT ĐỘNG**

**Request (Email sai format):**
```http
POST http://localhost:3000/users
Content-Type: application/json

{
  "email": "not-an-email",
  "name": "Test",
  "password": "pass123"
}
```

**Kết quả mong đợi (400 Bad Request):**
```json
{
  "statusCode": 400,
  "message": [
    "email must be an email"
  ],
  "error": "Bad Request"
}
```

**Test các cases:**
- [ ] Email không hợp lệ → 400 error
- [ ] Name quá ngắn (< 2 chars) → 400 error
- [ ] Password quá ngắn (< 8 chars) → 400 error
- [ ] Thiếu required field → 400 error
- [ ] Email trùng lặp → 500 error (sẽ fix sau)

---

### **TEST 3: GET ALL USERS**

**Request:**
```http
GET http://localhost:3000/users
```

**Kết quả mong đợi (200 OK):**
```json
[
  {
    "id": 1,
    "email": "test@example.com",
    "name": "Test User",
    "password": "password123",
    "createdAt": "...",
    "updatedAt": "..."
  }
]
```

**Verify:**
- [ ] Trả về array
- [ ] Có user vừa tạo
- [ ] Status code 200

---

### **TEST 4: GET USER BY ID**

**Request:**
```http
GET http://localhost:3000/users/1
```

**Kết quả mong đợi (200 OK):**
```json
{
  "id": 1,
  "email": "test@example.com",
  "name": "Test User",
  ...
}
```

**Test edge cases:**
```http
GET http://localhost:3000/users/999
```
→ Hiện tại trả về `null` (sẽ cải thiện sau)

**Verify:**
- [ ] ID tồn tại → trả về user
- [ ] ID không tồn tại → trả về null
- [ ] Invalid ID (abc) → auto transform hoặc error

---

### **TEST 5: UPDATE USER**

**Request:**
```http
PATCH http://localhost:3000/users/1
Content-Type: application/json

{
  "name": "Updated Name"
}
```

**Kết quả mong đợi (200 OK):**
```json
{
  "id": 1,
  "email": "test@example.com",
  "name": "Updated Name",  ← Changed!
  "password": "password123",
  "createdAt": "...",
  "updatedAt": "..."  ← Updated timestamp!
}
```

**Verify:**
- [ ] Name đã thay đổi
- [ ] updatedAt timestamp mới hơn createdAt
- [ ] Các fields khác không thay đổi

---

### **TEST 6: DELETE USER**

**Request:**
```http
DELETE http://localhost:3000/users/1
```

**Kết quả mong đợi (200 OK):**
```json
{
  "id": 1,
  "email": "test@example.com",
  ...
}
```

**Verify trong Prisma Studio:**
```bash
npx prisma studio
```
→ User đã bị xóa!

---

## ✅ **BƯỚC 5: VERIFY VỚI PRISMA STUDIO**

### Làm gì:
```bash
npx prisma studio
```

Browser mở: `http://localhost:5555`

### Kiểm tra:
- [ ] Table User có data
- [ ] Tạo user qua API → thấy trong Studio
- [ ] Xóa user qua API → biến mất trong Studio
- [ ] Update user qua API → thấy thay đổi

---

## ✅ **BƯỚC 6: HANDLE ERRORS**

### Vấn đề hiện tại:

**1. User không tồn tại:**
```
GET /users/999 → null  ❌ Nên trả 404!
```

**2. Email trùng lặp:**
```
POST /users với email đã tồn tại → 500 error ❌ Nên trả 400 hoặc 409!
```

**3. Delete user không tồn tại:**
```
DELETE /users/999 → 500 error ❌ Nên trả 404!
```

---

### **Bài tập: Thêm error handling**

**File:** `src/users/users.service.ts`

#### **Task 1: findOne - throw 404 nếu không tìm thấy**

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';

async findOne(id: number) {
  const user = await this.prisma.user.findUnique({ where: { id } });
  
  if (!user) {
    throw new NotFoundException(`User with ID ${id} not found`);
  }
  
  return user;
}
```

#### **Task 2: create - handle email duplicate**

```typescript
import { ConflictException } from '@nestjs/common';

async create(createUserDto: CreateUserDto) {
  try {
    return await this.prisma.user.create({
      data: createUserDto,
    });
  } catch (error) {
    if (error.code === 'P2002') {
      throw new ConflictException('Email already exists');
    }
    throw error;
  }
}
```

#### **Task 3: update & remove - check existence**

```typescript
async update(id: number, updateUserDto: UpdateUserDto) {
  await this.findOne(id); // Sẽ throw 404 nếu không tồn tại
  
  return this.prisma.user.update({
    where: { id },
    data: updateUserDto,
  });
}

async remove(id: number) {
  await this.findOne(id); // Sẽ throw 404 nếu không tồn tại
  
  return this.prisma.user.delete({
    where: { id },
  });
}
```

---

## ✅ **BƯỚC 7: IMPROVE SECURITY - ẨN PASSWORD**

### Vấn đề:
Response hiện tại trả về cả password (không an toàn!)

### Giải pháp:

**Option 1: Exclude trong query**
```typescript
async findAll() {
  return this.prisma.user.findMany({
    select: {
      id: true,
      email: true,
      name: true,
      createdAt: true,
      updatedAt: true,
      // password: false (không include)
    },
  });
}
```

**Option 2: Transform response trong controller**
```typescript
@Post()
async create(@Body() createUserDto: CreateUserDto) {
  const user = await this.usersService.create(createUserDto);
  delete user.password; // Xóa password khỏi response
  return user;
}
```

**Option 3: Interceptor (advanced - sau này)**

---

## ✅ **BƯỚC 8: TEST LẠI TẤT CẢ**

### Checklist:

**CRUD Operations:**
- [ ] POST /users - Tạo user thành công
- [ ] POST /users - Email trùng → 409 Conflict
- [ ] POST /users - Invalid email → 400 Bad Request
- [ ] POST /users - Password ngắn → 400 Bad Request
- [ ] GET /users - Lấy tất cả users
- [ ] GET /users/:id - Lấy user cụ thể
- [ ] GET /users/999 - Not found → 404
- [ ] PATCH /users/:id - Update thành công
- [ ] PATCH /users/999 - Not found → 404
- [ ] DELETE /users/:id - Xóa thành công
- [ ] DELETE /users/999 - Not found → 404

**Security:**
- [ ] Password không hiển thị trong response

---

## 📊 **TỰ ĐÁNH GIÁ**

Sau khi làm xong các bước trên, tự hỏi:

**Understanding:**
- [ ] Tôi hiểu tại sao dùng `this.prisma.user`?
- [ ] Tôi hiểu các Prisma query methods?
- [ ] Tôi hiểu validation pipe hoạt động thế nào?
- [ ] Tôi hiểu error handling với exceptions?

**Skills:**
- [ ] Tôi có thể tự tạo CRUD cho model khác?
- [ ] Tôi có thể add validation rules mới?
- [ ] Tôi có thể debug errors?
- [ ] Tôi có thể test APIs với Postman?

**Nếu trả lời YES cho tất cả → BẠN ĐÃ MASTER Users CRUD!** 🎉

---

## 🚀 **SAU KHI HOÀN THÀNH USERS MODULE**

Bạn sẽ học tiếp:

### **Giai đoạn tiếp theo 1: Authentication (3-4 ngày)**
- [ ] Hash passwords với bcrypt
- [ ] JWT authentication
- [ ] Login/Register endpoints
- [ ] Protected routes với Guards
- [ ] Get current user

### **Giai đoạn tiếp theo 2: Products Module (2-3 ngày)**
- [ ] Tạo Product model với relations
- [ ] Categories relationship (1:N)
- [ ] Product images (1:N)
- [ ] CRUD cho Products
- [ ] Filter, search, pagination

### **Giai đoạn tiếp theo 3: File Upload (1-2 ngày)**
- [ ] Upload hình ảnh sản phẩm
- [ ] Validate file types
- [ ] Resize/optimize images
- [ ] Store trong cloud (Cloudinary/S3)

---

## 📝 **CHECKLIST - HOÀN THIỆN USERS MODULE**

### **Task 1: Enable Validation ⏰ 5 phút**

**File cần sửa:** `src/main.ts`

**Cần làm:**
- [ ] Import `ValidationPipe` từ `@nestjs/common`
- [ ] Thêm `app.useGlobalPipes(new ValidationPipe({...}))`
- [ ] Config với whitelist, forbidNonWhitelisted, transform
- [ ] Thêm console.log để biết server đã start

**Verify:**
- [ ] Server restart OK
- [ ] Thấy console.log message

---

### **Task 2: Test Validation ⏰ 10 phút**

**Cần test:**
- [ ] POST với email sai format → 400 error
- [ ] POST với name ngắn (<2 chars) → 400 error
- [ ] POST với password ngắn (<8 chars) → 400 error
- [ ] POST với extra fields → stripped hoặc error
- [ ] POST với data hợp lệ → 201 success

**Tool:**
- Postman
- Thunder Client
- Hoặc curl

---

### **Task 3: Add Error Handling ⏰ 15 phút**

**File cần sửa:** `src/users/users.service.ts`

**Cần làm:**

**1. Import exceptions:**
```typescript
import { 
  Injectable, 
  NotFoundException, 
  ConflictException 
} from '@nestjs/common';
```

**2. Update `findOne()`:**
- [ ] Query user từ database
- [ ] Nếu `null` → throw `NotFoundException`
- [ ] Nếu tìm thấy → return user

**3. Update `create()`:**
- [ ] Wrap trong try-catch
- [ ] Catch error code `P2002` (unique constraint)
- [ ] Throw `ConflictException` với message rõ ràng

**4. Update `update()`:**
- [ ] Gọi `findOne()` trước để check existence
- [ ] Nếu tồn tại → update
- [ ] Return updated user

**5. Update `remove()`:**
- [ ] Gọi `findOne()` trước để check existence
- [ ] Nếu tồn tại → delete
- [ ] Return deleted user

**Verify:**
- [ ] GET /users/999 → 404 Not Found
- [ ] POST với email trùng → 409 Conflict
- [ ] DELETE /users/999 → 404 Not Found
- [ ] PATCH /users/999 → 404 Not Found

---

### **Task 4: Ẩn Password trong Response ⏰ 10 phút**

**Option 1: Trong Service (Recommend cho beginner)**

**Cần làm:**
- [ ] Thêm `select` vào tất cả queries
- [ ] Select tất cả fields TRỪ password
- [ ] Test lại tất cả endpoints

**Ví dụ:**
```typescript
findAll() {
  return this.prisma.user.findMany({
    select: {
      id: true,
      email: true,
      name: true,
      createdAt: true,
      updatedAt: true,
      // password: false (excluded)
    },
  });
}
```

**Verify:**
- [ ] GET /users → không có password
- [ ] GET /users/:id → không có password
- [ ] POST /users → không có password
- [ ] PATCH /users/:id → không có password

---

### **Task 5: Test toàn diện ⏰ 20 phút**

**Scenario 1: Happy Path**
1. [ ] POST tạo user mới → Success
2. [ ] GET /users → Thấy user vừa tạo
3. [ ] GET /users/:id → Lấy đúng user
4. [ ] PATCH update name → Success
5. [ ] DELETE user → Success
6. [ ] GET /users → Array rỗng

**Scenario 2: Error Cases**
1. [ ] POST email trùng → 409
2. [ ] POST email invalid → 400
3. [ ] GET user không tồn tại → 404
4. [ ] UPDATE user không tồn tại → 404
5. [ ] DELETE user không tồn tại → 404

**Scenario 3: Validation**
1. [ ] POST thiếu field → 400
2. [ ] POST extra field → stripped hoặc 400
3. [ ] POST với types sai → 400

---

### **Task 6: Verify trong Database ⏰ 5 phút**

```bash
npx prisma studio
```

**Kiểm tra:**
- [ ] Users tạo qua API xuất hiện trong database
- [ ] Updates reflect trong database
- [ ] Deletes remove khỏi database
- [ ] Timestamps tự động update

---

## 💡 **TIPS QUAN TRỌNG**

### **1. Debugging:**
- Thêm `console.log()` trong service để xem data
- Check terminal logs để xem errors
- Dùng Prisma Studio để verify database

### **2. Testing:**
- Test happy path TRƯỚC
- Test error cases SAU
- Save requests trong Postman collection

### **3. Code Quality:**
- Chạy `npm run format` thường xuyên
- Đọc error messages kỹ
- Google error nếu không hiểu

### **4. Learning:**
- Mỗi lần fix bug = học được điều mới
- Thử break code để hiểu nó hoạt động thế nào
- Document lại những gì học được

---

## ❓ **KHI NÀO HỎI TRỢ GIÚP**

### **HỎI NGAY nếu:**
- Stuck >30 phút không biết fix
- Error message không hiểu
- Không biết approach nào đúng
- Muốn review code

### **TỰ TÌM HIỂU TRƯỚC nếu:**
- Typo đơn giản
- Lỗi import path
- Syntax errors nhỏ
- Documentation có sẵn

**NHƯNG đừng ngại hỏi! Mình luôn sẵn sàng giúp!** 😊

---

## 🎯 **MILESTONE POINTS**

Sau khi hoàn thành Users Module:

### **Level 1: Beginner** 🌱
- [ ] Tất cả 5 endpoints hoạt động
- [ ] Validation basic hoạt động
- [ ] Có thể test với Postman

### **Level 2: Intermediate** 🌿
- [ ] Error handling đầy đủ (404, 409, 400)
- [ ] Password bị ẩn trong response
- [ ] Code clean, formatted

### **Level 3: Advanced** 🌳
- [ ] Understand Prisma queries deeply
- [ ] Có thể explain code cho người khác
- [ ] Ready cho Authentication module

**Bạn đang ở level nào rồi?** 🎯

---

## 📅 **TIMELINE ƯỚC TÍNH**

**Nếu làm full-time:**
- Task 1-3: 30 phút
- Task 4-6: 1 giờ
- Testing & debugging: 1-2 giờ
- **Tổng: 2-3 giờ**

**Nếu làm part-time (2-3h/ngày):**
- **Ngày 1:** Tasks 1-3, basic testing
- **Ngày 2:** Tasks 4-6, comprehensive testing
- **Tổng: 2 ngày**

---

## ✅ **BẮT ĐẦU NGAY**

**Bước đầu tiên của bạn:**

1. Format code: `npm run format`
2. Enable ValidationPipe trong `main.ts`
3. Restart server
4. Test POST endpoint tạo user đầu tiên!

**Làm từng task một, đừng rush!** 🐢

**Khi làm xong mỗi task, báo mình biết để review nhé!** 😊

---

## 🎁 **BONUS: POSTMAN COLLECTION TEMPLATE**

Tạo collection trong Postman:

```
Tarot API
├── Users
│   ├── Create User (POST /users)
│   ├── Get All Users (GET /users)
│   ├── Get User (GET /users/:id)
│   ├── Update User (PATCH /users/:id)
│   └── Delete User (DELETE /users/:id)
├── Auth (sau này)
└── Products (sau này)
```

Save requests để reuse!

---

**BẠN SẴN SÀNG BẮT ĐẦU TASK 1 CHƯA?** 🚀