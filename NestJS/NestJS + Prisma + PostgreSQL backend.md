<br/><br/>

<script src="https://gist.github.com/foxcaulfield/00bc40954e0fb3c68102c4373fddfebd.js"></script>
<!-- === === === === === -->
<!-- === === === === === -->
<!-- === === === === === -->

# 4. Install and Configure Dependencies

### dotenv [(docs)](https://www.npmjs.com/package/dotenv)

<details>
<summary>Settings</summary>
<dl><dd>

Install:

```sh
npm install dotenv --save
```
```sh
npm install @nestjs/config
```

Create and configure a `.env` file:

```sh
echo "" > .env
```

</dd></dl>
</details>

### Docker + PostgreSQL

<details>
<summary>Settings</summary>
<dl><dd>

Update the `.env` file

`.env`

```env
DATABASE_HOST=localhost
DATABASE_PORT=5432

POSTGRES_USER=postgresuser
POSTGRES_PASSWORD=postgrespassword
POSTGRES_DB=customdb

DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${DATABASE_HOST}:${DATABASE_PORT}/${POSTGRES_DB}?schema=public
# DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

Create a `docker-compose.yaml` file:

```sh
echo "" > docker-compose.yaml
```

Add the following configuration:

```yaml
services:
  db:
    image: postgres:17
    env_file: .env
    ports:
      - 5432:5432
    volumes:
      - db_data:/var/lib/postgresql/data
  # ...
volumes:
  db_data:
    driver: local
```

Then start the container:

```sh
docker compose up -d
```

Or start only the `db` service:

```sh
docker compose up db -d
```

</dd></dl>
</details>

### Prisma

<details>
<summary>Settings</summary>
<dl><dd>

_You can check the official NestJS docs [here](https://docs.nestjs.com/recipes/prisma#use-prisma-client-in-your-nestjs-services) and the Prisma docs [here](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql)._

```sh
npm install prisma --save-dev
```

Now create your initial Prisma setup using the `init` command of the Prisma CLI:

```sh
npx prisma init
```

This command creates a new prisma directory with the following contents:

- `schema.prisma`: Specifies your database connection and contains the database schema
- `.env`: A dotenv file (if not already present), typically used to store your database credentials in a group of environment variables

⚠️ Ensure the `output` field is commented out in the `schema.prisma` file. Otherwise, account for it in later configuration steps.

<details>
  <summary><strong>prisma/schema.prisma</strong></summary>

```prisma
generator client {
    provider = "prisma-client-js"
    //   output   = "../generated/prisma"
}

datasource db {
    provider = "postgresql"
    url      = env("DATABASE_URL")
}
```

</details>

<br/>

Then run the following command (this will also install `@prisma/client` automatically):

```sh
npx prisma generate
```

</dd></dl>
</details>

### Better Auth

<details>
<summary>Settings</summary>
<dl><dd>

[(Better Auth - General Installation)](https://www.better-auth.com/docs/installation)
[(Better Auth - NestJS Integration)](https://www.better-auth.com/docs/integrations/nestjs)
[(Better Auth - Official Repo)](https://github.com/better-auth/better-auth)
[(NestJS Better Auth Integration - Repo)](https://github.com/thallesp/nestjs-better-auth)

#### Better Auth Instance Configuration

```sh
npm install better-auth @thallesp/nestjs-better-auth
```

Create a `.env` file (if not already created).

Add the following environment variables:

```env
# ...rest of variables
BETTER_AUTH_SECRET=your_secret_string
BETTER_AUTH_URL=http://localhost:5000 # Base URL of your NestJS backend
```

Create a file named `auth.ts` in the `src` directory:

```sh
echo "" > src/auth.ts
```

In this file, import Better Auth and create your auth instance (include the `prismaAdapter`).

<details>
<summary><strong>File: src/auth.ts</strong></summary>

```ts
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";
// import { PrismaClient } from "generated/prisma";  /* Alt path, depends on your 'client' -> 'output' value in your schema.prisma file */

const prismaClient = new PrismaClient();
export const auth = betterAuth({
  secret: process.env.BETTER_AUTH_SECRET,

  database: prismaAdapter(prismaClient, {
    provider: "postgresql",
  }),

  emailAndPassword: {
    enabled: true,
  },

  user: {
    additionalFields: {
      role: {
        fieldName: "role",
        type: "string",
      },
    },
  },

  // Your frontend origin
  // trustedOrigins: ["http://localhost:5173"],
});
```

</details>

<br/>

Run the command and accept all prompts.
This overwrites your `schema.prisma` (models), but if following this guide from the beginning, no models should exist yet.

```sh
npx @better-auth/cli generate
```

Modify the `schema.prisma` file with the following changes:

<details>
<summary><strong>prisma/schema.prisma</strong></summary>

```prisma
// ..,
enum Role { // <- Define user roles
    USER
    MODERATOR
    ADMIN
}

