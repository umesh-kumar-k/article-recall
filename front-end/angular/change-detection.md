---
aliases:
  - Change Detection
  - Default vs OnPush
highlights: Default checks entire tree on every aynch event; OnPush checks only when @Input() reference changes, async pipe emits or markForCheck() called
Source 1: https://www.angularspace.com/how-angular-keeps-your-ui-in-sync/
Source 2: https://angular.love/deep-dive-into-the-onpush-change-detection-strategy-in-angular
Source 3: https://www.thinktecture.com/en/angular/whats-the-hype-onpush/
Source 4: https://angular.love/complete-guide-angular-lifecycle-hooks
Source 5: https://angular.love/a-gentle-introduction-into-change-detection-in-angular
Source 6: https://angular.love/change-detection-big-picture-overview
Source 7: https://angular.love/change-detection-big-picture-operations
Source 8: https://angular.love/change-detection-big-picture-rendering-cycle
---

# Change Detection Big Picture – Overview

## Overview
* **Key Points:**
  - In any application, the process of updating the screen takes the internal state of the program and projects it into something users can see on the screen. In web development, we take data structures like objects and arrays and end up with a DOM representation of that data in the form of images, buttons and other visual elements. Frameworks take care of synchronization between the internal state of the application and the underlying platform. But not only do they take some weight off our shoulders, they do the state tracking and DOM updates very efficiently.
  - This mechanism of synchronization is called rendering. It has different names – change detection in Angular or reconciliation in React – but the core operations it performs are similar across different implementations. Change detection is one of the most important pieces of an architecture since it's responsible for the updates of the platform model (DOM) responsible for visible parts. It's also the area that significantly affects an application's performance. Going forward I'll be using the terms rendering and change detection interchangeably.
  - To define the relationship between UI and application state we use expressions in templates. In the templates below, we're saying that the DOM property className depends on the component's property rating. So whenever the rating changes, the expression should be re-evaluated. If a change is detected, the className property should be updated.
  - In Angular, when the compiler analyzes the template, it identifies properties of a component that are associated with DOM elements. For each such association, the compiler creates a binding. A binding defines a relationship between a component's property (usually wrapped in some expression) and the DOM element property. The change detection mechanism executes instructions that process bindings. The job of these instructions is to check if the value of an expression with a component property has changed and perform DOM updates if necessary.
  - Processing bindings that perform dirty checks and update the relevant parts of the DOM are the core operations of change detection in Angular.
* **Technical Entities (Classes/Functions/APIs):** `change detection`, `rendering`, `reconciliation`, `DOM`, `binding`, `instructions`

## DOM updates overview
* **Key Points:**
  - For this part of the template, the compiler generates instructions that set up a binding, performs dirty checks and update the DOM. Here we can see the instruction property (prefixed with ɵɵ in the sources) that checks if the value of the expression has changed, and if so marks the binding as dirty and updates the value. Angular created the binding for the className and the current values of the binding is 'fa-star far'. Once the rating property in the component is updated, Angular runs change detection and processes the instructions.
  - First we get a reference to the LView, which is a container for Angular components and all relevant data. Then using nextBindingIndex we retrieve the index in the LView that holds information about the binding on the propName. In our case it's the className property.
  - The bindingUpdated function evaluates the expression and uses it to compare to the previous value remembered by the binding. This is where the name "dirty checking" comes from. If the value has changed, it updates the current value and returns true.
  - If the expression changed, Angular runs elementPropertyInternal function that uses the new value to update the DOM. In our case it will update the className property of the list item.
  - The property function returns itself so that it may be chained.
* **Technical Entities (Classes/Functions/APIs):** `property (ɵɵproperty)`, `bindingUpdated`, `elementPropertyInternal`, `LView`, `nextBindingIndex`, `tView`, `tNode`

## Triggering change detection
* **Key Points:**
  - There are two ways to initiate change detection. The first one is to explicitly tell the framework that something has changed or there's a possibility of a change so it should run change detection. Basically, in this way we initiate change detection manually. The second way is to rely on the external mechanism to know when there's a possibility of a change and run change detection automatically.
  - In Angular we have both options. We can use the Change Detector service to run change detection manually.
  - However, we can also rely on the framework to trigger change detection automatically. In this way, we simply update the property on a component.
  - Angular somehow needs to know that the property is updated and it should run change detection. To solve this Angular uses a library called zone.js. It patches all asynchronous events in a browser and can then notify Angular when a certain event occurs. Similarly to UI events, Angular can then wait until the application code has finished executing and initiate change detection automatically.
