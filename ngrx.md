- Start an Angular project

```shell
ng new project-name
ng new test_06__ngrx --inline-style=true --inline-template=true --interactive=true --skip-tests=true
```

- Install **store** and **effects**

```shell
ng add @ngrx/store
ng add @ngtx/effecs

npm i uuid
npm i -D @types/uuid

npm i immer
npm i ngrx-immer
```

## Way 1

- Create blanks

```
src
    cafe-menu
        models
            dish.model.ts
        store
            cafe-menu.actions.ts
            cafe-menu.effects.ts
            cafe-menu.reducer.ts
            cafe-menu.selectors.ts
    app-state.interface.ts
```

- Create a model (a feed post, for example)

```ts
export interface Dish {
  id: string;
  name: string;
  price: number;
}
```

- Create a feature state (in a reducer file or in a new file)

```ts
import { Dish } from "../models/dish.model";

export interface CafeMenuState {
  dishes: Array<Dish>;
}
export const initialState: CafeMenuState = {
  dishes: [
    {
      id: "initial",
      name: "Water",
      price: 1.0,
    },
  ],
};
```

- Create an app interface

```ts
import { CafeMenuState } from "./cafe-menu/store/cafe-menu.reducer";

export interface AppStateInterface {
  cafeMenu: CafeMenuState;
}
```

- Create actions with _createAction_

```ts
import { createAction, props } from "@ngrx/store";
import { Dish } from "../models/dish.model";

export const createDish = createAction(
  "[Food] Create dish",
  props<{ dish: Dish }>()
);

export const removeDish = createAction(
  "[Food] Remove dish",
  props<{ id: number }>()
);
```

- Create a reducer

```ts
import { createReducer, on } from "@ngrx/store";
import { Dish } from "../models/dish.model";
import * as CafeMenuActions from "./cafe-menu.actions";
import { v4 } from "uuid";
// import * as uuid from 'uuid';

export interface CafeMenuState {
  dishes: Array<Dish>;
}
export const initialState: CafeMenuState = {
  dishes: [
    {
      id: "initial",
      name: "Water",
      price: 1.0,
    },
  ],
};

export const CafeMenuReducer = createReducer(
  initialState,
  on(CafeMenuActions.createDish, (state, action) => ({
    ...state,
    dishes: [...state.dishes, { ...action.dish, id: v4() }],
  })),
  on(CafeMenuActions.removeDish, (state, action) => {
    return {
      ...state,
      dishes: state.dishes.filter((dish) => dish.id !== action.id),
    };
  })
);
```

- Register the reducer

```ts
bootstrapApplication(AppComponent, {
  providers: [
    provideStore(),
    provideState({ name: "game", reducer: scoreboardReducer }),
    // provideState(booksFeature)
  ],
});
```

- Create selectors

```ts
import { createSelector } from "@ngrx/store";
import { AppStateInterface } from "../../app-state.interface";

export const selectFeature = (state: AppStateInterface) => state.cafeMenu;

export const selectDishes = createSelector(
  selectFeature,
  (state) => state.dishes
);

export const selectDishesCount = createSelector(
  selectFeature,
  (state) => state.dishes.length
);
```

- Implement observables in a component

```ts
import { Component } from "@angular/core";
import { Store, select } from "@ngrx/store";
import { Observable } from "rxjs";
import { selectDishes, selectDishesCount } from "./store/cafe-menu.selectors";
import { AppStateInterface } from "../app-state.interface";
import { CommonModule } from "@angular/common";
import * as CafeMenuActions from "./store/cafe-menu.actions";
import { generate, count } from "random-words";
// import { removeDish } from './store/cafe-menu.actions';
@Component({
  selector: "app-cafe-menu",
  standalone: true,
  imports: [CommonModule],
  template: `
    <p>cafe-menu works!{{ count$ | async }}</p>
    <button (click)="addDish()">add</button>
    @for (dish of dishes$ | async; track dish.id) {
    <h5>Dish</h5>
    <div>
      {{ dish.name }}: {{ dish.price }}
      <button (click)="deleteDish(dish.id)">remove</button>
    </div>
    <hr />
    }
  `,
  styleUrl: "./cafe-menu.component.scss",
})
export class CafeMenuComponent {
  count$ = this.store.select(selectDishesCount);
  dishes$ = this.store.pipe(select(selectDishes));

  constructor(private store: Store<AppStateInterface>) {}

  addDish() {
    this.store.dispatch(
      CafeMenuActions.createDish({
        dish: { name: <string>generate(1), price: 2 },
      })
    );
  }

  deleteDish(id: string) {
    this.store.dispatch(CafeMenuActions.removeDish({ id }));
  }
}
```

