---
aliases:
  - NgRx Store
Source 1: https://ngrx.io/guide/store
Source 2: https://ngrx.io/guide/store/walkthrough
---
# @ngrx/store


* **Key Points:**
  - Store is RxJS powered global state management for Angular applications, inspired by Redux. Store is a controlled state container designed to help write performant, consistent applications on top of Angular.
  - Key concepts:
    - Actions describe unique events that are dispatched from components and services.
    - State changes are handled by pure functions called reducers that take the current state and the latest action to compute a new state.
    - Selectors are pure functions used to select, derive and compose pieces of state.
    - State is accessed with the Store, an observable of state and an observer of actions.
  - NgRx Store is mainly for managing global state across an entire application. In cases where you need to manage temporary or local component state, consider using NgRx Signals.
  - Note: All Actions that are dispatched within an application state are always first processed by the Reducers before being handled by the Effects of the application state.
* **Technical Entities (Classes/Functions/APIs):** `Store`, `Actions`, `Reducers`, `Selectors`, `NgRx Signals`, `provideStore()`, `createAction()`, `createReducer()`, `on()`

## Tutorial
* **Key Points:**
  - The following tutorial shows you how to manage the state of a counter, and how to select and display it within an Angular component.
* **Technical Entities (Classes/Functions/APIs):** `createAction`, `createReducer`, `on`, `provideStore`, `Store`, `selectSignal`, `dispatch`
* **Code Snippet:**
```typescript
// counter.actions.ts
import { createAction } from '@ngrx/store';

export const increment = createAction('[Counter Component] Increment');
export const decrement = createAction('[Counter Component] Decrement');
export const reset = createAction('[Counter Component] Reset');

// counter.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { increment, decrement, reset } from './counter.actions';

export const initialState = 0;

export const counterReducer = createReducer(
  initialState,
  on(increment, (state) => state + 1),
  on(decrement, (state) => state - 1),
  on(reset, () => 0)
);

// my-counter.component.ts
import { Component, Signal, inject } from '@angular/core';
import { Store } from '@ngrx/store';
import { increment, decrement, reset } from '../counter.actions';

@Component({
  selector: 'ngrx-my-counter',
  template: `
    <button id="increment" (click)="increment()">Increment</button>

    <div>Current Count: {{ count() }}</div>

    <button id="decrement" (click)="decrement()">Decrement</button>

    <button id="reset" (click)="reset()">Reset Counter</button>
  `,
})
export class MyCounterComponent {
  private readonly store: Store<{ count: number }> = inject(Store);
  count: Signal<number> = this.store.selectSignal((state) => state.count);

  increment() {
    this.store.dispatch(increment());
  }

  decrement() {
    this.store.dispatch(decrement());
  }

  reset() {
    this.store.dispatch(reset());
  }
}
```


--- 

# Walkthrough

### Tutorial (continued)

#### Book List Service
* **Key Points:**
  - In the book-list folder, we want to have a service that fetches the data needed for the book list from an API. Create a file in the book-list folder named books.service.ts, which will call the Google Books API and return a list of books.
* **Technical Entities (Classes/Functions/APIs):** `HttpClient`, `inject`, `Injectable`, `Observable`, `map`
* **Code Snippet:**
```typescript
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';

import { Observable, map } from 'rxjs';
import { Book } from './book';

@Injectable({ providedIn: 'root' })
export class GoogleBooksService {
  private readonly http = inject(HttpClient);

  getBooks(): Observable<Array<Book>> {
    return this.http
      .get<{
        items: Book[];
      }>(
        'https://www.googleapis.com/books/v1/volumes?maxResults=5&orderBy=relevance&q=oliver%20sacks'
      )
      .pipe(map((books) => books.items || []));
  }
}
```

#### BookList Component
* **Key Points:**
  - In the same folder (book-list), create the BookList with the following template. Update the BookList class to dispatch the add event.
