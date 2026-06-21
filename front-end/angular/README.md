## Angular Framework Architecture Fundamentals

- [Ngmodules vs Standalone Components](ngmodule-standalone-components.md) 
- [Hierarchial Injector Tree](hierarchical-injector-tree.md) 
- [Dependency Injection at Architecture Scale](dependency-injection.md) 
- [Zone.js & its removal](zone-js.md) 
- [Signal Architecture](signals-architecture.md) 
- [Compilation Model](compilation-model.md) 
- Platform Abstraction

[## Change Detection](change-detection.md) 

- [Default vs OnPush](change-detection/default-on-push.md) 
- Signal Based Change Detection
- [markForCheck() vs detectChanges()](mark-for-check-vs-detect-changes.md) 
- [detach() / Full Manual Control](detach.md) 
- [NgZone.runOutsideAngular()](ngzone-run-outside-angular.md) 
- [Aysnc Pipe vs Architecture Pattern](async-pipe.md) 


[## Routing Architecture](routing-architecture.md) 

 - Lazy Loading Routes
 - Preloading Strategies
 - Route Guards vs Authorization
 - Resolver vs Component Data Fetching
 - Router Event Stream
 - Component Input Binding from Router
 - Named Router Outlets

## Forms Architecture

- [Reactive Forms as State](forms-architecture.md) 
- [Typed Forms (v14+)](typed-forms.md) 
- [Dynamic Form Generation](dynamic-form-generation.md)  
- [ControlValueAccessor](control-value-accessor.md) 
- [Cross-Field Validation](cross-field-validation.md) 
- Async Validators
- Form Array Patterns

## State Management Architecture

- Signal Store
	- [NgRx Signal Store](ngrx/signal-store.md) 
	- Entities Feature
	- Local vs GLobal Signal Store
	- patchState() Immutability

- [NgRx Platform (Enterprise Standard)](ngrx/summary.md) 
	- [Store as Single Source of Truth](ngrx/store.md) 
	- [Selectors as Memoized Projections](ngrx/selectors.md) 
	- [Effects for Side Effect Isolation](ngrx/effects.md) 
	- Entity Adapter
	- [Router Store](ngrx/router-store.md) 
	- Action Hygiene
	- Facade Pattern

- Server State(Separate Concern)
	- TanStack Query
	- NgRx Data
	- CQRS on the Client

## Enterprise Ecosystem & Tooling

- Core Enterprise Stack with Angular
	- [Angular CLI + Nx](enterprise-tooling-cli-nx.md) 
	- [Angular Material + CDK:](enterprise-tooling-material-cdk.md) 
	- PrimeNG / AG Grid:
	- NgRx Ecosystem
	- RxJS
	- Apollo Angular/GraphQL 
	- Angular Fire

- Build & DevOps Tooling
	- Angular Esbuild Builder(v17+)
	- Nx Cloud
	- Webpack 5 Module Federation
	- Compodoc
	- Storybook for Angular
	- Chromatic

- Observability & Quality in Production
	- Angular DevTools
	- Sentry Angular SDK
	- Datadog/Dynatrace RUM
	- Lighthouse CI
	- Bundle Analyzer (esbuild-bundle-analyzer)

- API Integration Patterns
	- HttpClient Interceptors
	- OpenAPI Code Generation
	- MSW(Mock Service Worker)
	- SignalR/WebSockets
	- gRPC-Web

## Design Patterns & Architecture Styles

- Angular Specific Patterns
	- Smart/Dumb Component Split
	- Signal Input + Model
	- Content Projection Strategy
	- Dynamic Component Loading
	- Directive Composition API(v15+)
	- Functional Guards & Interceptors
	- APP_INITIALIZER
	- DestroyRef + takeUntilDestroyed()

- Application Level Patterns
	- Feature Shell Patterns
	- Repository Pattern for Data
	- Facade Services
	- Token Based Feature Flags
	- Plugin Architecture

- Micro Frontend Patterns with Angular
	- Module Federation + Angular
	- Native Federation(esbuild-based)
	- Shared Library Strategy
	- MFE Communication
	- Design Token Federation

- Performance Architecture
	- Bundle Optimization
	- Route-Level Code Splitting
	- Component-Level Lazy Loading
	- @defer with Prefetch
	- Tree-Shaking Discipline
	- Font Optimization

- Runtime Performance
	- trackBy in @for/ngFor
	- Virtual Scrolling(CDK)
	- OnPush + Signals Everywhere
	- runOutsideAngular() for Third-party libs
	- Image Optimization(NgOptimizedImage)
	- Web Vitals Profiling
 
- Memory Management
	- Subscription Lifecycle
	- Large Dataset Management
	- WeakMap for Component Metadata

## Testing Strategy

- Unit Testing
	- Jest over Karma
	- Vitest as Emerging Alternative
	- Test Isolation
	- Selector Testing
	- Effect Testing
	- Signal Testing

- Component/Integration Testing
	- Angular Testing Library
	- Harnesses(CDK Testing Harnesses)
	- TestBed.configureTestingModule()
	- MSW for HTTP Mocking
	- Spectator

- E2E Testing
	- Playwright as Default
	- Page Object Model(POM)
	- API Mocking in E2E
	- Cypress Component Testing
	- Visual Regression

## Deployment & Scaling Strategies

- Build & CI/CD Pipeline Architecture
	- Nx Affected Builds
	- Remote Caching(Nx Cloud/ Turborepo)
	- Docker Multi Stage Builds
	- Environment Configuration Strategy
	- Feature Branch Previews

- CDN & Static Hosting Architecture
	- SPA on CDN 
	- Cache-Control Strategy
	- CDN Edge Caching for SSR
	- Multi-Region Deployment
	- Blue-Green Deployments

- Angular SSR Deployment
	- Angular Universal/SSR
	- Node.js Express Server
	- Serverless Functions (AWS Lambda/Azure Functions)
	- Hybrid Rendering
	- App Engine/Cloud Run(GCP)

- Kubernetes & Container Orchestration
	- Deployment Manifest
	- Nginx as Reverse Proxy
	- ConfigMap for Runtime Config
	- Horizontal Pod Autoscaling
	- Init Containers
	- Readiness vs Liveness Probes

- Micro-Frontend Deployment
	- Independent CI/CD per MFE
	- Remote Entry Manifest
	- Semantic Versioning for Remotes
	- Canary Deployment for Remotes

- Production Scaling Considerations
	- Static Asset CDN Distribution
	- Server-Side Caching (SSR)
	- Request Coalescing
	- Connection Limits
	- Memory Profiling for SSR Node


## Front End Scaling Patterns

- Team Scaling
	- Conway's Law Applied
	- Nx Project Tags + Constraints 
	- Component Ownership Matrix
	- API Contract-First Development

- Codebase Scaling
	- Three Layer Library Taxonomy
	- Standalone Component Migration
	- Barrel File Discipline
	- Schematic Driven Scaffoling

- Performance at Scale
	- Performance Budgets in CI
	- Lazy Load Everything Non Critical
	- Server Timing Headers
	- HTTP/2 Push / Early Hints
	- Shared Worker for State Sync

## Emerging Tools, Frameworks & Packages

- Angular Ecosystem(2024-2026)
	- Angular Signals(Stable,v16+)
	- @defer Blocks
	- Angular Control Flow
	- Hydration & Full App Non- Destructive Hydration
	- Partial Hydration
	- Zoneless Angular
	- AnalogJS
	- @angular/build(Application Builder)
