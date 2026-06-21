---
aliases:
Source 1: https://angular.love/how-to-start-flying-with-angular-and-ngrx
---
# How to Start Flying with Angular and NgRx

## How NgRx works
* **Key Points:**
  - There are five parts that constitute NgRx: Store, Reducers (and Meta-Reducers), Actions, Selectors, Effects.
  - Your application's state is maintained in the store. The store is immutable.
  - Your application's components can subscribe to the store and get automatic updates of state through selectors.
  - Selectors enable components to get a slice (a part) of your application's state, and also mutate state with selector functions.
  - Actions modify the state of the store by using reducers (functions) that enable changes while keeping it immutable.
  - Meta-Reducers (not shown) are hooks where you can pre or post-process actions before they get invoked.
  - Effects occur as a result from actions, and can also create actions when called. Effects primary responsibility is to create async side-effects (like service calls to APIs), that ultimately generate other actions.
  - This is a big change in the way that traditional applications are built, and creates a paradigm that greatly simplifies complex applications.
  - NgRx can also simplify the architecture of your application, as you can use it to replace property and event bindings.
* **Technical Entities (Classes/Functions/APIs):** `Store`, `Reducers`, `Meta-Reducers`, `Actions`, `Selectors`, `Effects`, `NgRx`, `RxJS`

## Before you Start Coding
* **Key Points:**
  - Before we start, I need to point out that there is a bit of a learning curve with NgRx. I recommend getting familiar with RxJS Operators and Observables in particular.
  - The last step before you see NgRx in action is to add on the NgRx Redux Devtools Extension to Chrome. You won't be able to use this until you've setup NgRx, but it will enable you to view the store data while you are developing your application.
* **Technical Entities (Classes/Functions/APIs):** `RxJS Operators`, `Observables`, `NgRx Redux Devtools Extension`

## NgRx with Goose Weather
* **Key Points:**
  - The overall Goose Weather application is essentially a main weather component displaying several children components that are Material Cards. Additionally, there is an input field available in the main toolbar that lets you select a different location.
  - The goal for us is to: Dispatch the weather to the store on initial application startup; Dispatch the weather to the store when the location has changed via the toolbar selection; Connect all display cards to be automatically updated when the store values are updated.

### Install Dependencies and Scaffold the Project
* **Key Points:**
  - First, we're going to install the NgRx dependencies and use the NgRx Schematics to bootstrap the application.
* **Technical Entities (Classes/Functions/APIs):** `@ngrx/store`, `@ngrx/effects`, `@ngrx/store-devtools`, `@ngrx/schematics`, `ng generate store`, `ng generate action`

### Defining State and Reducers
* **Key Points:**
  - With the main project scaffolded, we are going to set up what your application state will look like next. The way you can define this is by first defining your state objects, and then by defining the reducers which dictate how state changes occur when actions are dispatched.
  - Actions can operate with or without reducers. A common flow is to fire off an action, and then use a reducer to handle how that action interacts with the store.
  - Here we build out what the application state looks like. There are many ways to do this, and I'm just using a naive approach here as this application's interactions with NgRx are very simple. The structures here are defining application state that includes: Location, Weather.
  - Here, we define two reducers (1) weather and (2) location. If you notice they return payload objects. This is a normal convention with NgRx. The payload is what the store is going to update itself to be. If the payload only consists of one value, it's often convention to not include a payload wrapper and just return the value.
  - Here, we are defining selectors. Selectors are ways that your application components access the state directly. The nomenclature for this is to either refer to the state as a whole, or if you are selecting specific parts, you are selecting a slice of state. Selectors can also transform state, and have many other potential functions that can be utilized for interacting with the store.
  - Finally, at the end of the file we define meta-reducers. These are hooks that enable you to pre-process actions or add middleware. You could define them to look for actions like INIT or UPDATE, which are the default actions that NgRx does when the application starts up or when the store changes respectively. Meta-reducers are also a great way to handle localStorage.
* **Technical Entities (Classes/Functions/APIs):** `ActionReducerMap`, `MetaReducer`, `WeatherState`, `LocationState`, `AppState`, `selectWeather`, `selectError`, `metaReducers`
* **Code Snippet:**
```typescript
export interface WeatherState {
  weatherData: WeatherData| null;
}

const initialWeatherState: WeatherState = {
  weatherData: null
};

export interface LocationState {
  location: LocationData| null;
  error: string| null;
}

const initialLocationState: LocationState = {
  location: null,
  error: null
};

export interface AppState {
  weather: WeatherState;
  location: LocationState;
}

export function weatherReducer(state: WeatherState = initialWeatherState, action: WeatherAction): WeatherState {
  switch (action.type) {
    case WeatherActionTypes.LoadWeather:
      return {
        weatherData: action.payload.weatherData
      };

    default:
      return state;
  }
}

export function locationReducer(state: LocationState = initialLocationState, action: LocationAction): LocationState {
  switch (action.type) {
    case LocationActionTypes.LoadLocations:
      return {
        location: action.payload.locationData,
        error: null
      };

    case LocationActionTypes.LocationsError:
      return {
        location: null,
        error: action.payload.error
      };

    default:
      return state;
  }
}

export const reducers: ActionReducerMap<AppState> = {

  weather: weatherReducer,
  location: locationReducer
};

export const selectWeather = (state: AppState) => state.weather.weatherData;

export const selectError = (state: AppState) => state.location.error;

export const metaReducers: MetaReducer<any>[] = !environment.production ? [] : [];
```

