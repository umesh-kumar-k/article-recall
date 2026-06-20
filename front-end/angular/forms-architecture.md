---
aliases:
  - Reactive Forms as State Machines
highlights: FormGroup/FormControl exposes valueChanges and statusChanges observablesl forms are reactive state, not just input containers
Source 1: https://angular.love/angular-forms-reactive-design-patterns-catalog
Source 2: https://angular.love/signal-forms-in-angular-21-complete-guide
---
# Angular Forms: reactive design patterns catalog

## Design Patterns
* **Key Points:**
  - A design pattern is a general, reusable solution to a commonly occurring problem within a given context. The patterns will be presented for Angular but they are also valid for other frameworks.
  - The main goals behind these patterns are: Enhancing maintainability; Reducing the number of bugs
  - Our design patterns are the core of a pattern language that I used to create complex Angular forms.
* **Technical Entities (Classes/Functions/APIs):** `ReactiveFormsModule`, `FormGroup`, `FormControl`, `RxJS`

## Form Model
* **Key Points:**
  - The most important components in Angular Forms are FormGroup and FormControl. The two components are untyped and present a low-level general-purpose API. In complex applications, FormGroup and FormControl, usually, are not the good abstraction to use in Angular components. Instead, we need a high-level abstraction which is specific to the web page components.
  - To tackle the application complexity, we need to introduce a layer of abstraction above the Angular API. We call this layer the Form Model pattern which defines a wrapper around a FormGroup instance. It provides a special-purpose API over the Angular generic forms API. So, instead of using Angular model terms, we use our application terminology.
  - The PersonCreationForm class presents the following characteristics:
    - present a high-level API: ageIsGreaterThan() is an abstraction over the low level plumbing details to determine if the age is greater than the passed parameter.
    - expose a reactive API: except asFormGroup() property, all data are accessible via Observables,
    - expose Angular non reactive properties via Observable: isValid() return an Observable that emit a value when FormGroup.valid property changes,
    - add missing functionality in Angular form model: Angular do not allow access to form initial value,
    - and make the FormGroup accessible.
  - Note: The form model is a leaky abstraction because the view needs to access the wrapped FormGroup.
* **Technical Entities (Classes/Functions/APIs):** `FormGroup`, `FormControl`, `FormBuilder`, `Observable`
* **Code Snippet:**
```typescript
class PersonCreationForm {
  readonly initialValue;

  constructor(private formGroup: FormGroup) {
    this.initialValue = formGroup.value;
  }

  get asFormGroup() {
    return this.formGroup;
  }

  isValid(): Observable<boolean> {
    return this.formGroup.statusChanges.pipe(
      map(() => this.formGroup.valid),
      startWith(false)
    );
  }

  ageIsGreaterThan(min: number): Observable<boolean> {
    return this.formGroup.valueChanges.pipe(
      map(value => value.age),
      distinctUntilChanged(),
      map(it => it > min),
      startWith(false)
    );
  }
}
```

## Form Factory
* **Key Points:**
  - The first step to add a complex Angular form to a web page is to create a FormGroup object and bind it to a view template. The form group creation mainly defines form fields, initial values and validators. This creational logic is usually leaked and mixed with other types of logic inside Angular components.
  - The creation of the FormGroup or better the Form Model should be separated from the rest of logic. To isolate it, we should use the Form Factory pattern. The Angular component showing the form should be unaware of this details.
  - In summary, Form Factory responsibilities are: create form group; define validation rules; define initial values
* **Technical Entities (Classes/Functions/APIs):** `FormBuilder`, `@Injectable()`, `useFactory`, `deps`, `providers`
* **Code Snippet:**
```typescript
@Injectable()
class PersonCreationFormFactory {
  constructor(private formBuilder: FormBuilder) {}

  create(): PersonCreationForm {
    const formGroup = this.createFormGroup();
    return new PersonCreationForm(formGroup);
  }

  private createFormGroup() {
    return this.formBuilder.group({
      name: [""],
      age: [""]
    });
  }
}
```

## Form Data Provider
* **Key Points:**
  - Form fields can provide advanced assistance to users. The most frequent assistance is the autocomplete. This logic is usually used only in the view. But, it is prepared by the component class.
  - The Form Data Provider is a pattern that promotes the segregation of this logic in a service which is used only by component's HTML template. The unique role of this service is to provide the dynamic data needed by the fields.
  - The data returned by this service can depend on the field search term and also on other field value.
  - In a big form with many autocomplete fields and a complex logic, we should use multiple data provider services to keep the code easy to grasp. We can even have a dedicated data provider for each field. But in less complex cases, we can create a single service for each form.
