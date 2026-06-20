---
aliases:
  - Change Detection Manual Control
Source 1: https://angular.love/running-change-detection-manual-control
---
# Running change detection – Manual control
## tick
* **Key Points:**
  - Even though Angular runs change detection automatically, sometimes you may need to run change detection manually. This would be the case if a change detector of the component is detached from the component view or updates happen outside of Angular zone.
  - Angular has two ways to manually trigger change detection:
    - calling tick through ApplicationRef
    - calling detectChanges through ChangeDetectorRef
  - The tick method runs change detection for the entire application starting from the root component.
  - Angular uses this method to run application wide change detection when it gets notifications from NgZone about no outstanding microtasks remaining.
  - But the method tick itself doesn't really have anything to do with zones or NgZone. It just invokes change detection for the whole application.
  - If we look under the hood, we can see that it simply iterates all top level (root) views and triggers detectChanges on each view.
  - We can also see that in development mode, tick also runs checkNoChanges that performs a second change detection cycle to ensure that no further changes are detected. If additional changes are picked up during this second cycle, it means that bindings in the app have side-effects that cannot be resolved in a single change detection pass. In this case, Angular throws an error ExpressionChanged error, since an Angular enforces unidirectional data flow.
  - Sometimes you may see that NgZone.run is recommended as way to run change detection globally. But as explained in the Autorun with zones chapter, the run method simply evaluates the callback function inside the Angular zone. There's not explicit call to ApplicationRef.tick() inside. This means that if there's no event notification from Angular zone when the callback has finished executing, change detection will not happen automatically.
* **Technical Entities (Classes/Functions/APIs):** `tick`, `ApplicationRef`, `detectChanges`, `ChangeDetectorRef`, `NgZone`, `checkNoChanges`, `ExpressionChanged`, `ngZone: 'noop'`

## detectChanges
* **Key Points:**
  - This method is available on the change detector service that is created by Angular for each component. It is used to explicitly process change detection and its side-effects for the tree of components starting with the component that you trigger detectChanges() on.
  - This so-called local change detection cycle is useful in many situations besides triggering change detection manually when automatic check is prevented. For example, if you are changing the state in a component with more ancestors than descendants you may get a performance increase by using detectChanges() since you aren't unnecessarily running change detection on the component's ancestors. Another case is detached change detectors.
  - Under the hood detectChanges simply calls refreshView function.
  - As you can see from the code excerpt above, a Change Detector service is basically a shallow wrapper around a component container implemented through LView. When a ViewRef is created for components the LView associated with the component is injected into the constructor. When ViewRef is created for an embedded view, the LView that it receives also describes the embedded view, not the component.
  - Angular implements a different subtype of a ViewRef for each type of views: ViewRef is used for component views, EmbeddedViewRef is used for embedded views and InternalViewRef is used for root/host views.
  - The only difference this time from using ApplicationRef.tick() is that change detection doesn't run for ancestor components, particularly for the root AppComponent.
* **Technical Entities (Classes/Functions/APIs):** `detectChanges`, `ChangeDetectorRef`, `refreshView`, `LView`, `ViewRef`, `EmbeddedViewRef`, `InternalViewRef`

### Surprising ngDoCheck behavior
* **Key Points:**
  - There's an unexpected behaviour related to detectChanges. The ngDoCheck hook is not triggered for the component that you trigger detectChanges on. This happens because lifecycle hooks are executed on child components when checking their parents, not the current component on which the call is made.
  - One of the reasons it's designed like this is to allow manual control of OnPush logic from the ngDoCheck hook. If a child component is defined as onPush, and no input bindings have changed, you still can call markForCheck() from ngDoCheck of this child component to mark the component as dirty.
* **Technical Entities (Classes/Functions/APIs):** `ngDoCheck`, `OnPush`, `markForCheck()`

## markForCheck
* **Key Points:**
  - Change Detector service implements markForCheck method that's very useful but somewhat confusing. In contrast to detectChanges(), it does not run change detection, but explicitly marks the current component (view) and its ancestors as changed (dirty). Next time a change detection cycle runs for any of its ancestors, this marked component view is guaranteed be checked. This means that if a view needs to be synchronously updated before some other action, you'll need to use detectChanges() as calling markForCheck() may not result in a timely update.
  - This markForCheck method is mostly used with components that use OnPush change detection strategy. Components are normally marked as dirty (in need of rerendering) when either values for input bindings have changed or UI related events have fired in the component's template. If none of these two triggers occurred, you need to explicitly call this method to ensure that component will be included in the check next time change detection happens.
  - Since Angular only checks object references, it may not detect objects property input change if the reference remain the same. One solution to this is using immutable objects. Another solution involves performing the required check inside ngDoCheck method and explicitly marking the view as dirty.
  - When the markForCheck method is called, under the hood Angular simply iterates upwards starting from the current component view and enables checks for every parent component up to the root component.
  - The most important part is this assignment: lView[FLAGS] |= LViewFlags.Dirty; That's where the code uses boolean OR to set the LViewFlags.Dirty flag.
  - The idea behind using boolean OR is that resulting bitmask value will be 1 if any of the inputs is 1.
  - Angular checks this flag inside refreshComponent function to determine whether a component requires the check. The function itself is executed from refreshView that's core to change detection.
  - The markForCheck method might also be useful for coalescing and avoiding the error thrown when a new change detection cycle is triggered while being in the middle of the current check cycle. If you can't be sure that this isn't he case, use cd.markForCheck().
  - When changes affect multiple components and you're sure the change detection run will follow, by calling markForCheck instead of detectChanges you're essentially reducing the number of times change detection will be called by coalescing future runs into one cycle.
* **Technical Entities (Classes/Functions/APIs):** `markForCheck()`, `ChangeDetectorRef`, `OnPush`, `ngDoCheck`, `LViewFlags.Dirty`, `refreshComponent`, `refreshView`, `LViewFlags.CheckAlways`
* **Code Snippet:**
```typescript
@Component({
  selector: 'o1-cmp',
  template: '<div *ngFor="let item of items">{{item}}</div>',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class O1 {
  @Input() items = [];
  prevLength = 0;

  constructor(private cdRef: ChangeDetectorRef) {}

  ngDoCheck() {
    if (this.items.length !== this.prevLength) {
      this.prevLength = this.items.length;
      this.cdRef.markForCheck();
    }
  }
}
```