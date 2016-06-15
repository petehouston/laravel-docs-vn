# HTTP Middleware

- [Giới thiệu](#introduction)
- [Tạo middleware](#defining-middleware)
- [Đăng kí middleware](#registering-middleware)
    - [Global middleware](#global-middleware)
    - [Thiết lập middleware cho routes](#assigning-middleware-to-routes)
    - [Tạo nhóm middleware](#middleware-groups)
- [Middleware Parameters](#middleware-parameters)
- [Terminable Middleware](#terminable-middleware)

<a name="introduction"></a>
## Giới thiệu

HTTP middleware cung cấp một giải pháp tiện ích cho việc lock các HTTP request vào ứng dụng. Ví dụ, Laravel có chứa một middleware xác thực người dùng đăng nhập vào hệ thống. Nếu user chưa đăng nhập, middleware sẽ chuyển hướng user tới màn hình login. Còn nếu user đã đăng nhập rồi, thì middleware sẽ cho phép request được thực hiện tiếp tiến trình xử lý.

Tất nhiên là có thể viết thêm middleware để thực hiện nhiều tác vụ nữa ngoài việc kiểm tra đăng nhập vào hệ thống. Middleware CORS chịu trách nhiệm cho việc thêm các header hợp lý vào trong tất cả các response gửi ra ngoài. Middleware log có thể thực hiện ghi log cho tất cả các request tới chương trình.

Vài middleware đã có sẵn trong Laravel framework, bao gồm middleware cho bảo trì, xác thực, phòng chống CSRF và còn nữa. Tất cả những middleware này nằm trong thư mục `app/Http/Middlware`.

<a name="defining-middleware"></a>
## Tạo middleware

Để tạo một middleware mới, sử dụng câu lệnh Artisan `make:middleware`:

    php artisan make:middleware AgeMiddleware

Câu lệnh này sẽ tạo class `AgeMiddleware` bên trong thư mục `app/Http/Middleware`. Trong middleware này, chúng ta sẽ chỉ cho phép truy cập vào route nếu như giá trị `age` cung cấp lớn hơn 200. Nếu không, thì sẽ chuyển hướng users trở lại "home" URI.

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AgeMiddleware
    {
        /**
         * Run the request filter.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->input('age') <= 200) {
                return redirect('home');
            }

            return $next($request);
        }

    }

Như bạn thấy, nếu `age` nhỏ hơn hoặc bằng `200`, middleware sẽ trả lại một HTTP chuyển hướng tới client; ngược lại, request sẽ được gửi tiếp để xử lý. Để truyền request vào sâu hơn trong ứng dụng (cho phép middleware "vượt qua"), đơn giản chỉ cần gọi callback `$next` với `$request`.

Tốt nhất là hãy hình dung middleware là một chuỗi các "layers" trên HTTP request cần phải đi qua trước khi đi vào trong chương trình. Mỗi layer sẽ thực hiện kiểm tra request và thậm chí có thể huỷ từ chối request hoàn toàn.

### *Before* / *After* Middleware

Việc một middleware chạy trước hay sau một request phụ thuộc vào chính nó. Ví dụ, middleware dưới đây sẽ thực hiện vài tác vụ **trước khi** request được chương trình xử lý:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

Tuy nhiên, middleware này sẽ thực hiện việc của nó **sau khi** request được xử lý bởi chương trình:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>
## Đăng kí middleware

<a name="global-middleware"></a>
### Global Middleware

Nếu bạn muốn một middleware có thể được thực thi trong mỗi HTTP request tới chương trình, đơn giản chỉ cần thêm tên class của middleware đó vào trong thuộc tính `$middleware` của class `app/Http/Kernel.php`.

<a name="assigning-middleware-to-routes"></a>
### Thiết lập middleware cho route

Nếu bạn muốn thiết lập middleware cho một số route cụ thể, bạn đầu tiên cần phải thêm middleware vào trong biến `$routeMiddleware` trong file `app/Http/Kernel.php` và đặt cho nó một key:

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

Sau khi đã được khai báo trong HTTP Kernel, bạn có thể sử dụng khoá `middleware` để thiết lập thông số cài vào trong route:

    Route::get('admin/profile', ['middleware' => 'auth', function () {
        //
    }]);

Sử dụng một `array` để thực hiện gán nhiều middleware vào trong route:

    Route::get('/', ['middleware' => ['first', 'second'], function () {
        //
    }]);

Ngoài ra, bạn cũng có thể thực hiện móc nối hàm `middleware` vào trong khai báo của route:

    Route::get('/', function () {
        //
    })->middleware(['first', 'second']);

Khi gán middleware, bạn cũng có thể sử dụng tên class đầy đủ của middleware muốn gán:

    use App\Http\Middleware\FooMiddleware;

    Route::get('admin/profile', ['middleware' => FooMiddleware::class, function () {
        //
    }]);

<a name="middleware-groups"></a>
### Tạo nhóm middleware

Sẽ có lúc bạn muốn thực hiện nhóm một vài middleware lại vào trong một khoá để có thể thực hiện gán vào route dễ dàng hơn. Bạn có thể làm như vậy bằng cách sử dụng thuộc tính `$middlewareGroups` của HTTP kernel.

Về cơ bản, Laravel cung cấp sẵn hai nhóm middleware thường sử dụng mà bạn có thể muốn áp dụng cho web UI hay API:

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

Các nhóm middleware được gán vào route và controller action sử dụng cú pháp tương tự như với từng middleware riêng. Việc sử dụng nhóm middleware sẽ làm cho việc gán các middleware vào trong một route trở nên tiện hơn:

    Route::group(['middleware' => ['web']], function () {
        //
    });

Nên nhớ là, nhóm middleware `web` được tự động áp dụng vào trong file `routes.php` qua `RouteServiceProvider`.

<a name="middleware-parameters"></a>
## Middleware Parameters

Middleware cũng có thể nhận các tham số truyền vào. Ví dụ, nếu chương trình cần xác nhận user đã được xác thực có "role" cụ thể trước khi thực hiện một thao tác nào đó, bạn có thể tạo ra `RoleMiddleware` để nhận tên của role như một tham số.

Các tham số của middleware sẽ được truyền vào thành tham số của hàm `handle` ngay sau tham số `$next`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * Run the request filter.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

Tham số cho middleware cũng có thể được khai báo trên route bằng cách phân cách tên middleware và tham số bởi dấu `:`. Nhiều tham số phân cách nhau bởi dấu phẩy `,`:

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

<a name="terminable-middleware"></a>
## Terminable Middleware

Sẽ có lúc một middleware cần thực hiện chỉ sau khi HTTP response đã được gửi xong cho trình duyệt. Ví dụ, "session" middleware đi kèm với Laravel cung cấp session data cho storage _sau khi_ response được gửi tới trình duyệt. Để làm được việc này, cần phải tạo một middleware kiểu "kết thúc" bằng cách thêm vào hàm `terminate` vào trong middleware:

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

Hàm `terminate` sẽ nhận cả request và response. Khi mà bạn khai báo một terminable middleware, bạn phải thêm nó vào trong danh sách global middleware trong HTTP kernel.

Khi gọi hàm `terminate` trong middleware, Laravel sẽ thực hiện resolve một instance mới cho middleware từ [service container](/docs/{{version}}/container). Nếu bạn muốn sử dụng cùng một instance khi mà `handle` và `terminate` được gọi, đăng kí middleware vào trong container sử dụng hàm `singleton`.