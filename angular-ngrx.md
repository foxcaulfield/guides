#### Init a project

```
ng new project-name
```

#### Install deps
```
ng add @ngrx/store 
ng add @ngrx/effects 

npm i uuid
npm i -D @types/uuid

npm i immer
npm i ngrx-immer
```

#### Create a component

```
ng g component posts/posts-wrapper
```

#### Bind a route

```
...
export const routes: Routes = [
    {path: "posts", component: PostsPageComponent}
];
...
```
#### Create enviroments

#### Create a service

#### Provide a http client in the configuration
```
    provideHttpClient(),
```

#### Complete the flow

Create folder for a feature ("posts" for example).
  Inside it create two folders: store and service.
    Inside store folder create files: actions, reducer, model, effects.
    Inside service folder create angular service.

Flow of programming:
- Add a method in a service class
- Add an action with two siblings (success, failure)
- Add an effect
- Add a reducer with selectors
- Provide store and provide effects in a app config

#### Set up the component

- Inject the store
- Add observables
- Dispatch a load action
```
  private store: Store = inject(Store);

    public constructor() {
        this.store.dispatch(postsActions.load());
    }

    public errorText$: Observable<string | null> = this.store.select(
        postsFeature.selectError
    );

    public isLoading$: Observable<boolean> = this.store.select(
        postsFeature.selectIsLoading
    );

    public items$: Observable<PostsModel[]> = this.store.select(
        postsFeature.selectItems
    );
```

