---
aliases:
  - "Component Communication with Signals: Inputs, Two-Way Bindings, and Content/ View Queries"
Source 1: https://www.angulararchitects.io/blog/component-communication-with-signals-inputs-two-way-bindings-and-content-view-queries/
---
# Component Communication with Signals: Inputs, Two-Way Bindings, and Content/View Queries

## Input Signals
* **Key Points:**
  - Inputs Signals allow us to receive data via Property Bindings in the form of Signals.
  - This input function is picked up by the Angular Compiler, emitting source code for property bindings. Hence, we should only use it together with properties.
  - An InputSignal is always read-only and can be used like a Signal.
  - All changes to the passed Signal will be reflected by the InputSignal in the component. Internally, both Signals are connected via the graph Angular is maintaining. Life cycle hooks like ngOnInit and ngOnChanges can now be replaced with computed and effect.
  - By definition, input.required cannot have a default value. This makes sense at first glance, however, there is a pitfall: If you try to read the value of a required input before it's been bound, Angular throws an exception.
  - Hence, you cannot directly access it in the constructor. Instead, you can use ngOnInit or ngOnChanges. Also, using inputs within computed or effect is always safe, as they are only first triggered when the component has been initialized.
  - Both input and input.require also take a parameter object that allows the definition of an alias.
  - In most cases, you should prevent the usage of aliases, as they create an unnecessary indirection. An often-seen exception to this rule is renaming one of a Directive's properties to match the configured attribute selector.
  - Transformer have already been available for traditional @Inputs. They allow the transformation of a value passed via a property binding.
* **Technical Entities (Classes/Functions/APIs):** `input()`, `input.required()`, `InputSignal`, `booleanAttribute`, `numberAttribute`
* **Code Snippet:**
```typescript
@Component({
  selector: 'app-option',
  standalone: true,
  imports: [],
  template: `
    <div class="option">
      {{ label() }}
    </div>
  `,
  styles: [...]
})
export class OptionComponent {
  label = input.required<string>();
}
```

## Two-Way Data Binding with Model Signals
* **Key Points:**
  - Input Signals are read-only. If you want to pass a Signal that can be updated by the called component, you need to set up a so-called Model Signal.
  - The options are similar to the ones for input: model.required defines a mandatory property, and you can provide an alias via an options object. However, a transformer can not be defined.
  - As usual in Angular, also Signal-based Two-way Bindings can be defined with a (read-only) Input and a respective Output. The Output's name must be the Input's name with the suffix Change.
* **Technical Entities (Classes/Functions/APIs):** `model()`, `model.required()`
* **Code Snippet:**
```typescript
@Component([...])
export class TabbedPaneComponent {
  current = model(0);
  [...]
}
```

## Content Queries with Signals
* **Key Points:**
  - The function contentChildren is the counterpart to the traditional @ContentChildren decorator. As TabComponent was passed as a so-called locator, it returns a Signal with an Array holding all projected TabComponents.
  - Having the projected nodes as a Signal allows us to project them using computed reactively.
  - By default, a Content Query only unveils direct content children. "Grandchildren" are ignored. To also get hold of such nodes, we can set the option descendants to true.
* **Technical Entities (Classes/Functions/APIs):** `contentChildren()`, `contentChild()`, `descendants: true`
* **Code Snippet:**
```typescript
@Component({
  selector: 'app-tabbed-pane',
  standalone: true,
  imports: [],
  template: `
    <div class="pane">
      <div class="nav" role="group">
        @for(tab of tabs(); track tab) {
        <button
            [class.secondary]="tab !== currentTab()"
            (click)="activate($index)">
                {{tab.title()}}
        </button>
        }
      </div>
      <article>
        <ng-content></ng-content>
      </article>
    </div>
  `,
  styles: [...]
})
export class TabbedPaneComponent {
  current = model(0);
  tabs = contentChildren(TabComponent);
  currentTab = computed(() => this.tabs()[this.current()]);

  activate(active: number): void {
    this.current.set(active);
  }
}
```

## Output API
* **Key Points:**
  - For the sake of API symmetricity, Angular 17.3 introduced a new output API. An output function is now used for defining an event provided by a component. Similar to the new input API, the Angular Compiler picks up the call to the output and emits respective code. The returned OutputEmitterRef's emit method is used to trigger the event.
  - Besides this simple way of setting up outputs, you can use an Observable as the source for an output. For this, you find a function outputFromObservable in the RxJS interop layer. The function outputFromObservable converts an Observable to an OutputEmitterRef.
