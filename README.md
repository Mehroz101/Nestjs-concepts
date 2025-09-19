## 📂 Folder Structure

```text
project-root/
├─ src/
│  ├─ main.ts
│  ├─ app.module.ts
│  ├─ config/
│  │  ├─ configuration.ts
│  │  ├─ validation.ts
│  │  └─ app.config.ts
│  ├─ common/
│  │  ├─ decorators/
│  │  ├─ filters/
│  │  ├─ guards/
│  │  ├─ interceptors/
│  │  ├─ middleware/
│  │  └─ pipes/
│  ├─ database/
│  │  ├─ database.module.ts
│  │  ├─ database.service.ts
│  │  ├─ naming.strategy.ts
│  │  └─ entities/
│  ├─ modules/
│  │  ├─ users/
│  │  │  ├─ users.module.ts
│  │  │  ├─ users.controller.ts
│  │  │  ├─ users.service.ts
│  │  │  ├─ dto/
│  │  │  ├─ entities/
│  │  │  ├─ repositories/
│  │  │  └─ tests/
│  │  ├─ auth/
│  │  ├─ products/
│  │  └─ orders/
│  ├─ shared/
│  └─ utils/
├─ test/
├─ migrations/
├─ .env
├─ package.json
└─ tsconfig.json
```

## 📚 What to Learn Next (with WHY)

### 1️⃣ **Exception Handling (Filters)**

**Why?** To catch and customize errors globally (like 404s, custom messages, etc.)

* Learn: `@Catch()`, `HttpException`, `ExceptionFilter`
* Goal: Return clean, structured error messages.

🔗 Bonus: Create a `GlobalExceptionFilter`.

---

### 2️⃣ **Middleware**

**Why?** To run code **before** the request hits your controller (e.g., logging, auth tokens)

* Learn: `NestMiddleware`, `app.use()`, applying to routes

Use cases:

* Logging all requests
* Modifying request headers
* Checking IPs, etc.

---

### 3️⃣ **Guards (Authorization)**

**Why?** To **protect routes** — like checking JWT tokens or user roles

* Learn: `@UseGuards()`, `CanActivate`, `AuthGuard`, `RolesGuard`

Use cases:

* Admin-only access
* JWT verification
* Role-based permissions

---

### 4️⃣ **Interceptors**

**Why?** To modify **responses** or **wrap logic** around method execution (e.g., timing, formatting)

* Learn: `@UseInterceptors()`, `NestInterceptor`, `ExecutionContext`

Use cases:

* Add response wrapper
* Measure API time
* Transform data shape

---

### 5️⃣ **Custom Pipes**

**Why?** To validate/transform input data before your controller uses it

* Learn: `PipeTransform`, `@UsePipes()`

Use cases:

* Convert strings to numbers
* Sanitize inputs
* Parse enums

---

### 6️⃣ **MongoDB or PostgreSQL Integration**

**Why?** To connect your APIs to a real database

Options:

* MongoDB → with **Mongoose** (ODM)
* PostgreSQL → with **TypeORM** or **Prisma**

Choose one and learn:

* How to connect
* Define schemas/entities
* Use inside services

---

### 7️⃣ **Authentication (JWT + Passport)**

**Why?** To add login/register functionality and secure routes

Learn:

* `@nestjs/passport`
* JWT tokens
* LocalStrategy
* AuthService + AuthGuard

---

### 8️⃣ **Global Pipes, Guards, Filters**

**Why?** So you don’t repeat code in every route

Learn how to:

* Register them globally in `main.ts`

---

### 9️⃣ **Testing (Unit + E2E)**

**Why?** To build reliable apps and work in teams

* Learn: `jest`, `supertest`, `@nestjs/testing`

---

### 🔟 **Deployment + Environment**

**Why?** To run the app in production

Learn:

* Using `.env`
* ConfigModule
* Production build (`npm run build`)
* Running on server (PM2, Docker, etc.)

---

## 🗂 Suggested Learning Path

1. ✅ Filters
2. ✅ Middleware
3. ✅ Guards (Auth)
4. ✅ Interceptors
5. ✅ Pipes
6. ✅ Connect to Database (MongoDB or PostgreSQL)
7. ✅ JWT Auth (Login/Register)
8. ✅ Global modules & config
9. ✅ Testing (optional)
10. ✅ Deployment

---

