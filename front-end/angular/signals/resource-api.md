---
aliases:
  - Asynchronous Data Flow with Angular’s new Resource API
Source 1: https://www.angulararchitects.io/blog/asynchronous-resources-with-angulars-new-resource-api/
Source 2: https://www.angularspace.com/everything-you-need-to-know-abour-resource-for-now/
---

# Everything about v19 Resource API (for now)

## The problem
* **Key Points:**
  - One of the most common tasks in front end development is to fetch some data from a server and then display it in the UI.
  - Previously, we did something like this: (component with ngOnInit, manual subscription, and property assignment). This more or less covered our concerns, but was verbose and not very flexible. Components that loaded lots of data had lots of disconnected pieces of code, properties declared empty only to be filled in later in subscribe callbacks in ngOnInit or elsewhere.
  - More savvy developers overcame this problems by using the async pipe, which allows us to read Observable values in the template. This solves the verbosity problem, but still does not address the full scope of concerns related to data fetching. What if I want to show a loading spinner? What about errors? How about retrying, also, what if I don't want to make the request immediately, but rather wait for some value change (like a signal update) or some user event (user clicked on a button)?
  - Very different approaches have been adopted by Angular developers, some using RxJS operators like switchMap, retry, and some adopting community-based solutions like the derivedAsync function from the ngxtension package. However, it was high time Angular introduced an own, native tool for solving these concerns.

## The solution
* **Key Points:**
  - In Angular v19, two functions were introduced, resource and rxResource. In essence, they do the same thing, the only difference being that resource works with Promises, while rxResource works with Observables.
  - As Angular's HTTP client is based on RxJS, let's discuss rxResource and keep in mind that the same things would be true if we used fetch instead of HttpClient.get and resource instead of rxResource.
* **Technical Entities (Classes/Functions/APIs):** `resource`, `rxResource`, `HttpClient`, `fetch`

### Loading data
* **Key Points:**
  - With the HttpClient, we fetch the list of countries.
  - The rxResource function accepts a configuration object, which contains a loader function.
  - This loader function is what will be performed to actually make the request. We can put whatever logic we want there, but mainly it would be a call to HttpClient.get.
  - We also import and use the ResourceStatus enum, which contains the possible states of a resource.
  - In the template, we first check if the resource is resolved (countriesResource.status() === status.Resolved).
  - Then, we use the resource to read it's values. The value that we use is actually a signal that will be updated when the resource is resolved.
  - Now, the status signal contains valuable information about the state of the resource, however, it is not always great for determining if the data is available to display. We will see some differences later in this article, but for now, let us update the example to use another signal called hasValue, which will be true if the resource finished loading and the data is available to read.
  - As we can see, everything related to this HTTP request is encapsulated in the countriesResource object.
* **Technical Entities (Classes/Functions/APIs):** `rxResource`, `loader`, `ResourceStatus`, `hasValue`, `status`, `value`

### Handling errors
* **Key Points:**
  - So, how can we learn if the request failed? And how do we use the actual error message/object? Simple, we can use the error signal and the status again.
  - As we can see, we only had to update the template to handle the error case. The ResourceRef object produced by rxResource contains all the information we need to know if the request failed, and what the actual error was via the error signal.
* **Technical Entities (Classes/Functions/APIs):** `error`, `ResourceRef`

### Retrying failed requests
* **Key Points:**
  - What if I want to retry? For example, I might want to show a button that will retry the request.
  - The ResourceRef has a reload function, which, in essence, will trigger the same loader function we have provided, allowing us to retry when we have an error, or just reload then request whenever we want.
* **Technical Entities (Classes/Functions/APIs):** `reload()`, `ResourceRef`

