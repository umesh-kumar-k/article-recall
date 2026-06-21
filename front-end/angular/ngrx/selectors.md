---
aliases:
  - Selectors
Source 1: https://ngrx.io/guide/store/selectors
---
# Selectors

## Introduction
* **Key Points:**
  - Selectors are pure functions used for obtaining slices of store state. @ngrx/store provides a few helper functions for optimizing this selection. Selectors provide many features when selecting slices of state: Portability, Memoization, Composition, Testability, Type Safety.
  - When using the createSelector and createFeatureSelector functions @ngrx/store keeps track of the latest arguments in which your selector function was invoked. Because selectors are pure functions, the last result can be returned when the arguments match without reinvoking your selector function. This can provide performance benefits, particularly with selectors that perform expensive computation. This practice is known as memoization.

## Using a selector for one piece of state
* **Technical Entities (Classes/Functions/APIs):** `createSelector`, `AppState`, `FeatureState`
* **Code Snippet:**
```typescript
import { createSelector } from '@ngrx/store';

export interface FeatureState {
  counter: number;
}

export interface AppState {
  feature: FeatureState;
}

export const selectFeature = (state: AppState) => state.feature;

export const selectFeatureCount = createSelector(
  selectFeature,
  (state: FeatureState) => state.counter
);
```

## Using selectors for multiple pieces of state
* **Key Points:**
  - The createSelector can be used to select some data from the state based on several slices of the same state.
  - The createSelector function can take up to 8 selector functions for more complete state selections.
  - The result will be just some of your state filtered by another section of the state. And it will be always up to date.
  - The createSelector function also provides the ability to pass a dictionary of selectors without a projector. In this case, createSelector will generate a projector function that maps the results of the input selectors to a dictionary.
* **Technical Entities (Classes/Functions/APIs):** `createSelector`, `User`, `Book`, `AppState`
* **Code Snippet:**
```typescript
import { createSelector } from '@ngrx/store';

export interface User {
  id: number;
  name: string;
}

export interface Book {
  id: number;
  userId: number;
  name: string;
}

export interface AppState {
  selectedUser: User;
  allBooks: Book[];
}

export const selectUser = (state: AppState) => state.selectedUser;
export const selectAllBooks = (state: AppState) => state.allBooks;

export const selectVisibleBooks = createSelector(
  selectUser,
  selectAllBooks,
  (selectedUser: User, allBooks: Book[]) => {
    if (selectedUser && allBooks) {
      return allBooks.filter(
        (book: Book) => book.userId === selectedUser.id
      );
    } else {
      return allBooks;
    }
  }
);
```

## Using selectors with props
* **Key Points:**
  - Selectors with props are deprecated.
  - To select a piece of state based on data that isn't available in the store you can pass props to the selector function. These props gets passed through every selector and the projector function. To do so we must specify these props when we use the selector inside our component.
  - The last argument of a selector or a projector is the props argument.
  - Keep in mind that a selector only keeps the previous input arguments in its cache. If you reuse this selector with another multiply factor, the selector would always have to re-evaluate its value. This is because it's receiving both of the multiply factors (e.g. one time 2, the other time 4). In order to correctly memoize the selector, wrap the selector inside a factory function to create different instances of the selector.
* **Technical Entities (Classes/Functions/APIs):** `createSelector`, `selectCounterValue`

## Selecting Feature States
* **Key Points:**
  - The createFeatureSelector is a convenience method for returning a top level feature state. It returns a typed selector function for a feature slice of state.
  - Using a Feature Creator generates the top-level selector and child selectors for each feature state property.
* **Technical Entities (Classes/Functions/APIs):** `createFeatureSelector`, `FeatureState`, `featureKey`
* **Code Snippet:**
```typescript
import { createSelector, createFeatureSelector } from '@ngrx/store';

export const featureKey = 'feature';

export interface FeatureState {
  counter: number;
}

export const selectFeature =
  createFeatureSelector<FeatureState>(featureKey);

export const selectFeatureCount = createSelector(
  selectFeature,
  (state: FeatureState) => state.counter
);
```