### Defining Actions
* **Key Points:**
  - The next step is to define the actions that will dispatch changes to the store using the application state and reducers we just defined. Actions should be treated as events, and live close to where they are dispatched from.
  - It's also a good idea to include the name of the page that the action is being fired from when you declare the action type. So for example "[Home Page] load locations" would be an action of type LoadLocation fired from the Home Page.
  - What is this file doing? Well it's defining what your location actions will look like. In the first section, we are defining the types of actions.
  - Next, you see classes for each action type being defined. Notice the use of payload here to define what is being sent with the action. As I mentioned before, if the action only returns a single argument then normally you would not wrap it with payload. I included payload here to demonstrate convention for larger applications.
  - Finally, you see these action classes being exported out to the project. These actions are how your code will interact with your store. Basically, think of it as a protocol for your app to interact with reducers.
  - On a side note, the LoadWeather action here is considered a "fetch" action in NgRx. Fetch actions normally follow the convention to have (1) Load, (2) LoadSuccess, or (3) LoadFailed. This is to handle the load itself, success, and failure respectively.
* **Technical Entities (Classes/Functions/APIs):** `Action`, `LocationActionTypes`, `LoadLocations`, `LocationsError`, `WeatherActionTypes`, `LoadWeather`
* **Code Snippet:**
```typescript
import { Action } from '@ngrx/store';
import { LocationData } from '../models/location-data/location-data';

export enum LocationActionTypes {
  LoadLocations = '[Home Page] Load Locations',
  LocationsError = '[Home Page] Locations Error'
}

export class LoadLocations implements Action {
  readonly type = LocationActionTypes.LoadLocations;

  constructor(readonly payload: {locationData: LocationData}) {

  }
}

export class LocationsError implements Action {
  readonly type = LocationActionTypes.LocationsError;

  constructor(readonly payload: {error: string}) {

  }
}

export type ActionsUnion = LoadLocations | LocationsError;
```

### Creating an Effect for the Actions
* **Key Points:**
  - With the actions and reducers setup, the last step is to build out an effect that will run whenever the location is updated. Basically we want to set it up so anytime the location changes, a new weather forecast is retrieved.
  - Effects occur as a result from actions, and can also create actions when called. Effects primary responsibility is to create async side-effects (like service calls to APIs), that ultimately generate other actions.
  - So what's this doing and how does this work?
    - Using the Effect decorator, the application's instance of NgRx becomes aware of this effect on startup.
    - This effect listens for actions of type LoadLocations and then uses mergeMap to pass the location data from the action to a call to the applications weather service.
    - Then the effect will dispatch a new LoadWeather action to include the new forecast information.
    - When the dispatch to the store is complete, the store is updated
    - If there are any errors that occur in the service call, the effect dispatches a LocationsError action with the error to the store.
* **Technical Entities (Classes/Functions/APIs):** `Effects`, `ofType`, `mergeMap`, `catchError`, `LoadWeather`, `LocationsError`
* **Code Snippet:**
```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { LoadWeather } from './weather.actions';
import { map, mergeMap, catchError } from 'rxjs/operators';
import { Store } from '@ngrx/store';
import { WeatherService } from './weather.service';
import { LocationActionTypes, LocationsError, LoadLocations } from './location.actions';
import { of } from 'rxjs';

@Injectable()
export class WeatherEffects {

  @Effect()
  loadLocation$ = this.actions$
    .pipe(
      ofType<LoadLocations>(LocationActionTypes.LoadLocations),
      mergeMap((action) => this.weatherService.getWeather(action.payload.locationData)
      .pipe(
        map(weather => {
          return (new LoadWeather({weatherData: weather}));
        }),
        catchError((errorMessage) => of(new LocationsError({error: errorMessage})))
      ))
  );

  constructor(private actions$: Actions, private store: Store<AppState>, private weatherService: WeatherService) { }

}
```

### Connecting NgRx to Angular Components
* **Key Points:**
  - With the reducers, actions, and effects setup, let's now connect our components to be able to receive streamed information from the store.
  - What did this do? In both the savePosition and onSelectionChanged methods, a change in location triggers a dispatch of LoadLocations to update the store. When the LoadLocations action fires, the WeatherEffect we created earlier will fire off to retrieve the forecast for that location.
  - Notice also that before making a service call in _onSelectionChanged_, a _LoadWeather_ action is dispatched with a _null_ value for _weatherData_. This was just to create the condition where the card's progress spinners show when the weather data is being retrieved.
  - This creates an observable that becomes a stream of data from the store using the selectWeather selector.
  - As you see here, the async pipe handles the observable subscription and only shows if the data when present (otherwise it shows a spinner). This use of unwrapping the observable with the async pipe is considered a best practice, as it handles the subscribe and destroy of the observable.
* **Technical Entities (Classes/Functions/APIs):** `Store<AppState>`, `dispatch`, `select`, `async pipe`

## File Cleanup
* **Key Points:**
  - The last step before we run the application is just to move some of the files around so that they're coupled more closely with were they are utilized. The schematics that we used to generate our application created folders for effects, reducers, and actions. Additionally, if you notice I included a services folder that include the main service that retrieves the weather forecast. Now that we have built everything out, lets move the action files, effects files, and service files to be within the weather component folder. This is because the weather component is what uses these actions, effects, and services directly. So now the NgRx files are coupled with the component that is using them.

## Putting it All Together
* **Key Points:**
  - Now that you've built out NgRx in the application and connected it to the components, go to your terminal and run ng serve to see it in action. If you open the Redux Devtools extension, you can also see how the state is updated when the application runs.

## Closing Thoughts
* **Key Points:**
  - I hope my post here has given you a good intro to using NgRx. My weather application is only using a few actions and reducers, but NgRx is much more robust and can do many more things. I encourage you to review the official NgRx site documentation and get more acquainted with the technology.