---
aliases:
  - Zone.js and its removal
highlights: Zone.js intercepts async ops to trigger CD;Zoneless apps(v18+) remove it entirely - mandatory understanding for modern Angular performance work
Source 1: https://angular.love/from-zone-js-to-zoneless-angular-and-back-how-it-all-works
Source 2: https://javascript-conference.com/blog/angular-20-zoneless-mode-performance-migration-guide/
Source 4: https://angular.dev/guide/zoneless
---
# [Manual Control](change-detection/manual-control.md) 


# From zone.js to zoneless Angular and back — how it all works

## NgZone and Zones

* **Key Points:**
  - Change detection (rendering) in Angular is usually triggered completely automatically as a result of async events in a browser. This is made possible by utilizing zones implemented by zone.js library. In general, zones provide a mechanism to intercept the scheduling and calling of asynchronous operations. Interceptor logic can execute additional code before or after the task and notify interesting parties about the event. These rules are defined individually for each zone when it's being created.
  - Zones are composed in a hierarchical parent-child relationship. At the start the browser runs in a special root zone, which is configured to behave exactly like the platform, making any existing code which is not zone-aware behave as expected. Only one zone can be active at any given time.
  - Contrary to popular belief, zones are not part of the change detection mechanism in Angular. In fact, Angular can work without zones using change detection services. To enable automatic change detection, Angular implements NgZone service that forks a child zone and subscribes to notifications from this zone.
  - This zone is referred to as Angular zone and all application specific code is expected to run inside this zone. That's because NgZone only gets notifications about events that occur inside this Angular zone and doesn't get any notifications about events in other zones.
  - If you explore NgZone, you'll see that the reference to the forked Angular zone is stored in the _inner property.
  - This is the zone that is used to run a callback when you execute NgZone.run().
  - The current zone at the moment of forking the Angular zone is kept in the _outer property and is used to run a callback when you execute NgZone.runOutsideAngular(). Most often this outer zone is the top-most "root" zone.
  - When Angular finished its bootstrapping, that's the hierarchy of zones you get: "root" -> "angular".
  - However, in the development mode, there's also AsyncStackTaggingZone that sits in between root and angular zone like this: "root" -> "AsyncStackTaggingZone" -> "angular". In this case, NgZone instance keeps a reference to the AsyncStackTaggingZone in its _inner property. AsyncStackTaggingZone provides linked stack traces to show where the async operation is scheduled.
  - The last bit that we need to know is that Angular instantiates NgZone during the bootstrapping phase.
  - Angular uses the onMicrotaskEmpty event inside ApplicationRef to automatically trigger change detection for the entire application.
* **Technical Entities (Classes/Functions/APIs):** `zone.js`, `NgZone`, `Zone.current`, `_inner`, `_outer`, `NgZone.run()`, `NgZone.runOutsideAngular()`, `AsyncStackTaggingZone`, `onMicrotaskEmpty`, `ApplicationRef`

## Zoneless application
* **Key Points:**
  - To run an Angular application without zone.js we need to pass noop value into bootstrapModule function for ngZone parameter like this: platformBrowserDynamic().bootstrapModule(AppModule, { ngZone: 'noop' });
  - If we now run this simple application, we'll see that change detection is fully operational and renders time value in the DOM.
  - However, if update the name property inside the callback for setTimeout, We'll see that the change is not updated. This is expected behavior since there's no Angular zone to notify Angular about the timeout event occurrence.
  - What's interesting that we can still inject NgZone into the constructor. But it's an empty implementation of NgZone which does nothing. We could use a change detector service to manually run change detection.
* **Technical Entities (Classes/Functions/APIs):** `ngZone: 'noop'`, `ɵNoopNgZone`, `ChangeDetectorRef`, `detectChanges()`

## Running code inside Angular zone
* **Key Points:**
  - You may find yourself in the situation where you have a function that somehow runs outside of Angular's zone and you don't get the beneif to automatic change detection. It's a common scenario with a third party library doing its stuff unaware of Angular context.
  - The common culprit is using techniques like JSONP that don't use common AJAX APIs like XMLHttpRequest or Fetch API which are patched and tracked by Zones. Instead, it creates a script tag with a source URL and defines a global callback to be triggered when the requested script with data is fetched from the server. This can't be patched or detected by Zones and hence the frameworks remains oblivious to requests performed using this technique.
  - The common solution to such problems is to simply run a callback inside Angular zone like this.
  - Another example is a component that emits notifications from a callback that runs outside the Angular zone. If we simply subscribe to the changes in the child N1 component, we won't see any updates on the screen, even though the this.value is updated to 3 after the parent emits it. To fix this, just like in the gapi example, we could run the callback in Angular zone.
* **Technical Entities (Classes/Functions/APIs):** `NgZone.run()`, `zone.runOutsideAngular()`


---

# No More Zone.js: A Better Way to Build Angular Apps with Angular 20.2

