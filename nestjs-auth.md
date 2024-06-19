## Preparing

- Init nestjs project

nest new .

- Install packages

npm i \
prisma \
@prisma/client
passport \
passport-local \
passport-jwt \
@types/passport-local \
@types/passport-jwt \
class-validator \
class-transformer \
@nestjs/jwt \
@nestjs/passport bcrypt @types/bcrypt \

- Init prisma
  npx prisma init

- Write Users table model

```ts


generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Users {
  id       Int    @id @default(autoincrement())
  username String @unique @db.VarChar(100)
  password String @db.VarChar(100)
  // firstName String @db.VarChar(100)
  // email String @email @db.VarChar(100)
}

```

- Generate prisma types

npx prisma generate

- Edit .env file (add keys for JWT)

```yml
JWT_SECRET = 'secretKey'
JWT_EXPIRES_IN = '1h'
```

- Create two resources (**auth** and **users**) and one service (**database**)
  nest generate resource auth
  nest generate resource users
  nest generate service database

- Remove all the spec files

- Remove all the _dto_ and _entity_ folders

- Remove _app.controller.ts_ and _app.service.ts_ files

## File editing

- Edit the _app.**module**.ts_ file

```ts
import { Module } from "@nestjs/common";

import { AuthModule } from "./auth/auth.module";
import { UsersModule } from "./users/users.module";
import { DatabaseService } from "./database/database.service";

@Module({
  imports: [AuthModule, UsersModule],
  controllers: [],
  providers: [DatabaseService],
})
export class AppModule {}
```

- Edit **database.service.ts**

```ts
import {
  //   INestApplication,
  Injectable,
  OnModuleDestroy,
  OnModuleInit,
} from "@nestjs/common";
import { PrismaClient } from "@prisma/client";

@Injectable()
export class DatabaseService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  public async onModuleInit() {
    return await this.$connect();
  }

  async onModuleDestroy() {
    return await this.$disconnect();
  }

  //   async enableShutdownHooks(app: INestApplication) {
  //     this.$on<INestApplication>('beforExit', async () => {
  //       await app.close();
  //     });
  //   }
}
```

### users

- Edit **users.service.ts**

```ts
import { Injectable } from "@nestjs/common";
import { Prisma, Users } from "@prisma/client";
import { DatabaseService } from "../database/database.service";

@Injectable()
export class UsersService {
  constructor(private databaseService: DatabaseService) {}

  async create(createUserDto: Prisma.UsersCreateInput): Promise<Users> {
    return this.databaseService.users.create({
      data: createUserDto,
    });
  }

  async findAll(): Promise<Users[]> {
    return this.databaseService.users.findMany();
  }

  // findOne(id: number) {
  //   return `This action returns a #${id} user`;
  // }

  // update(id: number, updateUserDto: UpdateUserDto) {
  //   return `This action updates a #${id} user`;
  // }

  // remove(id: number) {
  //   return `This action removes a #${id} user`;
  // }
}
```

- Edit **users.controller.ts**

```ts
import {
  Controller,
  Get,
  // Post,
  // Body,
  // Patch,
  // Param,
  // Delete,
} from "@nestjs/common";
import { UsersService } from "./users.service";

@Controller("users")
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // @Post()
  // create(@Body() createUserDto: CreateUserDto) {
  //   return this.usersService.create(createUserDto);
  // }

  @Get()
  async findAll() {
    return await this.usersService.findAll();
  }

  // @Get(':id')
  // findOne(@Param('id') id: string) {
  //   return this.usersService.findOne(+id);
  // }

  // @Patch(':id')
  // update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
  //   return this.usersService.update(+id, updateUserDto);
  // }

  // @Delete(':id')
  // remove(@Param('id') id: string) {
  //   return this.usersService.remove(+id);
  // }
}
```

- Edit **users.module.ts** (add database module to providers; getting an error otherwise)

```ts
import { Module } from "@nestjs/common";
import { UsersService } from "./users.service";
import { UsersController } from "./users.controller";
import { DatabaseService } from "src/database/database.service";

@Module({
  controllers: [UsersController],
  providers: [UsersService, DatabaseService],
})
export class UsersModule {}
```

### auth

- Edit **auth.guard.ts**

```ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { AuthGuard as PassportAuthGuard } from "@nestjs/passport";

@Injectable()
export class AuthGuard extends PassportAuthGuard("jwt") implements CanActivate {
  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    return super.canActivate(context);
  }

  handleRequest<TUser = any>(
    err: any,
    user: any
    // info: any,
    // context: ExecutionContext,
    // status?: any,
  ): TUser {
    if (err || !user) {
      throw err || new UnauthorizedException();
    }
    return user;
  }
}
```

- Edit **jwt.strategy.ts**

```ts
import { Injectable } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { DatabaseService } from "../database/database.service";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private readonly databaseService: DatabaseService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: { username: string }) {
    const users = await this.databaseService.users.findUnique({
      where: {
        username: payload.username,
      },
    });

    return users;
  }
}
```

- Content of **auth.service.ts**

```ts
import { Injectable, NotFoundException } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";
import { Prisma } from "@prisma/client";
import * as bcrypt from "bcrypt";
import { DatabaseService } from "src/database/database.service";
import { UsersService } from "src/users/users.service";
// import { CreateUserDto } from '../users/dto/create-user.dto';

@Injectable()
export class AuthService {
  constructor(
    private readonly databaseService: DatabaseService,
    private jwtService: JwtService,
    private readonly usersService: UsersService
  ) {}

  async login(loginDto: Prisma.UsersCreateInput): Promise<any> {
    const { username, password } = loginDto;

    const users = await this.databaseService.users.findUnique({
      where: { username },
    });

    if (!users) {
      throw new NotFoundException("user not found");
    }

    const validatePassword = await bcrypt.compare(password, users.password);

    if (!validatePassword) {
      throw new NotFoundException("Invalid password");
    }

    return {
      token: this.jwtService.sign({ username }),
    };
  }

  async register(createDto: Prisma.UsersCreateInput): Promise<any> {
    createDto.password = await bcrypt.hash(createDto.password, 10);

    const user = await this.usersService.create(createDto);

    return {
      token: this.jwtService.sign({ username: user.username }),
    };
  }
}
```

- Content of **auth.controller.ts**

```ts
import {
  Controller,
  // Get,
  Post,
  Body,
  // Patch,
  // Param,
  // Delete,
} from "@nestjs/common";
import { AuthService } from "./auth.service";
// import { CreateUserDto } from '../users/dto/create-user.dto';
import { Prisma } from "@prisma/client";

@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post("/login")
  async login(@Body() loginDto: Prisma.UsersCreateInput) {
    return await this.authService.login(loginDto);
  }

  @Post("/register")
  async register(@Body() registerDto: Prisma.UsersCreateInput): Promise<any> {
    return await this.authService.register(registerDto);
  }

  // @Post()
  // create(@Body() createAuthDto: CreateAuthDto) {
  //   return this.authService.create(createAuthDto);
  // }

  // @Get()
  // findAll() {
  //   return this.authService.findAll();
  // }

  // @Get(':id')
  // findOne(@Param('id') id: string) {
  //   return this.authService.findOne(+id);
  // }

  // @Patch(':id')
  // update(@Param('id') id: string, @Body() updateAuthDto: UpdateAuthDto) {
  //   return this.authService.update(+id, updateAuthDto);
  // }

  // @Delete(':id')
  // remove(@Param('id') id: string) {
  //   return this.authService.remove(+id);
  // }
}
```
