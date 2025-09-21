<br/><br/>

# Prerequisites

- Docker
- Node.js
- nvm (optional)

<br/><br/>

# Install NestJS CLI

### Install

```sh
npm install -g @nestjs/cli
```

### Verify the Installation

```sh
nest --version
```

<br/><br/>

# Set Up the Project

### Navigate to Your Project Directory

#### Option 1: Create the Project in a New Folder

Navigate to the directory where you want to place the project and run.

<!-- _Note: Add the `--no-spec` flag if you don’t want test files to be generated._ -->

```sh
cd path/to/your/directory
```

```sh
nest new my_project
```

<!-- nest new my_project . --no-spec -->

```sh
cd my_project
```

This creates a new folder `my_project` with a ready-to-use NestJS project.

#### Option 2: Create the Project in the Current Folder

Alternatively, if you want to initialize the project in the current folder:

```sh
mkdir my_project
```

```sh
cd my_project
```

```sh
nest new .
```

<!-- nest new . --no-spec -->

⚠️ Ensure the current folder is empty to avoid conflicts with existing files.

Then navigate to your project folder (if not already there).

<br/><br/>

# Add debug config (optional)

<details><summary><strong>.vscode/launch.json</strong><summary>

```JavaScript
{
	// Use IntelliSense to learn about possible attributes.
	// Hover to view descriptions of existing attributes.
	// For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
	"version": "0.2.0",
	"configurations": [
		{
			"type": "node",
			"request": "launch",
			"name": "Debug NestJS",
			"runtimeArgs": ["-r", "ts-node/register", "-r", "tsconfig-paths/register"],
			"args": ["-r", "tsconfig-paths/register", "src/main.ts"],
			"console": "integratedTerminal",
			"sourceMaps": true,
			"env": {
				"PORT": "3000" // Set your desired port here
			}
		},
		{
			"type": "node",
			"request": "launch",
			"name": "Alt. Debug NestJS",
			"runtimeExecutable": "npm",
			"runtimeArgs": ["run", "start:debug", "--", "--inspect-brk"],
			"console": "integratedTerminal",
			"restart": true,
			"autoAttachChildProcesses": true
		}
	]
}
```

</details>

# Install and Configure Dependencies

### dotenv [(docs)](https://www.npmjs.com/package/dotenv)

Install:

```sh
npm install dotenv --save
```

Create and configure a `.env` file:

```sh
echo "" > .env
```

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

<br/><br/>

### Docker + PostgreSQL

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

### Prisma

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
- `.env`: A dotenv file, typically used to store your database credentials in a group of environment variables

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

### Better Auth

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

<br/>

Then run the migration command:

```sh
npx prisma migrate dev --name init
```

This will:

- Create a new migration in prisma/migrations named init.
- Apply the migration to your database.
- Generate Prisma Client in node_modules/.prisma/client.

<br/><br/>

# Configure Project Files (Optional)

### **Customize project configuration files as needed. For example:**

<details>
<summary><strong>tsconfig.json</strong> — TypeScript Compiler Options</summary>

```ts
{
	"compilerOptions": {
		"module": "nodenext",
		"moduleResolution": "nodenext",
		"resolvePackageJsonExports": true,
		"esModuleInterop": true,
		"isolatedModules": true,
		"declaration": true,
		"removeComments": true,
		"emitDecoratorMetadata": true,
		"experimentalDecorators": true,
		"allowSyntheticDefaultImports": true,
		"target": "ES2023",
		"sourceMap": true,
		"outDir": "./dist",
		"baseUrl": "./",
		"incremental": true,
		"skipLibCheck": true,
		"forceConsistentCasingInFileNames": true,

		// Strict Checks
		"alwaysStrict": true,
		"noImplicitAny": true,
		"strictNullChecks": true,
		"strictPropertyInitialization": true,
		"strictFunctionTypes": true,
		"noImplicitThis": true,
		"strictBindCallApply": true,
		// "noPropertyAccessFromIndexSignature": true,
		"noUncheckedIndexedAccess": true,

		// Linter Checks
		"noImplicitReturns": true, // https://eslint.org/docs/rules/consistent-return ?
		"noFallthroughCasesInSwitch": true, // https://eslint.org/docs/rules/no-fallthrough
		"noUnusedLocals": true, // https://eslint.org/docs/rules/no-unused-vars
		"noUnusedParameters": true, // https://eslint.org/docs/rules/no-unused-vars#args
		"allowUnreachableCode": false, // https://eslint.org/docs/rules/no-unreachable ?
		"allowUnusedLabels": false, // https://eslint.org/docs/rules/no-unused-labels

		// Base Strict Checks
		"noImplicitUseStrict": false,
		"suppressExcessPropertyErrors": false,
		"suppressImplicitAnyIndexErrors": false,
		"noStrictGenericChecks": false
	}
}
```

