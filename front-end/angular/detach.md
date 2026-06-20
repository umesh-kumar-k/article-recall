---
aliases:
  - detach() / Full Manual Control
highlights: Detach component from CD tree; run detectChanges() explicitly; use for high-frequency data(real time charts trading feeds)
Source 1: https://angular.love/running-change-detection-detached-views
---
# Running change detection – Detached views

## Detached views
* **Key Points:**
  - In the chapter manual control we learnt how and where to use first 3 methods of a change detector service: detectChanges, checkNoChanges and markForCheck. In this chapter we'll explore the remaining two defined on the interface: detach and reattach.
  - The detach method simply updates the LViewFlags.Attached flag on a component view. It sets it to 0 using bitwise NOT (~) operator.
  - This flag is checked during change detection to determine if a component should be checked.
  - You could probably guess that reattach simply resets the LViewFlags.Attached to 1.
  - Imagine you have a component tree. We can detach A component using ChangeDetectorRef. Once we do that, A component won't be checked. But because it won't be checked, neither will its children, so the entire left branch starting with A will be skipped. It's worth stressing again – even though we detached component A, its child components A1 and A2 will not be checked as well. This means that even if expressions change in templates of these components we won't see it any updates on the screen.
* **Technical Entities (Classes/Functions/APIs):** `detach()`, `reattach()`, `ChangeDetectorRef`, `LViewFlags.Attached`, `viewAttachedToChangeDetector()`, `refreshComponent`

## Local checks
* **Key Points:**
  - What might be unexpected is that calling detectChanges will run change detection for the current component regardless of its state. This means that you can create local change detection by detaching the component from the main change detection tree and use detectChanges to run the check on demand.
  - In this example we detach the component from the main change detection tree and update the changed property. The property is updated within one second, but the component is not checked, so the change is not reflected on the screen. After two seconds we call detectChanges to run change detection locally. This will update the screen with the new value.
  - When we run detectChanges for a detached component, both its child components and embedded views will be checked as well.
* **Technical Entities (Classes/Functions/APIs):** `detectChanges()`, `ChangeDetectorRef`

## Use cases
* **Key Points:**
  - Angular docs describe a very interesting use case for the detach and reattach method to implement local change detection: The following example defines a component with a large list of readonly data. Imagine, the data changes constantly, many times per second. For performance reasons, we want to check and update the list every five seconds. We can do that by detaching the component's change detector and doing a local change detection check every five seconds.
  - When we click on a checkbox, the input binding on the LiveData component is updated with the corresponding value of true or false. If true, the setter logic attaches the component view to the change detection tree so that it's checked during the next global change detection run. If false, the view is detached.
  - This would also work with ngOnChanges hook which is still triggered for LiveData even if its component view is detached.
  - Here's the important observation. The reattach method enables checks only for the current component, but if changed detection is not enabled for its parent component, it will have no effect. It means that the approach with using either OnChanges or input setters to attach view will only work for the top-most component in the detached branch. It won't work for nested components in the disabled branch because the change detection won't run for their parent component.
  - We could implement similar version but using local change detection. The following example only re-renders for even values emitted by dataProvider.
* **Technical Entities (Classes/Functions/APIs):** `detach()`, `reattach()`, `ngOnChanges`, `ChangeDetectorRef`
* **Code Snippet:**
```typescript
@Component({
  selector: 'live-data-a',
  template: 'Data: {{dataProvider.data}}',
})
export class LiveDataA {
  constructor(
    private cdRef: ChangeDetectorRef,
    public dataProvider: DataProvider
  ) {
    this.cdRef.detach();

    setInterval(() => {
      if (dataProvider.data % 2 === 0) {
        this.cdRef.detectChanges();
      }
    });
  }
}
```