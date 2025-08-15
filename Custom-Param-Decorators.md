
## **Custom Param Decorators in NestJS**

### **1️⃣ What is a Param Decorator?**

In NestJS, a **param decorator** is a function you attach to a controller method parameter to **extract specific data from the request object** (like `@Body()`, `@Param()`, `@Query()`).

A **Custom Param Decorator** lets you create your own version — for example, `@AuthenticatedUser()` or `@ClientIP()` — to keep controllers clean and avoid repeating request extraction logic.

---

### **2️⃣ Why Use Them?**

* Avoid repeating `req.user` or `req.headers` everywhere.
* Make code **more readable**.
* Keep controllers **focused on business logic** instead of request plumbing.
* Easy to reuse across multiple controllers.

---

### **3️⃣ How They Work**

* Built with NestJS helper **`createParamDecorator()`**.
* Use **`ExecutionContext`** to access the request.
* Return the extracted value → Nest injects it into the controller parameter.

---

### **4️⃣ Syntax**

```ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { Request } from 'express';

export const AuthenticatedUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest<Request>();
    return request.user;
  },
);
```

---

### **5️⃣ Example Usage**

**Controller:**

```ts
import { Controller, Get } from '@nestjs/common';
import { AuthenticatedUser } from './authenticated-user.decorator';

@Controller('profile')
export class ProfileController {
  @Get()
  getProfile(@AuthenticatedUser() user: any) {
    return { message: 'Your profile', user };
  }
}
```

If your authentication guard sets:

```ts
req.user = { id: 101, name: 'Jane Doe', role: 'admin' };
```

Then:

```
GET /profile
→ Response: { "message": "Your profile", "user": { id: 101, name: 'Jane Doe', role: 'admin' } }
```

---

### **6️⃣ You Can Pass Arguments**

`data` in `createParamDecorator` is the value you pass to the decorator.

```ts
export const GetUserField = createParamDecorator(
  (field: string, ctx: ExecutionContext) => {
    const req = ctx.switchToHttp().getRequest();
    return field ? req.user?.[field] : req.user;
  }
);
```

**Usage:**

```ts
getName(@GetUserField('name') name: string) {
  return `Hello ${name}`;
}
```

---

### **7️⃣ Real-World Examples**

* `@AuthenticatedUser()` → returns the full logged-in user.
* `@GetUserField('email')` → returns just the email.
* `@ClientIP()` → returns `req.ip`.
* `@AuthToken()` → returns the JWT from headers.

---

### **8️⃣ Key Differences from Pipes**

| **Custom Param Decorator**  | **Custom Pipe**                                   |
| --------------------------- | ------------------------------------------------- |
| Injects data from request   | Transforms/validates data                         |
| Works before method runs    | Works before method runs, but after value is read |
| Does not validate by itself | Usually validates/transforms                      |

