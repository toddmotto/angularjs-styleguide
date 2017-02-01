# Panduan Gaya AngularJS (ES2015)

### Arsitektur, struktur berkas, komponen, aliran data satu-arah dan contoh-contoh praktis

*Panduan gaya yang mudah dipahami untuk tim oleh [@toddmotto](//twitter.com/toddmotto)*

Arsitektur dan panduan gaya ini telah ditulis kembali secara lengkap untuk ES2015, perubahan-perubahan yang ada di AngularJS 1.5+ untuk Anda yang ingin melakukan upgrade aplikasi ke Angular nantinya. Panduan ini menyertakan contoh-contoh praktis aliran data satu-arah, delegasi event, arsitektur komponen dan ruting komponen.

Anda dapat temukan panduan gaya yang lama [disini](https://github.com/toddmotto/angular-styleguide/tree/angular-old-es5), dan latar belakang yang baru [disini](https://toddmotto.com/rewriting-angular-styleguide-angular-2).

> Bergabung bersama pengalaman belajar Ultimate AngularJS untuk menguasai sepenuhnya fitur-fitur pemula dan lanjutan Angular untuk membangun aplikasi nyata yang cepat, dan berskala.

<a href="https://courses.toddmotto.com" target="_blank"><img src="https://toddmotto.com/img/ua.png"></a>

## Daftar Isi

  1. [Arsitektur Modular](#arsitektur-modular)
    1. [Teori](#teori-modul)
    1. [Modul Pangkal](#modul-pangkal)
    1. [Modul Komponen](#modul-komponen)
    1. [Modul Umum](#modul-umum)
    1. [Modul Tingkat-Rendah](#modul-tingkat-rendah)
    1. [Struktur Berkas Berskala](#struktur-berkas-berskala)
    1. [Konvensi Penamaan Berkas](#konvensi-penamaan-berkas)
  1. [Komponen](#komponen)
    1. [Teori](#teori-komponen)
    1. [Properti yang Didukung](#properti-yang-didukung)
    1. [Pengendali](#pengendali)
    1. [Aliran Data Satu-arah dan Event](#aliran-data-satu-arah-dan-event)
    1. [Komponen Berkondisi](#komponen-berkondisi)
    1. [Komponen Tanpa Kondisi](#komponen-tanpa-kondisi)
    1. [Komponen yang memiliki Rute](#komponen-yang-memiliki-rute)
  1. [Direktif](#direktif)
    1. [Teori](#teori-direktif)
    1. [Properti yang Disarankan](#properti-yang-disarankan)
    1. [Konstanta atau Kelas](#konstanta-atau-kelas)
  1. [Servis](#servis)
    1. [Teori](#teori-servis)
    1. [Kelas untuk Servis](#kelas-untuk-servis)
  1. [ES2015 dan Perangkat](#es2015-dan-perangkat)
  1. [Pengelolaan kondisi](#pengelolaan-kondisi)
  1. [Sumber](#sumber)
  1. [Dokumentasi](#dokumentasi)
  1. [Berkontribusi](#berkontribusi)

# Arsitektur modular

Setiap modul yang ada di dalam aplikasi Angular adalah sebuah komponen modul. Sebuah komponen modul adalah definisi pangkal untuk modul itu sendiri yang akan merangkum logika, templat, rute, dan komponen anakan.

### Teori modul

Rancangan modul dipetakan secara langsung ke dalam struktur folder kita, untuk menjaga agar semuanya dapat tetap terpelihara dan dapat diprediksi. Idealnya kita harus memiliki tiga modul tingkat-tertinggi: pangkal, komponen, dan umum. Modul pangkal menentukan modul dasar yang mem-bootstrap aplikasi kita, serta templat yang berhubungan dengannya. Kemudian kita mengimpor modul-modul komponen dan umum ke dalam modul pangkal untuk menyertakan ketergantungan (dependensi) kita. Modul-modul komponen dan umum kemudian memerlukan modul-modul komponen tingkat-yang-lebih-rendah, yang mengandung komponen-komponen kita, seperti pengendali, servis, direktif, filter, dan pengujian, untuk masing-masing fitur yang dapat dipakai-ulang.

**[Kembali ke atas](#daftar-isi)**

### Modul pangkal

Sebuah modul Pangkal dimulai dengan komponen pangkal yang menentukan unsur dasar dari keseluruhan aplikasi, bersama sebuah rute keluaran yang sudah ditentukan, contohnya dengan menggunakan `ui-view` dari `ui-router`.

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

Sebuah modul Pangkal kemudian dibuat, dengan `AppComponent` yang diimpor dan didaftarkan dengan `.component('app', AppComponent)`. Pengimporan submodul berikutnya (modul-modul komponen dan umum) dibuat untuk menyertakan semua komponen yang berhubungan dengan aplikasinya.

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

**[Kembali ke atas](#daftar-isi)**

### Modul komponen

Sebuah modul Komponen adalah suatu referensi kontainer untuk semua komponen yang dapat dipakai-ulang. Lihat diatas bagaimana kita mengimpor `ComponentsModule` dan menginjeksinya ke dalam modul Pangkal, ini memberikan kita satu tempat tunggal untuk mengimpor semua komponen aplikasi. Modul-modul yang kita butuhkan ini dipisahkan dari semua modul lainnya sehingga kita dapat memindahkannya ke aplikasi lain dengan mudah.

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

**[Kembali ke atas](#daftar-isi)**

### Modul umum

Modul Umum adalah refensi kontainer untuk semua komponen-komponen aplikasi yang spesifik, yang tidak ingin kita gunakan di aplikasi yang lain. Seperti tata letak, navigasi, dan kaki halaman. Lihat diatas bagaimana kita mengimpor `CommonModule` dan menginjeksinya ke dalam modul Pangkal, ini memberikan kita satu tempat tunggal untuk mengimpor semua komponen umum untuk aplikasi tersebut.

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

**[Kembali ke atas](#daftar-isi)**

### Modul tingkat-rendah

Modul-modul Tingkat-Rendah adalah modul-modul komponen individual yang mengandung logika untuk setiap blok fitur. Masing-masingnya akan menetapkan sebuah modul, yang akan diimpor ke sebuah modul tingkat-yang-lebih-tinggi, seperti sebuah modul komponen atau umum, lihat contoh dibawah ini. Ingatlah untuk selalu menambahkan akhiran `.name` untuk setiap `export` ketika membuat sebuah modul _baru_, bukan pada saat mereferensikannya. Anda akan lihat penetapan ruting hadir disini, kita akan gali lebih dalam di bab berikut panduan ini.

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

**[Kembali ke atas](#daftar-isi)**

# Konvensi penamaan berkas

Buatlah sesederhana mungkin dan berhuruf kecil, gunakan nama komponen, misalnya `calendar.*.js*`, `calendar-grid.*.js` - dengan nama jenis berkasnya di tengah. Gunakan `*.module.js` untuk berkas penetapan modulnya, sehingga Anda dapat mengimpor modul tersebut berdasarkan nama direktori.

```
calendar.module.js
calendar.controller.js
calendar.component.js
calendar.service.js
calendar.directive.js
calendar.filter.js
calendar.spec.js
```

**[Kembali ke atas](#daftar-isi)**

### Struktur berkas berskala

Struktur berkas adalah sangat penting, ini menjelaskan sebuah sebuah struktur yang berskala dan dapat diprediksi. Sebuah contoh struktur berkas untuk menggambarkan sebuah arsitektur komponen yang modular.

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

Tingkat yang paling tinggi dari struktur folder hanya mengandung `index.html` dan `app/`, sebuah direktori dimana semua modul pangkal, komponen, umum dan tingkat-rendah kita berada.

**[Kembali ke atas](#daftar-isi)**

# Komponen

### Teori komponen

Komponen pada dasarnya adalah templat dengan sebuah pengendali. Mereka _bukan_ Direktif, dan Anda juga seharusnya tidak menggantikan Direktif dengan Komponen, kecuali bila Anda mengupgrade "Direktif templat" dengan pengendali, yang sangat cocok sebagai sebuah komponen. Komponen juga mengandung ikatan yang menentukan input dan output untuk data dan event, kaitan siklus, dan kemampuan untuk menggunakan aliran data satu-arah dan Obyek event untuk mencadangkan data ke sebuah komponen induk. Ini adalah standar defacto yang baru di AngularJS 1.5 dan versi berikutnya. Semua yang didorong oleh templat dan pengendali yang kita buat akan mirip seperti sebuah komponen, yang mungkin saja berupa komponen berkondisi, tanpa-kondisi, atau memiliki rute. Anda bisa bayangkan "komponen" itu seperti sebuah potongan kode yang lengkap, bukan hanya sekedar Obyek penetapan `.component()` saja. Mari kita telusuri beberapa contoh praktis dan saran-saran tentang komponen, untuk selanjutnya menyelam lebih dalam tentang bagaimana seharusnya Anda menyusunnya melalui konsep-konsep komponen berkondisi, tanpa-kondisi dan memiliki rute.

**[Kembali ke atas](#daftar-isi)**

### Properti yang didukung

Berikut adalah properti yang didukung untuk `.component()` yang dapat/harus Anda gunakan:

| Properti | Dukungan |
|---|---|
| bindings | Ya, gunakan `'@'`, `'<'`, `'&'` saja |
| controller | Ya |
| controllerAs | Ya, standarnya adalah `$ctrl` |
| require | Ya (sintak Obyek baru) |
| template | Ya |
| templateUrl | Ya |
| transclude | Ya |

**[Kembali ke atas](#daftar-isi)**

### Pengendali

Pengendali harus digunakan bersama komponen, jangan pernah di tempat lain. Bila rasanya Anda memerlukan sebuah pengendali, maka apa yang sebenarnya Anda perlukan untuk mengelola perilaku tersebut adalah sebuah komponen tanpa-kondisi.

Berikut adalah beberapa saran tentang pemakaian `Class` untuk pengendali:

* Selalu gunakan `constructor` untuk tujuan-tujuan injeksi dependensi
* Tidak mengekspor `Class` secara langsung, ekspor namanya untuk menerangkan `$inject`
* Bila Anda perlu mengakses lingkup bahasa, gunakan fungsi panah
* Alternatif lain fungsi panah, `let ctrl = this;` juga dapat diterima dan mungkin lebih masuk akal tergantung kasusnya
* Balut semua fungsi publik langsung ke dalam `Class`
* Gunakan kaitan siklus yang wajar, `$onInit`, `$onChanges`, `$postLink` and `$onDestroy`
  * Catatan: `$onChanges` dipanggil sebelum `$onInit`, lihat [sumber](#sumber) untuk artikel yang menjelaskan tentang hal ini lebih dalam
* Gunakan `require` bersama `$onInit` untuk mereferensikan logika warisan apapun
* Jangan mengubah alias standar `$ctrl` untuk sintak `controllerAs`, oleh karena itu jangan gunakan `controllerAs` di sembarang tempat

**[Kembali ke atas](#daftar-isi)**

### Aliran data satu-arah dan Event

Aliran data satu-arah diperkenalkan di AngularJS 1.5, dan mengubah komunikasi komponen.

Berikut adalah beberapa saran menggunakan aliran data satu-arah:

* Di dalam komponen yang menerima data, selalu gunakan sintak databinding satu-arah `'<'`
* _Jangan_ gunakan `'='` sintak databinding dua-arah lagi, dimanapun
* Komponen yang memiliki `bindings` harus menggunakan `$onChanges` untuk mereplika databinding satu-arah guna memenggal Obyek yang diteruskan oleh referensi dan memperbarui data induk
* Gunakan `$event` sebagai argumen fungsi dalam metoda induknya (lihat contoh berkondisi `$ctrl.addTodo($event)` yang ada dibawah)
* Lempar sebuah `$event: {}` cadangan Obyek dari sebuah komponen tanpa-kondisi (lihat contoh tanpa-kondisi `this.onAddTodo` dibawah).
  * Bonus: Gunakan sebuah pembungkus `EventEmitter` dengan `.value()` untuk mencerminkan Angular, hindari pembuatan Obyek `$event` manual
* Mengapa? Ini mencerminkan Angular dan menjaga konsistensi setiap komponen. Juga membuat semua kondisi dapat diprediksi.

**[Kembali ke atas](#daftar-isi)**

### Komponen berkondisi

Mari kita lihat apa yang kita sebut dengan "komponen berkondisi".

* Kondisi penarikan, pada dasarnya berkomunikasi ke API backend melalui sebuah servis
* Tidak memutasi kondisi secara langsung
* Membuat komponen anakan yang memutasi kondisi
* Mengacu pada sebuah komponen yang pintar/kontainer

Contoh dari komponen berkondisi, lengkap dengan penetapan modul tingkat-rendahnya (ini hanya untuk demo, jadi beberapa kode telah dihilangkan untuk menyederhanakannya):

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

Contoh ini menunjukkan sebuah komponen berkondisi, yang menarik suatu kondisi di pengendali, melalui sebuah servis, kemudian melemparnya ke komponen anakan tanpa-kondisi. Perhatikan bagaimana disana tidak ada Direktif yang digunakan seperti misalnya `ng-repeat` dan sebagainya di dalam templat. Melainkan, data dan fungsi yang didelegasikan ke dalam `<todo-form>` dan `<todo-list>` komponen-komponen tanpa-kondisi.

**[Kembali ke atas](#daftar-isi)**

### Komponen tanpa-kondisi

Mari kita lihat apa yang kita sebut dengan "komponen tanpa-kondisi".

* Memiliki input dan output yang ditetapkan menggunakan `bindings: {}`
* Data masuk ke komponen melalui atribut bindings (inputs)
* Data keluar dari komponen melalui event (outputs)
* Memutasikan kondisi, melemparkan cadangan data atas permintaan (seperti sebuah event klik atau kirim)
* Tidak peduli dari mana datangnya data, semua tanpa-kondisi
* Merupakan komponen-komponen yang sangat bisa dipakai-ulang
* Mengacu pada komponen dumb/bersifat presentasi

Contoh dari komponen tanpa-kondisi (mari kita gunakan `<todo-form>` sebagai contohnya), lengkap dengan penetapan modul tingkat-rendahnya (ini hanya untuk demo, jadi beberapa kode telah dihilangkan untuk menyederhanakannya):

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

Perhatikan bagaimana komponen `<todo-form>` tidak menarik kondisi apapun, ia hanya menerimanya saja, memutasi sebuah Obyek melalui logika pengendali yang berhubungan dengannya, kemudian melemparnya ke komponen induk melalui properti bindings. Dalam contoh ini, kaitan siklus `$onChanges` mereplika `this.todo` Obyek binding awal dan menetapkannya kembali, yang artinya bahwa data induk tidak terpengaruh sama sekali sampai kita mengirim formnya, bersama dengan sintak `'<'` binding yang baru dari aliran data satu-arah.

**[Kembali ke atas](#daftar-isi)**

### Komponen yang memiliki rute

Mari kita lihat apa yang kita sebut dengan "komponen yang memiliki rute".

* Pada dasarnya adalah sebuah komponen berkondisi, dengan penetapan ruting
* Tidak ada lagi berkas-berkas `router.js`
* Kita menggunakan komponen yang memiliki Rute untuk menetapkan logika ruting mereka
* Data "input" untuk komponen telah rampung melalui penyelesaian rute (opsional, masih tersedia di pengendali dengan pemanggilan servis)

Untuk contoh ini, kita akan mengambil komponen `<todo>` yang ada, membuatnya kembali agar menggunakan sebuah penetapan rute dan `bindings` pada komponen yang menerima data (rahasianya disini `ui-router` adalah properti `resolve` yang kita buat, dalam hal ini `todoData` dipetakan secara langsung ke keseluruhan `bindings`). Kita memperlakukannya sebagai komponen yang memiliki rute karena pada dasarnya itu adalah sebuah "view":

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

**[Kembali ke atas](#daftar-isi)**

# Direktif

### Teori direktif

Direktif memberikan kita `template`, bindings `scope`, `bindToController`, `link` dan banyak hal lainnya. Pemakaiannya sekarang harus benar-benar memperhatikan keberadaan `.component()`. Direktif tidak boleh mendeklarasikan templat dan pengendali, atau menerima data melalui bindings. Direktif harus digunakan semata-mata untuk mendekorasi DOM. Dengan begini, maka artinya adalah memperluas keberadaan HTML - yang dibuat dengan `.component()`. Pemikiran sederhananya, bila Anda memerlukan sebuah event DOM/API tertentu dan logis, gunakanlah sebuah Direktif dan ikatkannya ke sebuah templat yang ada di dalam sebuah komponen. Bila Anda memerlukan sejumlah manipulasi DOM yang pintar, ada kaitan siklus `$postLink` sebagai pertimbangannya, tapi ini bukanlah tempatnya untuk memigrasikan semua manipulasi DOM Anda, sebisa mungkin gunakanlah Direktif untuk hal-hal yang bukan-Angular.

Berikut adalah beberapa saran menggunakan Direktif:

* Jangan pernah gunakan template, scope, bindToController atau controller
* Selalu `restrict: 'A'` dengan Direktif
* Gunakan compile dan link bila diperlukan
* Jangan lupa hilangkan (destroy) dan unbind pengendali event di dalam `$scope.$on('$destroy', fn);`

**[Kembali ke atas](#daftar-isi)**

### Properti yang disarankan

Faktanya direktif mendukung kebanyakan apa yang dilakukan oleh `.component()` (direktif templat adalah komponen aslinya), saya menyarankan pembatasan penetapan Obyek direktif Anda hanya ke properti-properti berikut ini, untuk menghindari pemakaian direktif yang tidak benar:

| Properti | Gunakan? | Mengapa |
|---|---|---|
| bindToController | Tidak | Gunakan `bindings` di dalam komponen |
| compile | Ya | Untuk manipulasi/event pra-kompilasi DOM |
| controller | Tidak | Gunakan sebuah komponen |
| controllerAs | Tidak | Gunakan sebuah komponen |
| link functions | Ya | Untuk manipulasi/event pra/post DOM |
| multiElement | Ya | [Lihat dok](https://docs.angularjs.org/api/ng/service/$compile#-multielement-) |
| priority | Ya | [Lihat dok](https://docs.angularjs.org/api/ng/service/$compile#-priority-) |
| require | Tidak | Gunakan sebuah komponen |
| restrict | Ya | Menetapkan pemakaian direktif, selalu gunakan `'A'` |
| scope | Tidak | Gunakan sebuah komponen |
| template | Tidak | Gunakan sebuah komponen |
| templateNamespace | Ya (kalau Anda harus) | [Lihat dok](https://docs.angularjs.org/api/ng/service/$compile#-templatenamespace-) |
| templateUrl | Tidak | Gunakan sebuah komponen |
| transclude | Tidak | Gunakan sebuah komponen |

**[Kembali ke atas](#daftar-isi)**

### Konstanta atau Kelas

Ada beberapa pendekatan menggunakan ES2015 dan direktif, baik itu dengan sebuah fungsi panah maupun penetapan yang lebih mudah, atau dengan menggunakan sebuah `Class` ES2015. Pilih mana yang terbaik untuk Anda dan tim, ingatlah Angular menggunakan `Class`.

Berikut adalah contoh bagaimana menggunakan konstanta dengan fungsi Panah dimana pembungkus eskpresi `() => ({})` mengembalikan sebuah Obyek literal (catat perbedaan pemakaian di dalam `.directive()`):

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

Atau menggunakan `Class` ES2015 (catat pemanggilan `new TodoAutoFocus` secara manual pada saat mendaftarkan direktif) untuk membuat Obyek:

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

**[Kembali ke atas](#daftar-isi)**

# Servis

### Teori servis

Servis pada dasarnya adalah kontainer-kontainer untuk logika bisnis dimana komponen-komponen kita seharusnya tidak dipanggil secara langsung. Servis mengandung servis-servis bawaan lainnya atau eksternal seperti `$http`, yang dapat kita injeksi ke dalam pengendali komponen manapun di aplikasi kita. Kita memiliki dua cara memperlakukan servis, dengan menggunakan `.service()` atau `.factory()`. Dengan `Class` ES2015, kita seharusnya hanya menggunakan `.service()` saja, lengkap dengan keterangan injeksi dependensinya menggunakan `$inject`.

**[Kembali ke atas](#daftar-isi)**

### Kelas untuk Servis

Berikut adalah beberapa contoh implementasi untuk aplikasi `<todo>` kita menggunakan `Class` ES2015:

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

**[Kembali ke atas](#daftar-isi)**

# ES2015 dan Perangkat

##### ES2015

* Gunakan [Babel](https://babeljs.io/) untuk mengkompilasi kode ES2015+ Anda dan polyfills apapun
* Pertimbangkan untuk menggunakan [TypeScript](http://www.typescriptlang.org/) untuk memungkinkan upgrade ke Angular

##### Perangkat
* Gunakan `ui-router` [versi alfa terakhir](https://github.com/angular-ui/ui-router) (lihat Readme) bila Anda ingin mendukung ruting-komponen
  * Bila tidak maka Anda akan terjebak dengan `template: '<component>'` dan tidak ada pemetaan penyelesaian/`bindings`
* Pertimbangkan pra-muat templat ke dalam `$templateCache` dengan `angular-templates`
  * [Versi Gulp](https://www.npmjs.com/package/gulp-angular-templatecache)
  * [Versi Grunt](https://www.npmjs.com/package/grunt-angular-templates)
* Pertimbangkan untuk menggunakan [Webpack](https://webpack.github.io/) untuk mengkompilasi kode ES2015 Anda
* Gunakan [ngAnnotate](https://github.com/olov/ng-annotate) untuk menerangkan properti `$inject` secara otomatis
* Bagaimana menggunakan [ngAnnotate dengan ES6](https://www.timroes.de/2015/07/29/using-ecmascript-6-es6-with-angularjs-1-x/#ng-annotate)

**[Kembali ke atas](#daftar-isi)**

# Pengelolaan kondisi

Untuk pengelolaan data, pertimbangkan Redux bersama AngularJS 1.5.

* [Angular Redux](https://github.com/angular-redux/ng-redux)

**[Kembali ke atas](#daftar-isi)**

# Sumber

* [Memahami metoda .component()](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [Menggunakan "require" dengan $onInit](https://toddmotto.com/on-init-require-object-syntax-angular-component/)
* [Memahami semua kaitan siklus, $onInit, $onChange, $postLink, $onDestroy](https://toddmotto.com/angular-1-5-lifecycle-hooks)
* [Menggunakan "resolve" di dalam rute](https://toddmotto.com/resolve-promises-in-angular-routes/)
* [Redux dan pengelolaan kondisi Angular](http://blog.rangle.io/managing-state-redux-angular/)
* [Contoh Aplikasi dari Komunitas](https://github.com/chihab/angular-styleguide-sample)

**[Kembali ke atas](#daftar-isi)**

# Dokumentasi
Untuk lainnya, termasuk referensi API, periksalah [dokumentasi AngularJS](//docs.angularjs.org/api).

# Berkontribusi

Bukalah sebuah isu terlebih dahulu untuk mendiskusikan perubahan/penambahan yang potensial. Mohon tidak membuka isu untuk pertanyaan-pertanyaan.

## Lisensi

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