* **Technical Entities (Classes/Functions/APIs):** `HttpClient`, `Observable`, `debounceTime`, `distinctUntilChanged`, `switchMap`
* **Code Snippet:**
```typescript
@Injectable()
class PersonCreationFormDataProvider {
  constructor(private httpClient: HttpClient) {}

  searchCountry = (termChanged: Observable<string>): Observable<string[]> =>
    termChanged.pipe(
        debounceTime(200),
        distinctUntilChanged(),
        switchMap(term => term.length < 3 ? of([]) : this.findCountryBy(term)),
        map(values => values.map(country => country.name))
      );

  private findCountryBy(term: string) {
    return this.httpClient.get<any[]>(`https://restcountries.eu/rest/v2/name/${term}?fields=name`);
  }
}
```

## Form Actions
* **Key Points:**
  - Each form has at least one action that can be executed after filling its fields. The form action is usually entangled with creational and computational logic. For a form with a few logic, it is not a real problem. But with complex forms, mixing this logic with other ones make the code implicit and difficult to understand very quickly.
  - A good solution to enhance the maintainability is to use the Form Actions pattern which puts the action logic in a dedicated class.
  - We no longer use the click event. Instead, we use a custom directive to define a click event listener. So, the view directly notify our service subjects.
* **Technical Entities (Classes/Functions/APIs):** `Subject`, `@Directive`, `@HostListener`, `@Input`
* **Code Snippet:**
```typescript
@Injectable()
class PersonCreationFormActions {
  validateButtonClicked = new Subject<void>();
  resetButtonClicked = new Subject<void>();

  constructor(private form: PersonCreateForm) {
    this.handleValidateButtonClick();
    this.handleResetButtonClick();
  }

  private handleValidateButtonClick() {
    this.validateButtonClicked
        .subscribe(() => alert('The form is validated!'))    
  }

  private handleResetButtonClick() {
    this.resetButtonClicked
        .subscribe(() => this.form.reset())    
  }
}
```

## Search Form
* **Key Points:**
  - In many cases, web application users want to share a search result or tag a search request/criteria in the browser bookmarks. In SPA, a best practice is to put the search criteria in the URL query parameters. The native browser behavior allows users to share URL and to navigate in the history of search.
  - The UrlStore is a generic class that can be extended by implementing the ParamsConverter interface. Its API is composed of two main parts:
    - First, it allows to change URL content using setSource method. This method could be replaced by a Subject as a class attribute.
    - Second, it expose three attributes to detect different types of change in URL: changed, refreshed and changedOrRefreshed.
  - In a search form, the URL store is used in the form actions class to update the URL query parameters.
* **Technical Entities (Classes/Functions/APIs):** `UrlStore`, `ParamsConverter`, `Router`, `ActivatedRoute`, `queryParams`, `NavigationExtras`, `setSource`, `changed`, `refreshed`, `changedOrRefreshed`
* **Code Snippet:**
```typescript
export interface ParamsConverter<T> {
  fromUrl(Params): T;
  toUrl(T): Params;
}

@Injectable()
export class UrlStore<T> {
  changed: Observable<T>;
  refreshed = new Subject<T>();
  changedOrRefreshed: Observable<T>;

  constructor(
    @Inject(URL_STORE_CONVERTER) private converter: ParamsConverter<T>,
    private router: Router,
    private route: ActivatedRoute,
  ) {
    this.changed = this.route.queryParams.pipe(map(converter.fromUrl));
    this.changedOrRefreshed = merge(this.changed, this.refreshed);
  }

