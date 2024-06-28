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
- Create two (shared) libs: one for a schema, one for a nestjs module (+ service). [Guide here](https://github.com/nrwl/nx-recipes/tree/main/nestjs-prisma).

- Perform prisma init
```
npx prisma init
```

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

- Update _.env_ file
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
