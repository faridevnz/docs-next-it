# Template Syntax

Vue.js usa una sintassi di templating basata su HTML che ti permette di fare il binding del DOM ai dati dell'istanza del corrispettivo componente in modo dichiarativo. Tutti i template Vue.js sono HTML validi che possono essere renderizzati dai browser compatibili con la spec e dai parser HTML. 

Sotto il cofano, Vue compila i template in render function per il virtual DOM. Insieme al sistema di reattività, Vue è capace di capire in modo intelligente qual è il numero minimo di componenti da re-renderizzare e capace di applicare il numero minimo di manipolazioni al DOM quando lo stato dell'applicazione cambia. 

Se hai familiartià con i concetti relativi al virtual DOM, e preferisci la pura espressività di JavaScript, puoi anche [scrivere direttamente le render functions](render-function.html) invece dei template, con supporto opzionale a JSX. 

## Interpolazione

### Testi

La forma più semplice di binding dei dati è l'interpolazione del testo usando la sintassi "Mustache" (doppia parentesi graffa):

```html
<span>Messaggio: {{ msg }}</span>
```

Il tag mustache verrà rimpiazzato con il valore della proprietà `msg` della corrispettiva istanza del componente. Sarà anche aggiornata ogni volta che la proprietà `msg` cambierà. 

Puoi anche fare un'interpolazione one-time che non si aggiorna quando i dati cambiano usando la [direttiva v-once](../api/directives.html#v-once), ma tieni a mente che questo avrà effetto su qualsiasi altro binding nello stesso nodo.

```html
<span v-once>Questo non cambierà mai: {{ msg }}</span>
```

### Raw HTML

