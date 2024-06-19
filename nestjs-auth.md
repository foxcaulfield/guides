## Preparing

- Init nestjs project

```sh
nest new .
```

- Install packages

```sh
npm i \
prisma \
@prisma/client \
passport \
passport-local \
passport-jwt \
@types/passport-local \
@types/passport-jwt \
class-validator \
class-transformer \
@nestjs/jwt \
@nestjs/passport \
bcrypt \
@types/bcrypt
```

- Init prisma

```sh
npx prisma init
```

- Write Users table model

```prisma
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

- Edit .env file (add keys for JWT)

```yml
JWT_SECRET = 'secretKey'
JWT_EXPIRES_IN = '1h'
```

- Generate prisma types

```sh
npx prisma generate
```

- [Optional] Make a migration

```sh
npx prisma migrate dev --name init
```

- Create two resources (**auth** and **users**) and one service (**database**)

```sh
nest generate resource auth
nest generate resource users
# nest generate resource products
nest generate service database
```

- Remove all the spec files

<!-- - Remove all the _dto_ and _entity_ folders -->

- [Optional] Remove _app.controller.ts_ and _app.service.ts_ files

- Create _strategies_ and _guards_ folderd inside _auth_ resource

**Structure so far**

```
./src/
├── app.controller.ts
├── app.module.ts
├── app.service.ts
├── auth
│   ├── auth.controller.ts
│   ├── auth.module.ts
│   ├── auth.service.ts
│   ├── dto
│   │   ├── auth-payload.dto.ts
│   │   └── sign-payload.dto.ts
│   ├── guards
│   │   ├── jwt.guard.ts
│   │   └── local.guard.ts
│   └── strategies
│       ├── jwt.strategy.ts
│       └── local.strategy.ts
├── database
│   └── database.service.ts
├── main.ts
├── products
│   ├── dto
│   │   ├── create-product.dto.ts
│   │   └── update-product.dto.ts
│   ├── entities
│   │   └── product.entity.ts
│   ├── products.controller.ts
│   ├── products.module.ts
│   └── products.service.ts
└── users
    ├── users.controller.ts
    ├── users.module.ts
    └── users.service.ts

```

## Content

- Content of **app.controller.ts**

```ts
import { Controller, Get } from "@nestjs/common";
import { AppService } from "./app.service";

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

- Content of **app.module.ts**

```ts
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { AuthModule } from "./auth/auth.module";
import { UsersModule } from "./users/users.module";
// import { ProductsModule } from './products/products.module';
import { DatabaseService } from "./database/database.service";

@Module({
  imports: [AuthModule, UsersModule /*ProductsModule*/],
  controllers: [AppController],
  providers: [AppService, DatabaseService],
})
export class AppModule {}
```

- Content of **app.service.ts**

```ts
import { Injectable } from "@nestjs/common";

@Injectable()
export class AppService {
  getHello(): string {
    return "Hello World!";
  }
}
```

- Content of **auth/auth.controller.ts**

```ts
import { Controller, Get, Post, Body, UseGuards } from "@nestjs/common";
import { AuthService } from "./auth.service";
import { LocalGuard } from "./guards/local.guard";
import { Prisma } from "@prisma/client";
import { JwtAuthGuard } from "./guards/jwt.guard";

@Controller("auth")
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post("register")
  async register(@Body() registerDto: Prisma.UsersCreateInput): Promise<any> {
    console.log("register", registerDto);
    return await this.authService.register(registerDto);
  }

  @Post("login")
  @UseGuards(LocalGuard)
  async login(@Body() loginDto: Prisma.UsersCreateInput) {
    console.log("login", loginDto);
    return await this.authService.login(loginDto);
  }

  @Get("status")
  @UseGuards(JwtAuthGuard)
  async status() {
    return "Ok";
  }
}

// @Controller('auth')
// export class AuthController {
//   constructor(private readonly authService: AuthService) {}

//   @Post()
//   create(@Body() createAuthDto: CreateAuthDto) {
//     return this.authService.create(createAuthDto);
//   }

//   @Get()
//   findAll() {
//     return this.authService.findAll();
//   }

//   @Get(':id')
//   findOne(@Param('id') id: string) {
//     return this.authService.findOne(+id);
//   }

//   @Patch(':id')
//   update(@Param('id') id: string, @Body() updateAuthDto: UpdateAuthDto) {
//     return this.authService.update(+id, updateAuthDto);
//   }

//   @Delete(':id')
//   remove(@Param('id') id: string) {
//     return this.authService.remove(+id);
//   }
// }
```

- Content of **auth/auth.module.ts**

```ts
import { Module } from "@nestjs/common";
import { AuthService } from "./auth.service";
import { AuthController } from "./auth.controller";
import { DatabaseService } from "src/database/database.service";
import { UsersModule } from "src/users/users.module";
import { PassportModule } from "@nestjs/passport";
import { JwtModule } from "@nestjs/jwt";
import { UsersService } from "src/users/users.service";
import { JwtStrategy } from "./strategies/jwt.strategy";
import { LocalStrategy } from "./strategies/local.strategy";

@Module({
  controllers: [AuthController],
  providers: [
    AuthService,
    DatabaseService,
    UsersService,
    LocalStrategy,
    JwtStrategy,
  ],
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: {
        expiresIn: process.env.JWT_EXPIRES_IN,
      },
    }),
  ],
})
export class AuthModule {}
```

- Content of **auth/auth.service.ts**

