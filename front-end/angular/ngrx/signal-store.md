---
aliases:
  - NgRx Signal Store
Source 1: https://angular.love/breakthrough-in-state-management-discover-the-simplicity-of-signal-store-part-1
Source 2: https://www.angulararchitects.io/en/blog/the-new-ngrx-signal-store-for-angular-2-1-flavors/
---

# The new NGRX Signal Store for Angular: 3+n Flavors

## Getting the Package

* **Key Points:**
  - To install the Signal Store, you just need to add the package @ngrx/signals to your application: `npm i @ngrx/signals`
* **Technical Entities (Classes/Functions/APIs):** `@ngrx/signals`

## Flavor 1: Lightweight with signalState
* **Key Points:**
  - A very lightweight way of managing Signals with the Signal Store is its signalState function (not to be confused with the signalStore function). It creates a simple container for managing the passed state using Signals. This container is represented by the type SignalState.
  - Each top-level state property gets its own Signal. These properties are retrieved as read-only Signals, ensuring a separation between reading and writing: Consumers using the Signals can just read their values. For updating the state, the service encapsulating the state provides methods (see below). This ensures that the state can only be updated in a well-defined manner.
  - Also, nested objects like the one provided by the preferences property above result in nested signals. Hence, one can retrieve the whole preferences object as a Signal but also its properties.
  - Currently, this isn't implemented for arrays, as Angular's envisioned Signal Components will solve this use case by creating a Signal for each iterated item.
* **Technical Entities (Classes/Functions/APIs):** `signalState`, `SignalState`, `@ngrx/signals`
* **Code Snippet:**
```typescript
@Injectable({ providedIn: 'root' })
export class FlightBookingFacade {
    private state = signalState({
        from: 'Paris',
        to: 'London',
        preferences: {
          directConnection: false,
          maxPrice: 350,
        },        
        flights: [] as Flight[],
        basket: {} as Record<number, boolean>,
    });

    // fetch read-only signals
    flights = this.state.flights;
    from = this.state.from;
    to = this.state.to;
    basket = this.state.basket;
}
```

### Selecting and Computing Signals
* **Key Points:**
  - As the Signal Store provides the state as Signals, we can directly use Angular's computed function.
  - Here, computed serves the same purpose as Selectors in the Redux-based NGRX Store: It enables us to calculate different state representations for different use cases. These so-called View Models are only recomputed when at least one of the underlying signals changes.
* **Technical Entities (Classes/Functions/APIs):** `computed`
* **Code Snippet:**
```typescript
selected = computed(() =>
  this.flights().filter((f) => this.basket()[f.id])
);
```

### Updating State
* **Key Points:**
  - For updating the SignalState, Signal Store provides us with a patchState function.
  - Here, we pass in the state container and a partial state. As an alternative, one can pass a function taking the current state and transforming it to the new state.
* **Technical Entities (Classes/Functions/APIs):** `patchState`, `SignalState`
* **Code Snippet:**
```typescript
import { patchState } from '@ngrx/signals';

updateCriteria(from: string, to: string): void {
  patchState(this.state, { from, to })
}

updateBasket(id: number, selected: boolean): void {
  patchState(this.state, state => ({
    basket: {
      ...state.basket,
      [id]: selected,
    },
  }));
}
```

### Side Effects
* **Key Points:**
  - Besides updating the state, methods can also trigger side effects like loading and saving objects.
* **Technical Entities (Classes/Functions/APIs):** `patchState`
* **Code Snippet:**
```typescript
async load() {
  if (!this.from() || !this.to()) return;

  const flights = await this.flightService.findPromise(
    this.from(),
    this.to()
  );

  patchState(this.state, { flights });
}
```

