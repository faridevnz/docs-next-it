# Gestione degli Eventi

## Ascoltare gli Eventi

Possiamo usare la direttiva `v-on`, che solitamente viene abbreviata con il simbolo `@`, per
ascoltare gli eventi del DOM ed eseguire del codice JavaScript quando questi vengono innescati.
L'utilizzo è `v-on:click="nomeMetodo"` o con l'abbreviazione, `@click="nomeMetodo"`

Per esempio:

```html
<div id="basic-event">
  <button @click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      counter: 0
    }
  }
}).mount('#basic-event')
```

Risultato:

<common-codepen-snippet title="Event handling: basic" slug="xxGadPZ" tab="html,result" :preview="false" />

## Metodi come Handler di Eventi

Se la logica degli handler degli eventi è più complessa, mettere il tuo JavaScript nel valore
dell'attributo `v-on` non è fattibile. Per questo `v-on` può anche accettare il nome di un
metodo che desideri chiamare.

Per esempio:

```html
<div id="event-with-method">
  <!-- `greet` è il nome del metodo definito più in basso -->
  <button @click="greet">Greet</button>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      name: 'Vue.js'
    }
  },
  methods: {
    greet(event) {
      // `this` all'interno dei metodi punta all'istanza correntemente attiva
      alert('Hello ' + this.name + '!')
      // `event` è l'evento del DOM nativo
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
}).mount('#event-with-method')
```

Risultato:

<common-codepen-snippet title="Event handling: with a method" slug="jOPvmaX" tab="js,result" :preview="false" />

## Metodi negli Handler Inline"

Anzichè fare il binding direttamente con il nome del metodo, possiamo anche usare i metodi in
un'istruzione JavaScript inline:

```html
<div id="inline-handler">
  <button @click="say('hi')">Say hi</button>
  <button @click="say('what')">Say what</button>
</div>
```

```js
Vue.createApp({
  methods: {
    say(message) {
      alert(message)
    }
  }
}).mount('#inline-handler')
```

Risultato:

<common-codepen-snippet title="Event handling: with an inline handler" slug="WNvgjda" tab="html,result" :preview="false" />

A volte possiamo aver bisogno di accedere all'evento del DOM originale in un'istruzione inline. Puoi passare l'evento in un metodo usando la variabile speciale `$event`:

```html
<button @click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```

```js
// ...
methods: {
  warn(message, event) {
    // ora abbiamo accesso all'evento nativo
    if (event) {
      event.preventDefault()
    }
    alert(message)
  }
}
```

## Handler di Eventi Multipli

In un handler puoi avere molteplici metodi separati da virgola come in questo esempio:

```html
<!-- entrambi one() e two() saranno eseguiti al click del pulsante -->
<button @click="one($event), two($event)">
  Submit
</button>
```

```js
// ...
methods: {
  one(event) {
    // logica del primo handler...
  },
  two(event) {
    // logica del secondo handler...
  }
}
```

## Modificatori di Eventi

È un'esigenza ricorrente chiamare `event.preventDefault()` o `event.stopPropagation()` all'interno degli handler. Sebbene possiamo farlo facilmente all'interno dei metodi,
sarebbe meglio se essi contengano soltanto la logica dei dati anzichè avere a che fare con
i dettagli dell'evento del DOM.

Per affrontare questo problema, Vue fornisce i **modificatori di eventi** per `v-on`. Ricorda che questi modificatori sono delle direttive postfisse denotate da un punto.

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```html
<!-- la propagazione dell'evento di click verrà stoppata -->
<a @click.stop="doThis"></a>

<!-- il submit dell'evento non farà più ricaricare la pagina -->
<form @submit.prevent="onSubmit"></form>

<!-- i modificatori possono essere concatenati -->
<a @click.stop.prevent="doThat"></a>

<!-- solamente il modificatore -->
<form @submit.prevent></form>

<!-- usare la modalità capture quando si aggiunge un listener di evento -->
<!-- i.e. l'handle di un evento che ha come target un elemento interno avverrà prima
dell'handle sull'elemento stesso -->
<div @click.capture="doThis">...</div>

<!-- il trigger dell'handler avverrà solamente se event.target è l'elemento stesso -->
<!-- i.e. non da un elemento figlio -->
<div @click.self="doThat">...</div>
```

::: tip Suggerimento
L'ordine conta quando si usano i modificatori perché il relativo codice è generato nello stesso
ordine. Quindi usare `@click.prevent.self` impedirà **tutti i click** mentre `@click.self.prevent` impedirà solamente i click sull'elemento stesso.
:::

```html
<!-- il trigger dell'evento di click sarà effettuato al massimo una volta -->
<a @click.once="doThis"></a>
```

