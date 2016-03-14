# Angular styleguide

*Opinionated Angular styleguide for teams.*

#### What is this?

A standardized approach for developing Angular applications at triplelift. This styleguide touches on concepts, syntax and conventions.

### How to use
**Rules are in bold** at the top of each bullet point, with example code shortly thereafter, and the "why" follows after that. The idea is that the rule is the most important - a standard approach has merits in its own right - and should be the most accessible. Code clarifying or even solidifying the rule comes next. If interested, explanations of the rule are provided after that, with the most significant explanation attached to the bullet point prefixed with "**Most importantly,**".  

#### Community
Most of the content and examples in this guide are based off of [John Papa's](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md) and [Todd Motto's](https://github.com/toddmotto/angular-styleguide) style guides. Check their's out to see the originals and to compare thoughts.


## Table of Contents

  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Directives](#directives)
  1. [Routing resolves](#routing-resolves)
  1. [Publish and subscribe events](#publish-and-subscribe-events)
  1. [Performance](#performance)
  1. [Angular wrapper references](#angular-wrapper-references)
  1. [Comment standards](#comment-standards)
  1. [Minification and annotation](#minification-and-annotation)

## Modules

  - **Definitions**: Declare modules without a variable using the setter and getter syntax

	```javascript
	// avoid
	var app = angular.module('app', []);
	app.controller();
	app.factory();

	// recommended
	angular
	  .module('app', [])
	  .controller()
	  .factory();
	```

  - Note: Using `angular.module('app', []);` sets a module, whereas `angular.module('app');` gets the module. Only set once and get for all other instances.


**[Back to top](#table-of-contents)**

## Controllers

  - **controllerAs syntax**: Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the classic "controller with $scope" syntax.

	```html
	<!-- avoid -->
	<div ng-controller="MainCtrl">
	  {{ someObject }}
	</div>
	
	<!-- recommended -->
	<div ng-controller="MainCtrl as vm">
	  {{ vm.someObject }}
	</div>
	```
	
	```javascript
	/* avoid */
	function CustomerController($scope) {
	  $scope.name = {};
	  $scope.sendMessage = function() { };
	}
	```
	
	```javascript
	/* recommended - but see next section */
	function CustomerController() {
	  this.name = {};
	  this.sendMessage = function() { };
	}
	```

	*Why?*
	
	- Controllers are constructed, "newed" up, providing a single new instance. The `controllerAs` syntax more closely resembles "JavaScript constructor" than "classic `$scope`" syntax.
	- Even with `controllerAs`, the controller object is still bound to `$scope`, so you can still bind to the View...this fact ultimately leads some to even see the `controllerAs` syntax as high level syntactic sugar.
	- It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual and easier to read.
	- Since the desired view binding can reached simply by `vm.viewBinding` even if nested within a child `$scope`, it discourages the use of `$parent` in the View.
	- Discourages the use of `$scope` methods inside a controller when they are better placed inside a factory or avoided altogether. 
	- It does not prevent the use of `$scope` methods, such as `$emit`, `$broadcast`, `$on` or `$watch`, which may be accessed by injecting `$scope` if necessary. 
	- **Most importantly**, the `controllerAs` syntax prevents "$scope soup". There may be nothing worse in large scale angular development than running into `ng-click="doOnClick(data)"` in the View, looking into the controller "corresponding" to such View and being unable to locate either the `doOnClick` or `data` definitions. At this point, of course, you must scale the $scope hierarchy until the nearest declarations of each property are found... no fun indeed. Taking advantage of the basic rules of JavaScript prototypal inheritance, placing wonderful `vm` in front of each variable, as in `ng-click="vm.doOnClick(vm.data)"` **ensures** that the controller **actually associated** with the View **contains** the corresponding definitions. That is because when Javascript attempts to locate `vm`, it **will find** the associated controller (bound to `$scope`) since it exists and is bound to `$scope`- success in this regard is guaranteed. Next, JavaScript will attempt to locate `doOnClick` and `data` **on such controller** and...vuala... either the corresponding definitions will be found **bound to such controller** or `undefined` will result.
		

  - **controllerAs 'vm'**: Capture the `this` context of the Controller using `vm` (short for "view model")

	  ```javascript
	  /* avoid */
	  function CustomerController() {
		  this.name = {};
		  this.sendMessage = function() {
		  // wrong!
		  ... this.name ...
		  };
	  }
	  ```
	
	  ```javascript
	  /* recommended */
	  function CustomerController() {
		  var vm = this;
		  vm.name = {};
		  vm.sendMessage = function() {
		  // right!
		  ... vm.name ...
		  };
	  }
	  ```

	*Why?*
	
	- **Most importantly**, the `this` keyword is made available inside every Javascript function in order to gain reference to the invocation context if invoked as a method (e.g. `someInvocationContext.someMethod()`) and the new object created if invoked with the `new` operator as a constructor (e.g. `someNewObject = new SomeConstructor()`). Since angular internally invokes all functions registered as controllers with the `new` operator, any `this` reference inside the controller function necessarily refers to the new object constructed, unless... (a big unless!) further nested inside another function inside the controller. Capturing the top level `this` reference of a controller once and at the top of the controller, instead of repeating `this` calls throughout, encourages consistency and prevents any nested `this` references from incorrectly referencing some object other the object contstructed by the controller and available in the View.
  
  - **Bindable members up top**: Place bindable members at the top of the controller, *perferably* alphabetized, instead of spreading throughout the controller code.
  
	  ```javascript
	  /* avoid */
	  function SessionsController() {
		  var vm = this;
	
		  vm.gotoSession = function() {
			/* ... */
		  };
		  vm.refresh = function() {
			/* ... */
		  };
		  vm.search = function() {
			/* ... */
		  };
		  vm.sessions = [];
		  vm.title = 'Sessions';
	  }
	  ```
	
	  ```javascript
	  /* recommended */
	  function SessionsController(sessionDataService) {
		  var vm = this;
	
		  vm.gotoSession = gotoSession;
		  vm.refresh = refresh;
		  vm.search = search;
		  vm.sessions = [];
		  vm.title = 'Sessions';
		  vm.write = sessionDataService.write; // 1 liner is OK
	
		  ////////////
	
		  function gotoSession() {
			/* */
		  }
	
		  function refresh() {
			/* */
		  }
	
		  function search() {
			/* */
		  }
	  }
	  ```
  
  
	*Why?*
	- Placing bindable members in one place is easier to read and easier to find.
	- Placing bindable members at the top helps you instantly identify which members are "publicly" available in the View.
	- Defining the functions below the bindable members moves the implementation details (& complexity) down.
	- In Javascript, function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions), even if one function references another.
	
  - **Presentational logic only (MVVM)**: Only place presentational logic in controllers; delegate any business logic to **[Services](#services)**.

	```javascript
	// avoid
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

	// recommended
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
	
	Note:
	- The `$http` service and the URL paths are not referenced directly in the controller.
	- Logic in controllers is used only to bind the appropriate data to the controller object.
	
	*Why?*
	- Logic may be reused by multiple controllers when placed within a service.
	- Logic in a service can more easily be isolated in a unit test and mocked (when testing the controller).
	- Simplifies and hides implementation details from the controller.
	
  - **Do not reuse controllers**: One controller per view; do not resuse controllers.
  
	*Why?*
	
	- Reusing controllers with several views is brittle and requires good end-to-end (e2e) test coverage to ensure stability across large applications.
	- Reusable logic can be moved into services should be moved into services (for all the reasons described in the previous rule), leaving controllers simple and focused.
	
  - **Assign controllers in route definitions instead of templates** 
  
	 ```javascript
	 /* avoid */
	
	 // route-config.js
	 angular
		 .module('app')
		 .config(config);
	
	 function config($routeProvider) {
		 $routeProvider
			 .when('/avengers', {
			   templateUrl: 'avengers.html'
			 });
	 }
	 ```
	
	 ```html
	 <!-- avengers.html -->
	 <div ng-controller="AvengersController as vm">
	 </div>
	 ```
	
	 ```javascript
	 /* recommended */
	
	 // route-config.js
	 angular
		 .module('app')
		 .config(config);
	
	 function config($routeProvider) {
		 $routeProvider
			 .when('/avengers', {
				 templateUrl: 'avengers.html',
				 controller: 'Avengers',
				 controllerAs: 'vm'
			 });
	 }
	 ```

	 ```html
	 <!-- avengers.html -->
	 <div>
	 </div>
	 ```
	*Why?*
	- Pairing a controller with a template through `$routeProvider` configurations allows for template reuse. In other words, when `ng-controller` is used inline, that's it... a pairing is made between template and controller, which prevents template reuse with another controller.
	- **Most importantly**, having all controller-template pairings upfront creates a sort of index for your application, which makes it easier to see where everything is, what goes with what and how the data flows throughout your application (see **[Routing with promises](#routing-with-promises)** below for more). 
	
	Note:
	- Although now even easier to do with the introduction of route definitions, *controller* reuse is *still ill-advised* for the reasons above.
	
  - **Make use of 'vm' when `$watch`ing in a controller**: `$watch`'es in a controller should make use of the `controllerAs` syntax.  

	 ```html
	 <input ng-model="vm.title"/>
	 ```
	
	 ```javascript
	 function SomeController($scope, $log) {
	  var vm = this;
	  vm.title = 'Some Title';
	
	  $scope.$watch('vm.title', function(current, original) {
		  $log.info('vm.title was %s', original);
		  $log.info('vm.title is now %s', current);
	  });
	 }
	 ``` 
  
    Note:
      - `$watch`'es are better avoided altogether to minimize `$diget` cycle load or placed in a **[Directive](#directives)**, if necessary. 

**[Back to top](#table-of-contents)**

## Services

  - **Always use a factory**: Use angular's `.factory()` method over its `.service()` method.
  
	  *Service*: acts as a `constructor` function and is instantiated with the `new` keyword. Use `this` for public methods and variables
	
	```javascript
	// avoid
	angular
		.module('app')
		.service('logger', logger);
	
	function logger() {
	  this.logError = function(msg) {
		/* */
	  };
	}
	```

	  *Factory*: is invoked like any ol' function and, accordingly, should return something!
	  
	```javascript  
	// recommended
	angular
		.module('app')
		.factory('logger', logger);
	
	function logger() {
		return {
			logError: function(msg) {
			  /* */
			}
	   };
 	}
	```
	
	*Why?*
	- **Most importantly**, functions registered with the `.factory()` and `.service()` methods are both [services](https://en.wikipedia.org/wiki/Service-oriented_programming) and all such Angular services are singletons (which, according to [angular documentation](https://docs.angularjs.org/guide/services), means that "each component dependent on a service gets a reference to the single instance generated by the service factory"). As a result, `.service()` and `.factory()` only differ in the way Objects are created, but not in what they can create.
	- Since they can perform the same tasks, it makes sense to only use one for the sake of consistency.
	- Since it doesn't require use of the `this` parameter, the `.factory()` method is arguably the simpler of the two.
	
  - **Follow the [Revealing Module Pattern](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript)**: Expose the callable members of the service (its interface) at the top.
  
	 ```javascript
	 /* avoid */
	 function dataService() {
	   var someValue = '';
	   var save = function() {
		 /* */
	   };
	   var validate = function() {
		 /* */
	   };
	
	   return {
		   save: save,
		   someValue: someValue,
		   validate: validate
	   };
	 }
	 ```
	
	 ```javascript
	 /* recommended */
	 function dataService() {
		 var someValue = '';
		 return {
			 save: save,
			 someValue: someValue,
			 validate: validate
		 };
	
		 ////////////
	
		 function save() {
			 /* */
		 };
	
		 function validate() {
			 /* */
		 };
	 }
	 ```
	 
	 *Why?*
	 - Placing accessible members at the top makes is easier to read.
	 - It helps you instantly identify which functions of the factory are publicly exposed and must be unit tested (or mocked).
	 - Defining the functions below the bindable members moves the implementation details (& complexity) down. 
	 - Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions), even if one function references another.
	
  - **Any interaction between controllers and data should be mediated by a service**: XHR calls, local storage, stashing in memory and other data operations belong in a service.
  
	 ```javascript
	 /* avoid */
	
	 // dataservice factory
	 angular
		 .module('app.core')
		 .factory('dataservice', function($http, logger) {
		 
			 return {
				 getAvengers: getAvengers
			 };
		
			 function getAvengers() {
				 return $http.get('/api/maa')
					 .then(getAvengersComplete)
					 .catch(getAvengersFailed);
		
				 function getAvengersComplete(response) {
					 return response.data.results;
				 }
		
				 function getAvengersFailed(error) {
					 logger.error('XHR Failed for getAvengers.' + error.data);
				 }
			 }
		 });
	```
	
	```javascript
	 /* recommended */
	
	 // controller calling the dataservice factory
	 angular
		 .module('app.avengers')
		 .controller('AvengersController', function(dataservice, logger) {
			 var vm = this;
			 vm.avengers = [];
		
			 activate();
		
			 function activate() {
				 return getAvengers().then(function() {
					 logger.info('Activated Avengers View');
				 });
			 }
		
			 function getAvengers() {
				 return dataservice.getAvengers()
					 .then(function(data) {
						 vm.avengers = data;
						 return vm.avengers;
					 });
			 }
		 });
	 ```
	 
	 *Why?*
	 - This makes it easier to test (mock or real) data calls when testing a controller.
	 - Data service implementations may have very specific code to commmunicate with the data repository. This may include headers, how to talk to the data, or other services such as $http. Encapsuating this logic into a data service hides the implementation from the outside consumers (perhaps a controller).
	 - Encapsulation/hiding implementation makes it easier to change.
	 - **Most importantly**, the controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows ***who*** to ask for it. Separating the data services cabins the logic related to ***how*** to get the data and lets the controller be simpler and more focused on the view.


**[Back to top](#table-of-contents)**

## Directives

  - **Declaration restrictions**: Only create element and attribute directives i.e. `{ restrict: 'EA' }`, `{ restrict: 'A' }` or `{ restrict: 'E' }`.

	```html
	<!-- avoid -->

	<!-- directive: my-directive -->
	<div class="my-directive"></div>

	<!-- recommended -->

	<my-directive></my-directive>
	<div my-directive></div>
	```
	*Why?*
  	- Comment and class name declarations are confusing.
  	- Comments do not play nicely with older versions of IE.

  - **Directives are the only place for DOM manipulation**: Favor CSS, animation services and custom angular directives to perform DOM manipuation. For example, if the directive simply hides and shows, use ngHide/ngShow. However, if other javascript DOM manipulation is required, do so in a directive.
  
	```javascript
	// avoid
	function UploadCtrl () {
	  $('.dragzone').on('dragend', function () {
		// handle drop functionality
	  });
	}
	angular
	  .module('app')
	  .controller('UploadCtrl', UploadCtrl);
	
	// recommended
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
	
	*Why?*
	- DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS)
	- **Most importantly**, while it's true that controllers are provided access to the parent-most `$element`, generally speaking, the desired DOM manipulation relates to some child element in the controller's View . Directives, on the other hand, are provided access to the `element` underlying the desired DOM changes. Using such `element` avoids unnecessarily having to search (possibly, by a unique identifier) for the element and more appropriately associates the DOM manipulation logic with the subject of such logic: *the element!*
	- Separating such logic out of the controllers avoids further complicating controller logic while providing encapsulation for DOM related logic. After all, **[Controllers](#controllers)** have their own reason for being that is unrelated to DOM manipulation. 

  - **Naming conventions**: Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo`, declared in HTML as `acme-sales-customer-info`.

	*Why?*
	- Directives enter a global namespace and must be verbosely named as a result.
	- The short prefix identifies the directive's context and origin. For example, a prefix of cc- may indicate that the directive is part of a CodeCamper app while acme- may indicate a directive for the Acme company.
	
  - **All templating directives should be considered "components", restricted to `'E'` and isolated**: Directives that have `template` or `templateUrl` definitions must specify `{ restrict: 'E' }` and `{ scope: {} }`.
	 
	  ```html
	 <!-- avoid -->
	 <div my-example></div>	 
	 ```
	 
	 ```javascript
	 /* avoid */
	 angular
	  .module('app')
	  .directive('myExample', function() {
		return {
		  restrict: 'EA', // <-- note
		  templateUrl: 'app/feature/example.directive.html',
		  controllerAs: 'vm',
		  controller: function() {}
		};
	 });
	
	 ```
	 
	 ```html
	 <!-- recommended -->
	 <my-example max="vm.max"></my-example>	 
	 ```	 
	 
	 ```javascript
	 /* recommended */
	 angular
	  .module('app')
	  .directive('myExample', function() {
		return {
		  restrict: 'E', // <-- note
		  templateUrl: 'app/feature/example.directive.html',
		  scope: {}, // <-- note
		  bindToController: { max: '=' }, // <-- note
		  controllerAs: 'vm',
		  controller: function() {}
		};
	 });
	
	 ```
	 
	 Note:
	 - Direcives with `template` or `templateUrl` definitions will replace any child nodes in the original html with the specified template (unless transcluding).
	 
	*Why?*
	 - Attribute directives, like `ng-click` and `ng-show`, frequently manipulate the DOM or provide other behavior peripheral to data flow and view management. Restricting templating directives to type `'E'` clearly distinguishes them from these.
	 - **Most importantly**, the negative effects of "scope soup" (see **[Controllers](#controllers)** for more) become even more pronounced in the case of directives because they can be placed anywhere in the DOM tree no matter the `$scope` lineage that awaits. Consequently, it can be even more important to isolate directive scopes.
	 - Together, these settings - `{ restrict: 'E' }` and `{ scope: {} }` - constitute what's become known within the angular community as a "component". Possibly the most successful design pattern of Angular 1 development, components were made a part of Angular 2 in a big way. In addition, the `angular.component` method was introduced in Angular 1.5, which is a shortcut (a.k.a syntactic sugar) for a `.directive` that defaults to these settings. Using these settings as frequently as possible, and especially in place of stand-alone controllers, can bring an angular codebase closer to the tried and true styles of the angular community and will better position the application for a future upgrade to angular 2 if/when desired.
	 

  - **Use bindToController and controllerAs**: When binding properties manually in the controller or passing them into the directive through HTML attributes, use `bindToController` and the `controllerAs` syntax.
  
	 ```html
	 <my-example max="77"></my-example>	 
	 ```
	
	 ```javascript
	 /* avoid */
	 angular
	  .module('app')
	  .directive('myExample', function() {
		return {
		  restrict: 'E',
		  templateUrl: 'app/feature/example.directive.html',
		  scope: { max: '=' },
		  controller: function($scope) {
		 	$scope.min = 3;
		 	console.log('CTRL: $scope.min = %s', $scope.min);
		 	console.log('CTRL: $scope.max = %s', $scope.max);
		  }
		};
	 });

	 ```
	 
	 ```html	 
	 <!-- example.directive.html -->
	 <!-- avoid -->
	 <div>hello world</div>
	 <div>max={{max}}<input ng-model="max"/></div>
	 <div>min={{min}}<input ng-model="min"/></div>
	 ```
	
	 ```javascript
	 /* recommended */
	 angular
	  .module('app')
	  .directive('myExample', function() {
		return {
		  restrict: 'E',
		  templateUrl: 'app/feature/example.directive.html',
		  scope: {},
		  bindToController: { max: '=' },
		  controllerAs: 'vm',
		  controller: function() {
		  	var vm = this;
		 	vm.min = 3;
		 	console.log('CTRL: vm.min = %s', vm.min);
		 	console.log('CTRL: vm.max = %s', vm.max);
		  }
		};
	 });

	 ```
	
	 ```html
	 <!-- example.directive.html -->
	 <!-- recommended -->
	 <div>hello world</div>
	 <div>max={{vm.max}}<input ng-model="vm.max"/></div>
	 <div>min={{vm.min}}<input ng-model="vm.min"/></div>
	 ```
	 
	Note:
	- `bindToController` automatically binds properties (that would otherwise be bound directly to `$scope`) directly to directive's controller, which itself is bound to `$scope`.
	 
	*Why?*
	- For all the reasons described in the **[Controllers section](#controller)** above.


**[Back to top](#table-of-contents)**


## Routing resolves

  - **Promises**: Resolve Controller dependencies in the `$routeProvider` (or `$stateProvider` for `ui-router`), not the Controller itself

	```javascript
	// avoid
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

	// recommended
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

  - **Controller.resolve property**: Never bind logic to the router itself. Reference a `resolve` property for each Controller to couple the logic

	```javascript
	// avoid
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

	// recommended
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

  - This keeps resolve dependencies inside the same file as the Controller and the router free from logic

**[Back to top](#table-of-contents)**

## Publish and subscribe events

  - **$scope**: Use the `$emit` and `$broadcast` methods to trigger events to direct relationship scopes only

	```javascript
	// up the $scope
	$scope.$emit('customEvent', data);

	// down the $scope
	$scope.$broadcast('customEvent', data);
	```

  - **$rootScope**: Use only `$emit` as an application-wide event bus and remember to unbind listeners

	```javascript
	// all $rootScope.$on listeners
	$rootScope.$emit('customEvent', data);
	```

  - Hint: Because the `$rootScope` is never destroyed, `$rootScope.$on` listeners aren't either, unlike `$scope.$on` listeners and will always persist, so they need destroying when the relevant `$scope` fires the `$destroy` event

	```javascript
	// call the closure
	var unbind = $rootScope.$on('customEvent'[, callback]);
	$scope.$on('$destroy', unbind);
	```

  - For multiple `$rootScope` listeners, use an Object literal and loop each one on the `$destroy` event to unbind all automatically

	```javascript
	var unbind = [
	  $rootScope.$on('customEvent1'[, callback]),
	  $rootScope.$on('customEvent2'[, callback]),
	  $rootScope.$on('customEvent3'[, callback])
	];
	$scope.$on('$destroy', function () {
	  unbind.forEach(function (fn) {
		fn();
	  });
	});
	```

**[Back to top](#table-of-contents)**

## Performance

  - **One-time binding syntax**: In newer versions of Angular (v1.3.0-beta.10+), use the one-time binding syntax `{{ ::value }}` where it makes sense

	```html
	// avoid
	<h1>{{ vm.title }}</h1>

	// recommended
	<h1>{{ ::vm.title }}</h1>
	```
	
	*Why?* : Binding once removes the watcher from the scope's `$$watchers` array after the `undefined` variable becomes resolved, thus improving performance in each dirty-check
	
  - **Consider $scope.$digest**: Use `$scope.$digest` over `$scope.$apply` where it makes sense. Only child scopes will update

	```javascript
	$scope.$digest();
	```
	
	*Why?* : `$scope.$apply` will call `$rootScope.$digest`, which causes the entire application `$$watchers` to dirty-check again. Using `$scope.$digest` will dirty check current and child scopes from the initiated `$scope`

**[Back to top](#table-of-contents)**

## Angular wrapper references

  - **$document and $window**: Use `$document` and `$window` at all times to aid testing and Angular references

	```javascript
	// avoid
	function dragUpload () {
	  return {
		link: function ($scope, $element, $attrs) {
		  document.addEventListener('click', function () {

		  });
		}
	  };
	}

	// recommended
	function dragUpload ($document) {
	  return {
		link: function ($scope, $element, $attrs) {
		  $document.addEventListener('click', function () {

		  });
		}
	  };
	}
	```

  - **$timeout and $interval**: Use `$timeout` and `$interval` over their native counterparts to keep Angular's two-way data binding up to date

	```javascript
	// avoid
	function dragUpload () {
	  return {
		link: function ($scope, $element, $attrs) {
		  setTimeout(function () {
			//
		  }, 1000);
		}
	  };
	}

	// recommended
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

**[Back to top](#table-of-contents)**

## Comment standards

  - **jsDoc**: Use jsDoc syntax to document function names, description, params and returns

	```javascript
	/**
	 * @name SomeService
	 * @desc Main application Controller
	 */
	function SomeService (SomeService) {

	  /**
	   * @name doSomething
	   * @desc Does something awesome
	   * @param {Number} x - First number to do something with
	   * @param {Number} y - Second number to do something with
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

**[Back to top](#table-of-contents)**

## Minification and annotation

  - **ng-annotate**: Use [ng-annotate](//github.com/olov/ng-annotate) for Gulp as `ng-min` is deprecated, and comment functions that need automated dependency injection using `/** @ngInject */`

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

  - Which produces the following output with the `$inject` annotation

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

**[Back to top](#table-of-contents)**

## Angular docs
For anything else, including API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions.

## License

#### (The MIT License)

Copyright (c) 2015-2016 Todd Motto

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
