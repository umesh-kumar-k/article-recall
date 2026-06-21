---
aliases:
  - NgRx Effects
Source 1: https://ngrx.io/guide/effects
---
# NgRx effects

## Introduction
* **Key Points:**
  - Effects are an RxJS powered side effect model for Store. Effects use streams to provide new sources of actions to reduce state based on external interactions such as network requests, web socket messages and time-based events.
  - In a service-based Angular application, components are responsible for interacting with external resources directly through services. Instead, effects provide a way to interact with those services and isolate them from the components. Effects are where you handle tasks such as fetching data, long-running tasks that produce multiple events, and other external interactions where your components don't need explicit knowledge of these interactions.
* **Technical Entities (Classes/Functions/APIs):** `@ngrx/effects`, `Store`, `Actions`, `createEffect`, `ofType`

## Key Concepts
* **Key Points:**
  - Effects isolate side effects from components, allowing for more pure components that select state and dispatch actions.
  - Effects are long-running services that listen to an observable of every action dispatched from the Store.
  - Effects filter those actions based on the type of action they are interested in. This is done by using an operator.
  - Effects perform tasks, which are synchronous or asynchronous and return a new action.

## Comparison with Component-Based Side Effects
* **Key Points:**
  - In a service-based application, your components interact with data through many different services that expose data through properties and methods. These services may depend on other services that manage other sets of data. Your components consume these services to perform tasks, giving your components many responsibilities.
  - The component has multiple responsibilities: Managing the state of the movies; Using the service to perform a side effect, reaching out to an external API to fetch the movies; Changing the state of the movies within the component.
  - Effects when used along with Store, decrease the responsibility of the component. In a larger application, this becomes more important because you have multiple sources of data, with multiple services required to fetch those pieces of data, and services potentially relying on other services.
  - Effects handle external data and interactions, allowing your services to be less stateful and only perform tasks related to external interactions.
* **Technical Entities (Classes/Functions/APIs):** `Store`, `select`, `dispatch`, `Actions`

## Writing Effects
* **Key Points:**
  - To isolate side effects from your component, you can create NgRx effects to listen for events and perform tasks.
  - Effects are injectable service classes with distinct parts:
    - An injectable Actions service that provides an observable stream of each action dispatched after the latest state has been reduced.
    - Metadata is attached to the observable streams using the createEffect function. The metadata is used to register the streams that are subscribed to the store. Any action returned from the effect stream is then dispatched back to the Store.
    - Actions are filtered using a pipeable ofType operator. The ofType operator takes one or more action types as arguments to filter on which actions to act upon.
    - Effects are subscribed to the Store observable.
    - Services are injected into effects to interact with external APIs and handle streams.
  - Note: Since NgRx v15.2, classes are not required to create effects. Learn more about functional effects here.
  - The loadMovies$ effect is listening for all dispatched actions through the Actions stream, but is only interested in the [Movies Page] Load Movies event using the ofType operator. The stream of actions is then flattened and mapped into a new observable using the exhaustMap operator. The MoviesService#getAll() method returns an observable that maps the movies to a new action on success, and currently returns an empty observable if an error occurs. The action is dispatched to the Store where it can be handled by reducers when a state change is needed. It's also important to handle errors when dealing with observable streams so that the effects continue running.
  - Note: Event streams are not limited to dispatched actions, but can be any observable that produces new actions, such as observables from the Angular Router, observables created from browser events, and other observable streams.
* **Technical Entities (Classes/Functions/APIs):** `Actions`, `createEffect`, `ofType`, `exhaustMap`, `mergeMap`, `concatMap`, `switchMap`, `EMPTY`
* **Code Snippet:**
```typescript
import { Injectable, inject } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { EMPTY } from 'rxjs';
import { map, exhaustMap, catchError } from 'rxjs/operators';
import { MoviesService } from './movies.service';

@Injectable()
export class MoviesEffects {
  private actions$ = inject(Actions);
  private moviesService = inject(MoviesService);

  loadMovies$ = createEffect(() => {
    return this.actions$.pipe(
      ofType('[Movies Page] Load Movies'),
      exhaustMap(() =>
        this.moviesService.getAll().pipe(
          map((movies) => ({
            type: '[Movies API] Movies Loaded Success',
            payload: movies,
          })),
          catchError(() => EMPTY)
        )
      )
    );
  });
}
```

