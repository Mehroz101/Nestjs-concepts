
## **1️⃣ What is Caching in NestJS?**

Caching is storing data temporarily so that subsequent requests can be served faster without recalculating or re-fetching from the database/API.

* **Without cache** → Every request hits the DB/API → Slow & costly.
* **With cache** → First request fetches data from DB/API and stores it → Later requests serve data from cache → Fast.

---

## **2️⃣ Caching Approaches in NestJS**

NestJS supports caching in multiple ways:

| Method                | Where It Lives         | Example Use Case                |
| --------------------- | ---------------------- | ------------------------------- |
| **In-memory cache**   | In server RAM          | Fast, but lost on restart       |
| **Distributed cache** | External store (Redis) | Works across multiple instances |
| **Manual caching**    | Your own logic         | Special scenarios               |

---

## \*\*3️⃣ Caching in NestJS using \*\***`CacheModule`**

NestJS provides **`CacheModule`** for quick caching.

### **Install dependencies**
```bash
npm install cache-manager @nestjs/cache-manager
```
### **Example – Simple In-Memory Cache**

```ts
// app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    CacheModule.register({
      ttl: 60, // cache time-to-live in seconds
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**Usage in service:**

```ts
import { Injectable, Inject } from '@nestjs/common';
import { Cache } from 'cache-manager';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager

@Injectable()
export class AppService {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}

  async setCacheKey(key: string, value: string) {
    await this.cacheManager.set(key, value, 60); // TTL 60s
  }

  async getCacheKey(key: string) {
    return await this.cacheManager.get(key);
  }
}
```

---

## **4️⃣ Using Redis with NestJS Cache**

To make caching **distributed** (works across multiple servers), use Redis.

### **Install dependencies**

```bash
npm install cache-manager-redis-store 
npm i -D  @types/cache-manager-redis-store
```

### **Register Redis in CacheModule**

```ts
// app.module.ts
import { Module } from '@nestjs/common';
import * as redisStore from 'cache-manager-redis-store';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: 'localhost',
      port: 6379,
      ttl: 600, // 10 minutes
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

---

## **5️⃣ Cache Interceptor (Automatic Caching)**

NestJS also has `CacheInterceptor` to **auto-cache GET responses**.

```ts
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { AppService } from './app.service';
import { CacheInterceptor, CacheKey } from '@nestjs/cache-manager';
@Controller('products')
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  constructor(private readonly appService: AppService) {}

  @Get()
  findAll() {
    return this.appService.getProducts();
  }
}
```

* When a GET request hits this endpoint, NestJS **automatically** caches the response.
* TTL is taken from `CacheModule` config.

---

## **6️⃣ Cache Manager + Redis in Microservices**

If you have **multiple services + API Gateway**, you want **all services to share the same Redis**.

Example:

* **Service A** → Writes to Redis
* **Service B** → Reads from Redis
* **API Gateway** → Can also read/write from the same Redis

**Just make sure**:

* All services use the same Redis connection config (host, port, password).
* Redis should be running in a separate container or instance accessible by all services.

---

## **📄 Best Practices**

1. **Use Redis for multi-instance apps** (in-memory cache will not sync between instances).
2. **Set proper TTL** to avoid stale data.
3. **Clear cache** when data changes (e.g., after DB update).
4. For large data, **store IDs in cache and fetch details on demand**.
5. Use **CacheInterceptor** only for read-heavy endpoints.


