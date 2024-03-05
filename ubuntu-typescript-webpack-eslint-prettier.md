This guide provides step-by-step instructions on how to set up a new project from scratch using Node.js, Webpack, and TypeScript. We'll also configure ESLint and Prettier for code linting and formatting, and set up optional Webpack plugins and Sass for advanced project needs.

1. Install Node.js
```
sudo apt install nodejs
```

2. Navigate to your project folder

3. Create a new project
```
npm init -y
```

4. Install Webpack
```
npm install --save-dev webpack webpack-cli webpack-dev-server
```

5. Install TypeScript
```
npm install --save-dev typescript ts-loader ts-node
```

6. Initialize a TypeScript project and creates a tsconfig.json file
```
tsc --init
```

7. Install Prettier and ESLint along with their plugins
```
npm install --save-dev eslint prettier eslint-plugin-prettier eslint-config-prettier
```

8. Install TypeScript types and ESLint parser
```
npm install --save-dev @types/node @types/webpack @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

9. Install Webpack plugins (optional)
```
npm install --save-dev compression-webpack-plugin copy-webpack-plugin html-webpack-plugin
```

10. Install Sass and its loader (optional)
```
npm install --save-dev sass style-loader css-loader sass-loader
```

11. Create configutation files
```
touch webpack.config.ts .eslintrc .eslintignore .prettierrc
```

12. Fill in the configuration files

13. Write scripts in a package.json

Remember to configure your webpack.config.ts, .eslintrc, .eslintignore, and .prettierrc files according to your project requirements. The scripts in package.json will be used to build and run your project.

Here's a single command that includes all the steps:
```
sudo apt install nodejs && \
mkdir my_project && \
cd my_project && \
npm init -y && \
npm install --save-dev webpack webpack-cli webpack-dev-server && \
npm install --save-dev typescript ts-loader ts-node && \
tsc --init && \
npm install --save-dev eslint prettier eslint-plugin-prettier eslint-config-prettier && \
npm install --save-dev @types/node @types/webpack @typescript-eslint/eslint-plugin @typescript-eslint/parser && \
npm install --save-dev compression-webpack-plugin copy-webpack-plugin html-webpack-plugin && \
npm install --save-dev sass style-loader css-loader sass-loader && \
touch webpack.config.ts .eslintrc .eslintignore .prettierrc && \
echo -e "module.exports = {\n  // your webpack config here\n}" > webpack.config.ts && \
echo -e "{\n  // your eslint config here\n}" > .eslintrc && \
echo -e "{\n  // your prettier config here\n}" > .prettierrc && \
echo '{
  "compilerOptions": {
    "target": "ES2020", /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */
    "module": "commonjs", /* Specify what module code is generated. */
    "strict": true, /* Enable all strict type-checking options. */

    "skipLibCheck": false, /* Skip type checking all .d.ts files. */
    "esModuleInterop": false, /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */
    "forceConsistentCasingInFileNames": false, /* Ensure that casing is correct in imports. */
    
    // Strict Checks
    "alwaysStrict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictPropertyInitialization": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,
    "strictBindCallApply": true,
    "noPropertyAccessFromIndexSignature": true,
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
    "noStrictGenericChecks": false,
  }
}' > tsconfig.json
```
