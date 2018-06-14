# Angular 1.x 编码风格 (TypeScript)

> 原文地址：[Angular 1.x styleguide (TypeScript)](https://github.com/toddmotto/angularjs-styleguide/blob/master/typescript/README.md)

### 架构，文件结构，组件，单向数据流以及最佳实践

*来自[@toddmotto](//twitter.com/toddmotto)团队的实用编码指南*

Angular 的编码风格以及架构已经使用ES2015进行重写,这些在AngularJS 1.5+的变化可以更好帮助您的更好的升级到Angular2.。 这份指南包括了新的单向数据流，事件委托，组件架构和组件路由。

老版本的指南你可以在[这里](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5)找到，在[这里](https://toddmotto.com/rewriting-angular-styleguide-angular-2)能看到最新的。

> 加入终极的 AngularJS 学习经验，完全掌握初级和高级的 AngularJS 特性，构建更快，易于扩展的真实应用程序。

![](https://camo.githubusercontent.com/ce15039b3bbe98dbbd58cb09b0fc4f3ba15a75f2/68747470733a2f2f756c74696d617465616e67756c61722e636f6d2f6173736574732f696d672f62616e6e6572732f75612d6769746875622e737667)

## 目录

1. [模块结构](#modular-architecture)
    1. [基本概念](#module-theory)
    1. [根模块](#root-module)
    1. [组件模块](#component-module)
    1. [公共模块](#common-module)
    1. [低级别模块](#low-level-modules)
    1. [文件命名规范](#file-naming-conventions)
    1. [可扩展的文件结构](#scalable-file-structure)
1. [组件](#components)
    1. [基本概念](#component-theory)
    1. [支持的属性](#supported-properties)
    1. [控制器](#controllers)
    1. [当向数据流和事件](#one-way-dataflow-and-events)
    1. [有状态组件](#stateful-components)
    1. [无状态组件](#stateless-components)
    1. [路由组件](#routed-components)
1. [指令](#directives)
    1. [基本概念](#directive-theory)
    1. [推荐的属性](#recommended-properties)
    1. [常量和类](#constants-or-classes)
1. [服务](#services)
    1. [基本概念](#service-theory)
    1. [服务的类](#classes-for-service)
1. [样式](#styles)
1. [TypeScript 和工具](#typescript-and-tooling)
1. [状态管理](#state-management)
1. [资源](#resources)
1. [文档](#documentation)
1. [贡献](#contributing)

# Modular architecture

Angular中的每个模块都是一个模块组件。模块组件是包括了组件逻辑，模板，路由和子组件的根。

### Module theory

模块的设计直接反映到我们的文件夹结构，从而保证我们项目的可维护性和可预测性。 我们最好应该有三个高层次的模块：根模块，组件模块和常用模块。根模块定义用于启动 app 和相应的模板的基础模块。 然后导入我们需要依赖的组件和通用模块。组件和通用模块然后需要低级别的组件模块，包含我们的组件，控制器，服务，指令，过滤器和给可重复使用的功能进行测试。

**[回到顶部](#目录)**

### Root module

根模块以一个根组件开始，它定义了整个应用程序的基本元素和路由出口，例如使用`ui-router`展示`ui-view`。

```ts
// app.component.ts
export const AppComponent: angular.IComponentOptions  = {
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

随着`AppComponent`导入和使用`.component('app', AppComponent)`注册，一个根模块就创建了。进一步导入子模块（组件和公共模块）包括与应用程序相关的所有组件。你可能会注意到在这里也导入了样式，我们将在本直男的后面章节介绍这个。

```ts
// app.ts
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { AppComponent } from './app.component';
import { ComponentsModule } from './components/components.module';
import { CommonModule } from './common/common.module';
import './app.scss';

const root = angular
  .module('app', [
    ComponentsModule,
    CommonModule,
    uiRouter
  ])
  .component('app', AppComponent)
  .name;

export default root;
```

**[回到顶部](#目录)**

### Component module

一个组件模块是引用所有可复用组件的容器。在上面我们看到如何导入`Components`并且将他们注入到根模块，这里给了我们一个导入所有应用程序需要的组件的地方。我们要求这些模块与其他模块都是解耦的，因此可以很容易的移动到其他任何应用程序中。

```ts
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

**[回到顶部](#目录)**

### Common module

公共模块是引用所有为应用程序提供的特殊组件的容器，我们不希望它在其他应用程序中使用。这可以是布局，导航和页脚之类的东西。在上面我们看到如何导入`Common`并且将他们注入到根模块，这里是给了我们一个导入应用程序需要的所有的公共组件的地方。

```ts
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

**[回到顶部](#目录)**

### Low-level modules

 Always remember to add the `.name` suffix to each `export` when creating a _new_ module, not when referencing one. You'll noticed routing definitions also exist here, we'll come onto this in later chapters in this guide.

低级别的模块是包含每个功能块逻辑的独立组件。每个模块都将定义成一个可以被导入较高级别的单独模块，例如一个组件或者公共模块，如下所示。 。一定要记住每次创建一个新的模块，而非引用的时候，记得给每个`export`中添加`.name`的后缀。你会注意到路由定义也在这里，我们将在随后的部分讲到它。

```ts
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { CalendarComponent } from './calendar.component';
import './calendar.scss';

export const CalendarModule = angular
  .module('calendar', [
    uiRouter
  ])
  .component('calendar', CalendarComponent)
  .config(($stateProvider: angular.ui.IStateProvider,
            $urlRouterProvider: angular.ui.IUrlRouterProvider) => {
    $stateProvider
      .state('calendar', {
        url: '/calendar',
        component: 'calendar'
      });
    $urlRouterProvider.otherwise('/');
  })
  .name;

```

**[回到顶部](#目录)**

### File naming conventions

使用小写并保持命名的简洁, 使用组件名称举例, `calendar.*.ts*`, `calendar-grid.*.ts` - 将文件类型的名称放到中间。使用 `index.ts` 作为模块的定义文件，这样就可以通过目录名导入模块了。

```ts
index.ts
calendar.component.ts
calendar.service.ts
calendar.directive.ts
calendar.filter.ts
calendar.spec.ts
calendar.html
calendar.scss
```

**[回到顶部](#目录)**

### Scalable file structure

文件目录结构非常重要，它有利于我们更好的扩展和预测。下面的例子展示了模块组件的基本架构。

```
├── app/
│   ├── components/
│   │  ├── calendar/
│   │  │  ├── index.ts
│   │  │  ├── calendar.component.ts
│   │  │  ├── calendar.service.ts
│   │  │  ├── calendar.spec.ts
│   │  │  ├── calendar.html
│   │  │  ├── calendar.scss
│   │  │  └── calendar-grid/
│   │  │     ├── index.ts
│   │  │     ├── calendar-grid.component.ts
│   │  │     ├── calendar-grid.directive.ts
│   │  │     ├── calendar-grid.filter.ts
│   │  │     ├── calendar-grid.spec.ts
│   │  │     ├── calendar-grid.html
│   │  │     └── calendar-grid.scss
│   │  ├── events/
│   │  │  ├── index.ts
│   │  │  ├── events.component.ts
│   │  │  ├── events.directive.ts
│   │  │  ├── events.service.ts
│   │  │  ├── events.spec.ts
│   │  │  ├── events.html
│   │  │  ├── events.scss
│   │  │  └── events-signup/
│   │  │     ├── index.ts
│   │  │     ├── events-signup.controller.ts
│   │  │     ├── events-signup.component.ts
│   │  │     ├── events-signup.service.ts
│   │  │     ├── events-signup.spec.ts
│   │  │     ├── events-signup.html
│   │  │     └── events-signup.scss
│   │  └── components.module.ts
│   ├── common/
│   │  ├── nav/
│   │  │     ├── index.ts
│   │  │     ├── nav.component.ts
│   │  │     ├── nav.service.ts
│   │  │     ├── nav.spec.ts
│   │  │     ├── nav.html
│   │  │     └── nav.scss
│   │  ├── footer/
│   │  │     ├── index.ts
│   │  │     ├── footer.component.ts
│   │  │     ├── footer.service.ts
│   │  │     ├── footer.spec.ts
│   │  │     ├── footer.html
│   │  │     └── footer.scss
│   │  └── index.ts
│   ├── index.ts
│   ├── app.component.ts
│   └── app.scss
└── index.html
```

顶级目录仅仅包含了`index.html`和`app/`, `app/`目录中则包含了我们要用到的根模块，组件，公共模块，以及低级别的模块。

**[回到顶部](#目录)**

# Components

### Component theory

组件实际上就是带有控制器的模板。他们即不是指令，也不应该使用组件代替指令，除非你正在用控制器升级“模板指令”，它是最适合作为组件的。 组件还包含数据事件的输入与输出，生命周期钩子和使用单向数据流以及从父组件上获取数据的事件对象备份。这些都是在AngularJS 1.5及以上推出的新标准。我们创建的所有模板和控制器都可能是一个组件，它可能是是有状态的，无状态或者路由组件。你可以将“组件”看作一段完整的代码，而不仅仅是`.component()`定义的对象。让我们来探讨一些组件最佳实践和建议，然后你应该可以明白如何通过有状态，无状态和路由组件的概念来组织结构。

**[回到顶部](#目录)**

### Supported properties

下面是一些你可能会使用到的`.component()`属性 :

| Property | Support |
|---|---|
| bindings | Yes, 仅仅使用 `'@'`, `'<'`, `'&'` |
| controller | Yes |
| controllerAs | Yes, 默认是`$ctrl` |
| require | Yes (新对象语法) |
| template | Yes |
| templateUrl | Yes |
| transclude | Yes |

**[回到顶部](#目录)**

### Controllers

控制器应该仅仅与组件一起使用，而不应该是任何地方。如果你觉得你需要一个控制器，你真正需要的可能是一个来管理特定行的无状态组件。

这里有一些使用`Class`构建控制器的建议:

* 始终使用`constructor`来依赖注入
* 不要直接导出`Class`，导出它的名字去允许使用`$inject`注解
* 如果你需要访问scope中的语法，请使用箭头函数
* 另外关于箭头函数，`let ctrl = this;`也是可以接受的，当然这更取决于使用场景
* 将所有公开的函数直接绑定到`Class`
* 适当的使用生命周期钩子，`$onInit`, `$onChanges`, `$postLink` 和 `$onDestroy`
  * 注意：`$onChanges`在`$onInit`之前被调用，查看这里的[扩展阅读](https://github.com/toddmotto/angularjs-styleguide/blob/master/README.md#resources)对生命周期有进一步的理解
* 在`$onInit`使用`require`去引用其他继承的逻辑
* 不要使用`controllerAs`语法去覆盖默认的`$ctrl`别名，当然也不要再其他地方使用`controllerAs`

**[回到顶部](#目录)**

### One-way dataflow and Events

单向数据流已经在Angular1.5中引入了，并且重新定义了组件之间的通信。

这里有一些使用单向数据流的建议:

* 在组件中始终使用单向数据绑定语法`<`来接收数据
* 不要在任何地方再使用双向绑定语法`'='`
* 有 `bindings` 的组件应该使用 `$onChanges` 克隆单向绑定数据而阻止通过引用传递对象，并且更新父级数据
* 在父级方法中使用 `$event` 作为一个函数参数（查看有状态组件的例子`$ctrl.addTodo($event)`）
* 从无状态组件传回一个 `$event: {}` 对象（查看无状态组件的例子`this.onAddTodo`）
  * Bonus：使用 `.value()` 包装 `EventEmitter` 以便迁移到 Anuglar2，避免手动创一个 `$event` 对象
* 为什么？这方便迁移到Angular2，并且在组件内部保持一致性。并且可以让状态可预测。

**[回到顶部](#目录)**

### Stateful components

我们来定义下什么叫作“有状态组件”：

* 本质上通过服务于后端API通信获取状态
* 不直接改变状态
* 状态改变的时候渲染子组件
* 可以作为小的容器组件引用

下面的是一个状态组件案例，它和一个低级别的模块组件共同完成（这只是演示，为了精简省略了一些代码）

```ts
/* ----- todo/todo.component.ts ----- */
import { TodoController } from './todo.controller';
import { TodoService } from './todo.service';
import { TodoItem } from '../common/model/todo';

export const TodoComponent: angular.IComponentOptions  = {
  controller: TodoController,
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


/* ----- todo/todo.controller.ts ----- */
export class TodoController {
  static $inject: string[] = ['TodoService'];
  todos: TodoItem[];

  constructor(private todoService: TodoService) { }

  $onInit() {
    this.newTodo = new TodoItem('', false);
    this.todos = [];
    this.todoService.getTodos().then(response => this.todos = response);
  }
  addTodo({ todo }) {
    if (!todo) return;
    this.todos.unshift(todo);
    this.newTodo = new TodoItem('', false);
  }
}

/* ----- todo/index.ts ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';


export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;

/* ----- todo/todo.service.ts ----- */
export class TodoService {
  static $inject: string[] = ['$http'];

  constructor(private $http: angular.IHttpService) { }

  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}


/* ----- common/model/todo.ts ----- */
export class TodoItem {
    constructor(
        public title: string,
        public completed: boolean) { }
    )
}


```


这个例子展示了一个有状态的组件，在控制器通过服务获取状态，然后再将它传递给无状态的子组件。注意这里并没有在模版中使例如如`ng-repeat`和其他指令。相反，将数据和函数代理到 `<todo-form>` 和 `<todo-list>` 这两个无状态的组件。


**[回到顶部](#目录)**

### Stateless components

我们来定义下什么叫作“无状态组件”：

* 使用 `bindings: {}` 定义输入输出
* 数据通过属性绑定进入组件（输入）
* 数据通过事件离开组件（输出）
* 状态改变，按需传回数据（离去点击和提交事件）
* 不关心数据来自于哪里，它是无状态的
* 可高频率复用的组件
* 也被称作哑巴或者展示性组件

下面是一个无状态组件的例子 (我们使用 `<todo-form>` 作为例子) , 使用低级别模块定义来完成(仅仅用于演示，省略了部分代码)：

```ts
/* ----- todo/todo-form/todo-form.component.ts ----- */
import { TodoFormController } from './todo-form.controller';

export const TodoFormComponent: angular.IComponentOptions = {
  bindings: {
    todo: '<',
    onAddTodo: '&'
  },
  controller: TodoFormController,
  template: `
    <form name="todoForm" ng-submit="$ctrl.onSubmit();">
      <input type="text" ng-model="$ctrl.todo.title">
      <button type="submit">Submit</button>
    </form>
  `
};


/* ----- todo/todo-form/todo-form.controller.ts ----- */
import { EventEmitter } from '../common/event-emitter';
import { Event } from '../common/event';

export class TodoFormController {
  static $inject = ['EventEmitter'];

  constructor(private eventEmitter: EventEmitter) {}
  $onChanges(changes) {
    if (changes.todo) {
      this.todo = Object.assign({}, this.todo);
    }
  }
  onSubmit() {
    if (!this.todo.title) return;
    // with EventEmitter wrapper
    this.onAddTodo(
      eventEmitter({
        todo: this.todo
      });
    );
    // without EventEmitter wrapper
    this.onAddTodo(new Event({
        todo: this.todo
      }));
  }
}

/* ----- common/event.ts ----- */
export class Event {
    constructor(public $event: any){ }
}

/* ----- common/event-emitter.ts ----- */
import { Event } from './event';

export function EventEmitter(payload: any): Event {
    return new Event(payload);
}

/* ----- todo/todo-form/index.ts ----- */
import angular from 'angular';
import { EventEmitter } from './common/event-emitter';
import { TodoFormComponent } from './todo-form.component';

export const TodoFormModule = angular
  .module('todo.form', [])
  .component('todoForm', TodoFormComponent)
  .value('EventEmitter', EventEmitter)
  .name;

```

请注意 `<todo-form>` 组件不获取状态，它只是简单的接收，它通过控制器的逻辑去改变一个对象，然后通过绑定的属性将改变后的值传回给父组件。 在这个例子中 `$onChanges` 生命周期钩子克隆了初始的 `this.todo` 对象并重新赋值，这意味着父组件的数据在我们提交表单之前不受影响，同时还要新的单向数据绑定语法'<' 。


**[回到顶部](#目录)**

### Routed components

我们来定义下什么叫作“路由组件”：

* 它本质上是一个具有路由定义的有状态组件
* 没有 `router.ts` 文件
* 我们使用路由组件去定义他们自己的路由逻辑
* 数据通过路由 resolve “输入” 组件（可选，依然可以在控制器中使用服务调用）

在这个例子中，我们将利用已经存在的 `<todo>`  组件，我们使用路由定义和组件上的 `bindings` 接收数据（这里使用`ui-router`的秘诀是我们创建的`reslove`属性，这个例子中`todoData`直接映射到了`bindings`）。我们把它看作一个路由组件，因为它本质上是一个“视图”：

```ts
/* ----- todo/todo.component.ts ----- */
import { TodoController } from './todo.controller';

export const TodoComponent: angular.IComponentOptions = {
  bindings: {
    todoData: '<'
  },
  controller: TodoController,
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

/* ----- todo/todo.controller.ts ----- */
import { TodoItem } from '../common/model/todo';

export class TodoController {
  todos: TodoItem[] = [];

  $onInit() {
    this.newTodo = new TodoItem();
  }

  $onChanges(changes) {
    if (changes.todoData) {
      this.todos = Object.assign({}, this.todoData);
    }
  }

  addTodo({ todo }) {
    if (!todo) return;
    this.todos.unshift(todo);
    this.newTodo = new TodoItem();
  }
}


/* ----- common/model/todo.ts ----- */
export class TodoItem {
    constructor(
        public title: string = '',
        public completed: boolean = false) { }
}


/* ----- todo/todo.service.ts ----- */
export class TodoService {
  static $inject: string[] = ['$http'];

  constructor(private $http: angular.IHttpService) { }

  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}


/* ----- todo/index.ts ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoService } from './todo.service';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .config(($stateProvider: angular.ui.IStateProvider, $urlRouterProvider: angular.ui.IUrlRouterProvider) => {
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

```

**[回到顶部](#目录)**

# Directives

### Directive theory

指令给了我们 `template` ，`scope` 绑定 ，`bindToController`，`link` 和许多其他的事情。使用这些我们应该慎重考虑现在的 `.component()`。指令不应该再声明模板和控制器了，或者通过绑定接收数据。指令应该仅仅是为了装饰DOM使用。这样，使用 `.component()` 创建就意味着扩展现有的HTML。简而言之，如果你需要自定义`DOM`事件/ APIs和逻辑，在组件里使用一个指令将其绑定到模板。如果你需要的足够的数量的 `DOM`操作，`$postLink` 生命周期钩子值得考虑，但是这并不是迁移所有的的DOM操作，如果可以的话，你可以使用指令来处理非Angular的事情。

下面是一些使用指令的建议：

* 不要使用templates、scope，bindToController 或者 controllers
* 指令通常使用`restrict: 'A'`
* 在需要的地方使用 `compile` 和 `link`
* 记得在`$scope.$on('$destroy', fn);`中销毁或者解绑事件处理

**[回到顶部](#目录)**

### Recommended properties

Due to the fact directives support most of what `.component()` does (template directives were the original component), I'm recommending limiting your directive Object definitions to only these properties, to avoid using directives incorrectly:

由于指令实际上支持了大多数 `.component()` 的语法 (模板指令就是最原始的组件), 我建议将指令对象定义限制在这些属性上，去避免错误的使用指令：

| Property | Use it? | Why |
|---|---|---|
| bindToController | No | 在组件中使用 `bindings` |
| compile | Yes | 预编译 DOM 操作/事件 |
| controller | No | 使用一个组件 |
| controllerAs | No | 使用一个组件 |
| link functions | Yes | 对于DOM 操作/事件的前后 |
| multiElement | Yes | [文档](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Yes | [文档](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | No | 使用一个组件 |
| restrict | Yes | 使用 `'A'` 去定义一个组件|
| scope | No | 使用一个组件 |
| template | No | 使用一个组件 |
| templateNamespace | Yes (if you must) | [文档](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | No | 使用一个组件 |
| transclude | No | 使用一个组件 |

**[回到顶部](#目录)**

### Constants or Classes

这里有使用 TypeScript 和 directives 实现的几种方式，不管是使用箭头函数还是更简单的复制，或者使用 TypeScript 的 `Class`。选择最适合你或者你团队的，Angular2中使用的是`Class`。

下面是使用箭头函数表达式使用常量的例子`() => ({})`，返回一个对象字面量（注意与使用`.directive()`的不同）：

```ts
/* ----- todo/todo-autofocus.directive.ts ----- */
import angular from 'angular';

export const TodoAutoFocus = ($timeout: angular.ITimeoutService) => (<angular.IDirective> {
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

/* ----- todo/index.ts ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoAutofocus } from './todo-autofocus.directive';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', TodoAutoFocus)
  .name;

```
或者使用 TypeScript `Class` （注意在注册指令的时候手动调用`new TodoAutoFocus`）去创建一个新对象：

```ts
/* ----- todo/todo-autofocus.directive.ts ----- */
import angular from 'angular';

export class TodoAutoFocus implements angular.IDirective {
  static $inject: string[] = ['$timeout'];
  restrict: string;

  constructor(private $timeout: angular.ITimeoutService) {
    this.restrict = 'A';
  }

  link($scope, $element: HTMLElement, $attrs) {
    $scope.$watch($attrs.todoAutofocus, (newValue, oldValue) => {
      if (!newValue) {
        return;
      }

      $timeout(() => $element[0].focus());
    });
  }
}


/* ----- todo/index.ts ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoAutofocus } from './todo-autofocus.directive';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', ($timeout: angular.ITimeoutService) => new TodoAutoFocus($timeout))
  .name;

```

**[回到顶部](#目录)**

# Services

### Service theory

服务本质上是包含业务逻辑的容器，而我们的组件不应该直接进行请求。服务包含其它内置或外部服务，例如`$http`，我们可以随时随地的在应用程序注入到组件控制器。我们在开发服务有两种方式，使用`.service()` 或者 `.factory()`。使用TypeScript `Class`，我们应该只使用`.service()`，通过`$inject`完成依赖注入。

**[回到顶部](#目录)**

### Classes for Service

下面是使用 TypeScript `Class` 实现`<todo>` 应用的一个例子：

```ts
/* ----- todo/todo.service.ts ----- */
export class TodoService {
  static $inject: string[] = ['$http'];

  constructor(private $http: angular.IHttpService) { }
  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}


/* ----- todo/index.ts ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoService } from './todo.service';

export const todo = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .name;

```

**[回到顶部](#目录)**

# Styles

利用[Webpack](https://webpack.github.io/) 我们现在可以在 `*.module.js` 中的 `.scss`文件上使用`import` 语句，让 Webpack 知道在我们的样式中包含这样的文件。 这样做可以使我们的组件在功能和样式上保持分离，它还与Angular2中使用样式的方式更加贴近。这样做不会让样式像Angular2一样隔离在某个组件上，样式还可以广泛应用到我们的应用程序上，但是它更加易于管理，并且使得我们的应用结构更加易于推理。

If you have some variables or globally used styles like form input elements then these files should still be placed into the root `scss` folder. e.g. `scss/_forms.scss`. These global styles can the be `@imported` into your root module (`app.module.js`) stylesheet like you would normally do.

如果你有一些变量或者全局使用的样式，像表单的input元素，那么这些文件应该放在根`scss`文件夹。例如`scss/_forms.scss`。这些全局的样式可以像通常意义被`@imported`到根模块(`app.module.ts`)。

**[回到顶部](#目录)**

# TypeScript and Tooling

##### TypeScript

* 使用[Babel](https://babeljs.io/) 编译 TypeScript 代码和其他 polyfills
* 考虑使用 [TypeScript](http://www.typescriptlang.org/)让你的代码迁移到Angular2


##### Tooling
* 如果你想支持组件路由，使用`ui-router`[latest alpha](https://github.com/angular-ui/ui-router)（查看Readme）
  * 否则你将会被 `template: '<component>'` 和 没有 `bindings` 困住
* 考虑使用[Webpack](https://webpack.github.io/)来编译你的 TypeScript 代码
* 使用[ngAnnotate](https://github.com/olov/ng-annotate) 来自动注解 `$inject` 属性
* 如何使用[ngAnnotate with TypeScript](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/#ng-annotate)

**[回到顶部](#目录)**

# State management

考虑在Angular1.5中使用Redux用于数据管理。

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[回到顶部](#目录)**

# Resources

* [Understanding the .component() method](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Using "require" with $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Understanding all the lifecycle hooks, $onInit, $onChange, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Using "resolve" in routes](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Redux and Angular state management](http://blog.rangle.io/managing-state-redux-angular/)

**[回到顶部](#目录)**

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
