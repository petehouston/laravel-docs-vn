# URL Generation

- [URL Generation](#url-generation)
  - [Giới thiệu](#gii-thiu)
  - [Cơ bản](#c-bn)
    - [Cơ bản về tạo URLs](#c-bn-v-to-urls)
    - [Truy cập vào URL hiện tại](#truy-cp-vao-url-hin-ti)
  - [URL cho route đã đặt tên](#url-cho-route-a-t-ten)
    - [Signed URLs](#signed-urls)
  - [URLs For Controller Actions](#urls-for-controller-actions)
  - [Default Values](#default-values)

## Giới thiệu

Laravel cung cấp vài trợ giúp bạn trong việc tạo ra URL cho ứng dụng của bạn. Tất nhiên, những điều này chủ yếu hữu ích khi tạo liên kết tới templates của bạn và API responses hoặc tạo ra phản hồi chuyển hướng tới phần khác của ứng dụng của bạn.

## Cơ bản

### Cơ bản về tạo URLs

Trợ giúp ``url`` có thể sử dụng để tạo ra URL tùy chỉnh cho ứng dụng của bạn. URl được tạo ra sẽ tự động sử dụng lược đồ (HTTP hoặc HTTPS) và máy chủ lưu trữ từ yêu cầu hiện tại:

```php
$post = App\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

### Truy cập vào URL hiện tại

Nếu không có đường dẫn nào được cung cấp cho phương thức ``url``, một cá thể của ``Illuminate\Routing\UrlGenerator`` được trả về, cho phép bạn truy cập thông tin về URL hiện tại:

```php
// Get the current URL without the query string...
echo url()->current();

// Get the current URL including the query string...
echo url()->full();

// Get the full URL for the previous request...
echo url()->previous();
```

Mỗi phương pháp này cũng có thể được truy cập thông qua URL [facade](facades.md):

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

## URL cho route đã đặt tên

``Route`` có thể được sử dụng để tạo URL cho các tuyến được đặt tên. Các v được đặt tên cho phép bạn tạo các URL mà không cần kết hợp với URL thực được xác định trên route. Bởi vậy, nếu URL của route thay đổi, thì cũng không cần thay đổi với nhưng hàm gọi roure đó. Ví dụ, giả sử ứng dụng của bạn chứa route đã được định nghĩa như ở dưới:

```php
Route::get('/post/{post}', function () {
    //
})->name('post.show');
```

Để tạo ra URL cho route này, bản có thể sử dụng như sau:

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

Bạn thường sẽ tạo URL bằng cách sử dụng khóa chính của các [Eloquent model](eloquent.md) . Vì lý do này, bạn có thể truyền các [Eloquent model](eloquent.md) làm giá trị tham số. Trình ``route`` sẽ tự động trích xuất khóa chính của v:

```php
echo route('post.show', ['post' => $post]);
```

### Signed URLs

Laravel cho phép bạn dễ dàng tạo ra các URLs đã `được ký` cho các route đã đặt tên. Những URL này sẽ này có một "chữ ký" nối thêm vào chuỗi truy vấn cho phép laravel xác nhận chuỗi URL đó chưa bị sửa đổi kể từ khi được tạo ra. URL được kí đặc biết hữu ích cho route có thể truy cập công khai nhưng cần nhưng cần một lớp bảo vệ chống lại thao tác URL.

Ví dụ, bạn sử dụng Signed URLs để tạo ra triển khai liên kết "hủy đăng ký" công khai được gửi qua email tới khách hàng của bạn. Để tạo ra Signed URLs cho route đã đặt tên, sử dụng phương thức ``signedRoute`` của ``URL`` facade:

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

Nếu bạn muốn tạo một URL tuyến đường có chữ ký có thời hạn, bạn có thể sử dụng phương thức ``temporarySignedRoute``:

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

#### Validating Signed Route Requests

Để xác nhận một yêu cầu gửi đến có chữ ký hợp lệ, bạn nên gọi phương thức ``hasValidSignature`` trong ``Request`` gửi đến:

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

Ngoài ra, bạn có thể gán ``Illuminate\Routing\Middleware\ValidateSignature`` cho route của bạn. Nếu nó không tồn tai, thì bạn nên cho nó trong mảng ``routeMiddleware`` của HTTP kernel:

```php
/**
 * The application's route middleware.
 *
 * These middleware may be assigned to groups or used individually.
 *
 * @var array
 */
protected $routeMiddleware = [
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
];
```

Khi bạn đã đăng ký cho phần mềm ở kernel của bạn, bạn có thể gán nó cho route. Nếu như request gửi đến không có chữ kí hợp lệ, phần mềm sẽ tự động gọi ra request lỗi ``403``

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

## URLs For Controller Actions

Hàm ``action`` tạo ra URL cho các hành động của controller đưa ra. Bạn không cần phải đưa ra đường dẫn đầy đủ của controler, thay vao đó chỉ cần đưa ra địa chỉ tương đối của controller so với ``App\Http\Controllers``

```php
$url = action('HomeController@index');
```

Nếu như phương thức của controller yêu cầu đối số từ route, bạn có thể đưa chúng vào như tham số thứ 2 của hàm:

```php
$url = action('UserController@profile', ['id' => 1]);
```

## Default Values

Với vài ứng dụng, bạn có thể sẽ muốn chỉ định giá trị đặc biệt cho các tham số trên URL. Ví dụ, giả sự bạn có định nghĩa route có tham số ``{locale}`` như sau:

```php
Route::get('/{locale}/posts', function () {
    //
})->name('post.index');
```

Nó sẽ cồng kênh khi đưa ``locale`` mỗi khi bạn gọi ``route`` nên bạn có thể sự dụng phương thức ``URL::defaults`` để đặt giá trị mặc định cho tham số đó  sẽ luôn được áp dụng trong yêu cầu hiện tại. Bạn có thể gọi phương thức này từ phần mềm trung gian của route để bạn có quyền truy cập vào yêu cầu hiện tại:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\URL;

class SetDefaultLocaleForUrls
{
    public function handle($request, Closure $next)
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```

Khi giá trị mặc định cho tham số locale đã được đặt, bạn không còn cần phải đưa giá trị của nó khi tạo URL thông qua trình trợ giúp ``route``.