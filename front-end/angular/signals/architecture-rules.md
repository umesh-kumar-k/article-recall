---
aliases:
  - Successful with Signals in Angular – 3 Effective Rules for Your Architecture
Source 1: https://www.angulararchitects.io/blog/successful-with-signals-in-angular-3-effective-rules-for-your-architecture/
---
# Successful with Signals in Angular – 3 Effective Rules for Your Architecture

## Rule 1: Derive State Synchronously Wherever Possible
* **Key Points:**
  - The previously mentioned disadvantages can be compensated for with Signals. Since the introduction of Signals makes the component reactive, OnPush can now be activated. In addition, the component can derive its state from the individual Signals using computed synchronously.
  - This makes the code a lot more straightforward: The loadRatings method simply loads the ratings and places them in a signal. The computed Signal ratedDesserts takes care of merging desserts and ratings. No matter when and where the application updates desserts or ratings, ratedDesserts is always up to date.
  - When applying this pattern, it is important to note that computed can only derive state in a synchronous manner. For an asynchronous derivation, Angular provides the Resource API.
* **Technical Entities (Classes/Functions/APIs):** `OnPush`, `computed()`, `Resource API`
* **Code Snippet:**
```typescript
@Component({
  […],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class DessertsComponent implements OnInit {
  #dessertService = inject(DessertService);
  #ratingService = inject(RatingService);

  originalName = signal('');
  englishName = signal('');
  loading = signal(false);

  desserts = signal<Dessert[]>([]);
  ratings = signal<DessertIdToRatingMap>({});
  ratedDesserts = computed(() => this.toRated(this.desserts(), this.ratings()));

  [...]

  loadRatings(): void {
    this.loading.set(true);

    this.#ratingService.loadExpertRatings().subscribe({
      next: (ratings) => {
        this.ratings.set(ratings);
        this.loading.set(false);
      },
      error: (error) => { […] }
  });

  […]
}
```

## Rule 2: Avoid Effects for Propagating State
* **Key Points:**
  - Effects are the right choice when presenting values cannot be achieved via data binding. However, they bring some pitfalls when used for propagating change.

### Proper Usage of Effects
* **Key Points:**
  - In most cases, Signals are bound in the template. However, it happens that the desired form of output cannot be achieved via data binding. An example of this is outputting a Signal to the console for debugging purposes. Another example are toasts that can be activated via services and are intended to present the value of a Signal. For these cases, Angular provides Effects.
* **Technical Entities (Classes/Functions/APIs):** `effect()`

### Signals are Glitch-free
* **Key Points:**
  - When writing code like in the previous section, we need to be aware that Signals are glitch-free. That means that if you change a signal several times in a row (within a stack frame), only the last change will be seen by the consumer, e.g. the effect.
  - This shows that Signals are not intended for modelling events but for data we want to bind to the view. In the latter case, we just need the current value while binding intermediate values would be counterproductive. For this reasons, the effects shown in the previous section are only triggered once even if there are several changes in a row.
  - In cases where we want to express events, Observables are the way to go, as they don't have this glitch-free guarantee by design.
* **Technical Entities (Classes/Functions/APIs):** `effect()`

### Problematic Use of Effects
* **Key Points:**
  - Even when using Effects, Signals are primarily used to transport the desired data into the view. In theory, however, Effects could also be used to transmit state to other Signals.
  - However, such approaches have several disadvantages, which is why Angular prohibits writing Signals within Effects by default.
  - One of these disadvantages is that unmanageable change cascades and thus difficult-to-maintain code and cyclic dependencies can arise. Since Effects register implicitly with all Signals used, the associated problems may not even be noticeable at first glance. If you still want to use Effects for writing, you can make Angular let things slide by setting allowSignalWrites.
  - The general consensus in the community is that application code should only use allowSignalWrites as a last resort. On the other hand, libraries like NGRX use this option internally. In this case, however, the authors of the library are responsible for its correct use, so application developers don't have to worry about it.
  - It is also important to note that the Effect itself registers with Signals in called methods too. This leads to a further increase in complexity. At least this problem could be alleviated with built-in features: The untracked function avoids the current reactive context spilling over to the called search method. Angular now also uses this pattern itself in selected cases. An example of this is triggering events in sub-components so that the event handler does not run in the reactive context of the code that triggered the event. Further popular libraries that use this technique are NGRX, NGRX Signal Store or ngextensions.
