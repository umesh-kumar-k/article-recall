---
aliases:
  - Core Enterprise Stack with Angular - Angular CLI + Nx
highlights: CLI for scaffolding; Nx adds monorepo orchestration, code generators, dependency graph, affected builds - standard for multi-team orgs
Source 1: https://www.angulararchitects.io/en/blog/tutorial-first-steps-with-nx-and-angular-architecture/
Source 2: https://nx.dev/blog/architecting-angular-applications
Source 3: https://angular.love/beyond-clean-code-building-a-scalable-angular-frontend-architecture-with-nx-monorepos
Source 4: https://angular.love/scalable-modular-angular-application-with-nx
Source 5: https://nx.dev/blog/scaffold-angular-rspack-applications
---

# Angular Architecture Guide To Building Maintainable Applications at Scale (Complete Extraction with Code Snippets)

## Project Paradox
* **Key Points:**
  - A good software architecture helps mitigate the Project Paradox by enabling reversible decisions, progressive evolution, and delaying commitments until more knowledge is available. This can be achieved by aiming for a modular design that facilitates encapsulation, allowing incremental changes, and decoupling dependencies with clearly defined boundaries.

## Pizza-boxes, Onions and Hexagons - How to organize code
* **Key Points:**
  - Probably the simplest and most widespread (and probably most straightforward) approach for separating different aspects of an application is the layered architecture (in Italy we call them Pizza-box architecture).
  - The common denominator of these architectures (often also denoted as horizontal approaches) is that they are mostly focused on dividing the system based on technical responsibilities.
  - On the other hand, vertical architecture approaches organize the application into functional segments or domains focusing on the business capabilities. This is a common approach in microservices architectures.

