- Init workspace
```
npx create-nx-workspace@latest \
--preset=nest \
--name=nestjs-project \
--appName=backend \
--routing=true \
--docker=true \
--e2eTestRunner=none \
--packageManager=npm \
--skipGit=true \
--ssr=false \
--workspaceType=integrated
```

- [Optional] Add git origin

- Install plugins
```
// npm install @nrwl/angular
nx add @nx/angular
npm i @nx/nest
```

```
nx generate @nx/angular:application \
--name frontend \
--style scss \
--prefix fse \
--tags type:app,scope:client \
--strict \
--backendProject backend \
--standalone \
--routing \
--e2eTestRunner=none \
--inlineStyle=true \
--inlineTemplate=true \
--skipTests=true \
--ssr=false \
--unitTestRunner=none \
--directory=apps/frontend
```



- You can add this to your nx.json and it will do what you want. I agree that it’s frustrating when they change it seemingly at random.
```
"workspaceLayout": {
    "appsDir": "apps",
    "libsDir": "packages"
  }
```

- Add prisma
```
npm i prisma @prisma/client
```
- Perform prisma init
```
npx prisma init
```
## Create two (shared) libs: one for a schema, one for a nestjs module (+ service). [Guide here](https://github.com/nrwl/nx-recipes/tree/main/nestjs-prisma).

```
npx nx generate @nx/js:library --name=libs/server/data-access --importPath=@ws5/server/data-access --projectNameAndRootFormat=as-provided --tags=type:data-access,scope:backend --unitTestRunner=none --no-interactive
```

### First one



- Move _schema.prisma_ file to the root of the _prisma-schema_ lib folder
- Remove _src_ from the root of the _prisma-schema_ lib folder
- Update _output_ path in the _schema.prisma_ file
```
generator client {
  provider = "prisma-client-js"
  output = "../../../node_modules/.prisma/client"
}
```
- Add models
```

enum Role {
  USER
  ADMIN
}

model User {
  id          Int     @id @default(autoincrement())
  username    String  @unique
  password    String
  displayName String? @default("")
  role        Role    @default(USER)
}

model UserSetting {
  id               Int     @id @default(autoincrement())
  isNotificationOn Boolean
  isSmsEnabled     Boolean
}

model Post {
  id          Int    @id @default(autoincrement())
  title       String
  description String
}

// Orders & Customers

model Customer {
  customer_id  String  @id @default(uuid())
  company_name String  @db.VarChar(40)
  orders       Order[]

  CustomerSetting CustomerSetting?
}

model CustomerSetting {
  setting_id           String  @id @default(uuid())
  customer_description String? @db.Text()

  Customer    Customer @relation(fields: [customer_id], references: [customer_id])
  customer_id String   @unique
}

model Order {
  orded_id    String        @id @default(uuid())
  order_date  DateTime      @db.Date
  customer_id String
  Customer    Customer      @relation(fields: [customer_id], references: [customer_id])
  OrderDetail OrderDetail[]
}

model Product {
  product_id   String        @id @default(uuid())
  product_name String        @db.VarChar(40)
  unit_price   Float?        @db.Real
  OrderDetail  OrderDetail[]
}

model OrderDetail {
  order_id   String
  product_id String

  quantity Int @db.SmallInt

  Order   Order   @relation(fields: [order_id], references: [orded_id])
  Product Product @relation(fields: [product_id], references: [product_id])

  @@id([order_id, product_id])
}
```

- Update _project.json_ for the _prisma-schema_ lib
```
{
  "name": "prisma-schema",
  "$schema": "../../../node_modules/nx/schemas/project-schema.json",
  "sourceRoot": "libs/shared/prisma-schema/src",
  "projectType": "library",
  "tags": ["type:lib", "scope:shared"],
  "// targets": "to see all targets run: nx show project prisma-schema --web",
  "targets": {
    "prisma": {
      "command": "prisma",
      "options": {
        "cwd": "./libs/shared/prisma-schema"
      }
    },
    "migrate": {
      "command": "prisma migrate dev",
      "options": {
        "cwd": "./libs/shared/prisma-schema"
      }
    },
    "generate-types": {
      "command": "prisma generate",
      "options": {
        "cwd": "./libs/shared/prisma-schema"
      }
    }
  }
}
```

- Update _.env_ file (move up?)
```
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=db1
POSTGRES_PORT=5432
POSTGRES_HOST=postgres_dev_cont
# POSTGRES_PORT=9999
# POSTGRES_HOST=localhost
JWT_SECRET = 'secretKey'
JWT_EXPIRES_IN = '1h'
DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}?schema=public
```

- Run _postgres_ in a container or any available way

- Run type generating and migration
```
nx run prisma-schema:generate-types
nx run prisma-schema:migrate
```
### Second one

- Command
```
npx nx generate @nx/nest:library \
--name=libs/backend/prisma-provider \
--importPath=@fs4/prisma-provider \
--projectNameAndRootFormat=as-provided \
--service=true \
--tags=type:lib,scope:backend \
--unitTestRunner=none \
--no-interactive
```


## Generate feature
```
npx nx generate @nx/nest:library --name=libs/backend/features/post --controller=true --importPath=@fs4/feature-post --projectNameAndRootFormat=as-provided --service=true --tags=type:feature,scope:backend --unitTestRunner=none --no-interactive 
```

- Import the prisma provider module to the feature module
```
...
import { PrismaProviderModule } from '@fs4/prisma-provider';

@Module({
  imports: [PrismaProviderModule],
...
```
- And then to the feature service and the controller

## Import feature
- Add the feature to the imports array of the backend app module
```
import { Module } from '@nestjs/common';

import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PostModule } from '@fs4/feature-post';

@Module({
  imports: [PostModule], // <---
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
