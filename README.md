# Using Winston Logger in NestJS Project

This guide explains how to set up and use **Winston logger** in a NestJS project for centralized, powerful logging with separate log files and context-aware logs.

---

## 1. Install Dependencies

```bash
npm install --save winston nest-winston
```

---

## 2. Configure Winston Logger

Create a config file at `src/logger/winston-logger.ts`:

```ts
// src/logger/winston-logger.ts
import { WinstonModuleOptions } from 'nest-winston';
import * as winston from 'winston';
import * as path from 'path';

export const winstonLoggerConfig: WinstonModuleOptions = {
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.printf(({ timestamp, level, message }) => {
      return `[${timestamp}] [${level.toUpperCase()}] ${message}`;
    }),
  ),
  transports: [
    new winston.transports.Console({
      format: winston.format.simple(),
    }),
    new winston.transports.File({
      filename: path.join(__dirname, '../../logs/error.log'),
      level: 'error',
    }),
    new winston.transports.File({
      filename: path.join(__dirname, '../../logs/combined.log'),
    }),
  ],
};

```

---



---

## 3. Setup Winston Logger Globally in `main.ts`

Modify `main.ts` to use the Winston logger globally:

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { WinstonModule } from 'nest-winston';
import { winstonLoggerConfig } from './logger/winston-logger';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(winstonLoggerConfig),
  });

  app.useGlobalPipes(new ValidationPipe());

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

---

## 4. Using Logger in Middleware

Since middleware runs outside dependency injection, use NestJS built-in `Logger` class:

```ts
import { Injectable, Logger, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class RequestTimerMiddleware implements NestMiddleware {
  private readonly logger = new Logger(RequestTimerMiddleware.name);

  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();

    res.on('finish', () => {
      const duration = Date.now() - start;
      const { method, originalUrl } = req;
      const { statusCode } = res;

      this.logger.log(`${method} ${originalUrl} ${statusCode} - ${duration}ms`);
    });

    next();
  }
}
```

Logs from this middleware will be routed through the global Winston logger setup.

---

## 6. Using Logger in Services or Controllers

Inject your custom `LoggerService`:

```ts
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(RequestTimerMiddleware.name);

  getAllUsers() {
    this.logger.log('Fetching all users', UsersService.name);
    // Your code here
  }
}
```

---

## Summary

- Setup Winston transports and formats in `winston-logger.ts`.
- Register Winston globally in `main.ts`.
- Use NestJS `Logger` class in middleware.
- Inject and use `LoggerService` in services/controllers for structured logging.

---

**Happy Logging! 🚀**
