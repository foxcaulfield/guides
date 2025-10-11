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
		},
{
			"type": "node",
			"request": "attach",
			"name": "Attach Debug NestJS",
			"port": 9229
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
  "trailingComma": "es5",
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
					singleQuote: false,
					trailingComma: "es5",
					useTabs: true,
					tabWidth: 4,
					printWidth: 120,
					bracketSameLine: false,
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

			// "function-call-argument-newline": ["error", "consistent"],
			// "function-paren-newline": ["error", "multiline"],
			// "object-curly-newline": ["error", { multiline: true, consistent: true }],
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
