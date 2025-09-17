# 1. Backend setup

### Install dependencies

```sh
npm install better-auth @better-auth/adapters-prisma @thallesp/nestjs-better-auth
```

---

### Configure environment

```env
BETTER_AUTH_SECRET=your_secret_string
DATABASE_URL=postgresql://USER:PASSWORD@localhost:5432/mydb
BETTER_AUTH_URL=http://localhost:3000 # Backend app URL
```

---

### Initialize Prisma

If not already done, run `npx prisma init` and configure your **schema.prisma** with the existing models (User, collections, exercises, etc.). 

Then (if not already done) create an initial migration:
```sh
npx prisma migrate dev --name init_schema
npx prisma generate
```

---

### Configure Better Auth Instance

<details>
  <summary>Click to expand</summary>

`File: src/auth.ts`
```TypeScript
import { PrismaClient } from "@prisma/client";
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
// import { Pool } from "pg";

const prismaClient = new PrismaClient();
export const auth = betterAuth({
	//...
	secret: process.env.BETTER_AUTH_SECRET,
	database: prismaAdapter(prismaClient, {
		provider: "postgresql",
	}),
	emailAndPassword: {
		enabled: true,
	},
	trustedOrigins: ["http://localhost:5173"], // Frontend URL
});
```

</details>

---

### Generate Better Auth schema

Run the Better Auth CLI to add its tables (users, sessions, accounts, etc.) to your Prisma schema. This will modify your **schema.prisma** to include the necessary models:

```sh
npx @better-auth/cli generate
```


Then run another migration to apply them to the database:

```sh
npx prisma migrate dev --name init_better_auth_schemas
npx prisma generate
```
# 2. Frontend setup
