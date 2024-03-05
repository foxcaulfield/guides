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

mkdir dist && \
mkdir --parents src/ts && \

touch webpack.config.ts .eslintrc .eslintignore .prettierrc && \
touch src/ts/main.ts && \
touch src/index.html && \

echo -e 'import path from "path";
import webpack from "webpack";
import HtmlWebpackPlugin from "html-webpack-plugin";
import CompressionPlugin from "compression-webpack-plugin";
// import FaviconsWebpackPlugin from "favicons-webpack-plugin";
import CopyWebpackPlugin from "copy-webpack-plugin";

import "webpack-dev-server";

const config: webpack.Configuration = {
	devtool: false,
	performance: {
		maxEntrypointSize: 512000,
		maxAssetSize: 512000,
		hints: false
	},
	mode: "production",
	entry: "./src/ts/main.ts",
	output: {
		path: path.resolve(__dirname, "dist"),
		filename: "[name].bundle.js"
	},
	resolve: {
		extensions: [".ts", ".tsx", ".js"]
	},
	module: {
		rules: [
			{
				test: /\.tsx?$/,
				use: "ts-loader",
				exclude: /node_modules/
			},
			{
				test: /\.s[ac]ss$/i,
				use: [
					// Creates `style` nodes from JS strings
					"style-loader",
					// Translates CSS into CommonJS
					"css-loader",
					// Compiles Sass to CSS
					"sass-loader"
				]
			}
		]
	},
	plugins: [
		new CompressionPlugin(),
		new HtmlWebpackPlugin({
			template: "./src/index.html"
		}),
		new CopyWebpackPlugin({
			patterns: [{ from: path.join(__dirname, "src", "favicon.ico"), to: path.join(__dirname, "dist", "favicon.ico") }]
		})
	],
	devServer: {
		static: {
			directory: path.join(__dirname, "dist")
		},
		compress: true,
		port: 9000,
		hot: true
	},

	optimization: {
		splitChunks: {
			chunks: "async",
			minSize: 20000,
			maxSize: 200000,
			minRemainingSize: 0,
			minChunks: 1,
			maxAsyncRequests: 30,
			maxInitialRequests: 30,
			enforceSizeThreshold: 50000,
			cacheGroups: {
				defaultVendors: {
					test: /[\\\\/]node_modules[\\\\/]/,
					priority: -10,
					reuseExistingChunk: true
				},
				default: {
					minChunks: 2,
					priority: -20,
					reuseExistingChunk: true
				}
			}
		}
	}
};

export default config;
' > webpack.config.ts && \

echo -e '{
    "root": true,
    "parser": "@typescript-eslint/parser",
    "plugins": [
        "@typescript-eslint",
        "prettier"
    ],
    "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/eslint-recommended",
        "plugin:@typescript-eslint/recommended",
        "prettier"
    ],
    "rules": {
        "no-console": 2,
        "prettier/prettier": [
            "error"
        ]
    },
    "env": {
        "node": true
    }
}' > .eslintrc && \

echo -e "node_modules\n dist" > .eslintignore && \

echo -e '{
    "semi": true,
    "trailingComma": "es5",
    "singleQuote": false,
    "printWidth": 120,
    "useTabs": true
}' > .prettierrc && \

echo '{
  "compilerOptions": {
    "target": "ES2020", /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */
    "module": "commonjs", /* Specify what module code is generated. */
    "strict": true, /* Enable all strict type-checking options. */

    "skipLibCheck": false, /* Skip type checking all .d.ts files. */
    "esModuleInterop": true, /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */
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
}' > tsconfig.json && \

echo -e '<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
</body>
</html>' > src/index.html && \

sed  '/scripts/a \\t"build": "webpack",\n\t"serve": "webpack serve --open",' package.json  && \

npm run serve
```
