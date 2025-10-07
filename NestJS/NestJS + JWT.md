# Install dependencies

```sh
npm i @nestjs/jwt @nestjs/passport passport passport-jwt passport-local bcrypt
```

```sh
npm i -D @types/passport-jwt @types/passport-local @types/bcrypt
```

<br/>

# Add JWT secret to .env

<details>
<summary>.env</summary>

```dotenv
JWT_SECRET=secret
```

</details>

<br/>

# Create JWT payload interface

<details>
<summary>src/interfaces/jwt-payload.interface.ts</summary>

```ts
export interface JwtPayload {
  userId: string;
  email: string;
}
```

</details>

<br/>

# Create a JWT strategy

This is where exactly JWT tokens are validated and decoded when JwtAuthGuard is used.

<details>
<summary>src/strategies/jwt.strategy.ts</summary>

```ts
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { PassportStrategy } from "@nestjs/passport";
import {
  ExtractJwt,
  Strategy as PassportJwtStrategyEntity,
} from "passport-jwt";
import { JwtPayload } from "src/interfaces/jwt-payload.interface";

@Injectable()
export class JwtStrategy extends PassportStrategy(
  PassportJwtStrategyEntity,
  "jwt"
) {
  public constructor(private readonly configService: ConfigService) {
    const secret = configService?.get<string>("JWT_SECRET");
    if (!secret) {
      throw new Error("JWT_SECRET is not defined in environment variables");
    }
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: secret,
    });
  }

  public validate(payload: JwtPayload): JwtPayload {
    console.log("JwtStrategy, payload:", payload);
    if (!payload.userId || !payload.email) {
      throw new Error("Invalid JWT payload");
    }

    return { userId: payload.userId, email: payload.email };
  }

  public diagnostic(): void {
    console.log(
      "JWT Strategy initialized with secret:",
      this.configService.get<string>("JWT_SECRET")
    );
  }
}
```

</details>

<br/>

# Create JWT guard

This is what used in controllers to protect routes (endpoints). They call strategies to validate requests under the hood.

> > Note: they simply extend AuthGuard from @nestjs/passport

<!-- _Local guard_

<details>
<summary>src/guards/local-auth.guard.ts</summary>

```ts
import { CanActivate, Injectable } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";

@Injectable()
export class LocalAuthGuard extends AuthGuard("local") implements CanActivate {}
```

</details> -->

_JWT guard_

<details>
<summary>src/guards/jwt-auth.guard.ts</summary>

```ts
import { CanActivate, Injectable } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";

@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") implements CanActivate {}
```

</details>

<br/>

# Create USERS resource

### Model

<details>
<summary>src/users/user.model.ts</summary>

```ts
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { HydratedDocument, Model } from "mongoose";

export enum UserRoleEnum {
  USER = "USER",
  ADMIN = "ADMIN",
}

@Schema()
export class User {
  @Prop({
    type: String,
    unique: true,
  })
  public email!: string;

  @Prop({
    type: String,
  })
  public passwordHash!: string;

  @Prop({
    type: String,
  })
  public fullname!: string;

  @Prop({
    type: String,
  })
  public phone!: string;

  @Prop({
    type: String,
    enum: UserRoleEnum,
    default: UserRoleEnum.USER,
  })
  public role!: UserRoleEnum;
}

export type UserDocument = HydratedDocument<User>;
export type UserModelType = Model<UserDocument>;
export const UserSchema = SchemaFactory.createForClass(User);
```

</details>

### DTOs