</details>

<details>
  <summary><strong>.prettierrc</strong> — Prettier Formatting Options</summary>

```json
{
  "singleQuote": false,
  "trailingComma": "all",
  "useTabs": true,
  "tabWidth": 4,
  "printWidth": 120
}
```

</details>

<details>
  <summary><strong>.eslint.config.mjs</strong> — ESLint Rules and Formatting</summary>

```ts
//...
rules: {
			// ... existing rules
			"@typescript-eslint/no-explicit-any": "off",
			"@typescript-eslint/no-floating-promises": "warn",
			"@typescript-eslint/no-unsafe-argument": "warn",
			"prettier/prettier": [
				"error",
				{
					// endOfLine: "auto",
					printWidth: 120,
					// trailingComma: "es5",
					// semi: false,
					doubleQuote: true,
					// jsxSingleQuote: true,
					singleQuote: false,
					useTabs: true,
					// tabWidth: 4,
				},
			],
			"@typescript-eslint/explicit-member-accessibility": ["error", { accessibility: "explicit" }],
			"@typescript-eslint/explicit-function-return-type": [
				"error",
				{
					allowExpressions: false,
					allowTypedFunctionExpressions: false,
					allowHigherOrderFunctions: false,
					allowDirectConstAssertionInArrowFunctions: false,
					allowConciseArrowFunctionExpressionsStartingWithVoid: false,
				},
			],
			// ... existing rules
		},
// ...
```

</details>

### **Ensure the Following VS Code Extensions Are Installed**

- **ESLint**
- **Prettier - Code formatter**

### **Remove Spec/Test Files and Scripts (Optional)**

You can remove `.spec.ts` files if you don’t plan to use them.

```sh
find ./src -type f -name "*.spec.ts" -exec rm -i {} \; && rm -rf ./test;
```

#### **Add "no-spec" Setting to `nest-cli.json`**

Edit `nest-cli.json` and add the following to disable test file generation globally:

```js
{
	// ...
	"generateOptions": {
		"spec": false
	}
	// ...
}
```

This way you won’t need to use the --no-spec flag every time you generate a new file.

#### **Remove Test Scripts and Folder**

If tests are not planned, remove all test-related scripts from `package.json` and update any remaining scripts or configurations referencing the `test` folder:

```js
	"format": "prettier --write \"src/**/*.ts\"",
	"lint": "eslint \"{src,apps,libs}/**/*.ts\" --fix",
```

### **Run Linting and Formatting**

Run the following commands to lint and format your code:

```sh
npm run lint
```

```sh
npm run format
```

Resolve all warnings and errors, then proceed.

<br/><br/>

# Develop the Project

### **Disable Body Parser**

Disable NestJS's built-in body parser to allow Better Auth to handle the raw request body:

<details>
  <summary><strong>src/main.ts</strong></summary>

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

</details>

### **Set Up a Swagger (Optional)**

```sh
npm install @nestjs/swagger
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

<br/>

### **Enable Validation and Transformation**

#### **Validation**

Bind `ValidationPipe` at the application level, thus ensuring all endpoints are protected from receiving incorrect data.

```ts
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
      // forbidNonWhitelisted: true // <- Optional
    })
  );

  await app.listen(process.env.PORT ?? 3000);
}

bootstrap().catch((e): void => console.error(e));
```

#### **Transformation**

To enable this behavior globally, set the option on a global pipe:

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common";
async function bootstrap(): Promise<void> {
  const app = await NestFactory.create(AppModule, {
    bodyParser: false,
  });

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      // forbidNonWhitelisted: true
      transform: true, // <- here
    })
  );

  await app.listen(process.env.PORT ?? 3000);
}

bootstrap().catch((e): void => console.error(e));
```

#### **Alternatively**

<details><summary><strong>👉 Alternatively</strong></summary>