### Decoupling Intention from Execution
* **Key Points:**
  - Sometimes, the caller of patchState only knows that some state needs to be updated without knowing where it's located. For such cases, you can provide Updaters. Updaters are just functions taking a current state and returning an updated version of it.
  - It's also fine to just return a partial state. It will be patched over the current state.
  - If you don't need to project the current state, just returning a partial state is fine too. In this case, you can skip the inner function.
  - Updater can be defined in the Store's (signalState's) "sovereign territory". For the consumer, it is just a black box.
  - Passing an Updater to patchState expresses an intention. This is similar to dispatching an Action in the classic NGRX store. However, other than with Redux, no eventing is involved, and we cannot prevent the caller from directly passing their own Updaters. For the latter reason, I'm hiding the SignalStore behind a facade.
* **Technical Entities (Classes/Functions/APIs):** `Updaters`, `patchState`, `SignalStore`
* **Code Snippet:**
```typescript
type BasketSlice = { basket: Record<number, boolean> };
type BasketUpdateter = (state: BasketSlice) => BasketSlice;

export function updateBasket(flightId: number, selected: boolean): BasketUpdateter {
  return (state) => ({
    ...state,
    basket: {
      ...state.basket,
      [flightId]: selected,
    },
  });
}
```

## Flavor 2: Powerful with signalStore
* **Key Points:**
  - Similar to signalState, the signalStore function creates a container managing state with Signals. However, now, this container is a fully-fledged Store that not only comes with state Signals but also with computed Signals as well as methods for updating the state and for triggering side effects. Hence, there is less need for crafting a facade by hand, as shown above.
  - Technically, the Store is an Angular service that is composed of several pre-existing features.
  - In this case, the service is also registered in the root scope. When skipping `{ providedIn: 'root' }`, one needs to register the service by hand, e. g., by providing it when bootstrapping the application, within a router configuration, or on component level.
* **Technical Entities (Classes/Functions/APIs):** `signalStore`, `withState`, `withComputed`, `withMethods`, `withHooks`
* **Code Snippet:**
```typescript
export const FlightBookingStore = signalStore(
  { providedIn: 'root' },
  withState({
    from: 'Paris',
    to: 'London',
    initialized: false,
    flights: [] as Flight[],
    basket: {} as Record<number, boolean>,
  }),

  // Activating further features
  withComputed([...]),
  withMethods([...]),
  withHooks([...]),
)
```

### Selecting and Computing Signals
* **Key Points:**
  - The withComputed feature takes the store with its state Signals and defines an object with calculated signals.
  - The returned computed signals become part of the store. A more compact version might involve directly destructuring the passed store.
* **Technical Entities (Classes/Functions/APIs):** `withComputed`, `computed`
* **Code Snippet:**
```typescript
withComputed((store) => ({
  selected: computed(() => store.flights().filter((f) => store.basket()[f.id])),
  criteria: computed(() => ({ from: store.from(), to: store.to() })),
})),
```

### Methods for Updating State and Side Effects
* **Key Points:**
  - Similar to withComputed, withMethods also takes the store and returns an object with methods.
  - withMethods runs in an injection context and hence can use inject to get hold of services. After withMethods was executed, the retrieved methods are added to the store.
* **Technical Entities (Classes/Functions/APIs):** `withMethods`, `patchState`, `inject`
* **Code Snippet:**
```typescript
withMethods((state) => {
  const { basket, flights, from, to, initialized } = state;
  const flightService = inject(FlightService);

  return {
    updateCriteria: (from: string, to: string) => {
      patchState(state, { from, to });
    },
    updateBasket: (flightId: number, selected: boolean) => {
      patchState(state, {
        basket: {
          ...basket(),
          [flightId]: selected,
        },
      });
    },
    delay: () => {
      const currentFlights = flights();
      const flight = currentFlights[0];

      const date = addMinutes(flight.date, 15);
      const updFlight = { ...flight, date };
      const updFlights = [updFlight, ...currentFlights.slice(1)];

      patchState(state, { flights: updFlights });
    },
    load: async () => {
      if (!from() || !to()) return;
      const flights = await flightService.findPromise(from(), to());
      patchState(state, { flights });
    }     
  };
}),
```

### Consuming the Store
* **Key Points:**
  - From the caller's perspective, the store looks a lot like the facade shown above. We can inject it into a consuming component.
