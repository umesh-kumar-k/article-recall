---
aliases:
  - Async Pipe as Architecture Pattern
highlights: Subscribes/unsubscribes, marks for check automatically - the only correct way to bind observables in templates at scale
Source 1: https://medium.com/@kellymarjorie/using-the-angular-async-pipe-b5288135b52a
Source 2: https://www.thinktecture.com/en/angular/understanding-the-async-pipe/
Source 3: https://modernangular.com/articles/handling-errors-reactively-with-async-pipe
Source 4: https://dev.to/petersaktor/avoiding-performance-mistakes-with-angulars-async-pipe-1166
---
# Understanding Angular's Async Pipe: Condensed Angular Experiences – Part 1

## Where Do We Profit From Using the Async Pipe?
* **Key Points:**
  - Every Angular developer knows the async pipe. It is almost always present when writing components and using some observables or promises.
  - Angular's async pipe is a tool to resolve the value of a subscribable in the template. A subscribable can be an Observable, an EventEmitter, or a Promise. The pipe listens for promises to resolve and observables and event emitters to emit values.
  - As simple as the async pipe function sounds, it opens the door to more readable code, fewer unintended memory leaks, and better performance. It makes the code more readable by moving the variable declaration and usage of the template.
  - Further, it handles the subscription for us and, more importantly, unsubscribes from the subscribable if the host component is destroyed.
  - But the most critical advantage plays into our hands in conjunction with the OnPush change detection strategy. Immutable values bound to a component input automatically refresh the corresponding component's view, but that does not apply to member variables that are used in the template. The async pipe marks the host view as dirty and ready to check if the value was updated.
* **Technical Entities (Classes/Functions/APIs):** `async pipe`, `Observable`, `EventEmitter`, `Promise`, `OnPush`

## How Does the Async Pipe Work?
* **Key Points:**
  - The pipe takes a subscribable, subscribes internally to it, stores, and re-emits its value. It also handles the teardown if you swap the underlying object.
  - Note that Angular distinguish between a promise and other types. Both types are explicitly handled to their specifications. Both receive the callback that updates the internal value.
  - The promise variant resolves the Promise via the then function. The Subscribable variant calls the subscribe function instead and returns the subscription. That way, it is possible to unsubscribe later on destroy or calling it directly while swapping the underlying subscription. While the SubscribableStrategy must unsubscribe, the PromiseStrategy does nothing on dispose and on destroy because it doesn't have to. Anyway, the method is called in both cases to be safe and fulfill the API.
  - The async pipe also unsubscribes inside the on-destroy lifecycle hook to prevent memory leaks. As explained in the first block, the pipe handles the teardown itself by calling the dispose function. Which will call dispose on the specifically selected strategy.
* **Technical Entities (Classes/Functions/APIs):** `transform()`, `_subscribe()`, `_selectStrategy()`, `_promiseStrategy`, `_subscribableStrategy`, `ngOnDestroy()`, `_dispose()`

### Hooking Into the Change Detection
* **Key Points:**
  - But the best part is yet to come. The pipe also helps us with the change detection. It can become cumbersome and complex to manage view updates using the OnPush change detection strategy. But the pipe automatically calls the markForCheck() method on updates for us. Afterward, Angular runs the change detection on the next cycle until reaching the host component itself.
* **Technical Entities (Classes/Functions/APIs):** `markForCheck()`, `OnPush`, `ChangeDetectorRef`

## The Async Pipe in Action
* **Key Points:**
  - Using the pipe is as simple as this: `<some-component [inputValue]="someObservable$ | async"></some-component>`. It takes the observable and maps its value to the input.
  - Alternatively, we could resolve it in an *ngIf and access the object's properties.
  - All that works fine until we must handle mutable objects. Angular's change detection works by comparing the identity of two objects obj1 === obj2 and Object.is(), disregarding internal changes. However, this does not work with a mutable object, but how does it come then that it seems to work in the demo application (as seen on the lower right card)? As described, the pipe marks the host component to be checked in the next change detection cycle. That check will go through the component's view and looks for changes. That way, even, at first, undetected changes will be updated.
  - If you can't use the async pipe for some reason, you could, for instance, resolve observables and promises in your component.

## An Alternative to the Async Pipe
* **Key Points:**
  - As alternatives to the async pipe, you can resolve the observable or promise in your component. But keep in mind, you have to make sure not to cause any memory leaks by unsubscribing from the observable.
  - When using OnPush you also have to make sure that the view is updated correctly. For this, we can call the markForCheck() method on the ChangeDetectorRef and notify it about changes.
* **Technical Entities (Classes/Functions/APIs):** `markForCheck()`, `ChangeDetectorRef`, `ngOnDestroy()`, `unsubscribe()`

## Conclusion
* **Key Points:**
  - This article explores the async pipe, how it works, what it is used for, and examines its source code in detail. We have seen how to use it and provided examples, demos, and alternatives. Now it is up to you to decide where to use it in your projects and when.
  - As seen in the examples above, we don't have to use Angular's build-in async pipe. We can resolve promises and subscribable ourselves. But we must handle the subscription clean-up and make sure the view is updated. The latter part is vital in conjunction with the OnPush change detection strategy.
  - By using the async pipe, we let Angular take on the heavy lifting for us. It simplifies the components and reduces written code by moving the complexity into the framework. It also unifies the way we handle Promises, Observables, EventEmitter, and Subjects; generally Subscribables – creating a pleasant dev experience at the cost of increasing the required framework knowledge. Isn't it what we want a Framework for to hide complexity and for a good consistent API? But we don't have to add extra imports to our components and write repetitive code. We don't have to worry about changes when using the OnPush change detection strategy if we use immutable objects or cut the components accordingly.

