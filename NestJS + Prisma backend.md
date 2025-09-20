# Prerequisites

- Docker
- Node.js
- nvm (optional)

# Install NestJS CLI

### Install

```sh
npm install -g @nestjs/cli
```

### Verify the Installation

```sh
nest --version
```

# Set Up the Project

### Navigate to Your Project Directory

#### Option 1: Create the Project in a New Folder

Navigate to the directory where you want to place the project and run:

```sh
cd path/to/your/directory
```

```sh
nest new my_project
```

```sh
cd my_project
```

This creates a new folder `my_project` with a ready-to-use NestJS project.

#### Option 2: Create the Project in the Current Folder

If you want to initialize the project in the current folder:

```sh
mkdir my_project
```

```sh
cd my_project
```

```sh
nest new .
```

⚠️ Ensure the current folder is empty to avoid conflicts with existing files.

Then navigate to your project folder (if not already there).

# Install Dependencies

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

# Set Up the Database

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

Then run:

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
  <summary>File: src/auth.ts</summary>

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

Run the command and accept all prompts:

```sh
npx @better-auth/cli generate
```

This overwrites your `schema.prisma` (models), but if following this guide from the beginning, no models should exist yet.

Then run the migration command:

```sh
npx prisma migrate dev --name init
```

This will:

- Create a new migration in prisma/migrations named init.
- Apply the migration to your database.
- Generate Prisma Client in node_modules/.prisma/client.

# Configure Project Files (Optional)

Customize project configuration files as needed. For example:

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

### Ensure the Following VS Code Extensions Are Installed

- **ESLint**
- **Prettier - Code formatter**

### Run Linting and Formatting

Run the following commands to lint and format your code:

```sh
npm run lint
```

```sh
npm run format
```

Resolve all warnings and errors, then proceed.

### Remove Spec Files (Optional)

If the generated `.spec.ts` files for testing are unnecessary, remove them.

### Add "no-spec" Setting to `nest-cli.json` (Optional)

Edit `nest-cli.json` and add the following:

```js
{
	// ...
	"generateOptions": {
		"spec": false
	}
	// ...
}
```

This ensures that the output directory is cleaned on each build.

### Remove Test Scripts and Folder (Optional)

If tests are not planned, remove all test-related scripts from `package.json` and update any remaining scripts or configurations referencing the `test` folder:

```js
	"format": "prettier --write \"src/**/*.ts\"",
	"lint": "eslint \"{src,apps,libs}/**/*.ts\" --fix",
```

### Run Linting and Formatting One More Time

```sh
npm run lint
```

```sh
npm run format
```

Confirm no errors or warnings remain.

# Develop the Project

### Disable Body Parser

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

### Import AuthModule

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

### Create and Set Up a Prisma Service

[(docs 1)](https://docs.nestjs.com/recipes/prisma#use-prisma-client-in-your-nestjs-services)
[(docs 2)](https://www.prisma.io/nestjs)

Run the following command to generate a Prisma service:

```sh
nest generate service prisma
```

This creates a Prisma service in the `src/prisma` directory.

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

### Create and Set Up the 'users' Feature

You can generate the feature with a single command:

```sh
nest generate resource users
```

Or you can create the files manually using separate commands.

The result should include the following files:

`users.service.ts` file
`users.controller.ts` file
`users.module.ts` file

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

# FAQ
