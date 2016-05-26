# Guía de estilo Angular

*Guía de estilos de Angular para equipos por [@toddmotto](//twitter.com/toddmotto)*

Un enfoque estandarizado para desarrollar aplicaciones Angular en equipo. Esta guía de estilo detalla conceptos, sintaxis, convenciones y esta basada en mi experiencia al [escribir](http:////toddmotto.com), [hablar](https://speakerdeck.com/toddmotto) y construir aplicaciones Angular.

<a href="http://voux.io" target="_blank"><img src="http://www.digital-results.com/hubfs/Images/voux-logo__colour.svg"></a>

#### Comunidad
[John Papa](//twitter.com/John_Papa) y yo hemos discutido a fondo sobre los patrones de estilo para Angular y ambos hemos publicado guías separadas. Gracias a estas discusiones, he aprendido algunos bunos tips de John que han ayudado a dar forma a esta guía. Ambos creamos nuestra propia visión de una guía de estilo. Los invito a revisar [la suya](//github.com/johnpapa/angularjs-styleguide) para comparar ideas.

> Lee el [artículo original](http://toddmotto.com/opinionated-angular-js-styleguide-for-teams) que dio pie a esto.

## Tabla de contenidos

  1. [Módulos](#modules)
  1. [Controladores](#controllers)
  1. [Servicios y Factories](#services-and-factory)
  1. [Directivas](#directives)
  1. [Filtros](#filters)
  1. [Resolución de rutas](#routing-resolves)
  1. [Publicar y suscribir eventos](#publish-and-subscribe-events)
  1. [Rendimiento](#performance)
  1. [Referencias del Wrapper de Angular](#angular-wrapper-references)
  1. [Estandares en comentarios](#comment-standards)
  1. [Minificación y anotación](#minification-and-annotation)

## Módulos

  - **Definición**: Declara modulos sin una variable usando la sintaxis de setter y getter.

    ```javascript
    // evitar
    var app = angular.module('app', []);
    app.controller();
    app.factory();

    // recomendado
    angular
      .module('app', [])
      .controller()
      .factory();
    ```

  - Nota: Utilizando `angular.module('app', []);` establece un módulo (setter), mientras que `angular.module('app');` obtiene el módulo (getter). Utiliza el setter una vez y el getter para el resto de las instancias.

  - **Métodos**: Pasa funciones en los métodos del módulo en lugar de asignarlas como callback.

    ```javascript
    // evitar
    angular
      .module('app', [])
      .controller('MainCtrl', function MainCtrl () {

      })
      .service('SomeService', function SomeService () {

      });

    // recomendado
    function MainCtrl () {

    }
    function SomeService () {

    }
    angular
      .module('app', [])
      .controller('MainCtrl', MainCtrl)
      .service('SomeService', SomeService);
    ```

  - Las clases de ES6 no tendrán "hoisting", por lo que si tu código se basa en esto se romperá.
  
  - Esto ayuda con la legibilidad y reduce el volumen de código anidado dentro del framework de Angular.
  
  - **IIFE scoping**: Para evitar contaminar el scope global con nuestra declaración de funciones que van pasando dentro de Angular, asegurate que las tareas de compilado envuelvan los archivos concatenados dentro de un IFFE.

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


**[Volver al inicio](#table-of-contents)**

## Controladores

  - **Sintaxis controllerAs**: Los controladores son clases, así que usa la sintaxis `controllerAs` todo el tiempo.

    ```html
    <!-- evitar -->
    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

    <!-- recomendado -->
    <div ng-controller="MainCtrl as vm">
      {{ vm.someObject }}
    </div>
    ```

  - En el DOM tenemos una variable por controlador, lo que permite controladores anidados, evitando cualquier llamada `$parent`.

  - La sintaxis `controllerAs` utiliza `this` dentro del controlador, que tiene enlazado el `$scope`.

    ```javascript
    // evitar
    function MainCtrl ($scope) {
      $scope.someObject = {};
      $scope.doSomething = function () {

      };
    }

    // recomendado
    function MainCtrl () {
      this.someObject = {};
      this.doSomething = function () {

      };
    }
    ```
    
  - Solo utiliza `$scope` en `controllerAs` cuando sea necesario; por ejemplo, publicando y suscribiendo eventos utilizando `$emit`, `$broadcast`, `$on` o `$watch`. Intenta limitar el uso de estos, no obsante, trata el `$scope` como un caso de uso especial.

  - **Herencia**: Usa herencia de prototipos al extender clases de controlador.
  
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

  - Utiliza `Object.create` con un polyfill para el soporte de los navegadores.

  - **controllerAs 'vm'**: Captura el contexto de `this` del Controlador utilizando `vm`, representando `ViewModel`

    ```javascript
    // evitar
    function MainCtrl () {
      var doSomething = function () {

      };
      this.doSomething = doSomething;
    }

    // recomendado
    function MainCtrl () {
      var vm = this;
      var doSomething = function () {
        
      };
      vm.doSomething = doSomething;
    }
    ```

    *¿Por qué?* : El contexto de una función cambia el valor de `this`, utilizalo para evitar llamadas `.bind()` y problemas con el scope.
    
  - **ES6**: Evitar `var vm = this;` cuando utilizas  ES6

    ```javascript
    // evitar
    function MainCtrl () {
      let vm = this;
      let doSomething = arg => {
        console.log(vm);
      };
      
      // exports
      vm.doSomething = doSomething;
    }

    // recomendado
    function MainCtrl () {
      
      let doSomething = arg => {
        console.log(this);
      };
      
      // exports
      this.doSomething = doSomething;
      
    }
    ```

    *¿Por qué?* : Usa arrow functions en ES6 cuando es necesario acceder al valor de `this` explicitamente.

  - **Solo lógica de presentación (MVVM)**: La lógica de presentación solo debe estar en un controlador, evita la lógica de negocios (delegala a los Servicios)

    ```javascript
    // evitar
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

    // recomendado
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

    *¿Por qué?* : Los controladores deberían obtener los datos del modelo de los servicios, evitando cualquier lógica de negocio. Los controladores deberían de actuar como una Vista-Modelo y controlar el flujo de datos entre el Model y la capa de presentación de la vista. La lógica de negocio en controladores hace que probar Servicios sea imposible.
    
**[Volver al inicio](#table-of-contents)**

## Servicios y Factory

  - Todos los servicios en Angular son singletons, utilizando `.service()` o `.factory()` difiera la manera en que los objetos son creados.

  **Servicios**: Actuan como una función `constructor` function y son instanciados con la palabra clave `new`. Usa `this` para metodos y variables publicas.

    ```javascript
    function SomeService () {
      this.someMethod = function () {

      };
    }
    angular
      .module('app')
      .service('SomeService', SomeService);
    ```

  **Factory**: Lógica del negocio o modulos provider, retorna un objeto o un closure.

  - Siempre regresa un objeto del entorno en lugar de seguir el [Revealing Module Pattern](https://toddmotto.com/mastering-the-module-pattern/), esto debido a la manera en que las referencias del objeto son ligadas y actualizadas.

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

    *¿Por qué?* : Los valores primitivos no pueden ser actualizados solo utilizando el [Revealing Module Pattern](https://toddmotto.com/mastering-the-module-pattern/).

**[Volver al inicio](#table-of-contents)**

## Directivas

  - **Restricciones de declaración**: Solo utiliza los métodos `custom element` y `custom attribute` para declarar tus Directivas(`{ restrict: 'EA' }`) dependiendo del rol de la directiva.

    ```html
    <!-- evitar -->

    <!-- directive: my-directive -->
    <div class="my-directive"></div>

    <!-- recomendado -->

    <my-directive></my-directive>
    <div my-directive></div>
    ```

  - En comentarios y declaraciones de clases son confusas y deben ser evitadas. Los comentarios no se llevan bien con versiones previas de IE. Utilizan un atributo es la manera más segura de cubrir el navegador.

  - **Plantillas**: Utiliza `Array.join('')` para plantillas claras.

    ```javascript
    // evitar
    function someDirective () {
      return {
        template: '<div class="some-directive">' +
          '<h1>My directive</h1>' +
        '</div>'
      };
    }

    // recomendado
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

    *¿Por qué?* : Mejora la lectura y además el código puede ser identado con propiedad, también evita que el operador `+` que es menos claro y puede llevar a error si es usado incorrectamente para dividir lineas.
    
  - **Manipulación del DOM**: Toma lugar solo dentro de la directiva, nunca en un controlador/servicio.

    ```javascript
    // evitar
    function UploadCtrl () {
      $('.dragzone').on('dragend', function () {
        // handle drop functionality
      });
    }
    angular
      .module('app')
      .controller('UploadCtrl', UploadCtrl);

    // recomendado
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

  - **Convenciones de nomenclatura**: Nunca usar el prefijo `ng-*` en directivas personalizadas, podrían causar conflicto en futuras directivas nativas.

    ```javascript
    // evitar
    // <div ng-upload></div>
    function ngUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('ngUpload', ngUpload);

    // recomendado
    // <div drag-upload></div>
    function dragUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - Las directivas y los filtros son las _únicos_ providers que tienen la primera letra en mínuscula; esto es debido a estrictas convenciones de nomenclatura en las directivas. Angular une con guiones el `camelCase`, así que `dragUpload` se convertirá en `<div drag-upload></div>` cuando es usado sobre un elemento.

  - **controllerAs**: Utiliza la sintaxis de `controllerAs` dentro de las directivas.

    ```javascript
    // evitar
    function dragUpload () {
      return {
        controller: function ($scope) {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);

    // recomendado
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

**[Volver al inicio](#table-of-contents)**

## Filtros

  - **Filtros globales**: Crear filtros globales utilizando solo `angular.filter()`. Nunca uses filtros locales dentro de un controlador/servicio.

    ```javascript
    // evitar
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

    // recomendado
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

  - Esto mejor el testing y la reutilización.

**[Volver al inicio](#table-of-contents)**

## Resolución de rutas

  - **Promesas**: Resuelve las dependencias del controlador en el `$routeProvider` (or `$stateProvider` o `ui-router`), no en el controlador mismo.

    ```javascript
    // evitar
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

    // recomendado
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

  - **Propiedad Controller.resolve**: Nunca unir lógica al mismo router. Referencia una propiedad `resolve` por cada controlador para acoplar la lógica


    ```javascript
    // evitar
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

    // recomendado
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

  - Esto permite resolver las dependencias dentro del mismo archivo tanto el controlador y el router libre de lógica

**[Volver al inicio](#table-of-contents)**

## Publicación y suscripción de eventos

  - **$scope**: Usa los métodos `$emit` y `$broadcast` para activar eventos a solo scopes con relación directa

    ```javascript
    // up the $scope
    $scope.$emit('customEvent', data);

    // down the $scope
    $scope.$broadcast('customEvent', data);
    ```

  - **$rootScope**: Usa solo `$emit` como un application-wide event bus y recuerda hacer unbind a los listeners

    ```javascript
    // todos los listener $rootScope.$on
    $rootScope.$emit('customEvent', data);
    ```

  - Pista: Debido a que `$rootScope` nunca es destruido, los listeners `$rootScope.$on` támpoco lo son,  a diferencia de los listener `$scope.$on` siempre persistirán, por lo que se necesitan destruir cuando el `$scope` relevante dispara el evento `$destroy`

    ```javascript
    // llamar al closure
    var unbind = $rootScope.$on('customEvent'[, callback]);
    $scope.$on('$destroy', unbind);
    ```

  - Para múltiples listeners de `$rootScope`, utilizar un Objecto e itera en cada uno en el evento `$destroy`para desenlazar todos de manera automática.

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

**[Volver al inicio](#table-of-contents)**

## Rendimiento

  - **One-time binding syntax**: En versiones nuevas de Angular (v1.3.0-beta.10+), utiliza la sintaxis one-time bindng `{{ ::value }}` cuando haga sentido

    ```html
    // evitar
    <h1>{{ vm.title }}</h1>

    // recomendado
    <h1>{{ ::vm.title }}</h1>
    ```
    
    *¿Por qué?* : Vincular una vez remueve el watcher del arreglo `$watchers` de los scope  después de que la variable `undefined` resuelve, mejorando así el rendimiento en cada dirty-check
   
  - **Considere $scope.$digest**: Utiliza `$scope.$digest` sobre `$scope.$apply` cuando haga sentido. Solo los scope hijos serán actualizados

    ```javascript
    $scope.$digest();
    ```
    
    *¿Por qué?* : `$scope.$apply` llamará `$rootScope.$digest`, lo que causa el chequeo completo `$$watchers` de la aplicación completa una vez más. Utilizando `$scope.$digest` comprobará el dirty-check actual y los scope hijo del `$scope` iniciado.
    
**[Volver al inicio](#table-of-contents)**

## Referencias al Angular wrapper

  - **$document y $window**: Utiliza `$document` y `$window` en todo momento para ayudar a las pruebas y referencias de Angular
  
    ```javascript
    // evitar
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          document.addEventListener('click', function () {

          });
        }
      };
    }

    // recomendado
    function dragUpload ($document) {
      return {
        link: function ($scope, $element, $attrs) {
          $document.addEventListener('click', function () {

          });
        }
      };
    }
    ```

  - **$timeout e $interval**: Utiliza `$timeout` e `$interval` sobre sus contrapartes nativas para permitir el two-way data binding de Angular funcionando
   
    ```javascript
    // evitar
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          setTimeout(function () {
            //
          }, 1000);
        }
      };
    }

    // recomendado
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

**[Volver al inicio](#table-of-contents)**

## Normas de comentario

  - **jsDoc**: Utiliza la sintaxis de jSDoc para documentar nombre de funciones, descripciones, parametros y retornos

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

**[Volver al inicio](#table-of-contents)**

## Minificación y anotación

  - **ng-annotate**: Utiliza [ng-annotate](//github.com/olov/ng-annotate) para Gulp porqué `ng-min` está en desuso, y comenta funciones que necesiten injección de dependencias automática utilizando`/** @ngInject */`

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

  - Que produce la siguiente salida con la anotación `$inject`

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

**[Volver al inicio](#table-of-contents)**

## Documentación Angular
Para algo más, incluyendo referencia al API, revisa la [Documentación de Angular] (//docs.angularjs.org/api).

## Contribuyendo

Abrir un tema primero para discutir posibles cambios / adiciones.

## Licencia

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
