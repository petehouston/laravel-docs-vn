# Routing

- [Routing](#routing)
  - [Routing cơ bản](#routing-c%C6%A1-b%E1%BA%A3n)
    - [Tệp tin routes mặc định](#t%E1%BB%87p-tin-routes-m%E1%BA%B7c-%C4%91%E1%BB%8Bnh)
    - [Các phương thức định tuyến có sẵn](#c%C3%A1c-ph%C6%B0%C6%A1ng-th%E1%BB%A9c-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn-c%C3%B3-s%E1%BA%B5n)
  - [Các tham số route](#c%C3%A1c-tham-s%E1%BB%91-route)
    - [Tham số bắt buộc](#tham-s%E1%BB%91-b%E1%BA%AFt-bu%E1%BB%99c)
    - [Tham số tùy chọn](#tham-s%E1%BB%91-t%C3%B9y-ch%E1%BB%8Dn)
    - [Hạn chế sử dụng biểu thức chính quy](#h%E1%BA%A1n-ch%E1%BA%BF-s%E1%BB%AD-d%E1%BB%A5ng-bi%E1%BB%83u-th%E1%BB%A9c-ch%C3%ADnh-quy)
    - [Hạn chế toàn cục](#h%E1%BA%A1n-ch%E1%BA%BF-to%C3%A0n-c%E1%BB%A5c)
  - [Tên của định tuyến](#t%C3%AAn-c%E1%BB%A7a-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn)
    - [Nhóm định tuyến và đặt tên các định tuyến](#nh%C3%B3m-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn-v%C3%A0-%C4%91%E1%BA%B7t-t%C3%AAn-c%C3%A1c-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn)
    - [Tạo URL từ định tuyến đã đặt tên](#t%E1%BA%A1o-url-t%E1%BB%AB-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn-%C4%91%C3%A3-%C4%91%E1%BA%B7t-t%C3%AAn)
  - [Nhóm định tuyến](#nh%C3%B3m-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn)
    - [Middleware](#middleware)
    - [namespaces](#namespaces)
    - [Định tuyến tên miền phụ](#%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn-t%C3%AAn-mi%E1%BB%81n-ph%E1%BB%A5)
    - [Các tiền tố định tuyến](#c%C3%A1c-ti%E1%BB%81n-t%E1%BB%91-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn)
  - [Bảo mật CSRF](#b%E1%BA%A3o-m%E1%BA%ADt-csrf)
    - [Giới thiệu](#gi%E1%BB%9Bi-thi%E1%BB%87u)
    - [Loại bỏ URIs khỏi bảo mật CSRF](#lo%E1%BA%A1i-b%E1%BB%8F-uris-kh%E1%BB%8Fi-b%E1%BA%A3o-m%E1%BA%ADt-csrf)
    - [X-CSRF-TOKEN](#x-csrf-token)
    - [X-XSRF-TOKEN](#x-xsrf-token)
  - [Route models binding](#route-models-binding)
    - [Ràng buộc ngầm định](#r%C3%A0ng-bu%E1%BB%99c-ng%E1%BA%A7m-%C4%91%E1%BB%8Bnh)
    - [Tùy biến tên khóa](#t%C3%B9y-bi%E1%BA%BFn-t%C3%AAn-kh%C3%B3a)
    - [Ràng buộc tường minh](#r%C3%A0ng-bu%E1%BB%99c-t%C6%B0%E1%BB%9Dng-minh)
      - [Ràng buộc một tham số cho một mô hình](#r%C3%A0ng-bu%E1%BB%99c-m%E1%BB%99t-tham-s%E1%BB%91-cho-m%E1%BB%99t-m%C3%B4-h%C3%ACnh)
      - [Tùy chỉnh the revolution logic](#t%C3%B9y-ch%E1%BB%89nh-the-revolution-logic)
      - [Tùy chỉnh "Not found"](#t%C3%B9y-ch%E1%BB%89nh-not-found)
  - [Form Method Spoofing](#form-method-spoofing)
  - [Truy cập vào định tuyến hiện tại](#truy-c%E1%BA%ADp-v%C3%A0o-%C4%91%E1%BB%8Bnh-tuy%E1%BA%BFn-hi%E1%BB%87n-t%E1%BA%A1i)

## Routing cơ bản

Tất cả các định tuyến Laravel được định nghĩa tại tập tin `app/Http/routes.php`, nó sẽ được Framework tự động tải. Định tuyến Laravel cơ bản nhất đơn giản chấp nhận một URI và một Closure (một hàm hay một tham chiếu đến một hàm cùng với môi trường tham chiếu), cung cấp một cách rất đơn giản và ý nghĩa cho việc định tuyến:

```PHP
Route::get('foo', function () {
    return 'Hello World';
});
```

### Tệp tin routes mặc định

Tệp tin mặc đinh `routes.php` được tải bởi `RouteServiceProvider` và được tự động bao gồm nhóm middleware `web`, cung cấp truy cập đến trạng thái phiên làm việc và vào mật CSRF. Hầu hết các định tuyến trong ứng dụng của bạn đều được định nghĩa trong tệp tin này.

### Các phương thức định tuyến có sẵn

```PHP
Route::get($uri, $callback);

Route::post($uri, $callback);

Route::put($uri, $callback);

Route::patch($uri, $callback);

Route::delete($uri, $callback);

Route::options($uri, $callback);
```

Đôi khi bạn có thể phải đăng ký nhiều định tuyến mà đáp ứng nhiều phương thức HTTP. Bạn có thể làm như vậy bằng cách sử dụng các phương thức `match`. Hoặc, bạn thậm chí có thể đăng ký một định tuyến mà đáp ứng tất cả các phương thức HTTP bằng cách sử dụng phương thức `any`:

```PHP
Route::match(['get', 'post'], '/', function () {
    //
});

Route::any('foo', function () {
    //
});
```

## Các tham số route

### Tham số bắt buộc

Tất nhiên, đôi khi bạn sẽ cần phải nắm bắt các phân đoạn trong URIs trong định tuyến của bạn. Ví dụ: bạn cần lấy được ID của người dùng từ các URL. Bạn có thể làm như vậy bằng cách định nghĩa các tham số như sau:

```PHP
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

Bạn có thể định nghĩa nhiều tham số theo yêu cầu:

```PHP
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

Các tham số định tuyến luôn được bao bọc bởi cặp dấu ngoặc nhon "{}". Các tham số này sẽ đi vào Closure của định tuyến, khi các định tuyến này được thực thi.

> **Chú ý:** Các tham số không được chứa ký tự `-`. Sử dụng dấu gạch dưới `_` để thay thế.

### Tham số tùy chọn

Thỉnh thoảng, bạn có thể phải chỉ định một số định tuyến, nhưng làm cho nó bắt buộc phải có. Bạn có thể làm như vậy bằng cách đặt một dấu chấm hỏi "`?`" sau tên tham số. Hãy chắc chắn rằng cung cấp cho biến tương ứng một giá trị mặc định:

```PHP
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

### Hạn chế sử dụng biểu thức chính quy

Bạn có thể hạn chế các định dạng của tham số định tuyến bằng cách sử dụng phương thức `where`. Phương thức `where` chấp nhận tên của tham số và một biểu thức chính quy quy định các tham số được hạn chế như thế nào:

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

### Hạn chế toàn cục

Nếu bạn muốn có một số định tuyến luôn bị hạn chế bởi một biểu thức chính quy, bạn có thể sử dụng phương thức `pattern`. Bạn nên xác định những quy định này trong phương thức `boot` của `RouteServiceProvider`:

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

Khi mô hình đã được xác định nó sẽ tự động áp dụng cho tất cả các định tuyến sử dụng tham số:

```PHP
Route::get('user/{id}', function ($id) {
    // Only called if {id} is numeric.
});
```

## Tên của định tuyến

Các định tuyến cho phép đặt tên để thuận tiện cho các URL hoặc chuyển hướng cho các định tuyến cụ thể. Bạn có thể chỉ định một tên cho định tuyến bằng cách sử dụng khóa `as` khi tạo định tuyến:

```PHP
Route::get('user/profile', ['as' => 'profile', function () {
    //
}]);
```

Bạn cũng có thể chỉ định tên cho hành động:

```PHP
Route::get('user/profile', [
    'as' => 'profile', 'uses' => 'UserController@showProfile'
]);
```

Ngoài ra, thay vì chỉ định tên trong mảng định nghĩa route, bạn cũng có thể thêm phương thức `name` vào cuối route:

```PHP
Route::get('user/profile', 'UserController@showProfile')->name('profile');
```

### Nhóm định tuyến và đặt tên các định tuyến

Nếu bạn sử dụng nhóm định tuyến (route groups) bạn có thể chỉ định một khóa bên trong mảng thuộc tính định tuyến, cho phép bạn thiết lập một tiền tố cho tất cả các định tuyến bên trong nhóm:

```PHP
Route::group(['as' => 'admin::'], function () {
    Route::get('dashboard', ['as' => 'dashboard', function () {
        // Route named "admin::dashboard"
    }]);
});
```

### Tạo URL từ định tuyến đã đặt tên

Một khi bạn đã gán tên cho một định tuyến xác định, bạn có thể sử dụng tên của nó khi tạo URLs hoặc chuyển hướng thông qua hàm `route()`:

```PHP
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```

Nếu như tên đã được định nghĩa các tham số, bạn có thể xuyên qua nó như là một đối số thứ hai trong hàm `route()`. Các đối số sẽ được chèn vào vị trí chính xác trên URLs:

```PHP
Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
    //
}]);

$url = route('profile', ['id' => 1]);
```

## Nhóm định tuyến

Nhóm định tuyến cho phép bạn chia sẻ các thuộc tính như middleware hay namespaces, trên nhiều định tuyến mà không cần phải xác định lại chúng trên mỗi định tuyến riêng. Các thuộc tính chung được quy định trong một mảng định dạng là tham số đầu tiên của phương thức `Route::group`.
Để tìm hiểu thêm, chúng ta sẽ đi một số trường hợp sử dụng phổ biến cho tính năng này:

### Middleware

Để gán middleware cho một nhóm, bạn có thể sử dụng khóa `middleware` trong mảng thuộc tính. Middleware sẽ được thực hiện theo thứ tự bạn định nghĩa mảng này:

```PHP
Route::group(['middleware' => 'auth'], function () {
    Route::get('/', function ()    {
        // Uses Auth Middleware
    });

    Route::get('user/profile', function () {
        // Uses Auth Middleware
    });
});
```

### namespaces

Một trường hợp sử dụng chung cho nhóm định tuyến là namespaces được chỉ định với một nhóm của bộ điều khiển. Bạn có thể sử dụng tham số `namespace` trong mangr thuộc tính của bạn để chỉ định namespace cho tất cả bộ điều khiển bên trong nhóm

```PHP
Route::group(['namespace' => 'Admin'], function()
{
    // Controllers Within The "App\Http\Controllers\Admin" Namespace

    Route::group(['namespace' => 'User'], function() {
        // Controllers Within The "App\Http\Controllers\Admin\User" Namespace
    });
});
```

Hãy nhớ rằng, theo mặc định `RouteServiceProvider` bao gồm tệp tin `routes.php` của bạn trong một nhóm namespaces cho phép bạn đăng ký các bộ điều khiển định tuyến mà không xác định đầy đủ tiền tố namespace `App\Http\Controllers`. Vì vậy, chúng ta chỉ cần xác định thành phần của tên đó được đưa ra sau `App\Http\Controllers`.

### Định tuyến tên miền phụ

Nhóm định tuyến cũng có thể được sử dụng để định tuyến đại diện các tên miền phụ. Tên miền phụ có thể được gán tham số định tuyến như URIs, cho phép bạn lấy một phần của tên miền phụ để sử dụng bên trong định tuyến hoặc bộ điều khiển của bạn. Các tên miền phụ có thể được xác định bằng cách sử dụng khóa `domain` trong mảng thuộc tính:

```PHP
Route::group(['domain' => '{account}.myapp.com'], function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

### Các tiền tố định tuyến

Thuộc tính `prefix` có thể sử dụng để thêm tiền tố cho mỗi định tuyến trong một nhóm với một URI. Ví dụ, bạn có thể muốn tất cả các tiền tố trong nhóm là admin:

```PHP
Route::group(['prefix' => 'admin'], function () {
    Route::get('users', function ()    {
        // Matches The "/admin/users" URL
    });
});
```

## Bảo mật CSRF

### Giới thiệu

Laravel làm cho nó dễ dàng để bảo vệ các ứng dụng của bạn từ tấn công giả mạo cross-site request forgery (CSRF). Cross-site request forgery (CSRF) là một loại mã độc, theo đó các lệnh trái phép được thực thi thay cho một người dùng đã xác thực.

Laravel tự động tạo ra một CSRF "token" cho mỗi người dùng hoạt động quản lý bởi ứng dụng. Mã này dử dụng để xác minh rằng người dùng là một trong những người thực sự gửi yêu cầu với ứng dụng.

Bất cứ khi nào bạn tạo một biểu mẫu HTML trong ứng dụng của bạn, bạn nên thêm một trường ẩn CSRF token để bảo mật CSRF có thể xác nhận yêu cầu. Để tạo ra `_token` bạn có thể sử dụng hàm `csrf_fiel` helper function:

```PHP
// Vanilla PHP
<?php echo csrf_field(); ?>

// Blade Template Syntax
{{ csrf_field() }}
```

`csrf_field` được tạo thông qua HTML:

```PHP
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```

Bạn không cần phải tự xác minh CSRF token trên các yêu cầu POST, PUT, DELETE. `VerifyCsrfToken` middleware, được bao gồm trong `web` middleware sẽ tự động xác minh token có phù hợp hay không.

### Loại bỏ URIs khỏi bảo mật CSRF

Đôi khi, bạn có thể muốn loại bỏ URIs khỏi bảo mật CSRF. Ví dụ, nếu bạn đang sử dụng Stripe để xử lý thanh toán và được sử dụng hệ thông  webhook của họ, bạn sẽ cần phải loại trừ đi các định tuyến xử lý của bạn khỏi bảo mật CSRF của Laravel.

Bạn có thể loại bỏ các bảo mật CSRF bằng cách xác định tuyến đường bên ngoài middleware web bên trong tệp tin `routes.php` hoặc bằng cách thêm thuộc tính `$except` trong middleware `VerifyCsrfToken`:

```PHP
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

class VerifyCsrfToken extends BaseVerifier
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
    ];
}
```

### X-CSRF-TOKEN

Ngoài việc kiểm tra CSRF như một tham số POST, middleware `VerifyCsrfToken` cũng sẽ kiểm tra các yêu cầu X-CSRF-TOKEN. Bạn có thể, ví dụ, lưu trữ token trong thẻ meta:

```PHP
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Một khi bạn đã tạo ra các thẻ `meta`, bạn có thể chỉ định một thư viện như jQuery để thêm các thẻ cho tất cả các yêu cầu (request headers). Điều này cung cấp thật đơn giản, thuận tiện đẻ bảo vệ các ứng dụng AJAX của bạn:

```PHP
$.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
});
```

### X-XSRF-TOKEN

Laravel cũng lưu X-XSRF-TOKEN trong một cookie `XSRF-TOKEN`. Bạn có thể sử dụng cookie để thiết lập các yêu cầu `X-XSRF-TOKEN`. Một số thư viện Javascript, như Agular, làm điều này tự động cho bạn. Nó không chác chắn rằng bạn phải tự tay làm điều này.

## Route models binding

Laravel models binding cung cấp một cách thuận tiện đêr đẩy model vào định tuyến của bạn. Ví dụ, thay vì đẩy ID của người dùng bạn có thể đẩy toàn bộ model `User` phù hợp với ID xác định.

### Ràng buộc ngầm định

Laravel sẽ tự động giải quyết gợi ý Eloquent Model được xác định bên trong định tuyến hoặc bộ điều khiển có tên biến phù hợp với tên một phân đoạn route. Ví dụ:

```PHP
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;
});
```

Trong ví dụ này, Eloquent đã gợi ý biến `$user` định nghĩa trong route phù hợp với `{user}` trong URI. Laravel tự động đẩy các mô hình hiện thực có một ID phù hợp với giá trị tương ứng từ URI.

Nếu không tìm thấy trong cơ sở dữ liệu, một phản hồi 404 HTTP sẽ tự động được tạo ra.

### Tùy biến tên khóa

Nếu bạn muốn các mô hình ràng buộc ngầm định để sử dụng một cột cơ sở dữ liệu khác ID khi lấy dữ liệu, bạn có thể ghi đè lên phương thức `getRouteKeyName` trên Eloquent Model:

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

Để đăng ký ràng buộc tường minh, sử dụng phương thức `model` của biến $router để xác định lớp cho một tham số. Bạn nên xác định các ràng buộc của bạn trong phương thức `RouteServiceProvider::boot`:

#### Ràng buộc một tham số cho một mô hình

```PHP
public function boot(Router $router)
{
    parent::boot($router);

    $router->model('user', 'App\User');
}

Tiếp theo, xác định route chứa tham số `{user}`:

$router->get('profile/{user}', function(App\User $user) {
    //
});
```

Vì chúng tôi đã ràng buộc tham số `{user}` trong `App\User`, một thể hiện của `User` sẽ được truyền vào route. Vì vậy, ví dụ, một yêu cầu đến `profile/1` sẽ truyền `User` có ID là 1.

Nếu thể hiện model không được tìm thấy trong cơ sở dữ liệu, một phản ứng 404 HTTP sẽ được tạo ra.

#### Tùy chỉnh the revolution logic

Nếu bạn muốn sử dụng logic giải quyết của riêng bạn, bạn nên sử dụng phương thức `Route::bind`. Thuộc tính bao đóng truyền qua phương thức `bind` sẽ nhận được giá trị của tham biến trên URI, và sẽ trả về một thể hiện của lớp bạn muốn truyền vào route:

$router->bind('user', function ($value) {
    return App\User::where('name', $value)->first();
});

#### Tùy chỉnh "Not found"

Nếu bạn muốn chỉ đinh hành vi "Not found" của bạn, qua một thuộc tính bao đóng `Closure` như một đối số thứ ba của phương thức `model`:

```PHP
$router->model('user', 'App\User', function () {
    throw new NotFoundHttpException;
});
```

## Form Method Spoofing

Biểu mẫu HTML không hỗ trợ phương thức `PUT`, 'DELETE', 'PATCH'. Vì vậy, khi xác định các phương thức trên được gọi từ HTML Form, bạn sẽ phải thêm một trường input ẩn `_method` vào biểu mẫu. Các giá trị được gửi với trường `_method` sẽ được sử dụng như các phương thức HTTP:

```PHP
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
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

## Truy cập vào định tuyến hiện tại

Phương thức `Route::current()` sẽ trả về xử lý các yêu cầu HTTP của định tuyến hiện tại, cho phép bạn xem đầy đủ trong `Illuminate\Routing\Route`:

```PHP
$route = Route::current();

$name = $route->getName();

$actionName = $route->getActionName();
```

Bạn cũng có thể sử dụng phương thức hỗ trợ `currentRouteName` và `currentRouteAction` trên class `Route` để truy cập:

```PHP
$name = Route::currentRouteName();

$action = Route::currentRouteAction();
```

Vui lòng tham khảo các [http://laravel.com/api/5.2/Illuminate/Routing/Router.html](lớp cơ bản của Route) để xem tất cả các phương thức.
