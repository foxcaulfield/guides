1. install "Prettier" VS Code plugin
2. install "ESLint" VS Code plugin

---

#### `settings.json`
```json
{
    "editor.insertSpaces": false,
    "javascript.preferences.quoteStyle": "double",
    "typescript.preferences.quoteStyle": "double",
    "editor.detectIndentation": false
}
```

---

#### `.prettierrc`
```json
{
	"singleQuote": false,
	"trailingComma": "all",
	"useTabs": true,
	"tabWidth": 4,
	"printWidth": 60
}
```

---

#### `eslint.config.mjs`

```JavaScript
// ...existing code...
rules: {
			"@typescript-eslint/no-explicit-any": "off",
			"@typescript-eslint/no-floating-promises":
				"warn",
			"@typescript-eslint/no-unsafe-argument": "warn",
			"prettier/prettier": [
				"error",
				{
					// endOfLine: "auto",
					// printWidth: 80,
					// trailingComma: "es5",
					// semi: false,
					doubleQuote: true,
					// jsxSingleQuote: true,
					singleQuote: false,
					useTabs: true,
					// tabWidth: 4,
				},
			],
		},
// ...existing code...
```
