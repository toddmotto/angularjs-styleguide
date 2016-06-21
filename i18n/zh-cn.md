# Angular 1.x 编码风格指南(ES2015)

### 架构, 文件结构, 组件, 单向数据流以及最佳实践

*来自 [@toddmotto](//twitter.com/toddmotto) 团队的编码指南*

Angular 的编码风格以及架构已经使用ES2015进行重写,这些在Angular 1.5+的变化可以更好帮助您的更好的升级到Angular2.。
这份指南包括了新的单向数据流的时间，事件委托，组件架构和组件路由。

老版本的指南你可以在[这里](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5)找到,  在这里[here](https://toddmotto.com/rewriting-angular-styleguide-angular-2)看到最新的.

> 加入我们无与伦比的angularjs学习体验中来，构建更加快速和易于扩展的App.

<a href="https://courses.toddmotto.com" target="_blank"><img src="https://toddmotto.com/img/ua.png"></a>

## 目录

  1. [模块架构](#module-architecture)
    1. [基本理论](#module-theory)
    1. [根 模块](#root-module)
    1. [组件模块](#component-module)
    1. [公共模块](#common-module)
    1. [低 级别模块](#low-level-modules)
    1. [可扩展的文件结构](#scalable-file-structure)
    1. [文件命名规范](#file-naming-conventions)
  1. [组件](#components)
    1. [基本里路](#component-theory)
    1. [支持的属性](#supported-properties) 
    1. [控制器](#controllers)
    1. [单向数据流和事件](#one-way-dataflow-and-events)
    1. [状态组件](#stateful-components)
    1. [无状态组件](#stateless-components)
    1. [路由组件](#routed-components)
  1. [指令](#directives)
    1. [基本理论](#directive-theory)
    1. [建议的属性](#recommended-properties) 
    1. [常量和类](#constants-or-classes)
  1. [服务](#services)
    1. [基本理论](#service-theory)
    1. [Service](#classes-for-service)
  1. [ES2015 和工具推荐](#es2015-and-tooling)
  1. [状态管理](#state-management)
  1. [资源](#resources)
  1. [文档](#documentation)
  1. [贡献](#contributing)

# Modular architecture

Angular 中的每一个模块都是一个模块组件。一个模块组件囊括了逻辑，模版，路由和子组件。

### Module theory

在模块的设计直接反映到我们的文件夹结构，从而保证我们项目的可维护性和可预测性。
我们最好应该有三个高层次的模块：根，组件和常用模块。根模块定义了用于启动App和相应的模板的基本架子。
然后，我们导入需要依赖的组件和通用模块。组件和通用模块然后需要低级别的组件模块，其中包含我们的组件，控制器，服务，指令，过滤器和给可重复使用的功能进行测试。

**[返回顶部](#table-of-contents)**

### 根模块

根模块会启动一个根组件，整个组件主要定义了整个应用的基本的元素和路由出口，例如使用`ui-view`和`ui-router`。

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

我们导入`AppComponent`并且使用`.component（"app"，AppComponent）`完成注册即表示一个根模块创建完成。
更进一步我们会导入一些子模块（组件和通用模块）用于引入相关的组件。

```js
// app.js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import AppComponent from './app.component';
import Components from './components/components';
import Common from './common/common';

const root = angular
  .module('app', [
    Components,
    Common,
    uiRouter
  ])
  .component('app', AppComponent);

export default root;
```

**[返回顶部](#table-of-contents)**

### 组件模块

一个组件模块就是引用所有课重复使用的组件容器。上面我们可以了解我们如何导入组件和将它们注入到根模块，
这样我么可以有一个地方导入所有应用程序需要的组件。
我们要求这些模块从所有其它模块分离出来，这样这些模块可以应用到其它的应用程序中。

```js
import angular from 'angular';
import Calendar from './calendar';
import Events from './events';

const components = angular
  .module('app.components', [
    Calendar,
    Events
  ])
  .name;

export default components;
```

**[Back to top](#table-of-contents)**

### 公共模块

公共模块为所有的应用提供一些特殊组件的引用，我们不希望它能够在另一个应用程序中使用。比如布局，导航和页脚。
前面我们已经知道如何导入`Common`并将其注入到根模块，而这里就是我们导入所有通用组件的地方。
```js
import angular from 'angular';
import Nav from './nav';
import Footer from './footer';

const common = angular
  .module('app.common', [
    Nav,
    Footer
  ])
  .name;

export default common;
```

**[Back to top](#table-of-contents)**

### Low-level modules

低层次的模块是一些独立的组件，它们包含逻辑和功能。这些将分别定义成模块，被引入到较高层次模块中，
比如如一个组件或通用模块。一定要记住每次创建一个新的模块时(并非引用)，记得在`export`中添加后缀。你会注意到路由定义也存在这里，我们将在随后的部分讲到它。
```js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import CalendarComponent from './calendar.component';

const calendar = angular
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

export default calendar;
```

**[Back to top](#table-of-contents)**

# 文件命名规范

使用小写并保持命名的见解, 比如使用组件名称时, e.g. `calendar.*.js*`, `calendar-grid.*.js` - 将名称放到中间. 使用 `index.js` 作为模块的定义文件 ，这样你就可以直接通过目录引入了。

```
index.js
calendar.controller.js
calendar.component.js
calendar.service.js
calendar.directive.js
calendar.filter.js
calendar.spec.js
```

**[Back to top](#table-of-contents)**

### 易于扩展的文件结构

文件目录结构实际上十分重要，它有利于我们更好的扩展和前瞻。下面的例子展示了模块组件的基本架构。

```
├── app/
│   ├── components/
│   │  ├── calendar/
│   │  │  ├── index.js
│   │  │  ├── calendar.controller.js
│   │  │  ├── calendar.component.js
│   │  │  ├── calendar.service.js
│   │  │  ├── calendar.spec.js
│   │  │  └── calendar-grid/
│   │  │     ├── index.js
│   │  │     ├── calendar-grid.controller.js
│   │  │     ├── calendar-grid.component.js
│   │  │     ├── calendar-grid.directive.js
│   │  │     ├── calendar-grid.filter.js
│   │  │     └── calendar-grid.spec.js
│   │  └── events/
│   │     ├── index.js
│   │     ├── events.controller.js
│   │     ├── events.component.js
│   │     ├── events.directive.js
│   │     ├── events.service.js
│   │     ├── events.spec.js
│   │     └── events-signup/
│   │        ├── index.js
│   │        ├── events-signup.controller.js
│   │        ├── events-signup.component.js
│   │        ├── events-signup.service.js
│   │        └── events-signup.spec.js
│   ├── common/
│   │  ├── nav/
│   │  │     ├── index.js
│   │  │     ├── nav.controller.js
│   │  │     ├── nav.component.js
│   │  │     ├── nav.service.js
│   │  │     └── nav.spec.js
│   │  └── footer/
│   │        ├── index.js
│   │        ├── footer.controller.js
│   │        ├── footer.component.js
│   │        ├── footer.service.js
│   │        └── footer.spec.js
│   ├── app.js
│   └── app.component.js
└── index.html
```

顶级目录 仅仅包含了 `index.html` 以及 `app/`, 而在`app/`目录中则包含了我们要用到的组件，公共模块，以及低级别的模块。

**[Back to top](#table-of-contents)**

# 组件

### 组件的基本概念

组件实际上就是带有控制器的模板。他们即不是指令，也不应该使用组件代替指令，除非你正在用控制器升级“模板指令”，
部件还包含定义为数据和⌚️输入与输出，生命周期钩子和使用单向数据流和从父组件上获取数据的事件对象。
获取数据备份到父组件的能力的输入和输出的绑定。这些都是在Angular 1.5及以上退出的新标准。
我们创建的一切模板，控制器都可能是一个组件，它们可能是是有状态的，无状态或路由组件。
你可以把一个“部件”作为一个完整的一段代码，而不仅仅是`.component（）`定义的对象。
让我们来探讨一些组件最佳实践和建议，然后让你明白应该如何组织他们。


**[Back to top](#table-of-contents)**

### Supported properties

下面是一些`.component()`你可能会使用到的属性 :

| Property | Support |
|---|---|
| bindings | Yes, 仅仅使用 `'@'`, `'<'`, `'&'`  |
| controller | Yes |
| controllerAs | Yes, 默认 `$ctrl` |
| require | Yes (new Object syntax) |
| template | Yes |
| templateUrl | Yes |
| transclude | Yes |

**[Back to top](#table-of-contents)**

### 控制器

控制器应该只与组件一起使用。如果你觉得你需要一个控制器，你真正需要的可能是一个无状态的组件来管理特定的行为。

这里有一些使用`Class`构建controller的建议:

* 始终使用 `constructor` 用于依赖注入；
* 不要直接导出 `Class`,导出它的名称，并允许`$inject`;
* 如果你需访问到 scope 里的语法，使用箭头函数；
* 另外关于箭头 函数, `let ctrl = this;` 也是可以接受的，当然这更取决于使用场景；
* 绑定到所有公共函数到`Class`上；
* 适当的利用生命周期的一些钩子, `$onInit`, `$onChanges`, `$postLink` 以及`$onDestroy`。
  * 注意: `$onChanges` 是 `$onInit`之后调用的, 这里 [resources](#resources) 有更深一步的讲解。
*  依靠 `$onInit`使用`require`  以便引用继承的逻辑；
* 不要覆盖默认的 `$ctrl`  使用`controllerAs` 语法 起的别名, 当然也不要在别的地方使用 `controllerAs` 

**[Back to top](#table-of-contents)**

### One-way dataflow and Events

单向数据流已经在Angular1.5中引入了，并且重新定义了组件之间的通信。

关于单向数据流的一些小建议:

* 在组件接受数据，始终使用 单向数据绑定符号`'<'`
* 不要再使用 `'='`双向的数据绑定的语法
* 拥有绑定的组件应该使用$ onChanges克隆单向绑定的数据阻止对象通过引用传递和更新原数据
* 使用 `$event` 作为一个父级方法中的的一个函数参数(参见有状态的例子 `$ctrl.addTodo($event)`)
* 传递一个`$event: {}` 从无状态的组件中进行备份(参见无状态的例子 `this.onAddTodo`).
  * Bonus: 使用 包裹 `.value()` 的 `EventEmitter` 以便迁到Angular2 , 避免手动创建一个 `$event`
* 为什么? 这和Angular2类似并且保持组件的一致性.并且可以让状态可预测。

**[Back to top](#table-of-contents)**

### 有状态的组件

我们定义什么事的“有状态的组件”

* 获取状态，通过服务与后端API通信
* 不直接发生状态变化
* 渲染发生状态变化的子组件
* 作为一个组件容器的引用

下面的是一个状态组件案例，它和一个低级别的模块组件共同完成（这只是演示，为了精简省略的一些代码）
```js
/* ----- todo/todo.component.js ----- */
import controller from './todo.controller';

const TodoComponent = {
  controller,
  template: `
    <div class="todo">
      <todo-form 
        todo="$ctrl.newTodo"
        on-add-todo="$ctrl.addTodo($event);">
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
    this.todoService.getTodos.then(response => this.todos = response);
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

/* ----- todo/index.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';

const todo = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;

export default todo;
```

这个例子显示了一个有状态的组件，在控制器哪通过服务获取状态，然后再将它传递给无状态的子组件。注意这里并没有在模版使用指令比如`ng-repeat`以及其他指令，相反，数据和功能委托到`<todo-form> `和 `<todo-list>`这两个无状态的组件。

**[Back to top](#table-of-contents)**

### 无状态的组件

Let's define what we'd call a "stateless component".

* Has defined inputs and outputs using `bindings: {}`
* Data enters the component through attribute bindings (inputs)
* Data leaves the component through events (outputs)
* Mutates state, passes data back up on-demand (such as a click or submit event)
* Doesn't care where data comes from, it's stateless
* Are highly reusable components
* Also referred to as dumb/presentational components

An example of a stateless component (let's use `<todo-form>` as an example), complete with it's low-level module definition (this is only for demonstration, so some code has been omitted for brevity):

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
  constructor(EventEmitter) {}
  $onChanges(changes) {
    if (changes.todo) {
      this.todo = Object.assign({}, this.todo);
    }
  }
  onSubmit() {
    if (!this.todo.title) return;
    // with EventEmitter wrapper
    this.onAddTodo(
      EventEmitter({
        todo: this.todo
      });
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

/* ----- todo/todo-form/index.js ----- */
import angular from 'angular';
import TodoFormComponent from './todo-form.component';

const todoForm = angular
  .module('todo')
  .component('todo', TodoFormComponent)
  .value('EventEmitter', payload => ({ $event: payload});

export default todoForm;
```

Note how the `<todo-form>` component fetches no state, it simply receives it, mutates an Object via the controller logic associated with it, and passes it back to the parent component through the property bindings. In this example, the `$onChanges` lifecycle hook makes a clone of the initial `this.todo` binding Object and reassigns it, which means the parent data is not affected until we submit the form, alongside one-way data flow new binding syntax `'<'`.

**[Back to top](#table-of-contents)**

### Routed components

Let's define what we'd call a "routed component".

* It's essentially a stateful component, with routing definitions
* No more `router.js` files
* We use Routed components to define their own routing logic
* Data "input" for the component is done via the route resolve (optional, still available in the controller with service calls)

For this example, we're going to take the existing `<todo>` component, refactor it to use a route definition and `bindings` on the component which receives data (the secret here with `ui-router` is the `resolve` properties we create, in this case `todoData` directly map across to `bindings` for us). We treat it as a routed component because it's essentially a "view":

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
        on-add-todo="$ctrl.addTodo($event);">
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

/* ----- todo/index.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';
import TodoService from './todo.service';

const todo = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .config(($stateProvider, $urlRouterProvider) => {
    $stateProvider
      .state('todos', {
        url: '/todos',
        component: 'todo',
        resolve: {
          todoData: TodoService => TodoService.getTodos();
        }
      });
    $urlRouterProvider.otherwise('/');
  })
  .name;

export default todo;
```

**[Back to top](#table-of-contents)**

# Directives

### Directive theory

Directives gives us `template`, `scope` bindings, `bindToController`, `link` and many other things. The usage of these should be carefully considered now `.component()` exists. Directives should not declare templates and controllers anymore, or receive data through bindings. Directives should be used solely for decorating the DOM. By this, it means extending existing HTML - created with `.component()`. In a simple sense, if you need custom DOM events/APIs and logic, use a Directive and bind it to a template inside a component. If you need a sensible amount of DOM manipulation, there is also the `$postLink` lifecycle hook to consider, however this is not a place to migrate all your DOM manipulation to, use a Directive if you can for non-Angular things.

Here are some advisories for using Directives:

* Never use templates, scope, bindToController or controllers
* Always `restrict: 'A'` with Directives
* Use compile and link where necessary
* Remember to destroy and unbind event handlers inside `$scope.$on('$destroy', fn);`

**[Back to top](#table-of-contents)**

### Recommended properties

Due to the fact directives support most of what `.component()` does (template directives were the original component), I'm recommending limiting your directive Object definitions to only these properties, to avoid using directives incorrectly:

| Property | Use it? | Why |
|---|---|---|
| bindToController | No | Use `bindings` in components |
| compile | Yes | For pre-compile DOM manipulation/events |
| controller | No | Use a component |
| controllerAs | No | Use a component |
| link functions | Yes | For pre/post DOM manipulation/events |
| multiElement | Yes | [See docs](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Yes | [See docs](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | No | Use a component |
| restrict | Yes | Defines directive usage, always use `'A'` |
| scope | No | Use a component |
| template | No | Use a component |
| templateNamespace | Yes (if you must) | [See docs](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | No | Use a component |
| transclude | No | Use a component |

**[Back to top](#table-of-contents)**

### Constants or Classes

There are a few ways to approach using ES2015 and directives, either with an arrow function and easier assignment, or using an ES2015 `Class`. Choose what's best for you or your team, keep in mind Angular 2 uses `Class`.

Here's an example using a constant with an Arrow function an expression wrapper `() => ({})` returning an Object literal (note the usage differences inside `.directive()`):

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

/* ----- todo/index.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';
import TodoAutofocus from './todo-autofocus.directive';

const todo = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', TodoAutoFocus)
  .name;

export default todo;
```

Or using ES2015 `Class` (note manually calling `new TodoAutoFocus` when registering the directive) to create the Object:

```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

class TodoAutoFocus {
  constructor() {
    this.restrict = 'A';
  }
  link($scope, $element, $attrs) {
    $scope.$watch($attrs.todoAutofocus, (newValue, oldValue) => {
      if (!newValue) {
        return;
      }
      $timeout(() => $element[0].focus());
    });
  }
}

TodoAutoFocus.$inject = ['$timeout'];

export default TodoAutoFocus;

/* ----- todo/index.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';
import TodoAutofocus from './todo-autofocus.directive';

const todo = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', () => new TodoAutoFocus)
  .name;

export default todo;
```

**[Back to top](#table-of-contents)**

# Services

### Service theory

Services are essentially containers for business logic that our components shouldn't request directly. Services contain other built-in or external services such as `$http`, that we can then inject into component controllers elsewhere in our app. We have two ways of doing services, using `.service()` or `.factory()`. With ES2015 `Class`, we should only use `.service()`, complete with dependency injection annotation using `$inject`.

**[Back to top](#table-of-contents)**

### Classes for Service

Here's an example implementation for our `<todo>` app using ES2015 `Class`:

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

/* ----- todo/index.js ----- */
import angular from 'angular';
import TodoComponent from './todo.component';
import TodoService from './todo.service';

const todo = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .name;

export default todo;
```

**[Back to top](#table-of-contents)**

# ES2015 and Tooling

##### ES2015

* Use [Babel](https://babeljs.io/) to compile your ES2015+ code and any polyfills
* Consider using [TypeScript](http://www.typescriptlang.org/) to make way for any Angular 2 upgrades

##### Tooling
* Use `ui-router` [latest alpha](https://github.com/angular-ui/ui-router) (see the Readme) if you want to support component-routing
  * Otherwise you're stuck with `template: '<component>'` and no `bindings`
* Consider using [Webpack](https://webpack.github.io/) for compiling your ES2015 code
* Use [ngAnnotate](https://github.com/olov/ng-annotate) to automatically annotate `$inject` properties
* How to use [ngAnnotate with ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/)

**[Back to top](#table-of-contents)**

# State management

Consider using Redux with Angular 1.5 for data management.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[Back to top](#table-of-contents)**

# Resources

* [Understanding the .component() method](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Using "require" with $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Understanding all the lifecycle hooks, $onInit, $onChange, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Using "resolve" in routes](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Redux and Angular state management](http://blog.rangle.io/managing-state-redux-angular/)

**[Back to top](#table-of-contents)**

# Documentation
For anything else, including API reference, check the [Angular documentation](//docs.angularjs.org/api).

# Contributing

Open an issue first to discuss potential changes/additions. Please don't open issues for questions.

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
