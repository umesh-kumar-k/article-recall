---
aliases:
  - "Signals in Angular: Building Blocks"
Source 1: https://www.angulararchitects.io/blog/angular-signals/
---
# Signals in Angular: Building Blocks

## Using Signals
* **Key Points:**
  - For using Signals with data binding, properties to be bound are expressed as signals.
  - It should be noted here that a Signal always has a value. Therefore, a default value must be passed to the signal function. If the data type cannot be derived from this, the example specifies it explicitly via a type parameter.
  - The Signal's getter is used to read the value of a signal. Technically, this means that the signal is called like a function.
  - To set the value, the signal offers an explicit setter in the form of a set method.
  - Method calls were frowned upon in templates in the past, especially since they could lead to performance bottlenecks. However, this does not generally apply to uncomplex routines such as getters. In addition, the template will appear here as a consumer, and as such, it can be notified of changes.
  - Meanwhile, Angular's Two-Way-Bindings directly support Signals.
  - In a future version, the Angular team will adopt the forms handling to Signals.
* **Technical Entities (Classes/Functions/APIs):** `signal()`, `set()`, `update()`, `Two-Way-Bindings`, `[(ngModel)]`
* **Code Snippet:**
```typescript
@Component([…])
export class FlightSearchComponent {

  private flightService = inject(FlightService);

  from = signal('Hamburg');
  to = signal('Graz');
  flights = signal<Flight[]>([]);

  […]

  async search(): Promise<void> {
    if (!this.from() || !this.to()) {
      return;
    }
    const flights = await this.flightService.findAsPromise(this.from(), this.to());
    this.flights.set(flights);
  }
}
```

## Updating Signals
* **Key Points:**
  - In addition to the setter shown earlier, Signals also provide an update method for projecting the current value to a new one.
* **Technical Entities (Classes/Functions/APIs):** `update()`

## Signal Values Should to be Immutable
* **Key Points:**
  - By default, a Signal's is supposed to be immutable. For this reason, just updating the flight date in the previous section would not be sufficient. Instead, we have to clone the changed parts, so that they get a new object reference.
  - By comparing these references, Angular's OnPush change detection strategy can efficiently determine the changed parts of an object managed by a Signal.

## Calculated Values, Side Effects, and Assertions
* **Key Points:**
  - Some values are derived from existing values. Angular provides calculated signals for this.
  - Such a signal is read-only and appears as both a consumer and a producer. As a consumer, it retrieves the values of the signals used - here from and to - and is informed about changes. As a producer, it returns a calculated.
  - If you want to consume signals programmatically, you can use the effect function.
  - The effect function executes the transferred lambda expression and registers itself as a consumer for the signals used. When one of these signals change, the the effect is triggered again.
  - Please note that usually, Signals are consumed via data binding. However, in some cases, we need a function or method call in order to present a Signal's value to the user. Examples are logging or displaying a toast message.
* **Technical Entities (Classes/Functions/APIs):** `computed()`, `effect()`

## Injection Context needed
* **Key Points:**
  - Several building blocks for Signals such as effect can only be used in an injection context. This is everywhere inject is allowed: in the constructor, as default values of class fields, and in provider factories. Also, you can use the runInInjectionContext function to run code in an injection context.
  - Hence, the effect set up in the constructor above would fail when placed in ngOnInit or in another method.
  - In this case, you'd get the following error: ERROR Error: NG0203: effect() can only be used within an injection context such as a constructor, a factory function,
  - The technical reason is that effects use inject to get hold of the current DestroyRef. This service provided since Angular 16 helps to find out about the life span of the current building block, e. g. the current component or service. The effect uses the DestroyRef to "unsubscribe" itself when this building block is about to be destroyed.
  - For this reason, you would typically setup your effects in the constructor as shown in the last section. If you really want to setup an effect somewhere else, you can go with the runInInjectionContext function. However, it needs a reference to an Injector.
* **Technical Entities (Classes/Functions/APIs):** `runInInjectionContext`, `DestroyRef`, `Injector`

## Glitch-Free Property
* **Key Points:**
  - If a signal changes several times in a row or if several signals change one after the other, undesired interim results could occur.
  - Like many other Signal implementations, Angular's implementation prevents such occurrences. The Angular team's readme, which also explains the push/pull algorithm used, calls this desirable assurance "glitch-free".

## Signals and Change Detection
* **Key Points:**
  - Similar to an Observable bound to a template using the async-Pipe, a bound Signal triggers change detection. This also works for the more efficient OnPush mode.
  - However, to help Angular in OnPush mode to also find out about child components to look at, you have to use Immutables, as discussed above.
* **Technical Entities (Classes/Functions/APIs):** `OnPush`, `ChangeDetectionStrategy.OnPush`

## RxJS Interop
* **Key Points:**
  - Admittedly, at first glance, signals are very similar to a mechanism that Angular has been using for a long time, namely RxJS Observables. However, signals are deliberately kept simpler.
  - If you need the power of RxJS and its operators, you can however convert them to Observables. The namespace @angular/core/rxjs-interop contains a function toObservable converting a Signal into an Observable and a function toSignal for the other way round. They allow using the simplicity of signals with the power of RxJS.
  - The initialValue passed to toSignal is needed because Signals always have a value. On the contrary, Observables might not emit a value at all. If you are sure your Observable has an initial value, e.g., because it's a BehaviorSubject or because of using the startsWith operator, you can also set requireSync to true.
  - If you neither set initialValue nor requireSync, the type of the returned Signal also supports the undefinied type, allowing an initial value of undefined. Consequently, your code has to check for undefined too.
* **Technical Entities (Classes/Functions/APIs):** `toObservable`, `toSignal`, `@angular/core/rxjs-interop`, `BehaviorSubject`, `startsWith`, `requireSync`
* **Code Snippet:**
```typescript
@Component([...])
export class FlightSearchComponent {
  private flightService = inject(FlightService);

  from = signal('Hamburg');
  to = signal('Graz');
  basket = signal<Record<number, boolean>>({ 1: true });
  flightRoute = computed(() => this.from() + ' to ' + this.to());

  from$ = toObservable(this.from);
  to$ = toObservable(this.to);

  flights$ = combineLatest({ from: this.from$, to: this.to$ }).pipe(
    filter(c => c.from.length >= 3 && c.to.length >= 3),
    debounceTime(300),
    switchMap(c => this.flightService.find(c.from, c.to))
  );

  flights = toSignal(this.flights$, {
    initialValue: []
  });
}
```

## Conclusion
* **Key Points:**
  - Signals make reactivity in Angular more lightweight. They make usual use cases simple and for more complex use cases we can bridge over to RxJS using a first-class interop implementation.
  - The Angular team remains true to itself: Signals are not hidden in the substructure or behind proxies but made explicit. Developers therefore always know which data structure they are actually dealing with. Also, signals are just an option. No one needs to change legacy code and a combination of traditional change detection and signal-based change detection will be possible.