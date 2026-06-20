---
aliases:
  - Routing Architecture
Source 1: https://angular.love/angular-router-everything-you-need-to-know-about
Source 2:
---
# Angular Router – everything you need to know about

## Routing Configuration
* **Key Points:**
  - To enable routing in your application the very first thing you need to do is using provideRouter function to set up necessary providers
  - This way is still relatively new as it was introduced with standalone components. In module-based applications it's been done using static method forRoot of RouterModule

## Routes definition
* **Key Points:**
  - Router is ready to use but we need to define Routes so an array of Route objects. It's a recipe for Angular how to navigate through the application. The two most important ingredients are: a path and a component associated with.
  - Path is a piece of text which builds a URL displayed in the browser's address bar. It can be static as well as dynamic. Dynamic path (param) starts with colon which indicates this piece of URL can be replaced with some value. Text after the colon is param's name which can be used to retrieve that value.
  - Paths in the URL are separated by slashes which creates a hierarchical structure. This characteristic is reflected in route configuration by children parameter. It accepts nested routes array (Routes) creating a tree of routes.
  - Order of the routes matters. Angular uses a first-match wins strategy and takes the first route it's satisfied with. That's why more specific routes should be declared before less specific ones.
  - Empty route defines pathMatch parameter. It affects matching process and accepts two values:
    - "prefix" (default) – this option matches the route when the configured path is a prefix of the entire URL
    - "full" – This option matches the route only if the entire URL matches the configured path
  - Two asterisks (**) is a wildcard and matches any URL so it should be always defined as the last route.
  - Sometimes you may need to redirect user to another page. To do this use the redirectTo property. It accepts static path user should be redirected to or function to handle more complicated cases
* **Technical Entities (Classes/Functions/APIs):** `provideRouter`, `RouterModule.forRoot()`, `Routes`, `Route`, `pathMatch`, `redirectTo`, `children`
* **Code Snippet:**
```typescript
const ROUTES: Routes = [
   { path: 'dashboard', component: DashboardComponent },
   { path: 'products', component: ProductsComponent, children: [
      { path: 'top', component: TopProductsComponent },
      { path: ':id', component: ProductDetailsComponent }
   ]},
   { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
   { path: '**', component: PageNotFoundComponent }
]
```

## Lazy loading
* **Key Points:**
  - Lazy loading is a basic but quite effective optimization technique. You can use it in your routing configuration to defer fetching components (or modules) until user navigates to a specific route instead of loading all of the code when application starts. It breaks your application code into smaller bundles (chunks) and loads them asynchronously which reduces initial bundle size improving performance.
  - Just passing a function to loadComponent property which asynchronously imports the component
  - It's very important to use the import function inside the loadComponent function. This way we don't have an import statement on top of the file which would make the component eagerly loaded anyway.
  - In module-based applications it's been done by using loadChildren function to load NgModule which imports RouterModule defining routes using forChild static method
  - Having one huge constant with all routes in application definitely sounds like an antipattern. We want to keep them separated in files on the feature level. But we can't just import a file because it would be eagerly loaded. Fortunately there is a simple and effective solution. Function assigned to loadChildren can lazy load not only a module but also a file with your routes
* **Technical Entities (Classes/Functions/APIs):** `loadComponent`, `import()`, `loadChildren`, `RouterModule.forChild()`
* **Code Snippet:**
```typescript
const ROUTES: Routes = [
  {
    path: 'user',
    loadComponent: () =>
      import('./user.component').then((c) => c.UserComponent),
  },
];
```

## Route Matcher
* **Key Points:**
  - Route matcher is a function that allows you to create your custom logic for matching routes. It's useful if you need more complex matching than default matcher offers. It returns UrlMatchResult, so consumed segments and an object holding resolved values for path parameters, or null If the URL doesn't meet your requirements.
  - Property consumed contains URL segments we have used (so "consumed") to match the route as every segment can be used only once for matching
  - Property posParams is an object defining path params.
* **Technical Entities (Classes/Functions/APIs):** `UrlMatcher`, `UrlMatchResult`, `UrlSegment`, `posParams`