<details>
<summary>src/users/dto/*</summary>

```ts
import { IsString, MinLength } from "class-validator";
// import { UserRoleEnum } from "../user.model";

export class CreateUserDto {
  @IsString()
  public email!: string;

  @MinLength(6)
  @IsString()
  public password!: string;

  @IsString()
  public fullname!: string;

  @IsString()
  public phone!: string;

  // @IsEnum(UserRoleEnum)
  // public role!: UserRoleEnum;
}
```

```ts
import { PartialType } from "@nestjs/swagger";
import { CreateUserDto } from "./create-user.dto";

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

```ts
import { Expose, Transform } from "class-transformer";
import { UserRoleEnum } from "../user.model";

interface HasToString {
  toString(): string;
}

interface HasId {
  id?: string;
  _id?: HasToString;
}

interface ArgWithObjWithId {
  obj: HasId;
}

export class ResponseUserDto {
  @Transform(
    ({ obj }: ArgWithObjWithId): string => obj._id?.toString() ?? obj.id ?? ""
  )
  @Expose()
  public id!: string;

  @Expose()
  public email!: string;

  @Expose()
  public fullname!: string;

  @Expose()
  public phone!: string;

  @Expose()
  public role!: UserRoleEnum;
}
```

```ts
import {
  Max,
  Min,
  IsInt,
  IsPositive,
  IsNumber,
  IsOptional,
} from "class-validator";

export class FilterUserDto {
  @Max(100)
  @Min(1)
  @IsInt()
  @IsPositive()
  @IsNumber()
  @IsOptional()
  public limit?: number = 10;
}
```

</details>

### Module

<details>
<summary>src/users/users.module.ts</summary>

```ts
import { Module } from "@nestjs/common";
import { UsersService } from "./users.service";
import { UsersController } from "./users.controller";
import { MongooseModule } from "@nestjs/mongoose";
import { User, UserSchema } from "./user.model";
import { CurrentUserPipe } from "src/pipes/current-user.pipe";

@Module({
  imports: [
    MongooseModule.forFeature([
      {
        name: User.name,
        schema: UserSchema,
      },
    ]),
  ],
  providers: [UsersService /*, CurrentUserPipe */],
  controllers: [UsersController],
  exports: [UsersService],
})
export class UsersModule {}
```

</details>

### Service

<details>
<summary>src/users/users.service.ts</summary>

```ts
import {
  ConflictException,
  Injectable,
  NotFoundException,
} from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { User, UserDocument, type UserModelType } from "./user.model";
import { CreateUserDto } from "./dto/create-user.dto";
import { ResponseUserDto } from "./dto/response-user.dto";
// import { plainToInstance } from "class-transformer";
import * as bcrypt from "bcrypt";
import { UpdateUserDto } from "./dto/update-user.dto";

@Injectable()
export class UsersService {
  private toResponseDto(entity: UserDocument): ResponseUserDto;
  private toResponseDto(entity: UserDocument[]): ResponseUserDto[];
  private toResponseDto(
    entity: UserDocument | UserDocument[]
  ): ResponseUserDto | ResponseUserDto[] {
    if (Array.isArray(entity)) {
      return entity.map(
        (user): ResponseUserDto => this.convertToResponseDto(user)
      );
    }
    return this.convertToResponseDto(entity);
  }

  private convertToResponseDto(user: UserDocument): ResponseUserDto {
    return {
      id: user._id.toString(),
      email: user.email,
      fullname: user.fullname,
      phone: user.phone,
      role: user.role,
    };
  }

  public constructor(
    @InjectModel(User.name) private readonly userModel: UserModelType
  ) {}
  public async createUser(dto: CreateUserDto): Promise<ResponseUserDto> {
    const { email, password, ...rest } = dto;

    const existingUser = await this.userModel.findOne({ email });

    if (existingUser) {
      throw new ConflictException("User with this email already exists");
    }

    const saltRounds = 10;
    const passwordHash = await bcrypt.hash(password, saltRounds);

    // const createdUser = await this.userModel.create(dto);
    const createdUser = new this.userModel({
      email,
      passwordHash,
      ...rest,
    });

    await createdUser.save();

    return this.toResponseDto(createdUser);
  }

  /* Admin */
  public async findByEmail(email: string): Promise<UserDocument> {
    const foundUser = await this.userModel.findOne({ email });

    if (!foundUser) {
      throw new NotFoundException();
    }

    return foundUser;
  }

