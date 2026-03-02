# 🚀 GETTING STARTED - BÀI TẬP ĐẦU TIÊN

## ✅ CHUẨN BỊ

Project NestJS đã được setup sẵn! Giờ bạn cần:

### 1. Khởi động server

```bash
cd d:\Project\tarot-project\BE
npm run start:dev
```

Server sẽ chạy tại: `http://localhost:3000`

### 2. Test API hiện tại

Mở browser hoặc Postman:
```
GET http://localhost:3000
```

Kết quả: `Hello World!` (từ file `src/app.controller.ts`)

---

## 📝 BÀI TẬP 1: TẠO HEALTH CHECK ENDPOINT

### Mục tiêu:
Tạo endpoint để check server status - thường dùng cho monitoring

### Yêu cầu:
- Endpoint: `GET /health`
- Response:
```json
{
  "status": "ok",
  "timestamp": "2026-03-02T12:00:00.000Z",
  "uptime": 123.456,
  "environment": "development"
}
```

### Hướng dẫn từng bước:

#### Bước 1: Mở file `src/app.controller.ts`

Hiện tại file này có:
```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

#### Bước 2: Thêm method mới

Thêm method `getHealth()` bên dưới method `getHello()`:

```typescript
@Get('health')
getHealth() {
  return {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV || 'development'
  };
}
```

#### Bước 3: Test endpoint

```
GET http://localhost:3000/health
```

Kỳ vọng:
```json
{
  "status": "ok",
  "timestamp": "2026-03-02T12:00:00.000Z",
  "uptime": 45.123,
  "environment": "development"
}
```

### Giải thích code:

- `@Get('health')`: Decorator định nghĩa route GET /health
- `new Date().toISOString()`: Lấy timestamp hiện tại
- `process.uptime()`: Số giây server đã chạy
- `process.env.NODE_ENV`: Môi trường (dev/production)

---

## 📝 BÀI TẬP 2: TẠO MODULE ĐẦU TIÊN

### Mục tiêu:
Tạo module "Tarot" với endpoint lấy thông tin về bài Tarot

### Yêu cầu:
- Endpoint: `GET /tarot/info`
- Response:
```json
{
  "name": "Tarot API",
  "version": "1.0.0",
  "description": "API cho trang web Tarot & Phong Thủy",
  "features": [
    "Products management",
    "User authentication",
    "Order processing",
    "Payment integration"
  ]
}
```

### Hướng dẫn:

#### Bước 1: Generate module, controller, service

Chạy lệnh NestJS CLI:

```bash
nest g module tarot
nest g controller tarot
nest g service tarot
```

**Giải thích:**
- `nest g module tarot`: Tạo TarotModule
- `nest g controller tarot`: Tạo TarotController
- `nest g service tarot`: Tạo TarotService

Files được tạo:
```
src/
  tarot/
    tarot.controller.ts
    tarot.service.ts
    tarot.module.ts
    tarot.controller.spec.ts
    tarot.service.spec.ts
```

#### Bước 2: Implement Service

Mở `src/tarot/tarot.service.ts`:

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class TarotService {
  getInfo() {
    return {
      name: 'Tarot API',
      version: '1.0.0',
      description: 'API cho trang web Tarot & Phong Thủy',
      features: [
        'Products management',
        'User authentication',
        'Order processing',
        'Payment integration'
      ]
    };
  }
}
```

**Giải thích:**
- `@Injectable()`: Decorator đánh dấu class này là service (có thể inject)
- `getInfo()`: Method chứa business logic

#### Bước 3: Implement Controller

Mở `src/tarot/tarot.controller.ts`:

```typescript
import { Controller, Get } from '@nestjs/common';
import { TarotService } from './tarot.service';

@Controller('tarot')
export class TarotController {
  constructor(private readonly tarotService: TarotService) {}

  @Get('info')
  getInfo() {
    return this.tarotService.getInfo();
  }
}
```

