# Routing

- [Routing](#routing)
  - [Routing cơ bản](#routing-c-bn)
    - [Tệp tin routes mặc định](#tp-tin-routes-mc-nh)
    - [Các phương thức router có sẵn](#cac-phng-thc-router-co-sn)
    - [Bảo mật CSRF](#bo-mt-csrf)
    - [Routes chuyển hướng](#routes-chuyn-hng)
    - [Routes hiển thị](#routes-hin-th)
  - [Các tham số route](#cac-tham-s-route)
    - [Tham số bắt buộc](#tham-s-bt-buc)
    - [Tham số tùy chọn](#tham-s-toy-chn)
    - [Hạn chế với biểu thức chính quy](#hn-ch-vi-biu-thc-chinh-quy)
  - [Tên của router](#ten-ca-router)
    - [Tạo URL từ route đã đặt tên](#to-url-t-route-a-t-ten)
    - [Xác định route hiện tại](#xac-nh-route-hin-ti)
  - [Nhóm router](#nhom-router)
    - [Middleware](#middleware)
    - [namespaces](#namespaces)
    - [router tên miền phụ](#router-ten-min-ph)
    - [Các tiền tố router](#cac-tin-t-router)
    - [Route name prefixes](#route-name-prefixes)
  - [Route models binding](#route-models-binding)
    - [Ràng buộc ngầm định](#rang-buc-ngm-nh)
    - [Tùy biến tên khóa](#toy-bin-ten-khoa)
    - [Ràng buộc tường minh](#rang-buc-tng-minh)
  - [Rate Limiting](#rate-limiting)
    - [Dynamic Rate Limiting](#dynamic-rate-limiting)
  - [Form Method Spoofing](#form-method-spoofing)
  - [Truy cập router hiện tại](#truy-cp-router-hin-ti)

## Routing cơ bản

Hầu hết các route của laravel nhận một URI và một ``Closure``, nó cung cấp 1 cách rất đơn giản để định nghĩa một route:

```PHP
Route::get('foo', function () {
    return 'Hello World';
});
```

### Tệp tin routes mặc định

Tất cả các route được định nghĩa ở trong file `route`, ở trong thư mục `routes`. Nó đó sẽ được tự động tải bởi framework.
File ``routes/web.php`` định nghĩa route cho giao diện web của bạn. Đấy là routes được gán vào thuộc nhóm middleware ``web``, nó cung cấp một số tính năng như session và bảo mật CSRF.
File ``routes/api.php`` định nghĩa cho route không rõ ràng và được gán vào nhóm middleware ``api``.

Hầu hết các ứng dụng, bạn sẽ bắt đầu định nghĩa route trong file ``routes/web.php``. Các route đã được định nghĩa trong ``routes/web.php`` có thể truy cập từ URL của route cụ thể từ trình duyệt web.

Ví dụ: bạn có thể truy cập router sau bằng cách điều hướng đến trong trình duyệt của bạn
``phttp://your-app.test/user``

```PHP
Route::get('/user', 'UserController@index');
```

### Các phương thức router có sẵn

Router cho phép bạn đăng ký routes đáp ứng nhiều phương thức HTTP:

- Route::get nhận resquest với phương thức GET.
- Route::post nhận resquest với phương thức POST.
- Route::put nhận resquest với phương thức PUT.
- Route::delete nhận resquest với phương thức DELETE.
- Route::match kết hợp nhiều phương phức như POST,GET,PUT,..
- Route::any nhận tất cả các phương thức.
- Route::group tạo ra các nhóm route.
- Route::controller gọi đến controller tương ứng mà chúng ta tự định nghĩa.

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

Đôi khi bạn có thể phải đăng ký nhiều router để đáp ứng nhiều phương thức HTTP. Bạn có thể làm như vậy bằng cách sử dụng các phương thức `match`. Hoặc, bạn thậm chí có thể đăng ký một router mà đáp ứng tất cả các phương thức HTTP bằng cách sử dụng phương thức `any`:

```PHP
Route::match(['get', 'post'], '/', function () {
    //
});

Route::any('foo', function () {
    //
});

```

### Bảo mật CSRF

Tất cả các HTML form gọi đến các route ``POST``, ``PUT``, hoặc ``DELETE`` được định nghĩa trong file route ``web`` thì cần được thêm trường CSRF token. Nếu không thì request sẽ bị từ trối. Bạn có thể đọc thêm về bảo mật CSRF tại [Tài liệu CSRF](csrfprotection.md):

```php
<form method="POST" action="/profile">
  @csrf
    ...
</form>
```

### Routes chuyển hướng

Nếu bạn muốn định nghĩa route chuyển hướng tới một URL khác, bạn có thể sử dụng phương thức ``Route:redirect``. Phương thức này cung cấp đường tắt giúp bạn không phải định nghĩa lại 1 route đầy đủ hay 1 controller cho việc chuyển hướng

```PHP
Route::redirect('/here', '/there', 301);
```

### Routes hiển thị

Nếu route của bạn chỉ trả lại một view, bạn có thể sử dụng phương thức ``Route:view``. Phương thức này cung cấp đường tắt giúp bạn không phải định nghĩa lại 1 route đầy đủ hay 1 controller. Phương thức ``View`` chấp nhận URL như là đối số đầu tiên và tên của view là đối số thứ hai. Ngoài ra bạn cũng có thể truyền một mảng dữ liệu như một đối số thứ ba.

```PHP
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

## Các tham số route

### Tham số bắt buộc

Tất nhiên, đôi khi bạn sẽ cần phải lấy ra vài đoạn của URIs trong router của bạn. Ví dụ: bạn cần lấy được ID của người dùng từ các URL. Bạn có thể làm như vậy bằng cách định nghĩa tham số như sau:

```PHP
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

Bạn có thể định nghĩa nhiều tham số theo ý muốn:

```PHP
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

Các tham số router luôn được bao bọc bởi cặp dấu ngoặc nhon "{}" và chỉ nên dùng kí tự chữ cái để đặt tên biến .
Các tham số này sẽ đi vào Closure của router theo thứ tự của chúng, tên của tham số trong các hàm không liên quan

> **Chú ý:** Các tham số không được chứa ký tự `-`. Sử dụng dấu gạch dưới `_` để thay thế.

### Tham số tùy chọn

Thỉnh thoảng, bạn có thể phải chỉ định một số router, nhưng sự xuất hiện của nó không bắt buộc. Bạn có thể làm như vậy bằng cách đặt một dấu chấm hỏi "`?`" sau tên tham số. Hãy chắc chắn rằng cung cấp cho biến tương ứng một giá trị mặc định:

```PHP
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

> Như các hàm của ngôn ngữ khác, tham số tùy chọn phải nằm ở cuối cùng.

### Hạn chế với biểu thức chính quy

Bạn có thể hạn chế các định dạng của tham số router bằng cách sử dụng phương thức `where`. Phương thức `where` sự dụng biểu thức chính quy để xác nhân dữ liệu biến được chấp nhận:

```PHP
Route::get('user/{name}', function ($name) {
    //
})
->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})
->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})
->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

#### Hạn chế toàn cục

Nếu bạn muốn giới hạn với mọi route, bạn có thể sử dụng phương thức ``pattern``. Bạn nên định nghĩa phương thức này trong phương thức ``boot`` ở file ``RouteServiceProvider.php``:

```PHP
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @param  \Illuminate\Routing\Router  $router
 * @return void
*/
public function boot(Router $router)
{
    $router->pattern('id', '[0-9]+');

    parent::boot($router);
}
```

Khi mô hình đã được xác định nó sẽ tự động áp dụng cho tất cả các router sử dụng tham số:

```PHP
Route::get('user/{id}', function ($id) {
    // Only called if {id} is numeric.
});
```

## Tên của router

Các router cho phép đặt tên để thuận tiện cho các URL hoặc chuyển hướng cho các router cụ thể. Bạn có thể chỉ định một tên cho router bằng cách sử dụng phương thức `name` khi tạo router:

```PHP
Route::get('user/profile', function () {
    //
})->name('profile');
```

Ngoài ra bạn cũng có thế chỉ định tên route cho controller:

```PHP
Route::get('user/profile', 'UserController@showProfile')->name('profile');
```

### Tạo URL từ route đã đặt tên

Một khi bạn đã gán tên cho một route xác định, bạn có thể dùng tên của nó khi tạo URL hoặc chuyển hướng thông qua hàm toàn cục ``route``:

```PHP
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```

Nếu tên route được định nghĩa với tham số, bạn có thể xuyên qua nó như là một đối số thứ hai trong phương thức routeroute. Các đối số sẽ được chèn vào theo đúng thứ tự chính xác trên URLs:

```PHP
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

### Xác định route hiện tại

Nếu bạn muốn kiểm tra route hiện tại đã được đặt tên hay chưa, bạn có thể sử dụng phương thức ``name`` ngay trên Instance(thực thể) của route đó.
Ví dụ, bạn có thể kiểm tra tên route hiện tại ở route middleware:

```PHP
/**
 * Handle an incoming request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```

## Nhóm router

Nhóm router cho phép bạn chia sẻ các thuộc tính như middleware hay namespaces, trên nhiều router mà không cần phải xác định lại chúng trên mỗi router riêng. Các thuộc tính chung được quy định trong một mảng định dạng là tham số đầu tiên của phương thức `Route::group`.

### Middleware

Để gán middleware cho một nhóm, bạn sử dụng phương thức `middleware` trước khi định nghĩa nhóm. Middleware sẽ được thực hiện theo thứ tự bạn định nghĩa mảng này:

```PHP
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second Middleware
    });

    Route::get('user/profile', function () {
        // Uses first & second Middleware
    });
});
```

### namespaces

Một trường hợp sử dụng chung cho nhóm route giống như PHP namespace được chỉ định với một nhóm của controllers. Bạn có thể sử dụng tham số namespace trong mảng thuộc tính:

```PHP
Route::namespace('Admin')->group(function () {
    // Controllers Within The "App\Http\Controllers\Admin" Namespace
});
```

Hãy nhớ rằng, theo mặc định `RouteServiceProvider` bao gồm tệp tin `routes.php` của bạn trong một nhóm namespaces cho phép bạn đăng ký các bộ điều khiển router mà không xác định đầy đủ tiền tố namespace `App\Http\Controllers`. Vì vậy, chúng ta chỉ cần xác định thành phần của tên đó được đưa ra sau `App\Http\Controllers`.

### router tên miền phụ

Nhóm router cũng có thể được sử dụng để đại diện các tên miền phụ. Tên miền phụ có thể được gán tham số router như URIs, cho phép bạn lấy một phần của tên miền phụ để sử dụng bên trong router hoặc bộ điều khiển của bạn. Các tên miền phụ có thể được xác định bằng cách sử dụng khóa `domain` trong mảng thuộc tính:

```PHP
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

### Các tiền tố router

Thuộc tính `prefix` có thể sử dụng để thêm tiền tố cho mỗi router trong một nhóm với một URI. Ví dụ, bạn có thể muốn tất cả các tiền tố trong nhóm là admin:

```PHP
Route::prefix('admin')->group(function () {
    Route::get('users', function () {
        // Matches The "/admin/users" URL
    });
});
```

### Route name prefixes

Phương thức ``name`` có thể dùng để thêm tiền tố cho tên của route trong nhóm với chuỗi bạn muốn. Ví dụ dưới sẽ thêm tiền tố ``admin`` vào tất cả tên route ở trong group. Chuỗi cần gắn sẽ được đặt trước tên route, nên chắc chắn có kí tự ``.`` trong chuỗi tiền tố.

```PHP
Route::name('admin.')->group(function () {
    Route::get('users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```

## Route models binding

Khi bạn chèn một model ID vào route hoặc controller, bạn sẽ thường truy vấn để để nhận model tương ứng với ID đó. Laravel models binding cung cấp một cách thuận tiện để đẩy model vào router của bạn. Ví dụ, thay vì đẩy ID của người dùng bạn có thể đẩy toàn bộ model `User` phù hợp với ID xác định.

### Ràng buộc ngầm định

Khi bạn inject một model ID vào route hoặc controller, bạn sẽ thường truy vấn để để nhận model tương ứng với ID đó.Laravel sẽ tự động giải quyết gợi ý Eloquent Model được xác định bên trong router hoặc bộ điều khiển có tên biến phù hợp với tên một phân đoạn route. Ví dụ:

```PHP
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;
});
```

Trong ví dụ này, Eloquent đã gợi ý biến `$user` định nghĩa trong route phù hợp với `{user}` trong URI. Laravel tự động đẩy các mô hình hiện thực có một ID phù hợp với giá trị tương ứng từ URI.

Nếu không tìm thấy trong cơ sở dữ liệu, một phản hồi 404 HTTP sẽ tự động được tạo ra.

### Tùy biến tên khóa

Nếu bạn muốn các mô hình ràng buộc ngầm định để sử dụng một cột cơ sở dữ liệu khác ``ID`` khi lấy dữ liệu, bạn có thể ghi đè lên phương thức `getRouteKeyName` trên Eloquent Model:

```PHP
/**
 * Get the route key for the model.
 *
 * @return string
 */
public function getRouteKeyName()
{
    return 'slug';
}
```

### Ràng buộc tường minh

Để đăng ký ràng buộc tường minh, sử dụng phương thức model để xác định class cho một tham số. Bạn nên định nghĩa các ràng buộc của bạn bên trong phương thức ``boot`` của lớp ``RouteServiceProvider``:

```PHP
public function boot()
{
    parent::boot();

    Route::model('user', App\User::class);
}
```

Tiếp thep, định nghĩa một route chứa tham số ``{user}``:

```PHP
$router->get('profile/{user}', function(App\User $user) {
    //
});
```

Khi chúng ta đã ràng buộc tham số {user} trong model App\User, một thể hiện của User sẽ được inject vào route. Vì vậy, ví dụ, một request đến profile/1 sẽ inject User có ID là 1.

Nếu thể hiện model không tìm thấy trong cơ sở dữ liệu, một phản hồi 404 HTTP sẽ tự động được tạo ra.

#### Tùy chỉnh theo revolution logic

Nếu bạn muốn sử dụng revolution logic của riêng bạn, bạn nên sử dụng phương thức `Route::bind`. Thuộc tính ``Closure`` sẽ truyền qua phương thức ``bind`` sẽ nhận được giá trị tham biến trên segment URI và sẽ trả về một thể hiện của class bạn muốn inject vào route:

```PHP
$router->bind('user', function ($value) {
    return App\User::where('name', $value)->first();
});
```

## Rate Limiting

Laravel bao gồm [phần mềm trung gian](middleware.md) sẽ giới hạn truy cập vào phần mềm của bạn. Để bắt đầu, gắn ``throttle`` middleware vào route hay nhóm route. ``throttle`` chấp nhận nhận 2 tham số để xác định số lượng tối đa truy cập có thể thực hiện trong một số phút nhất định. Ví dụ dưới sẽ giới hạn nhóm người dùng có thể truy cập vào route 60 lần mỗi phút:

```PHP
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

### Dynamic Rate Limiting

Bạn có thể giới hạn số lượng truy cấp tối đa một cách linh động dựa trên thuộc tính trong ``user`` model. Ví dụ dưới, nếu ``user`` model chứa thuộc tính ``rate_limit``, bạn có thể đặt nó trong phần thuộc tính của ``thorttle`` để sử dụng nó cho việc xác định số lượng truy cập tối đa có thể.

```PHP
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

## Form Method Spoofing

Biểu mẫu HTML không hỗ trợ phương thức `PUT`, `DELETE`, `PATCH`. Vì vậy, khi xác định các phương thức trên được gọi từ HTML Form, bạn sẽ phải thêm một trường input ẩn `_method` vào biểu mẫu. Các giá trị được gửi với trường `_method` sẽ được sử dụng như các phương thức HTTP:

```PHP
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

Bạn có thể sử dụng hàm method_field nó sẽ tự sinh ra một _method input:

```PHP
<form action="/foo/bar" method="POST">
    @method('PUT')
    @csrf
</form>
```

Để tạo ra trường ẩn `_method`, bạn cũng có thể sử dụng helper function:

```PHP
<?php echo method_field('PUT'); ?>
```

Hoặc sử dụng Blade template:

```PHP
{{ method_field('PUT') }}
```

## Truy cập router hiện tại

Bạn có thể sử dụng phương thức ``current``, ``currentRouteName``, và ``currentRouteAction`` trên ``Route`` facade để truy cập thông tin về resquest route xử lý đang đến:

```PHP
$route = Route::current();

$name = Route::currentRouteName();

$action = Route::currentRouteAction();
```

Vui lòng tham khảo các [lớp cơ bản của Route](http://laravel.com/api/5.6/Illuminate/Routing/Router.html) để xem tất cả các phương thức.