  public async findById(id: string): Promise<ResponseUserDto> {
    console.log("UsersService, method findById, id:", id);
    const foundUser = await this.userModel.findById(id);

    if (!foundUser) {
      throw new NotFoundException();
    }

    return this.toResponseDto(foundUser);
  }

  public async validatePassword(
    userPasswordHash: string,
    incomingPassword: string
  ): Promise<boolean> {
    return await bcrypt.compare(incomingPassword, userPasswordHash);
  }

  public async update(
    id: string,
    dto: UpdateUserDto
  ): Promise<ResponseUserDto> {
    const user = await this.userModel.findByIdAndUpdate(id, dto, { new: true });

    if (!user) {
      throw new NotFoundException("User not found");
    }

    return this.toResponseDto(user);
  }

  public async findAll(limit?: number): Promise<ResponseUserDto[]> {
    const query = this.userModel.find();
    if (limit) {
      query.limit(limit);
    }

    const result = await query.exec();

    return this.toResponseDto(result);
  }
}
```

</details>

### Current user pipe

<details>
<summary>src/pipes/current-user.pipe.ts</summary>

```ts
import {
  Injectable,
  PipeTransform,
  ArgumentMetadata,
  UnauthorizedException,
} from "@nestjs/common";
import { UsersService } from "src/users/users.service";
import { JwtPayload } from "src/interfaces/jwt-payload.interface";
import { ResponseUserDto } from "src/users/dto/response-user.dto";

@Injectable()
export class CurrentUserPipe implements PipeTransform {
  public constructor(private readonly usersService: UsersService) {}

  public async transform(
    value: JwtPayload,
    _metadata: ArgumentMetadata
  ): Promise<ResponseUserDto> {
    console.log("metadata:", _metadata);
    console.log("CurrentUserPipe, value:", value);

    if (!value?.userId) {
      throw new UnauthorizedException("No user id in JWT payload");
    }
    // Find user by id (sub)
    const user = await this.usersService.findById(value.userId);
    if (!user) {
      throw new UnauthorizedException("User not found");
    }
    return user;
  }
}
```

</details>

<br/>

> Then update users.module.ts again (add CurrentUserPipe import)

```ts
@Module({
  imports: [
    MongooseModule.forFeature([
      {
        name: User.name,
        schema: UserSchema,
      },
    ]),
  ],
  providers: [UsersService, CurrentUserPipe], // <--
  controllers: [UsersController],
  exports: [UsersService],
})
export class UsersModule {}
```

### Current user decorator

> To use with CurrentUserPipe

<details>
<summary>src/decorators/current-user.decorator.ts</summary>

```ts
import { createParamDecorator, ExecutionContext } from "@nestjs/common";
import { JwtPayload } from "src/interfaces/jwt-payload.interface";

export const CurrentUser = createParamDecorator(
  (
    data: keyof JwtPayload | undefined,
    ctx: ExecutionContext
  ): string | JwtPayload | undefined => {
    console.log("CurrentUser decorator data:", data);
    const request = ctx.switchToHttp().getRequest<{ user?: JwtPayload }>();
    console.log("CurrentUser decorator request.user:", request.user);
    const user = request.user;
    return data && user ? user[data] : user;
  }
);
```

</details>

### Controller

<details>
<summary>src/users/users.controller.ts</summary>

```ts
/* eslint-disable @typescript-eslint/no-unsafe-member-access */
import {
  Controller,
  Get,
  NotFoundException,
  Param,
  UseGuards,
} from "@nestjs/common";
import { JwtAuthGuard } from "src/guards/jwt-auth.guard";
import { UsersService } from "./users.service";
import { ResponseUserDto } from "./dto/response-user.dto";
import { UserRoleEnum } from "./user.model";
import { CurrentUser } from "src/decorators/current-user.decorator";
import { CurrentUserPipe } from "src/pipes/current-user.pipe";

