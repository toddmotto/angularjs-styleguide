# AngularJS styleguide

*Opinionated AngularJS styleguide for teams by [@toddmotto](//twitter.com/toddmotto)*

## Table of Contents

  1. [Modules](#modules)
  1. [Controllers](#controllers)

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

**[top](#table-of-contents)**

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

**[top](#table-of-contents)**

## Services

  - Services are classes and are instantiated with the `new` keyword, use `this` for public methods and variables

    ```javascript
    function SomeService () {
      this.someMethod = function () {

      };
    }
    ```

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

    - **Templating**: Use `Array.join()` for clean templating

      ```javascript
      // bad

      // good

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
      function ngUpload () {
        return {};
      }
      angular
        .module('app')
        .directive('ngUpload', ngUpload);

      // good
      function dragUpload () {
        return {};
      }
      angular
        .module('app')
        .directive('dragUpload', dragUpload);
      ```

    - Directives are the _only_ providers that we have the first letter as lowercase, this is due to strict naming conventions in the way Angular translates `camelCase` to hyphenated, so `focusFire` will become `<input focus-fire>` when used on an element.
