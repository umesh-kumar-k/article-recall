---
aliases:
  - "Angular Signals & Your Architecture: 5 Options"
Source 1: https://www.angulararchitects.io/blog/angular-signals-your-architecture-5-options/
---
# Angular Signals & Your Architecture: 5 Options

## Option 1: Signals in Component
* **Key Points:**
  - A straightforward approach is to use Signals in your components directly. Each property you want to data bind becomes a Signal.
  - Then, you can directly bind these signals in the template.
  - As of writing this, ngModel and other parts of the FormModule don't support two-way bindings for Signals. The Angular team will take care of a revision of forms support soon. For this reason, the previous example implements two-way binding manually by explicitly setting up a property binding for ngModel and an event binding for ngModelChange.
  - In a traditional component, binding a Signal in a template is similar to binding an Observable with the async pipe. Hence, we can also turn on OnPush.
  - Angular checks OnPush components when the async pipe provides a new value. Since Angular 16, the same is the case when a bound Signal is updated. Assigning a new value to the flights Signal makes Angular update the FlightSearchComponent. With Angular 17, we even got first performance improvements for data binding with Signals.
  - However, this is only one side of the coin: Angular also has to find out which child components need to be updated. It checks the values bound to the child components' properties. Only the object reference is checked when binding complex data types like objects or arrays.
  - This is why Observables are usually used together with immutable data structures (Immutables). Instead of mutating such objects, a new object with the changed values is created. For Signals it's the same: We need to use them with Immutables. While originally it was planned to also support mutable data structure, the Angular team decided to enforce Immutability for now to address the discussed challenge.
* **Technical Entities (Classes/Functions/APIs):** `signal()`, `OnPush`, `ChangeDetectionStrategy.OnPush`, `ngModel`, `ngModelChange`, `async pipe`
* **Code Snippet:**
```typescript
@Component({ ... })
export class FlightSearchComponent  {

  private flightService = inject(FlightService);

  from = signal('Hamburg');
  to = signal('Graz');
  flights = signal<Flight[]>([]);

  async search() {
    if (!this.from() || !this.to()) return;

    const flights = await this.flightService.findPromise(this.from(), this.to());
    this.flights.set(flights);
  }

  [...]
}
```

## Option 2: Move Signals to a Service
* **Key Points:**
  - Signals are not limited to be used in components. By design, they can be used everywhere. Hence, we can also move them into a service.
  - Such services can be used by several components and hence allow for sharing state. But even if only used by one component, they provide some added value: One can relieve the component from taking care of state management, and the service can preserve the state when the component is destroyed and re-created, e.g., when leaving a route and returning to it.
* **Technical Entities (Classes/Functions/APIs):** `signal()`, `@Injectable({ providedIn: 'root' })`

## Option 3: Information Hiding With Services
* **Key Points:**
  - If several components use a service, it might be a good idea to make the exposed Signals read/only. This forces all the components to update the state in a well-defined way, e.g., by calling service methods intended for the update in question.
  - For creating a read/only Signal out of a writable one, the method asReadonly can be used.
  - This style is also known from RxJS where writable Subjects are converted to read/only Observables. Unfortunately, this code is quite lengthy.
* **Technical Entities (Classes/Functions/APIs):** `asReadonly()`, `signal()`

## Option 3a: Service with State-Signal
* **Key Points:**
  - To streamline option 3 a bit, we could introduce a private writable state Signal and derive the individual read/only Signals from it via computed.
  - This already looks like the usage of a lightweight store like the NGRX Component Store. The used helper function patchState does indeed borrow one of the Component Store's ideas.
  - Of course, store libraries provide additional features. In particular, the NGRX Signal Store is really tempting.
* **Technical Entities (Classes/Functions/APIs):** `computed()`, `patchSignal()`, `NGRX Component Store`, `NGRX Signal Store`

## Option 4: Using a Store and the Redux Pattern
* **Key Points:**
  - We could also use a store library as an alternative to manage the state in a service by hand. Choosing one that follows the Redux pattern ensures the state is only modified in a well-defined way. The most popular Redux implementation for Angular is NGRX. Fortunately, it has been supporting Signals since version 16.
  - The following example shows how its new selectSignal method allows retrieving a part of the immutable state tree as a Signal.
* **Technical Entities (Classes/Functions/APIs):** `NGRX`, `Store`, `selectSignal()`, `dispatch()`

## Option 5: Hiding the Store Behind a Facade
* **Key Points:**
  - Sometimes, hiding the used store implementation behind a service can be beneficial. Such a so-called facade provides a domain-specific interface and allows to introduce the store gradually or to use it selectively.
  - If you know from the beginning that you will go all-in with your store, wrapping it in such a facade might be overhead.

## Conclusion
* **Key Points:**
  - Signals can be defined directly within components but also within a service. In the latter case, public read/only Signals can be derived from a private writable Signal. By providing methods for manipulating the state, one can ensure that the state is only modified in a well-defined way. Instead of hand-writing such a service, existing store libraries adapted for Signals can be used. A famous example is NGRX.
  - Also, it will be interesting to see how store libraries will evolve to better support Signals and the upcoming Signal Components. A promising development is the envisioned NGRX Signal Store.