export interface User {
  _id?: { toString(): string };
  id?: string;
  email: string;
  fullname?: string;
  phone?: string;
  role?: UserRoleEnum;
}

@Controller("users")
export class UsersController {
  public constructor(private readonly usersService: UsersService) {}

  @UseGuards(JwtAuthGuard)
  @Get("profile")
  public getProfile(
    @CurrentUser(undefined, CurrentUserPipe) user: ResponseUserDto
  ): ResponseUserDto {
    console.log("controller 'users', method 'getProfile', user:", user);
    return {
      id: user.id,
      email: user.email,
      role: user.role,
      fullname: user?.fullname || "FULLNAME NOT PROVIDED",
      phone: user?.phone || "PHONE NOT FOUND",
    };
  }

  @Get(":id")
  @UseGuards(JwtAuthGuard)
  public async findOne(@Param("id") id: string): Promise<ResponseUserDto> {
    const user = await this.usersService.findById(id);
    if (!user) {
      throw new NotFoundException("User not found");
    }
    return {
      id: user.id,
      email: user.email,
      fullname: user.fullname,
      phone: user.phone,
      role: user.role,
    };
  }
}
```

</details>

Then add the controller to `users.module.ts` (it should be already there if you used CLI to generate the resource).

```ts
@Module({
  // ...
  providers: [UsersService, CurrentUserPipe],
  controllers: [UsersController], // <--
  exports: [UsersService],
})
export class UsersModule {}
```

<br/>

# Create AUTH resource

### DTO

<details>
<summary>src/auth/dto/login.dto.ts</summary>

```ts
import { IsString, IsEmail, MinLength } from "class-validator";

export class LoginDto {
  @IsEmail()
  public email!: string;

  @IsString()
  @MinLength(6)
  public password!: string;
}
```

</details>

### Add a local guard

This is what used in controllers to protect routes (endpoints). They call strategies to validate requests under the hood.

> > Note: they simply extend AuthGuard from @nestjs/passport

_Local guard_

<details>
<summary>src/guards/local-auth.guard.ts</summary>

```ts
import { CanActivate, Injectable } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";

@Injectable()
export class LocalAuthGuard extends AuthGuard("local") implements CanActivate {}
```

</details>

### Module

Add JwtModule with secret and expiration options. Use `registerAsync` to load secret from .env via ConfigService.

<details>
<summary>src/auth/auth.module.ts</summary>

```ts
import { Module } from "@nestjs/common";
// import { AuthController } from "./auth.controller";
// import { AuthService } from "./auth.service";
import { UsersModule } from "src/users/users.module";
// import { LocalStrategy } from "src/strategies/local.strategy";
import { JwtModule, JwtModuleOptions } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
// import { JwtStrategy } from "src/strategies/jwt.strategy";
import { ConfigModule, ConfigService } from "@nestjs/config";

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (): JwtModuleOptions => {
        return {
          secret: process.env.JWT_SECRET,
          signOptions: {
            expiresIn: "60m",
          },
        };
      },
    }),
    ConfigModule,
  ],
  // controllers: [AuthController],
  // providers: [AuthService , LocalStrategy, JwtStrategy],
})
export class AuthModule {}
```

</details>

### Service

<details>
<summary>src/auth/auth.service.ts</summary>

```ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";
import { UsersService } from "src/users/users.service";
import { LoginDto } from "./dto/login.dto";
import { CreateUserDto } from "src/users/dto/create-user.dto";
import { ResponseUserDto } from "src/users/dto/response-user.dto";
import { JwtPayload } from "src/interfaces/jwt-payload.interface";

@Injectable()
export class AuthService {
  public constructor(
    private readonly usersService: UsersService,
    private readonly jwtService: JwtService
  ) {}