Diversamente dagli altri modificatori, che sono esclusivi degli eventi nativi del DOM, il modificatore `.once` può essere usato anche sui [component events](component-custom-events.html).
Se non hai ancora letto nulla sui componenti, non preoccuparti per adesso.

Vue offre anche il modificatore `.passive`, che corrisponde all' [opzione `passive` di `addEventListener`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters).

```html
<!-- il comportamento predefinito dell'evento di scoll (scrolling)       -->
<!-- accadrà immediatamente, anziché aspettare che `onScroll` sia        -->
<!-- completato nel caso in cui esso contenga `event.preventDefault()`   -->
<div @scroll.passive="onScroll">...</div>
```

Il modificatore `.passive` è particolarmente utile per ottimizzare le performance sui
dispositivi mobili.

::: tip Suggerimento
Non usare `.passive` e `.prevent` insieme, perché `.prevent` verrà ignorato e il tuo browser
probabilmente mostrerà un warning. Ricorda, `.passive` comunica al browser che tu _non_ vuoi impedire il comportamento predefinito dell'evento.
:::

## Modificatori di Tasti

Quando siamo in ascolto degli eventi della tastiera, spesso abbiamo bisogno di controllare dei
tasti specifici. Vue permette di aggiungere dei modificatori di tasto per `v-on` o `@` quando
ascoltano gli eventi dei tasti

```html
<!-- `vm.submit()` verrà chiamata solamente quando il `Tasto` sarà `Enter` -->
<input @keyup.enter="submit" />
```

Puoi usare direttamente qualunque nome dei tasti tramite [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) come modificatori convertendoli in kebab-case.

```html
<input @keyup.page-down="onPageDown" />
```

Nell'esempio sovrastante, l'handler verrà chiamato solamente se `$event.key` è uguale a `'PageDoun'`.

### Alias per i Tasti

Vue formisce degli alias per i tasti usati più comunemente:

- `.enter`
- `.tab`
- `.delete` (cattura sia il tasto "Delete" sia "Backspace")
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

## Tasti Modificatori di Sistema

Puoi usare i seguenti modificatori per scatenare i listener degli eventi di mouse e tastiera
solamente quando i modificatori corrispondenti verranno premuti:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

::: tip Nota
Sulle tastiere Macintosh, meta è il tasto command (⌘). Sulle tastiere Windows, meta è il
tasto Windows (⊞). Sulle tastiere Sun Microsystems, meta è contrassegnato da un diamante pieno (◆). Su alcune tastiere, nello specifico sulle tastiere delle macchine MIT, Lisp e successori, come la tastiera Knight, space-cadet, meta è contrassegnato dalla scritta "META". Sulle tastiere simboliche, meta è contrassegnato dalla scritta "META" o "Meta".
:::

Per esempio:

```html
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

::: tip Suggerimento
Nota che i tasti modificatori sono differenti dai tasti normali. Quando sono usati con gli eventi `keyup`, devono essere premuti quando l'evento è emesso. In altre parole, `keyup.ctrl` verrà attivato solo se rilasci un tasto mentre tieni premuto `ctrl`. Non verrà attivato se rilasci soltanto `ctrl`.
:::

### Il Modificatore `.exact`

Il modificatore `.exact` permette di controllare l'esatta combinazione di modificatori di sistema necessaria per scatenare un evento.

```html
<!-- questo verrà innescato ugualmente con Alt o Shift premuti -->
<button @click.ctrl="onClick">A</button>

<!-- questo verrà innescato solo quando Ctrl e nessun altro tasto sarà premuto -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- questo verrà innescato solo quando nessun modificatore di sistema sarà premuto -->
<button @click.exact="onClick">A</button>
```

### Modificatori Pulsanti Mouse

- `.left`
- `.right`
- `.middle`

Questi modificatori limitano l'handler agli eventi scatenati da uno specifico pulsante del mouse.

## Perché i Listener in HTML?

Potresti essere preoccupato del fatto che questo approccio all'ascolto degli eventi violi le buone vecchie regole sulla "separazione degli interessi". Tranquillo - dal momento che le funzioni handler di Vue sono strettamente vincolate al ViewModel che gestisce la vista corrente, non causeranno nessuna difficoltà di manutenzione. Infatti, ci sono molti benefit nell'usare `v-on` o `@`:

1. È più facile individuare l'implementazione delle funzioni handler all'interno del tuo codice JS esaminando il template HTML.

2. Dal momento in cui non devi collegare manualmente i listener degli eventi in JS, il codice del tuo ViewModel può contenere solamente logica ed essere DOM-free. Questo lo rende più facile da testare.

3. Quando un ViewModel viene distrutto, tutti i listener degli eventi vengono automaticamente rimossi. Tu non devi preoccuparti di pulirli manualmente.
