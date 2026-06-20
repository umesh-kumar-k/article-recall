---
aliases:
  - Deep dive into the OnPush change detection strategy in Angular
Source 1: https://angular.love/deep-dive-into-the-onpush-change-detection-strategy-in-angular
---
# Deep dive into the OnPush change detection strategy in Angular

## OnPush aka CheckOnce strategy
* **Key Points:**
  - Angular implements two strategies that control change detection behavior on the level of individual components. Those strategies are defined as Default and OnPush.
  - Angular uses these strategies to determine whether a child component should be checked while running change detection for a parent component. A strategy defined for a component impacts all child directives since they are checked as part of checking the host component. A defined strategy cannot be overridden in runtime.
  - The default strategy, internally referred to as CheckAlways, implies regular automatic change detection for a component unless the view is explicitly detached. What's known as OnPush strategy, internally referred to as CheckOnce, implies that change detection is skipped unless a component is marked as dirty. Angular implements mechanisms to automatically mark a component as dirty. When needed, a component can be marked dirty manually using markForCheck method exposed on ChangeDetectorRef.
  - When we define a strategy using @Component() decorator, Angular's compiler records it in a component's definition through the defineComponent function.
  - When Angular instantiates a component, it's using this definition to set a corresponding flag on the LView instance that represents the component's view. This means that all LView instances created for this component will have either CheckAlways or Dirty flag set. For the OnPush strategy the Dirty flag will be automatically unset after the first change detection pass.
  - The flags set on LView are checked inside the refreshView function when Angular determines whether a component should be checked.
* **Technical Entities (Classes/Functions/APIs):** `ChangeDetectionStrategy.OnPush`, `ChangeDetectionStrategy.Default`, `CheckAlways`, `CheckOnce`, `markForCheck`, `ChangeDetectorRef`, `@Component()`, `defineComponent`, `LView`, `refreshView`, `LViewFlags.CheckAlways`, `LViewFlags.Dirty`

## Default strategy
* **Key Points:**
  - Default change detection strategy means that a child component will always be checked if its parent component is checked. The only exception to that rule is that if you detach a change detector of the child component.
  - Note that I specifically highlighted the part about the parent component being checked. If a parent component isn't checked, Angular won't run change detection for the child component even if it uses default change detection strategy. This comes from the fact that Angular runs check for a child component as part of checking its parent.
  - Angular doesn't enforce any workflows on developers for detecting when a component's state changed, that's why the default behavior is to always check components. An example of the enforced workflow is object immutability that's passed through @Input bindings. This is what's used for OnPush strategy.
  - While the reference to the user object hasn't changed, it has been mutated inside but we can still see the new name rendered on the screen. That is why the default behavior is to check all components. Without the object immutability restriction in place Angular, can't know if inputs have changed and caused an update to the component's state.
* **Technical Entities (Classes/Functions/APIs):** `detach()`, `ChangeDetectorRef`

## OnPush aka CheckOnce strategy
* **Key Points:**
  - While Angular does not force object immutability on us, it gives us a mechanism to declare a component as having immutable inputs to reduce the number of times a component is checked. This mechanism comes in the form of OnPush change detection strategy and is a very common optimization technique. Internally this strategy is called CheckOnce, as it implies that change detection is skipped for a component until it's marked as dirty, then checked once, and then skipped again. A component can be marked dirty either automatically or manually using markForCheck method.
  - Here's the list of different scenarios that Angular uses to test OnPush behavior:
    - should skip OnPush components in update mode when they are not dirty
    - should not check OnPush components in update mode when parent events occur
    - should check OnPush components on initialization
    - should call doCheck even when OnPush components are not dirty
    - should check OnPush components in update mode when inputs change
    - should check OnPush components in update mode when component events occur
    - should check parent OnPush components in update mode when child events occur
    - should check parent OnPush components when child directive on a template emits event
  - The last batch of test scenarios ensures that automatic process of marking a component dirty occurs in the following scenarios:
    - an @Input reference is changed
    - a bound event is received triggered on the component itself