## Handling Errors
* **Key Points:**
  - Effects are built on top of observable streams provided by RxJS. Effects are listeners of observable streams that continue until an error or completion occurs. In order for effects to continue running in the event of an error in the observable, or completion of the observable stream, they must be nested within a "flattening" operator, such as mergeMap, concatMap, exhaustMap, and switchMap.
  - The loadMovies$ effect returns a new observable in case an error occurs while fetching movies. The inner observable handles any errors or completions and returns a new observable so that the outer stream does not die. You still use the catchError operator to handle error events, but return an observable of a new action that is dispatched to the Store.
* **Technical Entities (Classes/Functions/APIs):** `catchError`, `of`, `mergeMap`, `concatMap`, `exhaustMap`, `switchMap`
* **Code Snippet:**
```typescript
loadMovies$ = createEffect(() => {
  return this.actions$.pipe(
    ofType('[Movies Page] Load Movies'),
    exhaustMap(() =>
      this.moviesService.getAll().pipe(
        map((movies) => ({
          type: '[Movies API] Movies Loaded Success',
          payload: movies,
        })),
        catchError(() =>
          of({ type: '[Movies API] Movies Loaded Error' })
        )
      )
    )
  );
});
```

## Functional Effects
* **Key Points:**
  - Functional effects are also created by using the createEffect function. They provide the ability to create effects outside the effect classes.
  - To create a functional effect, add the functional: true flag to the effect config. Then, to inject services into the effect, use the inject function.
  - It's recommended to inject all dependencies as effect function arguments for easier testing. However, it's also possible to inject dependencies in the effect function body. In that case, the inject function must be called within the synchronous context.
* **Technical Entities (Classes/Functions/APIs):** `createEffect`, `functional: true`, `inject`
* **Code Snippet:**
```typescript
export const loadActors = createEffect(
  (
    actions$ = inject(Actions),
    actorsService = inject(ActorsService)
  ) => {
    return actions$.pipe(
      ofType(ActorsPageActions.opened),
      exhaustMap(() =>
        actorsService.getAll().pipe(
          map((actors) =>
            ActorsApiActions.actorsLoadedSuccess({ actors })
          ),
          catchError((error: { message: string }) =>
            of(
              ActorsApiActions.actorsLoadedFailure({
                errorMsg: error.message,
              })
            )
          )
        )
      )
    );
  },
  { functional: true }
);

export const displayErrorAlert = createEffect(
  () => {
    return inject(Actions).pipe(
      ofType(ActorsApiActions.actorsLoadedFailure),
      tap(({ errorMsg }) => alert(errorMsg))
    );
  },
  { functional: true, dispatch: false }
);
```

## Registering Effects
* **Key Points:**
  - Effect classes and functional effects are registered using the provideEffects method.
  - At the root level, effects are registered in the providers array of the application configuration.
  - Effects start running immediately after instantiation to ensure they are listening for all relevant actions as soon as possible. Services used in root-level effects are not recommended to be used with services that are used with the APP_INITIALIZER token.
  - Feature-level effects are registered in the providers array of the route config. The same provideEffects() method is used to register effects for a feature.
  - Registering an effects class multiple times (for example in different lazy loaded features) does not cause the effects to run multiple times.
* **Technical Entities (Classes/Functions/APIs):** `provideEffects`, `bootstrapApplication`, `provideStore`, `APP_INITIALIZER`
* **Code Snippet:**
```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideStore(),
    provideEffects(MoviesEffects, actorsEffects),
  ],
});
```

## Alternative Way of Registering Effects
* **Key Points:**
  - You can provide root-/feature-level effects with the provider USER_PROVIDED_EFFECTS.
