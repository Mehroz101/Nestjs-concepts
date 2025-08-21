# 🗄️ MongoDB + Mongoose in NestJS

## 1. Install Packages

```bash
npm install --save @nestjs/mongoose mongoose
```

---

## 2. Connect MongoDB

📄 `app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { UsersModule } from './users/users.module';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost:27017/nestjsdb'),
    UsersModule,
  ],
})
export class AppModule {}
```

* Connects to MongoDB at **localhost:27017**, DB = `nestjsdb`.

---

## 3. Define a Schema

📄 `user.schema.ts`

```ts
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

export type UserDocument = User & Document;

@Schema()
export class User {
  @Prop({ required: true })
  name: string;

  @Prop()
  age: number;

  @Prop({ default: Date.now })
  createdAt: Date;
}

export const UserSchema = SchemaFactory.createForClass(User);
```

---

## 4. Register Schema in Module

📄 `users.module.ts`

```ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User, UserSchema } from './user.schema';

@Module({
  imports: [
    MongooseModule.forFeature([{ name: User.name, schema: UserSchema }]),
  ],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

---

## 5. Use Schema in Service

📄 `users.service.ts`

```ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User, UserDocument } from './user.schema';

@Injectable()
export class UsersService {
  constructor(@InjectModel(User.name) private userModel: Model<UserDocument>) {}

  async getAll(): Promise<User[]> {
    return this.userModel.find().exec();
  }

  async getOne(id: string): Promise<User> {
    const user = await this.userModel.findById(id).exec();
    if (!user) throw new NotFoundException(`User with id ${id} not found`);
    return user;
  }

  async create(data: { name: string; age: number }): Promise<User> {
    const newUser = new this.userModel(data);
    return newUser.save();
  }
}
```

---

## 6. Controller

📄 `users.controller.ts`

```ts
import { Controller, Get, Post, Param, Body } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get()
  getAll() {
    return this.usersService.getAll();
  }

  @Get(':id')
  getOne(@Param('id') id: string) {
    return this.usersService.getOne(id);
  }

  @Post()
  create(@Body() body: { name: string; age: number }) {
    return this.usersService.create(body);
  }
}
```

---

## 7. Try it in Postman

* `POST http://localhost:3000/users`

  ```json
  {
    "name": "John",
    "age": 30
  }
  ```
* `GET http://localhost:3000/users`
* `GET http://localhost:3000/users/<mongo_id>`

---

## 🔑 Key Differences from TypeORM/Postgres

* No `entities`, instead we use `@Schema()` and `@Prop()`.
* Uses MongoDB `_id` instead of auto-increment `id`.
* Schema is flexible — you can add new props anytime.
* Relationships are handled differently (`ref`, `populate`).

---

✅ Now your NestJS API is connected to **MongoDB** using **Mongoose**.
