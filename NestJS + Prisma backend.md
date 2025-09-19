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

Navigate to your project folder (if you don't already).

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
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

### Prisma [(docs)](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql)

```sh
npm install prisma 
```
```sh
npx prisma init
```
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
mkdir src/auth.ts
```

In this file: import Better Auth and create your auth instance (don't forget the `prismaAdapter` here).

<details>
  <summary>File: src/auth.ts</summary>

```ts
import { PrismaClient } from "@prisma/client";
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";

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
