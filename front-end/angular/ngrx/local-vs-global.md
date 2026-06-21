---
aliases:
  - Local vs Global Signal Store
Source 1: https://tech.sparkfabrik.com/en/blog/the-lord-of-the-stores/
---
# The Lord Of The Stores: A Quest For Angular State Mastery

## A quest for Angular State Mastery'
* **Key Points:**
  - It began with the forging of the Great Stores. Three were given to Angular developers—structured, scalable, and reactive above all else. The Global Store, vast and powerful. The Component Store, lightweight and precise. And the Signal Store, fresh and efficient.
  - For within these stores was bound the power to manage state, to bring order to applications. Each with its own strengths, each suited for different needs. Because the world of state management is treacherous, and without guidance, chaos can still arise.

## The Global Store – Vilya, The Ring of Air
* **Key Points:**
  - Let's start with the Global Store, the most powerful option, associated with Vilya, the ring given to Elrond, symbolizing wisdom and governance.
  - Likewise, the Global Store manages large state in applications, providing structure and predictability. It is ideal for apps with complex state that multiple components must access and update.
  - However, this power comes with complexity: it requires a solid understanding of reducers, actions, effects, and RxJS, which adds to the learning curve. It demands boilerplate code and many interconnected parts, which can be overwhelming, especially for smaller apps or simpler state needs.
  - But this union of actions and effects is the Global Store's strength, enabling features like time-travel debugging with native Redux DevTools integration. Developers can step backward and forward in the application's state history, making changes easier to trace and debug.
  - Having matured over many years, the Global Store also offers an opinionated and structured way of managing state, helping teams organize logic across files and maintain consistency.
* **Technical Entities (Classes/Functions/APIs):** `Global Store`, `Redux DevTools`

### Global Store – Pros and Cons
* **Key Points:**
  - Pros:
    - Best for large-scale apps with complex state shared across multiple components
    - Easier debugging using time-travel with Redux DevTools
    - Mature: opinionated and structured way of managing state
  - Cons:
    - High complexity (reducers, actions, effects, selectors) + RxJS
    - Steep learning curve
    - Lots of boilerplate code to interconnect everything
    - Too much overhead for smaller applications
  - Example of use cases: large apps with complex, shared business logic (multi-step forms, shopping carts shared across multiple features) or global data like feature flags, notifications, or permissions.

## Nenya – The Ring of Water, The Component Store
* **Key Points:**
  - Then comes the Component Store, the underdog of the state management world, yet full of potential. I associate it with Nenya, Galadriel's ring, symbolizing preservation and stability.
  - The Component Store works seamlessly with the Global Store without conflict, allowing effective state management with minimal changes to your codebase. You can see it like a service managing state with Subjects, but with added clarity and best practices.
  - It's a lightweight solution ideal for managing local component or feature module state, promoting modularity and isolation. It supports multiple independent instances of the same component, maintaining clean separation of concerns.
  - Its API is simpler than that of the Global Store, as it doesn't require actions or reducers, making it easier to learn. However, it lacks developer tool support like time-travel debugging since it doesn't utilize trackable actions.
  - Trade-offs include scalability challenges: state, updaters, selectors, and effects typically reside in the same file, which can become difficult to manage as logic increases. Also, since state is tied to the component lifecycle, it is removed when the component is destroyed, making it inappropriate for persistent or shared state across components.
  - In summary, the Component Store offers store-like benefits without the Global Store's complexity.
  - Now, I must make a small confession: lately the NgRx team has put on the component store page a note saying that they consider the signal store the new default for local state management.
  - But, I nevertheless wanted to talk about component store as a lot of people may be stuck on older versions of Angular and may not be able to easily upgrade to signals and signal store usage.
  - As we can see from npm, still hundreds of thousands of people download Angular to versions less than 17 (signal store was released with NgRx version 17). But these same people may still be in need of a smaller store for their state management.
* **Technical Entities (Classes/Functions/APIs):** `Component Store`, `signal store`

### Component Store – Pros and Cons
* **Key Points:**
  - Pros:
    - Lightweight solution for local component state
    - Multiple independent instances of the same component allowing separation of concerns
    - Simple syntax and state management (no actions or reducers)
  - Cons:
    - Low scalability as everything is in the same file
    - No permanent state, as it is reset on component destruction
    - No Redux DevTools time-travel debugging
  - Example of use cases: managing a form's input values, validation, and submission state, or controlling the visibility and behavior of a modal within a component.