* **Technical Entities (Classes/Functions/APIs):** `ChangeDetectorRef`, `detectChanges()`, `zone.js`

## Order of checks
* **Key Points:**
  - Angular runs change detection for each component in the depth-first order. This means for a component tree depicted below the checks will happen in the following order A, K, L, J, O, C and so on.
  - Since a component check includes multiple operations, the order of executing certain steps may produce a bit different results. For example, operations that result in DOM and binding updates are executed in the proper depth-first order.
  - But if I add logging into the ngDoCheck hook, this is the order you will see: Angular checks A, K and then V, L and then C and so on.
  - This is neither a depth-first nor a proper breadth-first algorithm. The conventional implementation of the breadth-first algorithm checks all siblings on the same level, whereas in the diagram above as you can see the algorithm indeed checks L and C sibling components, but instead of checking X and F it goes down to J and O.
* **Technical Entities (Classes/Functions/APIs):** `ngDoCheck`, `logRender()`

---

## Change Detection Big Picture – Operations

## Operations
* **Key Points:**
  - When Angular runs change detection for a particular component (view) it performs a number of operations. Those operations are sometimes referred to as side effects, as in a side effect of the computation logic.
  - In Angular, the primary side effect of change detection is rendering application state to the target platform. Most often the target platform is a browser, application state has the form of a component properties and rendering involves updating the DOM.
  - When checking a component Angular runs a few other operations.
* **Technical Entities (Classes/Functions/APIs):** `refreshView`, `executeTemplate`, `executeCheckHooks`, `markTransplantedViewsForRefresh`, `refreshEmbeddedViews`, `refreshContentQueries`, `processHostBindingOpCodes`, `refreshChildComponents`, `executeViewQueryFn`, `RenderFlags.Update`, `LViewFlags`, `LViewFlags.Dirty`, `LViewFlags.FirstLViewPass`, `LViewFlags.RefreshTransplantedView`, `ViewContainerRef`

### Core Operations in Order
* **Key Points:**
  - Here's the list of such operations in the order specified:
    - executing a template function in update mode for the current view
    - checks and updates input properties on a child component/directive instance
    - execute the hooks on a child component ngOnInit, ngDoCheck and ngOnChanges if bindings changed
    - updates DOM interpolations for the current view if properties on current view component instance changed
    - executeCheckHooks if they have not been run in the previous step
      - calls OnChanges lifecycle hook on a child component if bindings changed
      - calls ngDoCheck on a child component (OnInit is called only during first check)
    - markTransplantedViewsForRefresh - find transplanted views that need to be refreshed down the LView chain and mark them as dirty
    - refreshEmbeddedViews - runs change detection for views created through ViewContainerRef APIs (mostly repeats the steps in this list)
    - refreshContentQueries - updates ContentChildren query list on a child view component instance
    - execute Content CheckHooks (AfterContentInit, AfterContentChecked) - calls AfterContentChecked lifecycle hooks on child component instance (AfterContentInit is called only during first check)
    - processHostBindingOpCodes - checks and updates DOM properties on a host DOM element added through @HostBinding() syntax inside the component class
    - refreshChildComponents - runs change detection for child components referenced in the current component's template. OnPush components are skipped if they are not dirty
    - executeViewQueryFn - updates ViewChildren query list on the current view component instance
    - execute View CheckHooks (AfterViewInit, AfterViewChecked) - calls AfterViewChecked lifecycle hooks on child component instance (AfterViewInit is called only during first check)
* **Technical Entities (Classes/Functions/APIs):** `ngOnInit`, `ngDoCheck`, `ngOnChanges`, `OnChanges`, `AfterContentInit`, `AfterContentChecked`, `AfterViewInit`, `AfterViewChecked`, `@HostBinding()`, `ContentChildren`, `ViewChildren`

### Observations
* **Key Points:**
  - There are few things to highlight based on the operations listed above. The first and most interesting one is that change detection for the current view is responsible for starting change detection for child views. This follows from the refreshChildComponents operation.
  - For each child component Angular executes the refreshComponent function.
  - There's a condition that defines if a component will be checked: the primary condition is that component's changeDetectorRef has to be attached to the components tree. If it's not attached, neither the component itself, nor its children or containing transplanted views will be checked.
  - If the primary condition holds, the component will be checked if it's not OnPush or if it's an OnPush and is dirty.
  - There's a logic at the end of the refreshView function that resets the dirty flag on a OnPush component.
  - And lastly, if the component includes transplanted views they will be checked as well.
