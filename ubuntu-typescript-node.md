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

5. Install TypeScript
```
npm install --save-dev typescript ts-node nodemon @types/node 
```

npm install --save-dev rimraf

npx tsc --init 
--rootDir src \
--outDir build \
\
--target ES2020 \
--module commonjs \
--strict \
--skipLibCheck false \
--esModuleInterop true \
--forceConsistentCasingInFileNames false \
--alwaysStrict true \
--noImplicitAny true \
--strictNullChecks true \
--strictPropertyInitialization true \
--strictFunctionTypes true \
--noImplicitThis true \
--strictBindCallApply true \
--noPropertyAccessFromIndexSignature true \
--noUncheckedIndexedAccess true \
--noImplicitReturns true \
--noFallthroughCasesInSwitch true \
--noUnusedLocals true \
--noUnusedParameters true \
--allowUnreachableCode false \
--allowUnusedLabels false \
--noImplicitUseStrict false \
--suppressExcessPropertyErrors false \
--suppressImplicitAnyIndexErrors false \
--noStrictGenericChecks false \
\
--resolveJsonModule \
--lib es6 \
--allowJs true \


Add a nodemon.json config.
{
  "watch": ["src"],
  "ext": ".ts,.js",
  "ignore": [],
  "exec": "npx ts-node ./src/index.ts"
  "start:dev": "npx nodemon",
  "build":"rimraf ./build && tsc",
  "start": "npm run build && node build/index.js"
}




```
sudo apt install nodejs && \

sudo npm init -y && \

sudo npm install --save-dev typescript ts-node nodemon @types/node  && \
sudo npm install --save-dev rimraf && \

tsc --init \
--rootDir src \
--outDir build \
--target ES2020 \
--module commonjs \
--strict \
--skipLibCheck false \
--esModuleInterop true \
--forceConsistentCasingInFileNames false \
--alwaysStrict true \
--noImplicitAny true \
--strictNullChecks true \
--strictPropertyInitialization true \
--strictFunctionTypes true \
--noImplicitThis true \
--strictBindCallApply true \
--noPropertyAccessFromIndexSignature true \
--noUncheckedIndexedAccess true \
--noImplicitReturns true \
--noFallthroughCasesInSwitch true \
--noUnusedLocals true \
--noUnusedParameters true \
--allowUnreachableCode false \
--allowUnusedLabels false \
--noImplicitUseStrict false \
--suppressExcessPropertyErrors false \
--suppressImplicitAnyIndexErrors false \
--noStrictGenericChecks false \
--resolveJsonModule \
--lib es6 \
--allowJs true && \

touch nodemon.json && \

echo '{
  "watch": ["src"],
  "ext": ".ts,.js",
  "ignore": [],
  "exec": "npx ts-node ./src/index.ts",
  "start:dev": "npx nodemon",
  "build":"rimraf ./build && tsc",
  "start": "npm run build && node build/index.js"
}' > nodemon.json && \

mkdir src build
```




