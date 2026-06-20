---
aliases:
  - Dependency Injection at Architecture Scale
highlights: InjectionToken<T> over concrete class injection for abstraction; enables swapping implementations (mock, real, feature-flagged) without touching consumers
Source 1: https://angular.dev/guide/di/lightweight-injection-tokens
Source 2: https://angular.love/why-is-inject-better-than-constructor
Source 3: https://angular.love/make-the-most-of-angular-di-private-providers-concept
---
# Why is inject() better than constructor?

## Why is inject() better than constructor?

## Summary: Dependency injection (DI)
* **Key Points:**
  - Dependency injection, also referred to as DI, is a widely used design pattern that implements Inversion of Control principle. Angular has a powerful built-in DI mechanism that allows creation and delivery of application parts to other parts of the application that need them. With the help of this mechanism you can use dependencies in your application in a flexible way.
  - In the DI system there is a dependency consumer and dependency provider.
  - Dependency providers can be of various types, most commonly being the class with @Injectable decorator with providedIn set.
  - Setting 'root' for providedIn makes the class a singleton and registers it in the application's root injector.
  - In practice, the provided dependencies are usually services and custom injectables.
  - There's an abstraction called Injector which checks its registry if an instance already exists upon request of a dependency, and if not creates and registers a new instance.
  - The injectors are created during the application bootstrap process so there's no need to create them manually.
* **Technical Entities (Classes/Functions/APIs):** `@Injectable`, `providedIn: 'root'`, `providers`, `Injector`

## Injection context
* **Key Points:**
  - DI relies on a runtime context called injection context. When dependency consumers want to inject services, directives, pipes, other custom injectables etc. they can only do it within the injection context. The rules of DI state that using any injection form has to be within the injection context for the program to work, otherwise an error will occur.
  - The situations in which injection context is available are as follows:
    - Construction block of a class with @Injectable or @Component including constructor() and the initializer fields of these classes
    - Factory function specified for useFactory or a Provider or an @Injectable
    - factory function specified for an InjectionToken
    - Inside a stack frame that runs in an injection context
  - Injecting a service outside the inject context will result in Angular throwing NG0203.
  - The above example injects the userService inside OnInit lifecycle hook with the purpose of initializing. It won't break the code, but the logic fails because injection works during the construction phase which is in the injection context and not after the construction phase is complete.
* **Technical Entities (Classes/Functions/APIs):** `injection context`, `NG0203`, `inject()`, `OnInit`

## How does inject() compare to constructor?
* **Key Points:**
  - DI via constructor is the traditional method for injecting dependencies and is still supported by the latest Angular versions to this date. With Angular 14 the inject() function was introduced which injects a token from the active Injector. Just like other methods of DI it's supported only within the injection context. Technically constructor can't fall out of injection context as a pose to inject(), however, compared to constructor, inject() offers a more powerful and often simpler way to handle dependencies, along with extended possibilities. Nowadays it's a more popular method than constructor.

### Injecting into standalone functions
* **Key Points:**
  - Standalone functions refer to external functions which you can use anywhere inside a class.
  - When using constructor as an injection method, you're only tied to the construction phase of the class, as functions don't have constructors. Which brings me to say that injection inside standalone functions is possible. If you call a function with injects a service, whether it's in the injection context or not depends on where you call the function.
  - As you can see, the getUsers() function serves a role in setting up the variables in the UserService class' construction phase. Thus, calling inject(HttpClient) is also done within the injection context. However, this method ties you to using inject(), as constructor based injection is not possible. On top of being able to call inject() in various places inside the class construction block it also grants the possibility to handle some construction tasks from outside the class via exported functions.

### Inheritance
* **Key Points:**
  - Another case where inject() function simplifies DI is when dealing with inheritance. Let's take an example of a BaseComponent class, and a ChildComponent class that extends BaseComponent.
  - ChildComponent class injects AnalyticsService class for itself. But notice that its constructor also takes the services which BaseComponent injects to fetch them by calling super(), in the correct order in the base's constructor. This method looks painful to implement for even bigger projects.
  - BaseComponent class constructor doesn't take any arguments. At this point, constructor() block can even be omitted for cleaner code.
  - ChildComponent injects what it needs. It already extends the BaseComponent class so constructor-wise you're left with much less work. You don't have to worry about the order of properties, as calling an empty super() will suffice.