### Loading State
* **Key Points:**
  - Another common concern for HTTP requests is showing some sort of loading indicator, like a spinner or at least some text. Of course, this is also easily achievable with a resource.
  - As we can see, there are two distinct statuses here, Loading and Reloading, which indicate whether it is the first time loading the data, or a load triggered later. This can be useful in some scenarios where we want to differentiate between the two.
  - However, this is a bit cumbersome in generic scenarios, and downright problematic in scenarios where we want to be able to update the resource value manually (later in the article, we will see that this is possible), because in that case the status would be ResourceStatus.Local, indicating that the value has been tampered by some action on the user's end. To avoid the hassle of doing multiple checks, ResourceRef contains a simple signal called isLoading that clearly indicates if there is a necessity to show a loading UI.
* **Technical Entities (Classes/Functions/APIs):** `isLoading`, `ResourceStatus.Loading`, `ResourceStatus.Reloading`, `ResourceStatus.Local`

## Re-evaluating loaded data based on other signals
* **Key Points:**
  - One very common concern is filtering, sorting or paginating the data fetched from a server. Usually, we have some inputs which the user can interact with to set the page, sorting parameters and so on, and based on those changes, we want to reload the data.
  - This can easily be achieved with rxResource, as it accepts another configuration parameter, a function that returns the request parameters to be used.
  - There are two changes here; first, we added a countryName signal and bound it to the input with [(ngModel)]. Next, we added a request function to the resource configuration object. So what this function does is compute parameters for the request, and if we use signals in this function, it will be re-evaluated whenever the signal changes. The signals in that callback are tracked just as they are tracked in computed of effect.
  - Also, if we are using resource instead of rxResource (thus using fetch), the first argument of the request function (which we named parameters in the example) will also contain an abort signal, which we can use to cancel the request if needed!
  - This abort signal is available both in resource, and rxResource, however, in the case of rxResource it is not necessary for cancellation purposes and can be ignored. With rxResource, we can still call destroy on the ResourceRef object returned by it, in which case it will simply unsubscribe from the Observable that we returned, thus cancelling the request.
* **Technical Entities (Classes/Functions/APIs):** `request`, `abortSignal`, `destroy()`, `ResourceRef`

## Some caveats
* **Key Points:**
  - However, if we run this exact code (or the previous example with resource) with this specific API URL, we will first encounter an error. This is because the API we are using expects a name, and initially it is an empty string, so the request will fail. So how do we handle this? This is also important in a context where we don't want to make the request immediately, but rather wait for some value change, like this very example.
  - Well, in our simple case, we can just check if the name parameter has any value, and if not, return an Observable of an empty array.
  - Furthermore, if we want to do some specific comparison, the parameters object also contains a previous property, which contains the previous request parameters. This can be useful in scenarios with many source signals, for example, when we want to only reload the data if the user has changed a specific value, but not the others.
* **Technical Entities (Classes/Functions/APIs):** `previous`

## Using the data locally
* **Key Points:**
  - Another great aspect of rxResource/resource is that they contain writable signals, which means we can modify them or bind them to inputs using [(ngModel)]. This is super useful for a very common scenario of loading some data from the server so that the user can then edit it.
  - As we can see, ResourceRef is a writable signal, and we can use it to update the value of the resource outside of just reloading it from the server.
  - So, like any other signal, we can call update on the ResourceRef object and, in this case, filter out the country we want to delete.
  - However, this is particularly useful in conjunction with template-driven forms. Because it is now possible to bind signals to inputs via [(ngModel)], we can simply load the data that the user will be able to edit, and then bind it directly to some input.
  - As we can see, we simply do a binding to the resource itself - eloquent and simple! This can also be coupled with the new linkedSignal primitive to produce complex forms with multiple inputs.
* **Technical Entities (Classes/Functions/APIs):** `set()`, `update()`, `linkedSignal`, `ResourceRef`