  public async validateUser(
    email: string,
    password: string
  ): Promise<JwtPayload | null> {
    console.log("AuthService, method validateUser, email:", email);
    const user = await this.usersService.findByEmail(email);
    const isValid = await this.usersService.validatePassword(
      user.passwordHash,
      password
    );
    if (isValid) {
      return {
        userId: user._id.toString(),
        email: user.email,
      };
    }

    // if (user && user.password === password) {
    // 	const { password, ...result } = user;
    // 	return result;
    // }

    return null;
  }

  public async logUserIn(dto: LoginDto): Promise<{ access_token: string }> {
    console.log("AuthService, method logUserIn, dto:", dto);
    const userData = await this.validateUser(dto.email, dto.password);

    if (!userData) {
      throw new UnauthorizedException("Invalid credentials");
    }
    const payload: JwtPayload = {
      email: userData.email,
      userId: userData.userId,
    };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }

  public async register(
    dto: CreateUserDto
  ): Promise<{ access_token: string; user: ResponseUserDto }> {
    const user = await this.usersService.createUser(dto);
    // const { passwordHash, ...result } = user;

    const payload = { email: user.email, /* sub: user._id, */ role: user.role };

    return {
      access_token: this.jwtService.sign(payload),
      user: user,
    };
  }
}
```

</details>

### Add local strategy

<details>
<summary>src/strategies/local.strategy.ts</summary>

```ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { Strategy as PassportLocalStrategyEntity } from "passport-local";
import { AuthService } from "src/auth/auth.service";
import { JwtPayload } from "src/interfaces/jwt-payload.interface";

@Injectable()
export class LocalStrategy extends PassportStrategy(
  PassportLocalStrategyEntity,
  "local"
) {
  public constructor(private readonly authService: AuthService) {
    super({
      usernameField: "email",
      passwordField: "password",
    });
  }

  public async validate(email: string, password: string): Promise<JwtPayload> {
    console.log("LocalStrategy, validate method, email:", email);
    const user = await this.authService.validateUser(email, password);

    if (!user) {
      throw new UnauthorizedException();
    }

    return user;
  }
}
```

</details>

### Controller

<details>
<summary>src/auth/auth.controller.ts</summary>

```ts
import { Body, Controller, Post, UseGuards } from "@nestjs/common";
import { LocalAuthGuard } from "src/guards/local-auth.guard";
import { AuthService } from "./auth.service";
import { LoginDto } from "./dto/login.dto";
import { CreateUserDto } from "src/users/dto/create-user.dto";
import { ResponseUserDto } from "src/users/dto/response-user.dto";

@Controller("auth")
export class AuthController {
  public constructor(private readonly authService: AuthService) {}

  @UseGuards(LocalAuthGuard)
  @Post("login")
  public async login(
    @Body() loginDto: LoginDto
  ): Promise<{ access_token: string }> {
    console.log("AuthController, method 'login', loginDto:", loginDto);
    return this.authService.logUserIn(loginDto);
  }

  @Post("register")
  public async register(@Body() createUserDto: CreateUserDto): Promise<{
    access_token: string;
    user: ResponseUserDto;
  }> {
    return this.authService.register(createUserDto);
  }
}
```

</details>

<br/>

Then add the controller and providers to `auth.module.ts`.

```ts
import { Module } from "@nestjs/common";
import { AuthController } from "./auth.controller"; // <--
import { AuthService } from "./auth.service"; // <--
import { UsersModule } from "src/users/users.module";
import { LocalStrategy } from "src/strategies/local.strategy"; // <--
import { JwtModule, JwtModuleOptions } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { JwtStrategy } from "src/strategies/jwt.strategy"; // <--
import { ConfigModule, ConfigService } from "@nestjs/config";

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (): JwtModuleOptions => {
        return {
          secret: process.env.JWT_SECRET,
          signOptions: {
            expiresIn: "60m",
          },
        };
      },
    }),
    ConfigModule,
  ],
  controllers: [AuthController], // <--
  providers: [AuthService, LocalStrategy, JwtStrategy], // <--
})
export class AuthModule {}
```