## What is Zone.js and How Does it Work?
* **Key Points:**
  - Zone.js has been at the heart of Angular's change detection since the beginning, but the framework is moving forward. With the introduction of zoneless mode and signals, Angular now supports a reactivity model that is simpler, faster, and more explicit.
  - It's a library that monkey-patches asynchronous browser APIs such as setTimeout, promises, DOM events, and HTTP requests. Each time one of these is completed, it notifies Angular that "something might have changed." But Zone.js couldn't provide details about what changed or where. As a result, Angular had to trigger change detection across the whole component tree to make sure the UI stayed in sync.
  - That trade-off defined much of the Angular developer experience. On the bright side, Zone.js made things feel almost magical, the UI updated automatically whenever async code finished, and you didn't have to think about it.
  - But the magic came with a price. Zone.js treated every async event as a possible change, which meant Angular often did more work than necessary. Over time, that extra overhead slowed apps down and made debugging harder.
* **Technical Entities (Classes/Functions/APIs):** `Zone.js`, `setTimeout`, `promises`, `DOM events`, `HTTP requests`

## Angular Zoneless – From Experimental to Stable
* **Key Points:**
  - The idea of running Angular without Zone.js has been a long-awaited change in the framework's evolution. Back in Angular 18, the team introduced the first experimental APIs for zoneless mode.
  - With the release of Angular 20.2, these APIs became stable, and we can now confidently build production applications in zoneless mode. Instead of relying on Zone.js, we work with an explicit change detection model where updates are triggered by signals, template events, async pipes, and manual checks when necessary.

## Why go zoneless?
* **Key Points:**
  - So why should we drop Zone.js now that we finally can? The main reasons come down to leaner applications, improved performance, and more predictable behavior.
  - Benefits of going zoneless:
    - Reduced bundle size and faster initial load: Without Zone.js, the bundle shrinks by about 33 KB. That's not huge on its own, but it translates directly into a faster initial load, since the browser no longer has to download and parse the library.
    - Better performance: Zone.js often triggered unnecessary change detection cycles, even when no data had changed. Zoneless mode removes that overhead. Change detection now runs only when it actually needs to, giving us more predictable and performant rendering.
    - Easier debugging: With Zone.js gone, stack traces are no longer wrapped in Zone-specific frames. You get a full, accurate stack trace that points exactly to where something happened. No more extra noise. This makes debugging and profiling significantly easier.
    - Full control over reactivity: In zoneless Angular, the developer explicitly decides when the UI should update. This is a major shift – instead of relying on Zone.js "magic," you know exactly what triggers change detection and when it happens. That makes the app's reactivity model both transparent and intentional.
  - Trade-offs to keep in mind:
    - You may need to adjust your mental model. Without automatic change detection, you must adopt a more deliberate strategy for updating the UI. That means paying more attention to using signals, async pipes, or calling markForCheck when necessary.
    - Migration effort: migrating a large app can take time, especially if it's heavily tied to Zone.js behaviors and doesn't use signals and onPush change detection strategy.

## Create a zoneless project
* **Key Points:**
  - Starting a zoneless project is surprisingly simple. In Angular v20.2, you can enable zoneless mode directly when you are creating a new project using CLI.
  - If you skip the flag, the Angular CLI will ask you a question during project setup. Select "Yes" and you will get a project fully Zone.js free!

## Migration to zoneless
* **Key Points:**
  - If you already have an Angular project and want to migrate to zoneless, the process takes a few steps:
    - In app.config.ts, swap: provideZoneChangeDetection({ eventCoalescing: true }) → provideZonelessChangeDetection()
    - Remove zone.js from angular.json build and test configs.
    - Delete imports: import zone.js and import zone.js/testing.
    - Uninstall Zone.js. Once nothing depends on it anymore, uninstall it.
    - Verify in the browser. Open your app in the browser, open the console, and type Zone. You should get an error: Zone is not defined. That confirms Zone.js has been fully removed.
* **Technical Entities (Classes/Functions/APIs):** `provideZonelessChangeDetection()`, `provideZoneChangeDetection()`, `angular.json`, `app.config.ts`

## How Change Detection Works Without Zone.js
* **Key Points:**
  - Once Zone.js is gone, Angular no longer "guesses" when to refresh the UI. Instead, the framework listens to specific, intentional triggers that tell it exactly when change detection should run.
  - Change detection triggers:
    - Bound host or template event listeners
    - Async pipe calls ChangeDetectorRef.markForCheck() under the hood whenever the observed value changes, ensuring your template reflects the new data.
    - Updating a signal used in a template
    - ComponentRef.setInput(): When you programmatically set an input on a dynamically created component, Angular marks that view as dirty and schedules change detection.
    - Manual call of ChangeDetectorRef.markForCheck(): While Angular handles change detection automatically in most cases, you can still force it with markForCheck(), ensuring Angular picks up changes it wouldn't catch otherwise.
  - It's important to understand that going zoneless doesn't rewrite Angular's change detection model from scratch. The two familiar strategies, Default and OnPush, are still in place, and their behavior hasn't changed. What changed is when the change detection process starts.
  - With OnPush, Angular checks only those components that have been explicitly marked as dirty.
  - When an asynchronous task triggers a change in a signal, this update does not mark all ancestors as dirty. Instead, only the component consuming that signal (the "consumer") is tagged as dirty.
  - Ancestors aren't marked as dirty, but instead receive a special marker called HasChildViewsToRefresh. This marker tells Angular that the component itself is clean, but it has children that need to be refreshed.
  - This optimization only works if the signal update isn't triggered by mechanisms that already mark components as dirty (for example, event listener). In that case, the ancestors will be marked both as dirty and with the HasChildViewsToRefresh flag, which means Angular will check them as well.