## Resetting Memoized Selectors
* **Key Points:**
  - The selector function returned by calling createSelector or createFeatureSelector initially has a memoized value of null. After a selector is invoked the first time its memoized value is stored in memory. If the selector is subsequently invoked with the same arguments it will return the memoized value. If the selector is then invoked with different arguments it will recompute and update its memoized value.
  - A selector's memoized value stays in memory indefinitely. If the memoized value is, for example, a large dataset that is no longer needed it's possible to reset the memoized value to null so that the large dataset can be removed from memory. This can be accomplished by invoking the release method on the selector.
  - Releasing a selector also recursively releases any ancestor selectors.
* **Technical Entities (Classes/Functions/APIs):** `release()`, `createSelector`, `createFeatureSelector`

## Using Store Without Type Generic
* **Key Points:**
  - The most common way to select information from the store is to use a selector function defined with createSelector. TypeScript is able to automatically infer types from createSelector, which reduces the need to provide the shape of the state to Store via a generic argument.
  - So, when injecting Store into components and other injectables, the generic type can be omitted. If injected without the generic, the default generic applied is Store.
  - It is important to continue to provide a Store type generic if you are using the string version of selectors as types cannot be inferred automatically in those instances.
  - When using strict mode, the select method will expect to be passed a selector whose base selects from an object. This is the default behavior of createFeatureSelector when providing only one generic argument.
* **Technical Entities (Classes/Functions/APIs):** `Store`, `createSelector`, `createFeatureSelector`

## Using Signal Selector
* **Key Points:**
  - The selectSignal method expects a selector as an input argument and returns a signal of the selected state slice. It has a similar signature to the select method, but unlike select, selectSignal returns a signal instead of an observable.
* **Technical Entities (Classes/Functions/APIs):** `selectSignal`, `Store`
* **Code Snippet:**
```typescript
import { Component, inject } from '@angular/core';
import { NgFor } from '@angular/common';
import { Store } from '@ngrx/store';

import { selectUsers } from './users.selectors';

@Component({
  standalone: true,
  imports: [NgFor],
  template: `
    <h1>Users</h1>
    <ul>
      <li *ngFor="let user of users()">
        {{ user.name }}
      </li>
    </ul>
  `,
})
export class UsersComponent {
  private readonly store = inject(Store);

  // type: Signal<User[]>
  readonly users = this.store.selectSignal(selectUsers);
}
```

## Selecting with Equality Function
* **Key Points:**
  - Similar to the computed function, the selectSignal method also accepts the equality function to stop the recomputation of the deeper dependency chain if two values are determined to be equal.

## Advanced Usage
* **Key Points:**
  - Selectors empower you to compose a read model for your application state. In terms of the CQRS architectural pattern, NgRx separates the read model (selectors) from the write model (reducers). An advanced technique is to combine selectors with RxJS pipeable operators.

### Breaking Down the Basics
* **Key Points:**
  - Select a non-empty state using pipeable operators: Let's pretend we have a selector called selectValues and the component for displaying the data is only interested in defined values, i.e., it should not display empty states. We can achieve this behaviour by using only RxJS pipeable operators.
* **Technical Entities (Classes/Functions/APIs):** `select()`, `filter`, `map`, `pipe`

### Solution: Extracting a pipeable operator
* **Key Points:**
  - To make the select() and filter() behaviour a reusable piece of code, we extract a pipeable operator using the RxJS pipe() utility function.
* **Technical Entities (Classes/Functions/APIs):** `select()`, `pipe`, `filter`
* **Code Snippet:**
```typescript
import { select } from '@ngrx/store';
import { pipe } from 'rxjs';
import { filter } from 'rxjs/operators';

export const selectFilteredValues = pipe(
  select(selectValues),
  filter((val) => val !== undefined)
);

store.pipe(selectFilteredValues).subscribe(/* .. */);
```

### Advanced Example: Select the last {n} state transitions
* **Key Points:**
  - Let's examine the technique of combining NgRx selectors and RxJS operators in an advanced example. In this example, we will write a selector function that projects values from two different slices of the application state. The projected state will emit a value when both slices of state have a value. Otherwise, the selector will emit an undefined value.
  - Then, the component should visualize the history of state transitions. We are not only interested in the current state but rather like to display the last n pieces of state. Meaning that we will map a stream of state values (1, 2, 3) to an array of state values ([1, 2, 3]).
* **Technical Entities (Classes/Functions/APIs):** `createSelector`, `scan`, `pipe`, `select`, `selectProjectedValues`