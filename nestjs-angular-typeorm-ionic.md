1. Install nestjs
2. Create project folder
3. Create nested folder for server in it by initializing a nestjs project

```shell
nest new server
```

4. Install additional dependencies

```shell
npm i @nestjs/typeorm typeorm pg @nestjs/config
```

5. Create the **.env** file

```shell
touch .env
```

6. Fill in the **.env** file

```js
POSTGRES_POST=127.0.0.1
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DATABASE=treesn_db
```

7. Set up configs in **app.module.ts**

```ts
// ...
@Module({
    imports: [
        ConfigModule.forRoot({isGlobal: true}),
        TypeOrmModule.forRoot({
            type: "postgres",
            host: process.env.POSTGRES_HOST,
            port: parseInt(<string>process.env.POSTGRES_PORT, 10),
            username: process.env.POSTGRES_USER,
            password: process.env.POSTGRES_PASSWORD,
            database: process.env.POSTGRES_DATABASE,
        })
    ]
})
```