* **Technical Entities (Classes/Functions/APIs):** `Default`, `OnPush`, `HasChildViewsToRefresh`, `ComponentRef.setInput()`, `ChangeDetectorRef.markForCheck()`

## Preparing for Zoneless
* **Key Points:**
  - If we want to go zoneless, we first need to prepare our apps. It's not just about removing Zone.js, we also need to make sure our components know how to notify Angular about changes. In other words, we have to replace the "magic" that Zone.js gave us with explicit signals, async pipes, or markForCheck calls. Once that's in place, the transition becomes smooth and more predictable.
  - The very first step is to switch all components to the OnPush change detection strategy. Why? Because it immediately reveals what will stop working once Zone.js is gone. By forcing Angular to update only when explicitly notified, we can clearly see which parts of the app rely on Zone.js magic, and fix them before the actual migration.
  - Once we switch the component to the OnPush change detection strategy, things suddenly break. Why does this happen? Previously, Zone.js automatically tracked async tasks, such as HTTP requests. When the request finished, it triggered change detection for us. Without Zone.js, nothing notifies Angular that the data has arrived, so the UI never updates.
  - At this point, we have to trigger change detection ourselves. One option is to inject ChangeDetectorRef and call markForCheck after updating crewMembers. You can use it if you have to, but there are usually better options.
  - A much better approach is to use the async pipe. It eliminates the need for manual subscription logic in your component and guarantees that Angular updates the view whenever data changes.
  - We can also take advantage of toSignal. With it, we transform an observable into a signal inside our component. Whenever the observable emits a new value, the signal's value is updated, and Angular reacts right away. Subscriptions are managed under the hood, so we avoid the extra boilerplate of manual unsubscribe logic.
  - Beyond async pipes and signals, there's also a new player: httpResource. It's still experimental, but it already works seamlessly in a zoneless environment. Why? Because it doesn't rely on Zone.js at all, it exposes its state through signals, making it a natural fit for the new change detection model.
  - Another common pitfall comes from using setTimeout or setInterval. In a zoneless app, they no longer trigger change detection automatically. If your code relies on them, you'll need to adjust it before migrating. Depending on the case, you can either call markForCheck to notify Angular manually or update a signal value directly. Just remember, for the signal update to refresh the UI, it has to be read in the template.
  - Angular also gives us a safety net to verify that our app is truly zoneless-ready. With provideCheckNoChangesConfig({ exhaustive: true, interval: < milliseconds > }) to app.config.ts, we can enable a periodic debug check that ensures no state changes slip by unnoticed. If Angular detects a binding update that wouldn't have been refreshed by zoneless change detection, it throws an ExpressionChangedAfterItHasBeenCheckedError. This helps us catch hidden dependencies on Zone.js before they become real issues in production.
* **Technical Entities (Classes/Functions/APIs):** `OnPush`, `ChangeDetectorRef.markForCheck()`, `async pipe`, `toSignal`, `httpResource`, `setTimeout`, `setInterval`, `provideCheckNoChangesConfig()`, `ExpressionChangedAfterItHasBeenCheckedError`
* **Code Snippet:**
```typescript
// app.config.ts - Safety net
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideZonelessChangeDetection(),
    provideCheckNoChangesConfig({exhaustive: true, interval: 1000}),
    provideRouter(routes),
    provideHttpClient(),
  ]
};
```

## Conclusion: The End of an Era, the Start of Another
* **Key Points:**
  - Zone.js has been part of Angular from the very beginning, bringing the "magic refresh" that automatically kept UIs in sync. For years, it simplified development and allowed developers to focus on building features instead of managing updates manually. But as applications grew larger and the web evolved, the hidden costs of that magic became harder to ignore: performance overhead, noisy debugging, compatibility issues, and extra complexity in testing.
  - That's why the shift to zoneless marks an important milestone in Angular's evolution. Developers can finally build apps without Zone.js, relying instead on signals, markForCheck, and OnPush-friendly patterns.
  - Zone.js was magic. Zoneless is mastery. With Angular 20.2, you can finally leave the overhead behind, build apps that are faster and easier to debug, and take full control of change detection. The future of Angular is zoneless. It's time to join it.