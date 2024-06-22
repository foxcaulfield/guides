# Comprehensive nestjs guide

## Installing and initializing

```shell
npm install -g @nestjs/cli
nest new project-name --skip-git
```

## Dependencies

- Note: Install **prisma** (you don't need to manually install the @prisma/client;
  it will be installed automatically on the first migration)

```sh
npm i \
prisma \
passport \
passport-local \
passport-jwt \
class-validator \
class-transformer \
bcrypt \
@nestjs/jwt \
@nestjs/passport \
@nestjs/throttler \
&& npm i -D \
@types/passport-local \
@types/passport-jwt \
@types/passport \
@types/bcrypt
# @prisma/client \
# @nestjs/typeorm \
# typeorm \
# pg \
# @nestjs/config
```

## Prisma initialization

```sh
npx prisma init
```

- Wtite models

- Set up _env_ file:

```yml
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=db1
POSTGRES_PORT=5432
POSTGRES_HOST=postgres_dev_cont
JWT_SECRET = 'secretKey'
JWT_EXPIRES_IN = '1h'

DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}?schema=public
```

```sh
npx prisma generate
```

- [Optional] Make a migration

```sh
npx prisma migrate dev --name init
```

---

Important note:
**Use** this `Prisma.UserCreateInput` in **service**.
**DON'T USE** this in **controller**.

---

In order to handle data in **controller** create and use _dto_ with `class-validator`: