To enforce explicit return type definitions for functions and methods in TypeScript, add the following rule to your ESLint config:

```JavaScript
// ...existing code...
{
    rules: {
        // ...existing rules...
       "@typescript-eslint/explicit-function-return-type":
				[
					"error",
					{
						allowEpressions: false,
						allowTypedFunctionExpressions: false,
						allowHigherOrderFunctions: false,
						allowDirectConstAssertionInArrowFunctions: false,
						allowConciseArrowFunctionExpressionsStartingWithVoid: false,
					},
				],
    },
},
// ...existing code...
```
