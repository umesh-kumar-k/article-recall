---
aliases:
  - Cross-Field Validation
highlights: Validator at FormGroup level, not FormControl; access sibling control values; correct architecture for password confirm, date range validation
Source 1: https://angular.love/the-best-way-to-implement-custom-validators
---
# The best way to implement custom validators

## Angular built-in validators
* **Key Points:**
  - Angular provides some handy built-in validators which can be used for reactive forms and template-driven forms. Most known validators are required, requiredTrue, min, max, minLength, maxLength and pattern.
  - Those validators are very helpful because they allow us to perform standard form validation. But as soon as we need validation for our particular use case, we may want to provide our custom validator.
* **Technical Entities (Classes/Functions/APIs):** `required`, `requiredTrue`, `min`, `max`, `minLength`, `maxLength`, `pattern`, `Validators`

## Implementing a custom validator
* **Key Points:**
  - The validator itself is just a function that accepts an AbstractControl and returns an Object containing the validation error or null if everything is valid.
  - To make our custom validator accessible for template-driven forms, we need to implement our validator as a directive and provide it as NG_VALIDATORS.
* **Technical Entities (Classes/Functions/APIs):** `AbstractControl`, `ValidatorFn`, `NG_VALIDATORS`, `Validator`, `Directive`, `@Directive`
* **Code Snippet:**
```typescript
import {AbstractControl, ValidatorFn} from '@angular/forms';

export function blue(): ValidatorFn {  
    return (control: AbstractControl): { [key: string]: any } | null =>  
        control.value?.toLowerCase() === 'blue' 
            ? null : {wrongColor: control.value};
}
```

## Reverse engineer Angular validators
* **Key Points:**
  - Angular has a class that implements all validators as static methods. With this approach, they are "grouped" and accessible over the Validators class.
  - Furthermore, we can recognize a consistent pattern. The validators are "only callable if configurable", They return a ValidatorFn for configurable validators and an error object or null for non-configurable validators.
  - The Directive uses a specific selector. Therefore it only works on individual form controls. The interesting part lies in the validate function. The Directive reuses the required function from the static Validators class.
* **Technical Entities (Classes/Functions/APIs):** `Validators`, `ValidatorFn`, `Directive`, `NG_VALIDATORS`

## Let's adapt what we have learned to our custom validator
* **Key Points:**
  - Instead of implementing a standalone validation function, we are going to add it as a static field inside a ColorValidator class.
  - This refactoring allows us to use Intellisense and access the blue validator over ColorValidators. Furthermore we know that our validation function is not configurable and therefore we don't need to call it.
* **Technical Entities (Classes/Functions/APIs):** `ColorValidators`, `ValidatorFn`, `AbstractControl`
* **Code Snippet:**
```typescript
import {AbstractControl, ValidatorFn} from '@angular/forms';

export class ColorValidators {  

static blue(control: AbstractControl): any | null {  
    return ColorValidators.color('blue')(control);  
}  

static red(control: AbstractControl): any | null {
    return ColorValidators.color('red')(control);  
}  

static white(control: AbstractControl): any | null {
    return ColorValidators.color('white')(control);  
}  

static color(colorName: string): ValidatorFn {
    return (control: AbstractControl): { [key: string]: any } | null => 
        control.value?.toLowerCase() === colorName 
        ? null : {wrongColor: control.value};
    }
}
```

## Provide grouped validators for template-driven forms
* **Key Points:**
  - Currently, our grouped validator can not yet be used in template-driven forms. We need to provide a directive and then call ColorValidators inside of it.
  - The usage of the validator inside a template-driven form doesn't change.
* **Technical Entities (Classes/Functions/APIs):** `NG_VALIDATORS`, `Validator`, `Directive`, `ColorValidators`

## Conclusion
* **Key Points:**
  - In the end, a custom validator is just a function that returns either an error object or null. If a validator is customizable, it needs to be wrapped with a function.
  - Most tutorials teach you to implement a validator as a factory function which is totally valid. However, the usage is not as nice as the built-in validators from Angular. We have no Intellisense, and if you don't follow a certain convention its not clear if a Validator needs to be called or not.
  - By reverse-engineering the Angular source code on validators we found an approach that shows us how to group validators. Implementing the validators as static class fields instead of standalone functions allows us to group our Validators and improve Intellisense. Furthermore, we can follow the "callable if configurable" convention. With this small refactoring we can improve developer experience.