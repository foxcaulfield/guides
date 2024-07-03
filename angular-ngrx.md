```
ng new project-name
```

```
ng g component posts/posts-wrapper
```

Create folder for a feature ("posts" for example).
  Inside it create two folders: store and service.
    Inside store folder create files: actions, reducer, model, effects.
    Inside service folder create angular service.


Flow of programming:
  Add a method in a service class
  Add an action with two siblings (success, failure)
  Add an effect
  Provide store and provide effects in a app config
