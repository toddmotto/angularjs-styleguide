# Linee guida per AngularJS (ES2015)

### Architettura, struttura dei files, componenti, flusso dati unidirezionale (one-way dataflow) e migliori prassi.

---

> Vuoi un esempio di struttura come riferimento ? Controlla il mio [component based architecture 1.5 app](https://github.com/toddmotto/angular-1-5-components-app).

---

*Una guida di stile sensata per i teams, da [@toddmotto](//twitter.com/toddmotto)*

Questa architettura e linee guida sono state riscritte da capo per ES2015, i cambiamenti di AngularJS 1.5+ e per un futuro upgrade della tua applicazione ad Angular. Questa guida include delle nuove e migliori prassi per il one-way dataflow (flusso di dati unidirezionale), event delegation (delegazione d'evento), component architecture (architettura di un componente) e component routing (rotta del componente).

Puoi trovare le vecchie linee guida [qui](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5), e il ragionameto
che c'è dietro a quelle nuove [qui](https://toddmotto.com/rewriting-angular-styleguide-angular-2).

> Unisciti a questa definitiva esperienza d'apprendimento per dominare le caratteristiche base ed avanzate di Angular per costruire applicazioni veloci e scalabili.

<a href="https://courses.toddmotto.com" target="_blank"><img src="https://toddmotto.com/img/ua.png"></a>

# Tabella dei contenuti

  1. [Architettura Modulare](#architettura-modulare)
    1. [Teoria](#teoria-del-modulo)
    1. [Modulo radice](#modulo-radice)
    1. [Modulo Component](#modulo-components)
    1. [Modulo Common](#modulo-common)
    1. [Moduli di basso livello](#moduli-di-basso-livello)
    1. [Nomenclatura dei files](#nomenclatura-dei-files)
    1. [Struttura files scalabile](#struttura-files-scalabile)
  1. [Componenti](#componenti)
    1. [Teoria](#teoria-del-component)
    1. [Proprietá supportate](#proprietá-supportate)
    1. [Controllers](#controllers)
    1. [Flusso dati unidirezionale ed eventi](#flusso-dati-unidirezionale-ed-eventi)
    1. [Componenti con stato](#stateful-components)
    1. [Componenti privi di stato](#componenti-con-stato)
    1. [Componenti Indirizzati](#component-indirizzati)
  1. [Direttive](#direttive)
    1. [Teoria](#teoria-della-direttiva)
    1. [Proprietá raccomandate](#proprietá-raccomandate)
    1. [Costanti o classi](#costanti-o-classi)
  1. [Servizi](#servizi)
    1. [Teoria](#teoria-del-service)
    1. [Classi per Service](#classi-per-service)
  1. [ES2015 e strumenti](#es2015-e-strumenti)
  1. [Gestione dello stato](#gestione-dello-stato)
  1. [Risorse](#risorse)
  1. [Documentazione](#documentazione)
  1. [Contribuire](#contribuire)

# Architettura Modulare

Ogni modulo in una applicazione Angular è un modulo di tipo `.component`. Un modulo di tipo component è la definizione della radice dello stesso modulo che incapsula la logica, i templates, le rotte (routes) e i componenti figli. 

### Teoria del modulo

Il design nei moduli rispecchia direttamente la nostra struttura delle cartelle, che mantiene le cose gestibili e prevedibili.
Dovremmo avere idealmente tre livelli di moduli : root , component e common. Il modulo radice (root) definisce la base del modulo che inizializza la nostra app, e il corrispondente template.
Successivamente importiamo i nostri moduli componente e common nel modulo radice per includere le nostre dipendenze.
I moduli component e common dopo richiedono component di più basso livello, che contengono altri components, controllers, services, directives, filters e tests per ogni caratteristica riutilizzabile. 


**[Torna su](#tabella-dei-contenuti)**

### Modulo Radice 

Un modulo radice (root module) inizia con un componente principale che definisce l'elemento di base per l'intera applicazione, con un'instradamento (routing outlet) definito, nell'esempio mostrato usando `ui-view` da` ui-router`. 

```js
// app.component.js
export const AppComponent = {
  template: `
    <header>
        Hello world
    </header>
    <div>
        <div ui-view></div>
    </div>
    <footer>
        Copyright MyApp 2016.
    </footer>
  `
};
```
Il modulo radice è successivamente creato con `AppComponent` ed importato e registrato con `.component('app', AppComponent)`. Ulteriori importazioni di sotto-moduli (`app.components` e `app.commons`) sono fatti per includere tutti i componenti dell'applicazione. Noterai che anche gli stili sono importati qui, arriveremo a questo nei capitoli successivi di questa guida.

```js
// app.module.js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { AppComponent } from './app.component';
import { ComponentsModule } from './components/components.module';
import { CommonModule } from './common/common.module';
import './app.scss';

export const AppModule = angular
  .module('app', [
    ComponentsModule,
    CommonModule,
    uiRouter
  ])
  .component('app', AppComponent)
  .name;
```

**[Torna Su](#tabella-dei-contenuti)**

### Modulo Components

Il modulo Components (`app.components`) è il contenitore per tutti i componenti riutilizzabili. 
Guarda nell'esempio di prima importavamo i `ComponentsModule` e li iniettavamo nel modulo radice (root module). Questo ci fornisce un singolo luogo dove importare tutti questi componenti riutilizzabili dell'applicazione. 
Questi moduli che richiediamo sono disaccoppiati da tutti gli altri moduli, e quindi possono essere spostati in qualsiasi altra applicazione con facilitá.

```js
import angular from 'angular';
import { CalendarModule } from './calendar/calendar.module';
import { EventsModule } from './events/events.module';

export const ComponentsModule = angular
  .module('app.components', [
    CalendarModule,
    EventsModule
  ])
  .name;
```

**[Torna Su](#tabella-dei-contenuti)**

### Modulo Common

Il modulo (`app.common`) è il contenitore di riferimento per tutti i componenti specifici dell'applicazione che non vogliamo riutilizzare in un'altra applicazione.
Questi possono essere il layout, la navigazione o i footers. Guarda come nell'esempio di prima importavamo `CommonModule` e gli iniettavamo nel modulo radice (Root module), questo ci fornisce un singolo posto 
dove importare tutti i componenti comuni dell'applicazione.


```js
import angular from 'angular';
import { NavModule } from './nav/nav.module';
import { FooterModule } from './footer/footer.module';

export const CommonModule = angular
  .module('app.common', [
    NavModule,
    FooterModule
  ])
  .name;
```

**[Torna Su](#tabella-dei-contenuti)**

### Moduli di basso livello

I moduli di basso livello sono moduli `component` individuali che contengono la logica per ogni blocco di funzionalitá. Questi definiscono moduli da importare di seguito a livello superiore, come in un `ComponentsModule` o `CommonModule`, un esempio di seguito. 
Ricordati sempre di aggiungere un suffisso `.name` and ogni `export` quando crei un _nuovo_ module, non quando lo state referenziando. Come avrete notato la definizione delle rotte esiste anche qui, lo approfondiremo nei capitoli successivi.

```js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { CalendarComponent } from './calendar.component';
import './calendar.scss';

export const CalendarModule = angular
  .module('calendar', [
    uiRouter
  ])
  .component('calendar', CalendarComponent)
  .config(($stateProvider, $urlRouterProvider) => {
    'ngInject';
    $stateProvider
      .state('calendar', {
        url: '/calendar',
        component: 'calendar'
      });
    $urlRouterProvider.otherwise('/');
  })
  .name;
```

**[Torna Su](#tabella-dei-contenuti)**

### Nomenclatura dei files
Mantenerli semplici e minuscolo, usa il nome del componente , es. `calendar.*.js*`, `calendar-grid.*.js` con il nome del tipo di file contenuto al centro. Usa *.module.js come nome del file della definizione del modulo principale, cosí è consistente con le linee guida di Angular.

```
calendar.module.js
calendar.component.js
calendar.service.js
calendar.directive.js
calendar.filter.js
calendar.spec.js
calendar.html
calendar.scss
```

**[Torna Su](#tabella-dei-contenuti)**

### Struttura files scalabile

La struttura dei files è estremamente importante, questa descrive una struttura scalabile e prevedibile. Un esempio di struttura dei files per illustrare un'architettura a componenti modulare.

```
├── app/
│   ├── components/
│   │  ├── calendar/
│   │  │  ├── calendar.module.js
│   │  │  ├── calendar.component.js
│   │  │  ├── calendar.service.js
│   │  │  ├── calendar.spec.js
│   │  │  ├── calendar.html
│   │  │  ├── calendar.scss
│   │  │  └── calendar-grid/
│   │  │     ├── calendar-grid.module.js
│   │  │     ├── calendar-grid.component.js
│   │  │     ├── calendar-grid.directive.js
│   │  │     ├── calendar-grid.filter.js
│   │  │     ├── calendar-grid.spec.js
│   │  │     ├── calendar-grid.html
│   │  │     └── calendar-grid.scss
│   │  ├── events/
│   │  │  ├── events.module.js
│   │  │  ├── events.component.js
│   │  │  ├── events.directive.js
│   │  │  ├── events.service.js
│   │  │  ├── events.spec.js
│   │  │  ├── events.html
│   │  │  ├── events.scss
│   │  │  └── events-signup/
│   │  │     ├── events-signup.module.js
│   │  │     ├── events-signup.component.js
│   │  │     ├── events-signup.service.js
│   │  │     ├── events-signup.spec.js
│   │  │     ├── events-signup.html
│   │  │     └── events-signup.scss
│   │  └── components.module.js
│   ├── common/
│   │  ├── nav/
│   │  │     ├── nav.module.js
│   │  │     ├── nav.component.js
│   │  │     ├── nav.service.js
│   │  │     ├── nav.spec.js
│   │  │     ├── nav.html
│   │  │     └── nav.scss
│   │  ├── footer/
│   │  │     ├── footer.module.js
│   │  │     ├── footer.component.js
│   │  │     ├── footer.service.js
│   │  │     ├── footer.spec.js
│   │  │     ├── footer.html
│   │  │     └── footer.scss
│   │  └── common.module.js
│   ├── app.module.js
│   ├── app.component.js
│   └── app.scss
└── index.html
```

La cartella di più alto livello contiene semplicemente index.html and app/, la directory dove tutti, root components, common modules e i moduli di basso livello risiedono, insieme con i loro template e stili.

**[Torna Su](#tabella-dei-contenuti)**

# Componenti

### Teoria del componente

I Componenti sono essenzialmente templates con un controller. Essi _non_ sono delle direttive, né si dovrebbe sostituire la direttive con i components, fino a che non effettui un upgrade dei "templates delle direttive" con controllers, che sono più adatte come componente. I components contengono anche dei legami che definiscono entrata ed uscita dei dati e degli eventi, i punti di ancoraggio del ciclo di vita del componente e l'abilitá di usare il flusso dati unidirezionale ed oggetti di tipo evento (event objects) per ottenere i dati dal componente genitore. Questi sono di fatto lo standard in AngularJS 1.5 e superiori. Tutto quello che è basato su controllers e templates somiglierá a un component, che potrebbe essere dinamico , privo di stato o un componente mappato. Puoi pensare ad un "component" come un pezzo di codice completo, non solo come la definzione dell'oggetto `.component()`.Esploriamo alcune linee guida e consigli per i components, per poi passare ai concetti di come dovrebbero essere strutturati in modo dinamico , privo di stato e mappato.

**[Torna Su](#tabella-dei-contenuti)**

### Proprietá supportate

Ci sono delle proprietá supportate per il metodo `.component()` che potresti/dovresti usare:

| Proprietá | Supporto |
|---|---|
| bindings | Si, usa solamente `'@'`, `'<'`, `'&'` |
| controller | Si |
| controllerAs | Si, di default è `$ctrl` |
| require | Si (nuova sintassi Object) |
| template | Si |
| templateUrl | Si |
| transclude | Si |

**[Torna Su](#tabella-dei-contenuti)**

### Controllers

I controllers dovrebbero essere usati solamente a fianco dei components, mai in altri punti. Se senti di aver bisogno di un controller, quello di cui hai veramente bisogno potrebbe essere un componente stateless per gestire questa particolare porzione di comportamento.

Di seguito alcuni consigli per usare `Class` per i controllers:

* Non utilizzare il suffisso "Controller", cioè utilizzi controller: class TodoComponent {...} per essere più vicino a come si fa in Angular
* Usare sempre il construttore (`constructor()`) per l'iniezione delle dipendenze.
* Usa la sintassi [ng-annotate](https://github.com/olov/ng-annotate) `'ngInject';` per iniettare (`$inject`) le annotazioni.
* Non esportare il `Class` direttamente, esporta il suo nome per permettere l'utilizzo della notazione con l'`$inject`
* Se hai bisogno di acceredere al blocco lessicale, utilizza le arrow functions ( `() => {}` ).
* Alternativamente alle arrow functions, `let ctrl = this` è anche accettabile e potrebbe avere più senso ma dipende dai casi d'uso.
* Lega tutte le funzioni pubbliche direttamente al `Class`
* Assicurati di utilizzare i punti d'ancoraggio del ciclo di vita del componente, `$onInit`, `$onChanges`, `$postLink` and `$onDestroy`
  * Nota :`$onChanges` è chiamato prima dell' `$onInit`, guarda la sezione [resources](#risorse) per aticoli che spiegano questo in modo più dettagliato.
* Usa `require` al fianco di `$onInit` per fare riferimento a qualsiasi logica ereditata
* Non sovrascrivere l'impostazione predefinita `$ctrl` sinonimo per la sintassi `controllerAs`, quindi non usare `controllerAs` ovunque


**[Torna Su](#tabella-dei-contenuti)**

### Flusso dati unidirezionale ed eventi

IL flusso dati unidirezionale è stato introdotto in AngularJS 1.5 e ridefinisce la comunicazione dei componenti. 

Di seguito alcuni consigli per usare il flusso dati unidirezionale:

* Nei componenti che ricevono dati, usare sempre la sintassi del flusso dati unidirezionale `'<'`.
* _Non utilizzare_ `'='` il flusso dati a due-vie, ovunque. 
* I Components che hanno il `bindings` dovrebbero utilizzare `$onChanges` per clonare i dai unidirezionali per gli oggetti passati per referenza e aggiornare i dati del genitore
* Usare `$event` come argomento della funzione nel metodo genitore (guarda l'esempio stateful sotto `$ctrl.addTodo($event)`)
* Passare un `$event: {}` oggetto di backup da un componente senza stato (vedi esempio sotto apolidi `this.onAddTodo`).
  * Bonus : Usa un `EventEmitter` come contenitore con `.value()` per rispecchiare Angular2, evitare la crezione manuale di un oggetto `$event` 
* Perchè? Questo rispecchia Angular2 e mantiene una consistenza all'interno di ogni componente. Questo rende lo stato prevedibile.


**[Torna Su](#tabella-dei-contenuti)**

### Componente Stateful 

Definiamo ciò che noi chiameremmo un "componente stateful". 

* Recupera lo stato essenzialmente comunicando con un'API backend attraverso un service
* Non muta il suo stato direttamente 
* Genera i componenti figlio che mutano lo stato
* Definiti anche componenti intelligenti/contenitore

Un esempio di un componente con stato (stateful), completo del suo modulo di basso livello definito (solo a scopo dimostrativo), parte del codice é stato omesso per brevitá

```js
/* ----- todo/todo.component.js ----- */
import templateUrl from './todo.html';

export const TodoComponent = {
  templateUrl,
  controller: class TodoComponent {
    constructor(TodoService) {
      'ngInject';
      this.todoService = TodoService;
    }
    $onInit() {
      this.newTodo = {
        title: '',
        selected: false
      };
      this.todos = [];
      this.todoService.getTodos().then(response => this.todos = response);
    }
    addTodo({ todo }) {
      if (!todo) return;
      this.todos.unshift(todo);
      this.newTodo = {
        title: '',
        selected: false
      };
    }
  }
};

/* ----- todo/todo.html ----- */
<div class="todo">
  <todo-form
    todo="$ctrl.newTodo"
    on-add-todo="$ctrl.addTodo($event);"></todo-form>
  <todo-list
    todos="$ctrl.todos"></todo-list>
</div>

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;
```
Questo esempio mostra un componente di tipo stateful, che legge lo stato all'interno del controller, attraverso un service e dopo lo passa ai componenti figli di tipo stateless. Note come non ci siano direttive utilizzate  come ad esempio `ng-repeat` and simili all'interno del template. Invece dati e funzioni sono delegati all'interno `<todo-form>` e `<todo-list>` che sono dei componenti di tipo stateless.

**[Torna Su](#tabella-dei-contenuti)**

### Componenti privi di stato

Andiamo a definire quelli che chiamiamo componenti privi di stato (stateless)

* Hanno dati in entrata ed uscita definiti usando `bindings:{}`
* I dati arrivano al componente attraverso l'attributo bindings (inputs)
* I dati lasciano il componente attraverso gli eventi (outputs)
* Mutano il loro stato, paddano i dati a livello superiore su richiesta (come ad esempio un click o un submit)
* Non si preoccupano da dove arrivano i dati, sono stateles
* Sono componenti altamente riutilizzabili
* Vengono anche definiti componenti muti / o di presentazione

Un esempio di componente stateless (useremo `<todo-form>` come esempio), completo del suo modulo di basso livello ( lo useremo solo a scopo dimostrativo, pate del codice è stato omesso per brevitá): 


```js
/* ----- todo/todo-form/todo-form.component.js ----- */
import templateUrl from './todo-form.html';

export const TodoFormComponent = {
  bindings: {
    todo: '<',
    onAddTodo: '&'
  },
  templateUrl,
  controller: class TodoFormComponent {
    constructor(EventEmitter) {
        'ngInject';
        this.EventEmitter = EventEmitter;
    }
    $onChanges(changes) {
      if (changes.todo) {
        this.todo = Object.assign({}, this.todo);
      }
    }
    onSubmit() {
      if (!this.todo.title) return;
      // with EventEmitter wrapper
      this.onAddTodo(
        this.EventEmitter({
          todo: this.todo
        })
      );
      // without EventEmitter wrapper
      this.onAddTodo({
        $event: {
          todo: this.todo
        }
      });
    }
  }
};


/* ----- todo/todo-form/todo-form.html ----- */
<form name="todoForm" ng-submit="$ctrl.onSubmit();">
  <input type="text" ng-model="$ctrl.todo.title">
  <button type="submit">Submit</button>
</form>

/* ----- todo/todo-form/todo-form.module.js ----- */
import angular from 'angular';
import { TodoFormComponent } from './todo-form.component';
import './todo-form.scss';

export const TodoFormModule = angular
  .module('todo.form', [])
  .component('todoForm', TodoFormComponent)
  .value('EventEmitter', payload => ({ $event: payload }))
  .name;
```

Nota come il componente `<todo-form>` non recupera nessuno stato, semplicemente lo riceve, muta un oggetto attraverso la logica del controller associato ad esso e lo passa al component parent attraverso le proprietá di bindings. In questo esempio l'hook `$onChanges` del ciclo di vita del componente crea un clone del suo stato iniziale `this.todo` aggancia l'oggetto e lo riassegna, questo significa che i date del componente genitore non vengono alaterati fino a che non inviamo il form a fianco al flusso unidirezionale con la nuova sintassi `'<'`.

**[Torna Su](#tabella-dei-contenuti)**

### Componenti con indirizzamento 

Andiamo a definire quello che chiamiamo un componente con indirizzamento (routed components)

* É essenzialmente un componente di tipo stateful, con una definizione di un rotta (route)
* Non utilizzeremo piú files `router.js`
* Useremo componenti con rotte per definire la loro propria logica
* I dati in "inputs" del componente vengono risolti con un resolve della rotta ( opzionale, ancora disponibile nel controller con la chiamata ad un service )


Per questo esempio andremo a prendere il componente `<todo>` esistente, rifattorizzandolo per utilizzare un rotta e usando `bindings` sul componente che riceveranno i dati (il segreto qui con `ui-router` é il `resolve` che creiamo, in questo caso `todoData`)
viene mappata per noi direttamente attraverso il `bindings`). Lo tratteremo come un componente con rotta perchè é essenzialmente una "vista";

```js
/* ----- todo/todo.component.js ----- */
import templateUrl from './todo.html';

export const TodoComponent = {
  bindings: {
    todoData: '<'
  },
  templateUrl,
  controller: class TodoComponent {
    constructor() {
      'ngInject'; // Not actually needed but best practice to keep here incase dependencies needed in the future
    }
    $onInit() {
      this.newTodo = {
        title: '',
        selected: false
      };
    }
    $onChanges(changes) {
      if (changes.todoData) {
        this.todos = Object.assign({}, this.todoData);
      }
    }
    addTodo({ todo }) {
      if (!todo) return;
      this.todos.unshift(todo);
      this.newTodo = {
        title: '',
        selected: false
      };
    }
  }
};

/* ----- todo/todo.html ----- */
<div class="todo">
  <todo-form
    todo="$ctrl.newTodo"
    on-add-todo="$ctrl.addTodo($event);"></todo-form>
  <todo-list
    todos="$ctrl.todos"></todo-list>
</div>

/* ----- todo/todo.service.js ----- */
export class TodoService {
  constructor($http) {
    'ngInject';
    this.$http = $http;
  }
  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { TodoComponent } from './todo.component';
import { TodoService } from './todo.service';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [
    uiRouter
  ])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .config(($stateProvider, $urlRouterProvider) => {
    'ngInject';
    $stateProvider
      .state('todos', {
        url: '/todos',
        component: 'todo',
        resolve: {
          todoData: TodoService => TodoService.getTodos()
        }
      });
    $urlRouterProvider.otherwise('/');
  })
  .name;
```

**[Torna Su](#tabella-dei-contenuti)**

# Direttive

### Teoria delle direttive

Le direttive ci forniscono molti strumenti `template`, associazioni con lo `scope` , `bindToController`,` link` e molte altre cose. L'utilizzo di questi strumenti deve essere valutato attentamente, ora che esiste `.component()`. Le Direttive non dovrebbero piú dichiarare templates o controllers, o ricevere dati tramite associazioni. Le direttive devono essere utilizzate esclusivamente per la decorazione del DOM. Questo vuol dire che si estende HTML esistente - creato con `.component()`. In un certo senso semplice, se avete bisogno di eventi DOM / API personalizzate e la logica, utilizzare una direttiva e associarlo a un modello all'interno di un componente. Se avete bisogno di una quantitá ragionevole di manipolazione del DOM, c'è anche l'hook `$postLink` da prendere in considerazione, ma questo non è un il punto dove migrare tutte le manipolazioni del DOM, utilizzare un direttiva, se è possibile per le cose non-Angular.


Di seguito alcuni consigli per usare le Direttive:

* Non usate templates, scope, bindToController o controllers
* Utilizzare sempre `restrict: 'A'` per le direttive
* Usare compile e link dove necessario
* Ricordati di distruggere e disassociare tutti i gestori di eventi dentro `$scope.$on('$destroy', fn);`



**[Torna Su](#tabella-dei-contenuti)**

### Recommended properties

Dato che le direttive supportano gran parte di quello che `.component()` fa (le direttive erano i components in origine), mi raccomando di limitare la definizione dell'oggetto della direttva a queste proprietá, per evitare di usare le direttive in modo scorretto.


| Proprietá | Usarla ? | Perché |
|---|---|---|
| bindToController | No | Usare `bindings` nei componenti |
| compile | Yes | Per pre-compilare la manipolazione del dom e degli eventi |
| controller | No | Usa un component |
| controllerAs | No | Usa un component |
| link functions | Yes | Per la pre/post manipolazione del DOM / Eventi |
| multiElement | Yes | [Guarda la guida](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Yes | [Guarda la guida](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | No | Usa un component |
| restrict | Yes | Definisci l'uso della direttiva, usa sempre `'A'` |
| scope | No | Usa un component |
| template | No | Usa un component |
| templateNamespace | Si (se devi) | [Guarda la guida](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | No | Usa un component |
| transclude | No | Usa un component |

**[Torna Su](#tabella-dei-contenuti)**

###  Costanti o Classi

Ci sono molti modi di approcciare l'utilizzo di ES2015 e direttive, sia con le arrow functions e assegnamento semplice, oppure usando il costruttore `Class` di ES2015. Scegli quello che ritieni migliore per te o per il tuo team, tieni a mente che Angular2 usa `Class`

Ecco un esempio utilizzando una costante con una funzione freccia un wrapper espressione `() => ({})` che restituisce un oggetto letterale (notare le differenze di utilizzo all'interno `.directive()`): 


```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

export const TodoAutoFocus = ($timeout) => {
  'ngInject';
  return {
    restrict: 'A',
    link($scope, $element, $attrs) {
      $scope.$watch($attrs.todoAutofocus, (newValue, oldValue) => {
        if (!newValue) {
          return;
        }
        $timeout(() => $element[0].focus());
      });
    }
  }
};

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoAutofocus } from './todo-autofocus.directive';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', TodoAutoFocus)
  .name;
```

O usando ES2015 `Class` (notare la chiamata manuale `new TodoAutoFocus` quando registriamo la direttiva) per creare l'oggetto.


```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

export class TodoAutoFocus {
  constructor($timeout) {
    'ngInject';
    this.restrict = 'A';
    this.$timeout = $timeout;
  }
  link($scope, $element, $attrs) {
    $scope.$watch($attrs.todoAutofocus, (newValue, oldValue) => {
      if (!newValue) {
        return;
      }
      this.$timeout(() => $element[0].focus());
    });
  }
}

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoAutofocus } from './todo-autofocus.directive';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', ($timeout) => new TodoAutoFocus($timeout))
  .name;
```

**[Torna Su](#tabella-dei-contenuti)**

# Servizi 

### Teoria del servizio (Service)

I Services sono essenzialmente contenitori per la logica di business che i nostri componenti non dovrebbero richiedere direttamente, I services contengono altri services iterni o esterni come `$http`, che possiamo iniettare dentro i controllers dei componenti in qualsiasi punto della nostra app. Abbiamo due modi per creare serivces, usando `.service()` oppure `.factory()`. Con ES2015 `Class`, dovremmo usare solo `.service()` corredato dell'annotazione per la iniezione delle dipendenze `$inject`.

**[Torna Su](#tabella-dei-contenuti)**

### Classi per Service

Qui un esempio dell'implementazione per la nostra `<todo>` app usando ES2015 `Class`:

```js
/* ----- todo/todo.service.js ----- */
export class TodoService {
  constructor($http) {
    'ngInject;'
    this.$http = $http;
  }
  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoService } from './todo.service';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .name;
```

**[Torna Su](#tabella-dei-contenuti)**

# Stili

Usando [Webpack](https://webpack.github.io/) possiamo utlizzare lo statement `import` nel nostro file `*.module.js` per permettere a Webpack  d'includere questo file nel nostro foglio di stile.
In questo modo ci permette di mantenere i nostri componenti isolati sia per funzionalitá e stile, si allinea molto più da vicino a come fogli di stile sono dichiarati per l'uso in Angular. 
In questo modo non si isolano i nostri stili per questo questo determinato componente come fa Angular,
gli stili saranno ancora utilizzabili per tutta l'applicazione, ma è più gestibile e rende la nostra struttura delle applicazioni più facile per ragionarci.
Se si dispone di alcune variabili o stili utilizzati a livello globale come elementi di input di un form, allora questi files devono ancora essere messi nella root della cartella `scss`. per esempio. `SCSS / _forms.scss`. 
Questi stili globali possono essere `@importati` nel vostro modulo principale (`app.module.js`) come faresti normalmente.


**[Torna Su](#tabella-dei-contenuti)**

# ES2015 e Strumenti

##### ES2015

* Usa [Babel](https://babeljs.io/) per compilare il tuo codice ES2015 ed i polyfills
* Considera l'utilizzo di [TypeScript](http://www.typescriptlang.org/) pre creare un aggiornameni indirizzato ad Angular. 


##### Strumenti
* Utilizza `ui-router` [ultima alpha](https://github.com/angular-ui/ui-router) (guarda il Readme) se vuoi supportare il component-routing.
  * Altrimenti rimarrai bloccato con `template: '<component>'` e nessuna mappatura del `bindings`/resolve 
* Considera di pre-caricare i templates all'interno dei `$templateCache` with `angular-template`
  * [Versione Gulp](https://www.npmjs.com/package/gulp-angular-templatecache)
  * [Versione Grunt](https://www.npmjs.com/package/grunt-angular-templates)
  * [Versione Webpack](https://github.com/WearyMonkey/ngtemplate-loader)
* Considera l'utlizzo di [Webpack](https://webpack.github.io/) per compilare il tuo codice ES2015.
* Usa [ngAnnotate](https://github.com/olov/ng-annotate) per annotare le proprietá atuomanticamente con `$inject`.
* Come utilizzarlo [ngAnnotate with ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/#ng-annotate)

**[Torna Su](#tabella-dei-contenuti)**

# Gestione dello stato

Considera l'uso di Redux con AngularJS 1.5 per la gestione dei dati.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[Torna Su](#tabella-dei-contenuti)**

# Risorse

* [Comprensione del metodo .component()](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Usare "require" con $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Comprensione di tutti i cicli di vita degli hooks, $onInit, $onChanges, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Usare "resolve" nelle rotte (routes) ](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Redux e Angular gestione dello stato](http://blog.rangle.io/managing-state-redux-angular/)
* [Applicazione d'esempio per la community](https://github.com/chihab/angular-styleguide-sample)

**[Torna Su](#tabella-dei-contenuti)**

# Documentazione
Per tutto il resto, incluso il riferimento all'API, controller la [documentazione di AngularJS](//docs.angularjs.org/api). 


# Contribuire
Aprire prima un problema per discutere i potenziali cambiamenti e aggiunte.Per favore non aprire problemi per effettuare domande.


## License

#### (The MIT License)

Copyright (c) 2016 Todd Motto

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