Later, this can optionally be done at the method (or controller) level [(docs)](https://docs.nestjs.com/techniques/validation#validation) [(more docs)](https://docs.nestjs.com/pipes#class-validator):

```ts
@Post()
@UsePipes(new ValidationPipe({ transform: true })) // <- here
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

Or (with auto-transformation disabled), you can explicitly cast values using the ParseIntPipe or ParseBoolPipe:

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

If you need to validate arrays in NestJS, refer to [the official documentation](https://docs.nestjs.com/techniques/validation#parsing-and-validating-arrays).

</details>

<br/>

### **Create and Set Up the 'users' Feature**

#### **Generate a Resource (Module/Service/Controller/DTOs)**

You can generate the feature template with a single command and make a few adjustments:

```sh
nest generate resource users
```

```sh
rm -rf ./src/users/entities
```

```sh
echo "" > ./src/users/dto/response-user.dto.ts
```

Or you can create the files manually using separate commands.

```sh
nest generate module users
```

```sh
nest generate service users
```

```sh
nest generate controller users
```

```sh
nest generate class users/dto/create-user.dto --flat
```

```sh
nest generate class users/dto/update-user.dto --flat
```

```sh
nest generate class users/dto/response-user.dto --flat
```

Note: Don’t add any content yet; just ensure the files are created.

- `src/users/users.service.ts` file
- `src/users/users.controller.ts` file
- `src/users/users.module.ts` file
- `src/users/dto/create-user.dto.ts` file
- `src/users/dto/update-user.dto.ts` file
- `src/users/dto/response-user.dto.ts` file

<br/>

Register the Prisma service in the `providers` array of the Users module:

<details>
  <summary><strong>src/users/users.module.ts</strong></summary>

```ts
import { Module } from "@nestjs/common";
import { UsersService } from "./users.service";
import { UsersController } from "./users.controller";
import { PrismaService } from "src/prisma/prisma.service";

@Module({
  imports: [],
  controllers: [UsersController],
  providers: [UsersService, PrismaService], // <-
})
export class UsersModule {}
```

</details>

<br/>

Inject the Prisma service into the `UsersService`:

<details>
  <summary><strong>src/users/users.service.ts</strong></summary>

```ts
import { Injectable } from "@nestjs/common";
import { PrismaService } from "./../prisma/prisma.service";

@Injectable()
export class UsersService {
  public constructor(private readonly prisma: PrismaService) {} // <- Prisma is injected here
  // ... the rest of the file
}
```

</details>

#### **Adjust DTOs**

Before proceeding, make sure these packages are installed.

They are required to generate DTOs.
[(docs1)](https://www.prisma.io/blog/nestjs-prisma-relational-data-7D056s1kOabc#define-the-user-entity-and-dto-classes)
[(docs2)](https://medium.com/@daiki01240/how-to-leverage-swagger-and-class-validator-in-nestjs-api-documentation-and-exporting-type-7577da98768d)

```sh
npm install @nestjs/swagger class-validator class-transformer
```

Adjust the files (DTOs) to fit your project’s style.

_A quick note:_

> In order to make the class properties visible to the `SwaggerModule`, you need to annotate them with the `@ApiProperty()` decorator [(docs)](https://docs.nestjs.com/openapi/types-and-parameters#types-and-parameters).

Then run `npm run format` and `npm run lint`, and fix any warnings or errors that may appear.”

The result should include the following files:

<details><summary><strong>src/users/dto/create-user.dto.ts</strong></summary>

```ts
import { ApiProperty } from "@nestjs/swagger";
import {
  IsEmail,
  IsNotEmpty,
  IsString,
  MaxLength,
  MinLength,
} from "class-validator";

export class CreateUserDto {
  @ApiProperty({
    description: "User name",
    example: "John Doe",
    minLength: 2,
    maxLength: 50,
  })
  @IsNotEmpty()
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  public name!: string;

  @ApiProperty({
    description: "User email address",
    example: "user@example.com",
  })
  @IsEmail()
  public email!: string;

  @ApiProperty({
    description: "User password",
    example: "strongPassword123",
    minLength: 6,
  })
  @IsString()
  @MinLength(6)
  public password!: string;

  // @ApiProperty({
  // 	description: "User profile image URL",
  // 	required: false,
  // 	example: "https://example.com/avatar.jpg",
  // })
  // @IsOptional()
  // @IsString()
  // public image?: string;

  // @ApiProperty({
  // 	description: "User role",
  // 	enum: ["USER", "MODERATOR", "ADMIN"],
  // 	required: false,
  // 	default: "USER",
  // })
  // @IsOptional()
  // @IsString()
  // role?: "USER" | "MODERATOR" | "ADMIN";
  public constructor(data: CreateUserDto) {
    if (data?.name) this.name = data.name;
    if (data?.email) this.email = data.email;
    if (data?.password) this.password = data.password;
    // if (data.image) {
    // 	this.image = data.image;
    // }
  }
}
```

</details>

<br/>

<details><summary><strong>`src/users/dto/update-user.dto.ts`</strong></summary>

```ts
import { ApiProperty } from "@nestjs/swagger";
import { IsString, MaxLength, MinLength } from "class-validator";

// export class UpdateUserDto extends PartialType(CreateUserDto) {
export class UpdateUserDto {
  @ApiProperty({
    description: "User name",
    example: "John Smith",
    minLength: 2,
    maxLength: 50,
  })
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  public name!: string;

  public constructor(data: UpdateUserDto) {
    if (data?.name) this.name = data.name;
    // super(data);
  }
}
```

</details>

<br/>

<details><summary><strong>`src/users/dto/response-user.dto.ts`</strong></summary>

```ts
import { ApiProperty } from "@nestjs/swagger";
// import { Exclude } from "class-transformer";

export class ResponseUserDto {
  @ApiProperty({
    description: "User ID",
    example: "00000000-0000-0000-0000-000000000000",
  })
  public id!: string;

  @ApiProperty({
    description: "User name",
    example: "John Doe",
  })
  public name!: string;

  @ApiProperty({
    description: "User email address",
    example: "user@example.com",
  })
  public email!: string;

  // @ApiProperty({
  // 	description: "Whether email is verified",
  // 	example: false,
  // })
  // public emailVerified: boolean;

  // @ApiProperty({
  // 	description: "User profile image URL",
  // 	required: false,
  // 	example: "https://example.com/avatar.jpg",
  // })
  // public image?: string;

  // @ApiProperty({
  // 	description: "User creation date",
  // 	example: "2023-01-01T00:00:00.000Z",
  // })
  // public createdAt: Date;

  // @ApiProperty({
  // 	description: "User last update date",
  // 	example: "2023-01-01T00:00:00.000Z",
  // })
  // public updatedAt: Date;

  // @ApiProperty({
  // 	description: "User role",
  // 	enum: ["USER", "MODERATOR", "ADMIN"],
  // 	example: "USER",
  // })
  // public role: "USER" | "MODERATOR" | "ADMIN";

  // @Exclude()
  // public password: string;

  // @Exclude()
  // public sessions: any[];

  // @Exclude()
  // public accounts: any[];

  public constructor(data: ResponseUserDto) {
    if (data?.id) this.id = data.id;
    if (data?.name) this.name = data.name;
    if (data?.email) this.email = data.email;

    // Object.assign(this, partial);
  }
}
```

<!-- ```ts
// src/users/dto/login-user.dto.ts
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

</details>

<br/>

#### **Update the controller and service**

Now implement the desired logic using the full power of `class-validator`, `@nestjs/swagger`, `better-auth`, and more.

[(Better Auth decorators)](https://github.com/ThallesP/nestjs-better-auth)
[(OpenAPI/Swagger general decorators)](https://docs.nestjs.com/openapi/decorators)
[(OpenAPI/Swagger response decorators)](https://docs.nestjs.com/openapi/operations#responses)
[(NestJS validation pipes)](https://docs.nestjs.com/techniques/validation)
[(NestJS class-validator decorators)](https://github.com/typestack/class-validator#validation-decorators)

Example resulting files are listed below:

<details>
  <summary><strong>src/users/users.module.ts</strong></summary>

```ts
import { Module } from "@nestjs/common";
import { UsersService } from "./users.service";
import { UsersController } from "./users.controller";
import { PrismaService } from "src/prisma/prisma.service";

@Module({
  controllers: [UsersController],
  providers: [UsersService, PrismaService],
})
export class UsersModule {}
```

</details>

<details>
  <summary><strong>src/users/users.service.ts</strong></summary>

```ts
import { Injectable } from "@nestjs/common";
import { PrismaService } from "src/prisma/prisma.service";
import { CreateUserDto } from "./dto/create-user.dto";
import { UpdateUserDto } from "./dto/update-user.dto";
import { ResponseUserDto } from "./dto/response-user.dto";
import { User } from "@prisma/client";

type UserId = User["id"];
type UserEmail = User["email"];

type FlagsOf<T> = { [K in keyof T]: boolean };

const safeUserResponseProps = {
  id: true,
  name: true,
  email: true,
} satisfies FlagsOf<ResponseUserDto>;

@Injectable()
export class UsersService {
  public constructor(private readonly prisma: PrismaService) {}

  public async create(createUserDto: CreateUserDto): Promise<ResponseUserDto> {
    const result = await this.prisma.user.create({
      data: { name: createUserDto.name, email: createUserDto.email },
      select: safeUserResponseProps,
    });
    return result;
    // You could also do, but it's redundant
    // return new ResponseUserDto(result);
  }

  public async findAll(): Promise<Array<ResponseUserDto>> {
    return this.prisma.user.findMany({ select: safeUserResponseProps });
  }

  public async findById(id: UserId): Promise<ResponseUserDto | null> {
    const result = await this.prisma.user.findUnique({
      where: {
        id,
      },
      select: safeUserResponseProps,
    });

    return result;
  }

  public async findByEmail(email: UserEmail): Promise<ResponseUserDto | null> {
    return this.prisma.user.findUnique({
      where: {
        email,
      },
      select: safeUserResponseProps,
    });
  }

  public async update(
    id: UserId,
    updateUserDto: UpdateUserDto
  ): Promise<ResponseUserDto> {
    return this.prisma.user.update({
      where: {
        id,
      },
      data: updateUserDto,
      select: safeUserResponseProps,
    });
  }

  public async delete(id: UserId): Promise<ResponseUserDto | null> {
    return this.prisma.user.delete({
      where: {
        id,
      },
      select: safeUserResponseProps,
    });
  }
}
```

</details>

<details>
  <summary><strong>src/users/users.controller.ts</strong></summary>

```ts
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  /* ParseArrayPipe, */ Post,
  Put,
  UseGuards,
  UsePipes,
  ValidationPipe,
} from "@nestjs/common";
import { UsersService } from "./users.service";
import { ResponseUserDto } from "./dto/response-user.dto";
import { CreateUserDto } from "./dto/create-user.dto";
import { UpdateUserDto } from "./dto/update-user.dto";
import {
  AuthGuard,
  Public,
  Session,
  type UserSession,
} from "@thallesp/nestjs-better-auth";

@UseGuards(AuthGuard)
@UsePipes(
  new ValidationPipe({
    transform: true,
    whitelist: true,
    forbidNonWhitelisted: true,
  })
)
@Controller("users")
export class UsersController {
  public constructor(
    // private authService: AuthService<typeof auth>,
    private readonly usersService: UsersService
  ) {}

  /* EXAMPLE */
  @Get("by_id/:id")
  public async findOne(
    @Param("id" /*, ParseIntPipe*/) id: string
  ): Promise<ResponseUserDto | null> {
    return this.usersService.findById(id);
  }

  /* EXAMPLE */
  @Public() // Mark this route as public (no authentication required)
  @Post("create")
  public async create(
    @Body(new ValidationPipe()) createUserDto: CreateUserDto
  ): Promise<ResponseUserDto> {
    return this.usersService.create(createUserDto);
  }

  /* EXAMPLE */
  @Put("update/:id")
  public async update(
    @Param("id") id: string,
    @Body() updateUserDto: UpdateUserDto
  ): Promise<ResponseUserDto> {
    return this.usersService.update(id, updateUserDto);
  }

  /* EXAMPLE */
  @Delete("delete/:id")
  public async delete(
    @Param("id") id: string
  ): Promise<ResponseUserDto | null> {
    return this.usersService.delete(id);
  }

  // @Get()
  // public async findAll(): Promise<Array<ResponseUserDto>> {
  // 	return this.usersService.findAll();
  // }

  // @Post()
  // createBulk(
  // 	@Body(new ParseArrayPipe({ items: CreateUserDto }))
  // 	createUserDtos: CreateUserDto[],
  // ) {
  // 	return "This action adds new users";
  // }

  @UsePipes(new ValidationPipe({ transform: true }))
  @UseGuards(AuthGuard)
  @Public()
  @Get("me")
  public getProfile(@Session() session: UserSession): ResponseUserDto {
    return {
      id: session.user.id,
      name: session.user.name,
      email: session.user.email,
    };
  }
}
```

</details>

You can optionally add a response validator (like [here](https://medium.com/@kuba.2001/reponse-validation-in-nestjs-0db70b955a6a) and [here](https://www.reddit.com/r/nestjs/comments/1knaeze/comment/mssircd/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)
), but it’s usually not needed as long as you clearly pick which fields/properties to return from the ORM in your service.

<br/><br/>

### **Create Auth service**

<details>
  <summary><strong>src/auth/auth.service.ts</strong></summary>
</details>
<details>
  <summary><strong>src/auth/auth.controller.ts</strong></summary>
</details>

<br/><br/>

# FAQ

[(better-auth endpoints)](https://www.better-auth.com/docs/plugins/username#usage)
Flow:

- Add model
- Generate prisma types
- Create the resource/module
- Import Prisma service in the module
- Inject Prisma service in the service
