# HTTP Controllers

- [HTTP Controllers](#http-controllers)
  - [Giới thiệu](#gii-thiu)
  - [Controllers cơ bản](#controllers-c-bn)
    - [Định nghĩa COntrollers](#nh-ngha-controllers)
    - [Controllers & Namespaces](#controllers-namespaces)
    - [Single Action Controllers](#single-action-controllers)
    - [Naming Controller Routes](#naming-controller-routes)
  - [Controller Middleware](#controller-middleware)
  - [Resource Controllers](#resource-controllers)
    - [Những hành động được xử lý bởi Resource Controller](#nhng-hanh-ng-c-x-l-bi-resource-controller)
    - [Định tuyến tài nguyên thành phần](#nh-tuyn-tai-nguyen-thanh-phn)
    - [Đặt tên các định tuyến tài nguyên](#t-ten-cac-nh-tuyn-tai-nguyen)
    - [Đặt tên tham số của định tuyến tài nguyên](#t-ten-tham-s-ca-nh-tuyn-tai-nguyen)
    - [Localizing Resource URIs](#localizing-resource-uris)
    - [Bổ sung resource controller](#b-sung-resource-controller)
  - [Dependency Injection & Controllers](#dependency-injection-controllers)
    - [Constructor Injection](#constructor-injection)
    - [Method Injection](#method-injection)
  - [Route Caching](#route-caching)

## Giới thiệu

Thay vì định nghĩa tất cả logic xử lý request của bạn ở file `routes.php`, thì bạn có thể muốn quản lý việc này bằng cách sử dụng các lớp Controller. Các Controller có thể nhóm các request HTTP có logic liên quan vào cùng một lớp. Các Controller được chứa tại thư mục `app/Http/Controllers`.

## Controllers cơ bản

### Định nghĩa COntrollers

Dưới đây là một ví dụ về lớp Controller cơ bản. Tất cả các Controller nên là class mở rộng của base Controller đi kèm trong bản cài đặt mặc định của Laravel.Class base controller cung cấp một vài phương thức như middleware có thể sử dụng để gắn middleware vào controller:

```PHP
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
      * Show the profile for the given user.
      *
      * @param  int  $id
      * @return Response
      */
    public function showProfile($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

Chúng ta có thể định tuyến (route) đến một hoạt động của controller như sau:

```PHP
Route::get('user/{id}', 'UserController@showProfile');
```

Bây giờ, khi mà một request khớp với URI của định tuyến đã được xác định, thì method `showProfile` của lớp `UserController` sẽ được thực thi. Tất nhiên, tham số của định tuyến cũng sẽ được truyền đến method.

>Controllers không yêu cầu kế thừa từ base class. Tuy nhiên, bạn sẽ không có quyền truy cập vào các tính năng tiện lợi như phương thức middleware, validate, và dispatch.

### Controllers & Namespaces

Có điều rất quan trọng cần lưu ý rằng chúng ta không cần phải ghi rõ không gian tên đầy đủ của controller khi định nghĩa định tuyến cho controller. Kể từ khi ``RouteServiceProvider`` tải file route bên trong nhóm route có chứa namespace, chúng ta chỉ cần chỉ định tên class sau  ``App\Http\Controllers`` namespace.

Nếu bạn muốn gộp hoặc sắp xếp các controller của bạn sử dụng không gian tên của PHP sâu hơn trong thư mục `App\Http\Controllers`, thì đơn giản chỉ cần định nghĩa tên lớp tương đối so với không gian tên gốc `App\Http\Controllers`. Do dó, nếu tên lớp controller đầy đủ của bạn là `App\Http\Controllers\Photos\AdminController`, thì bạn chỉ đăng ký route như sau:

```PHP
Route::get('foo', 'Photos\AdminController@method');
```

### Single Action Controllers

Nếu bạn muốn định nghĩa một controller xử lý duy nhất một action, bạn có thể dùng phương thức  ``__invoke`` trong controller:

```PHP
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class ShowProfile extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function __invoke($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

Khi đó bạn đăng ký một route cho một action controllers, bạn không cần xác định phương thức:

```PHP
Route::get('user/{id}', 'ShowProfile');
```

### Naming Controller Routes

Giống như định tuyến đóng kín (closure), bạn có thể đặt tên cho định tuyến controller:

```php
Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);
```

Bạn cũng có thể dùng phương thức trợ giúp `route` để sinh ra URL cho một định tuyến controller đã được đặt tên:

```php
$url = route('name');
```

## Controller Middleware

[Middleware](middleware.md) có thể được gán cho định tuyến của controller như sau:

```PHP
Route::get('profile', 'UserController@show')->middleware('auth');
```

Tuy nhiên, sẽ là tiện lợi hơn nếu như định nghĩa middleware từ trong hàm contructor của controller. Sử dụng method `middleware` từ trong controller của bạn, bạn có thể dễ dàng gán middleware cho controller. Bạn thậm chí có thể hạn chế middleware cho một vài method cụ thể trong lớp controller:

```PHP
class UserController extends Controller
{
    /**
     * Instantiate a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->middleware('subscribed')->except('store');
    }
}
```

Controller còn cho phép bạn đăng ký middleware sử dụng một Closure. Phương thức này khá thuận tiện để định nghĩa một middleware cho một controller mà không cần định nghĩa class middleware:

```PHP
$this->middleware(function ($request, $next) {
    // ...

    return $next($request);
});
```

>Bạn có thể gán middleware cho một tập con các action của controller; tuy nhiên, tập con action có thể to ra khi controller của bạn nhiều action. Vì thế, nên cân nhắc việc chia thành nhiều controller nhỏ hơn.

## Resource Controllers

Laravel resource routing có thể tạo ra route cho "CRUD" ở một controller chỉ với một dòng code. Ví dụ như, bạn có thể muốn tạo một controller xử lý những HTTP request liên quan đến "photos" được lưu trữ trong ứng dụng của bạn. Sử dụng câu lệnh Artisan `make:controller`, chúng ta có thể nhanh chóng tạo ra controller:

```Shell
php artisan make:controller PhotoController --resource
```

Câu lệnh Artisan sẽ tạo ra file controller tại `app/Http/Controllers/PhotoController.php`. Controller sẽ bao gồm method cho các hoạt động của tài nguyên sẵn có.

Tiếp theo, bạn có thể muốn đăng ký một định tuyến đa tài nguyên cho controller:

```PHP
Route::resource('photo', 'PhotoController');
```

Khai báo định tuyến duy nhất này tạo ra nhiều định tuyến để xử lý đa dạng các loại hành động cho tài nguyên "photo". Tương tự như vậy, controller được tạo ra sẽ có sẵn vài method gốc rễ cho từng hành động, bao gồm cả ghi chú thông báo cho bạn những URI và những HTTP method (POST, GET, PUT, PATCH, DELETE) nào chúng xử lý.

Bạn có thể xử lý nhiều resource controllers một lần bằng cách sử dụng mảng trong phương thức ``resources``

```PHP
Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

### Những hành động được xử lý bởi Resource Controller

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

#### Specifying The Resource Model

Nếu bạn đang sử dụng route model binding và muốn phương thức của resource controller được tạo ra theo gợi ý có sẵn, bạn có thể dùng tùy chọn ``--model`` khi tạo controller

```Shell
php artisan make:controller PhotoController --resource --model=Photo
```

#### Spoofing Form Methods

Hãy nhớ rằng, kể từ khi những HTML form không thể tạo ra các request như PUT, PATCH, hay DELETE, thì bạn cần phải thêm một trường ẩn `_method` để giả mạo những HTTP method.

```PHP
<input type="hidden" name="_method" value="PUT">
```

### Định tuyến tài nguyên thành phần

Khi bạn khai báo một resource route, bạn có thể chỉ định các tập con action của controller cần xử lý thay vì toàn bộ action mặc định ban đầu:

```PHP
Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);
```

#### API Resource Routes

Khi tuyên bố các tuyến đường tài nguyên sẽ được tiêu thụ bởi các API, bạn thường sẽ muốn loại trừ các tuyến đường dẫn các mẫu HTML như create và edit. Để thuận tiện, bạn có thể sử dụng phương thức ``apiResourcephương`` để tự động loại trừ hai tuyến đường:

```PHP
Route::apiResource('photos', 'PhotoController');
```

Bạn có thể đăng ký nhiều trình điều khiển tài nguyên API cùng một lúc bằng cách truyền một mảng vào phương thức``apiResourcesphương``:

```PHP
Route::apiResources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

Để nhanh chóng tạo ra một bộ điều khiển tài nguyên API không bao gồm các phương thức  createhoặc edit, sử dụng --apichuyển đổi khi thực hiện lệnh:``make:controller``

```Shell
php artisan make:controller API/PhotoController --api
```

### Đặt tên các định tuyến tài nguyên

Mặc định, tất cả các hành động của resource controller đều có tên; tuy nhiên, bạn có thể ghi đè những tên đó bằng cách truyền thêm chuỗi `names` tuỳ theo lựa chọn của bạn:

```PHP
Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);
```

### Đặt tên tham số của định tuyến tài nguyên

Mặc định, `Route::resource` sẽ tạo những tham số cho các định tuyến tài nguyên của bạn dựa trên tên tài nguyên. Bạn có thể dễ dàng ghi đè cho từng resource cơ bản bằng cách truyền `parameters` trong chuỗi tuỳ chọn. Chuỗi `parameters` nên là một mảng kết hợp giữa tên tài nguyên và tên tham số:

```PHP
Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);
```

 Ví dụ trên sẽ tạo ra những URI sau cho định tuyến `show` của tài nguyên:

```PHP
/user/{admin_user}
```

### Localizing Resource URIs

Theo mặc định, ``Route::resource`` sẽ tạo ra  URI tài nguyên bằng cách sử dụng các động từ tiếng Anh. Nếu bạn muốn đặt theo cách từ ở địa phương bạn, bạn có thể sử dụng phương thức ``Route::resourceVerbs``. Điều này có thể làm trong phương thức ``boot`` hoặc ``AppServiceProvider``

```php
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

Khi các động từ đã được chỉnh lại, các route resoure đã được tạo ra như ``Route::resource('fotos', 'PhotoController')`` sẽ tạo ra các URL sau:

```php
/fotos/crear

/fotos/{foto}/editar
```

### Bổ sung resource controller

Nếu cần phải thêm những định tuyến bổ sung cho một resource controller ngoài những định tuyến tài nguyên mặc định, thì bạn nên định nghĩa những định tuyến đó trước khi gọi `Route::resource`; nếu không thì những định tuyến đã được định nghĩa bởi method `resource` sẽ có thể vô tình bị ưu tiên hơn những định tuyến bổ sung của bạn.

```PHP
Route::get('photos/popular', 'PhotoController@method');

Route::resource('photos', 'PhotoController');
```

>Bạn nên tập trung vào controllers. Nếu bạn thấy mình thường xuyên thêm các route bên ngoài của các resource route thì hãy cân nhắc chia nhỏ controller hơn.

## Dependency Injection & Controllers

### Constructor Injection

Phần [service container](container.md) của Laravel được dùng để xử lý tất cả các controller của Laravel. Kết quả là, bạn có thể "type-hint" bất cứ thành phần phụ thuộc nào mà controller của bạn cần vào trong constructor của controller. Các thành phần phụ thuộc sẽ được tự động xử lý và được thêm vào trong "instance" của controller:

```PHP
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
      * The user repository instance.
      */
    protected $users;

    /**
      * Create a new controller instance.
      *
      * @param  UserRepository  $users
      * @return void
      */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
}
```

Tất nhiên là bạn cũng có thể "type-hint" bất cứ [Laravel contract](contracts.md) nào. Nếu như container có thể xử lý thì bạn có thể "type-hint" nó.

### Method Injection

Ngoài constructor injection, bạn cũng có thể "type-hint" các thành phần phụ thuộc trong các method của controller. Ví dụ, hãy "type-hint" instance của `Illuminate\Http\Request` vào một trong những method của ta:

```PHP
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
      * Store a new user.
      *
      * @param  Request  $request
      * @return Response
      */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}
```

Nếu như method của controller của bạn cũng chờ đợi đầu vào từ tham sổ của định tuyến, thì đơn giản là liệt kê các đối số của định tuyến vào phía sau các thành phần phụ thuộc khác. Ví dụ, nếu định tuyến của bạn được định nghĩa như sau:

```PHP
Route::put('user/{id}', 'UserController@update');
```

Bạn vẫn có thể "type-hint" `Illuminate\Http\Request` và truy cập vào tham số định tuyến của bạn `id` bằng cách định nghĩa method controller của bạn như sau:

```PHP
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
  /**
    * Update the specified user.
    *
    * @param  Request  $request
    * @param  string  $id
    * @return Response
    */
  public function update(Request $request, $id)
  {
      //
  }
}
```

## Route Caching

> **Note:** Bộ nhớ đệm định tuyến (Route Caching) không hoạt động với các định tuyến dạng Closure. Để sử dụng bộ nhớ đệm định tuyến thì bạn cần phải chuyển các định tuyến Closure sang sử dụng các lớp controller.

Nếu như ứng dụng của bạn chỉ sử dụng các định tuyến dạng controller, thì bạn có thể sử dụng phần nâng cao của bộ nhớ đệm định tuyến của Laravel. Sử dụng bộ nhớ đệm định tuyến sẽ giảm mạng thời gian cần để đăng ký tất cả các định tuyến trong ứng dụng của bạn. Trong một vài trường hợp, việc đăng ký định tuyến của bạn có thể nhanh hơn đến 100 lần! Để tạo ra bộ nhớ định tuyến, bạn chỉ cần chạy lệnh Artisan `route:cache`:

```Shell
php artisan route:cache
```

Sau khi chạy lệnh, file cached routes của bạn sẽ được tải với mọi request. Nhớ rằng, nếu bạn thêm một route mới bạn cần phải làm mới lại route cache. Vì ký do này bạn chỉ lên chạy một lần khi  ``route:cache`` ứng dụng của bạn deploy.

Để xoá file bộ nhớ đệm định tuyến mà không tạo file mới, sử dụng câu lện `route:clear`:

```Shell
php artisan route:clear
```