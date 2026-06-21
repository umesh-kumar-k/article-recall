---
aliases:
  - NgRx Store Module
Source 1: https://angular.love/understanding-the-magic-behind-storemodule-of-ngrx-ngrx-store
---
# Understanding the magic behind StoreModule of NgRx (@ngrx/store)

## Contents
* **Key Points:**
  - I was writing this article as I was going through the source code trying to understand how the main entities of the @ngrx/store module are tied together. As a result I found a bunch of interesting stuff related to each component of the module and this is what I'll be describing in this article. We're going to examine each entity in detail and I'll explain its role in the overall architecture.
  - Before diving in, let's quickly recap what the main entities are:
    - State: a data structure where the internal state of the application is kept
    - Store: a bridge between the data consumer and the state
    - Actions: a mechanism that triggers state changes
    - Reducers: a mechanism that performs state changes by writing into state
    - Meta-reducers: a mechanism to hook into the action -> reducer pipeline

## Actions
* **Key Points:**
  - Actions can be thought of as instructions for reducers and they also represent the base of an effect. They are usually dispatched from the view layer (a smart component, a service etc.) or from effects.

### Creating actions
* **Key Points:**
  - Actions can be created in 3 ways.
  - In each case, the return value of the function will be an object that will contain at least this property: `{ type: T }`.
  - Also, the type (first argument of createAction) will be attached as property to the function. This is useful to know when reducers are created.
* **Technical Entities (Classes/Functions/APIs):** `createAction`, `props`, `ActionCreator`, `TypedAction`, `defineType`
* **Code Snippet:**
```typescript
const action = createAction('[Entity] simple action');
action();

const action = createAction('[Entity] simple action', props<{ name: string, age: number, }>());
action({ name: 'andrei', age: 18 });

const action = createAction('action',(u: User, prefix: string) => ({ name: `${prefix}${u.name}` }) );
const u: User = { /* ... */ };
action(u, '@@@@');
```

### TypeScript's magic
* **Key Points:**
  - Now are going to reveal what important role TypeScript plays here. Ever wondered, for example, why is the props() function useful? Let's find out! The createAction function comes with 3 overloads.
  - ActionCreator<T, C> represents a function of type C that has a readonly property type of type T. The type can also be used to discriminate unions.
* **Technical Entities (Classes/Functions/APIs):** `createAction`, `ActionCreator`, `TypedAction`, `Props`, `NotAllowedCheck`, `FunctionWithParametersType`

#### createAction with only type parameter
* **Key Points:**
  - We can deduce from the above snippet that the return type will be a function which will return an object with a property type.
* **Code Snippet:**
```typescript
const action = createAction('[Entity] simple action');
action(); // { type: [Entity] simple action }
```

#### createAction with props
* **Key Points:**
  - This approach can be used when you want to dispatch an action that contains some data which is valuable to the reducer (e.g userActions.add({ name, age })).
  - What props<T>() does is to return an object with a predefined key(_as: 'props') and with a key of type T which is useful for type inference.
* **Technical Entities (Classes/Functions/APIs):** `props`, `Props`
* **Code Snippet:**
```typescript
const action = createAction('[Entity] simple action', props<{ name: string, age: number, }>());
action({ name: 'andrei', age: 18 });
```

#### createAction with a function
* **Key Points:**
  - This comes in handy when you want to modify the data before it reaches the reducer. Or you might simply want to determine the action's data based on some more complicated logic.
  - Creator<P, R> is simply a function takes up a parameter of type P and returns an object of type R. This will allow us to infer the P and R types. NotAllowedCheck<R> makes sure that the creator is not an existing action or an array. It must be a function that receives some arguments and based on them, it returns an object that represents the action's data.
  - FunctionWithParametersType<P, R & TypedAction<T>> & TypedAction<T>; means that the return type must be a function whose arguments are of type P(inferred from Creator<P, R>), which returns an object of type R(also inferred from Creator<P, R>) and has a property type.
* **Technical Entities (Classes/Functions/APIs):** `createAction`, `Creator`, `FunctionWithParametersType`
* **Code Snippet:**
```typescript
const action = createAction(
  'action', 
  (u: User, prefix: string) => ({ name: `${prefix}${u.name}` }) 
);
const u: User = { /* ... */ };
action(u, '@@@@');
```