* **Technical Entities (Classes/Functions/APIs):** `USER_PROVIDED_EFFECTS`, `multi: true`
* **Code Snippet:**
```typescript
providers: [
  MoviesEffects,
  {
    provide: USER_PROVIDED_EFFECTS,
    multi: true,
    useValue: [MoviesEffects],
  },
];
```

## Incorporating State
* **Key Points:**
  - If additional metadata is needed to perform an effect besides the initiating action's type, we should rely on passed metadata from an action creator's props method.
  - However, there may be cases when the required metadata is only accessible from state. When state is needed, the RxJS withLatestFrom or the @ngrx/effects concatLatestFrom operators can be used to provide it.
  - For performance reasons, use a flattening operator like concatLatestFrom to prevent the selector from firing until the correct action is dispatched.
  - To learn about testing effects that incorporate state, see the Effects that use State section in the testing guide.
* **Technical Entities (Classes/Functions/APIs):** `props`, `withLatestFrom`, `concatLatestFrom`, `Store.select`
* **Code Snippet:**
```typescript
addBookToCollectionSuccess$ = createEffect(
  () => {
    return this.actions$.pipe(
      ofType(CollectionApiActions.addBookSuccess),
      concatLatestFrom((_action) =>
        this.store.select(fromBooks.getCollectionBookIds)
      ),
      tap(([_action, bookCollection]) => {
        if (bookCollection.length === 1) {
          window.alert('Congrats on adding your first book!');
        } else {
          window.alert(
            'You have added book number ' + bookCollection.length
          );
        }
      })
    );
  },
  { dispatch: false }
);
```

## Using Other Observable Sources for Effects
* **Key Points:**
  - Because effects are merely consumers of observables, they can be used without actions and the ofType operator. This is useful for effects that don't need to listen to some specific actions, but rather to some other observable source.
  - For example, imagine we want to track click events and send that data to our monitoring server. This can be done by creating an effect that listens to the document click event and emits the event data to our server.
  - An example of the @ngrx/effects in module-based applications is available at the following link.
* **Technical Entities (Classes/Functions/APIs):** `fromEvent`, `concatMap`, `dispatch: false`
* **Code Snippet:**
```typescript
trackUserActivity$ = createEffect(
  () => {
    return fromEvent(document, 'click').pipe(
      concatMap((event) =>
        this.userActivityService.trackUserActivity(event)
      )
    );
  },
  { dispatch: false }
);
```


---

# Why @Effects?

* **Key Points:**
  - In a simple ngrx/store project without ngrx/effects there is really no good place to put your async calls. Suppose a user clicks on a button or types into an input box and then we need to make an asynchronous call.
  - We don't want to put the logic to do our async call right in the dumb component since we want to keep it dumb!
  - Then the smart component gets the event and it's handler function is triggered, but we don't want to put the async login right in there because we want to keep it lean and only dippingatching actions to our store so that the store can modify the state!
  - The store only handles actions in the reducer, and reducer are meant to be pure functions so where are we supposed to logically put our async calls so that we can put their response data in the store? The answer, friends, is @Effects! You can almost think of your Effects as special kinds of reducer functions that are meant to be a place for you to put your async calls in such a way that the returned data can then be easily inserted into the store's internal state for the application.
* **Technical Entities (Classes/Functions/APIs):** `ngrx/store`, `ngrx/effects`, `@Effects`, `@Output`, `store`

## A Separate Service For Async
* **Key Points:**
  - You might be thinking, "What if you have the smart component just communicate with another service that calls for async data, and then when that call comes back the service dispatchs an event to the store with the returned data as a payload?", and in a way you'd be right! In Angular 2 a service is just a regular old TypeScript class with the @Injectable metadata, and when working with @Effects you make a single "Effect Class" or "Effect Service" that then contains various @Effect functions, each corresponding to an action dispatched by your ngrx store.

## Installing Ngrx/Effects
* **Key Points:**
  - The first thing you'll need to do is install @ngrx/effects via npm: `npm install @ngrx/effects --save`
* **Technical Entities (Classes/Functions/APIs):** `@ngrx/effects`, `npm`