Il doppio mustaches interpreta i dati come semplice testo, non HTML. Per avere in output del vero HTML, dovrai usare la [direttiva `v-html`](../api/directives.html#v-html):

```html
<p>Usando i mustaches: {{ rawHtml }}</p>
<p>Usando la direttiva v-html: <span v-html="rawHtml"></span></p>
```

<common-codepen-snippet title="Rendering v-html" slug="yLNEJJM" :preview="false" />

Il contenuto dello `span` sarà rimpiazziato con il valore della proprietà `rawHTML`, interpretata come HTML - i data binding sono ignorati. Nota che non puoi usare `v-html` per comporre parti del template, perchè Vue non è un template engine basato su stringhe. Invece, i componenti sono preferiti come unità fondamentale per la composizione e il riuso delle UI. 

::: tip
Renderizzare dinamicamente HTML arbitrario sul tuo sito può essere pericoloso perchè può facilmente portare a [vulnerabilità XSS](https://en.wikipedia.org/wiki/Cross-site_scripting). Usa l'interpolazione HTML solo su componenti trusted e **mai** su contenuti forniti dagli utenti. 
:::

### Attributi

Mustaches cannot be used inside HTML attributes. Instead, use a [`v-bind` directive](../api/directives.html#v-bind):
I mustaches non possono essere usati dentro gli attributi HTML. Invece, use la [ direttiva `v-bind`](../api/directives.html#v-bind):

```html
<div v-bind:id="dynamicId"></div>
```

Se il valore collegato è `null` o `undefined` allora l'attributi non sarà incluso nell'elemento renderizzato.

In caso di un attributi booleano, dove la sua mera esistenza implica `true`, `v-bind` funziona in modo pò diverso. Per esempio:

```html
<button v-bind:disabled="isButtonDisabled">Button</button>
```

The `disabled` attribute will be included if `isButtonDisabled` has a truthy value. It will also be included if the value is an empty string, maintaining consistency with `<button disabled="">`. For other falsy values the attribute will be omitted.

### Using JavaScript Expressions

So far we've only been binding to simple property keys in our templates. But Vue.js actually supports the full power of JavaScript expressions inside all data bindings:

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

These expressions will be evaluated as JavaScript in the data scope of the current active instance. One restriction is that each binding can only contain **one single expression**, so the following will **NOT** work:

```html
<!-- this is a statement, not an expression: -->
{{ var a = 1 }}

<!-- flow control won't work either, use ternary expressions -->
{{ if (ok) { return message } }}
```

## Directives

Directives are special attributes with the `v-` prefix. Directive attribute values are expected to be **a single JavaScript expression** (with the exception of `v-for` and `v-on`, which will be discussed later). A directive's job is to reactively apply side effects to the DOM when the value of its expression changes. Let's review the example we saw in the introduction:

```html
<p v-if="seen">Now you see me</p>
```

Here, the `v-if` directive would remove/insert the `<p>` element based on the truthiness of the value of the expression `seen`.

### Arguments

Some directives can take an "argument", denoted by a colon after the directive name. For example, the `v-bind` directive is used to reactively update an HTML attribute:

```html
<a v-bind:href="url"> ... </a>
```

Here `href` is the argument, which tells the `v-bind` directive to bind the element's `href` attribute to the value of the expression `url`.

Another example is the `v-on` directive, which listens to DOM events:

```html
<a v-on:click="doSomething"> ... </a>
```

Here the argument is the event name to listen to. We will talk about event handling in more detail too.

### Dynamic Arguments

It is also possible to use a JavaScript expression in a directive argument by wrapping it with square brackets:

```html
<!--
Note that there are some constraints to the argument expression, as explained
in the "Dynamic Argument Expression Constraints" section below.
-->
<a v-bind:[attributeName]="url"> ... </a>
```

Here `attributeName` will be dynamically evaluated as a JavaScript expression, and its evaluated value will be used as the final value for the argument. For example, if your component instance has a data property, `attributeName`, whose value is `"href"`, then this binding will be equivalent to `v-bind:href`.

Similarly, you can use dynamic arguments to bind a handler to a dynamic event name:

```html
<a v-on:[eventName]="doSomething"> ... </a>
```

In this example, when `eventName`'s value is `"focus"`, `v-on:[eventName]` will be equivalent to `v-on:focus`.

### Modifiers

Modifiers are special postfixes denoted by a dot, which indicate that a directive should be bound in some special way. For example, the `.prevent` modifier tells the `v-on` directive to call `event.preventDefault()` on the triggered event:

```html
<form v-on:submit.prevent="onSubmit">...</form>
```

You'll see other examples of modifiers later, [for `v-on`](events.md#event-modifiers) and [for `v-model`](forms.md#modifiers), when we explore those features.

## Shorthands

The `v-` prefix serves as a visual cue for identifying Vue-specific attributes in your templates. This is useful when you are using Vue.js to apply dynamic behavior to some existing markup, but can feel verbose for some frequently used directives. At the same time, the need for the `v-` prefix becomes less important when you are building a [SPA](https://en.wikipedia.org/wiki/Single-page_application), where Vue manages every template. Therefore, Vue provides special shorthands for two of the most often used directives, `v-bind` and `v-on`:

### `v-bind` Shorthand

```html
<!-- full syntax -->
<a v-bind:href="url"> ... </a>

<!-- shorthand -->
<a :href="url"> ... </a>

<!-- shorthand with dynamic argument -->
<a :[key]="url"> ... </a>
```

### `v-on` Shorthand

```html
<!-- full syntax -->
<a v-on:click="doSomething"> ... </a>

<!-- shorthand -->
<a @click="doSomething"> ... </a>

<!-- shorthand with dynamic argument -->
<a @[event]="doSomething"> ... </a>
```

They may look a bit different from normal HTML, but `:` and `@` are valid characters for attribute names and all Vue-supported browsers can parse it correctly. In addition, they do not appear in the final rendered markup. The shorthand syntax is totally optional, but you will likely appreciate it when you learn more about its usage later.

> From the next page on, we'll use the shorthand in our examples, as that's the most common usage for Vue developers.

### Caveats

#### Dynamic Argument Value Constraints

Dynamic arguments are expected to evaluate to a string, with the exception of `null`. The special value `null` can be used to explicitly remove the binding. Any other non-string value will trigger a warning.

#### Dynamic Argument Expression Constraints

Dynamic argument expressions have some syntax constraints because certain characters, such as spaces and quotes, are invalid inside HTML attribute names. For example, the following is invalid:

```html
<!-- This will trigger a compiler warning. -->
<a v-bind:['foo' + bar]="value"> ... </a>
```

We recommend replacing any complex expressions with a [computed property](computed.html), one of the most fundamental pieces of Vue, which we'll cover shortly.

When using in-DOM templates (templates directly written in an HTML file), you should also avoid naming keys with uppercase characters, as browsers will coerce attribute names into lowercase:

```html
<!--
This will be converted to v-bind:[someattr] in in-DOM templates.
Unless you have a "someattr" property in your instance, your code won't work.
-->
<a v-bind:[someAttr]="value"> ... </a>
```

#### JavaScript Expressions

Template expressions are sandboxed and only have access to a [whitelist of globals](https://github.com/vuejs/vue-next/blob/master/packages/shared/src/globalsWhitelist.ts#L3) such as `Math` and `Date`. You should not attempt to access user defined globals in template expressions.