### Conditional injection
* **Key Points:**
  - Conditional injection is another case in which we're tied to the inject() function as we get to inject a dependency based on a condition, as the name suggests. You can't call constructor() inside an if block, so instead, within the constructor you check a condition and call the inject() function, which is legal within the injection context.
  - In the above example we have an AnimationService that runs on a browser environment only. First it injects the PLATFORM_ID which is a special token that tells you on which platform your app is running. You define the AnimationService instance as null, then check the condition inside the constructor and then inject it. With this approach you can avoid runtime errors on the server side and improve performance by not creating unnecessary services.
* **Technical Entities (Classes/Functions/APIs):** `PLATFORM_ID`, `isPlatformBrowser`, `AnimationService`

## Other ways of using inject()
* **Key Points:**
  - So far you have learned the concept of DI, the rules of the injection context and the difference between the inject() function and constructor-based injection. You have learned where to use injection and where not to and how strict the rules of injection context are.
  - However, despite being a more advanced topic, I'd like to introduce the other ways available in Angular which allow you to use inject() where you think is not possible. This includes some helper functions.
  - runInInjectionContext: This helper function is a way of calling inject() within an injection context while not actually being in one (e.g methods, lifecycle hooks). Using this method you also have to have access to the current injector.
  - injector.runInContext(): This method is used the same way as previously where runInInjectionContext is a standalone function and runInContext is a method of EnvironmentInjector.
* **Technical Entities (Classes/Functions/APIs):** `runInInjectionContext`, `injector.runInContext()`, `EnvironmentInjector`

## Summary
* **Key Points:**
  - Conventionally, dependency injection (DI) methods are strictly tied to injection context, meaning, they can't be used outside of injection context. In this article we looked at the differences between constructor and function-based injection. Construction-based injection is the traditional method and can't be called anywhere that will fall outside the injection context, thus we only have the inject() function in our concerns. You have learned where to use inject() function and where not to, and the ways of using it outside of injection context but making it fall inside. You have also learned how much inject() simplifies inheritance, enables conditional injection, and makes injection possible inside standalone functions. Function-based injection is the most popular way of DI to this date and that cannot be argued given these advantages and flexibility it brings to both Angular apps and developer experience.


---

# Make the most of Angular DI: private providers concept

## Typical DI pattern in Angular
* **Key Points:**
  - I review Angular code every day during my work and opensource projects. The uses of DI in most apps is limited by the following cases:
    - Get some Angular entities like ChangeDetectorRef, ElementRef and other from DI.
    - Get a service to use it in a component.
    - Get a global config via a token that is declared in the root of the app. For example, make an injection token API_URL in the app.module and get this URL in any entity in the app when you need it.
  - Sometimes developers transform already existing global token in a more convenient format. A good sample of such a transformation is a WINDOW token from a package @ng-web-apis/common.
  - Angular has a DOCUMENT token to get a page object in any place of your app. That way your components do not depend on global objects. It is easy to test them and nothing gets broken by Server-Side Rendering.
  - Now I want to propose another way to make such transformations to move them into providers field of component or directive that injects the result.
* **Technical Entities (Classes/Functions/APIs):** `ChangeDetectorRef`, `ElementRef`, `API_URL`, `WINDOW`, `@ng-web-apis/common`, `DOCUMENT`

## Private providers
* **Key Points:**
  - In my team, we use DI often and notice that sometimes we need to transform data coming from DI before using it. In other words, our component depends on one type of data, but we inject another one and do some transformations inside it.
  - So we have:
    - A component that shows information about some entity called "organization"
    - Query-param of the route with the ID of the organization to work with
    - A service that gets ID and returns an Observable with information about an organization
  - What we want to do:
    - We want to take ID from query-params, call a service method with it, and get a stream with organization information in return. This information is showed in a component.
* **Technical Entities (Classes/Functions/APIs):** `ActivatedRoute`, `OrganizationService`, `Observable`