* **Technical Entities (Classes/Functions/APIs):** `allowSignalWrites`, `untracked()`, `NGRX`, `NGRX Signal Store`, `ngextensions`

### Strategies for Preventing Effects With Signal Writes
* **Key Points:**
  - Effects that spread data via Signal writes can in many cases be prevented using the following approaches:
    - Consistent derivation of state using computed (see rule 1, above) or the Resource API
    - Direct use of events that caused the Signal to change.
  - Instead of calling search, as indicated above, in an Effect, the application could instead use the change event of the input fields for the search filters. Observables can also be used as a source for such actions. The search method could, for example, also be triggered by the valueChanges observable of a FormGroup. In cases where you have just Signals, they can be converted into Observables using the RxJS Interop offered by Angular.
  - The use of Observables has several advantages at this point:
    - In contrast to Signals, Observables are also suitable for triggering asynchronous actions.
    - toObservable function strips the current reactive context using untracked.
    - RxJS comes with a lot of powerful operators, like debounceTime.
    - The flattening operators offered by RxJS provide guarantees for overlapping asynchronous actions and thus prevent race conditions. In the example shown, switchMap ensures that when search queries overlap, only the result of the last one is used and all others are canceled.
  - In many cases, one could argue that instead of converting a Signal into an Observable, it would be more appropriate to directly use the event that led to the Signal change, as proposed above. On the other hand, as Angular APIs increasingly adopt Signal-based approaches, using them directly will likely become more convenient and feel more intuitive. Therefore, this appears to be a gray area where we need to be mindful of the consequences, such as those associated with the glitch-free guarantee of Signals.
* **Technical Entities (Classes/Functions/APIs):** `computed()`, `Resource API`, `valueChanges`, `toObservable`, `untracked()`, `debounceTime`, `switchMap`

## Rule 3: Stores Simplify Reactive Data Flow
* **Key Points:**
  - Stores like the classic NGRX Store or the lightweight NGRX Signal Store not only take care of state management, but also help to keep the reactive data flow manageable.
  - The application forwards its intention to the store as part of an event. At this point, I use the term intention in an abstract, technology-neutral way, especially since different stores realize this aspect differently. With Redux and therefore also when using the classic NGRX store the application sends an action to the store, which forwards it to Reducer and Effects. For lightweight stores like the NGRX Signal Store, the application delegates to a method offered by the store instead.
  - Offloading asynchronous operations to the store also compensates for the fact that Signals are currently only designed for synchronous actions.
  - The store then takes action and initiates synchronous or asynchronous operations. If the application uses RxJS for this, race conditions can be avoided with the flattening operators, as mentioned above. Offloading asynchronous operations to the store also compensates for the fact that Signals are currently only designed for synchronous actions.
  - The result of these operations leads to a change in the state managed by the store. This state can be expressed by Signals, which can be mapped to other Signals using computing (see rule 1). Such mappings can occur both in the store and in the component (or in another consumer of the store). This depends on how local or global the store and the data to be derived are.
  - The bottom line is that the consistent use of this approach supports the so-called unidirectional data flow, which makes system behavior more understandable.
* **Technical Entities (Classes/Functions/APIs):** `NGRX Store`, `NGRX Signal Store`, `Redux`, `Reducer`, `Effects`

## Summary
* **Key Points:**
  - To truly leverage the benefits of Signals, the application must be designed as a reactive system. This means, among other things, that writing values is avoided in favor of deriving values from existing ones. This simplifies the program code, especially since derived values are automatically kept up to date.
  - Signals are currently primarily suitable for transporting data into the view. Effects are used when API calls are necessary for that, e.g. when displaying a toast. The current Signals implementation is not intended to trigger asynchronous actions. Instead, classic events or observables are the way to go. Stores that can also handle asynchronous operations help establish unidirectional data flow and make reactive applications more manageable.