## Reducers
* **Key Points:**
  - Reducers are pure functions that are responsible for state changes.
  - As you can see, a reducer takes 2 parameters: the current state and the current action that has been dispatched.
* **Technical Entities (Classes/Functions/APIs):** `ActionReducer`, `ActionReducerMap`

### Providing reducers
* **Key Points:**
  - Reducers can be provided in two ways:
    - an object whose values are reducers created with the help of createReducer
    - an injection token
* **Technical Entities (Classes/Functions/APIs):** `StoreModule.forRoot`, `createReducer`, `InjectionToken`
* **Code Snippet:**
```typescript
// Object way
StoreModule.forRoot({ foo: fooReducer, user: UserReducer })

// Injection token way
const REDUCERS_TOKEN = new InjectionToken('REDUCERS');

@NgModule({
  imports: [
    StoreModule.forRoot(REDUCERS_TOKEN)
  ],
  providers: [
    { provide: REDUCERS_TOKEN, useValue: { foo: fooReducer } }
  ],
})
```

### How are reducers set up?
* **Key Points:**
  - StoreModule.forRoot will return a ModuleWithProviders object which contains, among others, these providers.
  - As you can see, unless you provide a custom reducer factory, the combineReducers function will be used instead. createReducerFactory is mainly used to add the meta-reducers.
  - The REDUCER_FACTORY token will only be injected in ReducerManager class.
  - As soon as that happens, the createReducerFactory function will be invoked, meaning that reducerFactory property will hold its return value, which is a function that takes an object of reducers and, optionally, the initialState.
  - Invoking super(reducerFactory(reducers, initialState)) will combine all the reducers into one object, whose keys represent the store's slices key corresponds to a reducer.
  - Additionally, we can see in the above snippet why the stored data must be immutable. If a reducer returned the same reference of an object, but with a property changed, this would not be reflected into the UI as nextStateForKey !== previousStateForKey would fail.
  - The gist resides in this snippet.
* **Technical Entities (Classes/Functions/APIs):** `_REDUCER_FACTORY`, `combineReducers`, `REDUCER_FACTORY`, `createReducerFactory`, `ReducerManager`, `ActionReducerFactory`, `InitialState`, `ActionReducerMap`

### createReducer helper
* **Key Points:**
  - In order to create reducers that will handle state changes, we can use the createReducer() function. It receives these arguments: the initialState and an indefinite number of on functions whose type will depend on the type of initialState.
  - The on functions are an alternative for using the switch statement. An on function can receive multiple action creators (results of createAction) and the actual reducer as the last argument.
  - It will return an object `{ types: string[], reducer: ActionReducer<S> }`, where types is the type of each provided action creator and reducer is a pure function which handles state changes based on the action and has this signature: `(state: T | undefined, action: V): T`.
  - The createReducer function will create a private Map<string, ActionReducer<S, A>> object, where the key is the type of the action, and the value is the corresponding reducer. It will also return a function whose arguments will be a given state and an action. Because it is a closure, it has access to the Map object. This function will be invoked every time an action is dispatched and what will do is to get the reducer based on the action type. Then, if the reducer is found, it will be called and will potentially return a new state.
  - The on function can bind a reducer to multiple actions. Then, in the reducer, with the help of discriminated unions, we can perform the appropriate state change depending on action.
  - Also, it is worth mentioning that the entire State type will be inferred from what is being passed as initialState.
  - Another great feature that createReducer comes with is composability. You can use the same action with multiple reducers. What this means is that an nth on's reducer state which has an action a will be result of the n-1th on's reducer that has the same action a.
* **Technical Entities (Classes/Functions/APIs):** `createReducer`, `on`, `On`, `OnReducer`, `ActionType`
* **Code Snippet:**
```typescript
const increment = createAction('increment');
const decrement = createAction('decrement');
const reset = createAction('reset');

const _counterReducer = createReducer(initialState,
  on(increment, state => state + 1 /* reducer#1 */),
  on(decrement, state => state - 1 /* reducer#2 */),
  on(reset, state => 0 /* reducer#3 */),
);

export function counterReducer(state, action) {
  return _counterReducer(state, action);
}
```

