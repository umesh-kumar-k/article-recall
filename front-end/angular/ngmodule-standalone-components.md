---
aliases:
  - NgModules vs Standalone Components
highlights: NgModules are legacy grouping/DI boundaries; Standalone is the current default since v17 - modules are now optional compilation/lazy-loading units
Source 1: https://javascript.plainenglish.io/how-standalone-components-changed-my-angular-development-forever-0ac4cd406303
Source 2: https://stackademic.com/blog/ngmodules-vs-standalone-components-or-angular2-vs-angular3
Source 3: https://dev.to/nicholajones075/angular-standalone-components-vs-traditional-modules-an-ultimate-guide-1923
---
# NgModules vs. Standalone Components in Angular

## What are NgModules?

* **Key Points:**
  - In Angular, a `@NgModule` is a decorator that represents a cohesive block of code that focuses on a specific functionality or a specific feature of the application. It acts as a container for components, directives, services, and other related entities, providing a clear organization for the application's codebase. Modules can be lazy loaded when they don't need to be readily available on startup, which improves the speed of the application.
  - List of metadata for @NgModule:
    - Declarations: In this array, you can add components, directives, and pipes.
    - Imports: other modules whose exported classes and services are needed in this module
    - Exports: The subset of declarations that should be reusable in the component templates of other Ngmodules.
    - Providers: creators of services that this ngModules contributes to the global collection of services.
    - Bootstrap: the main application view (entry point), called a root component, hosts all other application views. (Note):- only the root ngModule should set the bootstrap property.
  - Here, you can notice that ngModules manages a substantial amount of elements, which consequently results in an expansion of the application's size and a corresponding reduction in performance.
  - The confusion starts with components and services not having the same scope(visibility):
    - declarations: components are in local scope (private),
    - providers: services**** are generally in global scope (public).
  - It means the components you declared are only usable in the current module. If you need to use them outside, in other modules, you'll have to export them. So here the CoreModules and SharedModules are useful for your app.