## Route Guards
* **Key Points:**
  - Route guards serve as checkpoints for navigation, allowing you to control whether a user can navigate to or leave a particular route. They are essential for implementing security measures, managing authentication, and enforcing access control within your Angular application.
  - Functional guards were introduced in Angular 14. By this time developers had been defining guards as services implementing adequate interface. This solution is marked as deprecated but it's quite a chance you can still encounter it in some projects.

### CanActivate
* **Key Points:**
  - This guard determines whether a route can be activated or not. It's commonly used for implementing authentication or authorization checks before allowing access to protected routes.
  - One route can be protected by many guards so: if all of them return true navigation continues; if any of them return false navigation is canceled; if any of them return a UrlTree or RedirectCommand, the current navigation is canceled and user is redirected to route described by one.
  - Notice that guard functions are executed inside injection context so we can safely use inject function to get required dependencies.
* **Technical Entities (Classes/Functions/APIs):** `CanActivateFn`, `MaybeAsync`, `GuardResult`, `RedirectCommand`, `inject()`, `CanActivate` (service interface)

### CanActivateChild
* **Key Points:**
  - CanActivateChild allows control of whether child routes can be activated at a parent level.
  - We can use the Component-Less Route pattern and create a "middleware" route with an empty path and no component assigned. It doesn't change the way routing works and allows to group certain routes together.
  - It's also worth mentioning that since component-less route doesn't have any assigned component, it doesn't affect the way by which you could retrieve routing parameters in feature components using ActivatedRoute.
* **Technical Entities (Classes/Functions/APIs):** `CanActivateChildFn`, `ActivatedRoute`

### CanDeactivate
* **Key Points:**
  - This guard checks if a route can be deactivated or not. It's often utilized for prompting users with confirmation dialogs when they attempt to leave a page with unsaved changes.
  - It's a good practice to define an interface that every component assigned to route protected by CanDeactivate should implement
* **Technical Entities (Classes/Functions/APIs):** `CanDeactivateFn`, `SafeDeactivate`

### CanMatch
* **Key Points:**
  - CanMatch guard is called at the stage when Angular tries to match url with route configuration and makes it skippable.
  - If any of CanMatch guards return false the route is skipped for matching and further route configurations are processed instead. This is especially useful if you want to associate two different components with the same path segment.
* **Technical Entities (Classes/Functions/APIs):** `CanMatchFn`

### CanLoad
* **Key Points:**
  - CanLoad guard has been deprecated and replaced with CanMatch. The motivation to deprecate it was that lazy loading should serve solely as an optimization technique and should not function as an architectural feature controlling whether to lazy load a module based on some internal condition. Besides that it isn't as universal as CanMatch. CanLoad works only if you use lazy loading and you try to lazy load a module – it doesn't work for lazy-loaded standalone components.
* **Technical Entities (Classes/Functions/APIs):** `CanLoadFn` (deprecated)

### Resolver
* **Key Points:**
  - While not being strictly a guard, resolver is often grouped with route guards as it allows you to pre-fetch some of the data when the user navigates from one route to another. The router waits for the data to be resolved before the route is finally activated.
  - If you encounter an error while resolving the data you can redirect the user to another page using RedirectCommand.
  - Resolvers for the route aren't defined in array, like guards, but in the object assigned to resolve property. The key associated with resolver can be used to retrieve resolved data in component from ActivatedRoute.
  - To increase user experience even more we can let them know that the navigation is being processed. Here router events come handy
* **Technical Entities (Classes/Functions/APIs):** `ResolveFn`, `RedirectCommand`, `resolve`, `ActivatedRoute.data`

### Rerunning guards and resolvers
* **Key Points:**
  - Guards and resolvers always run when a route is activated or deactivated. When a route is unchanged you can define a policy whether they rerun or not using runGuardsAndResolvers property.
  - If you need even more flexibility you can also define a function which accepts snapshots of routes you navigate from and to as parameters and returns boolean
* **Technical Entities (Classes/Functions/APIs):** `runGuardsAndResolvers`, `RunGuardsAndResolversFn`

