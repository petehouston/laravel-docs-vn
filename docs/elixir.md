# Laravel Elixir

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Chạy Elixir](#running-elixir)
- [Làm việc Stylesheets](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Plain CSS](#plain-css)
    - [Source Maps](#css-source-maps)
- [Làm việc với Scripts](#working-with-scripts)
    - [CoffeeScript](#coffeescript)
    - [Browserify](#browserify)
    - [Babel](#babel)
    - [Scripts](#javascript)
- [Copy Files & Directories](#copying-files-and-directories)
- [Versioning / Cache Busting](#versioning-and-cache-busting)
- [BrowserSync](#browser-sync)
- [Gọi các Gulp tasks có sẵn](#calling-existing-gulp-tasks)
- [Viết extension cho Elixir](#writing-elixir-extensions)

<a name="introduction"></a>
## Giới thiệu

Laravel Elixir cung cấp một API gọn gàng và liền mạch cho việc tạo các [Gulp](http://gulpjs.com) task cho ứng dụng Laravel. Elixir cung cấp một số pre-processor phổ biến cho CSS và Javascript, cùng một số công cụ testing. Sử dụng móc nối hàm, Elixir cho phép bạn tạo các asset pipeline một cách liền mạch. Ví dụ:

```javascript
elixir(function(mix) {
    mix.sass('app.scss')
       .coffee('app.coffee');
});
```

Nếu bạn từng cảm thấy khó khăn trong việc bắt đầu sử dụng Gulp và đóng gói asset, bạn sẽ yêu thích Laravel Elixir. Tuy nhiên, bạn không cần thiết phải sử dụng nó khi phát triển ứng dụng. Bạn hoàn toàn thoải mái chọn lựa công cụ đóng gói asset nào bạn muốn, hoặc thậm chi không dùng gì cả.

<a name="installation"></a>
## Cài đặt

### Cài Node

Trước khi dùng Elixir, bạn phải đảm báo có NodeJS được cài trên máy của bạn.

    node -v

Mặc định, Laravel Homestead đã có sẵn tất cả mọi thứ; tuy nhiên, nếu bạn không sử dụng Vagrant, thì bạn có thể dễ dàng cài Node bằng cách [download NodeJS tại trang chủ](http://nodejs.org/download/).

### Gulp

Sau đó, bạn sẽ cần cài đặt [Gulp](http://gulpjs.com) vào trong nhóm thư viện global:

    npm install --global gulp

Nếu bạn sử dụng một hệ thống quản lý version, bạn có thể muốn chạy `npm shrinkwrap` để khoá các yêu cầu về NPM:

     npm shrinkwrap

Khi đã chạy lệnh này, bạn có thể thoải mái commit file [npm-shrinkwrap.json](https://docs.npmjs.com/cli/shrinkwrap) lên source control.

### Laravel Elixir

Việc còn lại là chỉ cần cài đặt Elixir! Đi cùng với bản cài mới đầy đủ của Laravel, bạn sẽ thấy một file `package.json` tại thư mục gốc. Hãy nghĩ đó tương tự file `composer.json`, chỉ khác là nó khai báo các dependency cho Node. Bạn có thể tiến hành cài đặt các dependency bằng cách chạy lệnh sau:

    npm install

Nếu bạn đang phát triển trên hệ điều hành Windows hay bạn đang trong VM trên host là Windows, bạn có thể cần chạy lệnh `npm install` với thông số `--no-bin-links`:

    npm install --no-bin-links

<a name="running-elixir"></a>
## Chạy Elixir

Elixir được xây dựng dựa trên [Gulp](http://gulpjs.com), vì thế, để chạy các Elixir task, bạn chỉ cần chạy lệnh `gulp` trên terminal. Thêm vào cờ `--production` vào câu lệnh để yêu cầu Elixir thực hiện minify CSS và Javascript:

    // Run all tasks...
    gulp

    // Run all tasks and minify all CSS and JavaScript...
    gulp --production

#### Theo dõi sự thay đổi của asset

Sẽ khá là bất tiện khi phải chạy lệnh `gulp` mỗi khi có sự thay đổi về assets, bạn có thể dùng `gulp watch`. Câu lệnh này sẽ tiếp tục chạy trên terminal và theo dõi sự thay đổi của asset, nếu phát hiện có sự thay đổi, hay file mới, thì asset sẽ lập tức được đóng gói lại:

    gulp watch

<a name="working-with-stylesheets"></a>
## Làm việc với Stylesheets

File `gulpfile.js` trong thư mục gốc của project chứa tất cả các Elixir task. Elixir task có thể được móc nối liên tiếp để cho biết chính xác cách mà asset sẽ được đóng gói.

<a name="less"></a>
### Less

Để đóng gói [Less](http://lesscss.org/) thành CSS, bạn có thể sử dụng hàm `less`. Hàm này lấy các file Less trong thư mục `resources/assets/less`. Mặc định, task sẽ đặt các file CSS đã đóng gói vào trong `public/css/app.css`:

```javascript
elixir(function(mix) {
    mix.less('app.less');
});
```

Bạn cũng có thể gộp các file Less thành một file CSS:

```javascript
elixir(function(mix) {
    mix.less([
        'app.less',
        'controllers.less'
    ]);
});
```

Bạn có thể thay đổi vị trí file CSS cuối cùng, bằng cách truyền vào tham số thứ hai trong hàm `less`:

```javascript
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets');
});

// Specifying a specific output filename...
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets/style.css');
});
```

<a name="sass"></a>
### Sass

Hàm `sass` cho phép bạn đóng gói [Sass](http://sass-lang.com/) thành file CSS. Với các file Sass lưu trong `resources/assets/sass`, bạn có thể sử dụng như sau:

```javascript
elixir(function(mix) {
    mix.sass('app.scss');
});
```

Tương tự với hàm `less`, bạn có thể đóng gói nhiều file Sass thành một file CSS duy nhất, và chọn vị trí lưu file:

```javascript
elixir(function(mix) {
    mix.sass([
        'app.scss',
        'controllers.scss'
    ], 'public/assets/css');
});
```

<a name="plain-css"></a>
### Plain CSS

Nếu bạn muốn gộp các file CSS thuần vào trong một file, bạn có thể sử dụng hàm `styles`. Các đường dẫn truyền vào trong hàm này là tương đối với thư mục `resources/assets/css` và file cuối cùng sẽ được lưu tại `public/css/all.css`:

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ]);
});
```

Dĩ nhiên, bạn có thể thay đổi vị trí file kết quả vào một vị trí khác bằng cách truyền vào tham số thứ hai của hàm `styles`:

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ], 'public/assets/css');
});
```

<a name="css-source-maps"></a>
### Source Maps

Source máp được kích hoạt sẵn. Vì thế, các file mà bạn đóng gói sẽ có một file `*.css.map` đi kèm trong cùng thư mục. Việc mapping này cho phép bạn theo dõi các CSS selector trên các file Sass, Less gốc khi thực hiện debug trên trình duyệt.

Nếu bạn không muốn sinh ra source maps, bạn có thể bỏ chúng đi trong cấu hình của Elixir:

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass('app.scss');
});
```

<a name="working-with-scripts"></a>
## Làm việc với Scripts

Elixir cũng cung cấp một vài hàm để giúp bạn làm việc với các file Javascript như đóng gói các file ECMAScript 6, CoffeeScript, Browserify, thu gọn, hay nối các file Javascript lại vào nhau.

<a name="coffeescript"></a>
### CoffeeScript

Hàm `coffee` được dùng để đóng gói các file [CoffeeScript](http://coffeescript.org/) thành file Javascript thuần. Hàm này nhận một chuỗi hay một mảng các file CoffeeScript tương đối trong thư mục `resources/assets/coffee` và tạo ra file `app.js` trong thư mục `public/js`:

```javascript
elixir(function(mix) {
    mix.coffee(['app.coffee', 'controllers.coffee']);
});
```

<a name="browserify"></a>
### Browserify

Elixir cũng cung cấp hàm `browserify`, cho phép bạn các lợi ích của việc require các module lên trình duyệt và sử dụng ECMAScript 6 và JSX.

Hàm này sẽ lấy các file script nằm trong thư mục `resources/assets/js` và sẽ tạo ra file `public/js/main.js`. Bạn có thể thay đổi vị trí output trong tham số thứ hai:

```javascript
elixir(function(mix) {
    mix.browserify('main.js');
});

// Specifying a specific output filename...
elixir(function(mix) {
    mix.browserify('main.js', 'public/javascripts/main.js');
});
```

Mặc dù Browserify có sẵn Partialify và Babelify, bạn hoàn toàn có thể chọn cài thêm mà bạn muốn:

    npm install aliasify --save-dev

```javascript
elixir.config.js.browserify.transformers.push({
    name: 'aliasify',
    options: {}
});

elixir(function(mix) {
    mix.browserify('main.js');
});
```

<a name="babel"></a>
### Babel

Hàm `babel` được sử dụng để đóng gói [ECMAScript 6 and 7](https://babeljs.io/docs/learn-es2015/) và [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) thành Javascript thuần. Hàm này nhận một mảng các file trong thư mục `resources/assets/js`, và tạo ra một file `all.js` duy nhất trong thư mục `public/js`:

```javascript
elixir(function(mix) {
    mix.babel([
        'order.js',
        'product.js',
        'react-component.jsx'
    ]);
});
```

Để chọn một vị trí output khác, bạn chỉ cần thêm vào tham số thứ hai. Đặc điểm và chức năng của hàm này hoàn toàn giống với `mix.scripts()`, chỉ trừ việc đóng gói Babel.


<a name="javascript"></a>
### Scripts

Nếu bạn có nhiều file Javascript mà bạn muốn đóng gói thành một file, bạn có thể sử dụng hàm `scripts`.

Hàm `scripts` sẽ lấy tất cả các file trong thư mục `resources/assets/js` và tạo ra file Javascript cuối dùng tại thư mục `public/js/all.js`:

```javascript
elixir(function(mix) {
    mix.scripts([
        'jquery.js',
        'app.js'
    ]);
});
```

Nếu bạn muốn tạo các tập hợp file scripts khác nhau, bạn có thể gọi nhiều hàm `scripts`.

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

Nếu bạn muốn gộp tất cả các scripts trong một thư mục, bạn có thể sử dụng hàm `scriptsIn`. File kết quả cuối cùng sẽ được đặt tại `public/js.all.js`:

```javascript
elixir(function(mix) {
    mix.scriptsIn('public/js/some/directory');
});
```

<a name="copying-files-and-directories"></a>
## Copy Files & Directories

Hàm `copy` được dùng để copy files và thư mục tới các vị trí mới. Tất cả đều ở vị trí relative tới thư mục gốc của project:

```javascript
elixir(function(mix) {
	mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});

elixir(function(mix) {
	mix.copy('vendor/package/views', 'resources/views');
});
```

<a name="versioning-and-cache-busting"></a>
## Versioning / Cache Busting

Nhiều lập trình viên muốn đóng gói assets với một timestamp hay token đặc biệt để ép trình duyệt phải load lại asset thay vì sử dụng file cache. Elixir có thể xử lý cho bạn bằng hàm `version`.

Hàm `version` nhận tên file relatie với thư mục `public`, và sẽ thêm vào một hash unique trong tên file. Ví dụ, tên file sinh ra sẽ có thể như thế này, `all-16d570a6.css`:

```javascript
elixir(function(mix) {
    mix.version('css/all.css');
});
```

Sau khi tạo ra file được version, bạn có thể sử dụng hàm helper `elixir` bên trong [views](/docs/{{version}}/views) để lấy ra asset đã được hash. Hàm `elixir` sẽ tự động xác định tên file:

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### Đánh version cho nhiều file

Bạn có thể truyền vào một mảng trong hàm `version` để đánh version cho nhiều file:

```javascript
elixir(function(mix) {
    mix.version(['css/all.css', 'js/app.js']);
});
```

Sau đó, bạn có thể sử dụng hàm helper `elixir` để lấy link tới các file đã được hash. Chú ý là bạn chỉ cần truyền vào tên file đầy đủ mà không có hash vào hàm `elixir`.

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

    <script src="{{ elixir('js/app.js') }}"></script>

<a name="browser-sync"></a>
## BrowserSync

BrowserSync tự động refresh lại trình duyệt sau khi có cập nhật ở frontend. Bạn có thể sử dụng hàm `browserSync` để bảo Elixir bắt đầu một BrowserSync server khi chạy lệnh `gulp watch`:

```javascript
elixir(function(mix) {
    mix.browserSync();
});
```

Sau khi chạy `gulp watch`, truy cập vào trang web sử dụng port 3000 để kích hoạt đồng bộ browser: `http://homestead.app:3000`. Nếu bạn đang sử dụng domain khác với `homestead.app` cho môi trường phát triển, bạn có thể truyền vào mảng [options](http://www.browsersync.io/docs/options/) ở tham số đầu tiên cho hàm `browserSync`:

```javascript
elixir(function(mix) {
    mix.browserSync({
    	proxy: 'project.app'
    });
});
```

<a name="calling-existing-gulp-tasks"></a>
## Gọi các Gulp task có sẵn

Nếu bạn cần gọi tới các Gulp task từ Elixir, bạn có thể sử dụng hàm `task`. Ví dụ, tưởng tượng bạn có một Gulp task đơn giản chỉ hiển thị text khi được gọi:

```javascript
gulp.task('speak', function() {
    var message = 'Tea...Earl Grey...Hot';

    gulp.src('').pipe(shell('say ' + message));
});
```

Nếu bạn muốn gọi task này từ Elixir, sử dụng hàm `mix.task` và truyền vào tên của task:

```javascript
elixir(function(mix) {
    mix.task('speak');
});
```

#### Custom Watchers

Nếu bạn muốn đăng kí một watcher để chạy task riêng của bạn mỗi khi có file bị thay đổi, truyền vào một regular expression ở tham số thứ hai trong hàm `task`:

```javascript
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');
});
```

<a name="writing-elixir-extensions"></a>
## Viết extensions riêng cho Elixir

Nếu bạn muốn xử lý linh hoạt hơn những gì mà hàm `task` của Elixir cung cấp, bạn có thể viết Elixir extension. Điều này cho phép bạn truyền vào một task riêng. Ví dụ, bạn có thể viết một extension như sau:

```javascript
// File: elixir-extensions.js

var gulp = require('gulp');
var shell = require('gulp-shell');
var Elixir = require('laravel-elixir');

var Task = Elixir.Task;

Elixir.extend('speak', function(message) {

    new Task('speak', function() {
        return gulp.src('').pipe(shell('say ' + message));
    });

});

// mix.speak('Hello World');
```

Chỉ có vậy! Chú ý là phần logic của Gulp cần được đặt bên trong hàm được truyền vào ở tham số thứ hai trong hàm khởi tạo của `Task`. Bạn có thể đặt chúng ở đầu Gulpfile, hoặc tách chúng ra thành một file riêng. Ví dụ, bạn có thể để extension trong `elixir-extensions.js`, bạn có thể lấy file này trong `Gulpfile` như sau:

```javascript
// File: Gulpfile.js

var elixir = require('laravel-elixir');

require('./elixir-extensions')

elixir(function(mix) {
    mix.speak('Tea, Earl Grey, Hot');
});
```

#### Custom Watchers

Nếu bạn muốn task của bạn được kích hoạt khi chạy `gulp watch`, bạn có thể đăng kí một watcher:

```javascript
new Task('speak', function() {
    return gulp.src('').pipe(shell('say ' + message));
})
.watch('./app/**');
```
