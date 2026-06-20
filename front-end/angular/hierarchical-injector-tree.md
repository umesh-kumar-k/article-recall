---
aliases:
  - Hierarchical Injector Tree
highlights: Root -> Platform -> Environment -> Element Injectors; Component level providers create scoped service instances per subtree
Source 1: https://www.angularspace.com/hidden-parts-of-angular-view-providers/
Source 2: https://angular.love/cracking-angular-di-the-hidden-layers-of-injectors
---
# Cracking Angular DI: The Hidden Layers of Injectors

## Injector 101
* **Key Points:**
  - It's crucial to understand how injectors provide services to fully grasp how DI works in Angular. Here's a simplified algorithm:
    - Angular uses the current injector.
    - Injector.get(token) is called with a service's token.
    - If the injector finds the token, it either creates and returns a new instance of the service or provides an existing instance.
  - Angular gathers all services from the providers in modules, components, and classes marked with the @Injectable({providedIn: …}) decorator. The injector then creates a Map, which allows us to retrieve instances of these services.
  - However, Angular doesn't rely on one injector — it uses an entire hierarchy of injectors. This tree of injectors is created during the application's bootstrap process. If the current injector doesn't contain the required token, Angular attempts to find it in the parent injector.
  - When Angular creates an injector, it saves a reference to the parent injector. If the current injector doesn't contain the token, Angular searches for it in the parent injector. But what happens if the token isn't found in any of the injectors? In that case, Angular uses a special NullInjector.
  - Angular provides NullInjector as the root of the injector tree. If a token isn't found in the injector chain, the NullInjector throws the familiar error: NullInjectorError: No provider for MyService!.
* **Technical Entities (Classes/Functions/APIs):** `Injector`, `Injector.get()`, `@Injectable({providedIn: …})`, `NullInjector`, `NullInjectorError`
* **Code Snippet:**
```typescript
type InjectionToken = string; // unique identifier

abstract class Injector {
  abstract get(token: InjectionToken): any; // function for getting a service
}

type Record = {
  factory: () => any; // factory for creating a service
  value: any; // existed service
}

export class ModuleInjector extends Injector {
  private records: Map<InjectionToken, Record>; // responsible for storing services
  
  constructor(providers: Array<[InjectionToken, Record]>) {
      this.records = new Map(providers);
  }
 
  get(token: InjectionToken): any {
      if (!this.records.has(token)) { // if token is not found then throw an error
          throw new Error(`Could not find the ${token}`);
      }
      
      const record = this.records.get(token);
      
      if (!record.value) { // if an instance is not created then just create it
          record.value = record.factory();
      }
      
      return record.value;
  }
}
```

## Hierarchy of injectors
* **Key Points:**
  - Three injectors are created during the application's bootstrap process:
    - NullInjector: Throws an error if the token is not found.
    - Platform Injector: Responsible for services that can be shared across multiple apps within the same Angular project.
    - Root Injector: Stores Angular's default services as well as services provided using the @Injectable({ providedIn: 'root' }) decorator or defined in the providers property of the AppModule metadata.
  - When we try to get a service, Angular first searches the root injector, followed by the platform injector. If the service is still not found, the NullInjector throws an error.
  - There's a tricky aspect to consider: if a module with providers is imported into the AppModule, a separate injector won't be created. Instead, its services will be added to the root injector.
  - This wasn't an obvious point for me when I was learning Angular — I had assumed that every module would create its own injector.
* **Technical Entities (Classes/Functions/APIs):** `NullInjector`, `Platform Injector`, `Root Injector`, `@Injectable({ providedIn: 'root' })`, `AppModule`

### Lazy-loaded module injector
* **Key Points:**
  - In addition to the three default injectors, Angular also creates a separate injector for each lazy-loaded module.
  - When you provide a service in the providers property of a lazy-loaded module, the module's injector will create an instance of the service for components that belong to that module. The rest of the application won't be aware of this service.
  - If the service is injected into a component within a lazy-loaded module, the search will begin with the injector of that module. If the service isn't found, the search continues through the standard chain: Root Injector → Platform Injector → NullInjector.

### Node Injector
* **Key Points:**
  - We've covered module injectors, but Angular also uses another type of injector: the node (element) injector.
    - The root component always creates its own node injector.
    - Node injectors are created for each HTML tag that serves as a component selector or acts as a host for directives.
  - To add a service to a component's node injector, we need to provide it in the providers or viewProviders property of the component's metadata.
  - The main difference between a node injector and a module injector is that the node injector is created and destroyed along with the component. As a result, services that belong to the node injector are created and destroyed the same way.
  - Node injectors have their own hierarchy, similar to module injectors. The search for a service starts with the current node injector and continues up through parent components until the service is found or the root component is reached.
  - When a service is injected into a component, Angular first looks for it in the current node injector. If it's not found, the search continues through parent node injectors, followed by the lazy-loaded module injectors, and finally through the standard chain: Root Injector → Platform Injector → NullInjector. The last one throws an error if none of the injectors contain the service's token.
* **Technical Entities (Classes/Functions/APIs):** `node injector`, `element injector`, `providers`, `viewProviders`, `Angular DevTools`

## Standalone revolution
* **Key Points:**
  - With the introduction of standalone components in Angular 14, the hierarchy of injectors has changed. Angular now uses the EnvironmentInjector instead of module injectors. In a fully standalone app, there are no modules, so the Angular team chose more consistent naming. However, the search chain for default injectors remains the same: Root Environment Injector → Platform Environment Injector → NullInjector.
  - If you want to provide a service to the root injector, you have two options: use the @Injectable({ providedIn: 'root' }) decorator or use ApplicationConfig, which replaces the AppModule providers.
* **Technical Entities (Classes/Functions/APIs):** `EnvironmentInjector`, `Root Environment Injector`, `Platform Environment Injector`, `ApplicationConfig`, `bootstrapApplication`

### Route Environment Injector
* **Key Points:**
  - In Angular 14, a new loadComponent function was introduced, which handles lazy-loading of components, similar to how loadChildren works for modules. You might assume that loadComponent could be used to create a separate injector, but this assumption is incorrect. In reality, loadComponent does not create a new injector.
  - If you need to create an injector for a specific route, similar to a lazy-loaded module injector, you can use the providers property in the route configuration. This functionality creates a separate injector for that route and its child routes, regardless of whether the route is lazy-loaded or not.
* **Technical Entities (Classes/Functions/APIs):** `loadComponent`, `loadChildren`, `providers` (route configuration)

### Backward compatibility with NgModule
* **Key Points:**
  - The Angular team recognizes that migrating to standalone components takes time, so they've added backward compatibility with the NgModule approach. You can import either a component or a module inside a standalone component.
  - This feature makes it easier to migrate to the new component type, but it can also lead to unexpected behavior. Issues happen when a module with services is imported into a standalone component. Since Angular treats standalone components as independent building blocks of the application, it would be odd if these services were added to the root injector. Components must encapsulate this logic.
  - To address this issue, Angular creates a separate injector-wrapper for standalone components if they import any modules, regardless of whether the modules have providers or not. This injector-wrapper collects all the services from the imported modules and stores them within itself.
  - The injector-wrapper is created in three cases:
    - During the application's bootstrap process: If the root component has modules or uses a component with modules inside.
    - For dynamically created components: If the component has modules or uses a component with modules inside.
    - For routed components: If the component has modules or uses a component with modules inside.
* **Technical Entities (Classes/Functions/APIs):** `injector-wrapper`, `imports` (standalone component), `NgModule`