## Important to know
* **Key Points:**
  - First of all, it is important to understand that both functions might yet still evolve, and, as mentioned above, they are still in developer preview.
  - One significant thing missing right now is switch-control; currently, the requests performed by rxResource/resource will switch from a previous request to a new one when source signals change. This means that previous requests will be canceled to make room for new ones. This is the most common behavior, however sometimes we might want to actually make several requests in parallel, or wait and perform all of them sequentially.
  - We could easily do that with plain RxJS operators like mergeMap or concatMap, but with resources we are still a bit limited. It remains an open question if the team will later update this API to include choices for switching behaviors.
  - Finally, and this is very important, as the documentation says, the rxResource and resource functions are only meant to be used for getting data, and not for POST/PUT/DELETE. So, do not use this to edit data, delete items or submit forms.
* **Technical Entities (Classes/Functions/APIs):** `mergeMap`, `concatMap`

## Simplified API reference
* **Key Points:**
  - Here I also want to include some tables showing the simplified API of rxResource and resource (both return ResourceRef, so only one table per concern) so you can easily recap what we talked about.
  - ResourceStatus enum: This enum contains the possible values of the status signal of a ResourceRef.
    - Idle: Status is Idle when either the resource has been destroyed manually, or it has not yet performed its very first request
    - Error: Loading failed with an error.
    - Loading: The resource is currently loading a new value as a result of a change in its request. This status happens when the source signals change, but not when we manually call ResourceRef.reload()
    - Reloading: The resource is currently reloading a fresh value for the same request. This status happens when we manually call ResourceRef.reload() but not when the source signals change
    - Resolved: Loading/Reloading has completed and the resource has the value returned from the loader.
    - Local: The resource's value was set locally via .set() or .update(). Important: this is different from Resolved, so you probably should use it in conjunction if you plan to check for loaded status and also modifying the resource manually
  - ResourceRef simplified API:
    - value: A WritableSignal holding the current value of the resource, or undefined if there is no current value (for instance if the call has not been made yet or it is still loading). Caution: using set or update on this will set the status signal to ResourceStatus.Local
    - status: A Signal indicating the current status of the resource (e.g., 'Loading', 'Error', 'Resolved').
    - error: A Signal holding the last known error from the resource, if in the Error state.
    - isLoading: A Signal indicating whether the resource is currently loading a new value or reloading the existing one. Is true when the status is Loading or Reloading, making it useful for loader checks
    - hasValue(): A reactive function that returns true if the resource has a valid current value.
    - reload(): Instructs the resource to reload its asynchronous dependency. Returns true if a reload was initiated, false otherwise.
    - set(value): Convenience method for setting the value of the resource. Use this instead of ResourceRef.value.set
    - update(updater): Convenience method for updating the value of the resource using an updater function. Use this instead of ResourceRef.value.update
    - asReadonly(): Returns a readonly version of this resource.
    - destroy(): Manually destroys the resource, canceling pending requests and returning it to the idle state. Use this to cancel HTTP calls manually
* **Technical Entities (Classes/Functions/APIs):** `ResourceStatus`, `ResourceRef`, `WritableSignal`, `isLoading`, `hasValue()`, `reload()`, `set()`, `update()`, `asReadonly()`, `destroy()`

## In Conclusion
* **Key Points:**
  - resource/rxResource are a great addition to the Angular ecosystem, and they are definitely expanding on the signals story that has been evolving for more than a year now.
  - resource is also the first step towards making Angular apps not dependant on HttpClient, which is one of the things that make Angular apps still dependant on RxJS. As Angular team moves in the direction of making RxJS optional, it is important to have tools in place that help replace existing tools that depend on RxJS, and resource is one big step towards that.
  - Finally, resource/rxResource are great tools to promote declarative programming in Angular apps, and as we all know too well, HTTP requests, especially ones that depend on dynamic values, have been a big source of imperative programming.

---


# Asynchronous Data Flow with Angular's new Resource API

