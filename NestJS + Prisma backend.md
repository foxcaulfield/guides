# Install NestJS CLI

## Install 
```sh
npm i -g @nestjs/cli
```

## Check version
```sh
nest --version
```
<hr/>

# Set up the project

## Navigate to your project directory

### Option 1: Create the project in a new folder

Navigate to the directory where you want to place the project and run:
```sh
cd path/to/your/directory
nest new my_project
```
This will create a new folder `my_project` with a ready-to-use NestJS project.

### Option 2: Create the project in the current folder

If you want to initialize the project in the current folder:
```sh
nest new .
```
⚠️ Make sure the current folder is empty to avoid conflicts with existing files.

# Install dependencies

```sh
npm install prisma
```