* **Technical Entities (Classes/Functions/APIs):** `inject`, `FlightBookingStore`

### Hooks
* **Key Points:**
  - The function withHooks provides another feature allowing to setup lifecycle hooks to run when the store is initialized or destroyed.
  - Both hooks get the store passed. One more time, by using destructuring, you can focus on a subset of the stores members.
* **Technical Entities (Classes/Functions/APIs):** `withHooks`, `onInit`, `onDestroy`
* **Code Snippet:**
```typescript
withHooks({
  onInit({ load }) {
    load()
  },
  onDestroy({ flights }) {
    console.log('flights are destroyed now', flights());
  },
}),
```

## rxMethod
* **Key Points:**
  - While Signals are easy to use, they are not a full replacement for RxJS. For leveraging RxJS and its powerful operators, the Signal Store provides a secondary entry point @ngrx/signals/rxjs-interop, containing a function rxMethod<T>. It allows working with an Observable representing side-effects that automatically run when specific values change.
  - The type parameter T defines the type the rxMethod works on. While the rxMethod receives an Obserable<T>, the caller can also pass an Observable<T>, a Signal<T>, or T directly. In the latter two cases, the passed values are converted into an Observable.
  - After defining the rxMethod, somewhere else in the application, e. g. in a hook or a regular method, you can call this effect.
* **Technical Entities (Classes/Functions/APIs):** `rxMethod`, `@ngrx/signals/rxjs-interop`
* **Code Snippet:**
```typescript
import { rxMethod } from '@ngrx/signals/rxjs-interop';

withMethods(({ $update, basket, flights, from, to, initialized }) => {
  const flightService = inject(FlightService);

  return {
    [...]
    connectCriteria: rxMethod<Criteria>((c$) => c$.pipe(
      filter(c => c.from.length >= 3 && c.to.length >= 3),
      debounceTime(300),
      switchMap((c) => flightService.find(c.from, c.to)),
      tap(flights => patchState(state, { flights }))
    ))
  }
});
```

## Custom Features - The Road Towards Further Flavors
* **Key Points:**
  - Besides configuring the Store with baked-in features, everyone can write their own features to automate repeating tasks. The playground provided by Marko Stanimirović, the NGRX contributor behind the Signal Store, contains several examples of such features.
  - One of the examples found in this repository is a CallState feature defining a state property informing about the state of the current HTTP call.
  - In this section, I'm using this example to explain how to provide custom features.
* **Technical Entities (Classes/Functions/APIs):** `signalStoreFeature`, `withState`, `withComputed`

### Defining Custom Features
* **Key Points:**
  - A feature is usually created by calling signalStoreFeature. This function constructs a new feature on top of existing ones.
  - For the state properties added by the feature, one can provide Updaters.
  - Updaters allows the consumer to modify the feature state without actually knowing how it's structured.
* **Technical Entities (Classes/Functions/APIs):** `signalStoreFeature`, `withState`, `withComputed`, `CallState`, `Updaters`
* **Code Snippet:**
```typescript
export type CallState = 'init' | 'loading' | 'loaded' | { error: string };

export function withCallState() {
  return signalStoreFeature(
    withState<{ callState: CallState }>({ callState: 'init' }),
    withComputed(({ callState }) => ({
      loading: computed(() => callState() === 'loading'),
      loaded: computed(() => callState() === 'loaded'),
      error: computed(() => {
        const state = callState();
        return typeof state === 'object' ? state.error : null
      }),
    }))
  );
}
```

### Using Custom Features
* **Key Points:**
  - For using Custom Features, just call the provided factory when setting up the store.
  - The provided properties, methods, and Updaters can be used in the Store's methods.
  - The consumer of the store sees the properties provided by the feature too.
  - As each feature is transforming the Store's properties and methods, make sure to call them in the right order. If we assume that methods registered with withMethods use the CallState, withCallState has to be called before withMethods.