model User {
    id            String    @id @default(uuid()) // <- Add this to automatically create a unique default ID
    name          String
    email         String
    emailVerified Boolean   @default(false)
    image         String?
    createdAt     DateTime  @default(now())
    updatedAt     DateTime  @default(now()) @updatedAt
    role          Role      @default(USER) // <- Sets the default role to USER
    sessions      Session[]
    accounts      Account[]

    @@unique([email])
    @@map("user")
}
// ...
```

</details>

In addition to the models generated above, add the following one manually.
This model is provided **for demonstration purposes only** and will later be used to demonstrate how to apply decorators.

<details>
<summary><strong>prisma/schema.prisma</strong></summary>

```prisma
// ...
model Note {
    id         String   @id @default(uuid())
    title      String
    content    String?
    source     String?
    priority   Int      @default(0)
    isArchived Boolean?
    createdAt  DateTime @default(now())

    @@map("notes")
}
```

</details>

<br/>

Then run the migration command:

```sh
npx prisma migrate dev --name init
```

This will:

- Create a new migration in prisma/migrations named init.
- Apply the migration to your database.
- Generate Prisma Client in node_modules/.prisma/client.

</dd></dl>
</details>

<br/><br/>

<!-- === === === === === -->
<!-- === === === === === -->
<!-- === === === === === -->

# 5. Set Up the Project

## Better Auth

<details><summary>Settings</summary>

<dl><dd>

### **Disable Body Parser**

Disable NestJS's built-in body parser to allow Better Auth to handle the raw request body:

<details>
  <summary>src/main.ts</summary>

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap(): Promise<void> {
  const app = await NestFactory.create(AppModule, {
    bodyParser: false, // <- Required for Better Auth
  });
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap().catch((e): void => console.error(e));
```

</details>

### **Import AuthModule from Better Auth**

Import the AuthModule in your root module:

<details>
  <summary><strong>src/app.module.ts</strong></summary>

```ts
import { Module } from "@nestjs/common";
import { AuthModule } from "@thallesp/nestjs-better-auth";
import { auth } from "./auth"; // Your Better Auth instance

@Module({
  imports: [AuthModule.forRoot(auth)],
})
export class AppModule {}
```

</dd></dl>

</details>
</details>

## Swagger (optional)

<details><summary>Settings</summary>
<dl><dd>

<!-- ```sh
npm install @nestjs/swagger
``` -->

```sh
npm install @nestjs/swagger class-validator class-transformer
```