```ts
import { Injectable, NotFoundException } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";
import { Prisma } from "@prisma/client";
import * as bcrypt from "bcrypt";
import { DatabaseService } from "src/database/database.service";
import { UsersService } from "src/users/users.service";
import { AuthPayloadDto } from "./dto/auth-payload.dto";
import { SignPayloadDto } from "./dto/sign-payload.dto";
// import { CreateUserDto } from '../users/dto/create-user.dto';

@Injectable()
export class AuthService {
  constructor(
    private readonly databaseService: DatabaseService,
    private jwtService: JwtService,
    private readonly usersService: UsersService
  ) {}

  async login(loginDto: Prisma.UsersCreateInput) {
    console.log("auth service login", loginDto);
    const { username, password } = loginDto;

    const { password: _, ...validationResult } = await this.validateUser({
      username,
      password,
    });

    _ && null; // just because

    const payload: SignPayloadDto = { username };

    return {
      data: validationResult,
      token: this.jwtService.sign(payload),
    };
  }

  async register(createDto: Prisma.UsersCreateInput) {
    console.log("auth service register", createDto);

    createDto.password = await bcrypt.hash(createDto.password, 10);
    const { password: _, ...creationResult } = await this.usersService.create(
      createDto
    );

    _ && null; // just because

    const payload: SignPayloadDto = { username: creationResult.username };

    return {
      data: creationResult,
      token: this.jwtService.sign(payload),
    };
  }

  async validateUser({ username, password }: AuthPayloadDto) {
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

    return users;
  }
}

// import { Injectable } from '@nestjs/common';
// import { CreateAuthDto } from './dto/create-auth.dto';
// import { UpdateAuthDto } from './dto/update-auth.dto';

// @Injectable()
// export class AuthService {
//   create(createAuthDto: CreateAuthDto) {
//     return 'This action adds a new auth';
//   }

//   findAll() {
//     return `This action returns all auth`;
//   }

//   findOne(id: number) {
//     return `This action returns a #${id} auth`;
//   }

//   update(id: number, updateAuthDto: UpdateAuthDto) {
//     return `This action updates a #${id} auth`;
//   }

//   remove(id: number) {
//     return `This action removes a #${id} auth`;
//   }
// }
```

- Content of **auth/dto/auth-payload.dto.ts**

```ts
export class AuthPayloadDto {
  username: string;
  password: string;
}
```

- Content of **auth/dto/sign-payload.dto.ts**

```ts
export class SignPayloadDto {
  username: string;
}
```

- Content of **auth/guards/jwt.guard.ts**

```ts
import { ExecutionContext, Injectable } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";
import { Observable } from "rxjs";

@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") {
  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    console.log("Inside JwtAuthGuard canActivate");
    return super.canActivate(context);
  }
}
```

- Content of **auth/guards/local.guard.ts**

```ts
import { ExecutionContext, Injectable } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";
import { Observable } from "rxjs";

@Injectable()
export class LocalGuard extends AuthGuard("local") {
  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    console.log("Inside LocalGuard canActivate");
    return super.canActivate(context);
  }
}
```

- Content of **auth/strategies/jwt.strategy.ts**

```ts
import { Injectable } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { DatabaseService } from "src/database/database.service";
import { AuthService } from "../auth.service";
import { SignPayloadDto } from "../dto/sign-payload.dto";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private readonly databaseService: DatabaseService,
    private readonly authService: AuthService
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(userDto: SignPayloadDto) {
    return { username: userDto.username };
  }
}
```

- Content of **auth/strategies/local.strategy.ts**

```ts
import { PassportStrategy } from "@nestjs/passport";
import { Strategy } from "passport-local";
import { AuthService } from "../auth.service";
import { Injectable, UnauthorizedException } from "@nestjs/common";

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super();
  }

  async validate(username: string, password: string) {
    console.log("Inside LocalStrategy validate", username, password);
    const user = await this.authService.validateUser({ username, password });
    console.log("user", user);
    if (!user) throw new UnauthorizedException();
    return user;
  }
}
```

- Content of **database/database.service.ts**

```ts
import { Injectable, OnModuleDestroy, OnModuleInit } from "@nestjs/common";
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
}

//   async enableShutdownHooks(app: INestApplication) {
//     this.$on<INestApplication>('beforExit', async () => {
//       await app.close();
//     });
//   }
```

- Content of **main.ts**

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

- Content of **users/users.controller.ts**

```ts
import {
  Controller,
  Get,
  UseGuards,
  // Post,
  // Body,
  // Patch,
  // Param,
  // Delete,
} from "@nestjs/common";
import { UsersService } from "./users.service";
import { JwtAuthGuard } from "src/auth/guards/jwt.guard";

@Controller("users")
@UseGuards(JwtAuthGuard)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll() {
    return await this.usersService.findAll();
  }
}
// @Post()
// create(@Body() createUserDto: CreateUserDto) {
//   return this.usersService.create(createUserDto);
// }

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
```

- Content of **users/users.module.ts**

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

- Content of **users/users.service.ts**

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
```

## Irrelevant
<!--
## Edit database service

- Edit **database.service.ts**

```ts
import { Injectable, OnModuleDestroy, OnModuleInit } from "@nestjs/common";
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
}

//   async enableShutdownHooks(app: INestApplication) {
//     this.$on<INestApplication>('beforExit', async () => {
//       await app.close();
//     });
//   }
```

## Create strategies and guards

- JWT strategy

```ts
import { Injectable } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { DatabaseService } from "src/database/database.service";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private readonly databaseService: DatabaseService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    const users = await this.databaseService.users.findUnique({
      where: {
        username: payload.username,
      },
    });

    return users;
  }
}
```



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
``` -->