  setSource(paramsChanges: Observable<T>) {
    paramsChanges.subscribe(params => {
      const urlParams = this.converter.toUrl(params);    
      const extras = {
        relativeTo: this.route,
        queryParams: removeEmptyAtrributes(urlParams),
      } as NavigationExtras;

      this.router.navigate(['.'], extras).then(result => {
        const urlIsTheSame = result === null;
        if (urlIsTheSame) {
          this.refreshed.next(params);
        }
      });
    });
  }
}
```

## FAQ
* **Key Points:**
  - Why Angular is not a full reactive framework? Angular form API for example is not fully reactive. FormControl has some properties such valid without a reactive equivalent. The other example is the Outputs: even under the hood Angular uses RxJS to handle the outputs, it does not expose an observable allowing it to listen to output events. But, on the other side Angular has many reactive API such as Router, HttpClient.
  - Is @Output() useless? Under the hood, the @Output() is based on RxJS Observables, but the subscribe isn't visible in our application. If we want to fully use Observable in our code we can pass a Subject as input.
  - Why we don't use Presentational and Container patterns? Because it is an anti-pattern. It was promoted in a framework having a limitation that Angular does not have. An Angular component should have only the responsibilities related to the view and the view model patterns. I stopped thinking about this pattern more than a year ago and my code is much better now.
  - Why do we use only component providers? Our implementation is based mainly on component providers. The main advantage of this type of provider is having the same lifecycle as their component.
  - What about code source navigation? Adopting the patterns of this post, makes the navigation between files a little bit harder. We should use a good IDE that simplifies navigation between related files.
  - What about third party libraries? If we adopt a reactive approach in our Angular applications, we should carefully choose our third party libraries. Libraries should provide reactive APIs. In our autocomplete example we used ngx-bootstrap, which exposes a reactive API that not all other component libraries expose. If we develop a home made components library encapsulating third party dependencies, it should expose the two style API the reactive and the old school: as not all teams can adopt a reactive approach.
* **Technical Entities (Classes/Functions/APIs):** `FormControl.valid`, `@Output()`, `Subject`, `ngx-bootstrap`

## Conclusion
* **Key Points:**
  - In this post we tried to present a pattern language for the more used part of Angular in Enterprise Intensive Form Applications. The set of patterns was explained with just enough details to expose the big picture. The implementation details of every pattern varies depending on the use cases.
  - Adopting this kind of patterns make the application better structured and easy to grasp and to maintain. Another advantage to have such a stable design is facilitating the adoption of a good testing strategy.

---

# Signal Forms – Complete Guide

## Introduction to Signal Forms
* **Key Points:**
  - Signal forms themselves are already a novelty, working based on signals that we already know quite well. Nevertheless, they introduce new nomenclature and features to the world of forms that we have no chance of knowing based on previous years with Reactive and Template Driven Forms. One of them is form model.

### Form Model
* **Key Points:**
  - Form model is a writable signal with which we initialize our form. This is crucial because form model directly corresponds to the type of our form. Forms are now very well typed and directly infer the type from the initializing object.
  - Another very important point is that any modifications and updates to our form model will be directly propagated and reflected by the form. Currently this initialization signal is the owner of the state that the form represents – they remain in full synchronization.
  - This is a fundamental change compared to reactive forms. In the previous approach, the form managed its own state independently – we mapped object fields to form controls, but the form state existed independently from the source object. Any synchronization of the form with an external model required manual value updates.
* **Technical Entities (Classes/Functions/APIs):** `signal()`, `form()`
* **Code Snippet:**
```typescript
export class LoginComponent {
 // Form model
 loginModel = signal({
   email: '',
   password: ''
 })

