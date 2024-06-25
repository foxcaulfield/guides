```
npx create-nx-workspace@latest \
--preset=nest \
--name=nestjs-project \
--appName=backend \
--routing=true \
--docker=true \
--e2eTestRunner=none \
--packageManager=npm \
--skipGit=true \
--ssr=false \
--workspaceType=integrated
```

```
// npm install @nrwl/angular
nx add @nx/angular
```

```
nx generate @nx/angular:application \
--name frontend \
--style scss \
--prefix fse \
--tags type:app,scope:client \
--strict \
--backendProject backend \
--standalone \
--routing \
--e2eTestRunner=none \
--inlineStyle=true \
--inlineTemplate=true \
--skipTests=true \
--ssr=false \
--unitTestRunner=none \
--directory=apps/frontend
```



You can add this to your nx.json and it will do what you want. I agree that it’s frustrating when they change it seemingly at random.
```
"workspaceLayout": {
    "appsDir": "apps",
    "libsDir": "packages"
  }
```

