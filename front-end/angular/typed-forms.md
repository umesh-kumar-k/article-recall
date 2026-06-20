---
aliases:
  - Typed Forms (v14+)
highlights: FormControl<string|null> with full generic inference; eliminates any leakage from form values - essential for large codebase
Source 1: https://angular.love/typed-forms-2
Source 2: https://angular.love/strongly-typed-reactive-forms-in-angular
---
# Strongly Typed Reactive Forms in Angular

## Introduction
* **Key Points:**
  - In version 14 of Angular a new feature was added to Reactive Forms, and that was type-safe forms. Prior to this addition, you could more easily introduce runtime errors in your reactive forms, as you could access values and controls on a form that didn't exist. You could also set the value of a control to whatever value you wanted; for example you could set the value of an email control to a number, even though you would never want that to happen.
  - With type safety being introduced, these runtime pitfalls are largely eliminated. You will be warned if you try to access an attribute that does not exist, or try to set the value of a field to an unsupported type.

## Creating Typed Forms
* **Key Points:**
  - The first step in using typed forms is creating a typed form. There are multiple methods of doing this: creating aFormControl, creating a FormGroup, and using the FormBuilder.
  - You can also explicitly set the value of the field when creating the control.
  - This type safety will help you avoid potential errors in your application, such as setting the value of an attribute to an unsupported type or patching the form with unsupported values. You should know exactly what is on the value of the form at any given time.
  - The same type safety is given on this form as in the FormGroup section above, so I won't cover it again here.
  - This form of typing works great, as it will alert you if you haven't implemented one of the attributes, and helps to make sure you give the attributes the right value.
  - Generally speaking, the explicit typing with the InfoForm interface is extra and not needed. It doesn't give you more type safety than not providing the interface.
* **Technical Entities (Classes/Functions/APIs):** `FormControl`, `FormGroup`, `FormBuilder`, `nonNullable`

### FormControl
* **Key Points:**
  - You can create a FormControl simply by using the new keyword and passing an initial value to the FormControl constructor.
* **Code Snippet:**
```typescript
public name = new FormControl('')
```

### FormGroup
* **Key Points:**
  - You can also create typed reactive forms by creating a new FormGroup. It is similar to creating a simpleFormControl.
  - The name control of this form will have the same type safety that theFormControl above has.
  - In addition to the type safety of the name control, you will get an error if you try to set the value of a field that doesn't exist on the FormGroup declaration.
* **Code Snippet:**
```typescript
public form = new FormGroup({
  name: new FormControl(''),
});
```

### FormBuilder
* **Key Points:**
  - TheFormBuilder is a third way you can create typed forms. It is very similar to creating a new FormGroup, but creates the controls for you instead of you explicitly creating the controls.
  - You can also type the form by providing an interface as a generic for theFormBuilder's group method.
  - If you want to add validators to the attributes, though, the above interface will produce errors, because you explicitly said the value of "name" would be a string, but you'll be setting its value to an array.
* **Code Snippet:**
```typescript
public form = this._fb.group({
  name: [''],
});
```

## FormControl Values
* **Key Points:**
  - I mentioned above that the value of a FormControl (whether created with the new keyword, or with a FormGroup or theFormBuilder) can also be null. By default, all the values are either the type that you provide as the initial value or null or undefined.
  - First, values can be undefined because the control can be disabled, and that means that the value of that control will not be included on the form's value. You can get the raw value of the form, withgetRawValue(), and you'll not run into any undefined attributes.
  - Next, the value can be null because that's just the default behavior of typed forms. The value can be null unless explicitly stated otherwise.
  - If you reset the control without providing a new value for the control, its reset value will be null.
  - If you declare theFormGroup or FormControl in any of the above ways, the value of the controls can not be null any longer. undefined is still fair game, as is the type of the initial value, but null will no longer be valid.
* **Technical Entities (Classes/Functions/APIs):** `getRawValue()`, `nonNullable`, `NonNullableFormBuilder`

## Benefits of Type Safe Forms
* **Key Points:**
  - There are too many benefits of type safe forms to reasonably name in an article, but here are some that come to mind that you are most likely to run into on a day-to-day basis.
  - The first is autocomplete in your IDE. We all rely heavily on our IDEs to get our daily work done. It allows us to not have to memorize every attribute of an object or every method in a class.
  - You get the same benefit of autocompletion when trying to access the controls on the form, perhaps to use the valueChanges observable on an individual control instead of the whole form.
  - The last thing I want to point out is similar, but relates to patching or setting the value of a form in your Typescript file. If you try and add an invalid attribute to the object, the app fails to compile and your IDE will warn you.
  - With untyped forms, you would be able to provide that value to thepatchValue method, and may expect to have an emailAddress on the form. This level of help that comes automatically from the framework and your IDE is extremely beneficial. Anything I can do to limit the amount of mistakes I make is welcome.
* **Technical Entities (Classes/Functions/APIs):** `valueChanges`, `patchValue`, `setValue`

## Common Gotchas with Typed Forms
* **Key Points:**
  - With all the good things that come from typed forms, there are a couple areas where you may run into a couple issues. These are things where the framework is working as expected, but you might feel like it's getting in your way a little bit.

### Initial Value as null
* **Key Points:**
  - The first thing, which I was running into a lot, is related to the value of a control when trying to update withsetValue or patchValue.
  - In this form, the beginning value of age is null, and it is a required field. I'll do this sometimes to ensure that the user explicitly selects an age. 0 could be a valid age, so setting it to 0 to begin with isn't an option. The issue here is that the only valid values for age are now null and undefined. If I try and manually patch the form, or interact with the value of age at some point, the type of the value will be null or undefined.
  - To overcome this, you will need to set the initial value of the control to a number, like -1. Then add another validator to the control, the min value validator, and set its value to 0. That way they still have to explicitly change the age, and they can select 0 as an option. This works, but it really works best if the age control is a select element in the HTML. You could set the default option's value to -1, and then they can choose their age after that. I don't love this option because it feels kind of hacky, but it would work.
  - There is a way to initialize the form correctly, though, that doesn't require any hackery. It involves explicitly declaring a control and its types for the age control.
  - In that same vein, we could make a single field non-nullable if we needed instead of only making the form as a whole non-nullable.
* **Technical Entities (Classes/Functions/APIs):** `setValue`, `patchValue`, `Validators.required`, `min`

### FormGroup Typed as any
* **Key Points:**
  - The next gotcha relates to where we initialize theFormGroup.
  - While this is valid, it does ruin our type safety. That's because the declaration of the form variable sets the type toFormGroup, which defaults to FormGroup<any>. Thus, we will get no autocomplete or type helping throughout the component.
  - You can fix the typing of the form in one of two ways:
    - initialize the form at the time of declaration; don't initialize the value in ngOnInit or even the constructor
    - Give a more detailed, explicit type for theFormGroup when declaring the variable
* **Technical Entities (Classes/Functions/APIs):** `FormGroup<any>`, `ngOnInit`

## Conclusion
* **Key Points:**
  - Having your forms typed is a much better experience in Angular than untyped forms. Perhaps my favorite thing about them is that there is very little to do to gain the benefits on the part of the developer. You basically have to go out of your way to lose the type safety. There are a lot of things we have to do in our apps to have them be type safe, but this is not one of those things.
  - Also, as I mentioned previously, it's very nice to be warned in advance if your form is not valid. It's never fun to deploy your application and then have a bug come up that requires you to diagnose the issue while it's in production and get a fix out. Knowing at compile time is always better, and the Angular team has provided us with that benefit.