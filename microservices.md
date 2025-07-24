
---


# 📚 NestJS Microservices: Bookstore Example

This is a microservice-based architecture using **NestJS** with an **API Gateway**, two microservices (`users`, `books`), and **authentication-ready structure**.

---

## 🏁 Project Setup

### ✅ Initial Steps


# Step 1: Create main monorepo project
```bash
nest new bookstore
```
# Step 2: Create apps inside the project
```bash
nest generate app api-gateway
nest generate app users
nest generate app books
```

### 🧹 Step 3: Clean up boilerplate

Remove the initial `bookstore` app if not needed:

- Delete `src` under `apps/bookstore`
- Remove `"bookstore"` entry in `nest-cli.json`

### 📦 Step 4: Install microservices package

```bash
npm install @nestjs/microservices
```

---

## ⚙️ Microservice Bootstrap (TCP-based)

Each microservice should be bootstrapped using **`MicroserviceOptions`** with TCP transport.

### 📝 Example: `apps/users/src/main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
    transport: Transport.TCP,
    options: {
      host: '127.0.0.1',
      port: 4001,
    },
  });

  await app.listen();
}
bootstrap();
```

### 📌 Repeat similarly in `apps/books/src/main.ts` with a different port (e.g. 4002)

---

## 🌐 Gateway Bootstrap (HTTP + TCP clients)

### 📝 `apps/api-gateway/src/main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

---

## 📂 Structure Overview

```bash
bookstore/
├── apps/
│   ├── api-gateway/
│   ├── users/
│   └── books/
├── node_modules/
├── nest-cli.json
└── package.json
```

---

## 🚀 Generate Controllers & Services

### 🧾 In `users` microservice

```bash
cd apps/users
nest g controller users
nest g service users
```

### 🧾 In `books` microservice

```bash
cd apps/books
nest g controller books
nest g service books
```

### 🧾 In `api-gateway` — separate modules

```bash
cd apps/api-gateway

# Users module
nest g module users
nest g controller users
nest g service users

# Books module
nest g module books
nest g controller books
nest g service books
```

---

## 🔧 API Gateway Services → Microservices

Use `ClientProxy` to send messages from the gateway to microservices.

### 📝 `apps/api-gateway/src/users/users.service.ts`

```ts
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { lastValueFrom } from 'rxjs';

@Injectable()
export class UsersService {
  constructor(
    @Inject('USERS_SERVICE') private readonly client: ClientProxy
  ) {}

  getUser() {
    return lastValueFrom(this.client.send('get_user', {}));
  }
}
```

### 📝 `apps/api-gateway/src/users/users.controller.ts`

```ts
import { Controller, Get } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('user')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  getUser() {
    return this.usersService.getUser();
  }
}
```

---

## 🧠 Best Practices & Key Concepts

| Concept                      | Explanation |
|-----------------------------|-------------|
| ✅ Gateway Folder Split     | Keep logic separate per domain (`users`, `books`) |
| 📡 Microservice Entry       | Use TCP transport with `@MessagePattern()` |
| 🔁 Gateway to Microservice  | Use `ClientProxy.send()` with a pattern like `'get_user'` |
| 📦 Modules per Domain       | Each feature has its own module, controller, and service |
| ♻️ Reusability               | Use tokens like `'USERS_SERVICE'` in providers for DI |

---

## 📄 Example: Microservice Message Handler

### 📝 `apps/users/src/users/users.controller.ts`

```ts
import { Controller } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';
import { UsersService } from './users.service';

@Controller()
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @MessagePattern('get_user')
  getUser() {
    return this.usersService.getUser();
  }
}
```

---

## 📌 Notes

- Microservices (users/books) **do not use HTTP routes** — only `@MessagePattern()`
- Gateway maps **HTTP → TCP messages**
- Use environment-based config for host/port later
- Prepare for auth by isolating user login/register flows in gateway

---

## ✅ Summary of CLI Commands

```bash
nest new bookstore
nest g app api-gateway
nest g app users
nest g app books

# Remove initial app and install dependencies
npm i @nestjs/microservices

# In users
cd apps/users
nest g controller users
nest g service users

# In books
cd ../books
nest g controller books
nest g service books

# In gateway
cd ../api-gateway
nest g module users
nest g controller users
nest g service users

nest g module books
nest g controller books
nest g service books
```

---

Happy Microservicing! 🚀



