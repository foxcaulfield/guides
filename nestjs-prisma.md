- Install nestjs

```shell
npm install -g @nestjs/cli
```

- Create a new nesjs app

```shell
nest new project-name
nest new project-name --skip-git
```

- Install **prisma** (you do need to manually install the @prisma/client;
  it will be installed automatically on the first migration)

```shell
npm i prisma @prisma/client
```

- Initialize **prisma**

```shell
npx prisma init
```

- Don't forget to add this line to env file (when using PostgreSQL)

```yml
POSTGRES_PASSWORD=password
```

- Create a model

```shell
npm i passport passport-jwt @types/passport
npm i @nestjs/passport @nestjs/jwt bcrypt
npm i -D @types/passport-jwt @types/bcrypt

npm i class-validator
```

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id   Int    @id @default(autoincrement())
  name String
}
```

- Generate a database service

```shell
nest generate service database
```

- Content of a database service file

```ts
import { Injectable, OnModuleDestroy, OnModuleInit } from "@nestjs/common";
import { PrismaClient } from "@prisma/client";

@Injectable()
export class DatabaseService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

- Create
