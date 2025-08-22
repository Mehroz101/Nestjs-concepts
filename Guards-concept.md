# ✅ 3️⃣ Guards in NestJS (Authorization Layer)

### 🛡 What is a Guard?

A **Guard** is used to determine **if a request should be handled or blocked**.

* It's like a **gatekeeper**.
* Usually used for **authentication** (who you are) or **authorization** (what you're allowed to do).

---

## 🤔 Example Use Cases

| Scenario                           | Use Guard? |
| ---------------------------------- | ---------- |
| Allow only logged-in users         | ✅ Yes      |
| Allow only users with role "admin" | ✅ Yes      |
| Validate API key                   | ✅ Yes      |
| Check if user owns a resource      | ✅ Yes      |

---

## 💡 How It Works in Simple Words

When a request comes in:

* Middleware → ✅ Always runs
* **Guards** → Runs **before controller**
* If guard returns `true` → Continue to controller
* If guard returns `false` → Nest returns `403 Forbidden`

---

## 🔧 Let's Build a Simple Guard

### 🔹 Step 1: Create a Guard

```bash
nest g guard auth
```

📄 `src/auth.guard.ts`

```ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();

    // Example logic: Check if header "x-auth-token" exists
    const token = request.headers['x-auth-token'];

    if (token === '123456') {
      return true; // allow
    }

    return false; // block
  }
}
```

---

### 🔹 Step 2: Use the Guard in Controller

```ts
import { Controller, Get, UseGuards } from '@nestjs/common';
import { AuthGuard } from './auth.guard';

@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(AuthGuard)
  findAll() {
    return ['user1', 'user2'];
  }
}
```

Now, only requests with header `x-auth-token: 123456` will work.

---

## 🔐 Real-world Use Example (JWT Guard)

```ts
const token = request.headers['authorization']?.split(' ')[1];

try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  request.user = decoded;
  return true;
} catch {
  return false;
}
```

---

# ✅ 4️⃣ Global Guards + Skipping for Public Routes

Sometimes you don’t want to attach a guard to every controller manually. Instead, make it **global**.

📄 `app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { AuthGuard } from './auth.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: AuthGuard, // ✅ Global Guard
    },
  ],
})
export class AppModule {}
```

👉 Now, every route in the app is protected.

---

### 🔓 Skipping Guard for Public Routes

Sometimes you want routes like **login/register** to be open. We can do this with a **custom decorator** + `Reflector`.

📄 `public.decorator.ts`

```ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

📄 `auth.guard.ts`

```ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from './public.decorator';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // ✅ Check if route has @Public()
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true; // Skip guard
    }

    const request = context.switchToHttp().getRequest();
    const token = request.headers['x-auth-token'];

    return token === '123456';
  }
}
```

📄 `auth.controller.ts`

```ts
import { Controller, Get } from '@nestjs/common';
import { Public } from './public.decorator';

@Controller('auth')
export class AuthController {
  @Get('login')
  @Public() // ✅ No guard here
  login() {
    return { msg: 'Public login endpoint' };
  }

  @Get('profile')
  getProfile() {
    return { msg: 'Protected profile endpoint' };
  }
}
```

---

### 🔍 Summary

| Feature               | Guard Behavior                      |
| --------------------- | ----------------------------------- |
| Local Guard           | Attach manually via `@UseGuards()`  |
| Global Guard          | Applied everywhere with `APP_GUARD` |
| Skip with `@Public()` | Lets some routes bypass the guard   |


