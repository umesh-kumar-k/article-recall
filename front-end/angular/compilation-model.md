---
aliases:
  - Compilation Model
highlights: Ivy compiles components to incremental DOM instructions; no virtual DOM diffing overhead; tree shakes unused framework code per component
Source 1: https://blog.nrwl.io/understanding-angular-ivy-incremental-dom-and-virtual-dom-243be844bf36
Source 2: https://www.angularminds.com/blog/virtual-dom-vs-incremental-dom-in-angular
Source 3: https://medium.com/google-developers/introducing-incremental-dom-e98f79ce2c5f
Source 4: https://medium.com/angular-in-depth/ivy-engine-in-angular-first-in-depth-look-at-compilation-runtime-and-change-detection-876751edd9fd
Source 5: https://blog.angular.dev/ivys-internal-data-structures-f410509c7480
---
# What is Incremental DOM in Angular?

* **Key Points:**
  - Before Angular version 4, the framework used traditional DOM to update and diffing approach But after Angular version 4, the framework started working on incremental DOM. One of the most powerful advantages of incremental DOM is it is more memory-efficient.
  - Whenever the state of the application changes, the incremental DOM approach changes the actual DOM tree directly, it does not create the new DOM tree representation for changes like the virtual DOM, Incremental DOM gets instructions to update the DOM tree and these instructions are executed directly on the actual DOM.
  - While Angular does not use Incremental DOM by default, principles of incremental updates can still be applied within Angular applications through strategies like OnPush change detection, optimizing *ngFor with trackBy, and minimizing unnecessary DOM updates.
  - One of the main advantages of Incremental dom is it is designed to be tree-shakable. The term tree-shaking is commonly used in JavaScript to describe the process of eliminating dead code from the final bundle during the build process.
  - This particularly helps in optimizing application performance by reducing the size of the JavaScript files that need to be downloaded and executed. Incremental DOM is built with a modular architecture, meaning its functions can be imported individually. This modularity allows modern JavaScript bundlers (like Webpack or Rollup) to analyze and remove unused code.
* **Technical Entities (Classes/Functions/APIs):** `Incremental DOM`, `OnPush`, `*ngFor`, `trackBy`, `tree-shaking`, `Webpack`, `Rollup`
* **Key Points (Advantages of using Incremental DOM):**
  - Efficient Memory Management: The Incremental DOM framework does not use separate memory to represent the component. It modifies the original DOM only so it minimizes memory usage and helps for memory management.
  - Selective Updates: Incremental DOM updates only that part of DOM that has changed, It does not re-render the Whole DOM tree. This can significantly reduce the amount of work the browser or rendering engine has to do, leading to better performance.
  - Faster Rendering: Unlike the Virtual DOM, which creates a full representation of the DOM in memory and then applies changes in a batch, Incremental DOM directly updates the real DOM as changes occur and does not create a new DOM tree in memory. This method can lead to faster rendering.

## What is a Virtual DOM in Angular?
* **Key Points:**
  - The Virtual DOM is a lightweight copy of the actual DOM. DOM tree represents the elements of a webpage, these elements include headers, headings, videos, and images. For large and complex applications with frequent updates, directly manipulating the DOM creates performance issues.
  - The virtual DOM concept solves this issue by creating another copy of the original virtual DOM object in memory. Instead of updating the real DOM directly, changes are first applied to the Virtual DOM. Then, the new virtual DOM tree is compared with the previous Virtual DOM, and whatever changes are made only that part of the elements are changed in the real DOM tree and updated elements will get rendered on the web page again.
* **Technical Entities (Classes/Functions/APIs):** `Virtual DOM`, `DOM`
* **Key Points (Advantages of using the Virtual DOM):**
  - Improved Performance: The real DOM is slow to manipulate directly. By reducing the number of direct interactions with the real and new entire virtual DOM, the Virtual DOM significantly improves the performance of your application. The selective updates ensure that your UI remains responsive, even in complex applications with frequent state changes.
  - Smoother User Experience: The efficiency of the Virtual DOM in handling updates leads to a smoother user experience. UI updates happen almost instantly, with reduced lag, providing a seamless interaction for users across mobile devices. Additionally, the Virtual DOM can batch multiple updates together, further optimizing performance.
  - Simplified Development: The Virtual DOM allows developers to adopt a declarative approach to building UIs. Instead of worrying about how to efficiently update the entire virtual DOM itself, developers can focus on what the UI should look like at any given state. This simplifies the development process and reduces the potential for errors.
  - Reduced Complexity: Manually optimizing the entire DOM tree for updates is complex and error-prone. The Virtual DOM abstracts this complexity, automatically optimizing updates and making the codebase easier to maintain. This also makes debugging simpler, as the Virtual DOM provides a clear and consistent path from state changes to UI updates.
  - Cross-platform Consistency: Different browsers and platforms handle DOM manipulation in slightly different ways, which can lead to inconsistencies in how UIs are rendered. The Virtual DOM abstracts these differences, ensuring that the UI behaves consistently across all platforms.

## Incremental Dom vs Virtual Dom
* **Key Points:**
  - Virtual DOM (VDOM):
    - Concept: A lightweight in-memory representation of the actual DOM.
    - Operation: Updates the VDOM, which is then efficiently compared with the actual DOM. Only the necessary changes are applied to the real DOM.
    - Advantages:
      - Performance: Significantly faster than direct DOM manipulation, especially for frequent updates.
      - Efficiency: Minimizes unnecessary DOM operations.
      - Abstraction: Provides a declarative approach to UI updates, making code easier to read and maintain.
    - Angular's Implementation: Uses a VDOM approach for rendering and updating components. Angular's VDOM is optimized for performance and change detection.
  - Incremental DOM (IDOM):
    - Concept: A VDOM-based approach that focuses on incremental updates.
    - Operation: Updates the VDOM and directly applies the changes to the real DOM, avoiding unnecessary diffing.
    - Advantages:
      - Efficiency: Can be more efficient for very simple updates, as it avoids the overhead of diffing.
      - Directness: Provides a more direct relationship between VDOM updates and DOM changes.
    - Angular's Implementation: While Angular doesn't explicitly use IDOM, its VDOM implementation incorporates some elements of incremental updates for efficiency.
  - Comparison Between VDOM and IDOM:
    - Performance: VDOM is generally faster for complex updates due to its diffing mechanism. However, IDOM can be more efficient for very simple updates.
    - Efficiency: VDOM is more efficient overall due to its ability to minimize unnecessary DOM operations.
    - Complexity: VDOM is slightly more complex to implement and understand due to the diffing process.
    - Directness: IDOM offers a more direct relationship between VDOM updates and DOM changes.
* **Technical Entities (Classes/Functions/APIs):** `Virtual DOM (VDOM)`, `Incremental DOM (IDOM)`

## Conclusion
* **Key Points:**
  - In Summary, Incremental DOM and Virtual DOM both aim to enhance application performance and developer experience, but they do so through different mechanisms. Incremental DOM offers direct and memory-efficient updates, while the Virtual DOM provides a powerful abstraction for managing complex UI changes. The two more important advantages of using virtual dom compares using incremental dom are it is tree-shakable and has a low memory footprint. Understanding these approaches allows developers to choose the best strategy or combine principles from both to meet their specific application needs and performance goals.