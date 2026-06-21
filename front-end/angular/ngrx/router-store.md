---
aliases:
  - NgRx Router Store
Source 1: https://danywalls.com/handling-router-url-parameters-using-ngrx-router-store
---
# Angular Router URL Parameters Using NgRx Router Store

## Scenario
* **Key Points:**
  - I want to create an edit page where users can modify the details of a selected place, share the URL, and return to the same state later. For example, http://localhost/places/2, where 2 is the ID of the place being edited. Users should also be able to return to the home page after performing an action.
  - 💡This article is part of my series on learning NgRx.

## The Edit Page
* **Key Points:**
  - First open terminal and using the Angular CLI, generate a new component: `ng g c pages/place-edit`
  - Next, open app.routes.ts and register the PlaceEditComponent with the parameter /places/:id.
* **Technical Entities (Classes/Functions/APIs):** `PlaceEditComponent`, `app.routes.ts`

## Get The Place To Edit
* **Key Points:**
  - My first solution is a combination of the service, effect, router and activated route. It will require make add logic in several places.
  - This solution seems promising. I need to dispatch the editPlace action and inject the router in place-card.component.ts to navigate to the /places:id route.
  - It works! But there are some side effects. If you select another place and go back to the page, the selection might not be updated, and you may load the previous one. Also, with slow connections, you might get a "not found" error because it is still loading.
  - 💡One solution, thanks to Jörgen de Groot, is to move the router to the effect. Open the places.effect.ts file and inject the service and router. Listen for the editPlace action, get the data, then navigate and dispatch the action.
  - Now we fixed the issue of navigating only when the user click in the list of places, but when reloading the page that it's not working, because our state is not ready in the new route, but we have an option use the effect lifecycle hooks.
  - The effects lifecycle hooks allow us to trigger actions when the effects are register, so I wan trigger the action loadPlaces and have the state ready.
  - Read more about Effect lifecycle and ROOT_EFFECTS_INIT
  - Okay, I have the state ready, but I'm still having an issue when getting the ID from the URL state.
  - A quick fix is to read the activatedRoute in ngOnInit. If the id is present, dispatch the action editPlace. This will redirect and set the selectedPlace state.
  - It works! Finally, we add a cancel button to redirect to the places route and bind the click event to call a new method, cancel.
  - Okay, it works with all features, but our component is handling many tasks, like dispatching actions and redirecting navigation. What will happen when we need more features? We can simplify everything by using NgRx Router, which will reduce the amount of code and responsibility in our components.
* **Technical Entities (Classes/Functions/APIs):** `PlacesPageActions.editPlace`, `PlacesApiActions.getPlaceSuccess`, `PlacesApiActions.getPlaceFailure`, `createEffect`, `ROOT_EFFECTS_INIT`, `ActivatedRoute`, `Router`, `ngOnInit`
* **Code Snippet:**
```typescript
export const getPlaceEffect$ = createEffect(
  (
    actions$ = inject(Actions),
    placesService = inject(PlacesService),
    router = inject(Router)
  ) => {
    return actions$.pipe(
      ofType(PlacesPageActions.editPlace),
      mergeMap(({ id }) =>
        placesService.getById(id).pipe(
          tap(() => console.log('get by id')),
          map((apiPlace) => {
            router.navigate(['/places', apiPlace.id]);
            return PlacesApiActions.getPlaceSuccess({ place: apiPlace });
          }),
          catchError((error) =>
            of(PlacesApiActions.getPlaceFailure({ message: error }))
          )
        )
      )
    );
  },
  { functional: true }
);
```

## Why NgRx Router Store ?
* **Key Points:**
  - The NgRx Router Store makes it easy to connect our state with router events and read data from the router using build'in selectors. Listening to router actions simplifies interaction with the data and effects, keeping our components free from extra dependencies like the router or activated route.
* **Technical Entities (Classes/Functions/APIs):** `NgRx Router Store`, `build'in selectors`

### Router Actions
* **Key Points:**
  - NgRx Router provide five router actions, these actions are trigger in order:
    - ROUTER_REQUEST: when start a navigation.
    - ROUTER_NAVIGATION: before guards and revolver , it works during navigation.
    - ROUTER?NAVIGATED: When completed navigation.
    - ROUTER_CANCEL: when navigation is cancelled.
    - ROUTER_ERROR: when there is an error.
  - Read more about ROUTER_ACTIONS
