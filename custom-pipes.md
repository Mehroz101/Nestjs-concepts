

## **1️⃣ What is a Pipe in NestJS?**

A **Pipe** in NestJS is a class that implements the `PipeTransform` interface and is used for:

* **Transforming** data (changing it before it reaches the controller).
* **Validating** data (throwing an error if it’s invalid).

Think of it like a **pre-processor** for incoming data.
If middleware is like a **security guard at the gate**, pipes are like the **ID checker** making sure your details are correct before you enter.

---

## **2️⃣ When do Pipes run?**

* After the request is routed to a specific **controller method**.
* Before your controller method is actually called.
* They run **on arguments** like `@Body()`, `@Param()`, `@Query()`.

---

## **3️⃣ Structure of a Custom Pipe**

Every custom pipe must:

1. Be a class.
2. Implement `PipeTransform`.
3. Have a `transform()` method.

---

## **4️⃣ Simple Example: Trim and Uppercase Pipe**

```ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class TrimAndUppercasePipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    if (typeof value !== 'string') {
      throw new BadRequestException('Value must be a string');
    }
    return value.trim().toUpperCase();
  }
}
```

**Usage in Controller:**

```ts
import { Controller, Get, Query } from '@nestjs/common';
import { TrimAndUppercasePipe } from './trim-uppercase.pipe';

@Controller('users')
export class UsersController {
  @Get()
  getUser(@Query('name', TrimAndUppercasePipe) name: string) {
    return { name };
  }
}
```

📌 If you call `/users?name= john ` → the pipe will return `"JOHN"` to your controller.

---

## **5️⃣ Different Ways to Use Custom Pipes**

### a) **Method Parameter Level** (most common)

```ts
getUser(@Param('id', MyPipe) id: string) { ... }
```

### b) **Method Level**

```ts
@UsePipes(MyPipe)
getUser(@Body() data) { ... }
```

### c) **Controller Level**

```ts
@UsePipes(MyPipe)
@Controller('users')
export class UsersController { ... }
```

### d) **Global Level** (in `main.ts`)

```ts
app.useGlobalPipes(new MyPipe());
```

---

## **6️⃣ Example: Validation Pipe (Custom)**

Instead of using `class-validator`, you can write your own.

```ts
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class PositiveIntPipe implements PipeTransform {
  transform(value: any) {
    const val = parseInt(value, 10);
    if (isNaN(val) || val <= 0) {
      throw new BadRequestException('Value must be a positive integer');
    }
    return val;
  }
}
```

**Usage:**

```ts
@Get(':id')
getUser(@Param('id', PositiveIntPipe) id: number) {
  return { id };
}
```

📌 `/users/5` ✅ works
📌 `/users/-1` ❌ throws 400 error.

---

## **7️⃣ Example: Transform Request Body**

```ts
@Injectable()
export class ParseJsonPipe implements PipeTransform {
  transform(value: any) {
    try {
      return JSON.parse(value);
    } catch {
      throw new BadRequestException('Invalid JSON format');
    }
  }
}
```

📌 Use when you expect JSON string but want an object in the controller.

---

## **8️⃣ When to Use Custom Pipes**

✅ Input validation without using class-validator.
✅ Transforming incoming data format.
✅ Sanitizing input before it hits your service layer.
✅ Enforcing specific formats (like converting dates to `Date` objects).