## First Steps with the Resource API
* **Key Points:**
  - Each resource has a loader function that returns a Promise with the loaded data. This loader is triggered when the resource has been initialized. Optionally, the resource can have a params Signal providing parameters (search criteria) for the loader. Every time the params Signal changes, the loader is triggered again.
  - In this example, dessertsResource uses the values in the originalName and englishName Signals as parameters. The loader is triggered initially and when these Signals change. The param object passed to the loader contains the current search criteria from the params Signal.
  - The resource's result is found in its value Signal. By default, a resource's value Signal is undefined until the loader asynchronously returns the first value. To prevent this, the shown example set the defaultValue to an empty array. The boolean isLoaded informs about the loading state, and error is a Signal with a possible error that occurred during loading.
  - The computed ratedDesserts Signal combines the received desserts with possibly loaded ratings. This shows that we have a clear reactive flow that extends from the user input through loading the resource to projecting the resource to a model bound in the view. One action reactively leads to another.
* **Technical Entities (Classes/Functions/APIs):** `resource()`, `params`, `loader`, `defaultValue`, `value`, `isLoading`, `error`, `computed()`
* **Code Snippet:**
```typescript
@Component([...])
export class DessertsComponent {
  #dessertService = inject(DessertService);

  [...]

  // Criteria for search
  originalName = signal('');
  englishName = signal('');

  // Combine criteria to computed Signal
  dessertsCriteria = computed(() => ({
    originalName: this.originalName(),
    englishName: this.englishName(),
  }));

  // Define resource with params (=search criteria) and loader
  // Every time, the params are changing, the loader is triggered
  dessertsResource = resource({
    params: this.dessertsCriteria,
    loader: (loaderParams) => {
      return this.#dessertService.findPromise(loaderParams.params);
    },
    defaultValue: []
  });

  // initially, resources are undefined
  desserts = this.dessertsResource.value;

  loading = this.dessertsResource.isLoading;
  error = this.dessertsResource.error;

  // The reactive flow goes on ...
  ratings = signal<DessertIdToRatingMap>({});
  ratedDesserts = computed(() => this.toRated(this.desserts(), this.ratings()));

  [...]
}
```

## Important: Loaders are Untracked!
* **Key Points:**
  - It's important to note that while the params are tracked, the loader isn't. That means that a change in the params Signal triggers a new loader execution, but a change in a Signal used in the loader does not.
  - This is because auto-tracking would only work in the first part of the loader that is executed synchronously. Everything that is executed after the first asynchronous operation, e.g., code after await or in a .then handler, cannot be subject to Angular's auto-tracking. To avoid such confusing situations where a part of the loader is handled differently, the whole loader is untracked.

## Race Conditions
* **Key Points:**
  - In many web applications, the user can easily trigger overlapping requests. This is especially the case in an reactive UI where just changing a filter can trigger a further loading operation.
  - In this case, the user would expect to only get results for 'Ice Cream Pancakes' although they shortly had different filters before. It would be quite confusing if results for ordinary 'Pancakes', and 'Sacher Cake' briefly flashed. It would be even more confusing if the first search took a bit longer. In this case, the unwanted intermediate results would flash in an order that does not match the order of requests. You call this a race condition.

### The Resource API has switchMap Semantics
* **Key Points:**
  - In Angular, we usually leverage RxJS and its switchMap Operator that cancels overlapping requests but the last one and hence prevents such situations. The good news is that resource uses the same behavior. However, by default it cannot cancel the former request. Instead, it just ignores its result.
  - If you really want to cancel the former request when another one is scheduled, you need to respect the AbortSignal the resource API passes into the loader.
  - Although it's called AbortSignal, it's not a Signal in the sense of Angular's reactivity primitives but a JavaScript API provided by all modern browsers and supported by several APIs such as the Fetch API. While Angular's HttpClient is meanwhile capable of using fetch under the hoods, its API does not allow passing in an AbortSignal.
  - While switchMap semantics might be the most common one used for loading data, RxJS provides further so-called flattening operators dealing with overlapping requests in different ways. However, for the discussed operations, resource always uses switchMap semantics. Besides this, the reload method discussed below always uses exhaustMap sematics.
  - This shows that the Angular team does not want to compete with existing libraries. The goal is to provide building blocks with reasonable default logic covering the majority of use cases. For everything that goes beyond that, we can implement our own solutions or add proven libraries like RxJS.