* **Technical Entities (Classes/Functions/APIs):** `ROUTER_REQUEST`, `ROUTER_NAVIGATION`, `ROUTER_NAVIGATED`, `ROUTER_CANCEL`, `ROUTER_ERROR`

### Router Selectors
* **Key Points:**
  - It helps read information from the router, such as query params, data, title, and more, using a list of built-in selectors provided by the function getRouterSelectors.
  - Read more about Router Selectors
  - Because, we have an overview of NgRx Router, so let's start implementing it in the project.
* **Technical Entities (Classes/Functions/APIs):** `getRouterSelectors`, `selectQueryParam`, `selectRouteParam`

## Configure NgRx Router
* **Key Points:**
  - First, we need to install NgRx Router. It provides selectors to read from the router and combine with other selectors to reduce boilerplate in our components.
  - In the terminal, install ngrx/router-store using the schematics: `ng add @ngrx/router-store`
  - Next, open app.config and register routerReducer and provideRouterStore.
  - We have the NgRx Router in our project, so now it's time to work with it!
  - Read more about install NgRx Router
* **Technical Entities (Classes/Functions/APIs):** `@ngrx/router-store`, `routerReducer`, `provideRouterStore`, `app.config`

## Simplify using NgRx RouterSelectors
* **Key Points:**
  - Instead of making an HTTP request, I will use my state because the ngrx init effect always updates my state when the effect is registered. This means I have the latest data. I can combine the selectPlaces selector with selectRouterParams to get the selectPlaceById.
  - Perfect, now it's time to update and reduce all dependencies in the PlaceEditComponent, and use the new selector PlacesSelectors.selectPlaceById.
  - Okay, but what about the cancel action and redirect? We can dispatch a new action, cancel, to handle this in the effect.
  - Update the cancel method in the PlacesComponent and dispatch the cancelPlace action.
  - The final step is to open place.effect.ts, add the returnHomeEffects effect, inject the router, and listen for the cancelPlace action. Use router.navigate to redirect when the action is dispatched.
  - Finally, the last step is to update the place-card to dispatch the selectPlace action and use a routerLink.
* **Technical Entities (Classes/Functions/APIs):** `getRouterSelectors`, `selectRouteParams`, `selectPlaces`, `createSelector`, `selectPlaceById`, `PlacesPageActions.cancelPlace`, `returnHomeEffect$`, `router.navigate`
* **Code Snippet:**
```typescript
export const { selectRouteParams } = getRouterSelectors();
 
export const selectPlaceById = createSelector(
  selectPlaces,
  selectRouteParams,
  (places, { id }) => places.find((place) => place.id === id),
);
```

## Recap
* **Key Points:**
  - I learned how to manage state using URL parameters with NgRx Router Store in Angular. I also integrated NgRx with Angular Router to handle state and navigation, keeping our components clean. This approach helps manage state better and combines with Router Selectors to easily read router data.

## Frequently Asked Questions
* **Key Points:**
  - How do I read URL route parameters in NgRx without using ActivatedRoute in components? The @ngrx/router-store package syncs Angular Router state into the NgRx store, allowing you to create selectors that read route parameters directly from the store. This removes the dependency on ActivatedRoute in your components and effects, making your code easier to test and keeping the router state as a single source of truth alongside the rest of your application state.
  - How do I install and configure @ngrx/router-store in Angular? Install the package with npm install @ngrx/router-store, then add provideRouterStore to your app configuration providers array. You can optionally provide a custom serializer to control which parts of the router snapshot are stored. Once configured, the router state is available in the store under the router key by default.
  - How do I create a selector to read a route parameter ID from the NgRx store? After configuring @ngrx/router-store, use the getRouterSelectors helper to get built-in selectors, including selectRouteParam. You can create your own selector by composing selectRouteParam with a specific parameter name like id. This selector can then be used in components or effects just like any other NgRx selector.
  - Why is storing URL route parameters in NgRx state better than reading them in every component? Centralizing route parameters in the NgRx store reduces duplication and keeps component code simpler because components only need to select state rather than inject and subscribe to ActivatedRoute. It also makes effects easier to write because they can read the current route ID directly from the store instead of needing to coordinate between the router and service calls.