**Giải thích:**
- `@Controller('tarot')`: Base path là `/tarot`
- `constructor()`: Inject TarotService
- `@Get('info')`: Route `/tarot/info`
- Gọi service method để lấy data

#### Bước 4: Test

```
GET http://localhost:3000/tarot/info
```

---

## 📝 BÀI TẬP 3: TẠO ENDPOINT VỚI PARAMETERS

### Mục tiêu:
Tạo endpoint lấy thông tin về một lá bài Tarot cụ thể

### Yêu cầu:
- Endpoint: `GET /tarot/cards/:id`
- Example: `GET /tarot/cards/1`
- Response:
```json
{
  "id": 1,
  "name": "The Fool",
  "number": "0",
  "arcana": "Major Arcana",
  "description": "Khởi đầu mới, sự ngây thơ, phiêu lưu"
}
```

### Hướng dẫn:

#### Bước 1: Tạo fake data trong Service

Mở `src/tarot/tarot.service.ts`, thêm method:

```typescript
private cards = [
  {
    id: 1,
    name: 'The Fool',
    number: '0',
    arcana: 'Major Arcana',
    description: 'Khởi đầu mới, sự ngây thơ, phiêu lưu'
  },
  {
    id: 2,
    name: 'The Magician',
    number: 'I',
    arcana: 'Major Arcana',
    description: 'Sức mạnh, kỹ năng, hành động'
  },
  {
    id: 3,
    name: 'The High Priestess',
    number: 'II',
    arcana: 'Major Arcana',
    description: 'Trực giác, bí ẩn, tiềm thức'
  }
];

getCardById(id: number) {
  const card = this.cards.find(c => c.id === id);
  if (!card) {
    throw new Error('Card not found');
  }
  return card;
}

getAllCards() {
  return this.cards;
}
```

#### Bước 2: Thêm routes vào Controller

Mở `src/tarot/tarot.controller.ts`, thêm:

```typescript
import { Controller, Get, Param } from '@nestjs/common';

@Controller('tarot')
export class TarotController {
  // ... existing code

  @Get('cards')
  getAllCards() {
    return this.tarotService.getAllCards();
  }

  @Get('cards/:id')
  getCardById(@Param('id') id: string) {
    return this.tarotService.getCardById(parseInt(id));
  }
}
```

**Giải thích:**
- `@Param('id')`: Lấy parameter từ URL
- `parseInt(id)`: Convert string → number

#### Bước 3: Test

```
GET http://localhost:3000/tarot/cards
GET http://localhost:3000/tarot/cards/1
GET http://localhost:3000/tarot/cards/2
```

---

## 📝 BÀI TẬP 4: ERROR HANDLING

### Mục tiêu:
Xử lý lỗi khi card không tồn tại

### Yêu cầu:
- `GET /tarot/cards/999` → Trả về 404 Not Found
- Response:
```json
{
  "statusCode": 404,
  "message": "Card with ID 999 not found",
  "error": "Not Found"
}
```

### Hướng dẫn:

#### Bước 1: Import NotFoundException

Mở `src/tarot/tarot.service.ts`:

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';

@Injectable()
export class TarotService {
  // ... existing code

