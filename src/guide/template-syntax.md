# Sintassi Template

Vue.js usa una sintassi di templating basata su HTML che ti permette di fare il binding del DOM ai dati dell'istanza del corrispettivo componente in modo dichiarativo. Tutti i template Vue.js sono HTML validi che possono essere renderizzati dai browser compatibili con la specifica e dai parser HTML. 

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

I mustaches non possono essere usati dentro gli attributi HTML. Invece, use la [ direttiva `v-bind`](../api/directives.html#v-bind):

```html
<div v-bind:id="dynamicId"></div>
```

Se il valore collegato è `null` o `undefined` allora l'attributo non sarà incluso nell'elemento renderizzato.

In caso di un attributo booleano, dove la sua mera esistenza implica `true`, `v-bind` funziona in modo po' diverso. Per esempio:

```html
<button v-bind:disabled="isButtonDisabled">Button</button>
```

L'attributo `disabled` sarà incluso se `isButtonDisabled` ha un valore che restituisce `true`. Sarà incluso anche se il valore è una stringa vuota, mantenendo consistenza con `<button disabled="">`. Per altri valori che restituiscono `false` l'attributo sarà omesso.

### Uso di espressioni JavaScript

Fin'ora abbiamo fatto solo il binding a semplici key di una proprietà nel nostro template. Ma in realtà Vue.js supporta la piena espressività delle espressioni JavaScript all'interno dei data binding:

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

Queste espressioni saranno valutate come espressioni JavaScript nel data scope dell'istanza attualmente attiva. Una restrizione è che ogni binding può contenere **una singola espressione**, quindi l'esempio seguente **NON** funzionerà:

```html
<!-- questo è uno statement, non un'espressione: -->
{{ var a = 1 }}

<!-- i flussi di controllo non funzioneranno, usa l'operatore ternario -->
{{ if (ok) { return message } }}
```

## Direttive

Le direttive sono attributi speciali con il prefisso `v-`. Ci si aspetta che i valori degli attributi direttiva siano un'**espressione JavaScript singola** (con l'eccezione di `v-for` e `v-on`, che illustreremo dopo). Il compito di una direttiva è di applicare side effect al DOM reattivamente quando il valore della sua espressione cambia. 
Riprendiamo l'esempio che abbiamo illustrato nell'introduzione:

```html
<p v-if="seen">Now you see me</p>
```

In questo esempio, la direttiva `v-if` rimuoverebbe/inserirebbe l'elemento `<p>` basandosi sulla veridicità del valore dell'espressione `seen`.

### Argomenti

Alcune direttive possono ricevere un "argomento", denotato dai due punti dopo il nome della direttiva. Per esempio, la direttiva `v-bind` è usata per aggiornare reattivamente un attributi HTML:

```html
<a v-bind:href="url"> ... </a>
```

In questo esempio `href` è l'argomento, che dice alla direttiva `v-bind` di fare il binding dell'attributo `href` dell'elemento al valore dell'espressione `url`. 

Un altro esempio è la direttiva `v-on`, che ascolta gli eventi DOM:

```html
<a v-on:click="doSomething"> ... </a>
```

Qui l'argomento è il nome dell'evento che si vuole ascoltare. Parleremo della gestione degli eventi più nel dettaglio più avanti. 

### Argomenti Dinamici

È anche possibile usare un'espressione JavaScript nell'argomento di una direttiva racchiudendola con le parentesi quadre:

```html
<!--
Nota che ci sono alcune restrizioni sulle espressioni degli argomenti, come spiegato
nella sezione "Restrizioni sulle Espressioni negli Argomenti Dinamici" sottostante.
-->
<a v-bind:[attributeName]="url"> ... </a>
```

Qui `attributeName` sarà valutato dinamicamente come un'espressione JavaScript, e il valore valutato sarà usato come valore finale per l'argomento. Per esempio, se l'istanza del tuo componente ha la proprietà data `attributeName`, il cui valore è `"href"`, allora questo binding è l'equivalente di `v-bind:href`. 

Similarmente, puoi usare gli argomenti dinamici per fare il binding al nome dinamico di un evento:

```html
<a v-on:[eventName]="doSomething"> ... </a>
```

