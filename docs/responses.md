# HTTP Responses

- [HTTP Responses](#http-responses)
  - [Tạo Responses](#to-responses)
    - [Gán Headers vào Responses](#gan-headers-vao-responses)
    - [Gán Cookies vào Responses](#gan-cookies-vao-responses)
    - [Cookies & Encryption](#cookies-encryption)
  - [Chuyển trang](#chuyn-trang)
    - [Chuyển trang về routes đã được đặt tên](#chuyn-trang-v-routes-a-c-t-ten)
    - [Populating Parameters Via Eloquent Models](#populating-parameters-via-eloquent-models)
    - [Chuyên trang đến Controller Actions](#chuyen-trang-n-controller-actions)
    - [Redirecting To External Domains](#redirecting-to-external-domains)
    - [Chuyển trang với dữ liệu Flashed Session](#chuyn-trang-vi-d-liu-flashed-session)
  - [Các kiểu Response khác](#cac-kiu-response-khac)
    - [View Responses](#view-responses)
    - [JSON Responses](#json-responses)
    - [File Downloads](#file-downloads)
    - [File Responses](#file-responses)
  - [Response Macros](#response-macros)

## Tạo Responses

#### Chuỗi & Mảng

Tất cả các route và controller nên trả về một response để gửi cho người dùng trình duyệt. Laravel cung cấp vài cách khác nhau trả về responses. Response cơ bản nhất là trả về một chuỗi từ một route hoặc controller. The framework sẽ tự động chuyển chuỗi thành HTTP response đầy đủ:

```PHP
Route::get('/', function () {
    return 'Hello World';
});
```

Ngoài việc trả về một chuỗi từ routes và controllers, bạn có thể trả về một mảng. The framework sẽ tự động chuyển mảng thành JSON response:

```PHP
Route::get('/', function () {
    return [1, 2, 3];
});
```

>Bạn có biết là bạn cũng có thể trả về [Eloquent collections](eloquent-collections.md) từ routes hoặc controllers? Chúng sẽ tự động chuyển thành JSON. Tuyệt vời!

#### Response Objects

Thông thường, bạn không chỉ trả về một chuỗi hoặc một mảng từ route hoặc controller. Mà bạn thường trả về đầy đủ ``Illuminate\Http\Response`` instances hoặc views.

Trả về đầy đủ Response instance cho phép bạn có thể tùy biến HTTP status code và headers của ``response``. Một ``Response``instance kế thừa từ class ``Symfony\Component\HttpFoundation\Response``, nó cung cấp một số phương thức của HTTP responses:

```PHP
Route::get('home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```

### Gán Headers vào Responses

Nhớ rằng hầu hết các phương thức response là có thể móc nối với nhau, cho phép dễ dàng tạo ra một tiến trình response instances. Ví dụ, bạn có thể sử dụng phương thức ``header`` để thêm một danh sách headers cho response trước khi gửi chúng lại cho người dùng:

```PHP
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```

Hoặc, bạn có thể dùng phương thức ``withHeaders`` truyền vào một mảng các headers để thêm vào response:

```PHP
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

### Gán Cookies vào Responses

Phương thức ``cookie``trong response instances cho phép bạn dễ dàng gán cookies cho response. Ví dụ, bạn có thể sử dụng phương thức ``cookie`` để tạo ra một cookie và dễ dàng gán nó vào response instance như sau:

```PHP
return response($content)
                ->header('Content-Type', $type)
                ->cookie('name', 'value', $minutes);
```

Phương thức ``cookie`` ngoài ra còn có một vài đối số ít được sử dụng. Nói chung, đó là những đối số có mục đích giống như những đối số của phương thức [`setcookie`](https://secure.php.net/manual/en/function.setcookie.php) của PHP:

```PHP
->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```

Ngoài ra, bạn cũng thể sử dụng facede của `cookie` tạo ra `queue` đính kèm vào phản hồi gửi đi từ ứng dụng của bạn. Phương thức `queue` chấp nhận `Cookie` instance hay các đối số cần thiết để tạo ra `Cookie` instance.
Các cookie này sẽ được đính kèm vào phản hồi gửi đi trước khi nó được gửi tới trình duyệt:

```php
Cookie::queue(Cookie::make('name', 'value', $minutes));

Cookie::queue('name', 'value', $minutes);
```

### Cookies & Encryption

Mặc định, tất cả cookies được sinh ra bởi Laravel đều được mã hòa và đăng ký vì vậy client không thể được chỉnh sửa hoặc đọc được. Nếu bạn muốn vô hiệu hóa mã hóa cho một tập con cookies tạo ra bởi ứng dụng, bạn có thể sử dụng thuộc tính `$except` trong ``App\Http\Middleware\EncryptCookies`` middleware, nó nằm ở trong thư mục ``app/Http/Middleware``:

```PHP
/**
 * The names of the cookies that should not be encrypted.
 *
 * @var  array
 */
protected $except = [
    'cookie_name',
];
```

## Chuyển trang

Chuyển responses là thể hiện của class ``Illuminate\Http\RedirectResponse``, và chứa các header cần thiếnt cho việc chuyển trang người dùng sang một URL khác. Có vài cách để tạo một  ``RedirectResponse`` instance. Cách đơn giản nhất là sử dụng helper global ```redirect```:

```PHP
Route::get('dashboard', function () {
    return redirect('home/dashboard');
});
```

Thỉnh thoảng bạn có thể muốn chuyển trang của người dùng đến trang trước đó, ví dụ như trường hơp submitted form có lỗi. Bạn có thể làm điều đó bằng cách sử dụng hàm ``back``. Kể từ khi nó kết hợp với [``session``](session.md), đảm bảo rằng route đang gọi hàm ``back`` là sử dụng nhóm middleware ``web`` hoặc có tất cả session middleware được áp dụng:

```PHP
Route::post('user/profile', function () {
    // Validate the request...

    return back()->withInput();
});
```

### Chuyển trang về routes đã được đặt tên

Khi bạn gọi hepler ``redirect`` không có tham số, một thể hiện  ``Illuminate\Routing\Redirector`` được trả về, cho phép bạn gọi bất cứ phương thức trên ``Redirector`` instance. Ví dụ, tạo ra một  ``RedirectResponse`` vào tên route, bạn có thể sử dụng phương thức ``route``:

```PHP
return redirect()->route('login');
```

Nếu route của bạn có tham số, bạn có thể truyền chúng như là đối số thứ hai của phương thức  ``route``:

```PHP
// For a route with the following URI: profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

### Populating Parameters Via Eloquent Models

Nếu bạn chuyển trang đến một route với một tham số "ID" là một thuộc tính thuộc Eloquent model, đơn giản bạn truyền bởi chính model đó. Tham số ID sẽ được lấy ra tự động:

```PHP
// For a route with the following URI: profile/{id}

return redirect()->route('profile', [$user]);
```

Bạn cũng có thể tùy biến giá trị tham số của route, bạn phải ghi đè phương thức ``getRouteKey`` trong Eloquent model của bạn:

```PHP
/**
 * Get the value of the model's route key.
 *
 * @return  mixed
 */
public function getRouteKey()
{
    return $this->slug;
}
```

### Chuyên trang đến Controller Actions

Bạn cũng có thể truyển trang đến [``controller actions``](controllers.md). Đề làm việc đó, truyền controller và tên action vào phương thức ``action``. Nhớ rằng, Bạn không cần phải có đường dẫn đầy đủ của namespace controller, ``RouteServiceProvider``của Laravel nó tự động làm điều đó giúp bạn:

```PHP
return redirect()->action('HomeController@index');
```

Nếu controller route của bạn có tham số, bạn có thể truyền qua như là một tham số thứ hai của phương thức ``action``:

```PHP
return redirect()->action(
    'UserController@profile', ['id' => 1]
);
```

### Redirecting To External Domains

Đôi khi bạn sẽ muốn chuyển tới domain ở ngoài chương trình của bạn, bạn có thể làm điều đó bằng cách gọi phương thức ``away``, nó tạo ra ``RedirectResponse`` mà không cần bất kỳ mã hóa, xác thực hoặc xác minh URL bổ sung nào:

### Chuyển trang với dữ liệu Flashed Session

Chuyển trang tới một URL mới và [``flashing dữ liệu vào session``](session.md#flash-data) có thể làm nó cùng một lúc. Thông thường, chuyển trang được thực hiện sau khi bạn flash vào session thành công. Cho thuận tiện, bạn có thể tạo một thể hiện ``RedirectResponse`` và flash dữ liệu vào session trong một lần, như bên dưới:

```PHP
Route::post('user/profile', function () {
    // Update the user's profile...

    return redirect('dashboard')->with('status', 'Profile updated!');
});
```

Sau khi người dùng được chuyển trang, bạn có thể hiển thị nội dung flashed từ `session`. Ví dụ, sử dụng ``Blade syntax``:

```PHP
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

## Các kiểu Response khác

Phương thức ``response`` có thể được dùng để tạo ra kiểu thể hiện response khác. Khi phương thức  response được gọi không có tham số, một thực hiện của  ``Illuminate\Contracts\Routing\ResponseFactory`` contract được trả về. [Contract](contracts.md) này cung cấp các phương thức cho việc tạo responses.

### View Responses

Nếu bạn muốn kiểm soát status và header của response nhưng bạn cũng muốn trả về một view chứa nội dung của response, bạn có thể sử dụng phương thức ``view``:

```PHP
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```

Tất nhiên, nếu bạn không cần truyền status tùy biến của HTTP hoặc tùy biến header, bạn có thể sử dụng hàm global view.

### JSON Responses

Phương thức ``json`` sẽ tự động đặt ``Content-Type`` header là application/json, cũng như chuyển mảng thành JSON bằng hàm ``json_encode`` của PHP:

```PHP
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);
```

Nếu bạn muốn tạo một JSONP response, bạn có thể sử dụng phương thức json kết hợp với phương thức ``withCallback``:

```PHP
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```

### File Downloads

Phương thức ``download`` có thể dùng để tạo ra một response bắt trình duyệt của người dùng tải file tại đường dẫn. Phương thức ``download`` chấp nhận tên file như là đối số thứ hai của phương thức, mà sẽ xác định tên file được người dùng đang tải. Cuối cùng, bạn có thể truyền một mảng của HTTP headers như là tham số thứ ba của phương thức:

```PHP
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

>Symfony HttpFoundation,là quản lý file tải, yêu cầu file tải có tên là định dạng ASCII.

#### Streamed Downloads

Đôi lúc, bạn sẽ muốn biến chuỗi response của một hoạt động nào đó thành  downloadable response mà không phải ghi nội dung của hoạt động vào ổ cứng, Bạn có thể sử dụng phương thức ``streamDownload`` trong trường hợp này. Phương thức này chấp nhận lời gọi, tên file và mảng tùy chỉnh cho header như là đối số của nó

### File Responses

Phương thức ``file`` sử dụng để hiển thị một file, như là ảnh hoặc PDF, trực tiếp trong trình duyệt của người dùng thay vì phải tải. Phương thức này chấp nhận đường dẫn như là đối số đầu tiên và mảng của header như là đối số thứ hai:

```PHP
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

## Response Macros

Nếu bạn muốn định nghĩa một tùy biến response bạn có thể sử dụng lại routes và controllers, bạn có thể dùng phương thức ``macro`` trong ``Response`` facade. Ví dụ, từ một phương thức ``service`` provider's boot:

```PHP
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Response;

class ResponseMacroServiceProvider extends ServiceProvider
{
    /**
     * Register the application's response macros.
     *
     * @return  void
     */
    public function boot()
    {
        Response::macro('caps', function ($value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

Phương thức macro chấp nhận tên là đối số thứ nhất, và một Closure là đối số thứ hai. Closure của macro sẽ thực thi khi đang gọi macro từ một thực hiện ResponseFactory hoặc phương thức response:

```PHP
return response()->caps('foo');
```