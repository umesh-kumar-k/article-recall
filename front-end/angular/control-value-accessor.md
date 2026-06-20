---
aliases:
  - ControlValueAccessor
highlights: Interface to integrate custom components with Angular's form system; the correct abstraction for design system input components
Source 1: https://javascript.plainenglish.io/controlvalueaccessor-is-dead-long-live-formvaluecontrol-4cf2e30a4fb0
---
# ControlValueAccessor Is Dead. Long Live FormValueControl! 🎉

## The Horror Show That Was ControlValueAccessor
* **Key Points:**
  - For years, ControlValueAccessor has been Angular's answer to custom form controls. And it worked! Kind of like how a Rube Goldberg machine "works"—technically functional, but unnecessarily complicated. You needed four callback methods, a provider configuration that looked like ancient hieroglyphics, and the kind of deep framework knowledge that only comes from years of suffering.
* **Technical Entities (Classes/Functions/APIs):** `ControlValueAccessor`, `NG_VALUE_ACCESSOR`, `forwardRef`, `writeValue()`, `registerOnChange()`, `registerOnTouched()`, `setDisabledState()`

## Enter Signal Forms: The Hero We Deserve
* **Key Points:**
  - Angular 21's Signal Forms looked at this mess and said, "What if we just… didn't?"
  - That's ONE property. One model() input. No callbacks. No providers. No forwardRef. No existential crisis at 2 AM. Just a signal that automatically keeps your component and form model in perfect harmony.
  - When you update value in your component, the form knows. When the form updates the value, your component knows. It's bidirectional, automatic synchronization that just works.
* **Technical Entities (Classes/Functions/APIs):** `Signal Forms`, `model()`, `FormValueControl`
* **Code Snippet:**
```typescript
import { Component, model } from '@angular/core';
import { FormValueControl } from '@angular/forms';
@Component({
  selector: 'app-star-rating',
  template: `
    <div class="star-container">
      @for (star of [1, 2, 3, 4, 5]; track star) {
        <span 
          class="star"
          (click)="selectStar(star)">
          {{ star <= value() ? '⭐' : '☆' }}
        </span>
      }
    </div>
  `
})
export class StarRatingComponent implements FormValueControl<number> {
  value = model<number>(0); // 👈 That's it. That's the whole thing.
  selectStar(rating: number): void {
    this.value.set(rating); // Automatic sync. Like magic, but real.
  }
}
```

## Handling Disabled State (The Easy Way)
* **Key Points:**
  - When your form disables the control, the disabled signal updates automatically. Your UI reacts. Users can't interact with it. No manual callback management. No forgetting to check the disabled state.
* **Technical Entities (Classes/Functions/APIs):** `input()`, `disabled`
* **Code Snippet:**
```typescript
import { Component, input, model } from '@angular/core';
import { FormValueControl } from '@angular/forms';
@Component({
  selector: 'app-reaction-picker',
  template: `
    <div [class.disabled]="disabled()">
      @for (emoji of reactions; track emoji) {
        <button 
          (click)="select(emoji)"
          [disabled]="disabled()">
          {{ emoji }}
        </button>
      }
    </div>
  `
})
export class ReactionPickerComponent implements FormValueControl<string> {
  value = model<string>('');
  disabled = input<boolean>(false); // 👈 One line. Automatic sync.
  
  reactions = ['😀', '😍', '🤔', '😢', '😡'];
  select(emoji: string): void {
    this.value.set(emoji);
  }
}
```

## Adding Required Indicators (Because Why Not?)
* **Key Points:**
  - Add a required validator to your form control? The required signal updates automatically. Remove the validator? Signal updates again. You're not calling anything. You're not registering anything. You're just... building features.
* **Technical Entities (Classes/Functions/APIs):** `required`

## Using Your Custom Control (The Moment of Truth)
* **Key Points:**
  - Notice how the custom control works exactly like a built-in control? That's the point. No special incantations. No mysterious provider configurations. Just bind it with the control directive and you're done.
* **Technical Entities (Classes/Functions/APIs):** `[control]`

## Special Cases: Checkbox Controls
* **Key Points:**
  - Got a toggle switch or a fancy checkbox replacement? Signal Forms have you covered with FormCheckboxControl. The only difference? Use checked instead of value. That's literally it.
* **Technical Entities (Classes/Functions/APIs):** `FormCheckboxControl`

## The Technical Magic (For the Nerds Like Us)
* **Key Points:**
  - Why does this work so much better than ControlValueAccessor? Two words: fine-grained reactivity. Signals provide automatic dependency tracking. When your signal changes, Angular knows exactly what needs to update. No zone.js running change detection on your entire component tree. No manual callback invocations that you might forget. Just precise, surgical updates exactly where they're needed.
  - Signal Forms eliminate an entire category of bugs by removing the manual synchronization layer. You can't forget to call a callback that doesn't exist.
* **Technical Entities (Classes/Functions/APIs):** `Signals`, `zone.js`, `change detection`

## "But What About My ControlValueAccessor Knowledge?"
* **Key Points:**
  - Don't panic. ControlValueAccessor isn't going anywhere. Template-driven and reactive forms will continue to exist alongside Signal Forms. Your existing apps won't spontaneously combust. In fact, Signal Forms use ControlValueAccessor under the hood for interoperability. It's abstraction layers all the way down, but now we get to work at a higher, saner level.
* **Technical Entities (Classes/Functions/APIs):** `ControlValueAccessor`

## The Future Is Signal-Based (And We're Here For It)
* **Key Points:**
  - Signal Forms are currently experimental, which in Angular-speak means "the API might change, so don't bet your production app on it just yet". But the core concepts are solid, and this is clearly where Angular is headed. As Angular moves away from zone-based change detection toward a signal-based reactive architecture, forms needed to evolve too.

## The Bottom Line
* **Key Points:**
  - Creating custom form controls in Angular went from: Four callback methods → One property; Complex provider config → Nothing; Manual synchronization → Automatic; Limited state access → Rich form state; Error-prone → Robust by default; 2 AM debugging sessions → Actually sleeping
  - Is ControlValueAccessor dead? Not technically. But is it the walking dead that we're all slowly backing away from while FormValueControl becomes the new hotness? Absolutely. Welcome to the future of Angular forms. It's simpler, more intuitive, and finally feels like the framework respects your time.