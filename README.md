# ngrx-ducks: A proposal for the ducks bundles in Angular with @ngrx (v2)

The first version of this document was based on the original [ducks](https://github.com/erikras/ducks-modular-redux) proposal, and the [re-ducks](https://github.com/alexnm/re-ducks) extension, and was made thinking first about Angular, and the Angular Style Guide. You can find the that version in [v1 branch](/michaeljota/ngrx-ducks/tree/v1).

---

## What is a duck?

A duck is a module proposal to bundle redux related code, reducer, actions, and actions types. The original proposal follow this simple rules:

- MUST export default a function called reducer().
- MUST export its action creators as functions.
- MUST have action types in the form. `npm-module-or-app/reducer/ACTION_TYPE`
- MAY export its action types as `UPPER_SNAKE_CASE`, if an external reducer needs to listen for them, or if it is a published reusable library.

Re-ducks extension is a proposal to work with a feature folder structure. Instead of using a file, you may use a folder, with an index file as a barrel for the duck, but exporting nearly the same.

The current schematics of ngrx allows to create all the members of a _duck_ according to the re-ducks extension. However, the default behavior doesn't provide an standard way to organize them, but instead the only option do so, organize them by folder, instead of by feature.

The previous version of this document have something called "Dispatcher". However, the concept is like the "Facade", introduced by [Thomas Burleson](https://twitter.com/ThomasBurleson) in [NgRx + Facades: Better State Management](https://medium.com/@thomasburlesonIA/ngrx-facades-better-state-management-82a04b9a1e39). In both, the intention is to hide the store implementation and use a service from the component to access it instead.

---

## Style Guide

The follow will extend and consider the Angular Style Guide as the main document. It will use additional vocabulary.

- _Extension_ Notes that extend the rule adapting it to use with `ngrx`.

## Naming

### Separate file names with dots and dashes

##### Extension: [Style 02-02](https://angular.io/guide/styleguide#style-02-02)

_Consider_ use conventional type names for store related code, including `actions`, `effect`, `reducer`, `selectors` and `facade`.

_Why?_ Those type names represents a duck.

### Service names

##### Extension: [Style 02-04](https://angular.io/guide/styleguide#style-02-04)

_Avoid_ naming files for store related services with the `.service` suffix.

_Why?_ Consistency with the duck naming, and the Style 02-02 [*extension*]

### Angular _NgModule_ names

##### Extension: [Style 02-12](https://angular.io/guide/styleguide#style-02-12)

_Do_ suffix the name a Store module with `StoreModule`.

_Do_ end the file name of a Store module with `-store.module`.

### Interfaces

##### Extension [Style 03-03](https://angular.io/guide/styleguide#style-03-03)

_Consider_ declaring the state of the store and reducers as a class.

_Consider_ using the class properties to initialize the states.

_Do_ initialize the state creating a new instance of the class inside a new object using spread operator.

_Why?_ Using spread operator ensures that you are using a serializable object.

_Why?_ Initialize all the state properties.

_Why?_ Reduce the boilerplate code.

```typescript
// Avoid
export interface AppState {
  count: number;
}

export const initialState: AppState = {
  count: 0,
};
```

```typescript
// Consider
export class AppState {
  count: number = 0;
}
```

```typescript
// Avoid
/* feature/component/component.reducer.ts */
export interface ComponentState {
  count: number;
}

export const initialComponentState: ComponentState = {
  count: 0,
};
/* feature/feature-store.module.ts */
. . .
import {
  ComponentState,
  initialComponentState,
} from './component/component.reducer';
. . .
interface FeatureState {
  . . .
  component: ComponentState;
}

const initialFeatureState: FeatureState = {
  . . .
  component: initialComponentState,
};
```

```typescript
// Consider
/* feature/component/component.reducer.ts */
export class ComponentState {
  counter: number = 0;
}
/* feature/feature-store.module.ts */
. . .
import { ComponentState } from './component/component.reducer';
. . .
class FeatureState {
  component = new ComponentState();
}
```

```typescript
// Do
@NgModule({
  imports: [
    StoreModule.forRoot(reducers, {
      metaReducers,
      initialState: { ...new AppState() }, // A new instance inside of a spread operator.
    }),
    StoreDevtoolsModule.instrument(),
    EffectsModule.forRoot([]),
  ],
  declarations: [],
})
export class AppStoreModule {}
```

> Currently the ngrx schematics produces reducers with the state as an interface, and the initial state as an object. You could however replace the produced code with a class, but do what works for you, and be consistent with that.

### Overall structural guidelines

##### Extension [Style 04-06](https://angular.io/guide/styleguide#style-04-06)

_Consider_ grouping the ducks files the same way you are grouping your components.

_Consider_ grouping the duck files in the component folder, only if the ducks are component related, _or_ you are grouping your components in top-level folders aside from feature module file declaration.

_Consider_ grouping the duck files in feature folder, inside a ducks container folder aside from feature module file declaration, only if you are grouping your components in a components container folder.

_Why?_ The Angular Style guide suggest organize the components in top-level folders aside from the feature module file declaration.

_Why?_ The Angular Style guide also suggest no more than seven files in each folder.

```
~ Consider Option 1~
feature
 |- feature.module.ts
 |- component
    |- component.component.ts|html|css|spec.ts
    |- component.reducer|actions|effects|selectors.ts
```

```
~ Consider Option 2 ~
feature
 |- feature.module.ts
 |- components
 |  |- my-component
 |  |  |- my-component.component.ts|html|css|spec.ts
 |- ducks
 |  |- my-duck
 |  |  |- my-duck.reducer|actions|effects|selectors|facade.ts
```

### Feature modules

##### Extension [Style 04-09](https://angular.io/guide/styleguide#style-04-09)

_Consider_ creating a feature store module.

_No additional whys besides the rules have_

```typescript
// Avoid
/* feature/feature.reducer.ts */
export class FeatureState {
  duck = new DuckState();
}

export const featureReducer = createReducer({ ...new FeatureState() }, {});

/* app.module.ts */
import { NgModule } from "@angular/core";
import { ActionReducerMap, MetaReducer, StoreModule } from "@ngrx/store";
import { EffectsModule } from "@ngrx/effects";
import { StoreDevtoolsModule } from "@ngrx/store-devtools";
import { FeatureState, featureReducer } from "./feature/feature.reducer";

export class AppState {
  feature = new FeatureState();
}

const reducers: ActionReducerMap<AppState> = {
  feature: featureReducer,
};

const metaReducers: MetaReducer<AppState>[] = [];

@NgModule({
  imports: [
    StoreModule.forRoot(reducers, {
      metaReducers,
      initialState: { ...new AppState() },
    }),
    StoreDevtoolsModule.instrument(),
    EffectsModule.forRoot([]),
  ],
  declarations: [],
})
export class AppStoreModule {}
```

```typescript
// Consider
/* feature/feature-store.ts */
import { NgModule } from "@angular/core";
import {
  ActionReducerMap,
  createFeatureSelector,
  MetaReducer,
  StoreModule,
} from "@ngrx/store";
import { EffectsModule } from "@ngrx/effects";

import {
  ComponentState,
  componentReducer,
} from "./component/component.reducer";

import { ComponentEffects } from "./component/component.effects";

class FeatureState {
  component = new ComponentState();
}

const featureReducers: ActionReducerMap<FeatureState> = {
  component: componentReducer,
};

export const heroesState = createFeatureSelector<FeatureState>("feature");

const metaReducers: MetaReducer<FeatureState>[] = [];

@NgModule({
  imports: [
    StoreModule.forFeature("feature", featureReducers, {
      metaReducers,
      initialState: { ...new FeatureState() },
    }),
    EffectsModule.forFeature([ComponentEffects]),
  ],
})
export class HeroesStoreModule {}
```

> Currently the ngrx schematics produces a reducers file, with the state and reduce definition, and setup the store module within the main module you declare. You could however replace the produced code with the recommendations, but do what works for you, and be consistent with that.

### Shared feature module

##### Extension [Style 04-10](https://angular.io/guide/styleguide#style-04-10)

_Consider_ putting the duck files you must use in the application wide here.

_Why?_ Using shared modules to shared ducks makes sense.

### Delegate complex component logic to services

##### Extension [Style 05-15](https://angular.io/guide/styleguide#style-05-15)

_Consider_ using store to manage the state of the component.

_Why?_ Allows you to reduce the number of the dependencies.

_Why?_ Components are easier to test, as the only show a given state.

```typescript
// Avoid
import { Component, OnInit } from "@angular/core";

import { Model } from "~app/model";
import { MyServiceService } from "./my-service.service";

@Component({
  selector: "app-component",
  templateUrl: "./my-component.component.html",
})
export class MyComponent implements OnInit {
  list: Model[];
  constructor(private service: MyServiceService) {}

  ngOnInit() {
    this.list = [];
    this.service.getAll().subscribe((list) => (this.list = list));
  }
}
```

```typescript
// Consider
import { Component, OnInit } from "@angular/core";
import { Store } from "@ngrx/store";

import { Model } from "~app/model";

import { selectModelList } from "./component.selectors";

@Component({
  selector: "app-component",
  templateUrl: "./component.component.html",
})
export class Component implements OnInit {
  list: Observable<Model[]>;
  constructor(private readonly store: Store) {}

  ngOnInit() {
    this.list = this.store.select(selectModelList);
  }
}
```

## Appendix

Useful tips for Angular applications with `@ngrx` platform.

> The order of the styles has been updated to introduce better the relationship between each feature.

### Reducers

##### Style NGRX-01

_Consider_ using `createReducer` function to create the reducer.

_Consider_ placing the reducer inside the store module configuration.

_Consider_ declaring the initial state of the reducer as a class.

_Consider_ initializing the state of the module reducer with properties calling a new instance of the component reducer states.

_Why?_ Module reducers should only import the components reducers and initialize them.

_Why?_ Class allows to initialize the state of the reducer in a cleaner way.

```typescript
// Consider
/* duck.reducer.ts */
import { ActionReducer } from "@ngrx/store";

import { Model } from "~app/model";

export class DuckState {
  public readonly model = new Model();
}

export const initialState = { ...new DuckState() };

export const duckReducer = createReducer(initialState);
```

> Currently the ngrx schematics produces a reducers file, with the state and reduce definition, and setup the store module within the main module you declare. You could however replace the produced code with the recommendations, but do what works for you, and be consistent with that.

### Actions and Actions Types

##### Style NGRX-02

_Consider_ using the `createAction` function to create action functions.

_Consider_ name your actions with a regular and predictable pattern.

_Consider_ use human readable names for the exported members.

_Why?_ Using the `createAction` allow to take advantage of ngrx schematics.

_Why?_ A pattern such as `[Duck/UI] Action` make easier to locate related code and debugging.

```typescript
// Consider
/* component.actions.ts */
import { createAction, props } from "@ngrx/store";

export const uiAction = createAction("[Duck/UI] UI Action", props<Props>());

export const apiAction = createAction("[Duck/API] API Action", props<Props>());
```

### Selectors

##### Style NGRX-03

_Consider_ using the `createFeatureSelector` function to select the feature state.

_Consider_ exporting the feature selector from the reducer configuration.

_Consider_ using the `createSelector` function to select additional parts of the feature state.

_Consider_ grouping all your selectors in a selectors file.

_Why?_ The `createFeatureSelector` function takes a generic type argument, and a string argument to select the piece of state, reducing the chances of a circular dependency.

_Why?_ Selectors are a type safe approach to return an observable of the feature state.

_Why?_ The `createSelector` function is a memoized function, resulting in a better performance when selecting the same piece multiple times, allowing to use`async` operator multiple times.

```typescript
// Consider
/* feature-store.module.ts */
import { createFeatureSelector } from "@ngrx/store";

export const selectFeatureState = createFeatureSelector("feature");

/* duck.selectors.ts */
import { createSelector } from "@ngrx/store";
import { selectFeatureState } from "somewhere/feature-store.module";

export const selectList = createSelector(
  selectFeatureState,
  (state) => state.list
);
```

### Facade

##### Style NGRX-04

_Consider_ creating a facade service to access the store.

_Consider_ use the same name for actions an selectors.

_Consider_ exporting the facade service from a facade file.

_Consider_ provide the facade in root.

_Consider_ using memoize functions methods.

_Why?_ A dispatcher service allows you to take advantage of Angular Dependency Injection.

_Why?_ You could change your actions, without modifying your dispatcher.

_Why?_ Allows you to use the dispatcher in your components, and in your effects services.

```typescript
// Consider
/* component.dispatcher.ts */
import { Injectable } from "@angular/core";

import { Model } from "~app/model";

import { loadList } from "./component.actions";
import { selectList } from "./component.selectors";

@Injectable()
export class DuckFacade {
  list$ = this.store.select(selectList);

  constructor(private readonly store: Store) {}

  loadList() {
    this.store.dispatch(loadList());
  }
}
```

### Effects

##### Style NGRX-03

_Consider_ using `createEffect` function to create an effect configuration.

_Consider_ using effects to integrate the results of all asynchronous operations with the state of the application.

_Avoid_ importing directly the effects into the application.

_Why?_ The `createEffect` function returns an observable and it is analyzed by the ngrx in runtime to ensure it returns a correct value.

_Why?_ The `createEffect` function doesn't need additional type annotations in runtime.

_Why?_ Using effects allows you practice the separation of concerns.

_Why?_ Effects does not need to be imported into the application. Only into the Effects Module configuration.

```typescript
// Consider
/* duck.effects.ts */
import { Inject, Injectable } from "@angular/core";
import { Actions, createEffect, ofType } from "@ngrx/effects";
import { loadList, loadListSuccess, loadListFailure } from "./duck.actions";
import { concatMap } from "rxjs/operators";
import { MyServiceService } from "~app/services/my-service.service";

@Injectable()
export class MetaEffects {
  setMetaTags$ = createEffect(() => {
    return this.actions$.pipe(
      ofType(loadList),
      concatMap(
        this.myService.get().pipe(
          map((list) => loadListSuccess({ list })),
          catchError((error) => of(loadListFailure({ error })))
        )
      )
    );
  });

  constructor(
    private readonly actions$: Actions,
    private readonly myService: MyServiceService
  ) {}
}
```

### Entities

##### Style NGRX-06

_Consider_ using the `@ngrx/entity` module to create and handle entities.

_Consider_ placing the entity adapter inside the reducer file.

_Consider_ exporting the selectors generated from the entity adapter as one const value.

_Why_ The adapter is a function helper to manage the entity state.

```typescript
// Consider
/* component.reducer.ts */
import { createEntityAdapter, EntityState, Dictionary } from "@ngrx/entity";

import { Model } from "~app/model";

const modelAdapter = createEntityAdapter<Model>();

export const modelSelectors = modelAdapter.getSelectors();

export class ComponentState implements EntityState<Model> {
  public readonly ids: number[] | string[] = [];
  public readonly entities: Dictionary<Model> = {};
  public readonly anotherProperty: boolean;
}
```

### Stores

##### Style NGRX-07

_Consider_ use an Angular module to setup the Store.

_Consider_ setup and export the state from the module.

_Consider_ setup and export the selector from the module.

_Why?_ Modules are a common practice in Angular, and a recommended way to keep your modules cleaner.

_Why?_ The store state, reducers and selectors will be easy to find after scaling.

```typescript
// Avoid
/* counter.ts */
import { Action } from "@ngrx/store";

export const INCREMENT = "INCREMENT";
export const DECREMENT = "DECREMENT";
export const RESET = "RESET";

export function counterReducer(state: number = 0, action: Action) {
  switch (action.type) {
    case INCREMENT:
      return state + 1;

    case DECREMENT:
      return state - 1;

    case RESET:
      return 0;

    default:
      return state;
  }
}

/* app.module.ts */
import { NgModule } from "@angular/core";
import { StoreModule } from "@ngrx/store";
import { counterReducer } from "./counter";

@NgModule({
  imports: [BrowserModule, StoreModule.forRoot({ count: counterReducer })],
})
export class AppModule {}
```

```typescript
// Consider
/* app-store.module.ts */
import { NgModule } from "@angular/core";
import { ActionReducerMap, MetaReducer, StoreModule } from "@ngrx/store";
import { EffectsModule } from "@ngrx/effects";
import { StoreDevtoolsModule } from "@ngrx/store-devtools";

export class AppState {}

const reducers: ActionReducerMap<AppState> = {};

const metaReducers: MetaReducer<AppState>[] = [];

@NgModule({
  imports: [
    StoreModule.forRoot(reducers, {
      metaReducers,
      initialState: { ...new AppState() },
    }),
    StoreDevtoolsModule.instrument(),
    EffectsModule.forRoot([]),
  ],
})
export class AppStoreModule {}

/* app.module.ts */
import { NgModule } from "@angular/core";
import { AppStoreModule } from "./app-store.module";

@NgModule({
  imports: [BrowserModule, AppStoreModule],
})
export class AppModule {}
```

---

Additional Info:

- If you group the ducks along the components, the current amount of files is up-to the recommended by the Angular Style Guide.
- Use the ngrx schematics whenever you can, as they will configure itself and will setup some test files for you.
