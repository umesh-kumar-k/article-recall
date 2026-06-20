---
aliases:
  - Signals Architecture
highlights: Fine grained reactivity primitive replacing zone triggered change detection;signal() , computed(), effect() from the reactive graph; direct replacement for most RxJS state patterns
Source 1: https://medium.com/angularwave/introduction-to-angular-signals-e20dba5737db
Source 2: https://www.angulararchitects.io/blog/angular-signals/
Source 3: https://blog.appsignal.com/2025/09/17/the-angular-signals-revolution-rethinking-reactivity.html
Source 4: https://medium.com/@eugeniyoz/angular-signals-best-practices-9ac837ab1cec
Source 5: https://www.angulararchitects.io/blog/angular-signals-your-architecture-5-options/
---
[Signals](signals/building-blocks.md)  [Signals Architecture Rules](signals/architecture-rules.md) [Signals Component Communication](signals/component-communication.md) [Signal Options](signals/options.md) [Resource API](signals/resource-api.md) 

# Angular Signals: Best Practices

## When to use Signals?
* **Key Points:**
  - In Angular templates, Signals are better than Observables: they schedule Change Detection without any pipes, they are glitch-free, and you can read the same signal multiple times and it will be "free" in terms of performance (and read values are guaranteed to be the same). There are other reasons that are not so easy to explain briefly, but that's already enough to make a rule: every variable (that might change) in your new templates should be a Signal.
  - Outside of templates, Signals also can be used for reactivity, but, as I mentioned, without a time aspect.
  - There are two ways to create reactive variables in Angular: Observables and Signals. If you describe in words, how your variable should express its reactivity, you'll see what you need to use.
  - If the role of a variable can be described as conditions, then you need a Signal:
    - "if this variable has this value, then display this list"
    - "if this variable has this value, this button is disabled"
  - If the description of a variable's role includes words, related to time, you need an Observable:
    - "when the cursor moves…"
    - "wait for the file uploading event and then…"
    - "every time this event happens, do this…"
    - "until this event…"
    - "for N seconds ignore…"
    - "after this request…"
  - Signals have no time axis, and they can not delay a value — they always have a value, and their consumers should be always able to read it.
  - Consumers of Signals, computed(), effect(), and templates do not guarantee that they will read every new value written to the Signals they watch. An updated Signal will be eventually read, not instantly after the update as it happens with Observables. Consumers decide when they will read the new value using their scheduling mechanisms. It might be "in the next task," "during the next Change Detection cycle," or at some other moment, up to the consumer.
* **Technical Entities (Classes/Functions/APIs):** `Signals`, `Observables`, `computed()`, `effect()`

## When to use computed()?
* **Key Points:**
  - Whenever you like!
  - computed() is the best thing in Angular Signals, incredibly handy and safe to use. Using computed(), you'll make your code more declarative.
  - There are just two rules about the usage of computed():
    - Do not modify things in computed(). It should compute a new result, that's it. Do not modify the DOM, do not mutate variables using this, and do not call functions that might do that. Do not push values to Observables — it will cause unintentional reactive context propagation (explained below for effect()). computed() should not have side effects, it should be a pure function.
    - Do not make asynchronous calls in computed(). This function does not allow modification of Signals (and it is amazingly helpful), but it can not track asynchronous code. Moreover, Angular Signals are strictly synchronous, so if you want to use asynchronous code in computed(), you are doing something wrong. So, no setTimeout(), no Promises, no other asynchronous things.

## When to use effect()?
* **Key Points:**
  - Angular docs say that you'll rarely need effect() and discourage you from using it. And that info is correct: you rarely need effect()… if your code is declarative ;)
  - The more imperative your code, the more often you'll need effect(). There is no code without imperative parts, but we all should try to make our code as declarative as possible, so we do need to use effect() as rarely as possible.
  - Besides the dangers, mentioned by Angular docs (infinite loops, change detection errors), there is another thing, that might be quite nasty: effects are executed in a reactive context, and any code you call in effect, will be executed in a reactive context. If that code reads some signals, they will be added as dependencies to your effect.
  - I still don't want to encourage you to use effect(), but I'll give you advice on how to use it as safely as possible:
    - The function you provide to effect() should be as small as possible. This way it will be easier to read and spot erroneous behavior.
    - Read signals first, then wrap the rest of the effect into untracked()
  - More information about cases where untracked() helps can be found in this article.
* **Technical Entities (Classes/Functions/APIs):** `effect()`, `untracked()`

## Mixing Signals and Observables
* **Key Points:**
  - …is ok!
  - Your code will have Signals and Observables, at least because Signals can not be used for every kind of reactivity. It is not an issue, it is perfectly fine.
  - If you need some value from an Observable in your computed(), create a Signal using toSignal() (outside of computed()).
  - If you need to read a Signal in Observable's pipe(), there are two ways:
    - If you need to react to changes in that Signal in your Observable, then you need to convert that Signal into Observable and add it using some join operator.
    - If you are sure you just need the current value of a Signal and you don't need to react to its changes — you can read a Signal directly in your operators or subscribe(). Observable is not a reactive context, so you don't need untracked() here.
* **Technical Entities (Classes/Functions/APIs):** `toSignal()`