* **Technical Entities (Classes/Functions/APIs):** `switchMap`, `AbortSignal`, `Fetch API`, `HttpClient`, `exhaustMap`

### Debouncing
* **Key Points:**
  - Usually, when triggering requests directly after the user changes a filter, we want to have some debouncing: Chances are, the user is changing further filters, and hence, we want to wait some moments not to trigger several unnecessary requests in a row.
  - An easy way to accomplish debouning is adding a delay to the loader. As a consequence, the Resource is already in loading state during debouncing. However, in a good discussion with some of my Angular GDE fellows and Angular team members we came to the conclusion that debouncing the event that leads to a new request might be better suited. There are several arguments for this. One argument is that debouncing the underlying event prevents the Resource from entering loading state before debouncing is complete. Another one is that it allows us to move the Resource in a store that is used regardless of whether the input is debounced or not.
  - Debouncing the underlying event is straightforward when using Reactive Forms. In this case, we can debounce the valueChanges Observable. In cases where we use Template-driven Forms with Two Way Bindings, we need a helper debouncing the bound Signal.
* **Technical Entities (Classes/Functions/APIs):** `debounce`, `Reactive Forms`, `valueChanges`, `Template-driven Forms`

## Reload and Manual Loading
* **Key Points:**
  - The resource used so far automatically triggered the loader after its initialization and after each change of the params Signal. This was convenient in the example at hand. However, it's not a behavior you want all the time.
  - If you want to disable the initial loader execution, just make sure your params Signal returns undefined.
  - As mentioned above, a params Signal returning undefined does not trigger the loader by design. In the current case, we don't have parameters influencing which ratings to load, hence I'm just going with the value true or undefined.
  - To trigger the loader the first time, we can now set the params Signal to true. For triggering the loader afterwards again, the application calls the Resource's reload method.
  - The reload method prevents overlapping requests by immediately returning if there is already a running one. Using terms from RxJS, this can be described as exhaustMap semantics. For this reason, we don't need a case distinction whether it's the first or a subsequent request. This, in turn, streamlines the implementation of loadRatings.
  - I admit that using the type undefined | true might feel a bit off. However, this allows to use the Resource as it is intended to be used: It's all about reactively loading data. This example would feel more natural if we had a real parameter for the resource, like an expertId.
* **Technical Entities (Classes/Functions/APIs):** `reload()`, `exhaustMap`

## linkedSignal for Updates
* **Key Points:**
  - The user can change the ratings. Normally, this is fine because a resource can be manually updated with its set and update methods used in a below section. However, a resource's value is undefined by default. At least since Angular applications use TypeScript's strict mode by default, I try to avoid null and undefined in favor of a default object, also known as Null Object. So far, I used computed to project null and undefined to the respective Null Object.
  - Unfortunately, a computed Signal is read only. The rescue is the linkedSignal function. It provides a computed Signal that can be changed.
  - If Signals used in the computation change, the computation is retriggered and the directly assigned value is overwritten. For the sake of completeness, I want to mention that the computation happens lazily: If no one is interested in the value, the recomputation does not happen.
  - This characteristic makes linkedSignal a perfect fit for forms: We can change a local copy and, on-demand, send it back for saving.
* **Technical Entities (Classes/Functions/APIs):** `linkedSignal()`, `set()`, `update()`

## Error Handling
* **Key Points:**
  - Also Error-Handling is baked into the resource API. When the loader throws or rejects the returned Promise, the resource switches into the state error. In this state, the error Signal provides the thrown value or the value passed to the Promise's reject function.
  - However, the resource will still proceed to work: If someone calls the reload method or if the params Signal is changing, the loader is executed again. If the loader succeeds, the resource clears the error Signal and moves to the resolved state.
  - This is comparable to today's default behavior of Effects in NgRx, where some additional logic under the covers prevents the RxJS pipe used from stopping in the case of an error.
  - However, since Angular 20 you are not allowed to access the resource's value if there is an error. Hence, in the template, you need to check for an error state first.