* **Technical Entities (Classes/Functions/APIs):** `Component`, `EventEmitter`, `Input`, `Output`
* **Code Snippet:**
```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';
import { Book } from './book';

@Component({
  selector: 'app-book-list',
  template: `
    @for (book of books; track book) {
      <div class="book-item">
        <p>{{ book.volumeInfo.title }}</p>
        <span> by {{ book.volumeInfo.authors }}</span>
        <button (click)="add.emit(book.id)">Add to Collection</button>
      </div>
    }
  `,
})
export class BookList {
  @Input() books: ReadonlyArray<Book> = [];
  @Output() add = new EventEmitter<string>();
}
```

#### BookCollection Component
* **Key Points:**
  - Create a new Component named book-collection in the app folder. Update the BookCollection template and class.
* **Technical Entities (Classes/Functions/APIs):** `Component`, `EventEmitter`, `Input`, `Output`
* **Code Snippet:**
```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';
import { Book } from '../book-list/book';

@Component({
  selector: 'app-book-collection',
  template: `
    @for (book of books; track book) {
      <div class="book-item">
        <p>{{ book.volumeInfo.title }}</p>
        <span> by {{ book.volumeInfo.authors }}</span>
        <button (click)="remove.emit(book.id)">Remove from Collection</button>
      </div>
    }
  `,
})
export class BookCollection {
  @Input() books: ReadonlyArray<Book> = [];
  @Output() remove = new EventEmitter<string>();
}
```

#### App Component Integration
* **Key Points:**
  - Add BookList and BookCollection to your App template, and to your imports in app.ts as well.
  - In the App class, add the selectors and corresponding actions to dispatch on add or remove method calls. Then subscribe to the Google Books API in order to update the state. (This should probably be handled by NgRx Effects, which you can read about here. For the sake of this demo, NgRx Effects is not being included).
* **Technical Entities (Classes/Functions/APIs):** `Store`, `selectSignal`, `BooksActions`, `BooksApiActions`, `GoogleBooksService`
* **Code Snippet:**
```typescript
import { Component, inject, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';

import { selectBookCollection, selectBooks } from './state/books.selectors';
import { BooksActions, BooksApiActions } from './state/books.actions';
import { GoogleBooksService } from './book-list/books-service';
import { BookList } from './book-list/book-list';
import { BookCollection } from './book-collection/book-collection';

@Component({
  selector: 'app-root',
  template: `
    <h1>Oliver Sacks Books Collection</h1>

    <h2>Books</h2>
    <app-book-list [books]="books()!" (add)="onAdd($event)" />

    <h2>My Collection</h2>
    <app-book-collection
      [books]="bookCollection()!"
      (remove)="onRemove($event)"
    />
  `,
  imports: [BookList, BookCollection],
})
export class App implements OnInit {
  private readonly booksService = inject(GoogleBooksService);
  private readonly store = inject(Store);

  protected books = this.store.selectSignal(selectBooks);
  protected bookCollection = this.store.selectSignal(selectBookCollection);

  protected onAdd(bookId: string) {
    this.store.dispatch(BooksActions.addBook({ bookId }));
  }

  protected onRemove(bookId: string) {
    this.store.dispatch(BooksActions.removeBook({ bookId }));
  }

  ngOnInit() {
    this.booksService
      .getBooks()
      .subscribe((books) =>
        this.store.dispatch(BooksApiActions.retrievedBookList({ books }))
      );
  }
}
```

#### Summary
* **Key Points:**
  - And that's it! Click the add and remove buttons to change the state.
  - Let's cover what you did:
    - Defined actions to express events.
    - Defined two reducer functions to manage different parts of the state.
    - Registered the global state container that is available throughout your application.
    - Defined the state, as well as selectors that retrieve specific parts of the state.
    - Created two distinct components, as well as a service that fetches from the Google Books API.
    - Injected the Store and Google Books API services to dispatch actions and select the current state.

---

## Why This Happened

The cut-off occurred because:
1. The article content in the `<URL_content>` block was **incomplete** — it stopped at the `app.ts` component code, but the actual tutorial had more content (the sections on creating BookList, BookCollection, and the final summary were presented in the narrative but not as separate section headings).
2. The extraction process stopped when it reached the end of the provided content.

**Going forward**, if you have the full article URL, I can fetch it directly. If the article is behind a paywall or requires JS rendering, you can paste the full text, and I'll process all sections completely.