## Setting up page title
* **Key Points:**
  - Page title is a piece of text you can see on a tab in your browser next to favicon. Setting meaningful titles for your routes increases user experience and positively affects SEO.
  - The most basic way is just to pass a value to title property in route definition. However, passing static strings isn't a convenient way and we often need to create title based on some dynamic value. In this case Angular allows us to pass a resolver to dynamically create a title.
  - If you have a generic part of the title, like your application name, which should be included in many (or all) routes, you can define a custom title strategy.
* **Technical Entities (Classes/Functions/APIs):** `title`, `TitleStrategy`, `Title`, `buildTitle`, `updateTitle`

## Defining provider for route
* **Key Points:**
  - Since Angular 14 it's possible to define providers in route configuration. This way dependency is provided in Environment Injector. This type of injector is created along with creation of dynamically loaded components. Angular looks into it right after traversing Element Injector's tree. This provider is available for route's component and its descendants
* **Technical Entities (Classes/Functions/APIs):** `providers` (route property), `Environment Injector`

## Router Features
* **Key Points:**
  - Function provideRouter accepts as parameter not only routes but also router features. These can be enabled by calling special functions but if your application is still module-based these features can be configured using an options object (second parameter of RouterModule.forRoot method).

### Component Input Binding
* **Key Points:**
  - It's a very useful feature introduced in version 16 which allows to bind routing-related properties, such as path params, query params, resolvers results and data (additional developer-defined data provided in route by data property), directly to inputs of component associated with route without the necessity of injecting ActivatedRoute service.
  - Using input binding saves a lot of boilerplate
  - As we don't bind values to those inputs manually in template but Angular does it automatically, not minding what type is declared, it's worth paying attention to type safety using transform functions to avoid errors in runtime.
* **Technical Entities (Classes/Functions/APIs):** `withComponentInputBinding`, `input()`, `transform`, `numberAttribute`

### Router configuration options
* **Key Points:**
  - The withRouterConfig function is a wrapper to pass a configuration object without creating a separate feature function for each property.
  - canceledNavigationResolution: Configure how the router should restore state when a navigation is cancelled
  - urlUpdateStrategy: Defines when the router should update the browser URL
  - onSameUrlNavigation: Determines how to handle navigation request to current URL
  - paramsInheritanceStrategy: Defines how the router merges parameters, data and resolved values from parent to child routes
  - resolveNavigationPromiseOnError: If set to true and navigation error occurs the navigation Promise will resolve with value false (like for other navigation fails e.g. guard rejection) instead of being rejected.
* **Technical Entities (Classes/Functions/APIs):** `withRouterConfig`, `canceledNavigationResolution`, `urlUpdateStrategy`, `onSameUrlNavigation`, `paramsInheritanceStrategy`, `resolveNavigationPromiseOnError`

### Preloading strategy
* **Key Points:**
  - With some core features, you can know in advance that a user will lazy load a component (module). In that case it might be useful to preload the component (module) and avoid unnecessary latency – it isn't included in the initial bundle yet ready to use (almost) at the start.
  - To customize preloading strategy use withPreloading function. It accepts a class that extends PreloadingStrategy abstract class.
  - Angular offers two predefined preloading strategies: NoPreloading (default) – doesn't preload any chunk; PreloadAllModules – preloads all chunks as quickly as possible.
  - Of course you can create your custom preloading strategy like one based on the flag passed to the data parameter.
* **Technical Entities (Classes/Functions/APIs):** `withPreloading`, `PreloadingStrategy`, `NoPreloading`, `PreloadAllModules`

### Scrolling Memory
* **Key Points:**
  - It would create a nice user experience to restore the scrolling position after navigating back. This is a feature the withInMemoryScrolling function offers.
  - Scroll restoration happens because of emitting a Scroll router event which stores the scroll position. If your list is fetched from a server (usually it is) you might be confused why scrolling restoration doesn't work. It's because the Scroll event is emitted right after the NavigationEnd event and data from a server arrives with some delay. To fix it you can create a service that stores a position to restore and do that job with the right timing.
