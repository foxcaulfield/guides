<br/><br/>

# Prerequisites

- Docker
- Node.js
- nvm (optional)

<br/><br/>

<!-- === === === === === -->
<!-- === === === === === -->
<!-- === === === === === -->

# 1. Install NestJS CLI

### Install

```sh
npm install -g @nestjs/cli
```

### Verify the Installation

```sh
nest --version
```

<br/><br/>

<!-- === === === === === -->
<!-- === === === === === -->
<!-- === === === === === -->

# 2. Set Up the Project

#### Option 1: Create the Project in a New Folder

Navigate to the directory where you want to place the project and run.

<!-- _Note: Add the `--no-spec` flag if you don’t want test files to be generated._ -->

```sh
cd path/to/your/directory
```

```sh
nest new my_project
```

<!-- nest new my_project . --no-spec -->

```sh
cd my_project
```

This creates a new folder `my_project` with a ready-to-use NestJS project.

</br>

#### Option 2: Create the Project in the Current Folder

Alternatively, if you want to initialize the project in the current folder:

```sh
mkdir my_project
```

```sh
cd my_project
```

```sh
nest new .
```

</br>

<!-- nest new . --no-spec -->

⚠️ Ensure the current folder is empty to avoid conflicts with existing files.

Then navigate to your project folder (if not already there).

<br/><br/>

<!-- === === === === === -->
<!-- === === === === === -->
<!-- === === === === === -->

# 3. Add and Customize Configs (optional)

### Customize Configs

Customize project configuration files as needed. For example:

<br/>

<!-- **VS Code Debug config** -->

<details>
<summary>.vscode/launch.json — VS Code Debugger Options</summary>

```JavaScript
{
	// Use IntelliSense to learn about possible attributes.
	// Hover to view descriptions of existing attributes.
	// For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
	"version": "0.2.0",
	"configurations": [
		{
			"type": "node",
			"request": "launch",
			"name": "Debug NestJS",
			"runtimeArgs": ["-r", "ts-node/register", "-r", "tsconfig-paths/register"],
			"args": ["-r", "tsconfig-paths/register", "src/main.ts"],
			"console": "integratedTerminal",
			"sourceMaps": true,
			"env": {
				"PORT": "3000" // Set your desired port here
			}
		},
		{
			"type": "node",
			"request": "launch",
			"name": "Alt. Debug NestJS",
			"runtimeExecutable": "npm",
			"runtimeArgs": ["run", "start:debug", "--", "--inspect-brk"],
			"console": "integratedTerminal",
			"restart": true,
			"autoAttachChildProcesses": true
		}
	]
}
```

</details>

<br/>

<!-- **TypeScript Compiler Config** -->

<details>
<summary>tsconfig.json — TypeScript Compiler Options</summary>

```ts
{
	"compilerOptions": {
		"module": "nodenext",
		"moduleResolution": "nodenext",
		"resolvePackageJsonExports": true,
		"esModuleInterop": true,
		"isolatedModules": true,
		"declaration": true,
		"removeComments": true,
		"emitDecoratorMetadata": true,
		"experimentalDecorators": true,
		"allowSyntheticDefaultImports": true,
		"target": "ES2023",
		"sourceMap": true,
		"outDir": "./dist",
		"baseUrl": "./",
		"incremental": true,
		"skipLibCheck": true,
		"forceConsistentCasingInFileNames": true,

		// Strict Checks
		"alwaysStrict": true,
		"noImplicitAny": true,
		"strictNullChecks": true,
		"strictPropertyInitialization": true,
		"strictFunctionTypes": true,
		"noImplicitThis": true,
		"strictBindCallApply": true,
		// "noPropertyAccessFromIndexSignature": true,
		"noUncheckedIndexedAccess": true,

		// Linter Checks
		"noImplicitReturns": true, // https://eslint.org/docs/rules/consistent-return ?
		"noFallthroughCasesInSwitch": true, // https://eslint.org/docs/rules/no-fallthrough
		"noUnusedLocals": true, // https://eslint.org/docs/rules/no-unused-vars
		"noUnusedParameters": true, // https://eslint.org/docs/rules/no-unused-vars#args
		"allowUnreachableCode": false, // https://eslint.org/docs/rules/no-unreachable ?
		"allowUnusedLabels": false, // https://eslint.org/docs/rules/no-unused-labels

		// Base Strict Checks
		"noImplicitUseStrict": false,
		"suppressExcessPropertyErrors": false,
		"suppressImplicitAnyIndexErrors": false,
		"noStrictGenericChecks": false
	}
}
```

</details>

<br/>

<!-- **Prettier Plugin Config** -->

<details>
<summary>.prettierrc — Prettier Formatting Options</summary>

```json
{
  "singleQuote": false,
  "trailingComma": "all",
  "useTabs": true,
  "tabWidth": 4,
  "printWidth": 120
}
```

</details>

<br/>

<!-- **ESLint Plugin Config** -->

<details>
<summary>.eslint.config.mjs — ESLint Rules and Formatting</summary>

```ts
//...
rules: {
			// ... existing rules
			"@typescript-eslint/no-explicit-any": "off",
			"@typescript-eslint/no-floating-promises": "warn",
			"@typescript-eslint/no-unsafe-argument": "warn",
			"prettier/prettier": [
				"error",
				{
					// endOfLine: "auto",
					printWidth: 120,
					// trailingComma: "es5",
					// semi: false,
					doubleQuote: true,
					// jsxSingleQuote: true,
					singleQuote: false,
					useTabs: true,
					// tabWidth: 4,
				},
			],
			"@typescript-eslint/explicit-member-accessibility": ["error", { accessibility: "explicit" }],
			"@typescript-eslint/explicit-function-return-type": [
				"error",
				{
					allowExpressions: false,
					allowTypedFunctionExpressions: false,
					allowHigherOrderFunctions: false,
					allowDirectConstAssertionInArrowFunctions: false,
					allowConciseArrowFunctionExpressionsStartingWithVoid: false,
				},
			],
			// ... existing rules
		},
// ...
```

