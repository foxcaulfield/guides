- Install nestjs

```shell
npm install -g @nestjs/cli
```

- Create a new nesjs app

```shell
nest new project-name
nest new project-name --skip-git
```

- Install **prisma**

```shell
npm i -D prisma @prisma/client
```

- Initialize **prisma**

```shell
npx prisma init
```

- Create a model

```shell
npm i passport passport-jwt @types/passport
npm i @nestjs/passport @nestjs/jwt bcrypt
npm i -D @types/passport-jwt @types/bcrypt
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
```
nest generate service database
```

