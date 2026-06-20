---
aliases:
  - NgZone.runOutsideAngular()
highlights: Execute code without triggering CD;essential for thrid-party animation libraries, WebSocket streams and requestAnimationFrame loops
Source 1: https://medium.com/@krzysztof.grzybek89/how-runoutsideangular-might-reduce-change-detection-calls-in-your-app-6b4dab6e374d
Source 2: https://angular.dev/best-practices/zone-pollution
---
# How `runOutsideAngular` might reduce change detection calls in your app

## Frequent timers
* **Key Points:**
  - It might occurs in every place where You fire timers frequently, like setInterval, setTimeout or requestAnimationFrame. Let's say we have to do some canvas animation inside our angular app. Our animation loop is simple setInterval.
  - Every 10ms our canvas is updated with new color. Looks fine, right? Now open console in devTools. Each time change detection is triggered, You will see log 'Change detection triggered'. Oh my, looks like something went wrong! How did it happened? By default, change detection is triggered every time when last function in call stack is executed — if it's in the NgZone. (it's actually very simplified, but basically that's how it works). So every 10ms we put callback from setInterval on the call stack, and after execution of this function, change detection is triggered on the root component. Angular doesn't care that we don't need it at all! But, as You probably assume, there's a simple way to fix it. You can just wrap it with NgZone.runOutsideAngular.
  - Why is that? Because now we run timers outside NgZone and angular isn't even "aware" that we fire these timers.
* **Technical Entities (Classes/Functions/APIs):** `setInterval`, `setTimeout`, `requestAnimationFrame`, `NgZone.runOutsideAngular()`, `NgZone`

## Outside clicks
* **Key Points:**
  - Second case: You want to react on click outside Your component. Typical situation for stuff like dropdown, select input or popup. So what You probably would do is to bind @HostListener to document and check if component host contains event target.
  - Host listeners are great, but... This cause additional change detection on each document click. If we have 10 dropdowns on page, it would make additional 10 change detection calls on each click! Let's use our tools to improve that. Because event listener was bound in NgZone, after each callback, views has to be checked for changes, so we want to bind listener outside NgZone. We want to be aware of state changes when click triggers dropdown close, so we have to get back to the Zone when needed.
* **Technical Entities (Classes/Functions/APIs):** `@HostListener`, `document:click`, `NgZone.runOutsideAngular()`, `NgZone.run()`

## Non-angular components integration
* **Key Points:**
  - Let's move to third case, the last one I'm gonna show you. It's about creating angular components based on vanilla js or jquery plugins packages. Let's say we need color picker component. We can write it from scratch, but why reinvent the wheel if there are non angular, stable solutions? We can just wrap it in angular component and we're ready to go.
  - It works, but there's one major drawback. Let's see how many change detections are fired. Change detection is fired after every time mousemove callback is finished. It's because event listener was added in NgZone. To fix it, let's just initialize plugin outside NgZone, and get back only if there's a need to do it (e.g. we update angular component property value).
* **Technical Entities (Classes/Functions/APIs):** `NgZone.runOutsideAngular()`, `NgZone.run()`, `mousemove`

## Summary
* **Key Points:**
  - We examined 3 cases where runOutsideAngular can reduce significantly amount of changeDetection calls. In simple apps, You probably won't spot the performance improvements, but in robust apps, or some specific cases it might be a nice boost. Notice that it required from us just a little tweak in our code and understating of zones. I hope that this will inspire You to increase your knowledge about zone.js and NgZone.

---
# Resolving zone pollution

## Identifying unnecessary change detection calls
* **Key Points:**
  - Zone.js is a signaling mechanism that Angular uses to detect when an application state might have changed. It captures asynchronous operations like setTimeout, network requests, and event listeners. Angular schedules change detection based on signals from Zone.js.
  - In some cases scheduled tasks or microtasks don't make any changes in the data model, which makes running change detection unnecessary. Common examples are: requestAnimationFrame, setTimeout or setInterval, Task or microtask scheduling by third-party libraries.
  - You can detect unnecessary change detection calls using Angular DevTools. Often they appear as consecutive bars in the profiler's timeline with source setTimeout, setInterval, requestAnimationFrame, or an event handler. When you have limited calls within your application of these APIs, the change detection invocation is usually caused by a third-party library.
* **Technical Entities (Classes/Functions/APIs):** `Zone.js`, `setTimeout`, `network requests`, `event listeners`, `requestAnimationFrame`, `setInterval`, `Angular DevTools`, `NgZone`

## Run tasks outside NgZone
* **Key Points:**
  - In such cases, you can instruct Angular to avoid calling change detection for tasks scheduled by a given piece of code using NgZone. The preceding snippet instructs Angular to call setInterval outside the Angular Zone and skip running change detection after pollForUpdates runs.
  - Third-party libraries commonly trigger unnecessary change detection cycles when their APIs are invoked within the Angular zone. This phenomenon particularly affects libraries that set up event listeners or initiate other tasks (such as timers, XHR requests, etc.). Avoid these extra cycles by calling library APIs outside the Angular zone.
  - Running Plotly.newPlot('chart', data); within runOutsideAngular instructs the framework that it shouldn't run change detection after the execution of tasks scheduled by the initialization logic. For example, if Plotly.newPlot('chart', data) adds event listeners to a DOM element, Angular does not run change detection after the execution of their handlers.
  - But sometimes, you may need to listen to events dispatched by third-party APIs. In such cases, it's important to remember that those event listeners will also execute outside of the Angular zone if the initialization logic was done there.
  - If you need to dispatch events to parent components and execute specific view update logic, you should consider re-entering the Angular zone to instruct the framework to run change detection or run change detection manually.
  - The scenario of dispatching events outside of the Angular zone may also arise. It's important to remember that triggering change detection (for example, manually) may result in the creation/update of views outside of the Angular zone.
* **Technical Entities (Classes/Functions/APIs):** `NgZone`, `runOutsideAngular()`, `NgZone.isInAngularZone()`, `ngZone.run()`
* **Code Snippet:**
```typescript
import { Component, NgZone, OnInit, inject } from '@angular/core';

@Component(...)
class AppComponent implements OnInit {
  private ngZone = inject(NgZone);

  ngOnInit() {
    this.ngZone.runOutsideAngular(() => setInterval(pollForUpdates, 500));
  }
}
```