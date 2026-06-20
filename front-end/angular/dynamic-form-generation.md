---
aliases:
  - Dynamic Form Generation
highlights: Schema driven forms from JSON/API config; decouples form structure from templates; critical for configurable enterprise UIs
Source 1: https://danywalls.com/creating-dynamic-forms-in-angular-a-step-by-step-guide
Source 2: https://www.angulararchitects.io/en/blog/dynamic-forms-building-a-form-generator-with-signal-forms/
---
# Creating Dynamic Forms in Angular: A Step-by-Step Guide

## Scenario
* **Key Points:**
  - We work for the marketing team, which wants a form to request the users' Firstname, Lastname, and age. Let's build it.
* **Technical Entities (Classes/Functions/APIs):** `FormGroup`, `FormControl`, `[formGroup]`, `formControlName`
* **Code Snippet:**
```typescript
import { Component, OnInit, VERSION } from '@angular/core';
import { FormControl, FormGroup } from '@angular/forms';
 
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent implements OnInit {
  registerForm: FormGroup;
 
  ngOnInit() {
    this.buildForm();
  }
 
  buildForm() {
    this.registerForm = new FormGroup({
      name: new FormControl(''),
      lastName: new FormControl(''),
      age: new FormControl(''),
    });
  }
}
```

## Create Dynamic FormControl and Inputs
* **Key Points:**
  - First, we will have two tasks to build our form dynamic. Create the Form Group from the business object. Show the list of fields to render in the form.
* **Technical Entities (Classes/Functions/APIs):** `FormGroup`, `FormControl`, `*ngFor`, `formControlName`
* **Code Snippet:**
```typescript
 getFormControlsFields() {
        const formGroupFields = {};
        for (const field of Object.keys(this.model)) {
            formGroupFields[field] = new FormControl("");
            this.fields.push(field);
        }
        return formGroupFields;
    }
```

## Separate the Form Process and FieldType
* **Key Points:**
  - The app.component does a few tasks, creating the model, form, and rendering the input. Let's clean it up a little bit.
* **Technical Entities (Classes/Functions/APIs):** `@Input()`, `FormGroup`

## Show Inputs By Type
* **Key Points:**
  - The Dynamic form renders a single type of input. In a real-world scenario, we need more types like date, select, input, radio, and checkbox. The information about the control types must come from the model to the dynamic-field.
* **Technical Entities (Classes/Functions/APIs):** `type`, `value`, `label`

## Add Selects, Radios, and Checkbox
* **Key Points:**
  - The dynamic field component shows the input, but adding controls like select, radio, or checkbox makes it a bit complex. I want to split each control into specific controls.
* **Technical Entities (Classes/Functions/APIs):** `ngSwitch`, `formControlName`, `ngFor`, `options`

## Show Dynamic Components And Update Model
* **Key Points:**
  - We have components for each control type, but the dynamic-field.component is a bridge between them. It picks the specific component by type. Using the ngSwitch directive, we determine the control matching with the component type.
* **Technical Entities (Classes/Functions/APIs):** `ngSwitch`, `ngSwitchCase`

## Validations
* **Key Points:**
  - We need a complete form with validations. I want to make this article brief, but validation is essential in the forms.
  - The validators are part of the form controls. We process the rule to set the validator for the formControl in a new method, addValidators, and the return value stored in the validators variable to assign in the formControl.
* **Technical Entities (Classes/Functions/APIs):** `Validators.required`, `invalid`, `dirty`, `touched`

## Refactor Time
### Propagation of FormGroup
* **Key Points:**
  - After @Juan Berzosa Tejero take time to review the article, he asked me about the propagation of FormGroup using the @Input() with formName, and it starts to make noise. Luckily I found the directive FormGroupDirective in the Angular Documentation. It helps us to bind an existing FormGroup or FormRecord to a DOM element.
* **Technical Entities (Classes/Functions/APIs):** `FormGroupDirective`, `ngOnInit`

### Remove the ngSwitch
* **Key Points:**
  - Yesterday, @Maxime Lyakhov, leave a message about the ngSwich. He was right about the ngSwitch in the HTML; it is difficult to maintain. My first idea is to load the specific component dynamically using ViewChild and ViewContainerRef and set the input variables with the setInput() method.
* **Technical Entities (Classes/Functions/APIs):** `ViewChild`, `ViewContainerRef`, `setInput()`, `createComponent`, `ngAfterViewInit`, `ChangeDetection`

### Trigger Event onChange or onBlur
* **Key Points:**
  - @Rakesh Prakash asked how to trigger events on changes or blur events attached to the input fields, and the first idea came to my head. by default, the form updates the values on every keystroke, triggers the validator and the update values, and may not always be desirable. Sometimes we want to have control over the moment value updates and validators, but Angular helps us with the updateOn in Angular Forms. The updateOn set the update strategy of our form controls and which DOM event triggers updates. The options for updateOn in the FormControl supported are 'change' | 'blur' | 'submit'.
* **Technical Entities (Classes/Functions/APIs):** `updateOn`, `change`, `blur`, `submit`

### Read Values
* **Key Points:**
  - @Fabian asked me how to read the values from the dynamic form. We should read a single value or all dynamics properties in the model. For a single value, use the field name and the dynamicForm.get method.
* **Technical Entities (Classes/Functions/APIs):** `get()`, `Object.keys()`, `JSON.stringify`

### React To Linked Components
* **Key Points:**
  - @Mohammed Tabbakh is inquiring about how to bind components that have data dependencies on other components, such as: We have two dropdown menus, one for selecting a country from a list of countries and another for selecting a city. In order to select a city, the user must first select the corresponding country from the first dropdown menu.
* **Technical Entities (Classes/Functions/APIs):** `Subject`, `takeWhile`, `filter`, `ngAfterViewInit`, `ngOnDestroy`

## Recap
* **Key Points:**
  - We learned how to add dynamic fields from a structure and generate inputs like select, checkbox, radio, or inputs a few times. The model may be an API response from the backend. If we wish to include a new field, it should be added to the API response. Additionally, you are welcome to incorporate more varieties in the dynamic-field component. To enhance the code, we could employ interfaces for every component type such as dropdown, checkbox, or even the form itself. Additionally, we could craft helper functions that generate the bulk of the boilerplate code for dropdowns.

## Frequently Asked Questions
* **Key Points:**
  - How do I build a dynamic form in Angular using reactive forms? To build a dynamic form in Angular, define a model that describes each field including its type, label, validators, and any control-specific options. Then iterate over that model in your component to programmatically create FormControl instances and add them to a FormGroup. In the template, use ngSwitch or a mapping strategy to render the appropriate input element for each field type.
  - What is the benefit of using dynamic forms over hardcoded forms in Angular? Dynamic forms make your application more flexible and easier to maintain because form structure is driven by data rather than static HTML. When business requirements change, such as adding a new field or modifying field types, you only need to update the configuration model rather than modifying both the component code and the template in multiple places.
  - How do I support different input types like text, radio buttons, and checkboxes in an Angular dynamic form? In the template, use a structural directive like ngSwitch on a field type property from your model. Each case in the switch renders a different input element such as an input for text, a group of radio buttons, or a group of checkboxes. All of these are connected to the same FormControl or FormGroup using standard reactive form directives like formControlName.
  - How do I add validators dynamically to form controls in Angular? You can pass an array of validators as the second argument when creating a FormControl, or use the setValidators method on an existing FormControl to add them programmatically. If your field model includes a validators property, you can read it during form construction and apply the appropriate Angular built-in validators like Validators.required or Validators.minLength.