* **Technical Entities (Classes/Functions/APIs):** `withCallState`, `patchState`, `setLoading`, `setLoaded`, `withMethods`
* **Code Snippet:**
```typescript
export const FlightBookingStore = signalStore(
  { providedIn: 'root' },
  withState({ [...] }),

  // Add feature:
  withCallState(),
  [...]

  withMethods([...])
  [...]
);
```

## Flavor 3: Built-in Features like Entity Management
* **Key Points:**
  - The NGRX Signal Store already comes with a convenient extension for managing entities. It can be found in the secondary entry point @ngrx/signals/entities and provides data structures for entities but also several Updaters, e. g., for inserting entities or for updating a single entity by id.
  - To setup entity management, just call the withEntities function.
  - The passed collection name prevents naming conflicts. In our case, the collection is called flight, and hence the feature creates several properties beginning with flight, e.g., flightEntities.
  - There is quite an amount of ready-to-use Updaters: addEntity, addEntities, removeEntity, removeEntities, removeAllEntities, setEntity, setEntities, setAllEntities, updateEntity, updateEntities, updateAllEntities.
  - Similar to @ngrx/entities, internally, the entities are stored in a normalized way. That means they are stored in a dictionary, mapping their primary keys to the entity objects. This makes it easier to join them together to View Models needed for specific use cases.
  - As we call our collection flight, withEntities creates a Signal flightEntityMap mapping flight ids to our flight objects. Also, it creates a Signal flightIds containing all the ids in the order. Both are used by the also added computed signal flightEntities used above. It returns all the flights as an array respecting the order of the ids within flightIds. Hence, if you want to rearrange the positions of our flights, just update the flightIds property accordingly.
  - For building the structures like the flightEntityMap, the Updaters need to know how the entity's id is called. By default, it assumes a property id. If the id is called differently, you can tell the Updater by using the idKey property.
  - The passed property needs to be a string or number. If it's of a different data type or if it doesn't exist at all, you get a compilation error.
* **Technical Entities (Classes/Functions/APIs):** `withEntities`, `@ngrx/signals/entities`, `addEntity`, `addEntities`, `removeEntity`, `removeEntities`, `removeAllEntities`, `setEntity`, `setEntities`, `setAllEntities`, `updateEntity`, `updateEntities`, `updateAllEntities`, `idKey`, `flightEntityMap`, `flightIds`
* **Code Snippet:**
```typescript
import { withEntities } from '@ngrx/signals/entities';

const BooksStore = signalStore(
  [...]

  // Defining an Entity
  withEntities({ entity: type<Flight>(), collection: 'flight' }),

  // withEntities created a flightEntities signal for us:
  withComputed(({ flightEntities, basket, from, to }) => ({
    selected: computed(() => flightEntities().filter((f) => basket()[f.id])),
    criteria: computed(() => ({ from: from(), to: to() })),
  })),

  withMethods((state) => {
    const { basket, flightEntities, from, to, initialized } = state;
    const flightService = inject(FlightService);

    return {
      [...],

      load: async () => {
        if (!from() || !to()) return;
        patchState(state, setLoading());

        const flights = await flightService.findPromise(from(), to());

        // Updating entities with out-of-the-box setAllEntities Updater
        patchState(state, setAllEntities(flights, { collection: 'flight' }));
        patchState(state, setLoaded());
      },

      [...],
    };
  }),
);
```

## Conclusion
* **Key Points:**
  - The upcoming NGRX Signal Store allows managing state using Signals. The most lightweight option for using this library is just to go with a SignalState container. This data structure provides a Signal for each state property. These signals are read-only. For updating the state, you can use the patchState function. To make sure updates only happen in a well-defined way, the signalState can be hidden behind a facade.
  - The SignalStore is more powerful and allows to register optional features. They define the state to manage but also methods operating on it. A SignalStore can be provided as a service and injected into its consumers.
  - The SignalStore also provides an extension mechanism for implementing custom features to ease repeating tasks. Out of the box, the Signal Store comes with a pretty handy feature for managing entities.
  - 

