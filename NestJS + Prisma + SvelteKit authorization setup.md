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
  <summary>See file:</summary>

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

---

### Integrate Better Auth in NestJS

Better Auth needs raw request bodies for its handlers. 
So, in **main.ts**, disable the default parser when creating the app (as per docs [better-auth.com](https://www.better-auth.com/docs/integrations/nestjs)).
Also make sure to set `credentials: true` and the correct origin.

<details>
	<summary>See file:</summary>

`File: src/main.ts`
```TypeScript 
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { bodyParser: false });
  app.enableCors({
    origin: ['http://localhost:5173'], // Svelte app server port
    credentials: true                  // allow cookies
  });
  await app.listen(process.env.PORT || 3000);
}
bootstrap();
```
 
</details>

---

### Import AuthModule

In your `AppModule`, import `AuthModule.forRoot(auth)`, passing the instance you created:

<details>
	<summary>See file:</summary>

`File: src/app.module.ts`
```TypeScript
import { Module } from "@nestjs/common";
import { AuthModule } from "@thallesp/nestjs-better-auth";
import { auth } from "./auth";  // path to your auth instance

@Module({
  imports: [ AuthModule.forRoot(auth) ],
  // ... controllers/providers as usual
})
export class AppModule {}

```
 
</details>

---

### Protect routes with AuthGuard

```TypeScript
```

---

# 2. Frontend setup