* **Technical Entities (Classes/Functions/APIs):** `refreshComponent`, `viewAttachedToChangeDetector`, `CheckAlways`, `OnPush`, `TRANSPLANTED_VIEWS_TO_REFRESH`, `refreshContainsDirtyView`

### Template function
* **Key Points:**
  - The job of the executeTemplate function that Angular runs first during change detection is to execute the template function from a component's definition. This template function is generated by the compiler for each component.
  - All the functions from the import are exported with the prefix ɵɵ identifying them as private.
  - This template can include various instructions. In our case it includes creation instructions element and text executed during the initialization phase, and property, advance and textInterpolate executed during change detection phase.
* **Technical Entities (Classes/Functions/APIs):** `executeTemplate`, `ɵɵdefineComponent`, `ɵɵelement`, `ɵɵtext`, `ɵɵproperty`, `ɵɵadvance`, `ɵɵtextInterpolate`

### Lifecycle hook
* **Key Points:**
  - Angular components can use lifecycle hook methods to tap into key events in the lifecycle of a component or directive. A lifecycle that starts when Angular creates the component class and renders the component view along with its child views. The lifecycle continues with change detection, as Angular checks to see when data-bound properties change, and updates both the view and the component instance as needed. The lifecycle ends when Angular destroys the component instance and removes its rendered template from the DOM.
  - It's important to understand that lifecycle hook methods are executed during change detection.
  - Here's the full list of available lifecycle methods: onChanges, onInit, doCheck, afterContentInit, afterContentChecked, afterViewInit, afterViewChecked, ngOnDestroy.
  - Out of those onInit, afterContentInit and afterViewInit are only executed during the initial change detection run (first run). The ngOnDestroy hook is executed only once before the component is destroyed. The remaining 4 methods are executed during each change detection cycle: onChanges, doCheck, afterContentChecked, afterViewChecked.
  - You can also think of a component's constructor as a kind of lifecycle event that is executed when an instance of the component is being created. However, there's a huge difference between a constructor and a lifecycle method from the perspective of the component initialization phase. Angular bootstrap process consists of the two major stages: constructing components tree and running change detection. And the constructor of the component is called when Angular constructs a components tree. All lifecycle hooks including ngOnInit are called as part of the subsequent change detection phase.
  - It's also important to understand that most lifecycle hooks are called on the child component while Angular runs change detection for the current component. The behavior is a bit different only for the ngAfterViewChecked hook.
* **Technical Entities (Classes/Functions/APIs):** `ngOnChanges`, `ngOnInit`, `ngDoCheck`, `ngAfterContentInit`, `ngAfterContentChecked`, `ngAfterViewInit`, `ngAfterViewChecked`, `ngOnDestroy`, `constructor`

## View and Content Queries
* **Key Points:**
  - View and Content queries is something we use to get a hold of the elements in a components template in runtime. Usually the result of the evaluated query is available in the ngAfterViewChecked or ngAfterContentChecked lifecycle method.
  - Angular updates the query results by running the function defined through the component definition.
* **Technical Entities (Classes/Functions/APIs):** `@ViewChild`, `@ContentChild`, `contentQueries`, `viewQuery`, `queryRefresh`, `ngAfterViewChecked`, `ngAfterContentChecked`

## Embedded views
* **Key Points:**
  - Angular provides a mechanism for implementing dynamic behavior in a component's template through the use of view containers. You create a view container using ng-container tag inside the components template and access it using @ViewChild query. These containers provide a way to instantiate a template code, allowing it to be reused and easily modified as needed.
  - View container provides API to create, manipulate and remove dynamic views. I call them dynamic views as opposed to static views created by the framework for static components found in templates. Angular doesn't use a view container for static views and instead holds a reference to child views inside the node specific to the child component.
  - Embedded views are created from templates by instantiating a TemplateRef with the viewContainerRef.createEmbeddedView() method. View containers can also contain host views which are created by instantiating a component with the createComponent() method. A view container instance can contain other view containers, creating a view hierarchy. All structural directives like ngIf and ngFor use a view container to create dynamic views from the directive's template.
  - These embedded views are checked during step #4 in the list: refreshEmbeddedViews(lView).
  - There's a special case of an embedded view called transplanted view. A transplanted view is an embedded view whose template is declared outside of the template of the component hosting the embedded view. The component that declares a template with <ng-template> is not the same component that uses a view container to insert the embedded view created with this template.