* **Technical Entities (Classes/Functions/APIs):** `OnPush`, `CheckOnce`, `markForCheck`, `@Input`, `ngDoCheck`

### @Input() bindings
* **Key Points:**
  - In most situations we will need to check the child component only when its inputs change. This is especially true for pure presentational components whose input comes solely through bindings.
  - As we have seen above, when we click on the button and change the name in the callback, the new name is not updated on the screen. That's because Angular performs shallow comparison for the input parameters and the reference to the user object hasn't changed. Mutating an object directly doesn't result in a new reference and won't mark the component dirty automatically.
  - We have to change the reference for the user object for Angular to detect the difference in @Input bindings. If we create a new instance of user instead of mutating the existing instance, everything will work as expected.
* **Technical Entities (Classes/Functions/APIs):** `@Input`, `Object.freeze`, `immer`, `produce`

### Bound UI events
* **Key Points:**
  - All native events, when triggered on a current component, will mark dirty all ancestors of the component up to the root component. The assumption is that an event could trigger change in the components tree. Angular doesn't know if the parents will change or not. That is why Angular always checks every ancestor component after an event has been fired.
  - If we attach an event listener inside the TodoComponent template, Angular marks dirty all ancestor components before it runs the event handler.
  - Hence the hierarchy of components is marked for the check once looks like this: Root Component -> LViewFlags.Dirty, ContentComponent -> LViewFlags.Dirty, TodoListComponent -> LViewFlags.Dirty, TodoComponent (event triggered here) -> markViewDirty() -> LViewFlags.Dirty.
  - During the next change detection cycle, Angular will check the entire tree of TodoComponent's ancestor components. Note that the HeaderComponent is not checked because it is not an ancestor of the TodoComponent.
* **Technical Entities (Classes/Functions/APIs):** `LViewFlags.Dirty`, `markViewDirty()`

### Manually marking components as dirty
* **Key Points:**
  - Let's come back to the example where we changed the reference to the user object when updating the name. This enabled Angular to pick up the change and mark B component as dirty automatically. Suppose we want to update the name but don't want to change the reference. In that case, we can mark the component as dirty manually.
  - For that we can inject changeDetectorRef and use its method markForCheck to indicate for Angular that this component needs to be checked.
  - What can we use for someMethodWhichDetectsAndUpdate? The NgDoCheck hook is a very good candidate. It's executed before Angular will run change detection for the component but during the check of the parent component. This is where we'll put the logic to compare values and manually mark component as dirty when detecting the change.
  - The design decision to run NgDoCheck hook even if a component is OnPush often causes confusion. But that's intentional and there's no inconsistency if you know that it's run as part of the parent component check. Keep in mind that ngDoCheck is triggered only for top-most child component. If the component has children, and Angular doesn't check this component, ngDoCheck is not triggered for them.
  - Don't use ngDoCheck to log the checking of the component. Instead, use the accessor function inside the template like this {{ logCheck() }}.
  - Remember that markForCheck neither triggers nor guarantees change detection run.
* **Technical Entities (Classes/Functions/APIs):** `ChangeDetectorRef`, `markForCheck()`, `NgDoCheck`, `ngDoCheck`

### Observables as @Inputs
* **Key Points:**
  - Now, let's make our example a bit more complex. Let's assume our child B component takes an observable based on RxJs that emits updates asynchronously. This is similar to what you might have in NgRx based architecture.
  - So we receive this stream of user objects in the child B component. We need to subscribe to the stream, check if the value is updated and mark the component as dirty if needed.
  - The logic inside the ngOnChanges is almost exactly what the async pipe is doing. That's why the common approach is to delegate the subscription and comparison logic to async pipe. The only restriction is that objects must be immutable.
  - This test is for the use case we explored here: the pipe subscribes to the observable inside the transform, and when the observable emits a new message, it marks the component as dirty.
* **Technical Entities (Classes/Functions/APIs):** `async pipe`, `ngOnChanges`, `markForCheck()`, `BehaviorSubject`