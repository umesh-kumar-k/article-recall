---
aliases:
  - NgRx Platform(Enterprise Standard)
Source 1: https://blog.bitsrc.io/how-ive-set-up-ngrx-router-store-with-standalone-component-in-angular-16-4af77a632746
Source 2: https://danywalls.com/handling-router-url-parameters-using-ngrx-router-store
---
- Store as Single Source of Truth
	- Immutable state tree; all components derive state via selectors; eliminates synchronization bugs across views

- Selectors as Memoized Projections
	- createSelectors() memoizes on input reference change; never expose raw state shape to components - enables store refactoring without component changes

- Effects for Side Effect Isolation
	- HTTP calls, localStorage, analytics in effects only; components dispatch actions; business logic testable without rendering

- Entity Adapter
	- Normalized state (dictionary by ID + ordered IDs array); O(1) lookups; standard for list-heavy enterprise data

- Router Store
	- Router state in NgRx store;time-travel debugging includes navigation; enables CanActivate guards using store selectors 

- Action Hygiene 
	- One action per unique event; never reuse actions across features; actions are a seriazliable audit log, not function calls

- Facade Patterns
	- Service wrapping store dispatch/selectl components never import NgRx directly; swap store implementation without touching components