* **Technical Entities (Classes/Functions/APIs):** `view container`, `ng-container`, `@ViewChild`, `ViewContainerRef`, `TemplateRef`, `createEmbeddedView()`, `createComponent()`, `ngIf`, `ngFor`, `transplanted view`, `markTransplantedViewsForRefresh`


---

# Change Detection Big Picture – Rendering cycle

## Rendering cycle
* **Key Points:**
  - Updates to the application state are caused by asynchronous events initiated by user actions.
  - For Angular to know when the application state might change, it needs to know when those events occur. This is where zone.js library comes into play. This library decorates (patches) browser platform's API so that all asynchronous events in the browser can be intercepted. Angular binds to the hooks exposed by zone.js and uses notifications about DOM events, timeouts, AJAX/XHR, Promise etc. as a cue to run change detection in Big Picture.
  - The assumption here is that most events cause application state change that needs to be reflected in the DOM and correspondingly on the screen.
  - This is how change detection is able to automatically run after each asynchronous event. Once an event handler finishes executing, Angular will run change detection. Angular doesn't directly interact with zone.js, but instead uses NgZone, which is a kind of a wrapper around zone.js to restrict scope of the events that Angular should be notified about.
  - When an event fires related to the Angular zone (NgZone), an event handler attached to the event runs. Most often this method is the application exposed on the component instance. The business logic can update whatever data it wants to – the shared application model/state and/or the component's view state. After that, when Angular gets the notification from NgZone that there are no outstanding microtasks, it then runs Angular's change detection algorithm.
  - By default (i.e., if you are not using the onPush change detection strategy on any of your components), every component in the tree is examined once from the top, in depth-first order. If you're in dev mode, change detection runs twice because of the extra check Angular performs to ensure stable state. It performs dirty checking on all of your bindings, using those change detector objects. Lifecycle hooks, queries and bindings are processed as part of change detection.
  - In Angular each component is represented as a data structured called LView. This is where the framework keeps track of the state (last value) for all template bindings, such as {{service.a}}. Those values are used during change detection for dirty checking to determine if the side effect related to the change needs to be executed. There's one to one relationship between a component instance and its corresponding LView.
  - Angular encapsulates an LView with the ChangeDetectorRef service that provides change detection functionality. You can get access to this object by injecting it into components constructor. There's one such change detector service per component, so Angular maintains a tree of change detectors that maps to the tree of components. The change detection graph is a directed graph (unidirectional data flow) and cannot have cycles.
  - At this point JavaScript JavaScript yields control to the browser. The event has been processed, business logic updated application state and Angular updated DOM during change detection. Time for the browser to render the updates on the screen and go on to execute macrotasks that might have been scheduled, like network requests or timers.
  - The process of updating the screen includes a well known pipeline of rendering and painting stages broken into substages:
    - [Rendering] Style calculations. This is the process of figuring out which CSS rules apply to which elements based on matching selectors, for example, .headline or .nav > .nav__item. From there, once rules are known, they are applied and the final styles for each element are calculated.
    - [Rendering] Layout. Once the browser knows which rules apply to an element it can begin to calculate how much space it takes up and where it is on screen. The web's layout model means that one element can affect others, for example the width of the <body> element typically affects its children's widths and so on all the way up and down the tree, so the process can be quite involved for the browser.
    - [Painting] Paint. Painting is the process of filling in pixels. It involves drawing out text, colors, images, borders, and shadows, essentially every visual part of the elements. The drawing is typically done onto multiple surfaces, often called layers.
    - [Painting] Compositing. Since the parts of the page were drawn into potentially multiple layers they need to be drawn to the screen in the correct order so that the page renders correctly. This is especially important for elements that overlap another, since a mistake could result in one element appearing over the top of another incorrectly.
  - The browser doesn't necessarily run every substage of the pipeline on every frame. Some changes might only affect a "paint only" property does not affect the layout of the page, like a background image, text color, or shadows, then the browser skips layout, but it will still do paint. The cheapest and most desirable pipeline is the one that skips all parts except compositing, which is usually the case for animations or scrolling.
* **Technical Entities (Classes/Functions/APIs):** `zone.js`, `NgZone`, `LView`, `ChangeDetectorRef`, `ApplicationRef.tick`, `onPush`, `unidirectional data flow`