 // We init form with defined form model 
 loginForm = form(this.loginModel)
}
```

### Initializing the Form
* **Key Points:**
  - Creating a new form is done through the form() function. This is already characteristic for Angular that more and more functionality is implemented in a functional way. The first argument of the function is our previously mentioned form model, which will give type to our form and based on it the Form Tree will be initialized – a hierarchical structure of fields where each object in the model becomes a node with its own children, and each primitive value becomes a terminal field (leaf). Thanks to this, navigation through the form naturally corresponds to navigation through data.
* **Technical Entities (Classes/Functions/APIs):** `form()`, `@angular/forms/signals`

## Typing – End of Compromises
* **Key Points:**
  - Signal forms, designed from the ground up with TypeScript in mind, solve these problems.
  - Signal forms – full navigation typing.
  - Signal forms preserve full structure.
  - Signal forms – model is the source of truth.
  - The heart of the type system is FieldTree<TModel> – a type that recursively maps model structure to form structure.

### Problem 1: Nullable Everywhere
* **Key Points:**
  - In Reactive Forms each FormControl by default has type T | null
  - Signal forms – type comes directly from the model: const model = signal({ email: '' }); const myForm = form(model); myForm.email().value(); // string - no null!

### Problem 2: get() Method Loses Types
* **Key Points:**
  - This is one of the most irritating aspects of typed Reactive Forms: The get() method takes a string – TypeScript is not able to verify if the path is correct.
  - Signal forms – full navigation typing.

### Problem 3: FormArray Loses Structure
* **Key Points:**
  - FormArray in typed forms can be problematic
  - Signal forms preserve full structure

### Problem 4: Dynamic Forms
* **Key Points:**
  - Adding controls at runtime is a typing nightmare
  - Signal forms – model is the source of truth

## Validation
* **Key Points:**
  - Similar to Reactive Forms, we have access to predefined validators. However, the way of applying them is completely different – instead of passing validators when creating a control, we call functions pointing to the field and validator.

### Predefined Validators
* **Key Points:**
  - Signal forms provide a set of built-in validators: required(path); min(path, minValue); max(path, maxValue); minLength(path, length); maxLength(path, length); pattern(path, regex); email(path)
* **Technical Entities (Classes/Functions/APIs):** `required()`, `min()`, `max()`, `minLength()`, `maxLength()`, `pattern()`, `email()`

### Custom Validators
* **Key Points:**
  - Creating custom validators is simpler than ever
  - Validator context (ctx) gives access to: value() – current field value; valueOf(path) – value of any other field; state – full field state (touched, dirty, etc.); stateOf(path) – state of any other field
* **Technical Entities (Classes/Functions/APIs):** `validate()`, `customError()`

### Validator Reactivity – Automatic Dependency Tracking
* **Key Points:**
  - Here lies one of the biggest advantages of signal forms. Validators work inside a reactive context, which means Angular automatically tracks all read signals.
  - This validator will run when: confirmPassword changes (because we call value()); password changes (because we call valueOf(f.password))
  - In Reactive Forms point 3 required manual work: this.form.get('password').valueChanges.subscribe(() => { this.form.get('confirmPassword').updateValueAndValidity(); });
  - In signal forms this happens automatically. Zero subscriptions, zero manual updateValueAndValidity() calls.

### Performance Consideration
* **Key Points:**
  - Since the validator reacts to all read signals, it's worth reading only what's really needed

### Asynchronous Validation
* **Key Points:**
  - For validation requiring server requests we have validateAsync and validateHttp
  - Asynchronous validation runs only when synchronous validation passes successfully.
* **Technical Entities (Classes/Functions/APIs):** `validateAsync`, `validateHttp`

## Conditional Functions
* **Key Points:**
  - Analogously to validators, we have functions allowing dynamic control of field state
  - Key difference from Reactive Forms: these functions are reactive. Changing orderType will automatically enable/disable the discountCode field – without manual subscribing and calling enable()/disable().
* **Technical Entities (Classes/Functions/APIs):** `disabled()`, `hidden()`, `readonly()`
* **Code Snippet:**
```typescript
import { form, disabled, hidden, readonly } from '@angular/forms/signals';

const orderForm = form(this.model, (order) => {
  // Field disabled conditionally
  disabled(order.discountCode, ({ valueOf }) => 
    valueOf(order.orderType) === 'wholesale'
  );

  // Field hidden conditionally
  hidden(order.companyName, ({ valueOf }) => 
    valueOf(order.customerType) !== 'business'
  );

  // Read-only field
  readonly(order.totalPrice);
});
```

## Reusability – Schema
* **Key Points:**
  - Schemas are a complete novelty. Schema allows defining a set of rules once and applying them in multiple places.
  - Schemas are a powerful tool for organizing application architecture. You define rules for Address once – you're sure that every form with an address validates it identically.
* **Technical Entities (Classes/Functions/APIs):** `schema()`, `apply()`, `applyEach()`, `applyWhen()`, `applyWhenValue()`
* **Code Snippet:**
```typescript
import { schema, required, email, minLength } from '@angular/forms/signals';

// Define once
const addressSchema = schema<Address>((addr) => {
  required(addr.street);
  required(addr.city);
  required(addr.zipCode);
  pattern(addr.zipCode, /^\d{2}-\d{3}$/);
});

