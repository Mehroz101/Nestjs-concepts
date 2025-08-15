

## **1. Middleware**

* **When it runs:** Before a request reaches any route handler or guard.
* **Purpose:**

  * Works like Express middleware.
  * Good for logging, modifying requests, adding headers, parsing body, etc.
* **Scope:** Can be applied globally or to specific routes/modules.
* **Example:**

```ts
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`[${req.method}] ${req.url}`);
    next();
  }
}
```

---

## **2. Guards**

* **When it runs:** After middleware, before entering the route handler.
* **Purpose:**

  * **Decide whether the request is allowed or not** (return `true`/`false`).
  * Mostly used for **authentication** and **authorization**.
* **Scope:** Can be applied globally, at the controller, or at the route level.
* **Example:**

```ts
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const req = context.switchToHttp().getRequest();
    return !!req.user; // allow only if user exists
  }
}
```

---

## **3. Interceptors**

* **When it runs:** Wraps around the route handler **before and after** it executes.
* **Purpose:**

  * Modify **incoming** request or **outgoing** response.
  * Transform data, log execution time, add caching, format responses.
* **Scope:** Can be applied globally or per controller/route.
* **Example:**

```ts
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    return next.handle().pipe(
      map(data => ({ success: true, data }))
    );
  }
}
```

---

## **4. Pipes**

* **When it runs:** Right **before the route handler is called**, after guards/interceptors.
* **Purpose:**

  * **Transform** request data (e.g., strings to numbers).
  * **Validate** request data (throw error if invalid).
* **Custom Pipe Example:**

```ts
@Injectable()
export class ParseIntPipe implements PipeTransform {
  transform(value: any) {
    const val = parseInt(value, 10);
    if (isNaN(val)) throw new BadRequestException('Validation failed');
    return val;
  }
}
```

---

## **5. Parameter Decorators**

* **When it runs:** At the moment route handler parameters are resolved.
* **Purpose:**

  * Extract specific values from request objects easily.
  * Example: `@Param()`, `@Body()`, or **custom** decorators like `@AuthenticatedUser()`.
* **Custom Param Example:**

```ts
export const AuthenticatedUser = createParamDecorator(
  (data, ctx) => ctx.switchToHttp().getRequest().user
);
```

---

## **Order of Execution in HTTP Request**

Here’s how they line up for a single incoming HTTP request:

1. **Middleware** – preprocess requests (logging, parsing, etc.)
2. **Guards** – allow/deny access (auth checks)
3. **Interceptors (before)** – modify request or log before handler
4. **Pipes** – transform & validate request data
5. **Route Handler + Parameter Decorators** – controller action runs
6. **Interceptors (after)** – modify or format the response
7. **Response sent**

---

## **Quick Analogy**

Imagine NestJS is a **restaurant**:

* **Middleware:** Security check at the door + host checking your reservation.
* **Guard:** Bouncer checking if you’re allowed in (VIP list / ticket check).
* **Interceptor:** Waiter taking your order and later bringing your food arranged nicely.
* **Pipe:** Kitchen verifying ingredients & converting units before cooking.
* **Param Decorator:** Giving the chef exactly your special dietary request without reading the whole menu.

