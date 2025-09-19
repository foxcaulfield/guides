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

# Install dependencies

### dotenv [(docs)](https://www.npmjs.com/package/dotenv)
Install
```sh
npm install dotenv --save
```
Create and configure a `.env`
```sh
mkdir .env
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

### Better Auth  [(docs)](https://www.better-auth.com/docs/integrations/nestjs)

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

In this file: import Better Auth and create your auth instance.
```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    database: // ...
})
```