## Add RunEffects To Your NgModule
* **Key Points:**
  - Next, you need to tell your application that you wan to use effects. In the imports array in your NgModule add a line where you call EffectsModule.run. Pass in the class (or classes) that you are using as the "Effects Class" (or classes).
* **Technical Entities (Classes/Functions/APIs):** `EffectsModule.run`, `StoreModule.provideStore`, `AppModule`
* **Code Snippet:**
```typescript
import { StoreModule } from '@ngrx/store';
import { MainStoreReducer } from './state-management/reducers/main-store-reducer';
import { EffectsModule } from '@ngrx/effects';
import { MainEffects } from "./state-management/effects/main-store-effects";
@NgModule({
  imports: [
    StoreModule.provideStore({mainStoreReducer}),
    EffectsModule.run(MainEffects),
  ]
  ]
})
export class AppModule { }
```

## Create An Effects Class
* **Key Points:**
  - The name of this class should be the same as what you reference in your NgModule step above. At it's core, the Effects Class in simply just an Angular 2 Service.
* **Technical Entities (Classes/Functions/APIs):** `@Injectable`, `Actions`, `Observable`
* **Code Snippet:**
```typescript
import {Effect, Actions, toPayload} from "@ngrx/effects";
import {Injectable} from "@angular/core";
import {Observable} from "rxjs";
@Injectable()
export class MainEffects {
  
  constructor(private action$: Actions) { }
}
```

## Hello World @Effect
* **Key Points:**
  - Were using the TypeScript metadata to label our variable update$ (the $ is commonly used as a suffix for variables whose value is an observable) as an "ngrx effect" that will be triggered when we dispatch actions with the store (the same was we always send actions to the reducer or reducers).
  - Then we see "this.action$.ofType('SUPER_SIMPLE_EFFECT')". Remeber, we're translating the dispatched event into an observable, so .ofType means your taking in an observable and then returning the observable only if it is of that type.
  - Then we do switchMap because we want to "switch over" from the original observable to a brand new observable. What you want to return from an ngrx/effect is an observable to an action, and when it all plays out on screen the intial action will be dispatched from the component (or some service). It will then bypass the reducer and be handled by an effect. This effect will then return an observable to some action, and the new action will then be handled in the reducer.
* **Technical Entities (Classes/Functions/APIs):** `@Effect`, `ofType`, `switchMap`, `Observable.of`
* **Code Snippet:**
```typescript
@Effect() update$ = this.action$
    .ofType('SUPER_SIMPLE_EFFECT')
    .switchMap( () =>
      Observable.of({type: "SUPER_SIMPLE_EFFECT_HAS_FINISHED"})
    );
```

## Payload Example
* **Key Points:**
  - In this next example we get a little bit fancier by dealing with payloads. We can both accept a payload from the initial action and return a payload.
  - In the code below immediately after we get an action obervable of type "SEND_PAYLOAD_TO_EFFECT" call "map(toPayload) on it. So we take in an observable with an action and a payload a bunch of other stuff, and we return an Observable with just the payload.
  - Then we do a switchMap because we want to switch over to our response observable, but we still have that payload as an argument in our switchMap function.
  - You can then see that, following a very Redux-ish pattern, we have an object with a type and a payload. The payload can be an object containing basically whatever you want. We then return that as an observable, and we're done!
* **Technical Entities (Classes/Functions/APIs):** `toPayload`, `map`, `switchMap`, `Observable.of`
* **Code Snippet:**
```typescript
@Effect() effectWithPayloadExample$ = this.action$
    .ofType('SEND_PAYLOAD_TO_EFFECT')
    .map(toPayload)
    .switchMap(payload => {
      console.log('the payload was: ' + payload.message);
      return Observable.of({type: "PAYLOAD_EFFECT_RESPONDS", payload: {message: "The effect says hi!"}})
    });
```

## Async Effect With A Timer
* **Key Points:**
  - Our next barrel of fun contains a timer. This is similar to the "setTimeout" as you might have seen it before in JavaScript. However, we want this timer to return, of course, and observable so we'll use Observable.timer().
  - Notice that we're taking in a payload as the number of seconds to set on the timer. The key thing to realize is that where we would normally have a callback function after asynchronous event we now just switchMap off of it. Once the timer completes then we return an observable to an action "TIMER_FINISHED" which then get's handled in the reducer.