---

# Handling Errors Reactively with the Async Pipe in Angular

## Step 1: Understand the Current Scenario
* **Key Points:**
  - We've already seen the success case for using async pipes. Now let's take a look at what happens when the stream errors.
  - When we do this, we will just get an eternal loading display because the observable stream has errored. Now let's look at how we can deal with this.
* **Technical Entities (Classes/Functions/APIs):** `async pipe`, `Observable`, `of()`, `delay()`, `tap()`, `concatMap()`

## Step 2: Create a Separate Error Stream
* **Key Points:**
  - The general idea is to create a separate stream to deal with errors and then utilize that in our template as well. When a stream errors, it is destroyed. We want to create an error stream that emits a value when it errors.
  - To do this, we can use the catchError operator, which catches when the stream errors and allows us to provide a replacement observable stream to subscribe to instead. The original stream is still destroyed, but now we are returning a new observable stream using of and providing the error as a value for that stream to emit. We can then utilize this value in our template.
  - The problem we have, though, is that this isn't just going to emit errors; it will also emit all the normal values of the stream. That's why we also have the ignoreElements operator. This will ignore all values except errors and completion.
* **Technical Entities (Classes/Functions/APIs):** `catchError`, `of()`, `ignoreElements()`

## Step 3: Update the Loading Template
* **Key Points:**
  - Now we need to update our loading template to handle errors. We have moved the entire view model up into an ng-container, which handles subscribing to everything with the async pipe. We need this ngIf to set up these values with the async pipe, but the If functionality isn't relevant here. This will always evaluate to true; we just need it to set up this object to use.
  - We have added an additional check within the loading template so that if the userError stream has emitted, we won't continue to show the loading UI.
* **Technical Entities (Classes/Functions/APIs):** `ng-container`, `*ngIf`, `async pipe`

## Step 4: Handle Temporal Values
* **Key Points:**
  - Now let's see how our solution deals with temporal values, that is, values emitted over time.
  - You will see that it works, but it's a bit awkward because the previous successful users continue to show, while we also have our error message indicating that the current user failed to be pulled in. If we want, we can add another ngIf to the user paragraph to remove it from the DOM when there is an error, so we aren't displaying old values.
  - Remember that when a stream errors, it is destroyed, so we won't continue to get the remaining values from this stream after the error.
* **Technical Entities (Classes/Functions/APIs):** `ngIf`, `async pipe`

## Conclusion
* **Key Points:**
  - We now have a nice way to deal with errors with streams and the async pipe in a reactive way. Keep in mind that this approach is highly context-dependent. In a real-world scenario, you would likely abstract some of the template logic out into separate presentational components.

---

# Avoiding Performance Mistakes with Angular's Async Pipe

## The Problem: Unnecessary Subscriptions Inside Loops
* **Key Points:**
  - The async pipe is a powerful feature in Angular that helps manage subscriptions automatically. However, improper usage can lead to performance bottlenecks.
  - Consider the following problematic code: `@for (item of items; track item.id) { <user-item [user]="user$ | async" [item]="item"></user-item> }`
  - At first glance, this might seem fine. However, let's examine how the async pipe works internally.
* **Technical Entities (Classes/Functions/APIs):** `async pipe`, `@for`

## How the Async Pipe Works
* **Key Points:**
  - When you pass an observable to the async pipe, Angular creates a subscription: `this._subscribe(obj);`
  - Here's how the transform method is implemented.
  - The subscription is assigned to _subscription.
  - Each time the async pipe is used, it creates a new subscription and updates the view.
* **Technical Entities (Classes/Functions/APIs):** `transform()`, `_subscribe()`, `_subscription`, `_updateLatestValue()`, `markForCheck()`

## Why is This a Problem?
* **Key Points:**
  - Using the async pipe inside a loop means that a new subscription is created for each iteration, leading to multiple unnecessary change detection triggers. This can severely impact performance.

## The Solution: Using Async Pipe Outside the Loop
* **Key Points:**
  - Instead of using the async pipe inside the loop, extract its value beforehand using *ngIf, the @if directive, or the let syntax in Angular's newer syntax.
  - Correct Approach 1: Using Let Syntax: `@let user = users$ | async;`
  - Correct Approach 2: Using *ngIf
* **Technical Entities (Classes/Functions/APIs):** `@let`, `*ngIf`, `*ngFor`, `trackBy`, `trackById`
* **Code Snippet:**
```html
@let user = users$ | async;
@for (item of items; track item.id) {
  <user-item [user]="user" [item]="item"></user-item>
}
```

## Conclusion
* **Key Points:**
  - Using the async pipe correctly can significantly improve performance in Angular applications. Avoid placing it inside loops and instead extract the observable value beforehand to prevent unnecessary subscriptions and change detection cycles. By following these best practices, you can ensure your application remains efficient and responsive.
  - For more information, check out the Angular async pipe implementation on GitHub.
  - Happy coding!