---

# Breakthrough in State Management – Discover the Simplicity of Signal Store

## Introduction
* **Key Points:**
  - The realm of Angular development is witnessing a transformative shift with the advent of @ngrx/signals, a tool that signifies a major step towards a functional approach in state management. This breakthrough aligns with Angular's evolving capabilities, notably its new primitives: signals. @ngrx/signals is not just about managing state; it's about redefining how state is managed in Angular applications. It brings simplicity, flexibility, and a new functional perspective to the forefront, making it a game-changer for developers seeking efficient state management solutions.
* **Technical Entities (Classes/Functions/APIs):** `@ngrx/signals`, `signals`

## Historical Context and Evolution of State Management in Angular
* **Key Points:**
  - The journey of state management in Angular has been marked by various approaches and solutions. Initially, Angular developers managed state using services and RxJS, an effective method but often leading to complex, unpredictable data flow and lack of specific structure which is hard to debug. The emergence of NgRx introduced a more structured way of managing the state based on the Redux pattern, improving state predictability, albeit with increased verbosity.
  - As Angular evolved, libraries like Akita and NGXS offered different philosophies for state management, focusing on simplifying the developer experience. Another notable development was @ngrx/component-store, a standalone library providing a more lightweight, reactive state management solution for Angular.
  - The introduction of @ngrx/signals marks the latest evolution in this trajectory. It combines the robust architecture of NgRx with a functional, less verbose approach. Drawing upon Angular's core primitives, @ngrx/signals delivers a flexible and efficient way to manage state, effectively reducing the boilerplate associated with traditional state management in Angular. This progression reflects the ongoing efforts to streamline Angular development, making it more approachable and efficient for developers.
* **Technical Entities (Classes/Functions/APIs):** `NgRx`, `Redux`, `Akita`, `NGXS`, `@ngrx/component-store`

## General Usage
* **Key Points:**
  - @ngrx/signals is not just another state management library; it's a paradigm shift in how developers approach state in Angular applications. Its application spectrum is vast and versatile, addressing the nuanced needs of modern web development.
  - Basic Implementation: The primary appeal of @ngrx/signals lies in its intuitive implementation. Integrating directly with Angular's core features offers a familiar yet innovative approach to state management. Developers can leverage their existing Angular expertise to harness the power of @ngrx/signals, experiencing a blend of familiarity and innovation.
  - Integration with Angular Features: Beyond basic implementation, @ngrx/signals excels in its synergy with Angular's broader ecosystem. It enhances the functionality of components and services, ensuring a seamless development experience. This integration facilitates a more coherent application architecture, where state management is an integral yet unobtrusive part of the overall design.
  - Diverse Application Scenarios: The flexibility of @ngrx/signals is evident in its application across a spectrum of development scenarios. It adeptly handles state in smaller applications, offering simplicity and speed. Its robustness and scalability come to the fore in larger, more complex systems, managing intricate state dynamics effectively. @ngrx/signals offers a performance edge, ensuring that state management is responsive and efficient.

## Technical Deep-Dive: The Intricacies of @ngrx/signals
* **Key Points:**
  - The signalStore function in @ngrx/signals is a masterclass in factory function design. Overloaded to support varying levels of feature complexity, it allows the creation of highly configurable stores. This design showcases the library's flexibility, catering to diverse state management needs.
  - Crucial to signalStore is its ability to compose features. Each feature represents a modular piece of store logic, which signalStore seamlessly combines into a unified, efficient state management solution. This compositional approach exemplifies modern software design principles, emphasizing modularity and reusability.
  - @ngrx/signals harnesses TypeScript generics to enforce type safety, a critical aspect for large-scale application development. This ensures developers can precisely shape their store structure, greatly reducing the risk of runtime errors and enhancing maintainability.
  - A standout feature of signalStore is its deep integration with Angular's core features, like dependency injection and lifecycle hooks. This ensures that @ngrx/signals align with Angular's development patterns, making it a natural fit for developers.
