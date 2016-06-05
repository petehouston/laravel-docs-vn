# HTTP Controllers

- [Giới thiệu](#introduction)
- [Controller cơ bản ](#basic-controllers)
- [Controller Middleware](#controller-middleware)
- [RESTful Resource Controllers](#restful-resource-controllers)
    - [Partial Resource Routes](#restful-partial-resource-routes)
    - [Naming Resource Routes](#restful-naming-resource-routes)
    - [Naming Resource Route Parameters](#restful-naming-resource-route-parameters)
    - [Supplementing Resource Controllers](#restful-supplementing-resource-controllers)
- [Dependency Injection & Controllers](#dependency-injection-and-controllers)
- [Route Caching](#route-caching)

<a name="introduction"></a>
## Giới thiệu

Thay vì định nghĩa tất cả logic xử lý request của bạn ở file `routes.php`, thì bạn có thể muốn quản lý việc này bằng cách sử dụng các lớp Controller. Các Controller có thể nhóm các request HTTP có logic liên quan vào cùng một lớp. Các Controller được chứa tại thư mục `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Controllers cơ bản

Dưới đây là một ví dụ về lớp Controller cơ bản. Tất cả các Controller nên là class mở rộng của base Controller đi kèm trong bản cài đặt mặc định của Laravel:

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

Chúng ta có thể định tuyến (route) đến một hoạt động của controller như sau:

    Route::get('user/{id}', 'UserController@showProfile');

Bây giờ, khi mà một request khớp với URI của định tuyến đã được xác định, thì method `showProfile` của lớp `UserController` sẽ được thực thi. Tất nhiên, tham số của định tuyến cũng sẽ được truyền đến method.

#### Controllers & Namespaces

Có điều rất quan trọng cần lưu ý rằng chúng ta không cần phải ghi rõ không gian tên đầy đủ của controller khi định nghĩa định tuyến cho controller. Chúng ta chỉ cần định nghĩa phần tên lớp mà theo sau không gian tên "root" `App\Http\Controllers`. Mặc định, `RouteServiceProvider` sẽ tải file `route.php` trong nhóm định tuyến chứa không gian tên gốc controller.

Nếu bạn muốn gộp hoặc sắp xếp các controller của bạn sử dụng không gian tên của PHP sâu hơn trong thư mục `App\Http\Controllers`, thì đơn giản chỉ cần định nghĩa tên lớp tương đối so với không gian tên gốc `App\Http\Controllers`. Do dó, nếu tên lớp controller đầy đủ của bạn là `App\Http\Controllers\Photos\AdminController`, thì bạn chỉ đăng ký route như sau:

    Route::get('foo', 'Photos\AdminController@method');

#### Naming Controller Routes

Giống như định tuyến đóng kín (closure), bạn có thể đặt tên cho định tuyến controller:

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

Bạn cũng có thể dùng phương thức trợ giúp `route` để sinh ra URL cho một định tuyến controller đã được đặt tên:

    $url = route('name');

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) may be assigned to the controller's routes like so:
[Middleware](/docs/{{version}}/middleware) có thể được gán cho định tuyến của controller như sau:

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

Tuy nhiên, sẽ là tiện lợi hơn nếu như định nghĩa middleware từ trong hàm contructor của controller. Sử dụng method `middleware` từ trong controller của bạn, bạn có thể dễ dàng gán middleware cho controller. Bạn thậm chí có thể hạn chế middleware cho một vài method cụ thể trong lớp controller:

    class UserController extends Controller
    {
        /**
         * Instantiate a new UserController instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log', ['only' => [
                'fooAction',
                'barAction',
            ]]);

            $this->middleware('subscribed', ['except' => [
                'fooAction',
                'barAction',
            ]]);
        }
    }

<a name="restful-resource-controllers"></a>
## RESTful Resource Controllers

Những resource controller làm cho việc xây dựng các RESTful controller xung quanh các nguồn tài nguyển trở nên dễ dàng hơn. Ví dụ như, bạn có thể muốn tạo một controller xử lý những HTTP request liên quan đến "photos" được lưu trữ trong ứng dụng của bạn. Sử dụng câu lệnh Artisan `make:controller`, chúng ta có thể nhanh chóng tạo ra controller:

    php artisan make:controller PhotoController --resource

Câu lệnh Artisan sẽ tạo ra file controller tại `app/Http/Controllers/PhotoController.php`. Controller sẽ bao gồm method cho các hoạt động của tài nguyên sẵn có.

Tiếp theo, bạn có thể muốn đăng ký một định tuyến đa tài nguyên cho controller:

    Route::resource('photo', 'PhotoController');

Khai báo định tuyến duy nhất này tạo ra nhiều định tuyến để xử lý đa dạng các loại hành động RESTful cho tài nguyên "photo". Tương tự như vậy, controller được tạo ra sẽ có sẵn vài method gốc rễ cho từng hành động, bao gồm cả ghi chú thông báo cho bạn những URI và những HTTP method (POST, GET, PUT, PATCH, DELETE) nào chúng xử lý.

#### Những hành động được xử lý bởi Resource Controller

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

Remember, since HTML forms can't make PUT, PATCH, or DELETE requests, you will need to add a hidden `_method` field to spoof these HTTP verbs:

Hãy nhớ rằng, kể từ khi những HTML form không thể tạo ra các request như PUT, PATCH, hay DELETE, thì bạn cần phải thêm một trường ẩn `_method` để giả mạo những HTTP method.

    <input type="hidden" name="_method" value="PUT">

<a name="restful-partial-resource-routes"></a>
#### Định tuyến tài nguyên thành phần

Khi khai báo một định tuyến tài nguyên, bạn có thể chỉ định một tập hợp con của các hành động để xử lý trên định tuyến:

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
#### Đặt tên các định tuyến tài nguyên

Mặc định, tất cả các hành động của controller tài nguyên đều có tên; tuy nhiên, bạn có thể ghi đè những tên đó bằng cách truyền thêm chuỗi `names` tuỳ theo lựa chọn của bạn:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
#### Đặt tên tham số của định tuyến tài nguyên

Mặc định, `Route::resource` sẽ tạo những tham số cho các định tuyến tài nguyên của bạn dựa trên tên tài nguyên. Bạn có thể dễ dàng ghi đè cho từng resource cơ bản bằng cách truyền `parameters` trong chuỗi tuỳ chọn. Chuỗi `parameters` nên là một mảng kết hợp giữa tên tài nguyên và tên tham số:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

 Ví dụ trên sẽ tạo ra những URI sau cho định tuyến `show` của tài nguyên:

    /user/{admin_user}

Thay vì truyền một chuỗi các tên tham số, bạn có thể đơn giản truyền chữ `singular` để hướng dẫn Laravel dùng tên tham số mặc định, nhưng mà làm "khác biệt hoá" chúng:

    Route::resource('users.photos', 'PhotoController', [
        'parameters' => 'singular'
    ]);

    // /users/{user}/photos/{photo}

Ngoài ra, bạn có thể thiết lập tham số của định tuyến tài nguyên trở thành khác biệt toàn cục hoặc thiết lập ánh xạ toàn cục cho tên tham số tài nguyên của bạn.

    Route::singularResourceParameters();

    Route::resourceParameters([
        'user' => 'person', 'photo' => 'image'
    ]);

Khi bạn tuỳ biến tên tham số tài nguyên, nhớ rằng giữ ưu tiên đặt tên là rất quan trọng:

1. Những tham số truyền vào `Route::resource` phải thật rõ ràng.
2. Những ánh xạ tham số toàn cục phải được thiết lập thông qua `Route::resourceParameters`.
3. Cài đặt `singular` phải được truyền qua chuỗi `parameters` đến `Route::resource` hoặc thiết lập thông qua `Route::singularResourceParameters`.
4. Những hành vi mặc định.

<a name="restful-supplementing-resource-controllers"></a>
#### Bổ sung controller tài nguyên

Nếu cần phải thêm những định tuyến bổ sung cho một controller tài nguyên ngoài những định tuyến tài nguyên mặc định, thì bạn nên định nghĩa những định tuyến đó trước khi gọi `Route::resource`; nếu không thì những định tuyến đã được định nghĩa bởi method `resource` sẽ có thể vô tình bị ưu tiên hơn những định tuyến bổ sung của bạn.

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers

#### Constructor Injection

Phần [service container](/docs/{{version}}/container) của Laravel được dùng để xử lý tất cả các controller của Laravel. Kết quả là, bạn có thể "type-hint" bất cứ thành phần phụ thuộc nào mà controller của bạn cần vào trong constructor của controller. Các thành phần phụ thuộc sẽ được tự động xử lý và được thêm vào trong "instance" của controller:

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

Of course, you may also type-hint any [Laravel contract](/docs/{{version}}/contracts). If the container can resolve it, you can type-hint it.

Tất nhiên là bạn cũng có thể "type-hint" bất cứ [Laravel contract](/docs/{{version}}/contracts) nào. Nếu như container có thể xử lý thì bạn có thể "type-hint" nó.

#### Method Injection

Ngoài constructor injection, bạn cũng có thể "type-hint" các thành phần phụ thuộc trong các method của controller. Ví dụ, hãy "type-hint" instance của `Illuminate\Http\Request` vào một trong những method của ta:

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

Nếu như method của controller của bạn cũng chờ đợi đầu vào từ tham sổ của định tuyến, thì đơn giản là liệt kê các đối số của định tuyến vào phía sau các thành phần phụ thuộc khác. Ví dụ, nếu định tuyến của bạn được định nghĩa như sau:

    Route::put('user/{id}', 'UserController@update');

Bạn vẫn có thể "type-hint" `Illuminate\Http\Request` và truy cập vào tham số định tuyến của bạn `id` bằng cách định nghĩa method controller của bạn như sau:
 
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

<a name="route-caching"></a>
## Route Caching

> **Note:** Bộ nhớ đệm định tuyến (Route Caching) không hoạt động với các định tuyến dạng Closure. Để sử dụng bộ nhớ đệm định tuyến thì bạn cần phải chuyển các định tuyến Closure sang sử dụng các lớp controller.

Nếu như ứng dụng của bạn chỉ sử dụng các định tuyến dạng controller, thì bạn có thể sử dụng phần nâng cao của bộ nhớ đệm định tuyến của Laravel. Sử dụng bộ nhớ đệm định tuyến sẽ giảm mạng thời gian cần để đăng ký tất cả các định tuyến trong ứng dụng của bạn. Trong một vài trường hợp, việc đăng ký định tuyến của bạn có thể nhanh hơn đến 100 lần! Để tạo ra bộ nhớ định tuyến, bạn chỉ cần chạy lệnh Artisan `route:cache`:

    php artisan route:cache

Đó là tất cả! File bộ nhớ đệm định tuyến của bạn sẽ được sử dụng thay cho file `app/Http/routes.php`. Nhớ rằng, nếu bạn thêm bất cứ route mới nào thì bạn cần phải tạo mới bộ nhớ đệm định tuyến. Do đó, bạn chỉ nên chạy câu lệnh `route:cache` trong quá trình phát triển dự án.

Để xoá file bộ nhớ đệm định tuyến mà không tạo file mới, sử dụng câu lện `route:clear`:

    php artisan route:clear