### 1. How you should not do it
* **Key Points:**
  - Sometimes I see the following work with data in a component. Please, do not do this.
  - This code works but it has some problems:
    - The 'organization' field is not defined at component creation. That is why there is a time when we can get 'undefined'. If we have non-strict TypeScript, we break typings. Or we can write the right type: organization?: Organization and now we need to add several checks.
    - This code is harder to support. For example, next time we need one more param and we add one more subscription into ngOnInit. It will harder to read and understand each time because of implicit variables and unclear data flow.
    - We can meet some problems with change detection and updating our component using OnPush strategy.

### 2. Let's do it well
* **Key Points:**
  - Erin made it well in her talk. Her sample from presentation looks like this.
  - This code works well and has no disadvantages of the previous approach: it looks rather neat and we have no unnecessary fields. If we want to extend the component with a similar stream, we just add another one — it is easy to do. We do not need to touch the code of the previous one to add a new stream.
  - In addition, the data stream is cleaner: we only have a stream that is created in a moment the component class is created. When it emits data, the information in our template will be shown.
* **Technical Entities (Classes/Functions/APIs):** `Observable`, `ActivatedRoute`, `OrganizationService`

### 3. Let's try to do it cooler with private providers
* **Key Points:**
  - Let's take a closer look at the previous solution.
  - In fact, the component does not depend on the router and even on OrganizationService. It depends on organization$. But there is no such thing in our dependency injection tree, so we have to transform data in the component.
  - But what if you transform the data before it enters the component? Let's write the Provider for the component in which we place all transformations.
  - For convenience, we can put the providers into a separate file next to the component.
  - So we have _organization.providers.ts_ file with Provider that transforms data and an injection token to get it in the component.
  - A note about DI: deps allow us to get some entities from DI tree and send them as arguments into a token factory. This way you can get any entity from DI. You can even use DI decorators.
  - So we need to set our providers for the component.
  - And now we are ready to get it in the component.
  - The whole class is just a single line of code with data injecting.
  - What does this approach give us?
    - Clear dependencies: a component does not inject any data that it does not need. It works only with entities that it needs to show data in a template.
    - It is testable: we can easily test a provider because its factory is just a function. It's also easier for us to test a component: in the tests, we don't have to build a dependency tree and replace many entities — we just pass ORGANIZATION_INFO token with stub data.
    - It scales: do you want your component to work with another type of data? No problem. We just change one line of code. If you need to edit a transformation logic, change a factory. If you need some new data, add another token because you can have an unlimited amount of tokens in your providers array.
  - After we started to use this approach, many of our components and directive look cleaner and simpler. Separating the logic of data transformation and data showing makes it easy to modify logic or expand functionality. It is also easier to catch bugs because you can define a problem area: the problem can be in data transformations or in its showing to a user.
* **Technical Entities (Classes/Functions/APIs):** `ORGANIZATION_INFO`, `ORGANIZATION_PROVIDERS`, `organizationFactory`, `InjectionToken`, `Provider`, `deps`, `useFactory`, `ActivatedRoute`, `OrganizationService`
* **Code Snippet:**
```typescript
// token to access a stream with the information you need
export const ORGANIZATION_INFO = new InjectionToken<Observable<Organization>>(
    'A stream with current organization information'
);

export const ORGANIZATION_PROVIDERS: Provider[] = [
    {
        provide: ORGANIZATION_INFO,
        deps: [ActivatedRoute, OrganizationService],
        useFactory: organizationFactory,
    },
];

export function organizationFactory(
    { params }: ActivatedRoute,
    organizationService: OrganizationService
): Observable<Organization> {
    return params.pipe(
        switchMap((params) => {
            const id = params.get('orgId');

            return organizationService.getOrganizationById$(id);
        })
    );
}
```

## In conclusion
* **Key Points:**
  - The described approach cannot fix all your design issues. You shouldn't add providers to any small case: sometimes the code is clearer if you transform data in a class method or use Angular pipes.
  - Nevertheless, I hope that private providers can help you simplify your components with a lot of dependencies or give you an alternative when gradually refactoring large pieces of logic.