* **Technical Entities (Classes/Functions/APIs):** `signalStore`, `TypeScript generics`

## Comparison with Other State Management Solutions
* **Key Points:**
  - Comparison with NgRx Store:
    - Complexity and Boilerplate: While NgRx Store offers a comprehensive Redux-like pattern, it often requires more boilerplate and setup, making it more complex. @ngrx/signals, in contrast, simplifies state management with less boilerplate and more straightforward state updates.
    - Learning Curve: NgRx Store has a steeper learning curve due to its intricate patterns (actions, reducers effects, selectors), whereas @ngrx/signals is more accessible, especially for developers new to state management.
  - Comparison with @ngrx/component-store:
    - Scope of Usage: @ngrx/component-store is designed primarily for component-level state management. @ngrx/signals, however, extends its utility to both component-level and global state management, offering a more versatile solution.
    - API and Design: @ngrx/signals introduces a more functional programming style compared to @ngrx/component-store.
    - Modularity: with SignalStore we can divide our implementation into separate building blocks like, updaters, side effects, computed state where each of them might be in separate file if needed. It's not possible with component store, what often lead for component store file rapidly growing.
  - Comparison with Akita and NGXS:
    - Simplicity: Both Akita and NGXS provide powerful abstractions for state management but can be more complex and feature-rich. @ngrx/signals focuses on simplicity and ease of use, offering a leaner solution for many applications.
    - Functional Approach: @ngrx/signals stands out with its functional approach to handling state changes, distinguishing it from the more object-oriented approaches of Akita and NGXS.
    - Extensibility: In comparison to the class-based state management is extensibility (We can't extend multiple classes with class-based approach).
  - Unique Advantages of @ngrx/signals:
    - Minimal Boilerplate: @ngrx/signals reduces the need for extensive boilerplate code, streamlining state management.
    - Ease of Integration: It integrates seamlessly with Angular's core features, making it a natural choice for Angular developers.
    - Reactive and Efficient: The functional, reactive nature of @ngrx/signals ensures efficient state updates and easier management of asynchronous operations.
* **Technical Entities (Classes/Functions/APIs):** `NgRx Store`, `@ngrx/component-store`, `Akita`, `NGXS`

## Key Components of @ngrx/signals
### signalStore
* **Key Points:**
  - signalStore is the core function that creates a new store service. It combines various features and state slices into a single store. You can think of it as a container that brings together all parts of your state management.
  - signalStore is utilized to instantiate CartStore globally. The Dependency Injection (DI) configuration, an optional first parameter of the function, allows for component-level provisioning if not globally defined. This DI configuration mirrors the conventional Angular service setup using the @Injectable decorator's providedIn property.
* **Technical Entities (Classes/Functions/APIs):** `signalStore`, `CartStore`
* **Code Snippet:**
```typescript
export const CartStore = signalStore(
 { providedIn: 'root' },
 /* other features */
);
```

### withState
* **Key Points:**
  - withState initializes and configures state slices within the store, essentially setting up the default state of your application or specific features. It's fundamental for defining initial state values.
* **Technical Entities (Classes/Functions/APIs):** `withState`
* **Code Snippet:**
```typescript
withState({
 cartItems: [] as CartItem[],
})
```

### withComputed
* **Key Points:**
  - withComputed is utilized for defining dynamic properties that automatically update in response to changes in the state, streamlining the creation of reactive state dependencies.
* **Technical Entities (Classes/Functions/APIs):** `withComputed`, `computed`
* **Code Snippet:**
```typescript
withComputed(({ cartItems }) => ({
 cartItemsCount: computed(() => cartItems().length),
}))
```

### withMethods
* **Key Points:**
  - withMethods enhances the store by adding functional methods for state updates or side effects, encapsulating complex state mutations into manageable operations.
* **Technical Entities (Classes/Functions/APIs):** `withMethods`, `patchState`
* **Code Snippet:**
```typescript
withMethods((store) => ({
 addItemToCart: (item: CartItem) => {
   patchState(state, { cartItems: [...state.cartItems(), item] });
 },
}))
```

### patchState
* **Key Points:**
  - patchState serves as a key utility in Signal Store for applying targeted updates to the state, facilitating direct modifications or transformative operations while maintaining the state's immutability and integrity. We can utilize function in various ways. In scope of our methods defined within withMethods or just directly within component which is consuming our store.
* **Technical Entities (Classes/Functions/APIs):** `patchState`
* **Code Snippet:**
```typescript
addCartItem(cartItem: CartItem): void {
  patchState(this.cartState, (state) => ({
    cartItems: [...state.cartItems, cartItem],
  }));
}
```

### withHooks
* **Key Points:**
  - withHooks seamlessly incorporates lifecycle hooks for initiating custom logic at critical phases like initialization (onInit) and destruction (onDestroy) of a store, aligning state management with Angular's lifecycle mechanisms.
* **Technical Entities (Classes/Functions/APIs):** `withHooks`, `onInit`, `onDestroy`
* **Code Snippet:**
```typescript
withHooks({
  onInit: (store) => console.log('Store initialized', store),
  onDestroy: (store) => console.log('Store destroyed', store)
})
```

## Declaring a Signal Store
* **Key Points:**
  - Combining all of the above pieces we can declare a Signal Store. It can be declared similarly to regular service. The store can be defined in its own dedicated file, offering modularity and reusability. Depending on the application's architecture, it can be provided globally or scoped to specific modules.
* **Technical Entities (Classes/Functions/APIs):** `signalStore`, `withState`, `withComputed`, `withMethods`, `withHooks`
* **Code Snippet:**
```typescript
export const CartStore = signalStore(
 withState({
   cartItems: [] as CartItem[],
 }),
 withComputed(({ cartItems }) => ({
   cartItemsCount: computed(() => cartItems().length),
 })),
 withMethods((store) => ({
   addItemToCart: (item: CartItem) => {
     patchState(store, { cartItems: [...store.cartItems(), item] });
   },
 })),
 withHooks({
   onInit: (store) => console.log('Store initialized', store),
   onDestroy: (store) => console.log('Store destroyed', store)
 }) 
);
```

## Injecting the Store
* **Key Points:**
  - The declared store can be injected into components or services as needed.
* **Technical Entities (Classes/Functions/APIs):** `inject`, `CartStore`
* **Code Snippet:**
```typescript
@Component({
  selector: 'cart-component',
  template: `
    <p>Cart items count: {{cartStore.cartItemsCount()}}</p>
    <button (click)="addItemToCart()">Add item to cart</button>
  `,
  standalone: true,
  providers: [CartStore]
})
export class CartComponent {
  cartStore = inject(CartStore);

  addItemToCart(): void {
    this.cartStore.addItemToCart({ price: 10 });
  }
}
```

## Extending Services with Signal Store
* **Key Points:**
  - Signal Stores can be used to extend existing Angular services, adding state management capabilities to them. This approach allows for the integration of state management directly into service classes, leveraging the power and simplicity of NgRx Signals.
  - In this example we can see the essence of the signalStore – modularity and simplicity of composing stores thanks to the functional approach. Utilizing the function provided by lib we created a simple store responsible for managing CartItems which is extremely easy to further extend.
* **Technical Entities (Classes/Functions/APIs):** `extends`, `CartStore`, `@Injectable`
* **Code Snippet:**
```typescript
@Injectable({
  providedIn: 'root'
})
export class CartService extends CartStore {
  // Additional logic for the CartService
}
```

## withEntities
* **Key Points:**
  - As many of us who have utilized @ngrx/store know, the @ngrx/entity package offers a robust API that dramatically streamlines the manipulation and querying of entity collections. It significantly reduces repetitive code by handling common operations, such as adding, updating, and removing items from collections, thus enhancing code maintainability and conciseness. Moreover, its built-in selectors facilitate easy querying and state selection, thereby simplifying interactions with complex data structures.
  - Building on this foundation, @ngrx/signals introduces withEntities, which further simplifies the management of entity collections in the state. It establishes an entity map and an array of entity IDs, along with computed signals for effortless access to entity lists. This streamlined approach enhances operations like the addition, removal, and updating of items.
  - This example highlights the ease with which withEntities facilitates entity management. Automatically created selectors efficiently retrieve collections of entities, while a suite of utility functions, including addEntity, addEntities, setEntity, and setEntities, manage the lifecycle of entities in the store. Whether you're adding, updating, or removing entities, functions such as updateEntity, updateEntities, removeEntity, and removeEntities follow intuitive patterns, making state management more efficient and developer-friendly.
  - In future articles, I'll delve into two complex yet rewarding aspects of @ngrx/signals. Firstly, custom store features leverage signalStoreFeature to extend core functionality and encapsulate common patterns, offering a structured approach to enhance Angular applications. Secondly, RxJS integration via rxMethod showcases the synergy between RxJS's reactive programming and state management in signal store.
* **Technical Entities (Classes/Functions/APIs):** `withEntities`, `addEntity`, `addEntities`, `setEntity`, `setEntities`, `updateEntity`, `updateEntities`, `removeEntity`, `removeEntities`, `signalStoreFeature`, `rxMethod`
* **Code Snippet:**
```typescript
export interface CartItem {
 id: number;
 name: string;
 price: number;
}

export const CartStore = signalStore(withEntities<CartItem>());

@Component({
 selector: 'cart-component',
 standalone: true,
 providers: [CartStore],
 template: `
       @for (cartItem of cartState.entities(); track cartItem.id) {
       <li>{{ cartItem.name }}</li>
     }
 `,
})
export class CartComponent {
 cartState = inject(CartStore);

 addCartItem(cartItem: CartItem): void {
   patchState(this.cartState, addEntity(cartItem));
 }
}
```

## Flexibility
* **Key Points:**
  - The standout feature of @ngrx/signals is its unparalleled flexibility, making it a perfect fit for virtually any Angular project.
  - Extensibility: @ngrx/signals is designed with extensibility at its core. It allows developers to build upon its foundation, creating custom extensions that cater to specific project needs. This adaptability means that @ngrx/signals can evolve alongside your project, accommodating new requirements and scenarios as they arise.
  - Compatibility with Other State Management Systems: One of the most compelling aspects of @ngrx/signals is its ability to integrate with other state management systems like NgRx or NGXS. This compatibility ensures that it can be seamlessly incorporated into existing projects without the need to overhaul the entire state management architecture. It acts as a bridge, bringing together different systems under a unified, functional approach.
  - Customization for Various Project Needs: Every project comes with its unique set of requirements and challenges. @ngrx/signals acknowledges this diversity, offering a suite of customization options. Whether you are working on a small-scale application or a large enterprise system, @ngrx/signals can be tailored to fit the specific needs of your project, ensuring that state management is always aligned with your development goals.

## Conclusion
* **Key Points:**
  - In conclusion, @ngrx/signals emerges as a transformative and innovative approach to state management in Angular applications. This library redefines traditional practices by introducing a functional, flexible, and less verbose method of managing state. It aligns seamlessly with Angular's core features, enabling developers to efficiently handle state in both small-scale projects and complex enterprise-level applications.
  - Through its comparison with other state management solutions, @ngrx/signals distinguishes itself as a more accessible and less boilerplate-intensive option, suitable for a wide range of development scenarios. The deep dive into its key components, like signalStore, withState, withComputed, and rxMethod, underscores its versatility and power.
  - @ngrx/signals stands out not just for its technical merits but also for its contribution to a more streamlined and developer-friendly Angular landscape. It offers a compelling option for developers seeking an efficient, modern approach to state management, promising to significantly enhance the Angular development experience.
  - As Angular continues to evolve, @ngrx/signals is poised to play a pivotal role in shaping the future of state management within this framework, making it an essential tool for developers looking to stay at the forefront of Angular application development.