* **Technical Entities (Classes/Functions/APIs):** `@NgModule`, `declarations`, `imports`, `exports`, `providers`, `bootstrap`, `CoreModules`, `SharedModules`
* **Code Snippet:**
```typescript
import { NgModule } from '@angular/core'
import { CommonModule } from '@angular/common'

import { HomeComponent } from './home.component'
import { HostDirective } from './host.directive'
import { customPipe } from './custom.pipe'
import { AppService } from './shared/some.service'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'

@NgModule({
  declarations: [HomeComponent, HostDirective, customPipe],
  imports: [AppRoutingModule, CommonModule],
  providers: [AppService],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

## What is a Standalone Component? 🌌
* **Key Points:**
  - Standalone components provide a simplified way to build Angular applications. Standalone components, directives, and pipes aim to streamline the authoring experience by reducing the need for NgModules. Existing applications can optionally and incrementally adopt the new standalone style without any breaking changes.
  - Angular 14 introduces the standalone component, a component not part of any ngModule that can be used with either other standalone or module-based components.
  - Besides standalone components, in Angular 14, you can also create:
    - Standalone directives and pipes
  - You can use a standalone component with:
    - Module-based components
    - Other standalone components
    - Lazy loading and lazy loading routing
  - Transform a module-based component into a standalone component by adhering to the following steps:
    - Set the standalone property to true.
    - Remove it from the declarationarray of the module of which it was a part.
    - Use importsarray to add dependencies.
  - The beauty of standalone components lies in their modular and self-contained nature. Each standalone component is developed independently, with its own set of dependencies and functionality. This isolation allows for better code organization and maintainability, as each component only has access to the dependencies it explicitly requires.
  - Here are some of the key differences between NgModules and standalone components 🎌:
    - NgModules are used to organize strand manage dependencies📦.
    - Standalone components are more flexible than NgModules 💪.
    - Standalone components are more lightweight than NgModules 🎈.
  - Overall, standalone components are a more flexible and lightweight way to build Angular applications. They can be used in any part of an application without being declared in a NgModule, and they are smaller and faster ️to load than NgModules.
* **Technical Entities (Classes/Functions/APIs):** `standalone`, `imports`
* **Code Snippet:**
```typescript
import { Component } from '@angular/core'
import { CommonModule } from '@angular/common'

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css'],
})
export class LoginComponent {
  constructor() {}
  ngOnInit(): void {}
}
```

## Codebase Structure 📝
* **Key Points:**
  - With NgModules: (standard component definition without standalone)
  - With Standalone Components: set standalone:true and imports:[NgIf]
* **Technical Entities (Classes/Functions/APIs):** `standalone`, `imports`, `NgIf`
* **Code Snippet:**
```typescript
// With Standalone Components
import { Component } from '@angular/core';
import { NgIf } from '@angular/common';
@Component({
  selector: 'my-app',
  standalone:true,
  imports:[NgIf]
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {
  name = 'Angular app';
}
```

## Bootstrapping application 🚀
* **Key Points:**
  - Using @ngModules, we can bootstrap the application by bootstrapping the AppModule in main.ts and adding AppRoutingModule in AppModule to configure the routing.
  - Using a Standalone approach we can directly bootstrap the AppComponent in main.ts file. and for adding other routes we can simply provide routes with the help of ProvideRoutes().🚀
  - This highlights how the independent API significantly decreases both the bundle size and the amount of code.
* **Technical Entities (Classes/Functions/APIs):** `bootstrapApplication`, `provideRouter`, `provideAnimations`, `AppRoutingModule`, `RouterModule.forRoot()`
* **Code Snippet:**
```typescript
// main.ts (Standalone)
import { bootstrapApplication } from '@angular/platform-browser'
import { AppComponent } from '@app/app.component'
import { provideRouter } from '@angular/router'
import { provideAnimations } from '@angular/platform-browser/animations'
import { appRoutes } from '@routes/app.routes'

bootstrapApplication(AppComponent, {
  providers: [provideRouter(appRoutes), provideAnimations()],
}).catch((err) => console.log(err))
```

## Guards and Interceptors 🛡️
* **Key Points:**
  - Using @NgModules the guards are class-based architecture. and it requires more boilerplate code.
  - Functional guards are introduced in version 14.2. They are a more modern and flexible way to implement guards in Angular.
  - One of the key benefits of functional guards is that they are easy to compose. This means that you can combine multiple functional guards to create more complex guards.
  - 💡How we can miss that the CanMatch guard a best alternative of CanActivate and CanLoad guard.
  - Using class-based interceptors should implement the HttpInterceptor interface. it requires you to implement intercept() method. After that, you need to register your custom interceptor in your app's module.
  - Functional interceptors provide a more functional and declarative approach to modifying or inspecting requests and responses compared to class-based interceptors. It uses higher-order functions and can be particularly useful when you want to perform specific tasks on the request or response in a concise and flexible manner.
* **Technical Entities (Classes/Functions/APIs):** `CanActivate`, `CanActivateFn`, `CanMatch`, `CanLoad`, `HttpInterceptor`, `HttpInterceptorFn`, `inject()`
* **Code Snippet:**
```typescript
// Functional Guard
export const AuthGuard: CanActivateFn = () => {
  const encryptDecryptService = inject(EncryptDecryptService)
  const router = inject(Router)
  const token = encryptDecryptService.getDecryptedLocalStorage(Constants.storageKeys.currentUser)
  if (token) {
    return true
  } else {
    return router.parseUrl('/auth/login')
  }
}
```

## Server-Side Rendering (SSR) Evolution
* **Key Points:**
  - Angular has supported SSR since Angular Universal, which was introduced in Angular 4. However, SSR in Angular 16 has been significantly improved with the introduction of client hydration.
  - Here are several benefits to using SSR in Angular, including:
    - Improved performance: SSR can improve the performance of Angular applications by sending the pre-rendered HTML to the client, which can be loaded and displayed more quickly.
    - Improved SEO: SSR can improve the SEO of Angular applications by making it easier for search engines to index the content of the application.
    - Improved user experience: SSR can improve the user experience of Angular applications by displaying the initial content of the application more quickly.
  - In Angular version 13 AppServerModule is bootstrapped in the server configuration file.
  - Until Angular version 15, Server-Side Rendering (SSR) was imported through the appModule. However, in version 16, a new approach has been introduced. You can enable server-side rendering using provideServerRendering() method.
  - In version 16, Angular introduced Client hydration. It's a process where the Angular application is re-rendered on the client using the same DOM that was rendered on the server. This allows Angular applications to take advantage of the performance benefits of SSR while still maintaining the interactivity of a client-rendered application.
  - The purpose of hydration is to improve application performance by reusing existing DOM elements, avoiding extra work to recreate them, and preventing visible UI flickers.
  - You can enable client hydration in your app by incorporating the provideClientHydration() method into the app configuration file.
* **Technical Entities (Classes/Functions/APIs):** `provideServerRendering()`, `provideClientHydration()`, `mergeApplicationConfig()`, `ngExpressEngine`
* **Code Snippet:**
```typescript
// app.config.ts (Angular 16+)
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(appRoutes),
    provideAnimations(),
    provideHttpClient(),
    provideClientHydration(),
  ],
}
```

## Conclusion
* **Key Points:**
  - In conclusion, the new updated Angular offers a wide range of new features and enhancements that can help to build powerful, efficient, and user-friendly web applications.
  - Thank you! Happy coding 👨‍💻

--- 


# Stop Fighting the Module: Why Standalone Components Are Your Only Real Path Forward

## The "NgModule Tax" and Why It Kills Velocity
* **Key Points:**
  - Angular development used to be defined by a ritual: you build a component, you register it in a module, you export it, and you pray your dependency graph doesn’t collapse under its own weight. For years, we accepted this ceremony as the “enterprise tax.” But today, clinging to NgModules isn't a sign of architectural maturity—it's a massive drag on your team’s velocity.
  - Standalone components aren’t just a syntax change; they are the end of the ‘NgModule tax’ that has been burying our intent under layers of boilerplate for a decade.
  - If you are still architecting new features around NgModule boundaries, you aren't just adding complexity; you’re actively sabotaging your project's ability to evolve.
  - We spent years building “module hierarchies” that existed only to keep the Angular compiler happy. We weren’t architecting for the user; we were architecting for the framework.
  - This approach led to a very specific set of symptoms that every senior dev has felt in a legacy repo:
    - The “Module Soup” Problem: You need a button, but it’s trapped inside a SharedModule. To get it, you have to import the entire module, dragging along fifty other components you don’t need, bloating your bundle.
    - The Dependency Fog: Looking at a component file tells you nothing about its dependencies. You have to jump to a sibling file, check the declarations array, verify the imports, and hope you didn't miss an exports declaration.
    - Onboarding Friction: Explaining the difference between imports, declarations, and exports to a new hire is a 30-minute meeting that has zero to do with the actual business logic of the app.
* **Technical Entities (Classes/Functions/APIs):** `NgModules`, `SharedModule`, `imports`, `declarations`, `exports`

## The Trap: Chasing “Clean” Architecture at the Cost of Reality
* **Key Points:**
  - The real trap isn’t that NgModules are inherently evil—it’s that they encourage a "top-down" mindset that is fundamentally at odds with modern component-driven development.
  - We treat our application like a nested set of boxes (modules), when it should be treated like a graph of capabilities (components). When you force everything into a module, you lose the ability to see the codebase for what it actually is. You stop asking, “What does this component need?” and start asking, “Which module does this belong to?”
  - Architecture diagrams don’t reduce complexity; they just document the mess you’ve already created.
  - By relying on modules, we’ve created a system where the “wiring” of the app is more complex than the app itself.

## Standalone: The End of Accidental Complexity
* **Key Points:**
  - Standalone components shift the responsibility of dependency management to the only place it belongs: the component itself. This is not just a stylistic preference; it is a profound win for maintainability.
  - When a component explicitly lists its imports, the imports array acts as a living document of its requirements. There is no guesswork. There is no SharedModule bloat.
  - Explicit Dependency Tracking: If I see a component, I know exactly what it depends on. No more digging through three layers of files to find out why a pipe isn’t working.
  - True Lazy Loading: With standalone components, lazy loading isn’t an “event” that requires its own module file. It’s a standard, readable route definition.
  - Improved Tree-Shaking: Because dependencies are atomic, the bundler can see exactly what isn’t being used. The days of “accidentally” shipping 200KB of unused UI components because they were imported in a SharedModule are over.

## How We’re Recreating Spaghetti (Without Even Trying)
* **Key Points:**
  - Even with standalone components, developers are finding new ways to over-engineer. The temptation to create “Mini-Modules” using just folders is high. We see teams creating huge barrel files (index.ts) just to emulate the old export behavior of modules.
  - Stop it.
  - Standalone components are designed to be consumed directly. You don’t need a middleman. You don’t need a “UI-Kit” barrel file that imports 40 components just to re-export them. That is just NgModule by another name.
  - The Symptom: You have a ui/ directory with an index.ts that is 500 lines long.
  - The Fix: If a component is needed, import it directly from its path. If that feels “messy,” you aren’t fighting clutter — you are fighting the reality that your app has dependencies. Embrace the explicitness.
* **Technical Entities (Classes/Functions/APIs):** `index.ts`, `NgModule`

## A Simpler Way to Think About This
* **Key Points:**
  - You don’t need a 40-page architectural document to move to standalone. You need a few simple rules that keep your team from spiraling into “module-less” chaos.
  - Default to Standalone: If you are writing a new component, it should be standalone. If it’s not, you better have a very good reason.
  - Explicit Imports Only: Do not create “giant barrel files” to hide imports. Let the imports array be your source of truth.
  - Bottom-Up Migration: Migrate your “leaf” components (the ones that don’t depend on anything else) first. This builds confidence and flattens the dependency tree naturally.
  - Use the Schematic: Use ng generate @angular/core:standalone. Don't do this by hand. Let the tooling do the heavy lifting—it’s faster and safer.
  - Stop “Frameworkizing” Folders: Don’t create fake “module-like” structures just to feel organized. A folder of components is just a folder. Trust the code, not the folder structure.
* **Technical Entities (Classes/Functions/APIs):** `ng generate @angular/core:standalone`

## The Real Takeaway for Senior Developers
* **Key Points:**
  - Standalone components are the best thing to happen to Angular because they align the framework with how we actually build modern software. They allow us to focus on the unit of value — the component — rather than the unit of infrastructure — the module.
  - If you are still holding onto modules because they provide “structure,” ask yourself: is it structure, or is it a security blanket? In an era where bundle size and startup speed are the primary metrics of quality, we cannot afford to carry the weight of unnecessary abstraction.
  - Refactor incrementally. Ship the boring, consistent pattern. Stop architecting for the framework and start architecting for the browser.
- 