* **Technical Entities (Classes/Functions/APIs):** `withInMemoryScrolling`, `anchorScrolling`, `scrollPositionRestoration`, `Scroll`, `ViewportScroller`

### View Transition
* **Key Points:**
  - The View Transitions API is another great tool to improve user experience. It allows you to create nice element transitions navigating between pages. Using the withViewTransitions function you don't have to configure the API manually and you can focus only on defining great animations for your transitions.
* **Technical Entities (Classes/Functions/APIs):** `withViewTransitions`, `skipInitialTransition`, `onViewTransitionCreated`, `ViewTransitionInfo`

### Navigation Error Handler
* **Key Points:**
  - To handle navigation errors better use withNavigationErrorHandler feature which provides a function called when navigation error occurs. This function runs inside the injection context so you can safely use your services for handling.
* **Technical Entities (Classes/Functions/APIs):** `withNavigationErrorHandler`, `NavigationError`

### Initial navigation
* **Key Points:**
  - Sometimes you need to configure when the router performs the initial navigation.
  - Default behavior ("enabledNonBlocking") is to start initial navigation after the root component has been created. The bootstrap is not blocked on the completion of the initial navigation.
  - You can block initial navigation ("enabledBlocking"), by calling a withEnabledBlockingInitialNavigation function, until the root component is created. The bootstrap is blocked until the initial navigation is complete. This feature is required for server-side rendering to work to avoid double loading or page flickering.
  - Another option is to block initial navigation ("disabled") by calling withDisabledInitialNavigation which you can use to have more control due to some complex initialization logic.
* **Technical Entities (Classes/Functions/APIs):** `withEnabledBlockingInitialNavigation`, `withDisabledInitialNavigation`

