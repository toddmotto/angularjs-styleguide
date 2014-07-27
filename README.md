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

    });

    // good
    function MainCtrl () {

    }
    angular
      .module('app', [])
      .controller('MainCtrl', MainCtrl);
    ```

  - This aids with readability and reduces the volume of code "wrapped" inside the Angular framework

**[⬆ top](#table-of-contents)**

## Controllers

  - **controllerAs syntax**: Use the `controllerAs` syntax at all times

    ```html
    // bad
    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

    // good
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