## An in-depth look
* **Key Points:**
  - Here's detailed overview of what's happening:
    - browser detects the click & adds an event handler to the event queue (browser)
    - zonejs starts with zoneAwareCallback which simply runs callback registered by Angular
    - angular executes the wrapper around the callback from a component's template; the wrapper marks the view and all its ancestors dirty
    - angular runs fetchData component method through the event listener registered in a component template function
    - business logic inside fetchData application code runs network request
    - the request is intercepted by zone.js which schedules macrotask with a browser
    - the event is processed now, zone.js triggers onMicrotaskEmpty through NgZone
    - angular reacts by running application wide change detection through ApplicationRef.tick
    - change detection updates the DOM and runs other side effects
    - browser renders updates on the screen and goes on to execute the macrotask related to the network request. When the network request arrives, the lifecycle repeats, running the event listener for the onreadystatechange event instead of the click event. JavaScript phase ends with Angular running change detection. After that the browser again goes through the rendering pipeline.
* **Technical Entities (Classes/Functions/APIs):** `zoneAwareCallback`, `onMicrotaskEmpty`, `ApplicationRef.tick`

## setTimeout and change detection
* **Key Points:**
  - You may often see the usage of setTimeout inside the application logic. Assume the requirement is to show the input control and immediately focus it when user clicks on a button.
  - The timeout is required because you can't focus() an element that is still hidden. Until Angular change detection has a chance to run (which will be after method showSearchInput() finishes executing), the hidden property in the DOM will not be updated, even though you set searchInputHidden to false in your method.
  - By using setTimeout() with a value of 0 (or no value, which defaults to ~4ms) we schedule a macrotask that will run after Angular gets a chance to run change detection and update the hidden property value.
  - Note that after the setTimeout() callback function finishes executing, change detection will run again (because Angular monkey-patches all setTimeout() calls that are made in the Angular zone). Since the only thing we are changing in our asynchronous callback function is the focus, we can be more efficient and run our callback function outside the Angular zone, to avoid the additional change detection cycle.
* **Technical Entities (Classes/Functions/APIs):** `setTimeout`, `NgZone`, `runOutsideAngular()`

---


# A gentle introduction into change detection in Angular

## Component views and bindings
* **Key Points:**
  - Every component in Angular has a template with HTML elements. When Angular creates the DOM nodes to render the contents of the template on the screen, it needs a place to store the references to those DOM nodes. For that purpose, internally there's a data structure known as View. It's also used to store the reference to the component instance and the previous values of binding expressions. There's a one to one relationship between a component and a view.
  - As the compiler analyzes the template, it identifies properties of the DOM elements that may need to be updated during change detection. For each such property, the compiler creates a binding. The binding defines the property name to update and the expression that Angular uses to obtain a new value.
  - In the actual implementation the binding is not a single object with all required information. A viewDefinition defines actual bindings for template elements and the properties to update. The expression used for a binding is placed in the updateRenderer function.
* **Technical Entities (Classes/Functions/APIs):** `View`, `viewDefinition`, `updateRenderer`, `oldValues`

## Checking a component view
* **Key Points:**
  - As you know, in Angular, change detection is performed for each component. Now that we know that components internally are represented as views, we can say that change detection is performed for each view.
  - When Angular checks a view, it simply runs over all bindings generated for a view by the compiler. It evaluates expressions and compares their result to the values stored in the oldValues array on the view. That's where the name dirty checking comes from. If it detects the difference, it updates the DOM property relevant to the binding. And it also needs to put the new value into the oldValues array on the view. And that's it. You now have an updated UI. Once Angular is done checking the current component, it repeats exactly the same steps for child components.
  - After each change detection cycle, in the development mode, Angular synchronously runs another check to ensure that expressions produce the same values as during the preceding change detection run. This check is not part of the original change detection cycle. It runs after the check is finished for the entire tree of components and performs exactly the same steps. However, this time, as Angular detects the difference, it doesn't update the DOM. Instead, it throws the ExpressionChangedAfterItHasBeenCheckedError.
* **Technical Entities (Classes/Functions/APIs):** `oldValues`, `ExpressionChangedAfterItHasBeenCheckedError`

