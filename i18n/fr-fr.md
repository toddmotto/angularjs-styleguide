# Guide de style AngularJS (ES2015)

### Architecture, structure de fichier, components (composants), one-way dataflow et bonnes pratiques

---

> Vous voulez découvrir un exemple de structure de projet ? Découvrez cette [architecture d'un projet AngularJS 1.5](https://github.com/toddmotto/angular-1-5-components-app).

---

*Un guide de style adapté aux équipes par [@toddmotto](//twitter.com/toddmotto)*

Cette architecture et guide de style a été réécrit avec ES2015 en tête, ainsi que les changements d'AngularJS 1.5+ pour une future mise à jour vers Angular. Ce guide inclut les nouvelles bonnes pratiques pour du one-way dataflow, la délégation d'évènements, l'architecture des components et le routing de ceux-ci.

Vous pouvez trouver l'ancien guide de style [ici](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5), et le raisonnement derrière le nouveau [ici](https://toddmotto.com/rewriting-angular-styleguide-angular-2).

> Rejoignez l'Ultime expérience d'apprentissage AngularJS pour maitriser complètement les fonctionnalités Angular, de base, et avancées, pour construire de vraies applications rapides et scalables.

<a href="https://courses.toddmotto.com" target="_blank"><img src="https://toddmotto.com/img/ua.png"></a>

## Table des matières

  1. [Architecture modulaire](#architecture-modulaire)
    1. [Théorie des modules](#théorie-des-modules)
    1. [Module Root](#module-root)
    1. [Module Component](#module-component)
    1. [Module Common](#module-common)
    1. [Modules Low-level](#modules-low-level)
    1. [Convention de nommage des fichiers](#convention-de-nommage-des-fichiers)
    1. [Structure des dossiers scalable](#structure-des-dossiers-scalable)
  1. [Components](#components)
    1. [Théorie des components](#théorie-des-components)
    1. [Propriétés supportées](#propriétés-supportées)
    1. [Controllers](#controllers)
    1. [One-way dataflow et évènements](#one-way-dataflow-et-évènements)
    1. [Components Stateful](#components-stateful)
    1. [Components Stateless](#components-stateless)
    1. [Components Routed](#components-routed)
  1. [Directives](#directives)
    1. [Théorie des directives](#théorie-des-directives)
    1. [Propriétés recommandées](#propriétés-recommandées)
    1. [Constantes ou Classes](#constantes-ou-classes)
  1. [Services](#services)
    1. [Théorie des services](#théorie-des-services)
    1. [Classes pour les services](#classes-pour-les-services)
  1. [Styles](#styles)
  1. [ES2015 et outils](#es2015-et-outils)
  1. [State management](#state-management)
  1. [Ressources](#ressources)
  1. [Documentation](#documentation)
  1. [Contribuer](#contribuer)

# Architecture modulaire

Chaque module dans une application Angular est un module component. Un module component est la définition qui encapsule la logique, les templates, le routage et les components enfants.

### Théorie des modules

La structure d'un module se retrouve directement dans la structure de nos répertoires, ce qui permet de garder les choses maintenables et prédictibles. Idéalement on devrait avoir trois modules de haut niveau: root, component et common. Le module root définit le module de base qui bootstrap notre app, et le template correspondant. On importe ensuite nos modules component et common dans le root module pour inclure ses dépendances. Les modules component et common font appel aux modules de bas niveau, qui contiennent nos components, controllers, services, directives, filtres, et nos tests pour chaque fonctionnalité réutilisables

**[Haut de page](#table-des-matieres)**

### Module Root

Un module root commence par un root component qui définit l'élément de base pour l'application, avec le routage défini, l'exemple ci-dessous utilise `ui-view` depuis `ui-router`.

```js
// app.component.js
export const AppComponent = {
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

Un module root est ensuite créé, avec `AppComponent` puis importé et enregistré avec `.component('app', AppComponent)`. Les imports supplémentaires pour les sous-modules (modules component et common) sont censés inclure tous les components pertinents pour l'application. Vous remarquerez que les styles sont aussi importés ici, nous verrons cela dans d'autres chapitres dans le guide.

```js
// app.module.js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { AppComponent } from './app.component';
import { ComponentsModule } from './components/components.module';
import { CommonModule } from './common/common.module';
import './app.scss';

export const AppModule = angular
  .module('app', [
    ComponentsModule,
    CommonModule,
    uiRouter
  ])
  .component('app', AppComponent)
  .name;
```

**[Haut de page](#table-des-matieres)**

### Module Component

Un module component contient les références pour tous les components réutilisables. Ci-dessus nous avons importé `ComponentsModule` puis injecté dans le module Root, cela nous donne un seul endroit où importer tous les components de l'app. Ces modules en question sont dissociés de tous les autres modules et peuvent donc etre aisément utilisé dans d'autres applications.

```js
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

**[Haut de page](#table-des-matieres)**

### Module Common

Le module Common contient les références pour tous les components spécifiques à l'application, ceux qu'on ne souhaite pas utiliser dans une autre application. Cela peut être de la mise en page (layout), de la navigation ou des bas de pages (footers). Ci-dessus nous avons importé `CommonModule` puis injecté dans le module Root, cela nous donne un seul endroit où importer tous les components common de l'app.

```js
import angular from 'angular';
import { NavModule = from './nav/nav.module';
import { FooterModule } from './footer/footer.module';

export const CommonModule = angular
  .module('app.common', [
    NavModule,
    FooterModule
  ])
  .name;
```

**[Haut de page](#table-des-matieres)**

### Modules Low-level

Les modules Low-level sont des modules component individuels qui contiennent la logique de chaque fonctionnalités. Ils definiront chacun un module, qui sera importé à un module de plus haut niveau, comme un component ou un module common (exemple ci-dessous). Ne pas oublier d'ajouter le suffix `.name` à chaque `export` à la création d'un _nouveau_ module et non pas lorsque l'on fait référence à un module existant. Vous noterez que les définitions de routage (routing) existent ici aussi, un prochain chapitre couvrira ce point.

```js
import angular from 'angular';
import uiRouter from 'angular-ui-router';
import { CalendarComponent } from './calendar.component';
import './calendar.scss';

export const CalendarModule = angular
  .module('calendar', [
    uiRouter
  ])
  .component('calendar', CalendarComponent)
  .config(($stateProvider, $urlRouterProvider) => {
    'ngInject';
    $stateProvider
      .state('calendar', {
        url: '/calendar',
        component: 'calendar'
      });
    $urlRouterProvider.otherwise('/');
  })
  .name;
```

**[Haut de page](#table-des-matieres)**

### Convention de nommage des fichiers

Gardez des noms simples, en minuscules, utilisez le nom du component, e.g. `calendar.*.js*`, `calendar-grid.*.js` - avec le nom du type de fichier au milieu. Utilisez `*.module.js` pour le fichier de définition de module, pour pouvoir importer ces modules par nom de dossier.

```
calendar.module.js
calendar.component.js
calendar.service.js
calendar.directive.js
calendar.filter.js
calendar.spec.js
calendar.html
calendar.scss
```

**[Haut de page](#table-des-matieres)**

### Structure des dossiers scalable

Structurer les dossiers du projet est extrêmement important, cela se transcrit par une structure scalable et predictible. Ci-dessous, un exemple de structure qui suit une architecture component modulaire.

```
├── app/
│   ├── components/
│   │  ├── calendar/
│   │  │  ├── calendar.module.js
│   │  │  ├── calendar.component.js
│   │  │  ├── calendar.service.js
│   │  │  ├── calendar.spec.js
│   │  │  ├── calendar.html
│   │  │  ├── calendar.scss
│   │  │  └── calendar-grid/
│   │  │     ├── calendar-grid.module.js
│   │  │     ├── calendar-grid.component.js
│   │  │     ├── calendar-grid.directive.js
│   │  │     ├── calendar-grid.filter.js
│   │  │     └── calendar-grid.spec.js
│   │  │     └── calendar-grid.html
│   │  │     └── calendar-grid.scss
│   │  ├── events/
│   │  │  ├── events.module.js
│   │  │  ├── events.component.js
│   │  │  ├── events.directive.js
│   │  │  ├── events.service.js
│   │  │  ├── events.spec.js
│   │  │  ├── events.html
│   │  │  ├── events.scss
│   │  │  └── events-signup/
│   │  │     ├── events-signup.module.js
│   │  │     ├── events-signup.component.js
│   │  │     ├── events-signup.service.js
│   │  │     └── events-signup.spec.js
│   │  │     ├── events-signup.html
│   │  │     ├── events-signup.scss
│   │  └── components.module.js
│   ├── common/
│   │  ├── nav/
│   │  │     ├── nav.module.js
│   │  │     ├── nav.component.js
│   │  │     ├── nav.service.js
│   │  │     └── nav.spec.js
│   │  │     ├── nav.html
│   │  │     ├── nav.scss
│   │  ├── footer/
│   │  │     ├── footer.module.js
│   │  │     ├── footer.component.js
│   │  │     ├── footer.service.js
│   │  │     └── footer.spec.js
│   │  │     └── footer.html
│   │  │     └── footer.scss
│   │  └── common.module.js
│   ├── app.module.js
│   └── app.component.js
│   └── app.scss
└── index.html
```

Le dossier racine contient simplement `index.html` et `app/`, un répertoire dans lequel existent tous les modules : root, component, common et low-level; avec leur styles et html pour chaque composant.

**[Haut de page](#table-des-matieres)**

# Components

### Théorie des components

Les Components sont en fait des templates avec un controller. Ils ne sont _pas_ des Directives, et vous ne devriez pas remplacer les Directives par des components, à moins d'upgrader les "Directives template" avec des controllers, qui sont mieux adaptés en tant que component. Les components contiennent aussi des bindings (liaisons) qui définissent les entrées et sorties pour les données et les évènements, les lifecycle hooks (fonctions qui s'exécutent à des moments clés du cycle de vie d'un component), et la possibilité d'utiliser le one-way data flow (propagation unidirectionnelle des données) et les event Objects (objets evenement) qui permettent de transmettre des données à un component parent. Le Component est le nouveau standard AngularJS 1.5 et au dessus. Tout ce que l'on créé qui contient un template et un controller sera vraisemblablement un component, qui peut être un component stateful (avec état), stateless (sans état), ou routed (routé). On peut réfléchir à un "component" comme un morceau de code complet, et non pas juste sa definition Object `.component()`. Explorons maintenant quelques bonne pratiques des components, et examinons la façon dont vous devriez les structurer via les concepts de components stateful, stateless, et routed.

**[Haut de page](#table-des-matieres)**

### Propriétés supportées

Voilà ci-dessous les propriétés supportées pour `.component()` que vous pouvez/devriez utiliser:

| Property | Support |
|---|---|
| bindings | Oui, utiliser seulement `'@'`, `'<'`, `'&'`|
| controller | Oui |
| controllerAs | Oui, par defaut est à la valeur `$ctrl` |
| require | Oui (nouvelle syntaxe Object) |
| template | Oui |
| templateUrl | Oui |
| transclude | Oui |

**[Haut de page](#table-des-matieres)**

### Controllers

Les Controllers ne devraient être utilisés qu'aux cotés de components, jamais autre part. Si vous pensez avoir besoin d'un controller, en réalité vous avez surement besoin d'un component stateless qui va gérer ce comportement particulier.

Voilà quelques conseils pour l'utilisation de `Class` pour les controllers:

* Enlever le nom "Controller", mais utiliser plutôt `controller: class TodoComponent {...}` pour faciliter la migration vers Angular
* Toujours utiliser le `constructor` pour l'injection de dépendance
* Utiliser la syntaxe [ng-annotate](https://github.com/olov/ng-annotate)'s `'ngInject';` pour les annotations `$inject`
* Si vous avez besoin d'accèder au lexical scope (portée lexicale) utilisez les fonctions flechées (arrow function).
* Une alternative aux fonctions flechées pour acceder au lexical scope `let ctrl = this;` est aussi acceptable et peut etre plus logique en fonction du cas.
* Lier toutes les fonctions publiques directement à la `Class`.
* Mettez à profit l'utilisation des lifecycle hooks appropriés, `$onInit`, `$onChanges`, `$postLink` et `$onDestroy`.
  * Note: `$onChanges` est appelé avant `$onInit`, voir la section [ressources](#ressources) pour des articles qui traitent du sujet plus en profondeur.
* Utilisez `require` de pair avec `$onInit` pour référencer toute logique héritée.
* Ne pas redéfinir la valeur par defaut `$ctrl` qui est l'alias de la syntaxe `controllerAs`, n'utilisez donc pas du tout `controllerAs`

**[Haut de page](#table-des-matieres)**

### One-way dataflow et évènements

Le one-way dataflow (propagation unidirectionnelle des données) a été introduit dans AngularJS 1.5, et redéfinit la communication entre components.

Voici quelques conseils pour l'utilisation du one-way dataflow

* Dans les components qui reçoivent des données, toujours utiliser la synthaxe `'<'` de one-way databinding.
* _Ne plus_ utiliser la synthaxe `'='` de two-way databinding.
* Les components qui possèdent des `bindings` devraient utiliser `$onChanges` pour cloner les données issues du one-way databinding, casser les passages d'objets par référence, et mettre à jour les données parentes.
* Utiliser `$event` comme argument de fonction dans la méthode parente (voir exemple stateful dessous `$ctrl.addTodo($event)`).
* Passer un Object `$event: {}`  depuis un stateless component (voir exemple stateless dessous `this.onAddTodo`).
  * Bonus: Utiliser un wrapper de `EventEmitter`  avec `.value()` pour mimer le comportement d'Angular, cela permet d'éviter la création manuelle d'Object `$event`
* Pourquoi ? Cela imite le comportement d'Angular et permet de rester cohérent entre chaque components. Cela permet aussi d'avoir un état (state) prédictible.

**[Haut de page](#table-des-matieres)**

### Components stateful

Definissons ce que l'on appelle "component stateful"

* Fetch le state, communique essentiellement avec une API backend au travers d'un service
* Ne fais pas d'opération de mutation sur le state
* Render les components enfants qui opèrent des mutations sur le state
* Aussi appelés components intelligents / contenants (smart / container)

Un exemple de component stateful, complet avec ses définitions de module low-level (seulement une démonstration, une partie du code à été omise par soucis de clareté):

```js
/* ----- todo/todo.component.js ----- */
import templateUrl from './todo.html';

export const TodoComponent = {
  templateUrl,
  controller: class TodoComponent {
    constructor(TodoService) {
      'ngInject';
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
};

/* ----- todo/todo.html ----- */
<div class="todo">
  <todo-form
    todo="$ctrl.newTodo"
    on-add-todo="$ctrl.addTodo($event);"></todo-form>
  <todo-list
    todos="$ctrl.todos"></todo-list>
</div>

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;
```

Cet exemple montre un component stateful, qui fetch le state dans un controller, au travers d'un service, puis qui le passe à ses component stateless enfants. Remarquez qu'il n'y a aucune Directive utilisée comme `ng-repeat` ou autres à l'intérieur du template.
À la place, les données et les fonctions sont déléguées dans les component stateless `<todo-form>` et `<todo-list>`.

**[Haut de page](#table-des-matieres)**

### Components stateless

Definissons ce qu'on appelle "component stateless"

* Possède des inputs et outputs définis en utilisant `bindings: {}`
* Les données arrivent au component via des bindings d'attributs (inputs)
* Les données quittent le component via des events (outputs)
* Opère des mutations sur le state, renvoie les données vers le parent sur demande (comme un click ou un event submit)
* Ne se soucie pas d'où viennent les données, il est stateless
* Sont des components hautement réutilisables
* Aussi appelés component idiot / de présentation

Un exemple de component stateless (utilisons `<todo-form>` comme exemple), complet avec les definitions de module low-level (seulement une démonstration, une partie du code à été omise par soucis de clareté):

```js
export const TodoComponent = {
  templateUrl,
  controller: class TodoComponent {
    constructor(TodoService) {
      'ngInject';
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
};

/* ----- todo/todo.html ----- */
<div class="todo">
  <todo-form
    todo="$ctrl.newTodo"
    on-add-todo="$ctrl.addTodo($event);"></todo-form>
  <todo-list
    todos="$ctrl.todos"></todo-list>
</div>

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;
```

Notez comme le component `<todo-form>` ne fetch pas de state, il le reçoit simplement, opère une mutation d'un Object via le code logique du controller associé au component, et le rend au component parent au travers des bindings de propriétés. Dans cet exemple, le lifecycle hook `$onChanges` fabrique un clone du binding Object original `this.todo` et le réassigne, ce qui veut dire que les données parentes ne sont pas affectées tant que le formulaire n'est pas soumis, cote a cote avec la nouvelle syntaxe de one-way dataflow `'<'`.

**[Haut de page](#table-des-matieres)**

### Components Routed

Definissons ce qu'on appelle un "routed component".

* C'est fondamentalement un component stateful, avec des definitions de routing
* Plus de fichiers `router.js`
* On utilise les routed components pour définir leur propre logique de routing
* Les "input" de données pour le component est fait via le route resolve (optionel, toujours disponible dans le controller avec les appels au service)

Pour cet exemple, nous allons prendre le component existant `<todo>`, le refactorer pour qu'il utilise une definition de route et des `bindings` sur le component qui recevront les données (le secret ici avec `ui-router` réside dans les propriétés `resolve` que l'on créé, en l'occurrence `todoData` map directement pour nous vers les `bindings`). On le considère comme un routed component parce que c'est essentiellement une "view":

```js
export const TodoComponent = {
  templateUrl,
  controller: class TodoComponent {
    constructor(TodoService) {
      'ngInject';
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
};

/* ----- todo/todo.html ----- */
<div class="todo">
  <todo-form
    todo="$ctrl.newTodo"
    on-add-todo="$ctrl.addTodo($event);"></todo-form>
  <todo-list
    todos="$ctrl.todos"></todo-list>
</div>

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .name;
```

**[Haut de page](#table-des-matieres)**

# Directives

### Théorie des directives

Les directives nous donnent un `template`, `scope` bindings, `bindToController`, `link` et beaucoup d'autres choses. L'usage de ces dernières doit etre examiné avec attention maintenant que `.component()` existe. Les directives ne doivent plus déclarer de templates ni de controllers, ou recevoir des données via des bindings. Les directives ne doivent être utilisées que pour "décorer" le DOM. Cela signifie enrichir le HTML existant - créé avec `component()`. Dans son aspect le plus simple, si on a besoin d'events/APIs DOM sur mesure et de code logique, utiliser une Directive et la binder à un template d'un component. Si on a besoin d'une quantité raisonnable de manipulations de DOM, il y a aussi le lifecycle hook `$postLink` à considérer, néanmoins ce n'est pas un endroit pour faire migrer toutes vos manipulation de DOM, n'utilisez les Directives que pour les choses hors Angular si vous le pouvez.

Voici quelques conseils pour l'utilisation des Directives:

* Ne jamais utiliser template, scope, bindToController ou controller.
* Toujours `restrict: 'A'` avec les Directives.
* Utiliser compile et link quand nécessaire.
* Ne pas oublier de destroy et unbind les event handlers dans `$scope.$on('$destroy', fn);`

**[Haut de page](#table-des-matieres)**

### Propriétés recommandées

Comme les directives supportent la plupart de ce que fait `.component()` (les directives avec template étaient les component originaux), je recommande de limiter vos définitions d'Object directive à ces propriétés suivantes, pour éviter d'utiliser les directives à mauvais escient:

| Propriété | L'utiliser ? | Pourquoi |
|---|---|---|
| bindToController | Non | Utiliser `bindings` dans les components |
| compile | Oui | Pour pre-compiler les manipulations/events de DOM |
| controller | Non | Utiliser un component |
| controllerAs | Non | Utiliser un component |
| link functions | Oui | Pour pre/post manipulations/events du DOM |
| multiElement | Oui | [Voir documentation](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Oui | [Voir documentation](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | Non | Utiliser un component |
| restrict | Oui | Définit l'usage de la directive, toujours utiliser `'A'` |
| scope | Non | Utiliser un component |
| template | Non | Utiliser un component |
| templateNamespace | Oui (si vous etes obligé) | [Voir documentation](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | Non | Utiliser un component |
| transclude | Non | Utiliser un component |

**[Haut de page](#table-des-matieres)**

### Constantes ou Classes

Il y a quelques façons d'aborder l'utilisation d'ES2015 et des directives, soit avec les "fat arrow functions" et la facilitation d'assignation, soit utiliser une `Class` ES2015. Choisissez ce qui convient le mieux à vous ou votre équipe, garder en tête qu'Angular utilise `Class`.

Voici un exemple qui utilise une constante avec une  "fat arrow function",  un wrapper d'expression `() => ({})` qui retourne un Object literal (notez les différences d'utilisation dans `.directive()`):

```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

export const TodoAutoFocus = ($timeout) => {
  'ngInject';
  return {
    restrict: 'A',
    link($scope, $element, $attrs) {
      $scope.$watch($attrs.todoAutofocus, (newValue, oldValue) => {
        if (!newValue) {
          return;
        }
        $timeout(() => $element[0].focus());
      });
    }
  }
};

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoAutofocus } from './todo-autofocus.directive';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', TodoAutoFocus)
  .name;
```

Ou en utilisant les `Class` d'ES2015 (notez qu'appeler manuellement `new TodoAutoFocus` lorsque l'on enregistre une directive) pour créér l'Object:

```js
/* ----- todo/todo-autofocus.directive.js ----- */
import angular from 'angular';

export class TodoAutoFocus {
  constructor($timeout) {
    'ngInject';
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

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoAutofocus } from './todo-autofocus.directive';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .directive('todoAutofocus', () => new TodoAutoFocus)
  .name;
```

**[Haut de page](#table-des-matieres)**

# Services

### Théorie des services

Les services sont essentiellement des contenants pour la logique métier que nos components ne devraient pas demander directement. Les services contiennent d'autres services intégrés ou externes comme `$http`, que nous pouvons injecter dans les component controllers autre part dans notre application. Il y a deux manières pour faire des services, en utilisant `.service()` ou `.factory()`. Avec `Class` de ES2015, nous ne devrions utiliser que `.service()`, avec l'annotation de l'injection des dépendances en utilisant `$inject`.

**[Haut de page](#table-des-matieres)**

### Classes pour les services

Voici un exemple d'implémentation pour notre app `<todo>` utilisant les `Class` ES2015:

```js
/* ----- todo/todo.service.js ----- */
export class TodoService {
  constructor($http) {
    'ngInject;'
    this.$http = $http;
  }
  getTodos() {
    return this.$http.get('/api/todos').then(response => response.data);
  }
}

/* ----- todo/todo.module.js ----- */
import angular from 'angular';
import { TodoComponent } from './todo.component';
import { TodoService } from './todo.service';
import './todo.scss';

export const TodoModule = angular
  .module('todo', [])
  .component('todo', TodoComponent)
  .service('TodoService', TodoService)
  .name;
```

**[Haut de page](#table-des-matieres)**

# Styles

En utilisant [Webpack](https://webpack.github.io/) nous pouvons utiliser la notion d'`import` de nos fichiers `.scss` dans nos fichiers `*.module.js` et laisser Webpack connaitre la manière d'inclure ces fichiers dans nos feuilles CSS. En procédant de cette manière, nous gardons nos composants isolés tant au niveau fonctionnalités que styles, et cela se rapproche plus clairement de l'inclusion de feuilles de styles d'Angular. En procédant de cette manière, nous n'allons pas juste isolé nos styles pour ce composant comme en Angular, les styles seront quand même utilisable de manière globale dans l'application, mais l'ensemble sera beaucoup plus maintenable et la structure de notre application plus compréhensive.

Si vous avez quelques variables ou des styles globaux comme des champs de formulaires, dans ce cas ces fichiers doivent être placés dans votre dossier `scss`. Par exemple `scss/_forms.scss`. Ces styles globaux peuvent ensuite être `@imported` dans la feuille de style de notre module root (`app.module.js`) comme on procède habituellement.

**[Haut de page](#table-des-matieres)**

# ES2015 et outils

##### ES2015

* Utiliser [Babel](https://babeljs.io/) pour compiler votre code ES2015+ et tous les polyfills
* Envisager d'utiliser [TypeScript](http://www.typescriptlang.org/) pour simplifier la transition vers Angular

##### Outils
* Utiliser `ui-router` [la dernière alpha](https://github.com/angular-ui/ui-router) (voir le Readme) si vous voulez supporter le component-routing
  * Sinon vous serez bloqués avec `template: '<component>'` et pas de `bindings`
* Envisager le pré-chargement de vos templates dans le `$templateCache` avec `angular-templates` ou `ngtemplate-loader`
  * [version Gulp](https://www.npmjs.com/package/gulp-angular-templatecache)
  * [version Grunt](https://www.npmjs.com/package/grunt-angular-templates)
  * [version Webpack](https://github.com/WearyMonkey/ngtemplate-loader)
* Envisager d'utiliser [Webpack](https://webpack.github.io/) pour la compilation du code ES2015 et des styles
* Utiliser [ngAnnotate](https://github.com/olov/ng-annotate) pour annoter automatiquement les propriétés `$inject`
* Comment utiliser [ngAnnotate avec ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/#ng-annotate)

**[Haut de page](#table-des-matieres)**

# State management

Envisager d'utiliser Redux avec AngularJS 1.5 pour la gestion des données.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[Haut de page](#table-des-matieres)**

# Ressources

* [Composants stateful et stateless, le manuel](https://toddmotto.com/stateful-stateless-components)
* [Comprendre la méthode .component()](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Utilisation de "require" avec $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Comprendre tous les lifecycle hooks, $onInit, $onChanges, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Utiliser "resolve" dans vos routes](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Gestion du state avec Redux et Angular](http://blog.rangle.io/managing-state-redux-angular/)
* [Exemple d'application](https://github.com/chihab/angular-styleguide-sample)

**[Haut de page](#table-des-matieres)**

# Documentation
Pour d'autres informations, y compris les références à l'API, voici la [documentation AngularJS](//docs.angularjs.org/api)

# Contribuer

D'abord, ouvrir un issue pour discuter des changements/additions potentiels. N'ouvrez pas d'issues pour des questions.

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
