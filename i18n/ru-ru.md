# AngularJS руководство по стилю кода (ES2015)

### Архитектура, структура файлов, компоненты, односторонний обмен данными и лучшие практики

*Осмысленное руководство по стилю кода для разработчиков от [@toddmotto](//twitter.com/toddmotto)*

Архитектура и руководство были переписаны с нуля с использованием ES2015, изменений в AngularJS 1.5+, чтобы в будущем можно было легко обновить ваш проект на Angular. Это руководство сожержит лучшие практики по использованию одностороннего обмена данными, делегацию событий, компонентную архитектуру и маршрутизацию.

Старое руководство вы можете прочитать [тут](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5), и рассуждения о новом руководстве по стилю кода можно найти [тут](https://toddmotto.com/rewriting-angular-styleguide-angular-2).

> Присоединяйтесь к Ultimate AngularJS курсу, чтобы пройти путь от новичка к профессионалу и узнать про крутые возможности в Angular, а так же научиться создавать крутые, современные приложения, которые быстрые и масштабируемые.

<a href="https://courses.toddmotto.com" target="_blank"><img src="https://toddmotto.com/img/ua.png"></a>

## Содержание

  1. [Модульная архитектура](#Модульная-архитектура)
    1. [Теория](#Теория-о-модуле)
    1. [Главный/Корневой модуль](#Корневой-модуль)
    1. [Модуль Component](#Модуль-component)
    1. [Модуль Common](#Модуль-common)
    1. [Низкоуровневые модули](#Низкоуровневые-модули)
    1. [Конвенция наименования файлов](#Конвенция-наименования-файлов)
    1. [Масштабируемая структура файлов/папок](#Масштабируемая-структура-файлов)
  1. [Компоненты](#Компоненты)
    1. [Теория](#Теория-о-компонентах)
    1. [Поддерживаемые свойства](#Поддерживаемые-свойства)
    1. [Контроллеры](#Контроллеры)
    1. [События и односторонний обмен данными](#События-и-Односторонний-обмен-данными)
    1. [Stateful компоненты](#stateful-компоненты)
    1. [Stateless компоненты](#stateless-компоненты)
    1. [Компоненты маршрутизации](#Компоненты-маршрутизации)
  1. [Директивы](#Директивы)
    1. [Теория](#Теория-о-директивах)
    1. [Рекомендуемые свойства](#Рекомендуемые-свойства)
    1. [Константы или Классы](#Константы-или-Классы)
  1. [Сервисы](#Сервисы)
    1. [Теория](#Теория-о-сервисах)
    1. [Используем классы в сервисе](#Используем-классы-в-сервисе)
  1. [ES2015 и Утилиты](#ES2015-и-Утилиты)
  1. [Управление состоянием](#Управление-состоянием)
  1. [Ресурсы](#Ресурсы)
  1. [Документация](#Документация)
  1. [Помощь проекту](#Помощь-проекту)

# Модульная архитектура

Каждый модуль в Angular приложении - это модульный компонент. Модульный компонент - это определение для модуля который инкапсулирует логику, шаблоны, маршрутизацию и дочерние компоненты.

### Теория о модуле

Проектирование модуля напрямую зависит от структуры папки, которая проста в поддерживании и легко читаемая. В идеальном случае у нас должно быть три корневых модуля: root, component и common. Root модуль является нашим базовым модулем который собирает наше приложение в одно целое и соответствующие шаблоны. Мы импортируем наши component'ы и common модули в root модуль, чтобы подключить все зависимости. А component и common модули в свою очередь подключают низкоуровневые модули-компоненты, которые содержут наши компоненты, контроллеры, сервисы, директивы, фильтры и тесты для каждой фичи которую можно многоразово использовать.


**[Наверх](#содержание)**

### Корневой модуль

Корневой модуль состоит из корневого компоненты который задает базовый элемент для всего приложения, с исходящей маршрутизацией, пример показывает использование `ui-view` из `ui-router`.

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

Далее корневой модуль создан и экспортирован как`AppComponent`, после его импорта он так же зарегистрирован как `.component('app', AppComponent)`. Дальнейшие импорты сабмодулей(component и common модулей) сделаны, чтобы включить все необходимые компоненты для нашего приложения.

```js
// app.module.js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import AppComponent from './app.component';
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

**[Наверх](#содержание)**

### Модуль Component

Модуль Component - это контейнер с ссылками на все переиспользуемые компоненты. Смотрите ниже как мы импортируем `ComponentsModule` и внедряем его в Root модуль, таким образом мы получили единое место для импорта всех компонентов в приложение. Эти модули не связаны с другими модулями и это дает нам возможность переиспользовать их легко в других приложениях.

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

**[Наверх](#содержание)**

### Модуль Common

Модуль Common - это контейнер для всех компонентов с логикой для вашего приложения, эти компоненты мы не будем переиспользовать в другом приложении. Например, это могут быть вещи такие как: layout, навигация и footers. Смотрите далее как мы импортируем `CommonModule` и внедряем его в Root модуль, это дает нам возможность импортировать все наши common компоненты в одном месте.

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

**[Наверх](#содержание)**

### Низкоуровневые модули

Низкоуровневые модули - это отдельные компонентные модули которые содержат логику для конкретного блока в приложении, для фичи. Мы определяем модуль, который далее будет импортирован в выше стоящий модуль, такой как component или common модуль, смотрите пример ниже. Помните всегда добавлять суффикс `.name` к каждому `export` когда создаете _новый_ модуль, а не когда ссылаетесь на модуль. Так же вы можете заметить определение маршрутизации в примере, мы рассмотрим этот момент более детально в следующих главах этого руководства.

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

**[Наверх](#содержание)**

### Конвенция наименования файлов

Используйте простые имена в нижнем регистре, используйте имя компонента, например `calendar.*.js*`, `calendar-grid.*.js` - вместе с типом файла в середине. Используйте `*.module.js` как точка входа в модуль, так что вы сможете импортировать модуль используя имя директории.

```
calendar.module.js
calendar.controller.js
calendar.component.js
calendar.service.js
calendar.directive.js
calendar.filter.js
calendar.spec.js
```

**[Наверх](#содержание)**

### Масштабируемая структура файлов

Структура файлов проекта очень важная часть, эта секция описывает масштабируемую и предсказуемую структуру. Ниже показан пример файловой структуры для модульной компонентной архитектуры.

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

В корне проекта находится `index.html` и `app/`, директория в которой находятся все наши root, component, common и низкоуровневые модули.

**[BНаверх](#содержание)**

# Компоненты

### Теория о компонентах

Проще говоря, Компоненты - это шаблоны с контроллером. Компоненты _не являются_ директивами, и вы не должны заменять директивы компонентами, до тех пор пока вы не столкнулись с ситуацией когда надо улучшить шиблон директивы и добавить контроллер, которые лучше использовать вместе с компонентами. Компоненты так же имеют "связки" которые определяют направления для данных и событий, хуки и возможность использовать односторонний обмен данными и объекты события, для отправки данных обратно в родительский компонент. Это новый стандарт _по умолчанию_ начиная с AngularJS 1.5 и ниже. Все что имеет шаблон и контроллер, с уверенностью на 99%, должно быть компонентом, который может быть stateful, stateless(не зависимый от состояния) или маршрутизатором. Думайте о "компоненте", как о полноценном куске кода, не только как определение объекта `.component()`. Давайте рассмотрим лучшие практики и советы по использованию компонентов, и далее копнем глубже, чтобы понять как вы могли бы использовать компоненты как stateful, stateless и маршрутизатор.

**[Наверз](#содержание)**

### Поддерживаемые свойства

Ниже приведен список поддерживаемых свойст у `.component()` который вам следуем использовать:

| Свойство | Поддержка |
|---|---|
| bindings | Да, используйте только `'@'`, `'<'`, `'&'` |
| controller | Да |
| controllerAs | Да, по умолчанию это `$ctrl` |
| require | Да (new Object синтакс) |
| template | Да |
| templateUrl | Да |
| transclude | Да |

**[Наверх](#содержание)**

### Контроллеры

Контролеры должны использоваться вместе с компонентами, и больше нигде. Если вы видите, что вам нужен контроллер, знайте, вам на самом деле нужен stateless компонент, чтобы описать то или иное свецифическое поведение.

Далее вы найдете некоторые советы по использованию `Class` для контроллеров:

* Всегда используйте `constructor` если у вас есть внешние зависимости
* Не экспортируйте `Class` напрямую, экспортируйте имя для использования с`$inject` аннотациями
* Если вам необходимо достучаться до lexical scope, используйте arrow функции
* Как альтернатива для arrow функций, `let ctrl = this;` так же приемлемо и может быть более прозрачным в использовании
* Определите все публичные функции прямо в `Class`
* Используйте необходимые lifecycle хуки, `$onInit`, `$onChanges`, `$postLink` и `$onDestroy`
  * Внимание: `$onChanges` вызывается до `$onInit`, смотрите статью с дисскучией на эту тему в секции [ресурсы](#resources)
* Используйте `require` вместе с `$onInit` для ссылок на внутреннию/наследованную логику
* Не переопределяйте алиас `$ctrl` если используете `controllerAs` синтакс, но лучше забудьте про `controllerAs`

**[Наверх](#содержание)**

### События и Односторонний обмен данными

Односторонний обмен данными появился в AngularJS 1.5, и переопределяет то как взаимодействуют компоненты.

Далее приведен список советов по использованию одностороннего обмена данными:

* В компонентах которые получают данные, всегда используйте синтаксис одностороннего обмена данными `'<'`
* _Не_ используйте синтакс двухстороннего обмена данными`'='` больше нигде и никогда
* Компоненты в которых используется `bindings` должны использовать `$onChanges` и клонировать переданные данные путем одностороннего обмена данными, для удаления
 ссылки на переданные объекты и таким образом предвидеть обновление переданного объекта в родительском скопе.
* Используйте `$event` как аргумент функции в родительском методе (смотрите пример stateful компонента ниже `$ctrl.addTodo($event)`)
* Возвращайте `$event: {}` объект обратно из stateless компонента (смотрите пример stateless компонента ниже `this.onAddTodo`).
  * Бонус: Используйте `EventEmitter` обертку вместе с `.value()` прямо как в Angular, что позволит избежать ручного создания `$event` объекта
* Почему? Потому что так же сделано в Angular и это позволяет сохранить консистентность компонентов. Так же это позволяет предсказать состояние.

**[Наверх](#содержание)**

### Stateful компоненты

Давайте определим, что такое "stateful компонент".

* Получает состояние, в большинстве случаем через обзение с серверным API через сервис
* Напрямую не изменяет состояние
* Отрисовывает нижестоящие компоненты которые изменяют состояние
* Так же может называться умным компонентом или компонентом контейнером

Пример stateful компонента, вместе с определением низкоуровневого модуля (этот пример создан только для демонстрации, и лишний код был убран для краткости):

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

Этот пример показывает stateful компонент, который получает состояние внутри контроллера, с помощью сервиса, и передает изменения далее в ниже стоящие stateless компоненты. Заметьте как мы обошлись без использования директив, таких как `ng-repeat` и его друзей внутри шаблона. Вместо этого, данные и функции переданы в `<todo-form>` и `<todo-list>` - stateless компоненты.

**[Наверх](#содержание)**

### Stateless компоненты

Давайте определим, что такое "stateless компонент".

* Имеет определение точек входа и выхода данных через `bindings: {}`
* Данные поступают в компонент через аттрибуты (inputs)
* Данные уходят из компонента путем событий (outputs)
* Изменяет состояние, передает данные обратно вверх по запросу (к примеру клик или событие сабмита формы)
* Ему все равно откуда пришли данные, этот компонент stateless
* Такой компонент является переиспользуемым, можно использовать в других приложениях
* Так же известен как простой компонент или компонент вида(вьюшки, презентации)

Пример stateless компонента (давайте возьмем `<todo-form>` как пример), вместе с низкоуровневым определением модуля (этот пример создан только для демонстрации, и лишний код был убран для краткости):

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
    // с EventEmitter оберткой
    this.onAddTodo(
      this.EventEmitter({
        todo: this.todo
      })
    );
    // без EventEmitter обертки
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

Обратите внимание как `<todo-form>` компонент использует состояние просто получив его, изменяет Object путем логики контроллера ассоциированной с ним, и передает обратно в родительский компонент через свойство в bindings. В данном примере `$onChanges` хук клонирует `this.todo` Object и переприсваивает его. С помощью такого финта мы убираем ссылку на объект в родительском компоненте, чтобы случайно не обновить его. Мы используем синтаксис одностороннего обмена данным `'<'`.

**[Наверх](#содержание)**

### Компоненты маршрутизации

Давайте определим, что такое "routed component".

* В целом это stateful компонент, с определением маршрутов/навигации
* Забудьте про `router.js` файлы
* В компонентах маршрутизации мы можем писать логику связанную с навигацией и маршрутизацией
* Данные поступают в компонент через route resolve (опционально, можно продолжить использовать сервисы в контроллере)

Для примера мы возьмем `<todo>` компонент, изменим его добавив маршруты и `bindings` для компонентов которые получают данные (фишка с `ui-router` в использовании свойства `resolve`, в нашем примере `todoData` напрямую передается через `bindings`). Мы называем этот компонент маршрутизатором потому, что по существу это "view":

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

**[Наверх](#содержание)**

# Директивы

### Теория о директивах

Директивы дают нам возможность использовать `template`, `scope` связки, `bindToController`, `link` и множество других вещей. Использование всех этих свойств должно быть тщательно обдуманно, так как теперь есть `.component()`. Директивы больше не должны определять шаблоны и контроллеры или получать данные через bindings. Директивы должны быть использованы теперь только для декорирования DOM. Т.е. расширение существующего HTML - созданного с помощью `.component()`. Проще говоря, если вам нужны специфические DOM события/API и логика, используйте директиву и присоедините ее внутри шаблона который используется в компоненте. Если вам нужно внушительное количество манипуляций с DOM, используйте `$postLink` хук, но помните, что это не то место куда надо перенести все ваши манипуляции с DOM. Используйте директиву, если можете, для не-Angular фишек.

Список некоторых полезных советов по использованию директив:

* Больше никогда не используйте templates, scope, bindToController или controllers
* Испоьзуйте всегда `restrict: 'A'` вместе с директивами
* Используйте compile и link где необходимо
* Помните destroy и unbind обрабочики событий внутри `$scope.$on('$destroy', fn);`

**[Наверх](#содержание)**

### Рекомендуемые свойства

Из-за того что директивы поддерживают большинство свойст из `.component()` (шаблон директивы был изначально компонентом), я рекомендую ограничить стек свойст директивы для использования, чтобы использовать директивы правильно:

| Свойство | Использовать? | Почему |
|---|---|---|
| bindToController | Нет | Используйте `bindings` в компонентах |
| compile | Да | Для pre-compile DOM манипуляций и событий |
| controller | Нет | Используйте компонент |
| controllerAs | Нет | Используйте компонент |
| link functions | Да | Для pre/post DOM манипуляций и событий |
| multiElement | Да | [Смотрите документацию](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Да | [Смотрите документацию](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | Нет | Используйте компонент |
| restrict | Да | Определяет как будет использоваться директива, всегда используйте `'A'` |
| scope | Нет | Используйте компонент |
| template | Нет | Используйте компонент |
| templateNamespace | Да (если не обойтись уж никак) | [Смотрите документацию](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | Нет | Используйте компонент |
| transclude | Нет | Используйте компонент |

**[Наверх](#содержание)**

### Константы или Классы

Существует несколько путей совместного использования ES2015 и директив, с помощью arrow функций и легкого назначения, или используя ES2015 `Class`. Выберите то, что лучше всего подходит вам и вашей команде, но помните, что Angular использует `Class`. Далее рассмотрим пример использования констант с arrow функцией, обертка выражения `() => ({})` возвращает Object литерал (обратите внимание на разницу в использовании внутри `.directive()`):

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

Или с использование ES2015 ключего слова `Class` (обратите внимание на ручное создание объекта с помощью `new TodoAutoFocus` когда регистрируем директиву) для создания объекта:

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

**[Наверх](#содержание)**

# Сервисы

### Теория о сервисах

Сервисы по существу это контейнеры для бизнес логики, которую наши компоненты не должны запрашивать/использовать напрямую. Сервисы используют другие встроенные или внешние сервисы, например `$http`, которые мы можем внедрить в контроллер компонентов где угодно в нашем приложении. Существуют два пути создания сервисов, используя `.service()` или `.factory()`. Используя ES2015 ключевое слово `Class`, мы должны использовать только `.service()`, вместе с dependency injection аннотацией используя `$inject`.

**[Назад](#содержание)**

### Используем классы в сервисе

Ниже приведен пример нашего `<todo>` используя ES2015 ключевое слово`Class`:

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

**[Наверх](#содержание)**

# ES2015 и Утилиты

##### ES2015

* Используйте [Babel](https://babeljs.io/) чтобы скомпилировать ES2015+ код и любые полифилы
* Не стесняйтесь использовать [TypeScript](http://www.typescriptlang.org/) чтобы легче прошел переход на Angular

##### Утилиты
* Используйте `ui-router` [последняя alpha](https://github.com/angular-ui/ui-router) (смотрите Readme) если вы хотите использовать компонентную маршрутизацию
  * В другом случае вы застряли с `template: '<component>'` без `bindings`
* Начинайте использовать [Webpack](https://webpack.github.io/) для компиляции вашего ES2015 кода
* Используйте [ngAnnotate](https://github.com/olov/ng-annotate) для автоматической аннотации `$inject` свойств
* Как использовать [ngAnnotate вместе с ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/#ng-annotate)

**[Наверх](#содержание)**

# Управление состоянием

Начните использовать Redux вместе с AngularJS 1.5 для управления данными.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[Назад](#содержание)**

# Ресурсы

* [Объяснение метода .component()](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Использование "require" вместе с $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Объяснение всех lifecycle хуков, $onInit, $onChanges, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Использование "resolve" в маршрутах](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Управление состоянием в Angular с помощью Redux](http://blog.rangle.io/managing-state-redux-angular/)
* [Sample Application from Community](https://github.com/chihab/angular-styleguide-sample)

**[Наверх](#содержание)**

# Документация
Все что было упущено, можно найти в [AngularJS документации](//docs.angularjs.org/api).

# Помощь проекту

Создайте issue, чтобы обсудить возможные изменения/добавления. Пожалуйста, не создавайте issue ради вопросов.

## Лицензия

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
