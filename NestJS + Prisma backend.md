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

### Prisma [(docs)](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql)

```sh
npm install prisma
```

```sh
npx prisma init
```

⚠️ Ensure the `output` field is commented out in the `schema.prisma` file. Otherwise, account for it in later configuration steps.

#### Prisma Schema File Example

<details>
  <summary>File: prisma/schema.prisma</summary>

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

[(docs 1)](https://www.better-auth.com/docs/installation)
[(docs 2)](https://www.better-auth.com/docs/integrations/nestjs)
[(docs 3)](https://github.com/ThallesP/nestjs-better-auth)

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

### Set Up a Swagger (Optional)

```sh
npm install @nestjs/swagger
```

Once the installation process is complete, open the `main.ts` file and initialize Swagger using the `SwaggerModule` class [(docs)](https://docs.nestjs.com/openapi/introduction#bootstrap).

<details>
  <summary><strong>src/main.ts</strong></summary>

```ts
import { DocumentBuilder, OpenAPIObject, SwaggerModule } from "@nestjs/swagger";
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
  SwaggerModule.setup("api", app, documentFactory);
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

### **Create and Set Up 'Users' DTOs**

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

The result should include the following files:

<details><summary><strong>`src/generated-dto/dto/connect-user.dto.ts`</strong></summary>

```ts
export class ConnectUserDto {
  public id?: string;
  public email?: string;

  public constructor(data: ConnectUserDto) {
    if (data.id) {
      this.id = data.id;
    }
    if (data.email) {
      this.email = data.email;
    }
  }
}
```

</details>

<br/>

<details><summary><strong>`src/generated-dto/dto/create-user.dto.ts`</strong></summary>

```ts
import { IsNotEmpty, IsOptional, IsString } from "class-validator";

export class CreateUserDto {
  @IsNotEmpty()
  @IsString()
  public name: string;

  @IsNotEmpty()
  @IsString()
  public email: string;

  @IsOptional()
  @IsString()
  public image?: string;

  public constructor(data: CreateUserDto) {
    this.name = data.name;
    this.email = data.email;
    if (data.image) {
      this.image = data.image;
    }
  }
}
```

</details>

<br/>

<details><summary><strong>`src/generated-dto/dto/update-user.dto.ts`</strong></summary>

```ts
import { IsOptional, IsString } from "class-validator";

export class UpdateUserDto {
  @IsOptional()
  @IsString()
  public name?: string;

  @IsOptional()
  @IsString()
  public email?: string;

  @IsOptional()
  @IsString()
  public image?: string;

  public constructor(data: UpdateUserDto) {
    if (data.image) {
      this.image = data.image;
    }
    if (data.name) {
      this.name = data.name;
    }
    if (data.email) {
      this.email = data.email;
    }
  }
}
```

</details>

<br/>

<details><summary><strong>`src/generated-dto/dto/user.dto.ts`</strong></summary>

```ts
import { Role } from "@prisma/client";
import { ApiProperty } from "@nestjs/swagger";

export class UserDto {
  public id: string;
  public name: string;
  public email: string;
  public emailVerified: boolean;
  public image: string | null;

  @ApiProperty({
    type: `string`,
    format: `date-time`,
  })
  public createdAt: Date;

  @ApiProperty({
    type: `string`,
    format: `date-time`,
  })
  public updatedAt: Date;

  @ApiProperty({
    enum: Role,
  })
  public role: Role;

  public constructor(data: UserDto) {
    this.id = data.id;
    this.name = data.name;
    this.email = data.email;
    this.emailVerified = data.emailVerified;
    this.image = data.image;
    this.createdAt = data.createdAt;
    this.updatedAt = data.updatedAt;
    this.role = data.role;
  }
}
```

</details>

<br/>

<details><summary><strong>`src/generated-dto/entities/user.entity.ts`</strong></summary>

```ts
import { Role } from "@prisma/client";
import { ApiProperty } from "@nestjs/swagger";
import { Session } from "../../session/entities/session.entity";
import { Account } from "../../account/entities/account.entity";

export class User {
  public id: string;
  public name: string;
  public email: string;
  public emailVerified: boolean;
  public image: string | null;

  @ApiProperty({
    type: `string`,
    format: `date-time`,
  })
  public createdAt: Date;

  @ApiProperty({
    type: `string`,
    format: `date-time`,
  })
  public updatedAt: Date;

  @ApiProperty({
    enum: Role,
  })
  public role: Role;

  public sessions?: Session[];
  public accounts?: Account[];

  public constructor(data: User) {
    this.id = data.id;
    this.name = data.name;
    this.email = data.email;
    this.emailVerified = data.emailVerified;
    this.image = data.image;
    this.createdAt = data.createdAt;
    this.updatedAt = data.updatedAt;
    this.role = data.role;
    this.sessions = data.sessions;
    this.accounts = data.accounts;
  }
}
```

</details>

<br/>

#### **Enable Validation**

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

#### **Enable Transformation**

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

Later, this can optionally be done at the method (or controller) level [(docs)](https://docs.nestjs.com/techniques/validation#validation) [(more docs)](https://docs.nestjs.com/pipes#class-validator):

```ts
@Post()
@UsePipes(new ValidationPipe({ transform: true })) // <- here
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

Alternatively (with auto-transformation disabled), you can explicitly cast values using the ParseIntPipe or ParseBoolPipe:

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

If you need to validate arrays in NestJS, refer to 👉 [the official documentation](https://docs.nestjs.com/techniques/validation#parsing-and-validating-arrays).

### **Create and Set Up the 'users' Feature**

#### **Generate a Resource (Module/Service/Controller)**

You can generate the feature template with a single command:

```sh
nest generate resource users
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

Note: Don’t add any content yet; just ensure the files are created.

- `src/users/users.service.ts` file
- `src/users/users.controller.ts` file
- `src/users/users.module.ts` file

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
import { CreateUserDto } from "./dto/create-user.dto";
// import { UpdateUserDto } from "./dto/update-user.dto";
import { PrismaService } from "./../prisma/prisma.service";

@Injectable()
export class UsersService {
  public constructor(private readonly prisma: PrismaService) {} // <- Prisma is injected here
  // ... the rest of the file
}
```

</details>

#### **Update the controller and service**

And now implement the desired logic. Example resulting files are listed below:

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
import { CreateUserDto } from "src/generated-dto/user/dto/create-user.dto";
import { UpdateUserDto } from "src/generated-dto/user/dto/update-user.dto";
import { UserDto } from "src/generated-dto/user/dto/user.dto";
import { UserEntity } from "src/generated-dto/user/entities/user.entity";
import { PrismaService } from "src/prisma/prisma.service";

type UserId = UserEntity["id"];
type UserEmail = UserEntity["email"];

@Injectable()
export class UsersService {
  public constructor(private readonly prisma: PrismaService) {}

  public async create(createUserDto: CreateUserDto): Promise<UserDto> {
    return this.prisma.user.create({ data: createUserDto });
  }

  public async findAll(): Promise<Array<UserDto>> {
    return this.prisma.user.findMany();
  }

  public async findById(id: UserId): Promise<UserDto | null> {
    return this.prisma.user.findUnique({
      where: {
        id,
      },
    });
  }

  public async findByEmail(email: UserEmail): Promise<UserDto | null> {
    return this.prisma.user.findUnique({
      where: {
        email,
      },
    });
  }

  public async update(
    id: UserId,
    updateUserDto: UpdateUserDto
  ): Promise<UserDto> {
    return this.prisma.user.update({
      where: {
        id,
      },
      data: updateUserDto,
    });
  }

  public async delete(id: UserId): Promise<UserDto | null> {
    return this.prisma.user.delete({
      where: {
        id,
      },
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
  UsePipes,
  ValidationPipe,
} from "@nestjs/common";
import { UsersService } from "./users.service";
import { CreateUserDto } from "src/generated-dto/user/dto/create-user.dto";
import { UserDto } from "src/generated-dto/user/dto/user.dto";
import { UpdateUserDto } from "src/generated-dto/user/dto/update-user.dto";

@UsePipes(
  new ValidationPipe({
    transform: true,
    whitelist: true,
    forbidNonWhitelisted: true,
  })
)
@Controller("users")
export class UsersController {
  public constructor(private readonly usersService: UsersService) {}

  @UsePipes(new ValidationPipe({ transform: true }))
  @Get("by_id/:id")
  public async findOne(
    @Param("id" /*, ParseIntPipe*/) id: string
  ): Promise<UserDto | null> {
    return this.usersService.findById(id);
  }

  @Post()
  public async create(
    @Body(new ValidationPipe()) createUserDto: CreateUserDto
  ): Promise<UserDto> {
    return this.usersService.create(createUserDto);
  }

  @Put(":id")
  public async update(
    @Param("id") id: string,
    @Body() updateUserDto: UpdateUserDto
  ): Promise<UserDto> {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete()
  public async delete(@Param("id") id: string): Promise<UserDto | null> {
    return this.usersService.delete(id);
  }
  // @Get()
  // public async findAll(): Promise<Array<UserDto>> {
  // 	return this.usersService.findAll();
  // }

  // @Post()
  // createBulk(
  // 	@Body(new ParseArrayPipe({ items: CreateUserDto }))
  // 	createUserDtos: CreateUserDto[],
  // ) {
  // 	return "This action adds new users";
  // }
}
```

</details>

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

Flow:

- Add model
- Generate prisma types
- Create the resource/module
- Import Prisma service in the module
- Inject Prisma service in the service