## Store
* **Key Points:**
  - This is one of the ngrx/store's foundations. This is not the place where the information is kept, but rather a bridge between the data consumer (a smart component) and the place where the information is kept (the State entity).
  - From the above snippet we can tell that the Store class is a hot observable because the data it emits comes from outside, namely state$. This means that every time the source (state$) emits, the Store class will send the value to its subscribers. This is possible because state$ extends BehaviorSubject and by setting this as a source to any other observable, whenever that observable is subscribed to, that observer (subscriber) will be added to the subscribers list maintained by the BehaviorSubject.
  - It is worth noticing the presence of ActionsSubject, with which we are able to push values in the action stream whenever the Store.dispatch is called.
  - You can think of this Store class as a dispatcher, because it can dispatch actions with Store.dispatch(action), but you can also think of it as a data receiver, because you can be notified about changes in the state is by subscribing to a Store instance (Store.subscribe()).
  - So, the State class is the place where actions meet reducers, where the reducers are called with the existing state and based on the action, it will generate and emit a new state through the Store (because the Store's source is the State itself).
  - I'd see the Store class as some sort of middleman between the Model (the place where the data is actually stored) and the Data Consumer.
  - As a side note, Store can not only be used as an observable, but also as an observer (e.g: when intercepting actions emitted by the effects).
* **Technical Entities (Classes/Functions/APIs):** `Store`, `Observable`, `Observer`, `StateObservable`, `ActionsSubject`, `ReducerManager`, `dispatch`, `BehaviorSubject`
* **Code Snippet:**
```typescript
export class Store<T> extends Observable<T> implements Observer<Action> {
  constructor(
    state$: StateObservable,
    private actionsObserver: ActionsSubject,
    private reducerManager: ReducerManager
  ) {
    super();
    this.source = state$;
  }
}
```

### Selecting from the Store
* **Key Points:**
  - Because Store's source is State, which is where the data is kept, selecting from the store and receiving eventual updates is a seamless operation.
  - We can use both Store.select('path' | customSelector) or Store.pipe(select('path') | select(customSelector)). As you can see, both approaches will make use of the select function and will return an observable.
  - There are a couple of ways to fetch data from the store:
    - Provide the path of the slice we're interested in with the help of a sequence of string values
    - Provide a custom operator that will do the mapping
  - Under the hood the mapping is done with the help of the pluck operator, which provides a declarative way to select properties from objects.
  - This is similar to the previous approach, but instead of listing the properties, you provide a custom operator and optionally another argument (props). props may contain some data that is not part of the store and you can use it to alter the shape of the state.
  - Another great benefit of this approach is that it can be used in conjunction with custom selectors, provided by the createSelector() function.
  - createSelector will return a MemoizedSelector/MemoizedSelectorWithProps which extends the base Selector.
  - Sometimes you might want to have more control on the situation. In this case, we can use the createSelector function which can take a bunch of selectors and lastly, a projection function. The selectors will allow us to select certain slices from the store, whereas the last provided function will give the shape of the emitted value (i.e: the state object), based on the existing selectors. It will then project the value into the stream so the subscribers can consume it.
  - This feature is strongly based on the power of pure functions. Because selectors must be pure functions, memoization can take place, which prevents us from redoing the same task multiple times if the same arguments are provided.
* **Technical Entities (Classes/Functions/APIs):** `Store.select`, `select`, `pluck`, `createSelector`, `MemoizedSelector`, `MemoizedSelectorWithProps`, `Selector`, `SelectorWithProps`, `createSelectorFactory`, `defaultMemoize`
* **Code Snippet:**
```typescript
const state = {
  todos: [
    { id: 1, name: 't1', done: true },
    { id: 2, name: 't2', done: true },
    { id: 3, name: 't3', done: false },
  ],
  filterStatus: true,
};

const completedTodosSelector = createSelector(
  (s: typeof state) => s.todos,
  (s: typeof state) => s.filterStatus,
  (todos, crtFilterStatus) => todos.filter(t => t.done === crtFilterStatus)
);

const store$ = of(state);

store$.pipe(
  select(completedTodosSelector)
).subscribe(console.log);
```

### How does the memoization actually work?
* **Key Points:**
  - In order to get a better understanding of how this process works, let's have a look at its foundation.
  - It will return a function (memoizedState.memoized) that can be called with 2 arguments: state and props. memoizedState.memoized is the result of createSelector().
  - Whenever memoizedState.memoized is called, it will verify if there is any difference between the current function's arguments and previous ones. If that's the case, it will call the callback function provided to defaultMemoize.
  - This also justifies why we should always strive for immutability. Imagine you have a custom selector, which takes a userSelector created by createSelector() that depends on feat.users. When adding a new user to feat.users, if you're not creating a new reference of that array, the projection function of userSelector will return the memoized value, because the reference would be same.
* **Technical Entities (Classes/Functions/APIs):** `defaultMemoize`, `MemoizedProjection`, `memoizedState.memoized`, `release`, `setResult`, `clearResult`

## State
* **Key Points:**
  - Among other traits, this is the place where the application's information is kept.
  - This is the place where actions are intercepted and applied to the existing reducers. After the reducers are called with the new action, the resulted state will be sent to the consumers. In this case, it is the Store entity, because it acts as a middleman between the consumer (e.g: a component, a service) and the State (the model, where the information is stored).
  - Will make sure that although actionsOnQueue$ emits, if reducer$ didn't, no values will be pushed forwards into the stream. If both emitted, the values will be emitted only if the observable which emits again is actionsOnQueue$. This way, if reducers are added/removed later, each new action will be applied to the most up to date reducers object.
  - reducer, when called, will loop through the provided reducers and will call them with the existing state and with the current action. It will eventually return a new state which will be pushed forwards into the stream.
  - As mentioned before, this stream is the source of the Store entity, which is how the data consumers can be notified of new state changes.
* **Technical Entities (Classes/Functions/APIs):** `State`, `BehaviorSubject`, `ActionsSubject`, `ReducerObservable`, `ScannedActionsSubject`, `INITIAL_STATE`, `queueScheduler`, `withLatestFrom`, `scan`, `reduceState`
* **Code Snippet:**
```typescript
constructor(
  actions$: ActionsSubject,
  reducer$: ReducerObservable,
  scannedActions: ScannedActionsSubject,
  @Inject(INITIAL_STATE) initialState: any
) { /* ... */ }
```

## Meta-reducers
* **Key Points:**
  - Simply put, meta-reducers are functions that receive a reducer and return a reducer. Additionally, in the same way that interceptors act on an HTTP request, meta-reducers can add behavior before and after a reducer is invoked.

### Setting up meta-reducers
* **Key Points:**
  - _RESOLVED_META_REDUCERS when injected in createReducerFactory, it will be an array resulted from merging the built-in meta-reducers with the custom ones.
  - There are 3 built-in meta-reducers: immutabilityCheckMetaReducer, serializationCheckMetaReducer and inNgZoneAssertMetaReducer.
  - createReducerFactory will return a function that will be called with 2 arguments: reducers and initialState. At the beginning, when the app is barely loaded, the function will be called with the arguments provided in StoreModule.forRoot({ reducers, }, { initialState }). When called, it will create a chain (sort of linked list) of meta-reducers, whose extremity is going to be the reducer. This way, each meta-reducer can add behavior before and after the reducer's invocation.
  - The reason it returns that function is that createReducerFactory will be called when REDUCER_FACTORY is injected in ReducerManager class. ReducerManager will keep reducers up to date when features are added/removed. So, for instance, when a feature comes with its reducer, ReducerManager will combine the existing reducer with the new one.
* **Technical Entities (Classes/Functions/APIs):** `StoreModule`, `USER_PROVIDED_META_REDUCERS`, `_RESOLVED_META_REDUCERS`, `META_REDUCERS`, `createReducerFactory`, `ReducerManager`, `compose`, `immutabilityCheckMetaReducer`, `serializationCheckMetaReducer`, `inNgZoneAssertMetaReducer`

### Providing custom meta-reducers
* **Key Points:**
  - Armed with the knowledge from the previous section, we can now explore how to use custom meta-reducers.
  - Which means we can provide a custom meta-reducer like this: StoreModule.forRoot(reducersMap, { metaReducers, })
* **Technical Entities (Classes/Functions/APIs):** `StoreModule.forRoot`, `RootStoreConfig`, `StoreConfig`, `MetaReducer`
* **Code Snippet:**
```typescript
const myMetaReducer: MetaReducer = (reducer: ActionReducer<any, any>) => {
  return (state, action) => {
    console.log('before', action, state);

    const result = reducer(state, action);

    console.log('after', result);

    return result;
  }
}

export const metaReducers: MetaReducer[] = [myMetaReducer];
```

### Injecting dependencies into a meta-reducer
* **Key Points:**
  - Sometimes we might want to inject dependencies in our meta-reducers. We can take advantage of the META_REDUCER multi provider token.
  - We can inject dependencies by registering the meta-reducer as factory provider with the help of META_REDUCER.
* **Technical Entities (Classes/Functions/APIs):** `META_REDUCER`, `useFactory`, `deps`, `multi`
* **Code Snippet:**
```typescript
export const metaReducerWithDepFactory: (d: any) => MetaReducer = 
  (logger: LogService) => reducer => (state, action) => {
  console.log('meta reducer with dep!', logger, action)

  return reducer(state, action);
}

// Registration
{
  provide: META_REDUCERS,
  multi: true,
  useFactory: metaReducerWithDepFactory,
  deps: [LogService]
}
```

## Using Features
* **Key Points:**
  - Adding a feature module to a root module (where all the reducers reside) can be seen as adding a decoupled slice of cake back to its initial plate. The initial plate can be thought of as the root module and the slice of cake as the feature module. What this means is that there will still be a single source of truth (the plate) but each slice (feature module) can have its own decorations (meta-reducers, reducers).

### Registering feature modules
* **Key Points:**
  - Registering a feature can be achieved with: Store.forFeature(featureName, reducer: ActionReducerMap | ActionReducer, config)
  - where reducer is either an object of reducers (ActionReducerMap) or a function ActionReducer. You can register multiple feature modules at once.
  - It all starts in StoreFeatureModule, where all the provided configurations are collected.
  - Once everything (initialState, reducers, meta-reducers) is gathered in once place (feats array), ReducerManager comes in to play. ReducerManager.addFeatures will sort out the features' reducers. Remember that a feature module's reducer can be either a function or an object of reducers (functions).
  - If it is an object of reducers ({ feat: featReducer }), it will follow the same steps as the ones described in "How are reducers set up?" section above. More concisely, the awesome-feat's reducer will be a function that accepts state and action as arguments and, when invoked, will iterate over the feature's registered reducers (in this case feat, which was created by createReducer) and will call them with the given arguments.
  - If instead the provided feature reducer is function (created by createReducer), it will simply invoke it with the state and action arguments. As with the other approach, the meta-reducer chain will still be created, but the way it is created it slightly different. That's because when a single function is provided, it means it can't be something more than that, it can't be an object of reducers, so there is no need to create another function that, when called, will iterate over the object of reducers and invoke them (which is what happens when an object of reducers is provided).
  - After the reducers have been created accordingly, the single source of truth (the object) will have to be updated.
  - this.next(this.reducerFactory(this.reducers, this.initialState)) will make sure that whenever actions are dispatched, the reducer of each slice will be invoked (including the new slices added). This is how the store is kept update to date every time a new feature is added/removed.
* **Technical Entities (Classes/Functions/APIs):** `Store.forFeature`, `ActionReducerMap`, `ActionReducer`, `StoreFeatureModule`, `_STORE_FEATURES`, `FEATURE_REDUCERS`, `ReducerManager.addFeatures`, `createFeatureReducerFactory`
* **Code Snippet:**
```typescript
StoreModule.forRoot({ foo: fooReducer }),
StoreModule.forFeature('awesome-feat', { feat: featReducer }), // `reducer` - ActionReducerMap
StoreModule.forFeature('counter', counterReducer), // `reducer` - function
```