## rxResource for RxJS-Interop
* **Key Points:**
  - If you already have an Observable, you don't need to convert it into a Promise. Instead, you can use the rxResource API. It works like the resource but the loader is expected to return an Observable. Also, the loader is defined via the property streaming because an rxResource is a so called streaming resource emitting that can emit several values over time.
  - Also, the loader does not need the AbortSignal as Observables can be aborted (by unsubscribing implicitly or explicitly) in general.
* **Technical Entities (Classes/Functions/APIs):** `rxResource`, `streaming`

## Resource in Services and Stores
* **Key Points:**
  - The shown examples directly used the resource in a component. This was primarily for the sake of simplicity. Often, you will use resources in services or stores. One reason for this is state management: The loaded data will survive when Angular destroys the component at hand (e.g., because of switching to another route) so that it can be used later by an instance of the same or another component.
  - The second reason is that stores help to streamline your reactive dataflow. This result in a so-called unidirectional data flow: The components send intentions to the store. I'm using the term 'intension' in an abstract way because it depends on the store, how it is expressed. When using a Redux store, it will be a triggered action; in a more lightweight store like the NGRX Signal Store, it might just be a called method.
  - The expressed intention makes the store perform some tasks that result in new values put into Signals. These Signals can be projected via computed and transport the data down to the component's view. A very simple store is a service providing Signals.
* **Technical Entities (Classes/Functions/APIs):** `Redux`, `NGRX Signal Store`, `computed()`

## Updating Resources
* **Key Points:**
  - Resources are not just read-only. We can also update them with a new value. While writing back changed values needs to be done manually, locally updating the resource ensures that the new value becomes part of our reactive flow. That means it will be displayed and projected, and it will be the foundation for further updates.
  - The save method gets an updated dessert and writes it back to the server. The respective HTTP call is only indicated with a comment here. After performing this call, the dessertResource is updated with the new value using set. As an alternative, there is also an update method that allows to project the current value to a new one. If a loading process takes place while updating the resource with such a local value, this loading process is aborted.
  - In the example above, the resource was private to the service. Hence, the service can make sure the resource is updated only properly. If you want to expose the whole resource, you can use its asReadonly method to prevent consumers from changing the managed data.
* **Technical Entities (Classes/Functions/APIs):** `set()`, `update()`, `asReadonly()`

## Interlude 2: linkedSignal for the Details Form
* **Key Points:**
  - When binding the loaded dessert to a template-driven form, we need writable Signals. However, stores usually provide read-only Signals. Also, the value Signal provided by resource is read-only, although it can be changed directly via the resource's set and update methods.
  - As already mentioned above, this is a good case for linkedSignal.
  - Here, each property we want to bind to a form field is represented by a linked Signal. They are updated whenever their source is changing. Nevertheless, they can be updated with a local value. The save method takes this local value and writes it back to the store which delegates to the resource.
  - As a result, we can directly bind ngModel to our Signals.
  - You can see these linked Signals as the counter part of the FormControl objects in reactive forms. Of course, FormControl objects provide more features like registering validators. But, and this is my personal impression, this shows that with Signals both approaches are not that far from each other.
  - While I decided to use template-driven forms in this example, you can totally leverage reactive forms, too. In this case, you might want to use an effect for connecting the Signals obtained from the store to your FormControls.
* **Technical Entities (Classes/Functions/APIs):** `linkedSignal()`, `ngModel`, `FormControl`, `reactive forms`, `effect()`

## Conclusion
* **Key Points:**
  - The new Resource API is indeed a missing link in Angular's Signal story: It gives us an official way for loading asynchronous resources directly within the reactive flow. Also, it takes care of race conditions and contains basic error handling.
  - With this new API, we have an easy-to-use built-in feature for very common use cases. For more advanced scenarios, such as working with several parallel data streams, we can switch to something more powerful like RxJS. Thanks to the RxJS interop and rxResource, both worlds can be bridged.