In questo esempio, quando il valore di `eventName` è `"focus"`, `v-on:[eventName]` sarà l'equivalente di `v-on:focus`.

### Modificatori

I modificatori sono postfissi speciali denotati da un punto, che indica il binding di una direttiva dovrebbe essere fatto in un qualche modo particolare. Per esempio, il modificare `.prevent` dice alla direttiva `v-on` di chiamare `event.preventDefault()` su l'evento innescato:

```html
<form v-on:submit.prevent="onSubmit">...</form>
```

Vedrai altri esempi di modificatori più tardi, [per `v-on`](events.md#event-modifiers) e [per `v-model`](forms.md#modifiers), quando esploreremo quelle funzionalità. 

## Shorthands

Il prefisso `v-` è un indizio visivo che aiuta ad identeficare un attributo specifico di Vue nel template. Questo è utile quando stai usando Vue.js per applicare comportamenti dinamici a un markup già esistente, ma può sembrare verboso per alcune direttive usate frequentemente. In questo caso, il bisogno del prefisso `v-` diventa meno importante quando stai sviluppando una [SPA](https://en.wikipedia.org/wiki/Single-page_application), dove Vue gestisce ogni template. Per questo motivo, Vue fornisce una forma abbreviata speciale per due delle sue direttive usate più frequentemente, `v-bind` e `v-on`:

### Forma abbreviata `v-bind`

```html
<!-- sintassi completa -->
<a v-bind:href="url"> ... </a>

<!-- forma abbreviata -->
<a :href="url"> ... </a>

<!-- forma abbreviata con argomenti dinamici -->
<a :[key]="url"> ... </a>
```

### Forma abbreviata `v-on`

```html
<!-- sintassi completa -->
<a v-on:click="doSomething"> ... </a>

<!-- forma abbreviata -->
<a @click="doSomething"> ... </a>

<!-- forma abbreviata con argomenti dinamici -->
<a @[event]="doSomething"> ... </a>
```

Potranno sembrare un po' diversi dal normale HTML, ma `:` e `@` sono caratteri validi per i nomi di attributi e tutti i browser supportati da Vue possono farne il parsing correttamente. In aggiunta, non appaiono nel markup renderizzato alla fine. La sintassi forma abbreviata è completamente opzionale, ma probabilmente la apprezzerai quando imparerai di più sul suo uso, più tardi nella guida. 

> Dalla prossima pagina, useremo la forma abbreviata nei nostri esempi, dato che è questo è l'uso più comune tra gli sviluppatori Vue.

### Avvertimenti 

#### Restrizioni sui Valori negli Argomenti Dinamici

Ci si aspetta che gli argomenti dinamici siano valutabili a stringhe, con l'eccezzione di `null`. Il valore speciale `null` può essere usato per rimuovere esplicitamente il binding. Qualsiasi altro valore non stringa innescherà un warning.

#### Restrizioni sulle Espressioni negli Argomenti Dinamici

Le espressioni negli argomenti dinamici hanno alcune restrizioni sulla sintassi perchè alcuni caratteri, come lo spazio e le virgolette, non sono validi all'interno del nome degli attributi HTML. Per esempio, la seguente espressione non è valida:

```html
<!-- Questo innescherà un warning del compilatore. -->
<a v-bind:['foo' + bar]="value"> ... </a>
```

Raccomandiamo di rimpiazzare ogni espressione complessa con una [computed property](computed.html), uno dei pezzi fondamentali di Vue, che spiegheremo a breve. 

Quando si sta utilizzato i template in-DOM (template scritti direttamente in un file HTML), dovresti anche evitare di avere delle key con caratteri in maiuscolo, dato che i browser faranno la coercizione del nome in minuscolo:

```html
<!--
Questo verrà covertito in v-bind:[someattr] nei template in-DOM.
A meno che tu non abbia una qualche proprietà chiamata "someattr", questo codice non funzionerà.
-->
<a v-bind:[someAttr]="value"> ... </a>
```

#### Espressioni JavaScript

Le espressioni nel template sono sandboxed e hanno accesso solo ad una [whitelist di globals](https://github.com/vuejs/vue-next/blob/master/packages/shared/src/globalsWhitelist.ts#L3) come `Math` e `Date`. Non dovresti provare ad accedere globals definite dall'utente nelle espressioni dentro il template. 
