

---

## 🎬 What is an Interceptor?

An **interceptor** in NestJS is like a **middle layer** that sits **before and after** your controller logic.

You can think of it as a **"wrapper"** around your route handler.

---

## 🧠 Simple Analogy

Imagine you’re ordering food:

* 🍽️ You place an order (request).
* 👨‍🍳 The kitchen (controller) prepares your food.
* 🧼 But before giving it to you, someone adds **garnish** or **logs** how long it took (interceptor).

So, interceptors can:

* Transform input/output
* Add extra data (like timestamps)
* Log request durations
* Format responses
* Handle caching
* Bind extra metadata

---

## 📌 Interceptor Lifecycle

```txt
Request → Middleware → Guard → Interceptor → Controller → Interceptor (again) → Response
```

It runs:

* **Before** your route handler
* **After** your route handler (response can be modified)

---

## 🛠️ Real Example: Logging Request Duration

---

### 1. Create an Interceptor

```bash
nest g interceptor logger
```

You’ll get `logger.interceptor.ts`:

```ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggerInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    const req = context.switchToHttp().getRequest();
    const method = req.method;
    const url = req.url;

    return next
      .handle()
      .pipe(
        tap(() => {
          const time = Date.now() - now;
          console.log(`[${method}] ${url} - ${time}ms`);
        }),
      );
  }
}
```

---

### 🔍 Line-by-Line Explanation

```ts
intercept(context: ExecutionContext, next: CallHandler): Observable<any>
```

* `context`: gives access to request data.
* `next.handle()` → runs the controller logic.
* `.pipe(tap())` → lets you do something **after** the response.

```ts
const now = Date.now();
```

* Start the timer.

```ts
const req = context.switchToHttp().getRequest();
```

* Get Express request to access method and URL.

```ts
tap(() => {
  const time = Date.now() - now;
  console.log(`[${method}] ${url} - ${time}ms`);
})
```

* After controller is done, log how long it took.

---

### 2. Apply the Interceptor

#### Option A: Global (applies to all routes)

```ts
// main.ts
import { LoggerInterceptor } from './common/interceptors/logger.interceptor';

app.useGlobalInterceptors(new LoggerInterceptor());
```

#### Option B: Controller level

```ts
@UseInterceptors(LoggerInterceptor)
@Controller('users')
export class UsersController {
  @Get()
  getAll() {
    return ['user1', 'user2'];
  }
}
```

---

## 🧰 Other Use Cases for Interceptors

| Use Case              | Example                                               |
| --------------------- | ----------------------------------------------------- |
| ⏱ Log execution time  | Measure time taken by route                           |
| 🎁 Wrap responses     | Format success response `{ data: ..., status: 'ok' }` |
| 🔁 Transform results  | Map raw DB objects to DTO                             |
| 🗃 Cache responses    | Store result in memory/Redis                          |
| 🐞 Add error metadata | Tag more info when error is thrown                    |

---

## ✅ Example: Wrap All Responses

Want all your APIs to return in this format?

```json
{
  "status": "success",
  "data": [/* actual result */]
}
```

Create this interceptor:

```ts
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, any> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        status: 'success',
        data,
      })),
    );
  }
}
```

Apply it globally or to selected routes with `@UseInterceptors`.

---

## 🧠 Summary

| Feature     | Description                                           |
| ----------- | ----------------------------------------------------- |
| Purpose     | Wrap controller logic to transform input/output       |
| Runs        | Before and after controller                           |
| Common Uses | Logging, formatting response, caching, error wrapping |
| Implements  | `NestInterceptor` with `intercept()` method           |
| Applied via | `@UseInterceptors()` or globally in `main.ts`         |

---


* Or move to the **5th concept: Exception Filters**?

You're doing great so far — just tell me where to go next!
