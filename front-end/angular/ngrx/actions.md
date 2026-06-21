---
aliases:
  - Actions
Source 1: https://ngrx.io/guide/store/actions
---
# Actions

## Introduction
* **Key Points:**
  - Actions are one of the main building blocks in NgRx. Actions express unique events that happen throughout your application. From user interaction with the page, external interaction through network requests, and direct interaction with device APIs, these and more events are described with actions.
  - Actions are used in many areas of NgRx. Actions are the inputs and outputs of many systems in NgRx. Actions help you to understand how events are handled in your application.

## The Action interface
* **Key Points:**
  - An Action in NgRx is made up of a simple interface: `interface Action { type: string; }`
  - The interface has a single property, the type, represented as a string. The type property is for describing the action that will be dispatched in your application. The value of the type comes in the form of [Source] Event and is used to provide a context of what category of action it is, and where an action was dispatched from. You add properties to an action to provide additional context or metadata for an action.
* **Technical Entities (Classes/Functions/APIs):** `Action`, `type`

## Writing actions
* **Key Points:**
  - There are a few rules to writing good actions within your application.
    - Upfront - write actions before developing features to understand and gain a shared knowledge of the feature being implemented.
    - Divide - categorize actions based on the event source.
    - Many - actions are inexpensive to write, so the more actions you write, the better you express flows in your application.
    - Event-Driven - capture events not commands as you are separating the description of an event and the handling of that event.
    - Descriptive - provide context that are targeted to a unique event with more detailed information you can use to aid in debugging with the developer tools.
  - Following these guidelines helps you follow how these actions flow throughout your application.
  - The createAction function returns a function, that when called returns an object in the shape of the Action interface. The props method is used to define any additional metadata needed for the handling of the action. Action creators provide a consistent, type-safe way to construct an action that is being dispatched.
  - The category of the action is captured within the square brackets [].
  - The category is used to group actions for a particular area, whether it be a component page, backend API, or browser API.
  - The Login text after the category is a description about what event occurred from this action. In this case, the user clicked a login button from the login page to attempt to authenticate with a username and password.
  - Note: You can also write actions using class-based action creators, which was the previously defined way before action creators were introduced in NgRx. If you are looking for examples of class-based action creators, visit the documentation for versions 7.x and prior.
* **Technical Entities (Classes/Functions/APIs):** `createAction`, `props`
* **Code Snippet:**
```typescript
import { createAction, props } from '@ngrx/store';

export const login = createAction(
  '[Login Page] Login',
  props<{ username: string; password: string }>()
);
```

## Dispatching actions on signal changes
* **Key Points:**
  - You can also dispatch functions that return actions, with property values derived from signals.
  - dispatch executes initially and every time the bookId changes. If dispatch is called within an injection context, the signal is tracked until the context is destroyed. In the example above, that would be when BookComponent is destroyed.
  - When dispatch is called outside a component's injection context, the signal is tracked globally throughout the application's lifecycle. To ensure proper cleanup in such a case, provide the component's injector to the dispatch method.
  - When passing a function to the dispatch method, it returns an EffectRef. For manual cleanup, call the destroy method on the EffectRef.
* **Technical Entities (Classes/Functions/APIs):** `dispatch`, `EffectRef`, `destroy`, `Injector`
* **Code Snippet:**
```typescript
book.component.ts

class BookComponent {
  bookId = input.required<number>();
  injector = inject(Injector);
  store = inject(Store);

  ngOnInit() {
    // runs outside the injection context
    this.store.dispatch(() => loadBook({ id: this.bookId() }), {
      injector: this.injector,
    });
  }
}
```

## Next Steps
* **Key Points:**
  - Action's only responsibilities are to express unique events and intents. Learn how they are handled in the guides below.
  - Reducers
  - Effects