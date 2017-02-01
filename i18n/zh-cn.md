# AngularJS 编码风格指南(ES2015)

### 架构, 文件结构, 组件, 单向数据流以及最佳实践

*来自 [@toddmotto](//twitter.com/toddmotto) 团队的编码指南*

Angular 的编码风格以及架构已经使用ES2015进行重写,这些在AngularJS 1.5+的变化可以更好帮助您的更好的升级到Angular2.。
这份指南包括了新的单向数据流，事件委托，组件架构和组件路由。

老版本的指南你可以在[这里](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5)找到,  在[这里](https://toddmotto.com/rewriting-angular-styleguide-angular-2)你能看到最新的.

> 加入我们无与伦比的angularjs学习体验中来，构建更加快速和易于扩展的App.

<a href="https://courses.toddmotto.com" target="_blank"><img src="https://toddmotto.com/img/ua.png"></a>

## 目录

  1. [模块架构](#module-architecture)
    1. [基本概念](#module-theory)
    1. [根 模块](#root-module)
    1. [组件模块](#component-module)
    1. [公共模块](#common-module)
    1. [低级别模块](#low-level-modules)
    1. [可扩展的文件结构](#scalable-file-structure)
    1. [文件命名规范](#file-naming-conventions)
  1. [组件](#components)
    1. [基本概念](#component-theory)
    1. [支持的属性](#supported-properties)
    1. [控制器](#controllers)
    1. [单向数据流和事件](#one-way-dataflow-and-events)
    1. [状态组件](#stateful-components)
    1. [无状态组件](#stateless-components)
    1. [路由组件](#routed-components)
  1. [指令](#directives)
    1. [基本概念](#directive-theory)
    1. [建议的属性](#recommended-properties)
    1. [常量和类](#constants-or-classes)
  1. [服务](#services)
    1. [基本概念](#service-theory)
    1. [服务](#classes-for-service)
  1. [ES2015 和工具推荐](#es2015-and-tooling)
  1. [状态管理](#state-management)
  1. [资源](#resources)
  1. [文档](#documentation)
  1. [贡献](#contributing)

# Modular architecture

Angular 中的每一个模块都是一个模块组件。一个模块组件囊括了逻辑，模版，路由和子组件。

### Module 基本概念

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

**[返回顶部](#table-of-contents)**

### 组件模块

一个组件模块就是引用所有课重复使用的组件容器。上面我们可以了解我们如何导入组件和将它们注入到根模块，
这样我么可以有一个地方导入所有应用程序需要的组件。
我们要求这些模块从所有其它模块分离出来，这样这些模块可以应用到其它的应用程序中。

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

**[返回目录](#table-of-contents)**

### 公共模块

公共模块为所有的应用提供一些特殊组件的引用，我们不希望它能够在另一个应用程序中使用。比如布局，导航和页脚。
前面我们已经知道如何导入`CommonModule`并将其注入到根模块，而这里就是我们导入所有通用组件的地方。
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

**[返回目录](#table-of-contents)**

### 低级别的模块

低层次的模块是一些独立的组件，它们包含逻辑和功能。这些将分别定义成模块，被引入到较高层次模块中，
比如一个组件或通用模块。一定要记住每次创建一个新的模块时(并非引用)，记得在`export`中添加后缀。你会注意到路由定义也是在这里，我们将在随后的部分讲到它。
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

**[返回目录](#table-of-contents)**

# 文件命名规范

使用小写并保持命名的简介, 比如使用组件名称时, e.g. `calendar.*.js*`, `calendar-grid.*.js` - 将名称放到中间. 使用 `*.module.js` 作为模块的定义文件 ，这样你就可以直接通过目录引入了。

```
calendar.module.js
calendar.controller.js
calendar.component.js
calendar.service.js
calendar.directive.js
calendar.filter.js
calendar.spec.js
```

**[返回目录](#table-of-contents)**

### 易于扩展的文件结构

文件目录结构实际上十分重要，它有利于我们更好的扩展和预测。下面的例子展示了模块组件的基本架构。

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

顶级目录 仅仅包含了 `index.html` 以及 `app/`, 而在`app/`目录中则包含了我们要用到的组件，公共模块，以及低级别的模块。

**[返回目录](#table-of-contents)**

# 组件

### 组件的基本概念

组件实际上就是带有控制器的模板。他们即不是指令，也不应该使用组件代替指令，除非你正在用控制器升级“模板指令”，
组件还包含数据事件的输入与输出，生命周期钩子和使用单向数据流以及从父组件上获取数据的事件对象。
从父组件获取数据备份。这些都是在AngularJS 1.5及以上推出的新标准。
我们创建的一切模板，控制器都可能是一个组件，它们可能是是有状态的，无状态或路由组件。
你可以把一个“部件”作为一个完整的一段代码，而不仅仅是`.component（）`定义的对象。
让我们来探讨一些组件最佳实践和建议，然后你应该可以明白如何组织他们。


**[返回目录](#table-of-contents)**

### 支持的属性

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

**[返回目录](#table-of-contents)**

### 控制器

控制器应该只与组件一起使用。如果你觉得你需要一个控制器，你真正需要的可能是一个无状态的组件来管理特定的行为。

这里有一些使用`Class`构建controller的建议:

* 始终使用 `constructor` 用于依赖注入；
* 不要直接导出 `Class`,导出它的名称，并允许`$inject`;
* 如果你需访问到 scope 里的语法，使用箭头函数；
* 另外关于箭头 函数, `let ctrl = this;` 也是可以接受的，当然这更取决于使用场景；
* 绑定到所有公共函数到`Class`上；
* 适当的利用生命周期的一些钩子, `$onInit`, `$onChanges`, `$postLink` 以及`$onDestroy`。
  * 注意: `$onChanges` 是 `$onInit`之后调用的, 这里 [扩展阅读](#resources) 有更深一步的讲解。
*  在`$onInit`使用`require`  以便引用继承的逻辑；
* 不要覆盖默认  `$ctrl`  使用`controllerAs` 起的别名, 当然也不要在别的地方使用 `controllerAs`

**[返回目录](#table-of-contents)**

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

**[返回目录](#table-of-contents)**

### 有状态的组件

什么是“有状态的组件”

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

这个例子显示了一个有状态的组件，在控制器哪通过服务获取状态，然后再将它传递给无状态的子组件。注意这里并没有在模版使用指令比如`ng-repeat`以及其他指令，相反，数据和功能委托到`<todo-form> `和 `<todo-list>`这两个无状态的组件。

**[返回目录](#table-of-contents)**

### 无状态的组件

什么是无状态的组件

* 使用`bindings: {}` 定义了输入和输出;
* 数据通过属性绑定进入到组件内
* 数据通过事件离开组件
* 状态变化，会将数据进行备份 (比如触发点击和提交事件)
* 并不需要关心的数据来自哪里
* 可高度重复利用的组件
* 也被称作无声活着表面组件

下面是一个无状态组件的例子 (我们使用`<todo-form>` 作为例子) (仅仅用于演示，省略了部分代码):

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

请注意`<todo-form>`组件不获取状态，它只是接收，它通过控制器的逻辑去改变一个对象然后通过绑定的属性将改变后的值传回给父组件。
在这个例子中，`$onChanges`周期钩子 产生一个`this.todo`的对象克隆并重新分配它，这意味着原数据不受影响，直到我们提交表单，沿着单向数据流的新的绑定语法'<' 。

**[返回目录](#table-of-contents)**

### 路由组件

什么是路由组件

* 它本质上是个有状态的组件，具备路由定义
* 没有`router.js` 文件
*我们使用路由组件去定义它自己的路由逻辑
*数据流入到组件是通过路由分解获得 (当然在控制器中我们通过服务获得)

在这个例子中，我们将利用现有<TODO>组件，我们会重构它，使用路由定义和以及组件上的数据绑定接收数据（在这里我们我们是通过`ui-router`产生的`reslove`,这个例子`todoData`直接映射了数据绑定）。我们把它看作一个路由组件，因为它本质上是一个"view"：

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

**[返回目录](#table-of-contents)**

# 指令

### 基本概念

指令给予了我们的模板，scope ，与控制器绑定，链接和许多其他的事情。这些的使用使我们慎重考虑 `.component（）`的存在。指令不应在声明模板和控制器了，或者通过绑定接收数据。指令应该仅仅是为了装饰DOM使用。这样，就意味着扩展现有的HTML - 如果用`.component（）`创建。简而言之，如果你需要自定义DOM事件/ API和逻辑，使用一个指令并将其绑定到一个组件内的模板。如果你需要的足够的数量的 DOM变化，`postLink`生命周期钩子值得考虑，但是这并不是迁移所有的的DOM操作。你可以给一个无需Angular的地方使用directive

使用指令的小建议:

* 不要使用模板 ,scope，控制器
* 一直设置 `restrict: 'A'`
* 在需要的地方使用 `compile` and `link`
* 记得 `$scope.$on('$destroy', fn) 进行销毁和事件解除;`



**[返回目录](#table-of-contents)**

### 推荐的属性

由于指令支持了大多数 `.component()` 的语法 (模板指令就是最原始的组件), 建议限制指令中的的 Object,以及避免使用错误的指令方法。

| Property | Use it? | Why |
|---|---|---|
| bindToController | No | 在组件中使用 `bindings` |
| compile | Yes | 预编译 DOM 操作/事件 |
| controller | No | 使用一个组件 |
| controllerAs | No | 使用一个组件 |
| link functions | Yes | 对于 DOM操作/事件 的前后|
| multiElement | Yes | [文档](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Yes | [文档](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | No | 使用一个组件 |
| restrict | Yes | 定义一个组件并使用 `A` |
| scope | No | 使用一个组件  |
| template | No | 使用一个组件  |
| templateNamespace | Yes (if you must) | [See docs](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | No | 使用一个组件|
| transclude | No | 使用一个组件 |

**[返回目录](#table-of-contents)**

### 常量 和 类

下面有几个使用es2015和指令的方法，无论是带有箭头函数，更容易的操作，或使用ES2015`Class`。记住选择最适合自己或者团队的方法，并且记住 Angular 中使用 Class.



下面是一个恒在箭头函数的表达式`（）=>（{}）`使用常量的例子，它返回一个对象面（注意里面与`.directive`的使用差异（））:


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

或者用ES2015 Class（注意在注册指令时手动调用 `new TodoAutoFocus`）来创建对象:

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

**[返回目录](#table-of-contents)**

# 服务

### 基本理论

服务本质上是包含业务逻辑的容器，而我们的组件不应该直接进行请求。服务包含其它内置或外部服务，如`$http`，我们可以随时随地的在应用程序注入到组件控制器。我们在开发服务有两种方式，`.service()` 以及 `.factory()`。使用ES2015`Class`，我们应该只使用`.service()`，通过$inject完成依赖注入。

**[返回目录](#table-of-contents)**

### 构建服务Class

下面的 `<todo>` 就是使用 ES2015 `Class`:

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

**[返回目录](#table-of-contents)**

# ES2015 以及相关工具

##### ES2015

* 使用 [Babel](https://babeljs.io/) 将ES2015进行转换为当前浏览器所支持的代码
* 考虑使用 [TypeScript](http://www.typescriptlang.org/) 让你更好的迁移到Angular2

##### 工具
* 使用 `ui-router` [latest alpha](https://github.com/angular-ui/ui-router) (查看 Readme) 如果你希望支持路由钻
  * 你可能会在 `template: '<component>'` 以及 不需要 `bindings`中遇到一些挫折
* 考虑使用 [Webpack](https://webpack.github.io/) 来编译es2016的代码
* 使用 [ngAnnotate](https://github.com/olov/ng-annotate) 自动完成 `$inject` 属性注入
* 如何使用[ngAnnotate with ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/)

**[返回目录](#table-of-contents)**

# 状态管理

考虑使用 Redux 用于 数据管理.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[返回目录](#table-of-contents)**

# 资源

* [理解 .component() 方法](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [使用 "require" 与 $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [理解生命周期钩子, $onInit, $onChanges, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [在路由中使用 "resolve"](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Redux 以及 Angular 状态管理](http://blog.rangle.io/managing-state-redux-angular/)
* [Sample Application from Community](https://github.com/chihab/angular-styleguide-sample)

**[返回目录](#table-of-contents)**

# 文档
关于Angular API [AngularJS documentation](//docs.angularjs.org/api).

# 贡献

打开issues，讨论可能的更改/添加。请不要在issues提任何问题.

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