## Way 2

- Create blanks

```
├── app
│   ├── app.component.ts
│   ├── app.config.ts
│   ├── app.routes.ts
│   ├── app-state.interface.ts
│   └── todo-list
│       ├── components
│       ├── models
│       │   ├── todo-item.model.ts
│       │   └── TodoListFeatureName.ts
│       ├── store
│       │   ├── todo-list.actions.ts
│       │   └── todo-list.reducer.ts
│       └── todo-list.component.ts
├── index.html
├── main.ts
└── styles.scss
```

- Create **todo-item.model.ts**

```ts
export interface ITodoItem {
  id: string;
  content: string;
  isDone: boolean;
  createdAt: Date;
}
```

- Create **TodoListFeatureName.ts**

```ts
export const TodoListFeatureName = "todoList";
```

- Create **todo-list.actions.ts**

```ts
import { createActionGroup, props } from "@ngrx/store";
import { ITodoItem } from "../models/todo-item.model";
import { TodoListFeatureName } from "../models/TodoListFeatureName";
import { TodoListFeature } from "./todo-list.reducer";

export const todoListActions = createActionGroup({
  source: TodoListFeatureName,
  events: {
    addTodoItem: props<{ item: Omit<ITodoItem, "id"> }>(),
    removeTodoItem: props<{ itemId: string }>(),
    updateTodoItem: props<{ item: ITodoItem; itemId: string }>(),
  },
});
```

- Create **todo-list.reducer.ts**

```ts
import { createFeature, createReducer, on } from "@ngrx/store";
import { ITodoItem } from "../models/todo-item.model";
import { TodoListFeatureName } from "../models/TodoListFeatureName";
import { todoListActions } from "./todo-list.actions";
import { immerOn } from "ngrx-immer/store";
import * as uuid from "uuid";

export interface ITodoListState {
  items: Array<ITodoItem>;
}

export const initialState: ITodoListState = {
  items: [],
};

export const TodoListFeature = createFeature({
  name: TodoListFeatureName,
  reducer: createReducer(
    initialState,
    immerOn(todoListActions.addTodoItem, (state, action) =>
      state.items.push({ ...action.item, id: uuid.v4() })
    ),
    immerOn(todoListActions.removeTodoItem, (state, { itemId }) => {
      const index = state.items.findIndex((item) => item.id === itemId);
      if (index !== -1) state.items.splice(index, 1);
    }),
    immerOn(todoListActions.updateTodoItem, (state, { itemId, item }) => {
      const index = state.items.findIndex((item) => item.id === itemId);
      if (index !== -1) {
        state.items[index] = { ...state.items[index], ...item };
      }
    })
  ),
});

// export const { name, reducer, selectTodoListState, selectItems } =
//   TodoListFeature;
```

- Create **todo-list.component.ts**

```ts
// Notrhing special
import { Component } from "@angular/core";
import { FormControl, ReactiveFormsModule } from "@angular/forms";
@Component({
  selector: "app-todo-list",
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <p>
      todo-list works!
      <input type="text" [formControl]="todoInputControl" />
    </p>
  `,
  styles: ``,
})
export class TodoListComponent {
  public todoInputControl = new FormControl("");
}
```

- Create **app-state.interface.ts**

```ts
import {
  ITodoListState,
  TodoListFeature,
} from "./todo-list/store/todo-list.reducer";

export const IAppState = {
  [TodoListFeature.name]: TodoListFeature.reducer,
};
```

- Create **app.config.ts**

```ts
import { ApplicationConfig, provideZoneChangeDetection } from "@angular/core";
import { provideRouter } from "@angular/router";

import { routes } from "./app.routes";
import { provideState, provideStore } from "@ngrx/store";
import { provideEffects } from "@ngrx/effects";
import { TodoListFeature } from "./todo-list/store/todo-list.reducer";

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideStore(),
    provideState(TodoListFeature),
    provideEffects(),
  ],
};
```
