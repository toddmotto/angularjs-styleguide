# AngularJS 风格指南

*AngularJS 团队风格指南一己之见 [@toddmotto](//twitter.com/toddmotto)*

针对团队开发 AngularJS 应用的标准方法。这份风格指南涉及到了概念，语法，习惯并且基于我的写作 [writing](http:////toddmotto.com)，演讲 [talking](https://speakerdeck.com/toddmotto)，还有开发 Angular 应用的经验。

#### 社区
[John Papa](//twitter.com/John_Papa) 和我深度讨论过 Angular 的风格模式，并且都发布了各自的风格指南。感谢那些讨论，我从 John 哪里学到了许多，帮助我打造了这份指南。我们都在风格指南上创造了我们自己的东西。我督促你 [参考一下他的](//github.com/johnpapa/angularjs-styleguide) 来对比一下想法.

> 看看这份 [原始文档](http://toddmotto.com/opinionated-angular-js-styleguide-for-teams)，它引发了这一切。

## 目录

  1. [模块](#modules)
  1. [控制器](#controllers)
  1. [服务和工厂](#services-and-factory)
  1. [指令](#directives)
  1. [过滤器](#filters)
  1. [路由解析](#routing-resolves)
  1. [发布和订阅事件](#publish-and-subscribe-events)
  1. [性能](#performance)
  1. [Angular 包装参考](#angular-wrapper-references)
  1. [注释标准](#comment-standards)
  1. [压缩和标注](#minification-and-annotation)

## 模块

  - **定义**: 不要使用带 setter 和 getter 的变量来申明模块

    ```javascript
    // 避免
    var app = angular.module('app', []);
    app.controller();
    app.factory();

    // 建议
    angular
      .module('app', [])
      .controller()
      .factory();
    ```

  - 注意: 使用 `angular.module('app', []);` 设置模块, 再通过 `angular.module('app');` 获取这个模块。只需设置一次，其他所有实例都可获取

  - **方法**: 传递函数进入模块而不要使用回调

    ```javascript
    // 避免
    angular
      .module('app', [])
      .controller('MainCtrl', function MainCtrl () {

      })
      .service('SomeService', function SomeService () {

      });

    // 推荐
    function MainCtrl () {

    }
    function SomeService () {

    }
    angular
      .module('app', [])
      .controller('MainCtrl', MainCtrl)
      .service('SomeService', SomeService);
    ```

  - 这样可以提高可读性，减少 Angular 框架包裹的代码量
  
  - **IIFE 作用域**: 为避免我们传入 Angular 的函数申明污染全局作用域，确保建立任务来把拼接的文件包在 IIFE 里
  
    ```javascript
    (function () {

      angular
        .module('app', []);
      
      // MainCtrl.js
      function MainCtrl () {

      }
      
      angular
        .module('app')
        .controller('MainCtrl', MainCtrl);
      
      // SomeService.js
      function SomeService () {

      }
      
      angular
        .module('app')
        .service('SomeService', SomeService);
        
      // ...
        
    })();
    ```


**[回到顶部](#目录)**

## 控制器

  - **控制器语法**: 控制器是类，所以要坚持使用 controllerAs 语法

    ```html
    <!-- 避免 -->
    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

    <!-- 推荐 -->
    <div ng-controller="MainCtrl as vm">
      {{ vm.someObject }}
    </div>
    ```

  - 在 DOM 里，我们每一个控制器获得一个变量，这可以帮助嵌套的控制器方法，避免任何 `$parent` 调用。 

  - `controllerAs` 语法，在控制器里使用 `this` 来绑定到 `$scope`

    ```javascript
    // 避免
    function MainCtrl ($scope) {
      $scope.someObject = {};
      $scope.doSomething = function () {

      };
    }

    // 推荐
    function MainCtrl () {
      this.someObject = {};
      this.doSomething = function () {

      };
    }
    ```

  - 在`controllerAs` 内只有必要时才使用 `$scope`； 例如, 使用`$emit`, `$broadcast`, `$on` 或者 `$watch` 来发布和订阅事件. 尽量限制使用这些, 不过, 可以把 `$scope` 作为一种特殊的用例来对待

  - **继承**: 使用原型继承来扩展控制器类

    ```javascript
    function BaseCtrl () {
      this.doSomething = function () {

      };
    }
    BaseCtrl.prototype.someObject = {};
    BaseCtrl.prototype.sharedSomething = function () {

    };

    AnotherCtrl.prototype = Object.create(BaseCtrl.prototype);

    function AnotherCtrl () {
      this.anotherSomething = function () {

      };
    }
    ```

  - 使用带 polyfill 的 `Object.create` 来支持各种浏览器

  - **controllerAs 'vm'**: 使用 `vm` 来获取控制器上下文 `this` , `vm` 是 `ViewModel` 的缩写

    ```javascript
    // 避免
    function MainCtrl () {
      var doSomething = function () {

      };
      this.doSomething = doSomething;
    }

    // 推荐
    function MainCtrl () {
      var vm = this;
      var doSomething = function () {
        
      };
      vm.doSomething = doSomething;
    }
    ```

    *为什么?* : 函数上下文会改变 `this` 值, 使用它来避免 `.bind()` 的调用和作用域问题
    
  - **ES6**: 如果使用 ES6，避免使用 `var vm = this;` 

    ```javascript
    // 避免
    function MainCtrl () {
      let vm = this;
      let doSomething = arg => {
        console.log(vm);
      };
      
      // exports
      vm.doSomething = doSomething;
    }

    // 推荐
    function MainCtrl () {
      
      let doSomething = arg => {
        console.log(this);
      };
      
      // exports
      this.doSomething = doSomething;
      
    }
    ```

    *为什么?* : 当必须要用到 `this` 语义时，使用 ES6 箭头函数

  - **只有表示逻辑 (MVVM)**: 控制器里只有表示层的逻辑, 避免业务逻辑 (委托到服务)

    ```javascript
    // 避免
    function MainCtrl () {
      
      var vm = this;

      $http
        .get('/users')
        .success(function (response) {
          vm.users = response;
        });

      vm.removeUser = function (user, index) {
        $http
          .delete('/user/' + user.id)
          .then(function (response) {
            vm.users.splice(index, 1);
          });
      };

    }

    // 推荐
    function MainCtrl (UserService) {

      var vm = this;

      UserService
        .getUsers()
        .then(function (response) {
          vm.users = response;
        });

      vm.removeUser = function (user, index) {
        UserService
          .removeUser(user)
          .then(function (response) {
            vm.users.splice(index, 1);
          });
      };

    }
    ```

    *为什么?* : 控制器应该从服务取回模型数据，避免业务逻辑。控制器应该作为 ViewModel 并且控制模型和视图表示层之间的数据流。业务逻辑在控制器里使得测试服务 Services 不可能。

**[回到顶部](#目录)**

## 服务和工厂

  - 所有 Angular 服务都是单例模式，使用 `.service` 或者 `.factory` 来区分对象 Objects 创建的方式。

  **服务 Services**: 行为像 `constructor` 构造函数并且使用 `new` 关键字来初始化. 使用 `this` 来引用公众方法和变量
  
    ```javascript
    function SomeService () {
      this.someMethod = function () {

      };
    }
    angular
      .module('app')
      .service('SomeService', SomeService);
    ```

  **工厂 Factory**: 业务逻辑或者提供者模块，返回一个对象或者闭包

  - 永远返回一个托管对象而不是 revealing Module 模式，因为对象引用是绑定的并可以更新。 

    ```javascript
    function AnotherService () {
      var AnotherService = {};
      AnotherService.someValue = '';
      AnotherService.someMethod = function () {

      };
      return AnotherService;
    }
    angular
      .module('app')
      .factory('AnotherService', AnotherService);
    ```

    *为什么?* :  使用 revealing module 模式不能单独更行原始类型的值 
    
**[回到顶部](#目录)**

## 指令

  - **申明的限制**: 依据指令的角色，只使用 `custom element` 和 `custom attribute` 方式来申明你的指令 (`{ restrict: 'EA' }`) 

    ```html
    <!-- 避免 -->

    <!-- directive: my-directive -->
    <div class="my-directive"></div>

    <!-- 推荐 -->

    <my-directive></my-directive>
    <div my-directive></div>
    ```

  - 注释和类申明让人迷惑，应当避免。注释和老版本的 IE 玩不到一起去。使用属性 atrribute 是最安全的方式来覆盖更多浏览器。 

  - **模板 Templating**: 用 `Array.join('')` 来使模板清爽

    ```javascript
    // 避免
    function someDirective () {
      return {
        template: '<div class="some-directive">' +
          '<h1>My directive</h1>' +
        '</div>'
      };
    }

    // 推荐
    function someDirective () {
      return {
        template: [
          '<div class="some-directive">',
            '<h1>My directive</h1>',
          '</div>'
        ].join('')
      };
    }
    ```

    *为什么?* : 提高可读性，因为代码可以被恰当缩进，同时也避免没那么清爽的 `+` 操作符，并且如果不恰当地分行的话会导致错误。 

  - **DOM 操作**: 只在指令里发生, 永远不要在控制器和服务里使用

    ```javascript
    // 避免
    function UploadCtrl () {
      $('.dragzone').on('dragend', function () {
        // handle drop functionality
      });
    }
    angular
      .module('app')
      .controller('UploadCtrl', UploadCtrl);

    // 推荐
    function dragUpload () {
      return {
        restrict: 'EA',
        link: function (scope, element, attrs) {
          element.on('dragend', function () {
            // handle drop functionality
          });
        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - **命名习惯**: 永远不要给自定义指令加 `ng-*` 前缀, 他们可能会和将来的原生指令冲突 
  - 
    ```javascript
    // 避免
    // <div ng-upload></div>
    function ngUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('ngUpload', ngUpload);

    // 推荐
    // <div drag-upload></div>
    function dragUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - 只有指令和过滤器是首字母为小写的 provider; 这是因为指令的命名习惯限制造成的。Angular 会给`camelCase`样式的词加上中划线，所以 `dragUpload` 在页面的元素中将变为`<div drag-upload></div>` 
  
  - **controllerAs**: 在指令中也同样要使用 `controllerAs` 语法

    ```javascript
    // 避免
    function dragUpload () {
      return {
        controller: function ($scope) {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);

    // 推荐
    function dragUpload () {
      return {
        controllerAs: 'vm',
        controller: function () {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

**[回到顶部](#目录)**

## 过滤器

  - **全局过滤器**: 只使用 `angular.filter()` 创建全局过滤器. 永远不要在控制器和服务里使用本地过滤器
  - 
    ```javascript
    // 避免
    function SomeCtrl () {
      this.startsWithLetterA = function (items) {
        return items.filter(function (item) {
          return /^a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .controller('SomeCtrl', SomeCtrl);

    // 推荐
    function startsWithLetterA () {
      return function (items) {
        return items.filter(function (item) {
          return /^a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .filter('startsWithLetterA', startsWithLetterA);
    ```

  - 这会提高测试和可重用性

**[回到顶部](#目录)**

## 路由解析

  - **Promises**: 在 `$routeProvider` (或者 `$stateProvider` 如果是 `ui-router`) 里解析控制器依赖, 而不是控制器本身 
  - 
    ```javascript
    // 避免
    function MainCtrl (SomeService) {
      var _this = this;
      // unresolved
      _this.something;
      // resolved asynchronously
      SomeService.doSomething().then(function (response) {
        _this.something = response;
      });
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

    // 推荐
    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        resolve: {
          // resolve here
        }
      });
    }
    angular
      .module('app')
      .config(config);
    ```

  - **Controller.resolve 属性**: 永远不要把逻辑绑定在路由里。引用每个控制器的 `resolve` 属性来连接逻辑。  
  - 
    ```javascript
    // 避免
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'vm',
        controller: 'MainCtrl'
        resolve: {
          doSomething: function () {
            return SomeService.doSomething();
          }
        }
      });
    }

    // 推荐
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    MainCtrl.resolve = {
      doSomething: function (SomeService) {
        return SomeService.doSomething();
      }
    };

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'vm',
        controller: 'MainCtrl'
        resolve: MainCtrl.resolve
      });
    }
    ```

  - 这样可以把依赖解析保存在控制器和路由的同一个文件里而又免于逻辑侵入 
**[回到顶部](#目录)**

## 发布和订阅事件

  - **$scope**: 只用 `$emit` 和 `$broadcast` 方法来触发有直接关联的作用域事件 
  
    ```javascript
    // up the $scope
    $scope.$emit('customEvent', data);

    // down the $scope
    $scope.$broadcast('customEvent', data);
    ```

  - **$rootScope**: 只把 `$emit` 当做整个应用的事件总线，并且记得解除监听者绑定
  
    ```javascript
    // 所有 $rootScope.$on 监听者
    $rootScope.$emit('customEvent', data);
    ```

  - 提示: 因为`$rootScope` 永远不会被销毁，`$rootScope.$on` 监听者也不会，不像`$scope.$on`监听者，`$rootScope.$on` 监听者会永远存在，因此当相关 `$scope`触发`$destroy`事件时，他们需要被销毁 
  
    ```javascript
    // 调用闭包
    var unbind = $rootScope.$on('customEvent'[, callback]);
    $scope.$on('$destroy', unbind);
    ```

  - 对于多个`$rootScope` 监听者, 使用对象字面量和循环`$destroy`事件来自动解除绑定 

    ```javascript
    var rootListeners = {
      'customEvent1': $rootScope.$on('customEvent1'[, callback]),
      'customEvent2': $rootScope.$on('customEvent2'[, callback]),
      'customEvent3': $rootScope.$on('customEvent3'[, callback])
    };
    for (var unbind in rootListeners) {
      $scope.$on('$destroy', rootListeners[unbind]);
    }
    ```

**[回到顶部](#目录)**

## 性能

  - **一次性绑定语法 One-time binding syntax**: 对于较新的 Angular (v1.3.0-beta.10+) 版本, 尽量在合乎道理的地方使用一次性绑定 one-time binding syntax `{{ ::value }}` 

    ```避免
    // avoid
    <h1>{{ vm.title }}</h1>

    // 推荐
    <h1>{{ ::vm.title }}</h1>
    ```
    
    *为什么?* : 在`undefined`变量被解析之后，绑定一旦从作用域的`$$watchers`数组解除观察者，就会提升每次脏检查的性能 
    
  - **考虑 $scope.$digest**: 在合乎道理的地方使用 `$scope.$digest` 超过 `$scope.$apply`. 只有子作用域会更新

    ```javascript
    $scope.$digest();
    ```
    
    *为什么?* : `$scope.$apply` 会调用 `$rootScope.$digest`, 这会导致整个应用的`$$watchers` 再次进行脏检查。使用 `$scope.$digest` 会从发起的作用域`$scope`检查当前的和子作用域 

**[回到顶部](#目录)**

## Angular 包装参考

  - **$document 和 $window**: 任何时候都使用 `$document` 和 `$window` 来帮助测试和 Angular 引用

    ```javascript
    // 避免
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          document.addEventListener('click', function () {

          });
        }
      };
    }

    // 推荐
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs, $document) {
          $document.addEventListener('click', function () {

          });
        }
      };
    }
    ```

  - **$timeout 和 $interval**: 使用 `$timeout` 和 `$interval` 超过他们的原生对应者来保持 Angular 的双向数据绑定到最新
  -  
    ```javascript
    // 避免
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          setTimeout(function () {
            //
          }, 1000);
        }
      };
    }

    // 推荐
    function dragUpload ($timeout) {
      return {
        link: function ($scope, $element, $attrs) {
          $timeout(function () {
            //
          }, 1000);
        }
      };
    }
    ```

**[回到顶部](#目录)**

## 注释标准

  - **jsDoc**: 使用 jsDoc 语法来记录函数名，描述，参数和返回 
  - 
    ```javascript
    /**
     * @name SomeService
     * @desc 主应用控制器
     */
    function SomeService (SomeService) {

      /**
       * @name doSomething
       * @desc 做点酷毙事
       * @param {Number} x - 做事会用到的第一个数字
       * @param {Number} y - 做事会用到的第二个数字
       * @returns {Number}
       */
      this.doSomething = function (x, y) {
        return x * y;
      };

    }
    angular
      .module('app')
      .service('SomeService', SomeService);
    ```

**[回到顶部](#目录)**

## 压缩和标注

  - **ng-annotate**: 对 Gulp 作为`ng-min`来使用 [ng-annotate](//github.com/olov/ng-annotate) 已经弃用, 使用`/** @ngInject */`注释需要自动依赖注入的函数

    ```javascript
    /**
     * @ngInject
     */
    function MainCtrl (SomeService) {
      this.doSomething = SomeService.doSomething;
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);
    ```

  - 这会产生如下带有 `$inject`标注的输出 

    ```javascript
    /**
     * @ngInject
     */
    function MainCtrl (SomeService) {
      this.doSomething = SomeService.doSomething;
    }
    MainCtrl.$inject = ['SomeService'];
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);
    ```

**[回到顶部](#目录)**

## Angular 文档
其他任何问题, 包括 API 参考, 查看 [Angular 文档](//docs.angularjs.org/api).

## 贡献

先创建一条 issue 来讨论潜在的改变和添加  

## 许可证

#### (The MIT License)

Copyright (c) 2015 Todd Motto

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
