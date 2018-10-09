# ngrx-ducks: A proposal for the ducks bundles in Angular with @ngrx

This document is based on the original [ducks](https://github.com/erikras/ducks-modular-redux) proposal, and the [re-ducks](https://github.com/alexnm/re-ducks) extension, and was made thinking first about Angular, and the current Angular Style Guide, as such I will try to use the same vocabulary with the same meaning.

---

## What is a duck?

A duck is a module proposal to bundle redux related code, reducer, actions, and actions types. The original proposal follow this simple rules:

* MUST export default a function called reducer().
* MUST export its action creators as functions.
* MUST have action types in the form. `npm-module-or-app/reducer/ACTION_TYPE`
* MAY export its action types as `UPPER_SNAKE_CASE`, if an external reducer needs to listen for them, or if it is a published reusable library.

Re-ducks extension is a proposal to work with a feature folder structure. Instead of using a file, you may use a folder, with an index file as a barrel for the duck, but exporting nearly the same.

However, some of these rules are not usable in Angular, and others can be improved to take full advantage of the Typescript and Angular.

* `default` exports doesn't work with AOT.
* `function` creators does not allow to use Typescript type discrimination. You could create and return an interface to use this, but Angular Style Guide discourage the use of interfaces.
* Action Types can be expressed as Typescript strings `enums`.
* `UPPER_SNAKE_CASE` is discourage by Angular Style Guide.

---
## Style Guide

As I said, I will use Angular Style Guide as the main document. I will use additional vocabulary.

* *Extension* Notes that extend the rule adapting it to use with `ngrx`.

## Naming

### Separate file names with dots and dashes

##### Extension: [Style 02-02](https://angular.io/guide/styleguide#style-02-02)

*Consider* use conventional type names for store related code, including `actions`, `effects`, `reducer`, `selectors` and `dispatcher`.

*Why?* Those type names represents a duck.

### Service names

##### Extension: [Style 02-04](https://angular.io/guide/styleguide#style-02-04)

*Avoid* naming files for store related services with the `.service` suffix.

*Why?* Consistency with the duck naming, and the Style 02-02 [*extension*]

### Angular *NgModule* names

##### Extension: [Style 02-12](https://angular.io/guide/styleguide#style-02-12)

*Do* suffix the name a Store module with `StoreModule`.

*Do* end the file name of a Store module with `-store.module`. 

### Interfaces

##### Extension [Style 03-03](https://angular.io/guide/styleguide#style-03-03)

*Avoid* declaring the states of the store and reducers as an interface.

*Consider* declaring the state of the store and reducers as a class.

*Consider* using the class properties to initialize the states.

*Do* initialize the state creating a new instance of the class inside a new object using spread operator.

*Why?* Using spread operator ensures that you are initializing the state as a POJO.

*Why?* Angular Compiler needs to statically analyze the code.

*Why?* You make sure you initiate all the state properties.

*Why?* Reduce the boilerplate code.

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

### Overall structural guidelines

##### Extension [Style 04-06](https://angular.io/guide/styleguide#style-04-06)

*Consider* putting the duck files in the component folder.

*Why?* Duck files most of the times are used by a single component.

```
~ Avoid ~
feature
 |- feature.module.ts
 |- component
 |  |- component.component.ts|html|css|spec.ts
 |- reducers
    |- component.reducer|actions|effects|selectors.ts
```
```
~ Consider ~
feature
 |- feature.module.ts
 |- component
    |- component.component.ts|html|css|spec.ts
    |- component.reducer|actions|effects|selectors.ts
```

### Feature modules

##### Extension [Style 04-09](https://angular.io/guide/styleguide#style-04-09)

*Consider* creating a feature store module.

*No additional whys besides the rules have*

```typescript
// Avoid
/* feature/feature.reducer.ts */
export class FeatureState {
  component = new ComponentState();
}

export function featureReducer(state) {
  return state;
}
/* app.module.ts */
import { NgModule } from '@angular/core';
import { ActionReducerMap, MetaReducer, StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { FeatureState, featureReducer } from './feature/feature.reducer';

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
import { NgModule } from '@angular/core';
import {
  ActionReducerMap,
  createFeatureSelector,
  MetaReducer,
  StoreModule,
} from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';

import {
  ComponentState,
  componentReducer,
} from './component/component.reducer';

import { ComponentEffects } from './component/component.effects';

class FeatureState {
  component = new ComponentState();
}

const featureReducers: ActionReducerMap<FeatureState> = {
  component: componentReducer,
};

export const heroesState = createFeatureSelector<FeatureState>('feature');

const metaReducers: MetaReducer<FeatureState>[] = [];

@NgModule({
  imports: [
    StoreModule.forFeature('feature', featureReducers, {
      metaReducers,
      initialState: { ...new FeatureState() },
    }),
    EffectsModule.forFeature([ComponentEffects]),
  ],
})
export class HeroesStoreModule {}
```

### Shared feature module

##### Extension [Style 04-10](https://angular.io/guide/styleguide#style-04-10)

*Consider* putting the duck files you must use in the application wide here.

*Why?* Using shared modules to shared ducks makes sense.

### Delegate complex component logic to services

##### Extension [Style 05-15](https://angular.io/guide/styleguide#style-05-15)

*Consider* using store to manage the state of the component.

*Why?* Allows you to reduce the number of the dependencies.

*Why?* Components are easier to test, as the only show a given state.

```typescript
// Avoid
import { Component, OnInit } from '@angular/core';

import { Model, ComponentService } from '../shared';

@Component({
  selector: 'app-component',
  templateUrl: './component.component.html',
})
export class Component implements OnInit {
  list: Model[];
  constructor(private service: Component) {}
  
  ngOnInit() {
    this.list = [];
    this.service.getAll()
      .subscribe(list => this.list = list);
  }
}
```
```typescript
// Consider
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';

import { AppState } from '../../app-store.module';

import { Model } from '../shared';

import { selectList } from './component.selectors';

@Component({
  selector: 'app-component',
  templateUrl: './component.component.html',
})
export class Component implements OnInit {
  list: Observable<Model[]>;
  constructor(private readonly store: Store<AppState>) {}
  
  ngOnInit() {
    this.heroes = this.store.select(selectHeroesList);
  }
}
```

## Appendix

Useful tips for Angular applications with `@ngrx` platform.

### Actions and Actions Types

##### Style NGRX-01

*Consider* creating all the actions as classes.

*Consider* using a type union of all actions classes.

*Consider* using strings enums to declaring the actions types.

*Consider* prefix the value of all enums keys with a unique name.

*Consider* making the unique name related to the component.

*Consider* exporting the actions, the type and the enum of the actions types from an action file.

*Consider* using named properties instead of generic payload

*Why?* Using classes for actions allow to take advantage of the discriminatory types of Typescript.

```typescript
// Consider
/* component.actions.ts */
import { Action } from '@ngrx/store';

import { Model } from './../shared/model';

export enum ComponentActions {
  Load = '[Component] Load',
  LoadSuccess = '[Component] Load Success',
  Save = '[Component] Save',
  SaveSuccess = '[Component] Save Success',
}

export type ComponentActionType =
  | LoadModel
  | LoadModelSuccess
  | SaveModel
  | SaveModelSuccess;

export class LoadModel implements Action {
  public readonly type = ComponentActions.Load;
  constructor(public readonly modelId: string) {}
}

export class LoadModelSuccess implements Action {
  public readonly type = ComponentActions.LoadSuccess;
  constructor(public readonly model: Readonly<Model>) {}
}

export class SaveModel implements Action {
  public readonly type = ComponentActions.Save;
  constructor(public readonly model: Readonly<Model>) {}
}

export class SaveModelSuccess implements Action {
  public readonly type = ComponentActions.SaveSuccess;
}
```

### Dispatcher

##### Style NGRX-02

*Consider* creating a dispatcher service to dispatch actions.

*Consider* use human readable names for the methods in the dispatcher.

*Consider* exporting the dispatcher service from a dispatcher file.

*Consider* registering the dispatcher in the feature module as a provider.

*Consider* using memoize functions methods.

*Why?* A dispatcher service allows you to take advantage of Angular Dependency Injection.

*Why?* You could change your actions, without modifying your dispatcher.

*Why?* Allows you to use the dispatcher in your components, and in your effects services.

```typescript
// Consider
/* component.dispatcher.ts */
import { Injectable } from '@angular/core';

import { Model } from './../shared/model';

import {
  LoadModel,
  LoadModelSuccess,
  SaveModel,
  SaveModelSuccess,
} from './component.actions';

@Injectable()
export class ComponentDispatcher {
  public load(id: string) {
    return new LoadModel(id);
  }

  public loadSuccess(data: Model) {
    return new LoadModelSuccess(data);
  }

  public save(data: Model) {
    return new SaveModel(data);
  }

  public saveSuccess() {
    return new SaveModelSuccess();
  }
}
```

### Effects

##### Style NGRX-03

*Consider* using effects to integrate the results of all asynchronous operations with the state of the application.

*Avoid* importing directly the effects into the application.

*Why?* Using effects allows you practice the separation of concerns.

*Why?* Effects does not need to be imported into the application. Only into the Effects Module configuration.

```typescript
// Consider
/* component.effects.ts */
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { map, switchMap } from 'rxjs/operators';

import { ComponentService } from './../shared/component.service';

import {
  ComponentActions,
  ComponentActionType,
  LoadModel,
  SaveModel,
} from './component.actions';
import { ComponentDispatcher } from './component.dispatcher';

@Injectable()
export class ComponentEffects {
  constructor(
    private readonly actions: Actions<ComponentActionType>,
    private readonly service: ComponentService,
    private readonly dispatcher: ComponentDispatcher,
  ) {}

  @Effect()
  public readonly getModel = this.actions.pipe(
    ofType<LoadModel>(ComponentActions.Load),
    switchMap(({ modelId }) => this.service.get(modelId)),
    map(model => this.dispatcher.loadSuccess(model)),
  );

  @Effect({ dispatch: false })
  public readonly saveModel = this.actions.pipe(
    ofType<SaveModel>(ComponentActions.Save),
    switchMap(({ model }) => this.service.update(model.id, model)),
  );
}
```

### Reducers

##### Style NGRX-04

*Consider* placing a module reducer inside the store configuration.

*Consider* using a component reducer aside the component that will use it.

*Consider* declaring the initial state of the reducer as classes.

*Consider* Initialize the state of the module reducer with properties calling a new instance of the component reducer states.

*Why?* Module reducers should only import the components reducers and initialize them.

*Why?* Classes allows to initialize the state of the reducer in a cleaner way.

```typescript
// Consider
/* component.reducer.ts */
import { ActionReducer } from '@ngrx/store';

import { Model } from './../shared/model';

import { ComponentType, ComponentActions } from './component.actions';

export class ComponentState {
  public readonly model: Readonly<Model> = new Model();
}

/**
 * @type {ActionReducer<ComponentState,ComponentType>}
 */
export function componentReducer(
  state: ComponentState,
  action: ComponentType,
): ComponentState {
  switch (action.type) {
    case ComponentActions.LoadSuccess:
      return {
        ...state,
        model: action.model,
      };
    default:
      return state;
  }
}
```
### Selectors

##### Style NGRX-05

*Consider* exporting a feature selector in the store configuration.

*Consider* using `select` pipe operator with a function argument to select component state and properties.

*Consider* create a new selector file only if you are using a composed or complex selectors.  

*Why?* Selectors are a type safe approach to return an observable of the feature state.

*Why?* pipe operators are typed so they are also a type safe approach to get the properties.

```typescript
// Consider
/* feature-store.module.ts */
...
import { createFeatureSelector } from '@ngrx/store';
...

export const selectFeatureState = createFeatureSelector('feature');

/* component.component.ts */
...
export class ComponentComponent {
  models: Observable<Model[]>

  constructor(private readonly store: Store) {
    this.models = store.select(selectFeatureState).pipe(
      select((state) => state.component.models),
    );
  }
}
```

### Entities

##### Style NGRX-06

*Consider* using the `@ngrx/entity` module to create and handle entities.

*Consider* placing the entity adapter inside the reducer file.

*Consider* exporting the selectors generated from the entity adapter.

*Consider* using `select` pipe operator and the entity adapter selectors to  select entities properties.

*Why* The adapter is a function helper to manage the entity state, so you would need it there.

```typescript
// Consider
/* component.reducer.ts */
import {
  createEntityAdapter,
  EntityState,
  Dictionary,
  Update,
} from '@ngrx/entity';

import { Model } from '../model';

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

*Consider* use an Angular barrel module to setup the Store. 

*Consider* setup and export the state from the module.

*Consider* setup and export the selector from the module.

*Why?* Barrel modules are a common practice in Angular, and a recommended way to keep your modules cleaner.

*Why?* The store state, reducers and selectors will be easy to find after scaling.

```typescript
// Avoid
/* counter.ts */
import { Action } from '@ngrx/store';

export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';
export const RESET = 'RESET';

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
import { NgModule } from '@angular/core'
import { StoreModule } from '@ngrx/store';
import { counterReducer } from './counter';

@NgModule({
  imports: [
    BrowserModule,
    StoreModule.forRoot({ count: counterReducer }),
  ],
})
export class AppModule {}
```

```typescript
// Consider
/* app-store.module.ts */
import { NgModule } from '@angular/core';
import { ActionReducerMap, MetaReducer, StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';

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
import { NgModule } from '@angular/core'
import { AppStoreModule } from './app-store.module';

@NgModule({
  imports: [
    BrowserModule,
    AppStoreModule,
  ],
})
export class AppModule {}
```

-----
To-Do: 
- The current amount of files is up-to the recommended by the Angular Style Guide.
- There is not test files for the ducks.