* **Technical Entities (Classes/Functions/APIs):** `output()`, `OutputEmitterRef`, `emit()`, `outputFromObservable`
* **Code Snippet:**
```typescript
@Component([...])
export class TabbedPaneComponent {
  current = model(0);
  tabs = contentChildren(TabComponent);
  currentTab = computed(() => this.tabs()[this.current()]);

  tabActivated = output<TabActivatedEvent>();

  activate(active: number): void {
    const previous = this.current();
    this.current.set(active);
    this.tabActivated.emit({ previous, active });
  }
}
```

## View Queries with Signals
* **Key Points:**
  - While a Content Query returns projected nodes, a View Query returns nodes from its own view. These are nodes found in the template of the respective component. In most cases, using data binding instead is the preferable solution. However, getting programmatic access to a view child is needed in some situations.
  - View children can be represented by different types: The type of the respective Component or Directive, an ElementRef representing its DOM node, or a ViewContainerRef. The latter one is used in the next section. The desired type can be mentioned using the read option used in the previous example.
* **Technical Entities (Classes/Functions/APIs):** `viewChild()`, `viewChildren()`, `ElementRef`, `ViewContainerRef`, `read`
* **Code Snippet:**
```typescript
@Component({
  selector: "app-form",
  standalone: true,
  imports: [FormsModule, JsonPipe],
  template: `
    <h1>Form Demo</h1>
    <form autocomplete="off">
      <input
        [(ngModel)]="userName"
        placeholder="User Name"
        name="userName"
        #userNameCtrl
        required
      />
      <input
        [(ngModel)]="password"
        placeholder="Password"
        type="password"
        name="password"
        #passwordCtrl
        required
      />
      <button (click)="save()">Save</button>
    </form>
  `,
  styles: `
    form {
      max-width: 600px;
    }
  `,
})
export class FormDemoComponent {
  form = viewChild.required(NgForm);

  userNameCtrl =
    viewChild.required<ElementRef<HTMLInputElement>>("userNameCtrl");
  passwordCtrl =
    viewChild.required<ElementRef<HTMLInputElement>>("passwordCtrl");

  userName = signal("");
  password = signal("");

  save(): void {
    const form = this.form();

    if (form.controls["userName"].invalid) {
      this.userNameCtrl().nativeElement.focus();
      return;
    }

    if (form.controls["password"].invalid) {
      this.passwordCtrl().nativeElement.focus();
      return;
    }

    console.log("save", this.userName(), this.password());
  }
}
```

## Programmatically Setting up an Output
* **Key Points:**
  - To set up an handler for this event, we can directly use the returned ComponentRef's instance property. It points to the added component instance and hence provides access to all its properties.
  - The OutputEmitterRef's subscribe method allows to define an event handler.
  - Even though the OutputEmitterRef provides a subscribe method, it is not an Observable. However, the original EventEmitter used together with the @Output decorator was. To get back all the possibilities associated with Observable-based outputs, you can use the function outputToObservable that is part of the RxJS interop layer.
  - The function outputToObservable converts an OutputEmitterRef to an Observable.
  - The Observable returned by outputToObservable completes when Angular destroys the the output's component. For this reason, there is no need to unsubscribe by hand.
* **Technical Entities (Classes/Functions/APIs):** `ComponentRef`, `setInput()`, `instance`, `outputToObservable()`, `EventEmitter`
* **Code Snippet:**
```typescript
import { outputToObservable } from '@angular/core/rxjs-interop';
[...]

@Component([...])
export class ToastDemoComponent {
  counter = 0;
  placeholder = viewChild.required('placeholder', { read: ViewContainerRef });

  show() {
    const ref = this.placeholder()?.createComponent(ToastComponent);
    this.counter++;
    const title = 'Message #' + this.counter;
    ref.setInput('label', title);

    const confirmed$ = outputToObservable(ref.instance.confirmed)
      .pipe(map(title => ({ trigger: 'confirmed', title })));

    const timer$ = timer(5000);
      .pipe(map(() => ({ trigger: 'timeout', title })));

    race(confirmed$, timer$).subscribe(action => {
      ref?.destroy();
      console.log('action', action);
    });

  }

}
```

## Conclusion
* **Key Points:**
  - Several new functions replace property decorators and help to set up data binding concepts. These functions are picked up by the Angular compiler emitting respective code.
  - The function input defines Inputs for property bindings, model defines Inputs for Two Way Data Binding, and contentChild(ren) and viewChild(ren) take care of Content and View Queries. Using these functions results in Signals that can be projected with computed and used within effects.