Once the installation process is complete, open the `main.ts` file and initialize Swagger using the `SwaggerModule` class [(docs)](https://docs.nestjs.com/openapi/introduction#bootstrap).

<details>
<summary><strong>src/main.ts</strong></summary>

```ts
import { DocumentBuilder, OpenAPIObject, SwaggerModule } from "@nestjs/swagger";
import { writeFileSync } from "node:fs";
import { join } from "node:path";

// ...
async function bootstrap(): Promise<void> {
  // ...
  const config = new DocumentBuilder()
    .setTitle("Application Title")
    .setDescription("The Application API Description")
    .setVersion("1.0")
    .addTag("app")
    .build();
  const documentFactory = (): OpenAPIObject =>
    SwaggerModule.createDocument(app, config);

  // Setup SwaggerUI
  SwaggerModule.setup("api", app, documentFactory);

  // Save OpenAPI File
  const outputPath = join(process.cwd(), "swagger-spec.json");
  writeFileSync(outputPath, JSON.stringify(documentFactory(), null, 2));
  // ...
}
```

</details>

<br/>

**Adjust `Swagger`**

> _A quick note:_
> Usually, in order to make the class properties visible to the `SwaggerModule`, you need to annotate them with the `@ApiProperty()` decorator [(docs)](https://docs.nestjs.com/openapi/types-and-parameters#types-and-parameters).

Alternatively, you can use the appropriate plugin, as described below. [(Swagger/CLI plugin)](https://docs.nestjs.com/openapi/cli-plugin) [(docs1)](https://www.prisma.io/blog/nestjs-prisma-relational-data-7D056s1kOabc#define-the-user-entity-and-dto-classes) [(docs2)](https://medium.com/@daiki01240/how-to-leverage-swagger-and-class-validator-in-nestjs-api-documentation-and-exporting-type-7577da98768d).

<!-- Before proceeding, make sure these packages are installed. -->

<!-- {
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/swagger"]
  }
} -->

To [enable]() the plugin, open `nest-cli.json` and add the following plugins configuration. You can also use the options property to customize the behavior of the plugin.

<details><summary><strong>nest-cli.json</strong></summary>

```JavaScript
{
// ...
  "compilerOptions": {
	// ...
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "classValidatorShim": true,
          "introspectComments": true,
          "skipAutoHttpCode": true
        }
      }
    ]
	// ...
  }
}
```

</details>

<br/>

The NestJS OpenAPI (Swagger) CLI plugin will automatically:

- Annotate DTO properties with `@ApiProperty` and set `required`, `type`, and `default`.
- Apply validation rules from `class-validator` (if `classValidatorShim` is enabled).
- Add response decorators to endpoints with proper status and types.
- Generate descriptions and examples from comments (if `introspectComments` is enabled).
- Generate and update Swagger (OpenAPI) documentation for your project.

</dd></dl>
</details>

## Prisma

<details>
<summary>Settings</summary>
<dl><dd>

### **Create and Set Up a Prisma Service**

[(docs 1)](https://docs.nestjs.com/recipes/prisma#use-prisma-client-in-your-nestjs-services)
[(docs 2)](https://www.prisma.io/nestjs)

Run the following command to generate a NestJS service (_for further working with Prisma_):

```sh
nest generate service prisma
```

This creates a service in the `src/prisma` directory.

Update the generated `prisma.service.ts` file to extend `PrismaClient` and implement lifecycle hooks for database connection management.

<details>
<summary><strong>src/prisma/prisma.service.ts</strong></summary>

```ts
import { Injectable, OnModuleDestroy, OnModuleInit } from "@nestjs/common";
import { PrismaClient } from "@prisma/client";

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  public async onModuleInit(): Promise<void> {
    await this.$connect();
  }

  public async onModuleDestroy(): Promise<void> {
    await this.$disconnect();
  }
}
```

</details>

</dd></dl>
</details>

<!-- ### **Create and Set Up 'Users' DTOs** (deprecated)

Before proceeding, you need to install these packages. They are required to generate DTOs.
[(docs1)](https://github.com/Brakebein/prisma-generator-nestjs-dto)
[(docs2)](https://www.prisma.io/blog/nestjs-prisma-relational-data-7D056s1kOabc#define-the-user-entity-and-dto-classes)
[(docs3)](https://medium.com/@daiki01240/how-to-leverage-swagger-and-class-validator-in-nestjs-api-documentation-and-exporting-type-7577da98768d)

```sh
npm install --save-dev @brakebein/prisma-generator-nestjs-dto
```

```sh
npm install @nestjs/swagger class-validator class-transformer
```

Update your `schema.prisma` to include the DTO generator. Add following to `prisma/schema.prisma` file:

<details>
  <summary><strong>prisma/schema.prisma</strong></summary>

```prisma
// ...
generator nestjsDto {
    provider                        = "prisma-generator-nestjs-dto"
    output                          = "../src/generated-dto"
    prismaClientImportPath          = ""
    outputToNestJsResourceStructure = "true" // <- organizes generated files into a resource-like folder structure (not directly inside your project structure)
    flatResourceStructure           = "false"
    exportRelationModifierClasses   = "true"
    reExport                        = "false"
    generateFileTypes               = "all"
    createDtoPrefix                 = "Create"
    updateDtoPrefix                 = "Update"
    dtoSuffix                       = "Dto"
    entityPrefix                    = ""
    entitySuffix                    = "Entity" // <- adds mark to entity names for easier identification
    classValidation                 = "true" // <- adds class-validator decorators to generated classes
    fileNamingStyle                 = "camel"
    noDependencies                  = "false"
    outputType                      = "class"
    definiteAssignmentAssertion     = "true" // <- adds ! to required fields to satisfy strict property checks
    requiredResponseApiProperty     = "true"
    prettier                        = "false"
    wrapRelationsAsType             = "false"
    showDefaultValues               = "false"
}

// ...
```

</details>

<br/>

And run the command to generate DTOs:

```sh
npx prisma generate
```

Adjust the files (DTOs, entities) to fit your project’s style. Then run `npm run format` and `npm run lint`, and fix any warnings or errors that may appear.”

**dtos were here**

</details> -->

</dd></dd>
</details>

<br/><br/>

# 6. Create and Set Up the Feature

    🏁 From now on, adding a new feature will involve pretty much the same set of steps. 🏁

<!-- ### Add Prisma Model

### Perform Prisma Migration

<br/> -->

### Generate a Feature (Module/Service/Controller/DTOs)

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

You can generate the feature template with a single command and make a few adjustments:

```sh
nest generate resource notes
```

```sh
rm -rf ./src/notes/entities
```

```sh
echo "" > ./src/notes/dto/response-note.dto.ts
```

Or you can create the files manually using separate commands.

```sh
nest generate module notes
```

```sh
nest generate service notes
```

```sh
nest generate controller notes
```

```sh
nest generate class notes/dto/create-note.dto --flat
```

```sh
nest generate class notes/dto/update-note.dto --flat
```

```sh
nest generate class notes/dto/response-note.dto --flat
```

Note: Don’t add any content yet; just ensure the files are created.

- `src/notes/notes.service.ts` file
- `src/notes/notes.controller.ts` file
- `src/notes/notes.module.ts` file
- `src/notes/dto/create-note.dto.ts` file
- `src/notes/dto/update-note.dto.ts` file
- `src/notes/dto/response-note.dto.ts` file

</dd></dl>
</details>

<br/>

<!-- ### Adjust the files (DTOs) to fit your project’s style. -->

### Register the Prisma Service in the `providers` Array of the Feature Module

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

<details>
<summary><strong>src/notes/notes.module.ts</strong></summary>

```ts
import { Module } from "@nestjs/common";
import { NotesService } from "./notes.service";
import { NotesController } from "./notes.controller";
import { PrismaService } from "src/prisma/prisma.service";

@Module({
  controllers: [NotesController],
  providers: [NotesService, PrismaService], // <-
})
export class NotesModule {}
```

</details>

</dd></dl>
</details>

<br/>

### Inject the Prisma Service into the Feature Service

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

<details>
<summary><strong>src/notes/notes.service.ts</strong></summary>

```ts
import { PrismaService } from "src/prisma/prisma.service";

@Injectable()
export class NotesService {
  public constructor(private readonly prisma: PrismaService) {} // <- Prisma is injected here
  // ... the rest of the file
}
```

</details>

[(NestJS class-validator decorators)](https://github.com/typestack/class-validator#validation-decorators)

</dd></dl>
</details>

<br/>

### Add Decorators to DTOs

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

Add any decorators you need, then run `npm run format` and `npm run lint`, and fix any warnings or errors that may appear.”

The result should include the following files:

<details><summary><strong>src/notes/dto/create-note.dto.ts</strong></summary>

```ts
// import { ApiProperty } from "@nestjs/swagger";
import {
  IsNotEmpty,
  IsNumber,
  IsOptional,
  IsString,
  Length,
  Max,
  MaxLength,
  Min,
  MinLength,
} from "class-validator";

export class CreateNoteDto {
  // @ApiProperty()
  @IsNotEmpty()
  @IsString()
  @MinLength(3)
  @MaxLength(20)
  public title!: string;

  // @ApiProperty()
  @IsOptional()
  @IsString()
  @Length(5, 50)
  public content?: string;

  // @ApiProperty()
  @IsOptional()
  @IsString()
  @Length(1, 30)
  public source?: string;

  // @ApiProperty()
  @IsOptional()
  @IsNumber()
  // @IsPositive()
  @Min(0)
  @Max(5)
  public priority?: number;

  public constructor(data: CreateNoteDto) {
    if (data?.title != null) this.title = data?.title;
    if (data?.content != null) this.content = data?.content;
    if (data?.source != null) this.source = data?.source;
    if (data?.priority != null) this.priority = data?.priority;
  }
}
```

</details>

<br/>

<details>
	<summary><strong>src/notes/dto/update-note.dto.ts</strong></summary>

```ts
import {
  IsBoolean,
  IsNotEmpty,
  IsNumber,
  IsOptional,
  IsString,
  Length,
  Max,
  MaxLength,
  Min,
  MinLength,
} from "class-validator";

export class UpdateNoteDto {
  @IsOptional()
  @IsNotEmpty()
  @IsString()
  @MinLength(3)
  @MaxLength(20)
  public title?: string;

  @IsOptional()
  @IsString()
  @Length(5, 50)
  public content?: string;

  @IsOptional()
  @IsString()
  @Length(1, 30)
  public source?: string;

  @IsOptional()
  @IsNumber()
  // @IsPositive()
  @Min(0)
  @Max(5)
  public priority?: number;

  @IsOptional()
  @IsBoolean()
  public isArchived?: boolean;

  public constructor(data: UpdateNoteDto) {
    if (data?.title != null) this.title = data?.title;
    if (data?.content != null) this.content = data?.content;
    if (data?.source != null) this.source = data?.source;
    if (data?.priority != null) this.priority = data?.priority;
    if (data?.isArchived != null) this.isArchived = data?.isArchived;
  }
}
```

</details>

<br/>

<details><summary><strong>src/notes/dto/response-note.dto.ts</strong></summary>

```ts
export class ResponseNoteDto {
  public id!: string;
  public title!: string;
  public content?: string | null;

  public constructor(data: ResponseNoteDto) {
    if (data?.id != null) this.id = data?.id;
    if (data?.title != null) this.title = data?.title;
    if (data?.content != null) this.content = data?.content;
  }
}
```

</details>

<br/>

<!-- ```ts
// src/notes/dto/login-note.dto.ts
import { ApiProperty } from "@nestjs/swagger";
import { IsEmail, IsString, MinLength } from "class-validator";

export class LoginUserDto {
	@ApiProperty({
		description: "User email address",
		example: "user@example.com",
	})
	@IsEmail()
	public email: string;

	@ApiProperty({
		description: "User password",
		example: "strongPassword123",
		minLength: 6,
	})
	@IsString()
	@MinLength(6)
	public password: string;

	public constructor(data: LoginUserDto) {
		this.email = data.email;
		this.password = data.password;
	}
}

``` -->

</dd></dl>
</details>

<br/>

### Add `ValidationPipe` (+Transformation)

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

<!-- **Update the Controller and Service, Add Pipes** -->

#### **Validation and Transformation**

 <!-- (and others `сlass-validator` pipes)  -->

Validation and transformation can be enabled in the following ways:

<details>
<summary>Globally: In `src/main.ts`, apply the `ValidationPipe` to the entire application</summary>

```TypeScript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common"; // <- here

async function bootstrap(): Promise<void> {
  const app = await NestFactory.create(AppModule, {
    bodyParser: false,
  });

  /* here */
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      transform: true,
      forbidNonWhitelisted: true,
      forbidUnknownValues: true,
      // disableErrorMessages: true,
    })
  );

  await app.listen(process.env.PORT ?? 3000);
}

bootstrap().catch((e): void => console.error(e));
```

</details>

<details>
<summary>At the controller level: apply `ValidationPipe` using the `@UsePipes` decorator.</summary>

Use the `@UsePipes` to apply the `ValidationPipe` to an entire controller

```TypeScript
@UsePipes(
	new ValidationPipe({
		whitelist: true,
		transform: true,
		forbidNonWhitelisted: true,
		forbidUnknownValues: true,
		// disableErrorMessages: true,
	})
)
@Controller("notes")
export class NotesController {}
```

</details>

<details>
<summary>At the method level: apply `ValidationPipe` using the `@UsePipes` decorator.</summary>

```TypeScript
@Post()
@UsePipes(
	new ValidationPipe({
		whitelist: true,
		transform: true,
		forbidNonWhitelisted: true,
		forbidUnknownValues: true,
		// disableErrorMessages: true,
	})
)
async create(/* ... */) {}
```

</details>

<details>
<summary>At the parameter level: use `ParseIntPipe` or `ParseBoolPipe` to cast values explicitly</summary>

<br/>
You can explicitly cast values using the `ParseIntPipe` or `ParseBoolPipe` inside `@Param()` or `@Query()` decorators. A separate pipe is used for each, so `ValidationPipe` is not needed for them to work:

```ts

@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,
  @Query('sort', ParseBoolPipe) sort: boolean,
) {
  console.log(typeof id === 'number'); // true
  console.log(typeof sort === 'boolean'); // true
  return 'This action returns a user';
}
```

</details>

<!-- </details> -->

<!-- <details>
<summary><strong>**How to use `class-validator` validation pipes**</strong></summary> -->

<br/>

> **How do pipes work?**
> If a ValidationPipe is enabled (globally, at the controller, or method level), it automatically validates the DTO passed as an argument.

```TypeScript
create(@Body() createUserDto: CreateUserDto) {}
```

<br/>

> With the auto-transformation option enabled, the ValidationPipe will also perform primitive type conversion. For example, the findOne() method below takes an id path parameter and automatically converts its type to a number.

```TypeScript
@Get(':id')
findOne(@Param('id') id: number) {
  console.log(typeof id === 'number'); // true
  return 'This action returns a user';
}
```

<br/>

📝 If you need to validate arrays in NestJS, refer to [the official documentation](https://docs.nestjs.com/techniques/validation#parsing-and-validating-arrays).

<!-- Now implement the desired logic using the full power of `class-validator`, `@nestjs/swagger`, `better-auth`, and more. -->

**Docs:**

- [(NestJS validation pipes)](https://docs.nestjs.com/techniques/validation)
- [(Better Auth decorators)](https://github.com/ThallesP/nestjs-better-auth)
- [(OpenAPI/Swagger general decorators)](https://docs.nestjs.com/openapi/decorators)
- [(OpenAPI/Swagger response decorators)](https://docs.nestjs.com/openapi/operations#responses)
- [(docs)](https://docs.nestjs.com/techniques/validation#validation)
- [(more docs)](https://docs.nestjs.com/pipes#class-validator)
</details>

<br/>

### Add `Better Auth` Authentication Guard

<!-- To use Better Auth validation pipes:

You need to import the module. However, simply importing it is not enough to enable it, so keep reading for the next step.

```TypeScript
@Module({
  imports: [
    AuthModule.forRoot(auth),
  ],
})
export class AppModule {}
``` -->

<details>
<summary>Info</summary>
<dl><dd>

You can protect specific parts of your application with the `AuthGuard` from Better Auth.

<!-- Now you can choose on what level to protect your app: -->

<details>
<summary>Global Level</summary>
<dl><dd>

For application-wide protection, register the guard as a global provider.

```TypeScript
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { AuthModule, AuthGuard } from '@thallesp/nestjs-better-auth';
import { auth } from "./auth";

@Module({
  imports: [
    AuthModule.forRoot(auth),
  ],
  providers: [
    {
      provide: APP_GUARD, // <-
      useClass: AuthGuard, // <-
    },
  ],
})
export class AppModule {}
```

</dd></dl>
</details>

<details>
<summary>Controller Level</summary>
<dl><dd>

Use `@UseGuards(AuthGuard)` to protect all routes within a controller.

```TypeScript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { AuthGuard } from '@thallesp/nestjs-better-auth';

@Controller('users')
@UseGuards(AuthGuard) // <- Apply to all routes in this controller
export class UserController {}
```

</dd></dl>
</details>

<details>
<summary>Method Level</summary>
<dl><dd>

Use `@UseGuards(AuthGuard)` to protect a single, specific route.

```TypeScript
@Controller("notes")
export class NotesController {
	public constructor(private readonly notesService: NotesService) {}

	@UseGuards(AuthGuard) // <-
	@Get("by_id/:id")
	public async findOne(@Param("id", ParseUUIDPipe) id: string): Promise<ResponseNoteDto | null> {
		return this.notesService.findById(id);
	}
}
```

</dd></dl>
</details>

<br/>

<details>
<summary>Other Available Decorators</summary>
<dl><dd>

Here are some of the available decorators:

- **`@Session()`** (parameter level)
- **`@Public()`** (controller level / method level)
- **`@Optional()`** (controller level / method level)
- ...and others, such as `@Hook()`, `@Request()`, etc.

The main difference between `@Public()` and `@Optional()` is that `@Optional()` allows access to the session, which may be empty or null, while `@Public()` does not.

</dd></dl>
</details>

<details><summary>AuthService</summary>
<dl><dd>

The `AuthService` is automatically provided by the `AuthModule` and can be injected into your controllers to access the Better Auth instance and its API endpoints.

```TypeScript

@Controller('users')
@UseGuards(AuthGuard)
export class UsersController {
  constructor(private authService: AuthService<typeof auth>) {}

  @Get('accounts')
  async getAccounts(@Request() req: ExpressRequest) {
    // Pass the request headers to the auth API
    const accounts = await this.authService.api.listUserAccounts({
      headers: fromNodeHeaders(req.headers),
    });

    return { accounts };
  }
}
```

</dd></dl>
</details>
</dd></dl>
</details>

<br/>

### Create and Add Role Access Guard

<details>
<summary>Info</summary>
<dl><dd>

[(docs)](https://docs.nestjs.com/security/authorization)

1. Create the decorator

```sh
nest generate decorator decorators/roles --flat
```

<details>
<summary>src/decorators/roles.decorator.ts</summary>
<dl><dd>

```ts
import { CustomDecorator, SetMetadata } from "@nestjs/common";

export const ROLES_KEY = "roles";
export const Roles = (...args: string[]): CustomDecorator =>
  SetMetadata(ROLES_KEY, args);
```

</dd></dl>
</details>

<br/>

2. Create the guard

```sh
nest generate guard guards/roles --flat
```

<details>
<summary>src/guards/roles.guard.ts</summary>
<dl><dd>

```ts
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  ForbiddenException,
} from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { ROLES_KEY } from "../decorators/roles.decorator";
import { UserSession } from "@thallesp/nestjs-better-auth";
import { Role } from "@prisma/client";
import { IncomingMessage } from "node:http";
// import { Request } from "express";

interface AuthenticatedRequest extends UserSession, IncomingMessage {
  user: UserSession["user"] & {
    role?: Role;
  };
}

@Injectable()
export class RolesGuard implements CanActivate {
  public constructor(private reflector: Reflector) {}

  public canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }

    const request = context.switchToHttp().getRequest<AuthenticatedRequest>();
    const user = request.user;

    if (!user) {
      throw new ForbiddenException("User is not authenticated");
    }

    const hasRole = user?.role && requiredRoles.includes(user?.role);
    if (!hasRole) {
      throw new ForbiddenException("Insufficient permissions");
    }

    return true;
  }
}
```

</dd></dl>
</details>

<br/>

3. Apply the guard:

<details>
<summary>Globally</summary>
<dl><dd>

```ts
// app.module.ts
import { Module } from "@nestjs/common";
import { APP_GUARD } from "@nestjs/core";
import { RolesGuard } from "./guards/roles.guard";

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

</dd></dl>
</details>

<details>
<summary>On the controller level</summary>
<dl><dd>

```ts
import { Controller, UseGuards } from "@nestjs/common";
import { RolesGuard } from "../guards/roles.guard";
import { Roles } from "../decorators/roles.decorator";

@UseGuards(RolesGuard)
@Controller("users")
export class UsersController {
  // ...
}
```

</dd></dl>
</details>

<details>
<summary>On the method level</summary>
<dl><dd>

```ts
import { Controller, UseGuards } from "@nestjs/common";
import { RolesGuard } from "../guards/roles.guard";
import { Roles } from "../decorators/roles.decorator";

@Controller("users")
export class UsersController {
  // ...
  @UseGuards(RolesGuard)
  @Post("create")
  public async create(/* ... */) {}
  // ...
}
```

</dd></dl>
</details>

<br/>

4. Restrict access using the `@Roles` decorator in the controller or in methods

<details>
<summary>Example</summary>
<dl><dd>

```ts
import { Controller, Get } from "@nestjs/common";
import { Roles } from "../decorators/roles.decorator";
import { Session } from "@thallesp/nestjs-better-auth";

@Roles(Role.ADMIN)
@Controller("admin")
export class AdminController {
  @Roles(Role.ADMIN) // Accessible only to administrators
  @Get("dashboard")
  getDashboard(@Session() session: UserSession) {
    return { message: "Welcome to the admin dashboard!", user: session.user };
  }
}
```

</dd></dl>
</details>

<br/>

📝 You can combine it with other guards (such as `AuthGuard`) and specify multiple roles:

<details>
<summary>Multiple guards and roles</summary>
<dl><dd>

```ts
import { Controller, UseGuards } from "@nestjs/common";
import { RolesGuard } from "../guards/roles.guard";
import { Roles } from "../decorators/roles.decorator";

@Controller("users")
export class UsersController {
  // ...
  @UsePipes(new ValidationPipe())
  @UseGuards(AuthGuard, RolesGuard)
  @Roles(Role.USER, Role.ADMIN)
  @Post("create")
  public async create(/* ... */) {}
  // ...
}
```

</dd></dl>
</details>

</dd></dl>
</details>

<br/><br/>

# Example resulting files are listed below:

<details>
<summary><strong>src/notes/notes.module.ts</strong></summary>

```ts
import { Module } from "@nestjs/common";
import { NotesService } from "./notes.service";
import { NotesController } from "./notes.controller";
import { PrismaService } from "src/prisma/prisma.service";

@Module({
  controllers: [NotesController],
  providers: [NotesService, PrismaService],
})
export class NotesModule {}
```

</details>

<details>
<summary><strong>src/notes/notes.service.ts</strong></summary>

```ts
import { Injectable } from "@nestjs/common";
import { CreateNoteDto } from "./dto/create-note.dto";
import { UpdateNoteDto } from "./dto/update-note.dto";
import { PrismaService } from "src/prisma/prisma.service";
import { Note } from "@prisma/client";
import { ResponseNoteDto } from "./dto/response-note.dto";

type NoteId = Note["id"];

type FlagsOf<T> = { [K in keyof T]: boolean };

const safeNoteResponseProps = {
  id: true,
  title: true,
  content: true,
} satisfies FlagsOf<ResponseNoteDto>;

@Injectable()
export class NotesService {
  public constructor(private readonly prisma: PrismaService) {} // <- Prisma is injected here
  // ... the rest of the file
  public async create(createNoteDto: CreateNoteDto): Promise<ResponseNoteDto> {
    const result = await this.prisma.note.create({
      data: createNoteDto,
      select: safeNoteResponseProps,
    });
    return result;
    // You could also do, but it's redundant
    // return new ResponseNoteDto(result);
  }

  public async findById(id: NoteId): Promise<ResponseNoteDto | null> {
    const result = await this.prisma.note.findUnique({
      where: {
        id,
      },
      select: safeNoteResponseProps,
    });

    return result;
  }

  public async update(
    id: NoteId,
    updateNoteDto: UpdateNoteDto
  ): Promise<ResponseNoteDto | null> {
    const result = await this.prisma.note.update({
      where: {
        id,
      },
      data: updateNoteDto,
      select: safeNoteResponseProps,
    });

    return result;
  }

  public async remove(id: NoteId): Promise<NoteId | null> {
    const result = await this.prisma.note.delete({
      where: {
        id,
      },
      select: {
        id: true,
      },
    });

    return result.id;
  }

  public async findAll(): Promise<Array<ResponseNoteDto>> {
    return this.prisma.note.findMany({ select: safeNoteResponseProps });
  }
}
```

</details>

<details>
<summary><strong>src/notes/notes.controller.ts</strong></summary>

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  ParseUUIDPipe,
  UsePipes,
  ValidationPipe,
  UseGuards,
  Request,
} from "@nestjs/common";
import { NotesService } from "./notes.service";
import { CreateNoteDto } from "./dto/create-note.dto";
import { UpdateNoteDto } from "./dto/update-note.dto";
import { ResponseNoteDto } from "./dto/response-note.dto";
import { Note, Role } from "@prisma/client";
import {
  AuthGuard,
  AuthService,
  Optional,
  Public,
  Session,
  type UserSession,
} from "@thallesp/nestjs-better-auth";
import { auth } from "src/auth";
import { fromNodeHeaders } from "better-auth/node";
import type { Request as ExpressRequest } from "express";
import { RolesGuard } from "src/guards/roles.guard";
import { Roles } from "src/decorators/roles.decorator";

@UsePipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true, // <- here
    forbidUnknownValues: true,
    // disableErrorMessages: true,
  })
)
@UseGuards(AuthGuard)
@Controller("notes")
export class NotesController {
  public constructor(
    private readonly notesService: NotesService,
    private authService: AuthService<typeof auth>
  ) {}

  @UsePipes(new ValidationPipe())
  @UseGuards(AuthGuard, RolesGuard)
  @Roles(Role.USER, Role.ADMIN)
  @Post("create")
  public async create(
    @Body() createNoteDto: CreateNoteDto,
    @Session() session: UserSession
  ): Promise<ResponseNoteDto> {
    console.log("User id " + session.user.id);
    return this.notesService.create(createNoteDto);
  }

  @Get("by_id/:id")
  public async findOne(
    @Param("id", ParseUUIDPipe) id: string
  ): Promise<ResponseNoteDto | null> {
    return this.notesService.findById(id);
  }

  @Patch("update/:id")
  public async update(
    @Param("id", ParseUUIDPipe) id: string,
    @Body() updateNoteDto: UpdateNoteDto
  ): Promise<ResponseNoteDto | null> {
    return this.notesService.update(id, updateNoteDto);
  }

  @Delete("delete/:id")
  public async remove(
    @Param("id", ParseUUIDPipe) id: string
  ): Promise<Note["id"] | null> {
    return this.notesService.remove(id);
  }

  @Public()
  @Get("all")
  public async findAll(
    @Session() session: UserSession
  ): Promise<Array<ResponseNoteDto>> {
    console.log(session);
    return this.notesService.findAll();
  }

  // Common examples
  @Optional() // <- Just example
  @Get("news")
  public getNews(@Session() session: UserSession): {
    message: string;
    user?: string;
  } {
    if (session) {
      return {
        message: "Welcome!",
        user: session.user.email,
      };
    }

    return {
      message: "No news",
    };
  }

  @Get("accounts")
  public async getAccounts(@Request() req: ExpressRequest): Promise<{
    accounts: {
      id: string;
      providerId: string;
      createdAt: Date;
      updatedAt: Date;
      accountId: string;
      scopes: string[];
    }[];
  }> {
    const accounts = await this.authService.api.listUserAccounts({
      headers: fromNodeHeaders(req.headers), // This is required to be authenticated
    });

    return { accounts };
  }
}
```

</details>

> You can optionally add a response validator (like [here](https://medium.com/@kuba.2001/reponse-validation-in-nestjs-0db70b955a6a) and [here](https://www.reddit.com/r/nestjs/comments/1knaeze/comment/mssircd/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)), but it's typically unnecessary if you explicitly select the fields to return from your ORM in the service layer.

<br/><br/>

<details><summary><strong>content of `src/main.ts`</strong></summary>

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { DocumentBuilder, OpenAPIObject, SwaggerModule } from "@nestjs/swagger";
import { join } from "node:path";
import { writeFileSync } from "node:fs";
import { ValidationPipe } from "@nestjs/common";

async function bootstrap(): Promise<void> {
  const app = await NestFactory.create(AppModule, {
    bodyParser: false,
  });

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true, // <- here
      forbidUnknownValues: true,
      // disableErrorMessages: true,
    })
  );

  const config = new DocumentBuilder()
    .setTitle("Application Title")
    .setDescription("The Application API Description")
    .setVersion("1.0")
    .addTag("app")
    .build();
  const documentFactory = (): OpenAPIObject =>
    SwaggerModule.createDocument(app, config);

  // Setup SwaggerUI
  SwaggerModule.setup("api", app, documentFactory);

  // Save OpenAPI File
  const outputPath = join(process.cwd(), "swagger-spec.json");
  writeFileSync(outputPath, JSON.stringify(documentFactory(), null, 2));

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap().catch((e): void => console.error(e));
```

<!-- </details> -->
<!-- ### **Create Auth service** -->

<!-- <details> -->
  <!-- <summary><strong>src/auth/auth.service.ts</strong></summary> -->
<!-- </details> -->
<!-- <details> -->
  <!-- <summary><strong>src/auth/auth.controller.ts</strong></summary> -->
<!-- </details> -->

<br/><br/>

<!-- === === === === === -->
<!-- === === === === === -->
<!-- === === === === === -->

# FAQ

[(better-auth endpoints)](https://www.better-auth.com/docs/plugins/username#usage)

[(better-auth email auth)](https://www.better-auth.com/docs/authentication/email-pa)
Flow:

- Add model
- Generate prisma types
- Create the resource/module
- Import Prisma service in the module
- Inject Prisma service in the service
