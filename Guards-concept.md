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

### 🔍 Summary

| Feature  | Guard                               |
| -------- | ----------------------------------- |
| Purpose  | Decide if a request is allowed      |
| Runs     | Before route handler                |
| Returns  | `true` (allow) or `false` (block)   |
| Best For | Auth, Role checks, Ownership checks |

---