</details>

<br/>

### **Remove Spec/Test Files and Scripts**

<details>
<summary>Settings</summary>
</dl></dd>

You can remove `.spec.ts` files if you don’t plan to use them.

```sh
find ./src -type f -name "*.spec.ts" -exec rm -i {} \; && rm -rf ./test;
```

#### **Add "no-spec" Setting to `nest-cli.json`**

Edit `nest-cli.json` and add the following to disable test file generation globally:

```js
{
	// ...
	"generateOptions": {
		"spec": false
	}
	// ...
}
```

This way you won’t need to use the --no-spec flag every time you generate a new file.

#### **Remove Test Scripts and Folder**

If tests are not planned, remove all test-related scripts from `package.json` and update any remaining scripts or configurations referencing the `test` folder:

```js
	"format": "prettier --write \"src/**/*.ts\"",
	"lint": "eslint \"{src,apps,libs}/**/*.ts\" --fix",
```

</dd></dl>
</details>

<br/>

### **Ensure the Following VS Code Extensions Are Installed**

- **ESLint**
- **Prettier - Code formatter**

<br/>

### **Run Linting and Formatting**

Run the following commands to lint and format your code:

```sh
npm run lint
```

```sh
npm run format
```

Resolve all warnings and errors, then proceed.

<br/><br/>

<!-- === === === === === -->
<!-- === === === === === -->
<!-- === === === === === -->

# 4. Install and Configure Dependencies

For validation and documentation

```sh
npm install @nestjs/swagger class-validator class-transformer
```

```sh
npm install @nestjs/config
```

```sh
npm install @nestjs/mongoose mongoose
```

Add mongoose module to app

# 5. Set Up the Project

## Swagger (optional)

<details><summary>Settings</summary>
<dl><dd>

<!-- ```sh
npm install @nestjs/swagger
``` -->

```sh
npm install @nestjs/swagger class-validator class-transformer
```

Once the installation process is complete, open the `main.ts` file and initialize Swagger using the `SwaggerModule` class [(docs)](https://docs.nestjs.com/openapi/introduction#bootstrap).

<details>
<summary><strong>src/main.ts</strong></summary>

```ts
import { DocumentBuilder, OpenAPIObject, SwaggerModule } from "@nestjs/swagger";
import { writeFileSync } from "node:fs";
import { join } from "node:path";

// ...
async function bootstrap(): Promise<void> {
  // ...
  const config = new DocumentBuilder()
    .setTitle("Application Title")
    .setDescription("The Application API Description")
    .setVersion("1.0")
    .addTag("app")
    .build();
  const documentFactory = (): OpenAPIObject =>
    SwaggerModule.createDocument(app, config);

  // Setup SwaggerUI
  SwaggerModule.setup("api", app, documentFactory);

  // Save OpenAPI File
  const outputPath = join(process.cwd(), "swagger-spec.json");
  writeFileSync(outputPath, JSON.stringify(documentFactory(), null, 2));
  // ...
}
```

</details>

<br/>

**Adjust `Swagger`**

> _A quick note:_
> Usually, in order to make the class properties visible to the `SwaggerModule`, you need to annotate them with the `@ApiProperty()` decorator [(docs)](https://docs.nestjs.com/openapi/types-and-parameters#types-and-parameters).

Alternatively, you can use the appropriate plugin, as described below. [(Swagger/CLI plugin)](https://docs.nestjs.com/openapi/cli-plugin) [(docs1)](https://www.prisma.io/blog/nestjs-prisma-relational-data-7D056s1kOabc#define-the-user-entity-and-dto-classes) [(docs2)](https://medium.com/@daiki01240/how-to-leverage-swagger-and-class-validator-in-nestjs-api-documentation-and-exporting-type-7577da98768d).

<!-- Before proceeding, make sure these packages are installed. -->

<!-- {
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/swagger"]
  }
} -->

To [enable]() the plugin, open `nest-cli.json` and add the following plugins configuration. You can also use the options property to customize the behavior of the plugin.

<details><summary><strong>nest-cli.json</strong></summary>

```JavaScript
{
// ...
  "compilerOptions": {
	// ...
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "classValidatorShim": true,
          "introspectComments": true,
          "skipAutoHttpCode": true
        }
      }
    ]
	// ...
  }
}
```

</details>

<br/>

The NestJS OpenAPI (Swagger) CLI plugin will automatically:

- Annotate DTO properties with `@ApiProperty` and set `required`, `type`, and `default`.
- Apply validation rules from `class-validator` (if `classValidatorShim` is enabled).
- Add response decorators to endpoints with proper status and types.
- Generate descriptions and examples from comments (if `introspectComments` is enabled).
- Generate and update Swagger (OpenAPI) documentation for your project.

</dd></dl>
</details>

# 6. Create and Set Up the Feature

    🏁 From now on, adding a new feature will involve pretty much the same set of steps. 🏁

### Generate a Feature (Module/Service/Controller/DTOs)

### Register the Prisma Service in the `providers` Array of the Feature Module

### Inject the Prisma Service into the Feature Service

### Add Decorators to DTOs

### Add `ValidationPipe` (+Transformation)

### Add `Better Auth` Authentication Guard

### Create and Add Role Access Guard

# Example resulting files are listed below:
