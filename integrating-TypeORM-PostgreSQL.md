Awesome—here’s a **clean, scalable, production-ready template** for integrating **TypeORM + PostgreSQL** in NestJS, step by step. You can copy this structure into any new project.

---

# 0) Install & scaffold

```bash
# Nest + TypeORM + Postgres + Config + Validation
npm i @nestjs/typeorm typeorm pg @nestjs/config joi
# Optional but recommended
npm i typeorm-naming-strategies
# Dev deps for builds/migrations
npm i -D ts-node tsconfig-paths
```

---

# 1) Folder structure (scales well)

```
src/
  config/
    env.validation.ts
    database.config.ts
  database/
    data-source.ts            # TypeORM CLI datasource (JS at runtime in dist/)
    migrations/               # TS migration files (compiled to dist)
  modules/
    users/
      users.module.ts
      users.controller.ts
      users.service.ts
      user.entity.ts
      dto/
  app.module.ts
main.ts
```

---

# 2) Environment files (12-factor style)

`.env.development`

```
NODE_ENV=development
APP_PORT=3000

DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_NAME=nestjsdb
DB_SSL=false
DB_LOGGING=true
DB_POOL_SIZE=10
```

`.env.production` (example)

```
NODE_ENV=production
APP_PORT=3000

DB_HOST=<prod-host>
DB_PORT=5432
DB_USERNAME=<user>
DB_PASSWORD=<pass>
DB_NAME=<db>
DB_SSL=true
DB_LOGGING=false
DB_POOL_SIZE=20
```

---

# 3) ConfigModule + env validation (fail fast)

`src/config/env.validation.ts`

```ts
import * as Joi from 'joi';

export const envValidationSchema = Joi.object({
  NODE_ENV: Joi.string().valid('development','test','production').required(),
  APP_PORT: Joi.number().default(3000),

  DB_HOST: Joi.string().required(),
  DB_PORT: Joi.number().default(5432),
  DB_USERNAME: Joi.string().required(),
  DB_PASSWORD: Joi.string().allow(''),
  DB_NAME: Joi.string().required(),
  DB_SSL: Joi.boolean().default(false),
  DB_LOGGING: Joi.boolean().default(false),
  DB_POOL_SIZE: Joi.number().default(10),
});
```

`src/config/database.config.ts`

```ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  type: 'postgres' as const,
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT ?? '5432', 10),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  ssl: process.env.DB_SSL === 'true',
  logging: process.env.DB_LOGGING === 'true',
  poolSize: parseInt(process.env.DB_POOL_SIZE ?? '10', 10),
}));
```

---

# 4) Bootstrap AppModule with TypeORM (async, env-driven)

`src/app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import databaseConfig from './config/database.config';
import { envValidationSchema } from './config/env.validation';
import { SnakeNamingStrategy } from 'typeorm-naming-strategies';

// feature modules
import { UsersModule } from './modules/users/users.module';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: [`.env.${process.env.NODE_ENV || 'development'}`],
      load: [databaseConfig],
      validationSchema: envValidationSchema,
    }),

    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => {
        const db = config.get('database');
        return {
          type: 'postgres',
          host: db.host,
          port: db.port,
          username: db.username,
          password: db.password,
          database: db.database,
          // Best practice: migrations for schema; never sync in prod
          synchronize: false,
          autoLoadEntities: true, // ok in modular monoliths; disable if you prefer explicit entities
          logging: db.logging,
          ssl: db.ssl ? { rejectUnauthorized: false } : false,
          namingStrategy: new SnakeNamingStrategy(),
          extra: { max: db.poolSize }, // pool
        };
      },
    }),

    UsersModule,
  ],
})
export class AppModule {}
```

> **Why**:
>
> * `forRootAsync` keeps config centralized and testable.
> * `SnakeNamingStrategy` gives `snake_case` columns/indexes.
> * `synchronize: false` + migrations = safe schema evolution.
> * `autoLoadEntities: true` lets feature modules register entities automatically (great for scaling modules).

---

# 5) Create an entity (with scalable conventions)

`src/modules/users/user.entity.ts`

```ts
import {
  Entity, PrimaryGeneratedColumn, Column,
  CreateDateColumn, UpdateDateColumn, DeleteDateColumn, Index
} from 'typeorm';

@Entity({ name: 'users' })
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Index()
  @Column({ type: 'varchar', length: 120 })
  name: string;

  @Index({ unique: true })
  @Column({ type: 'varchar', length: 180 })
  email: string;

  @Column({ type: 'int', nullable: true })
  age: number | null;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  @DeleteDateColumn({ name: 'deleted_at', nullable: true })
  deletedAt?: Date | null;
}
```

> **Why**:
>
> * UUID ids scale well across services.
> * Indexed/unique columns for performance & integrity.
> * Timestamps + soft deletes are table-stakes in real apps.

---

# 6) Wire repository in the feature module

`src/modules/users/users.module.ts`

```ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])], // exposes Repository<User> for DI
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

`src/modules/users/users.service.ts`

```ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, DataSource } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User) private readonly users: Repository<User>,
    private readonly dataSource: DataSource, // for transactions when needed
  ) {}

  findAll() {
    return this.users.find();
  }

  async findOne(id: string) {
    const user = await this.users.findOne({ where: { id } });
    if (!user) throw new NotFoundException('User not found');
    return user;
  }

  create(payload: Partial<User>) {
    const entity = this.users.create(payload);
    return this.users.save(entity);
  }

  // Example transaction (bulk or multi-entity writes)
  async updateEmailTransactional(id: string, email: string) {
    const qr = this.dataSource.createQueryRunner();
    await qr.connect();
    await qr.startTransaction();
    try {
      const repo = qr.manager.getRepository(User);
      const user = await repo.findOne({ where: { id } });
      if (!user) throw new NotFoundException('User not found');
      user.email = email;
      await repo.save(user);
      await qr.commitTransaction();
      return user;
    } catch (e) {
      await qr.rollbackTransaction();
      throw e;
    } finally {
      await qr.release();
    }
  }
}
```

`src/modules/users/users.controller.ts`

```ts
import { Controller, Get, Param, Post, Body } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')
export class UsersController {
  constructor(private readonly users: UsersService) {}

  @Get()
  findAll() {
    return this.users.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.users.findOne(id);
  }

  @Post()
  create(@Body() dto: { name: string; email: string; age?: number }) {
    return this.users.create(dto);
  }
}
```

---

# 7) TypeORM CLI for migrations (safe schema changes)

**DataSource just for CLI** (reads `.env`, uses *compiled* JS paths):

`src/database/data-source.ts`

```ts
import 'reflect-metadata';
import { DataSource } from 'typeorm';
import * as path from 'path';
import * as dotenv from 'dotenv';

dotenv.config({ path: `.env.${process.env.NODE_ENV || 'development'}` });

export default new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432', 10),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  // point to compiled JS for entities & migrations
  entities: [path.join(__dirname, '../**/*.entity.js')],
  migrations: [path.join(__dirname, './migrations/*.js')],
  synchronize: false,
  logging: process.env.DB_LOGGING === 'true',
  ssl: process.env.DB_SSL === 'true' ? { rejectUnauthorized: false } : false,
});
```

**tsconfig note**
Make sure `tsconfig.json` compiles to `dist` and includes `src/**/*`.

**package.json scripts** (compile then run CLI on `dist`):

```json
{
  "scripts": {
    "build": "nest build",
    "typeorm": "typeorm",
    "migration:generate": "npm run build && typeorm migration:generate ./src/database/migrations/Init -d dist/database/data-source.js",
    "migration:run": "npm run build && typeorm migration:run -d dist/database/data-source.js",
    "migration:revert": "npm run build && typeorm migration:revert -d dist/database/data-source.js"
  }
}
```

Usage:

```bash
npm run migration:generate
npm run migration:run
npm run migration:revert
```

> **Why build first?** The CLI will execute using the compiled `dist/database/data-source.js` and the compiled JS migrations.

---

# 8) Production-grade toggles

* **Never** use `synchronize: true` in prod. Use **migrations**.
* Turn off SQL logging in prod (`DB_LOGGING=false`).
* Use SSL when your DB requires it (`DB_SSL=true`).
* Use proper **indexes** and **unique constraints** (see entity example).
* Prefer **UUID** primary keys for horizontally scalable systems.
* Keep **entities inside feature modules** to scale with your domain.
* For heavy reads, consider **query builder** + **indexes**.
* For multi-service systems, consider **outbox pattern** and **strict transactions**.

---

# 9) Optional goodies

* **Result caching** (TypeORM): `cache: { duration: 10000 }` on queries for read-heavy endpoints.
* **Health checks**: `@nestjs/terminus` to expose `/health` (DB ping).
* **Testing**: a `.env.test` and a separate test DB; load `ConfigModule` with that file.