const contactSchema = schema<Contact>((contact) => {
  required(contact.email);
  email(contact.email);
  minLength(contact.phone, 9);
});
```

## Field Directive – One Way for Everything
* **Key Points:**
  - In Reactive Forms we had to remember about different directives
  - Signal forms simplify this to one [field] directive
  - The Field directive is strictly typed. When you try to bind a number type field to an input expecting string, TypeScript reports an error. This is something unseen before in Angular forms – type errors detected in template!
  - The Field directive automatically synchronizes state between field and UI control.
* **Technical Entities (Classes/Functions/APIs):** `[field]`, `FormValueControl`

## Migration – compatForm
* **Key Points:**
  - If you have an existing application with Reactive Forms, you probably won't rewrite everything at once (and rightly so). Fortunately, Angular anticipated this scenario and provides compatForm() – a function allowing to mix both worlds.
  - How Does It Work? compatForm automatically "unwraps" values from FormControl
  - State is synchronized both ways
  - Validators Are Respected
  - Limitation: No Rules on FormControl Fields. You cannot apply signal forms rules (like required(), validate()) directly to fields that are FormControl – TypeScript will block it.
* **Technical Entities (Classes/Functions/APIs):** `compatForm()`, `FormControl`
* **Code Snippet:**
```typescript
import { compatForm } from '@angular/forms/signals';
import { FormControl, Validators } from '@angular/forms';

// Existing FormControl with validators
const ageControl = new FormControl(5, Validators.min(3));

// Model mixing signal forms with Reactive Forms
const model = signal({
  name: 'Jan',           // regular signal forms field
  age: ageControl        // existing FormControl
});

const myForm = compatForm(model);
```

## Submit and Reset
* **Key Points:**
  - Signal forms provide a submit() function that handles typical submission flow
  - What submit() does under the hood: Marks all fields as touched (to show errors); Checks valid() – if false, aborts and doesn't call action; Sets submitting to true; Calls the action; Applies any server errors to appropriate fields; Sets submitting to false
  - You can use submitting() to block UI
  - The reset() method clears interaction state (touched, dirty)
* **Technical Entities (Classes/Functions/APIs):** `submit()`, `submitting()`, `reset()`

## Debouncing
* **Key Points:**
  - For fields where we don't want to react to every keystroke (e.g. search, async validation), we have debounce()
  - Debouncing is inherited – if you set it on parent, children will also be debounced (unless they override with their own).
* **Technical Entities (Classes/Functions/APIs):** `debounce()`

## Custom Controls – End of ControlValueAccessor
* **Key Points:**
  - In Reactive Forms creating a custom form control required implementing ControlValueAccessor – an interface with four methods, magical provider with forwardRef, and manual calling of onChange/onTouched. Every Angular developer knows this boilerplate.
  - Signal Forms reduce this to one line.
  - To create a control compatible with [field] directive, you just need to implement FormValueControl<T> interface
  - FormValueControl defines a number of optional inputs. If you declare them, [field] directive will automatically fill them
  - For checkbox-type controls there is a separate FormCheckboxControl contract
  - Control doesn't have to be a component – it can be a directive on a native element
* **Technical Entities (Classes/Functions/APIs):** `FormValueControl`, `FormCheckboxControl`, `ControlValueAccessor`
* **Code Snippet:**
```typescript
import { Component, model } from '@angular/core';
import { FormValueControl } from '@angular/forms/signals';

@Component({
  selector: 'my-input',
  template: `
    <input 
      [value]="value()" 
      (input)="value.set($event.target.value)"
    />
  `
})
export class MyInputComponent implements FormValueControl<string> {
  readonly value = model('');
}
```

## Before You Start – A Few Notes
* **Key Points:**
  - Signal forms are marked as @experimental 21.0.0
  - Does this mean they're not worth using? In my opinion – they are worth it, especially in new projects. But in critical production applications consider whether you're ready for potential API migrations.
  - Signal forms live in a separate entry point: import { form, required, validate, ... } from '@angular/forms/signals';
* **Technical Entities (Classes/Functions/APIs):** `@angular/forms/signals`

## Summary
* **Key Points:**
  - Signal forms are not an evolution of Reactive Forms – they are a rethought from scratch implementation of forms in Angular. Key changes: Model as source of truth – form and data are always synchronized; Real typing – TypeScript knows everything, without compromises; Reactivity out of the box – validators react to dependency changes without manual binding; One API – [field] directive instead of a zoo of directives; Schemas – reusable validation rules; Simple Controls – FormValueControl instead of ControlValueAccessor
  - Should you migrate existing applications? If you have time and budget – yes. If not – compatForm allows introducing signal forms gradually, form by form.
  - And new projects? There's no dilemma here. Signal forms are the future of forms in Angular.