## Narya – The Ring of Fire, The Signal Store
* **Key Points:**
  - And lastly, the Signal Store, represented by Narya, Gandalf's ring of courage and action.
  - It reflects the NgRx team's bold shift toward simplicity and performance. Built on Angular signals, it provides reactive and efficient state management, removing unnecessary recalculations with a straightforward syntax that you can use right away.
  - In this store, state properties are signals, allowing components to access them directly without selectors or boilerplate code. Its lightweight design and optional RxJS support make it ideal for modern Angular apps that want to reduce complexity and bundle size.
  - However, it is still new, so official guidelines are limited. Features like selector composition, route guard integration, and DevTools support are still being developed. The Signal Store also forces you to use signals or convert Observables into signals. This makes it best suited for small to medium-scale features or isolated domains, while large-scale applications may still prefer the Global Store.
  - One downside is that the Signal Store does not support Redux DevTools yet. This means you lose features like time-travel debugging and state inspection.
  - There is an unofficial alternative, the Angular Architects NgRx Toolkit browser extension, which offers partial support but does not fully match the Redux DevTools experience.
* **Technical Entities (Classes/Functions/APIs):** `Signal Store`, `Angular signals`, `Redux DevTools`

### Signal Store – Pros and Cons
* **Key Points:**
  - Pros:
    - State properties as signals (no selectors)
    - Plug and play (simple syntax and small bundle size)
    - Optional RxJS use
    - Good for reactive programming
    - Extreme composability (you choose which features to use)
  - Cons:
    - Still new, with limited official guidelines and documentation
    - Advanced features are still evolving
    - Obligatory use of signals
    - Best suited for small to medium-scale apps; not yet ideal for large-scale or enterprise apps
    - No out-of-the-box Redux DevTools
  - Example of use cases: managing UI-related states like toggles, modals, filters, and tabs where reactive updates are needed but complexity is low.

## Choosing the Right Ring and Power
* **Key Points:**
  - We now have a better understanding of what each store can give us. But we still need to make a choice to understand which one is the right power for us.
  - Like the Elves wielding their rings wisely, developers must choose the right store for the right situation, as we know that the wrong power in the wrong hands can lead to disastrous consequences.
  - We must avoid at all costs the temptation of using too much or too little power just because we like one of the stores. Using a Global Store when local state would be better overcomplicates a simple problem. Conversely, sometimes a simple service+subjects/signals is all you need instead of any store at all.
  - The choice between NgRx stores depends primarily on your project's complexity, team size, and growth expectations. The Global Store excels at enterprise-level applications. The Signal Store offers a simpler alternative for smaller projects. The Component Store provides a balanced approach suitable for medium-sized applications or feature modules.
  - Consider starting with the simplest solution that meets your needs and scaling up as your application grows.

## One Store to Bind All State
* **Key Points:**
  - But what if you want more power, more control, and you cannot choose between the three rings? Well, you also have the choice to combine them to balance their respective flaws and strengths.
  - For example, you can use the Global Store for managing global state across the application, employ the Component Store for encapsulating state within specific features, and integrate the Signal Store where reactive and functional updates are most beneficial.
  - These stores were deliberately made to work well together. The complexity lies not in the implementation but in ensuring your team agrees on conventions and enforces them during reviews. Otherwise, wrongful mixes could lead to painful refactors.
  - Having said that, I made a little task management dashboard on Stackblitz, to have a visual reference on how the code works. This is just a simple app to show how you can intertwine all of the aforementioned stores. Of course using all three is a bit too much, especially for this kind of small project, but this is obviously just an example to show how they can easily work together if the need arises. Usually I would suggest using only two of them together: one for managing global properties and one for managing component properties. This dashboard has:
    - A global quest assigner (managed by the Global Store)
    - A status card specifically for each member (managed by the Component Store)
    - A reactive search input (managed by the Signal Store)

## The Fifth Age: The Age of NgRx State Management
* **Key Points:**
  - With the information we acquired we now know that these three stores bring order to the Middle Earth of state management, but only if used wisely. No one store rules them all — each has its own purpose.
  - Additionally, we now know that you can combine what they can do, to be even more powerful. But it is up to you to avoid the temptation of abusing their power.