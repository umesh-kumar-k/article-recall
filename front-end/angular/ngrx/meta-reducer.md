---
aliases:
  - Meta Reducer
Source 1: https://ngrx.io/guide/store/metareducers
---

# Meta-Reducer


* **Key Points:**
  - @ngrx/store composes your map of reducers into a single reducer.
  - Developers can think of meta-reducers as hooks into the action->reducer pipeline. Meta-reducers allow developers to pre-process actions before normal reducers are invoked.
  - Use the metaReducers configuration option to provide an array of meta-reducers that are composed from right to left.
  - Note: Meta-reducers in NgRx are similar to middleware used in Redux.
* **Technical Entities (Classes/Functions/APIs):** `@ngrx/store`, `metaReducers`, `ActionReducer`, `MetaReducer`, `StoreModule.forRoot()`

## Using a meta-reducer to log all actions
* **Technical Entities (Classes/Functions/APIs):** `ActionReducer`, `MetaReducer`
* **Code Snippet:**
```typescript
import { StoreModule, ActionReducer, MetaReducer } from '@ngrx/store';
import { reducers } from './reducers';

// console.log all actions
export function debug(
  reducer: ActionReducer<any>
): ActionReducer<any> {
  return function (state, action) {
    console.log('state', state);
    console.log('action', action);

    return reducer(state, action);
  };
}

export const metaReducers: MetaReducer<any>[] = [debug];

@NgModule({
  imports: [StoreModule.forRoot(reducers, { metaReducers })],
})
export class AppModule {}
```