### Location strategy
* **Key Points:**
  - In SPA applications we allow the client (browser) to handle routing instead of doing traditional requests to the server after URL changes to get results from there. To handle routing logic Angular Router can use two location strategies.
  - PathLocationStrategy is the default strategy in Angular. It takes advantage of History API which requires browsers to support HTML 5.
  - Unfortunately it has a big downside: if we reload a page with address e.g. "my-app/users/123/orders" browser makes a request to the server with that address so using the PathLocationStrategy server needs to be able to return main application code (so index.html with app-root tag) for every URL, not just the root one. Moreover we need to tell the browser what should be prefixed to the requested path to generate the URL. It can be done by specifying base href in head section of index.html or by providing a value for the APP_BASE_HREF token.
  - Another option is HashLocationStrategy. To enable it use withHashLocation function. This strategy uses hash fragments so part of the URL prepended with hash (#) character
  - Nowadays you should use HasLocationStrategy only if you need to support older browsers.
* **Technical Entities (Classes/Functions/APIs):** `PathLocationStrategy`, `HashLocationStrategy`, `withHashLocation`, `APP_BASE_HREF`, `base href`

### Debug tracing
* **Key Points:**
  - Calling function withDebugTracing causes logging all router events to the console which might be helpful during debugging.
* **Technical Entities (Classes/Functions/APIs):** `withDebugTracing`

## Utilizing routing in components

### RouterLink
* **Key Points:**
  - Apply routerLink directive to an element in template to make that element a link initiating navigation to a route.
  - Defining route links it's very useful to define relative paths. This can be achieved by using the prefix. So if the first segment starts with: / – router looks up the route from the root; ./ or without prefix – router looks up in the children of current route; ../ – router goes up one level in route tree.
  - Besides URL creation options you can also use other inputs related to navigation behavior: relativeTo (ActivatedRoute), skipLocationChange (boolean), replaceUrl (boolean), state (object), info (unknown)
* **Technical Entities (Classes/Functions/APIs):** `routerLink`, `queryParams`, `queryParamsHandling`, `fragment`, `preserveFragment`, `relativeTo`, `skipLocationChange`, `replaceUrl`, `state`, `info`

### RouterLinkActive
* **Key Points:**
  - For better UX user should be aware which link is associated with the active route. The routerLinkActive directive allows to specify CSS classes applying styles to element when linked route is active.
  - To directly check status of the link you can assign RouterLinkActive instance to template variable.
  - To configure how to determine if the router link is active use routerLinkActiveOptions input which accepts configuration object
* **Technical Entities (Classes/Functions/APIs):** `routerLinkActive`, `RouterLinkActive`, `isActiveChange`, `routerLinkActiveOptions`, `IsActiveMatchOptions`

### RouterOutlet
* **Key Points:**
  - The RouterOutlet directive inserts the component matched from the URL.
  - The RouterOutlet exposes four outputs: activate, deactivate, attach, detach
  - Each outlet can have a unique name defined by name input. The name needs to be a static value. If not set, the default name is "primary".
  - As the users section is displayed by the primary outlet navigation work as usual. But for the orders section RouterLinks need some adjustments. To define a link for navigation through the named outlet branch use a configuration object that has outlets property.
  - Part of the URL related to named outlet is inside the brackets preceded by outlet name.
* **Technical Entities (Classes/Functions/APIs):** `RouterOutlet`, `activate`, `deactivate`, `attach`, `detach`, `name`, `outlet` (route property)

### Router Service
* **Key Points:**
  - To interact with the router in such cases we use Router service.
  - The most useful functionality is of course performing navigation. This can be done using two methods. The first one is the navigate which configuration is very similar to the RouterLink directive. The second method is navigateByUrl which accepts URL and NavigationBehaviorOptions object.
  - The Router service gives us also very handy utility methods to transform URL: serializeUrl transforms UrlTree into string; parseUrl transforms string into UrlTree; createUrlTree creates UrlTree from an array of segments.
  - Each step of this process is represented by the RouterEvent.
  - Router events in order of being triggered: NavigationStart, RouteConfigLoadStart, RouteConfigLoadEnd, RoutesRecognized, GuardsCheckStart, ChildActivationStart, ActivationStart, GuardsCheckEnd, ResolveStart, ResolveEnd, ChildActivationEnd, ActivationEnd, NavigationEnd, Scroll; NavigationCancel, NavigationError
* **Technical Entities (Classes/Functions/APIs):** `Router`, `navigate()`, `navigateByUrl()`, `serializeUrl()`, `parseUrl()`, `createUrlTree()`, `events`, `NavigationStart`, `NavigationEnd`, `NavigationCancel`, `NavigationError`, `navigationTrigger`

### ActivatedRoute Service
* **Key Points:**
  - The ActivatedRoute service is a real treasury of knowledge about a route associated with component loaded in an outlet. It provides all information about URL (url, params and paramMap, queryParams and queryParamsMap, fragment, data) route configuration (title, routeConfig) and its position in router tree (root, parent, firstChild, children, pathFromRoot).
  - As you may know, properties related to the URL are Observables as they can change over time. If you need a static value use snapshot property. The ActivatedRouteSnaptshot class has the same shape and holds the latest values from these Observables.
* **Technical Entities (Classes/Functions/APIs):** `ActivatedRoute`, `paramMap`, `queryParamMap`, `data`, `snapshot`, `ActivatedRouteSnapshot`

## Configuration Injection Tokens

### UrlSerializer
* **Key Points:**
  - Serializing and deserializing the URL can be easily customizable using UrlSerializer.
* **Technical Entities (Classes/Functions/APIs):** `UrlSerializer`, `DefaultUrlSerializer`

### RouteReuseStrategy
* **Key Points:**
  - Navigating between pages means Angular needs to remove the currently displayed component from DOM, destroy its instance and then create a new component to be rendered in DOM. It's an expensive process as it requires JS script to be executed. By default it happens with every navigation, except from navigating to the same route, and may lead to performance issues if component size is large.
  - To face that problem Angular allows to define custom reuse strategy to store a component instead of destroying it and reuse its instance when a route is accessed again. This behavior can be defined by a class implementing RouteReuseStrategy.
  - It defines five methods: shouldDetach, store, shouldAttach, retrieve, shouldReuseRoute
* **Technical Entities (Classes/Functions/APIs):** `RouteReuseStrategy`, `shouldDetach`, `store`, `shouldAttach`, `retrieve`, `shouldReuseRoute`, `DetachedRouteHandle`