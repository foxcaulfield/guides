Yes, there is a way to enforce explicit access modifiers (`private`, `public`, `protected`) in TypeScript classes by using a custom ESLint rule or a TSLint rule, depending on your linting setup. While TypeScript itself doesn't have a compiler option to enforce this directly, you can achieve this through linting tools.

Here's how you can set it up using ESLint with the `@typescript-eslint` plugin:

### Step-by-Step Setup for ESLint

1. **Install ESLint and TypeScript Plugin:**
   If you haven't already, install ESLint and the TypeScript plugin for ESLint.

   ```bash
   npm install eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev
   ```

2. **Create or Update Your ESLint Configuration:**
   Create an `.eslintrc.json` file in your project root (if you don't already have one) and configure it to use the TypeScript plugin. Add a rule to enforce explicit access modifiers.

   ```json
   {
     "parser": "@typescript-eslint/parser",
     "parserOptions": {
       "ecmaVersion": 2020,
       "sourceType": "module"
     },
     "plugins": ["@typescript-eslint"],
     "extends": [
       "eslint:recommended",
       "plugin:@typescript-eslint/recommended"
     ],
  rules: {
    '@typescript-eslint/interface-name-prefix': 'error',
    '@typescript-eslint/explicit-function-return-type': 'error',
    '@typescript-eslint/explicit-module-boundary-types': 'error',
    '@typescript-eslint/no-explicit-any': 'error',
    "@typescript-eslint/explicit-member-accessibility": ["error", { "accessibility": "explicit" }],
    "@typescript-eslint/typedef": [
      "error",
      {
        "memberVariableDeclaration": true
      }
    ]
  },
   }
   ```

3. **Run ESLint:**
   You can now run ESLint to check your TypeScript files for explicit member accessibility. 

   ```bash
   npx eslint . --ext .ts
   ```

### Example

Here's an example of how the rule works. Given the following TypeScript class:

```typescript
class Example {
  name: string;  // This will trigger a linting error
  constructor(name: string) {
    this.name = name;
  }

  getName() {  // This will also trigger a linting error
    return this.name;
  }
}
```

After applying the rule, you'll need to specify the access modifiers explicitly:

```typescript
class Example {
  public name: string;
  constructor(name: string) {
    this.name = name;
  }

  public getName(): string {
    return this.name;
  }
}
```

### Summary
By using ESLint with the `@typescript-eslint` plugin, you can enforce the use of explicit access modifiers in your TypeScript classes. This setup helps maintain code consistency and clarity regarding the accessibility of class members.
