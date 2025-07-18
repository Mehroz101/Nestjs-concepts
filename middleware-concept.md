# NestJS Middleware

## ✅ Why Use Middleware?


| Use Case	| Example  |
| :---------------- | :------: |
| Request Logging	| Log incoming HTTP requests |
 | Authentication | 	Check headers/tokens before controller | 
 | Request Validation | 	Custom checks before hitting a route | 
 | Modify Request | 	Add custom properties to req object | 



# 🧾 NestJS Logger Middleware Guide

This guide walks you through how to create, configure, and apply middleware in a NestJS project using Winston for logging.

---

## 1️⃣ Create a Middleware File

```bash
nest g middleware logger
```

---

## 2️⃣ Implement the Middleware

📄 `src/logger.middleware.ts`

```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const { method, originalUrl } = req;
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] ${method} - ${originalUrl}`);
    next(); // ✅ Important to continue to controller
  }
}
```

---

## 3️⃣ Apply Middleware in a Module

📄 `src/app.module.ts`

```ts
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { LoggerMiddleware } from './logger.middleware';
import { UsersModule } from './users/users.module';

@Module({
  imports: [UsersModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*'); // ✅ applies to ALL routes (you can also use specific routes)
  }
}
```

---

## ✅ Result

When you hit any route like `GET /users`, you’ll see this in the console:

```bash
[2025-07-17T15:45:30.123Z] GET - /users
```

---

## 👇 More Usage Examples

### 🛡️ Auth Middleware (Custom Token Check)

```ts
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers['authorization'];
    if (!token) {
      return res.status(401).json({ message: 'Unauthorized' });
    }
    next();
  }
}
```

---

# 🧩 NestJS Middleware Usage Options

| Use Case                   | How to Do It                       |
| -------------------------- | ---------------------------------- |
| For all routes (global)    | `forRoutes('*')`                   |
| For specific route         | `forRoutes('route')`               |
| For specific method+route  | `forRoutes({ path, method })`      |
| For a specific controller  | `forRoutes(ControllerClass)`       |
| For a specific module only | Add in that module's `configure()` |
| Exclude certain routes     | `.exclude()` method                |

---

## ✅ Basic Setup Reminder

📄 `logger.middleware.ts`

```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`[${req.method}] ${req.originalUrl}`);
    next();
  }
}
```

---

## 1️⃣ Apply to All Routes (Global)

```ts
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('*'); // applies to every route
}
```

---

## 2️⃣ Apply to Specific Route Only

```ts
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('users'); // applies to /users
}
```

---

## 3️⃣ Apply to Route + Method (like GET /users)

```ts
import { RequestMethod } from '@nestjs/common';

configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes({ path: 'users', method: RequestMethod.GET });
}
```

---

## 4️⃣ Apply to Specific Controller

```ts
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes(UsersController); // applies to all routes in this controller
}
```

---

## 5️⃣ Apply Inside a Specific Feature Module

📄 `users.module.ts`

```ts
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { LoggerMiddleware } from '../logger.middleware';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes(UsersController);
  }
}
```

---

## 6️⃣ Exclude Routes (Don't Run Middleware Here)

```ts
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .exclude('auth/login')
    .forRoutes('*');
}
```

Exclude multiple:

```ts
.exclude(
  { path: 'auth/login', method: RequestMethod.POST },
  { path: 'health', method: RequestMethod.GET },
)
```

---

## 🔍 Other Creative Uses of Middleware

| Purpose             | How Middleware Helps                       |
| ------------------- | ------------------------------------------ |
| Add custom property | `req.user = decodedJwt;`                   |
| Add request ID      | `req.id = uuid();`                         |
| Request profiling   | log start time and calculate response time |
| Block bad IPs       | check `req.ip` against blacklist           |

---

## 🧠 Summary

* **Nest middleware = Express-style but class-based**
* You can apply middleware:

  * 🔁 Globally
  * 🔀 Per route/controller
  * 🧩 Per module
  * 🛑 With exclusions
* Great for: logging, auth, validation, shaping `req`

---
