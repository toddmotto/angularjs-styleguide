# AngularJS styleguide

*Opinionated AngularJS styleguide for teams by [@toddmotto](//twitter.com/toddmotto)*

## Table of Contents

  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Factory](#factory)
  1. [Directives](#directives)
  1. [Filters](#filters)
  1. [Routing resolves](#routing-resolves)

## Modules

  - **Definitions**: Declare modules without a variable using the setter and getter syntax

    ```javascript
    // bad
    var app = angular.module('app', []);
    app.controller();
    app.factory();

    // good
    angular
      .module('app', [])
      .controller()
      .factory();
    ```

  - Note: Using `angular.module('app', []);` sets a module, whereas `angular.module('app');` gets the module. Only set once and get for all other instances.

  - **Methods**: Pass functions into module methods rather than assign as a callback

    ```javascript
    // bad
    angular
      .module('app', [])
      .controller('MainCtrl', function MainCtrl () {

      })
      .service('SomeService', function SomeService () {

      });

    // good
    function MainCtrl () {

    }
    function SomeService () {

    }
    angular
      .module('app', [])
      .controller('MainCtrl', MainCtrl)
      .service('SomeService', SomeService);
    ```

  - This aids with readability and reduces the volume of code "wrapped" inside the Angular framework

**[Back to top](#table-of-contents)**

## Controllers

  - **controllerAs syntax**: Controllers are classes, so use the `controllerAs` syntax at all times

    ```html
    <!-- bad -->
    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

    <!-- good -->
    <div ng-controller="MainCtrl as main">
      {{ main.someObject }}
    </div>
    ```

  - In the DOM we get a variable per controller, which aids nested controller methods avoiding any `$parent` calls

  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

    ```javascript
    // bad
    function MainCtrl ($scope) {
      $scope.someObject = {};
      $scope.doSomething = function () {

      };
    }

    // good
    function MainCtrl () {
      this.someObject = {};
      this.doSomething = function () {

      };
    }
    ```

  - Only use `$scope` in `controllerAs` when necessary, for example publishing and subscribing events using `$emit`, `$broadcast`, `$on` or using `$watch`, try to limit the use of these, however and treat `$scope` uses as special

  - **Inheritance**: Use prototypal inheritance when extending controller classes

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

  - Use `Object.create` with a polyfill for browser support

  - **Zero-logic**: No logic inside a controller, delegate to services

    ```javascript
    // bad
    function MainCtrl () {
      this.doSomething = function () {

      };
    }

    // good
    function MainCtrl (SomeService) {
      this.doSomething = SomeService.doSomething;
    }
    ```

  - Think "skinny controller, fat service"

**[Back to top](#table-of-contents)**

## Services

  - Services are classes and are instantiated with the `new` keyword, use `this` for public methods and variables

    ```javascript
    function SomeService () {
      this.someMethod = function () {

      };
    }
    ```

**[Back to top](#table-of-contents)**

## Factory

  - **Singletons**: Factories are singletons, return a host Object inside each Factory to avoid primitive binding issues

    ```javascript
    // bad
    function AnotherService () {
      var someValue = '';
      var someMethod = function () {

      };
      return {
        someValue: someValue,
        someMethod: someMethod
      };
    }

    // good
    function AnotherService () {
      var AnotherService = {};
      AnotherService.someValue = '';
      AnotherService.someMethod = function () {

      };
      return AnotherService;
    }
    ```

  - This way bindings are mirrored across the host Object, primitive values cannot update alone using the revealing module pattern

**[Back to top](#table-of-contents)**

## Directives

  - **Declaration restrictions**: Only use `custom element` and `custom attribute` methods for declaring your Directives

    ```html
    <!-- bad -->

    <!-- directive: my-directive -->
    <div class="my-directive"></div>

    <!-- good -->

    <my-directive></my-directive>
    <div my-directive></div>
    ```

  - Comment and class name declarations are confusing and should be avoided. Comments do not play nicely with older versions of IE, using an attribute is the safest method for browser coverage.

  - **Templating**: Use `Array.join('')` for clean templating

    ```javascript
    // bad
    function someDirective () {
      return {
        template: '<div class="some-directive">' +
          '<h1>My directive</h1>' +
        '</div>'
      };
    }

    // good
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

  - **DOM manipulation**: Only takes place inside Directives, never a controller/service

    ```javascript
    // bad
    function UploadCtrl () {
      $('.dragzone').on('dragend', function () {
        // handle drop functionality
      });
    }
    angular
      .module('app')
      .controller('UploadCtrl', UploadCtrl);

    // good
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

  - **Naming conventions**: Never `ng-*` prefix custom directives, they might conflict future native directives

    ```javascript
    // bad
    // <div ng-upload></div>
    function ngUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('ngUpload', ngUpload);

    // good
    // <div drag-upload></div>
    function dragUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - Directives and Filters are the _only_ providers that we have the first letter as lowercase, this is due to strict naming conventions in Directives due to the way Angular translates `camelCase` to hyphenated, so `dragUpload` will become `<div drag-upload></div>` when used on an element.

  - **controllerAs**: Use the `controllerAs` syntax inside Directives also

    ```javascript
    // bad
    function dragUpload () {
      return {
        controller: function ($scope) {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);

    // good
    function dragUpload () {
      return {
        controllerAs: 'dragUpload',
        controller: function () {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

**[Back to top](#table-of-contents)**

## Filters

  - **Global filters**: Create global filters only using `angular.filter()` never use local filters inside Controllers/Services

    ```javascript
    // bad
    function SomeCtrl () {
      this.startsWithLetterA = function (items) {
        return items.filter(function (item) {
          return /$a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .controller('SomeCtrl', SomeCtrl);

    // good
    function startsWithLetterA () {
      return function (items) {
        return items.filter(function (item) {
          return /$a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .filter('startsWithLetterA', startsWithLetterA);
    ```

  - This enhances testing and reusability

**[Back to top](#table-of-contents)**

## Routing resolves

  - **Promises**: Resolve dependencies of a Controller in the `$routeProvider` (or `$stateProvider` for `ui-router`) not the Controller itself

    ```javascript
    // bad
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

    // good
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

  - **Controller.resolve property**: Never bind logic to the router itself, reference a `resolve` property for each Controller to couple the logic

    ```javascript
    // bad
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'main',
        controller: 'MainCtrl'
        resolve: {
          doSomething: function () {
            return SomeService.doSomething();
          }
        }
      });
    }

    // good
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    MainCtrl.resolve = {
      doSomething: function () {
        return SomeService.doSomething();
      }
    };

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'main',
        controller: 'MainCtrl'
        resolve: MainCtrl.resolve
      });
    }
    ```

  - This keeps resolve dependencies inside the same file as the Controller, which means more maintainable code and less logic in the router
