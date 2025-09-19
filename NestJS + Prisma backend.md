Prerequisites
- Docker
- Node.js
- nvm (optional)

# Install NestJS CLI

### Install 
```sh
npm i -g @nestjs/cli
```

### Check version
```sh
nest --version
```
<hr/>

# Set up the project

### Navigate to your project directory

#### Option 1: Create the project in a new folder

Navigate to the directory where you want to place the project and run:
```sh
cd path/to/your/directory
```
```sh
nest new my_project
```
This will create a new folder `my_project` with a ready-to-use NestJS project.

#### Option 2: Create the project in the current folder

If you want to initialize the project in the current folder:
```sh
nest new .
```
⚠️ Make sure the current folder is empty to avoid conflicts with existing files.
<br/><br/>
Then navigate to your project folder (if you don't already).

# Install dependencies

### dotenv [(docs)](https://www.npmjs.com/package/dotenv)
Install
```sh
npm install dotenv --save
```
Create and configure a `.env`
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


# Set up the database

Create a `docker-compose.yaml`
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
Or start *just* the `db` service:
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


⚠️ Make sure the `output` field is commented out at `shema.prisma` file. Otherwise, you’ll need to take it into account in later configuration steps.

#### Prisma scheme file example:

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

<br/><br/>
Then run:
```sh
npx prisma generate
```

### Better Auth  

[(docs 1)](https://www.better-auth.com/docs/installation)
[(docs 2)](https://www.better-auth.com/docs/integrations/nestjs)
[(docs 3)](https://github.com/ThallesP/nestjs-better-auth)

#### Better Auth instance configuration

```sh
npm install better-auth @thallesp/nestjs-better-auth
```

Create a `.env` file (if not perfomed yet)

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

In this file: import Better Auth and create your auth instance (don't forget the `prismaAdapter` here).

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
This will overwrite your `schema.prisma` (models), but if you’ve been following this guide from the beginning, you shouldn’t have any yet.

Then run the migration command:
```sh
npx prisma migrate dev --name init
```
This will:
- Create a new migration in prisma/migrations with the name init.
- Apply the migration to your database.
- Generate Prisma Client in node_modules/.prisma/client.

# Configure project files (optional)

You can set up the project configuration files to match your preferences. For example:

<details>
	<summary><strong>tsconfig.json</strong> — TypeScript compiler options</summary>

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
	<summary>.prettierrc</strong> — Prettier formatting options</summary>

```prettier
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
	<summary><strong>.eslintrc</strong> — ESLint rules and formatting</summary>

```ts
```

</details>


# Develop the project