## The why
* **Key Points:**
  - So now we know when the error is thrown. But why does Angular need this check? Well, imagine that some properties of components have been updated during the change detection run. As a result, expressions produce new values that are inconsistent with what's rendered in the user interface. So, what does Angular do? It certainly could run another change detection cycle to synchronize the application state with the user interface. But what if during that process some properties are updated again? See the pattern? Angular could actually end up in an infinite loop of change detection runs. And actually, that happened quite often in AngularJS.
  - To avoid that situation, Angular imposed the so-called Unidirectional Data Flow. And this check that runs after change detection and the resulting ExpressionChangedAfterItHasBeenCheckedError error is the enforcement mechanism. Once Angular has processed bindings for the current component, you can no longer update the properties of the component that are used in expressions for bindings.

## Fixing the error
* **Key Points:**
  - To prevent the error, we need to ensure that the values returned by expressions during the change detection run and the following check are the same.
  - We learned earlier that the check that produces the error runs synchronously right after the change detection cycle. So if we update it asynchronously, we will avoid the error.
  - This implementation solves our original problem. But, unfortunately, it introduces a new one. All timing events, like setInterval, trigger change detection in Angular. That means that with this implementation we would end up in an infinite loop of change detection cycles. To avoid that, we need a way to run setInterval and not trigger change detection. Luckily for us, there's a way to do that.

## Automatic change detection with zones
* **Key Points:**
  - As opposed to React, change detection in Angular can be triggered completely automatically as a result of any async event in a browser. This is made possible by using the library called zone.js that introduces a concept of zones. Contrary to popular belief, zones are not part of the change detection mechanism in Angular. In fact, Angular can work without them. The library simply provides a way to intercept async events, like setInterval, and notify Angular about them. Based on that notification, Angular runs change detection.
  - What's interesting is that you can have many different zones on a web page. One of them is going to be NgZone. It's created when Angular bootstraps. This is the zone that the Angular application runs in. And Angular only gets notifications about events that occur inside this zone.
  - But, zone.js also provides an API to run some code in a zone other than the Angular zone. Angular is not notified about async events happening in other zones. And no notification means no change detection. The method name to do this is called runOutsideAngular and it's implemented by the NgZone service.
  - Using NgZone to run some code outside of Angular to avoid triggering change detection is a common optimization technique.
* **Technical Entities (Classes/Functions/APIs):** `zone.js`, `NgZone`, `runOutsideAngular()`

## Debugging
* **Key Points:**
  - You might be wondering if there's any way to see this view and the bindings inside Angular. In fact, there is. There's a function named checkAndUpdateView inside the @angular/core module. It runs over each view (component) in a tree of components and performs the check for each view. This is the function I always start debugging when I have problems with change detection.
* **Technical Entities (Classes/Functions/APIs):** `checkAndUpdateView`, `@angular/core`

## Order of operations
* **Key Points:**
  - We've just learned that because of the unidirectional data flow restriction you can't change some properties of a component during change detection after this component has been checked. Most often, this update happens through a shared service or synchronous event broadcasting when Angular runs change detection for child components. But it's also possible to directly inject a parent component into a child component and update the parent state in a lifecycle hook.
  - To understand this behavior, we need to know what operations Angular performs during change detection and their order. And, we already know where we can find them: the checkAndUpdateView function.
  - As you can see, Angular also triggers lifecycle hooks as part of change detection. What's interesting is that some hooks are called before the rendering part when Angular processes bindings and some are called after that.
  - What we can notice here is that Angular calls the AfterViewChecked lifecycle hook for the child component after it's processed the bindings of the parent component. On the other hand, the OnInit hook is called before the bindings are processed. So even if there's a change on the text value in the OnInit, it's still going to be the same during the following check. And that explains the seemingly weird behavior of not having the error with the ngOnInit hook. Mystery solved ?.
* **Technical Entities (Classes/Functions/APIs):** `checkAndUpdateView`, `updateDirectives`, `updateRenderer`, `execComponentViewsAction`, `callLifecycleHooksChildrenFirst`, `AfterViewChecked`, `AfterViewInit`, `OnInit`, `DoCheck`, `OnChanges`

## Summary
* **Key Points:**
  - Okay, let's now summarize what we've just learned. All components in Angular internally are represented in a data structure known as the view. Angular's compiler parses a template and creates the bindings. Each binding defines a property of a DOM element to update and the expression used to obtain the value. The previous values used for comparison during change detection are stored on a view in the oldValues property. During change detection Angular runs over the bindings, evaluates expressions, compares them to the previous values and updates the DOM if necessary. After each change detection cycle, Angular runs a check to ensure the component state is in sync with user interface. This check is performed synchronously and may throw the ExpressionChangedAfterItWasChecked error.