* **Technical Entities (Classes/Functions/APIs):** `Observable.timer`, `switchMap`, `Observable.of`
* **Code Snippet:**
```typescript
@Effect() timeEffect = this.action$
    .ofType('SET_TIMER')
    .map(toPayload)
    .switchMap(payload =>
      Observable.timer(payload.seconds * 1000)
        .switchMap(() =>
          Observable.of({type: "TIMER_FINISHED"})
        )
    )
```

## Pulling Data From Firebase With AngularFire2
* **Key Points:**
  - I personally like use Firebase a lot. This is a NoSQL database from Google that's super high performance and easy to use. The AngularFire2 library fits in especially awesomely here because it allows you to query your database in a way such that the result is an observable.
* **Technical Entities (Classes/Functions/APIs):** `angularfire2`, `Firebase`, `Observable`

### Async Effect Pull Array From Firebase
* **Key Points:**
  - Ok, let's jump into an @Effect that pulls data from firebase! So we get an action, we see that it's of type PULL_ARRAY_FROM_FIREBASE, and then we switchMap over so we can start a new Observable.
  - Here's where our async call comes in! In this case we're using the Firebase Realtime Database, and the slick AngularFire2 library gives us a very nice api. The key thing to realize here is that in the AngularFire2 library af.database.list returns an Observable! The string you pass in allows you to "drill down" into your NoSQL JSON object data store to pull some given node as an array.
  - Next, we switchMap over to a new Observable, the one we want to return. We'll then return an Observable to an action of type, "GOT_FIREBASE_ARRAY" with a payload that contains the array we got back from Firebase.
* **Technical Entities (Classes/Functions/APIs):** `af.database.list`, `switchMap`, `Observable.of`
* **Code Snippet:**
```typescript
@Effect() pullArrayFromFirebase$ = this.action$
    .ofType('PULL_ARRAY_FROM_FIREBASE')
    .switchMap( () => 
        this.af.database.list('/cypherapp/rooms/')
        .switchMap(result =>
          Observable.of({type: "GOT_FIREBASE_ARRAY", payload: {pulledArray:  result}})
          )
    )
```

### Async Effect Pull Object From Firebase
* **Key Points:**
  - So, now we know how to pull a node from Firebase as an array, but what if we want to pull it as a regular JavaScript Object? Well, all we need to do is change af.database.object instead of af.database.list, and the rest of the code is exactly the same!
* **Technical Entities (Classes/Functions/APIs):** `af.database.object`, `switchMap`, `Observable.of`
* **Code Snippet:**
```typescript
@Effect() pullObjectFromFirebase$ = this.action$
    .ofType('PULL_OBJECT_FROM_FIREBASE')
    .switchMap( () =>
      this.af.database.object('/cypherapp/rooms/')
        .switchMap(result =>
          Observable.of({type: "GOT_FIREBASE_OBJECT", payload: {pulledObject: result}})
        )
    )
```

## SwitchMap Just Takes A Function
* **Key Points:**
  - We're using a lot of switchMaps here so it's important to understsand the magic behind it.
  - The first (and often only) parameter to switchMap is a function that is applied to an item emitted by a source Observable and returns an Observable.
  - It's also worth noting that the fat arrows we have here are really just shorthand for saying that we're making a function. Since the one observable is the only thing in the body of the function and it's only a single line we can omit the curly braces and the return keyword.
* **Technical Entities (Classes/Functions/APIs):** `switchMap`
* **Code Snippet:**
```typescript
@Effect() pullObjectFromFirebase$ = this.action$
    .ofType('PULL_OBJECT_FROM_FIREBASE')
    .switchMap(() => {
      console.log('in the first switchMap!');
      return this.af.database.object('/cypherapp/rooms/')
        .switchMap(result => {
          console.log('oh yeah, we got the result!');
          return Observable.of({type: "GOT_FIREBASE_OBJECT", payload: {pulledObject: result}})
        })
    });
```