## Breaking up the Monolith - How to identify where boundaries are
* **Key Points:**
  - Instead of organizing code by technical types (components, services, directives) we want to structure our codebase around business domains. Good candidates for domains are areas that:
    - have distinct business capabilities (e.g. in the example of an online shop: orders, products, payments)
    - reflect team structure within the organization; domain boundaries often mirror organizational structure (Conway's Law)
    - can evolve independently from each other, at different speeds
    - have clear responsibilities and boundaries
  - In general, start broad and then refine over time as you gain more insights.
  - Having such boundaries clearly separated not only helps with the longer-term maintainability of the application, but also helps assign teams and minimizes cross-team dependencies.

## Start Small, Grow as you Need It
* **Key Points:**
  - A common mistake is to think too much about the ideal end goal and prepare things "just in case". Yep, over-engineering. Exactly, you might not need a monorepo (at least not yet). However, you want to make sure to not add roadblocks in your way.
  - A lot of our Angular users don't necessarily start to use Nx because they need a monorepo, but because they want to be able to modularize their monolithic codebase.
  - If you want to start building a single Angular application with Nx, you can use the `--preset=angular-standalone` flag: `npx create-nx-workspace myshop --preset=angular-standalone`
  - If you have an existing Angular CLI project, you can also add Nx support to it by running: `npx nx@latest init`
  - If you already know you want to go straight to an Nx monorepo, you can add the `--integrated` flag to the nx init command.
* **Technical Entities (Classes/Functions/APIs):** `Nx`, `--preset=angular-standalone`, `npx nx@latest init`, `project.json`
* **Code Snippet:**
```json
{
  "name": "myshop",
  "sourceRoot": "./src",
  "targets": {
    "build": {
      "executor": "@angular-devkit/build-angular:browser",
      "outputs": ["{options.outputPath}"],
      "options": {
        "outputPath": "dist/myshop",
        "index": "./src/index.html",
        "main": "./src/main.ts",
        ...
      },
      "configurations": {...},
      "defaultConfiguration": "production"
    },
    "serve": {
      "executor": "@angular-devkit/build-angular:dev-server",
      ...
    },
    ...
    "lint": {
      "executor": "@nx/eslint:lint",
      "options": {
        "lintFilePatterns": ["./src"]
      }
    },
    "test": {
      "executor": "@nx/jest:jest",
      "outputs": ["{workspaceRoot}/coverage/{projectName}"],
      "options": {
        "jestConfig": "jest.config.ts"
      }
    },
    "serve-static": {
      "executor": "@nx/web:file-server",
      "options": {
        "buildTarget": "myshop:build",
        "port": 4200,
        "spa": true
      }
    }
  }
}
```

## Modularize your Code into Projects Following Your Domain Areas
* **Key Points:**
  - This feature-based organization is already an improvement over the traditional "type-based" structure (where code is organized by technical type like components/, services/, etc.). However, it still has limitations:
    - Boundaries are purely folder-based with no real enforcement
    - Easy to create unwanted dependencies between features
    - Hard to maintain as the application grows
    - No clear rules about what can depend on what
  - Instead of relying on folder-based separation, we can create dedicated projects (also called libraries) for different parts of our application.
  - These projects aren't necessarily meant to be published as npm packages - their main purpose is to create clear boundaries in your codebase. The application still builds everything together, but the project structure helps maintain clear separation of concerns.
  - As domains grow more complex, you might want to split them further into more specialized libraries.
  - This structure follows a pattern where each domain can have:
    - Feature libraries (feat-*): Implement specific business features or pages
    - UI libraries (ui-*): Contain presentational components
    - Data-access libraries: Handle API communication and state management
  - Since this is a standalone application (not a monorepo), we use TypeScript path mappings to link these projects together.
  - These mappings allow you to have clear imports in your code.
* **Technical Entities (Classes/Functions/APIs):** `TypeScript path mappings`, `tsconfig.base.json`
* **Code Snippet:**
```json
{
  "compilerOptions": {
    "paths": {
      "@myshop/products-data-access": [
        "packages/products/data-access/src/index.ts"
      ],
      "@myshop/products-feat-product-list": [
        "packages/products/feat-product-list/src/index.ts"
      ],
      "@myshop/products-ui-product-card": [
        "packages/products/ui-product-card/src/index.ts"
      ],
      ...
    }
  }
}
```

## The Application is Your Linking and Deployment Container
* **Key Points:**
  - In a well-modularized architecture, your application shell should be surprisingly thin. Think of your main application as primarily a composition layer: it imports and coordinates the various domain libraries but contains minimal logic itself.
  - The ideal application structure has:
    - Thin application shell - Contains mainly routing configuration, bootstrap logic, and layout composition
    - Domain libraries - All business logic, UI components, and data access code
  - When building, Nx compiles all the imported libraries together with your application code, creating a single deployable bundle. The libraries themselves don't produce deployable artifacts, they are implementation details that are consumed by the application.
  - This pattern makes it much easier to move features between applications later if needed, as your business logic isn't tied to any specific application shell. It also provides a clear mental model: applications are for deployment, libraries are for code organization and reuse.
* **Technical Entities (Classes/Functions/APIs):** `Route`, `loadComponent`
* **Code Snippet:**
```typescript
export const appRoutes: Route[] = [
  {
    path: 'products',
    loadComponent: () =>
      import('@myshop/products-feat-product-list').then(
        (m) => m.ProductsFeatProductListComponent
      ),
  },
  {
    path: 'product/:id',
    loadComponent: () =>
      import('@myshop/products-feat-product-detail').then(
        (m) => m.ProductsFeatProductDetailComponent
      ),
  },
  {
    path: 'reviews',
    loadComponent: () =>
      import('@myshop/products-feat-product-reviews').then(
        (m) => m.ProductsFeatProductReviewsComponent
      ),
  },
  {
    path: 'orders',
    loadComponent: () =>
      import('@myshop/orders-feat-order-history').then(
        (m) => m.OrdersFeatOrderHistoryComponent
      ),
  },
  {
    path: 'create-order',
    loadComponent: () =>
      import('@myshop/orders-feat-create-order').then(
        (m) => m.OrdersFeatCreateOrderComponent
      ),
  },
  {
    path: 'checkout',
    loadComponent: () =>
      import('@myshop/checkout-feat-checkout-flow').then(
        (m) => m.CheckoutFeatCheckoutFlowComponent
      ),
  },
  ...
  {
    path: '',
    redirectTo: 'products',
    pathMatch: 'full',
  },
];
```

## When to Create a New Library
* **Key Points:**
  - There is really no correct or wrong answer here. You should not just go and create a library for each component. That's probably too much. It really depends on how closely related various components or use cases are.
  - A good rule of thumb is to understand and see how often various parts change over time. As your application grows, watch for these signs that your library boundaries might need adjustment:
    - Frequent Cross-Library Changes - You consistently need to modify multiple libraries for a single feature change
    - Circular Dependencies - Libraries depend on each other in ways that create circular references
    - Unclear Ownership - Multiple teams frequently need to coordinate to modify the same library
    - Complex Dependencies - Simple features require importing from many different libraries or domains
    - Excessive Shared Code - You find yourself duplicating types and utilities across domains
  - Remember that library boundaries aren't set in stone - they should evolve with your application. Start with broader boundaries and refine them as you gain insights into how your code changes together.

## Guard Your Boundaries - Automatically Enforcing Clear Dependencies
* **Key Points:**
  - Once you've established your domain boundaries and architectural layers, you need to ensure they remain intact as your codebase grows. Nx provides powerful tools to enforce these boundaries through module boundary rules that can be configured in your ESLint configuration.
  - In our example, we use a dual-tagging approach:
    - Scope tags (scope:<name>): These reflect our domain boundaries, representing different business capabilities like products, orders, checkout etc. They encode our vertical slicing approach.
    - Type tags (type:<name>): These represent our horizontal architectural layers such as feature, ui, data-access, and util.
  - The rules enforce a clear dependency structure:
    - Type rules ensure architectural layering. For instance, UI components can only depend on other UI components, utilities, and data-access libraries. This prevents circular dependencies and maintains a clean architecture.
    - Domain rules control which domains can talk to each other. For example, the orders domain can depend on products (since orders contain products), but products cannot depend on orders.
    - Every domain can depend on shared code, but shared code can only depend on other shared code, preventing it from becoming a source of circular dependencies.
  - These rules are enforced at build time through ESLint. If a developer tries to import from a forbidden domain or layer, they'll receive an immediate error, helping maintain the architectural integrity of your application. This allows you to get feedback as early as possible when you run your PR checks on CI.
  - Note that this tagging structure is just a suggestion - you can adapt it to your specific needs. The key is to have clear, enforceable boundaries that reflect both your technical architecture and your business domains.
* **Technical Entities (Classes/Functions/APIs):** `ESLint`, `module boundary rules`, `scope tags`, `type tags`
* **Code Snippet:**
```typescript
// Type-based rules
{
  sourceTag: 'type:feature',
  onlyDependOnLibsWithTags: ['type:feature', 'type:ui', 'type:data-access']
},
{
  sourceTag: 'type:ui',
  onlyDependOnLibsWithTags: ['type:ui', 'type:util', 'type:data-access']
},

// Domain-based rules
{
  sourceTag: 'scope:orders',
  onlyDependOnLibsWithTags: ['scope:orders', 'scope:products', 'scope:shared']
},
{
  sourceTag: 'scope:products',
  onlyDependOnLibsWithTags: ['scope:products', 'scope:shared']
}
```

## Automate Your Standards
* **Key Points:**
  - As your workspace grows, it becomes increasingly important to automate and enforce your team's standards and best practices.
  - The key to successful automation is finding the right balance. Start with automating the most common patterns that need standardization, and gradually add more automation as patterns emerge. Focus on the standards that provide the most value to your team. These automation capabilities are really here to ensure that your team's standards are not just documented but actively enforced through tooling, making it easier for developers to do the right thing by default.

### Custom Generators for Consistent Code Generation
* **Key Points:**
  - Nx is extensible. As such it allows you to create custom code generators that you can use to encode your organization's standards and best practices.
  - The generator itself is just a function that manipulates files.
  - Your team can then run these generators through the Nx CLI (via the nx generate ... command) or Nx Console.
* **Technical Entities (Classes/Functions/APIs):** `Nx`, `custom code generators`, `nx generate`, `Nx Console`
* **Code Snippet:**
```typescript
import { Tree, formatFiles } from '@nx/devkit';

export default async function (tree: Tree, schema: any) {
  // Add your generator logic here
  // For example, create files, modify configurations, etc.

  await formatFiles(tree);
}
```

### Leverage Nx Console AI Integration
* **Key Points:**
  - If you use Nx Console, Nx's editor extension for VSCode and IntelliJ, then you should already have the latest AI capabilities enabled.
  - Nx Console just got some enhancements with the goal of providing contextual information to editor integrated LLMs such as Copilot and Cursor. By providing Nx workspace metadata to these models they are able to provide much more valuable, context specific information and perform actions via the Nx CLI.
* **Technical Entities (Classes/Functions/APIs):** `Nx Console`, `Copilot`, `Cursor`

## Single-app vs Multiple App Deployment
* **Key Points:**
  - While this approach is simple and works well for smaller applications, there are several scenarios where you might want to split your application:
    - Different scaling requirements: Your customer-facing store might need high availability and scalability to handle thousands of concurrent users, while your admin interface serves a much smaller number of internal users
    - Resource optimization: Not all users need all features. For example, administrative features like inventory management are only needed by staff members
    - Independent deployment cycles: Different parts of your application might need to evolve at different speeds. Your admin interface might need frequent updates for internal tools, while your customer-facing store remains more stable
    - Security considerations: Keeping administrative features in a separate application can reduce the attack surface of your customer-facing application
  - As your application grows, you might want to split it into multiple applications - perhaps separating your customer-facing storefront from your administrative interface. The first step is to convert your standalone application into a monorepo structure. Nx comes with a convert-to-monorepo command to do exactly that: `nx g convert-to-monorepo`
  - This command moves your existing application into an apps directory and adjusts any configuration that needs to be adjusted for supporting multiple side-by-side applications in a monorepo setup.
* **Technical Entities (Classes/Functions/APIs):** `nx g convert-to-monorepo`, `apps directory`, `monorepo`

## Creating Multiple Applications
* **Key Points:**
  - Once you have a monorepo structure, you can create additional applications that share code with your original app. For example, you might want to create an admin application: `nx g @nx/angular:app admin`
  - Now you can move administrative features (like inventory management) to the new admin app by importing libraries relevant to the new app (such as for example the inventory management libraries).
  - You can see how the already modular structure allows you to adjust your application structure and re-link some of the packages into the new application. Something that would have otherwise been a major undertaking.
  - Similarly to how we now have two applications that can be deployed and scaled independently, we could go even further and convert it into a microfrontend approach. But more on that in another article.
* **Technical Entities (Classes/Functions/APIs):** `nx g @nx/angular:app`

## Scaling Development
* **Key Points:**
  - Obviously as your codebase keeps growing you need to have the tooling support that helps keep it sustainable. In particular CI might become a concern as the number of projects grows. For that purpose Nx has several features to keep your CI fast and efficient:
    - Remote Caching (Nx Replay) ensures your code is never rebuilt or retested unnecessarily.
    - Distributed Task Execution (Nx Agents) intelligently allocates tasks across multiple machines.
    - Atomizer helps manage growing test suites by automatically splitting them into more fine-grained runs and by leveraging Nx Agents to parallelize them across machines.
    - Flaky Task Detection identifies flaky tasks (often automated unit or e2e tests) and re-runs them automatically for you.
  - One of the key advantages of using Nx is that it's not limited to just Angular either. As your application grows, you might need to:
    - Add a documentation site using static site generators like Analog
    - Create landing pages with Next.js or Astro
    - Build backend services with NestJS or Express
    - Add specialized tools for specific business needs
  - Nx supports all these scenarios while maintaining the ability to share code between different technologies.
* **Technical Entities (Classes/Functions/APIs):** `Nx Replay`, `Nx Agents`, `Atomizer`, `Flaky Task Detection`, `Analog`, `Next.js`, `Astro`, `NestJS`, `Express`

## Wrapping up
* **Key Points:**
  - Building maintainable Angular applications at scale requires thoughtful architecture decisions and proper tooling support. We've covered several key aspects:
    - Domain-Driven Structure: Moving from traditional layered architectures to organizing code around business domains creates clearer boundaries and better maintainability.
    - Incremental Adoption: Starting small with a standalone application and growing into a more complex structure as needed, rather than over-engineering from the start.
    - Clear Boundaries: Using projects/libraries to create explicit boundaries between different parts of your application, with automated enforcement through module boundary rules.
    - Automation & Standards: Leveraging custom generators and AI-enhanced tooling to maintain consistency and best practices across your codebase.
    - Scalability Options: Understanding when and how to evolve from a single application to multiple applications or even microfrontends, while maintaining code sharing and reusability.
  - Remember that architecture is not a one-time decision but an evolving process - start with clear boundaries and good practices, then adapt as your application and team's needs grow. The key to success lies in having the proper tooling and automation in place to support you as your application grows.