  getCardById(id: number) {
    const card = this.cards.find(c => c.id === id);
    if (!card) {
      throw new NotFoundException(`Card with ID ${id} not found`);
    }
    return card;
  }
}
```

#### Bước 2: Test

```
GET http://localhost:3000/tarot/cards/999
```

Kỳ vọng: 404 với message error

**NestJS built-in exceptions:**
- `NotFoundException` - 404
- `BadRequestException` - 400
- `UnauthorizedException` - 401
- `ForbiddenException` - 403
- `InternalServerErrorException` - 500

---

## 📝 BÀI TẬP 5: POST REQUEST & DTO

### Mục tiêu:
Tạo endpoint để "đọc bài" Tarot (POST với body)

### Yêu cầu:
- Endpoint: `POST /tarot/reading`
- Body:
```json
{
  "question": "Công việc của tôi sẽ như thế nào?",
  "spreadType": "three-card"
}
```
- Response:
```json
{
  "question": "Công việc của tôi sẽ như thế nào?",
  "cards": [
    { "position": "past", "card": { ... } },
    { "position": "present", "card": { ... } },
    { "position": "future", "card": { ... } }
  ],
  "timestamp": "..."
}
```

### Hướng dẫn:

#### Bước 1: Tạo DTO (Data Transfer Object)

Tạo file `src/tarot/dto/reading.dto.ts`:

```typescript
export class CreateReadingDto {
  question: string;
  spreadType: 'three-card' | 'celtic-cross' | 'single-card';
}
```

#### Bước 2: Thêm method vào Service

```typescript
createReading(dto: CreateReadingDto) {
  // Random 3 cards
  const shuffled = [...this.cards].sort(() => 0.5 - Math.random());
  const selectedCards = shuffled.slice(0, 3);
  
  return {
    question: dto.question,
    cards: [
      { position: 'past', card: selectedCards[0] },
      { position: 'present', card: selectedCards[1] },
      { position: 'future', card: selectedCards[2] }
    ],
    timestamp: new Date().toISOString()
  };
}
```

#### Bước 3: Thêm POST endpoint vào Controller

```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { CreateReadingDto } from './dto/reading.dto';

@Controller('tarot')
export class TarotController {
  // ... existing code

  @Post('reading')
  createReading(@Body() dto: CreateReadingDto) {
    return this.tarotService.createReading(dto);
  }
}
```

**Giải thích:**
- `@Post()`: POST request
- `@Body()`: Lấy data từ request body
- DTO: Define shape of data

#### Bước 4: Test với Postman

```
POST http://localhost:3000/tarot/reading
Content-Type: application/json

{
  "question": "Công việc của tôi sẽ như thế nào?",
  "spreadType": "three-card"
}
```

---

## ✅ CHECKLIST - BẠN ĐÃ HỌC ĐƯỢC

Sau khi hoàn thành 5 bài tập trên, bạn đã biết:

- [x] Decorators cơ bản: `@Controller`, `@Get`, `@Post`, `@Param`, `@Body`
- [x] Modules, Controllers, Services
- [x] Dependency Injection
- [x] Error handling với Exceptions
- [x] DTOs (Data Transfer Objects)
- [x] Route parameters
- [x] Request body handling

---

## 🎯 BƯỚC TIẾP THEO

Khi hoàn thành 5 bài tập này, báo mình biết!

Mình sẽ:
1. Review code của bạn
2. Giải đáp thắc mắc
3. Chuyển sang **GIAI ĐOẠN 2: DATABASE SETUP**

---

## 💡 TIPS

### Sử dụng Postman:
1. Download Postman Desktop
2. Tạo collection "Tarot API"
3. Lưu tất cả requests để test lại

### Hot Reload:
- Server tự restart khi save file
- Nếu không reload, stop và chạy lại `npm run start:dev`

### Debug:
- `console.log()` trong service để xem data
- Check terminal logs để xem errors
- Read error messages carefully!

### Keyboard Shortcuts:
- `Ctrl+C`: Stop server
- `Ctrl+Space`: Auto-complete trong VS Code

---

## ❓ CÂU HỎI THƯỜNG GẶP

**Q: Server không start được?**
A: Kiểm tra port 3000 có bị chiếm không. Hoặc change port trong `main.ts`

**Q: Code thay đổi nhưng không thấy kết quả?**
A: Check xem server có auto-reload không. Nếu không, restart server.

**Q: Import bị lỗi?**
A: Check đường dẫn import, phải đúng relative path

**Q: Không biết test API?**
A: Dùng Postman hoặc browser (cho GET requests)

---

## 🚀 BẮT ĐẦU NGAY!

```bash
# 1. Start server
npm run start:dev

# 2. Mở VS Code
code .

# 3. Bắt đầu làm Bài tập 1!
```

**Ready? Let's code!** 💪

Khi xong, show code cho mình xem nhé! 🎉
