# Guia de AngularJS (ES2015)

### Arquitetura, estrutura de ficheiros, componentes, fluxo de data unidireccional e boas práticas

*Um guia de estilo sensato para equipas [@toddmotto](//twitter.com/toddmotto)*

Esta arquitetura e guia de estilo foi totalmente reescrita para ES2015, mudanças em AngularJS 1.5+ para futuro upgrade para Angular. Este guia de estilo contem boas práticas para fluxo de data unidirecional, delegação de eventos, arquitetura por componentes e component routing.

O guia de estilo antigo pode ser encontrado [aqui](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5), e os motivos para a criação de um novo [aqui](https://toddmotto.com/rewriting-angular-styleguide-angular-2).

> Junte-se á experiencia de ensino Ultimate AngularJS para dominar funcionalidades básicas e avançadas de Angular e criar aplicações rápidas e escaláveis.

<a href="https://courses.toddmotto.com" target="_blank"><img src="https://toddmotto.com/img/ua.png"></a>

## Tabela de conteúdos

  1. [Arquitetura modular](#arquitetura-modular)
    1. [Teoria](#teoria)
    1. [Root module](#root-module)
    1. [Component module](#component-module)
    1. [Common module](#common-module)
    1. [Low-level modules](#low-level-modules)
    1. [Estrutura de ficheiros escalável](#estrutura-de-ficheiros-escalável)
    1. [Convenções para nomes de ficheiros](#convenções-para-nomes-de-ficheiros)
  1. [Componentes](#componentes)
    1. [Teoria](#componentes-teoria)
    1. [Propriedades suportadas](#propriedades-suportadas)
    1. [Controllers](#controllers)
    1. [One-way dataflow e Eventos](#one-way-dataflow-e-eventos)
    1. [Stateful Components](#stateful-components)
    1. [Stateless Components](#stateless-components)
    1. [Routed Components](#routed-components)
  1. [Diretivas](#diretivas)
    1. [Teoria](#diretivas-teoria)
    1. [Propriedades recomendadas](#propriedades-recomendadas)
    1. [Classes e constantes](#classes-e-constantes)
  1. [Serviços](#serviços)
    1. [Teoria](#serviços-teoria)
    1. [Classes para serviços](#classes-para-serviços)
  1. [ES2015 e Tooling](#es2015-e-tooling)
  1. [State management](#state-management)
  1. [Recursos](#recursos)
  1. [Documentação](#documentação)
  1. [Contribuições](#contribuições)

# Arquitetura modular

Cada module numa aplicação Angular é um module component. Um module component é a definição raiz para esse module e contém a lógica, templates, routing e child components (componentes filhos).

### Teoria

O esquema dos módulos relaciona-se diretamente com a nossa estrutura de ficheiros, que mantém tudo gerível e escalável. Devemos idealmente ter três high-level modules : root, component e common. O root module define o módulo base que faz o bootstrap (inicialização) da nossa app, e correspondente template. Podemos então importar os nossos component e common modules para o nosso root module para incluir as nossas dependências. Os component e common modules requerem por sua vez lower-level component modules, que contêm os nossos componentes, serviços, diretivas, filtros e testes para cada funcionalidade reutilizável.

**[Voltar ao topo](#tabela-de-conteúdos)**

### Root module

Um root module começa com um root component que define o elemento base para toda a aplicação, com uma routing outlet definida. o exemplo demonstra utilizando `ui-view` do `ui-router`.

```js
// app.component.js
const AppComponent = {
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

export default AppComponent;
```

Um root module é então criado, com `AppComponent` importado e registado com `.component('app', AppComponent)`. Imports para outros submodules (component e common modules) são também feitos para incluir todos os componentes relevantes para a aplicação.

```js
// app.module.js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import ComponentsModule from './components/components.module';
import CommonModule from './common/common.module';

const AppModule = angular
  .module('app', [
    ComponentsModule,
    CommonModule,
    uiRouter
  ])
  .component('app', AppComponent)
  .name;

export default AppModule;
```

**[Voltar ao topo](#tabela-de-conteúdos)**

### Component module

Um component module é a aquele que contém referência para todos os componentes reutilizáveis. Em baixo importamos `ComponentsModule` e injectamolos para o Root module, isto dá-nos um local onde podemos importar todos os componentes de uma app. Estes módulos que requeremos estão desassociados de quaisquer outros módulos e podem portanto ser importados para outra aplicação facilmente.

```js
import angular from 'angular';
import CalendarModule from './calendar/calendar.module';
import EventsModule from './events/events.module';

const ComponentsModule = angular
  .module('app.components', [
    CalendarModule,
    EventsModule
  ])
  .name;

export default ComponentsModule;
```

**[Voltar ao topo](#tabela-de-conteúdos)**

### Common module

O Common module é aquele que contém referência para todos os componentes específicos à aplicação, que não queremos utilizar numa outra aplicação. Podem ser coisas como Layout, navegação e footers. Ver em baixo como importamos `CommonModule` o e os injetamos para o Root module, isto dá-nos um local onde podemos colocar e importar todos os common components para a app.

```js
import angular from 'angular';
import NavModule from './nav/nav.module';
import FooterModule from './footer/footer.module';

const CommonModule = angular
  .module('app.common', [
    NavModule,
    FooterModule
  ])
  .name;

export default CommonModule;
```

**[Voltar ao topo](#tabela-de-conteúdos)**

### Low-level modules

Low-level moduels são component modules individuais que contêm a lógica de cada feature block. Cada um deles definirá um modulo, para ser importado para um higher-level module, como um component ou common module tal como exemplificado antes. Sempre lembrar de adicionar o sufixo `.name` a cada `export` ao criar um _novo_ modulo, e não quando referenciando um. Poderá perceber que routing definitions também existiram aqui, iremos abordar este tema mais a frente neste guia.

```js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import CalendarComponent from './calendar.component';

const CalendarModule = angular
  .module('calendar', [
    uiRouter
  ])
  .component('calendar', CalendarComponent)
  .config(($stateProvider, $urlRouterProvider) => {
    $stateProvider
      .state('calendar', {
        url: '/calendar',
        component: 'calendar'
      });
    $urlRouterProvider.otherwise('/');
  })
  .name;

export default CalendarModule;
```

**[Voltar ao topo](#tabela-de-conteúdos)**

# Convenções para nomes de ficheiros

Manter tudo simples e lowercase, utilizar o nome do component, i.e. `calendar.*.js*`, `calendar-grid.*.js` - com o tipo do ficheiro no meio. Utilize `*.module.js` para o ficheiro do modulo de definição, para que possa importar o modulo pelo nome da diretoria.

```
calendar.module.js
calendar.controller.js
calendar.component.js
calendar.service.js
calendar.directive.js
calendar.filter.js
calendar.spec.js
```

**[Voltar ao topo](#tabela-de-conteúdos)**

### Estrutura de ficheiros escalável

Estrutura de ficheiros é extremamente importante, aqui descrevemos uma estrutura escalável e previsível. Um exemplo de uma estrutura que ilustra uma arquitetura modular de componentes.

```
├── app/
│   ├── components/
│   │  ├── calendar/
│   │  │  ├── calendar.module.js
│   │  │  ├── calendar.controller.js
│   │  │  ├── calendar.component.js
│   │  │  ├── calendar.service.js
│   │  │  ├── calendar.spec.js
│   │  │  └── calendar-grid/
│   │  │     ├── calendar-grid.module.js
│   │  │     ├── calendar-grid.controller.js
│   │  │     ├── calendar-grid.component.js
│   │  │     ├── calendar-grid.directive.js
│   │  │     ├── calendar-grid.filter.js
│   │  │     └── calendar-grid.spec.js
│   │  ├── events/
│   │  │  ├── events.module.js
│   │  │  ├── events.controller.js
│   │  │  ├── events.component.js
│   │  │  ├── events.directive.js
│   │  │  ├── events.service.js
│   │  │  ├── events.spec.js
│   │  │  └── events-signup/
│   │  │     ├── events-signup.module.js
│   │  │     ├── events-signup.controller.js
│   │  │     ├── events-signup.component.js
│   │  │     ├── events-signup.service.js
│   │  │     └── events-signup.spec.js
│   │  └── components.module.js
│   ├── common/
│   │  ├── nav/
│   │  │     ├── nav.module.js
│   │  │     ├── nav.controller.js
│   │  │     ├── nav.component.js
│   │  │     ├── nav.service.js
│   │  │     └── nav.spec.js
│   │  ├── footer/
│   │  │     ├── footer.module.js
│   │  │     ├── footer.controller.js
│   │  │     ├── footer.component.js
│   │  │     ├── footer.service.js
│   │  │     └── footer.spec.js
│   │  └── common.module.js
│   ├── app.module.js
│   └── app.component.js
└── index.html
```

A primeira diretoria de topo apenas contém um `index.html` e `app/`, a diretoria onde todos os nossos root, component, common e low-level modules estão inseridos.

**[Voltar ao topo](#tabela-de-conteúdos)**

# Componentes

### Componentes Teoria

Componentes são essencialmente templates com controllers. Eles _não_ são diretivas, nem se deve tentar substituir diretivas com componentes, a não ser que esteja a atualizar "template Directives" com controllers, que funcionam melhor como componentes. Componentes também contêm bindings que definem inputs e outputs para dados e eventos, lifecycle hooks e a habilidade de utilizar fluxo de dados unidirecional e Objetos de eventos para enviar dados para o topo para um parent component. Estes são os novos standards para AngularJS 1.5 e para cima. Tudo o que for template e controller driven que criarmos e muito provável que seja um componente, que pode ser stateful, stateless ou um routed component. Pode-se pensar num componente como um pedaço de código completo e não só o `.component()` definition Object. Vamos analisar algumas boas práticas e conselhos para componentes, e depois entrar em como se deve organizá-los em conceitos de stateful, stateless e routed components.

**[Voltar ao topo](#tabela-de-conteúdos)**

### Propriedades suportadas

Estas são as propriedades suportadas para `.component()` que podem ser utilizadas :

| Propriedade | Suporte |
|---|---|
| bindings | Sim, usar `'@'`, `'<'`, `'&'` apenas |
| controller | Sim |
| controllerAs | Sim, default é `$ctrl` |
| require | Sim (new Object syntax) |
| template | Sim |
| templateUrl | Sim |
| transclude | Sim |

**[Voltar ao topo](#tabela-de-conteúdos)**

### Controllers

Controllers devem ser utilizados juntamente com componentes, e nunca em qualquer outro sitio. Se pensamos que precisamos de um controller, o que realmente precisamos é um stateless component para gerir este pedaço de comportamento.

Algumas conselhos para utilizar `Class` para controllers:

* Utilizar sempre o `constructor` para propósitos de dependency injection
* Não exportar a `Class` diretamnete mas sim o nome para permitir `$inject` annotations
* Se for preciso aceder ao lexical scope utilizar arrow functions
* Como alternativa a arrow functions `let ctrl = this;` também é aceitavel e pode até fazer mais sentido dependendo do use caso
* Fazer Bind the todas as funções públicas direcatamente à `Class`
* Utilizar os lifecycles hooks apropriados,`$onInit`, `$onChanges`, `$postLink` e `$onDestroy`
  * Nota: `$onChanges` é chamado antes de `$onInit`, ver secção [recursos](#recursos) para artigoes detalhando melhor este comportamento.
* Utilizar `require` juntamente com `$onInit` para referenciar qualquer logica herdada
* Não substituir o default alias `$ctrl` para utilização da sintaxe `controllerAs`, ou seja não utilizar `controllerAs` em lado nenhum

**[Voltar ao topo](#tabela-de-conteúdos)**

### Fluxo de dados unidireccional e Eventos

Fluxo de dados unidireccional foi introduzido com AngularJS 1.5 e redefine a comunicação entre componentes

Alguns conselhos ao utilizar fluxo de dados unidireccional:

* Em componentes que recebem dados, utilizar sempre a sintaxe de one-way databinding `'<'`
* _Não_ utilizar `'='` sintaxe de two-way databinding em lado nenhum, nunca
* Componentes que tenham `bindings` devem utilizar `$onChanges` para clonar os dados passados via one-way binding e partir Objectos passados por referência e actualizar os dados dos seus parents
* Utilizar `$event` com argumento de função no método pai (ver examplo de stateful abaixo `$ctrl.addTodo($event)`)
* Passar um objecto `$event: {}` para cima de um stateless component (ver exemplo de stateless abaixo `this.onAddTodo`).
  * Bonus : Utilizar um wrapper `EventEmitter` com `.value()` para replicar Angular2, evita criação manual do objecto `$event`
* Porquê? De forma a replicar o comportamento de Angular2 e manter consistência dentro de cada componente. Mantém também o estado (state) previsível.

**[Voltar ao topo](#tabela-de-conteúdos)**

### Stateful components

Vamos definir o que chamamos de "stateful component".

* Requer state (estado da aplicação), essencialmente comunicando com uma API backend através de um serviço
* Não altera estado da aplicação diretamente
* Faz o render de componentes filhos que alteram estado
* Também referidos como smart/container components

Um exemplo de um stateful component, completo com a sua definição low-level (apenas como demonstração, algum código foi omitido por razões de brevidade):

```js
/* ----- todo/todo.component.js ----- */
import controller from './todo.controller';

const TodoComponent = {
  controller,
  template: `
    <div class="todo">
      <todo-form
        todo="$ctrl.newTodo"
        on-add-todo="$ctrl.addTodo($event);"></todo-form>
      <todo-list
        todos="$ctrl.todos"></todo-list>
    </div>
  `
};

export default TodoComponent;

/* ----- todo/todo.controller.js ----- */
class TodoController {
  constructor(TodoService) {
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

TodoController.$inject = ['TodoService'];

export default TodoController;

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';

const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;

export default TodoModule;
```

Este exemplo demonstra um stateful component, que requer estado dentro de um controller, através de um serviço, e o passa para os seus filhos stateless components. De notar que não existem quaisquer directivas a serem utilizadas como `ng-repeat` dentro do template. Em vez disso, dados e funções são delegadas para `<todo-form>` e `<todo-list>` stateless components.

**[Voltar ao topo](#tabela-de-conteúdos)**

### Stateless components

Vamos definir o que chamamos de "stateless component".

* Tem input e outputs definidos utilizando `bindings: {}`
* Dados entram no componente através de attribute bindings (inputs)
* Dados saem do componente através de eventos (outputs)
* Altera estado, passa dados para cima quando necessário (como um click ou um evento submit)
* Não lhe interessa de onde os dados vêm originalmente, é stateless
* São componentes altamente reutilizáveis
* Também referidos como componentes dumb / de apresentação

Um exemplo de um stateless component (vamos utilizar `<todo-form>` como exemplo), completo com as suas definições de módulos low-level (apenas como demonstração, algum código foi omitido por razões de brevidade):

```js
/* ----- todo/todo-form/todo-form.component.js ----- */
import controller from './todo-form.controller';

const TodoFormComponent = {
  bindings: {
    todo: '<',
    onAddTodo: '&'
  },
  controller,
  template: `
    <form name="todoForm" ng-submit="$ctrl.onSubmit();">
      <input type="text" ng-model="$ctrl.todo.title">
      <button type="submit">Submit</button>
    </form>
  `
};

export default TodoFormComponent;

/* ----- todo/todo-form/todo-form.controller.js ----- */
class TodoFormController {
  constructor(EventEmitter) {
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

TodoFormController.$inject = ['EventEmitter'];

export default TodoFormController;

/* ----- todo/todo-form/todo-form.module.js ----- */
import angular from 'angular';
import TodoFormComponent from './todo-form.component';

const TodoFormModule = angular
  .module('todo.form', [])
  .component('todoForm', TodoFormComponent)
  .value('EventEmitter', payload => ({ $event: payload}))
  .name;

export default TodoFormModule; 
```

De notar como o componente `<todo-form>` não requer qualquer estado, apenas o recebe, muta (altera) um Objecto através do seu controller com a lógica associada, e o passa de volta para o componente pai através de property bindings. Neste exemplo lo lifecycle hook `$onChanges` faz um clone do binding inicial de `this.todo` e o reatribui, o que significa que os dados do componente pai não serão afetados até submetermos o form, juntamente com a nova sintaxe de fluxo de dados unidirecional `'<'`.

**[Voltar ao topo](#tabela-de-conteúdos)**

### Routed components

Vamos definir o que chamamos de "routed component".

* É essencialmente um stateful component, com definições de routing
* Não se utilizam mais ficheiros `router.js`
* Utilizamos Routed components para definir a sua lógica de routing
* Inputs para o componente é feito através de route resolve (opcional, mas ainda disponível no controller com chamadas a serviços)

Para este exemplo, vamos utilizar o componente `<todo>`, redefini-lo para utilizar uma definição de routing e `bindings` no componente que recebe dados (o segredo aqui como `ui-router` é o `resolve` das propriedades que criamos, neste caso `todoData` diretamente mapeado para `bindings`)
. Tratamo-lo como routed component porque é essencialmente uma "view":

```js
/* ----- todo/todo.component.js ----- */
import controller from './todo.controller';

const TodoComponent = {
  bindings: {
    todoData: '<'
  },
  controller,
  template: `
    <div class="todo">
      <todo-form
        todo="$ctrl.newTodo"
        on-add-todo="$ctrl.addTodo($event);"></todo-form>
      <todo-list
        todos="$ctrl.todos"></todo-list>
    </div>
  `
};

export default TodoComponent;

/* ----- todo/todo.controller.js ----- */
class TodoController {
  constructor() {}
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

export default TodoController;

/* ----- todo/todo.service.js ----- */
class TodoService {
  constructor($http) {
    this.$http = $http;
  }
  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}

TodoService.$inject = ['$http'];

export default TodoService;

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import TodoComponent from './todo.component';
import TodoService from './todo.service';

const TodoModule = angular
  .module('todo', [
    uiRouter
  ])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .config(($stateProvider, $urlRouterProvider) => {
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

export default TodoModule;
```

**[Voltar ao topo](#tabela-de-conteúdos)**

# Diretivas

### Diretivas Teoria

Diretivas dão-nos `template`, `scope` bindings, `bindToController`, `link` e muitas outras coisas. A utilização delas deve ser agora cuidadosamente avaliada sendo que existe `.component()`. Diretivas não devem criar templates e controllers, nem receber dados através de bindings. Diretivas devem ser utilizadas unicamente para decoração da DOM. Com isto, significa estender HTML existente - criado com `.component()`. De uma forma simples, se precisamos de DOM events/API's custom e lógica, utilizamos diretivas e fazemos bind a um template dentro de um componente. Se precisamos de pouca quantidade de manipulação da DOM existe o lifecycle hook `$postLink` a considerar, no entanto não é o local para onde se deve migrar toda a manipulação da DOM, utilizando uma Diretiva se possível para coisas que não sejam do âmbito de Angular.

Alguns conselhos quando utilizando diretivas:

* Nunca utilizar templates, scope, bindToController ou controllers
* Utilizar sempre `restrict: 'A'` com diretivas
* Utilizar compile e link quando necessário
* Lembrar de destruir e fazer unbind aos event handlers dentro de `$scope.$on('$destroy', fn);`

**[Voltar ao topo](#tabela-de-conteúdos)**

### Propriedades recomendadas

Devido ao facto de suportarem a maioria do que `.component()` suporta (template directives eram o componente original), recomendo limitar o Objecto de definição de directivas para apenas estas propriedades para evitar utilização de directivas incorrectamente:

| Propriedade | Utilizar? | Porquê |
|---|---|---|
| bindToController | Não | Utilizar `bindings` em componentes |
| compile | Sim | Para pre-compile DOM manipulation/events |
| controller | Não | Utilizar um componente |
| controllerAs | Não | Utilizar um componente |
| link functions | Sim | Para pre/post DOM manipulation/events |
| multiElement | Sim | [Ver docs](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Sim | [Ver docs](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | Não | Utilizar um componente |
| restrict | Sim | Definir utilização de diretivas, utilizar sempre `'A'` |
| scope | Não | Utilizar um componente |
| template | Não | Utilizar um componente |
| templateNamespace | Sim (se necessário) | [Ver docs](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | Não | Utilizar um componente |
| transclude | Não | Utilizar um componente |

**[Voltar ao topo](#tabela-de-conteúdos)**

### Classes e Constantes

Existem algumas formas de abordar ES2015 e diretivas, sendo com arrow function e easier assignment, ou utilizando uma ES2015 `Class`. Recomenda-se escolher que for melhor para a equipa, tendo em mente que Angular utiliza `Class`.

Um exemplo de utilização de uma constante com arrow function `() => ({})` devolvendo um Object literal (de notar as diferenças dentro de `.directive()`):

```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

const TodoAutoFocus = ($timeout) => ({
  restrict: 'A',
  link($scope, $element, $attrs) {
    $scope.$watch($attrs.todoAutofocus, (newValue, oldValue) => {
      if (!newValue) {
        return;
      }
      $timeout(() => $element[0].focus());
    });
  }
});

TodoAutoFocus.$inject = ['$timeout'];

export default TodoAutoFocus;

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';
import TodoAutofocus from './todo-autofocus.directive';

const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', TodoAutoFocus)
  .name;

export default TodoModule;
```

Ou utilizando ES2015 `Class` (de notar a chamada manual de `new TodoAutoFocus` ao registar a diretiva) para criar o Objecto:

```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

class TodoAutoFocus {
  constructor($timeout) {
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

TodoAutoFocus.$inject = ['$timeout'];

export default TodoAutoFocus;

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';
import TodoAutofocus from './todo-autofocus.directive';

const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', () => new TodoAutoFocus)
  .name;

export default TodoModule;
```

**[Voltar ao topo](#tabela-de-conteúdos)**

# Serviços

### Serviços Teoria

Serviços são essencialmente containers com lógica de negócio que os nossos componentes não deviam requisitar diretamente. Serviços contêm outros serviços built-in como `$http`, que depois podemos injetar para controllers de componentes noutros locais da nossa aplicação. Existem duas formas de utilizar serviços, através de `.service()` ou `.factory()`. Com ES2015 `Class`, devemos utilizar apenas`.service()`, completo com dependency injection annotation utilizando `$inject`.

**[Voltar ao topo](#tabela-de-conteúdos)**

### Classes para Serviços

Aqui está um exemplo de implementação da nossa `<todo>` app utilizando ES2015 `Class`:

```js
/* ----- todo/todo.service.js ----- */
class TodoService {
  constructor($http) {
    this.$http = $http;
  }
  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}

TodoService.$inject = ['$http'];

export default TodoService;

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';
import TodoService from './todo.service';

const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .name;

export default TodoModule;
```

**[Voltar ao topo](#tabela-de-conteúdos)**

# ES2015 e Tooling

##### ES2015

* Utilizar [Babel](https://babeljs.io/) para compilar código ES2015+ e outros polyfills
* Considerar utilizar [TypeScript](http://www.typescriptlang.org/) para abrir caminho para qualquer upgrade para Angular

##### Tooling
* Utilizar `ui-router` [ultima alpha](https://github.com/angular-ui/ui-router) (ver Readme) se queremos suportar component routing
  * Caso contrário está preso a `template: '<component>'` sem quaisquer `bindings`
* Considerar utilizar [Webpack](https://webpack.github.io/) para compilar o código ES2015
* Utilizar [ngAnnotate](https://github.com/olov/ng-annotate) para automaticamente anotar propriedades com `$inject`
* Como utilizar [ngAnnotate com ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/)

**[Voltar ao topo](#tabela-de-conteúdos)**

# State management

Considerar utilizar Redux com AngularJS 1.5 para data management.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[Voltar ao topo](#tabela-de-conteúdos)**

# Recursos

* [Compreender o método .component()](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Utilizar "require" com $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Compreender todos os lifecycle hooks, $onInit, $onChanges, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Utilizando "resolve" em routes](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Redux e Angular para state management](http://blog.rangle.io/managing-state-redux-angular/)
* [Exemplos de Aplicativos da Comunidade](https://github.com/chihab/angular-styleguide-sample)

**[Voltar ao topo](#tabela-de-conteúdos)**

# Documentação
Para qualquer outra informação, incluindo referências de API, consultar [Documentação de AngularJS](//docs.angularjs.org/api).

# Contribuições

Abrir primeiro um issue para discutir potenciais alterações/adições. Por favor não abrir issues para questões.

## Licença

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
