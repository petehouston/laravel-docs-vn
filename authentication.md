# Authentication

- [Introduction](#introduction)
    - [Database Considerations](#introduction-database-considerations)
- [Bắt đầu nhanh với Authentication](#authentication-quickstart)
    - [Routing](#included-routing)
    - [Views](#included-views)
    - [Authenticating](#included-authenticating)
    - [Retrieving The Authenticated User](#retrieving-the-authenticated-user)
    - [Protecting Routes](#protecting-routes)
    - [Authentication Throttling](#authentication-throttling)
- [Manually Authenticating Users](#authenticating-users)
    - [Remembering Users](#remembering-users)
    - [Other Authentication Methods](#other-authentication-methods)
- [HTTP Basic Authentication](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication](#stateless-http-basic-authentication)
- [Resetting Passwords](#resetting-passwords)
    - [Database Considerations](#resetting-database)
    - [Routing](#resetting-routing)
    - [Views](#resetting-views)
    - [After Resetting Passwords](#after-resetting-passwords)
    - [Customization](#password-customization)
- [Social Authentication](https://github.com/laravel/socialite)
- [Adding Custom Guards](#adding-custom-guards)
- [Adding Custom User Providers](#adding-custom-user-providers)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Laravel giúp cho việc thực hiện việc xác thực vô cùng đơn giản. Trong thực tế, hầu hết mọi thứ đã được cấu hình cho bạn mà bạn đéo thể tưởng tượng nổi (out of the box). Các file cấu hình xác thực được đặt tại `config/auth.php`, bao gồm một số hướng dẫn tùy biến rõ ràng cho việc tinh chỉnh cách xử lí của các dịch vụ authentication.

Tại phần lõi của nó, các cơ sở của Laravel's authentication được tạo bởi các "guards" và "providers". Guards định nghĩa cái cách mà các user được xác thực cho mỗi request. Ví dụ, Laravel mang theo một `session` guard cái mà duy trì trạng thái bằng cách sử dụng session storage và cookies và một `token` guard, cái mà xác thực user bằng cách sử dụng một "API token" cái mà được truyền cùng mỗi request.

Providers định nghĩa cách mà user được truy xuất từ lưu trữ không đổi (persistent storage) của bạn. Laravel hỗ trợ cho việc truy xuất các user sử dụng Eloquent và Query Builder.

Đùng lo lắng nếu tất các điều này nghe có vẻ bối rối. Hầu hết các ứng dụng sẽ không cần tùy biến các cấu hình xác thực mặc định.

<a name="introduction-database-considerations"></a>
### Database Considerations

Mặc định, Laravel bao gồm một [Eloquent model](/docs/{{version}}/eloquent) `App\User` trong thư mục `app`. Model này có thể sử dụng với Eloquent authentication driver mặc định.

Khi xây dựng database schema cho model `App\User`, đảm bảo rằng độ dài cột password tối thiểu là 60 kí tự, mặc định với 255 kí tự sẽ là 1 lựa chọn tốt.

Bạn cũng nên xác nhận table `user` ( hoặc một table khác tương đương ) gồm một giá trị nullable, cột `remember_token` 100 kí tự. Cột này sẽ được dùng để lưu một token cho session "remember me" khi đang được duy trì bởi ứng dụng của bạn.

<a name="authentication-quickstart"></a>
## Bắt đầu nhanh với Authentication

Laravel mang tới 2 authentication controllers tuyệt vời, được đặt trong namespace `App\Http\Controllers\Auth`. `AuthController` xử lí user đăng kí mới và xác nhận họ, trong khi `PasswordController` bao gồm logic giúp cho các user đã tồn tại reset password. Mỗi controllers sử dụng một trait để bao gồm các phương thức cần thiết của chúng. Với nhiều ứng dụng, bạn sẽ không cần phải sửa đổi toàn bộ các controller.

<a name="included-routing"></a>
### Routing

Laravel cung cấp một cách nhanh chóng để sinh ra toàn bộ các route và view cần thiết cho authentication chỉ với 1 command:

    php artisan make:auth

Command này nên được dùng trên các ứng dụng mới và sẽ cài đặt các view đăng kí và đăng nhập cũng như các route cho toàn bộ việc xác thực đầu cuối. Một `HomeController` cũng sẽ được sinh ra, phục vụ các request post-login tới ứng dụng. Tuy nhiên, bạn có thể tự do tùy chỉnh hoặc xóa controller này dựa trên sự cần thiết trong ứng dụng của bạn.

<a name="included-views"></a>
### Views

Như đã đề cập ở phần trên, command `php artisan make:auth` cũng sẽ tạo toàn bộ các view cần thiết cho việc xác thực và đặt chúng trong thư mục `resources/views/auth`.

Command `make:auth` cũng tạo một thư mục `resources/views/layouts` bao gồm các layout cơ bản cho ứng dụng. Toàn bộ những view này sử dụng framework Bootstrap, nhưng bạn tự do tùy chỉnh nếu bạn thích.

<a name="included-authenticating"></a>
### Authenticating

Bây giờ bạn có các route và view chuẩn bị cho các authentication controllers, bạn đã sẵn sàng để đăng kí và xác nhận những user mới cho ứng dụng. Bạn chỉ đơn giản truy cập ứng dụng thông qua trình duyệt. Các authentication controller đã sẵn sàng gồm các logic (thông qua trait của chúng) để xác nhận những user đã tồn tại và lưu những user mới vào database.

#### Tùy chỉnh đường dẫn

Khi một user được xác nhận thành công, họ sẽ được chuyển sang URI '/'. Bạn có thể tùy biến địa chỉ chuyển hướng post-authentication bằng cách định nghĩa thuộc tính `redirectTo` trong `AuthController`:

    protected $redirectTo = '/home';

Khi một user không được xác nhận thành công, họ sẽ tự động chuyển hướng quay lại form đăng nhập.

#### Tùy chỉnh Guard

Bạn cũng có thể tùy biến "guard" cái mà sử dụng để xác thực user. Để bắt đầu, định nghĩa một thuộc tính `guard` trong `AuthController`. Giá trị của thuộc tính này nên tương ứng với một trong những guard đã được cấu hình trong file `auth.php`.

    protected $guard = 'admin';

#### Tùy biến Validation / Storage

Để thay đổi các trường trong form được yêu cầu khi người dùng đăng kí với ứng dụng của bạn, hoặc tùy biến các bản ghi user mới được chèn vào database như thế nào, bạn có thể chỉnh sửa class `AuthController`. Class này chịu trách nhiệm việc valite và tạo user mới của ứng dụng.

Phương thức `validator` của `AuthController` bao gồm các luật validate cho user mới của ứng dụng. Bạn hoàn toàn tự do tùy chỉnh các phương thức này nếu bạn muốn.

Phương thức `create` của `AuthController` chịu trách nhiệm cho việc tạo bản ghi mới `App\User` trong database sử dụng [Eloquent ORM](/docs/{{version}}/eloquent). Bạn tự do chỉnh sửa những phương thức này cho phù hợp database.

<a name="retrieving-the-authenticated-user"></a>
### Truy xuất người dùng đã được xác thực

Bạn có thể truy cập người dùng đã được xác thực thông qua facade `Auth`:

    $user = Auth::user();

Ngoài ra, mội khi user đã được xác thực, bạn có thể truy cập thông qua môt instance `Illuminate\Http\Request`. Hãy nhớ, các class gợi ý sẵn sẽ tự động được thêm vào trong các phương thức của controller:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user()) {
                // $request->user() returns an instance of the authenticated user...
            }
        }
    }

#### Kiểm tra việc xác thực của User

Để xác định user đã đăng nhập vào ứng dụng của bạn hay chưa, bạn có thể sử dụng phương thức `check` trên face `Auth`, cái mà sẽ trả về `true` nếu user đã được xác thực:

    if (Auth::check()) {
        // The user is logged in...
    }

Tuy nhiên, bạn có thể sử dụng middleware để kiểm tra user đã được xác thực trước khi cho phép user truy cập vào các route / controller nhất định. Để tìm hiểu nhiều hơn về việc này, hãy xem qua tài liệu tại [protecting routes](/docs/{{version}}/authentication#protecting-routes). 

<a name="protecting-routes"></a>
### Bảo vệ các Route

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để cho phép chỉ những user đã được xác thực truy cập vào các route đã cho. Laravel mang tới middleware `auth`, cái mà được định nghĩa trong `app\Http\Middleware\Authenticate.php`. Toàn bộ những gì bạn cần là đính kèm middleware vào định nghĩa (khai báo) của route.

    // Using A Route Closure...

    Route::get('profile', ['middleware' => 'auth', function() {
        // Only authenticated users may enter...
    }]);

    // Using A Controller...

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'ProfileController@show'
    ]);

Tất nhiên, nếu bạn đang sử dụng [controller classes](/docs/{{version}}/controllers), bạn có thể gọi phương thức `middleware` từ contructor của controller thay vì đính kèm nó trong khai báo trực tiếp trong route:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Chỉ định một Guard

When attaching the `auth` middleware to a route, you may also specify which guard should be used to perform the authentication:
Khi đính kèm middleware `auth` vào một route, bạn cũng có thể chỉ định guard nào sẽ được dùng để thực thi việc xác thực:

    Route::get('profile', [
        'middleware' => 'auth:api',
        'uses' => 'ProfileController@show'
    ]);

The guard specified should correspond to one of the keys in the `guards` array of your `auth.php` configuration file.
Guard được chỉ định nên tương ứng với một trong các key trong mang `guards` của file cấu hình `auth.php`.

<a name="authentication-throttling"></a>
### Authentication Throttling

Nếu bạn đang sử dụng lớp `AuthController` được tích hợp trong Laravel, `Illuminate\Foundation'Auth\ThrottlesLogins` trait có thể được dùng để điều chỉnh các nỗ lực đăng nhập vào ứng dụng của bạn. Mặc định, người dùng sẽ không thể đăng nhập trong 1 phút nếu họ thất bại trong việc cung cấp thông tin chính xác một vài lần. Việc điều phối (throttling) này là duy nhất với một username / e-mail và địa chỉ IP của họ:

    <?php

    namespace App\Http\Controllers\Auth;

    use App\User;
    use Validator;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

    class AuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;

        // Rest of AuthController class...
    }

<a name="authenticating-users"></a>
## Xác thực người dùng thủ công

Tất nhiên, bạn không bắt buộc phải sử dụng các authentication controller trong Laravel. Nếu bạn lựa chọn xóa những controller này, bạn sẽ cần phải quản lí việc xác thực user bằng cách sử dụng các class Laravel xác thực trực tiếp. Đừng lo lắng, nó là chắc chắn rồi!

Chúng ta sẽ truy cập vào các Laravel's authentication services thông qua [facade](/docs/{{version}}/facades) `Auth`, vì vậy chúng ta cần đảm bảo import facade `Auth` tại đầu class. Tiếp theo, hãy kiểm tra phương thức `attempt`:

    <?php

    namespace App\Http\Controllers;

    use Auth;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

Phương thức `attempt` chấp nhận một mảng các cặp key / value như là tham số đầu tiên. Các giá trị trong mảng sẽ được dùng để tìm user trong database. Vì vậy trong ví dụ trên, user sẽ được lấy ra bởi giá trị của cột `email`. Nếu tìm thấy user, hashed password được lưu trong database sẽ được dùng để so sánh với giá trị hashed `password` mà được truyền vào phương thức thông qua mảng. Nếu 2 hashed passowrd trùng hợp, một session sẽ được bắt đầu cho user.

Phương thức `attemp` sẽ trả về `true` nếu xác thực thành công. Ngược lại là false.

Phương thức `intended` trên redirector sẽ chuyển hướng user tới URL họ vừa cố gắn truy cập trước khi bị bắt bởi authentication filter. Một fallback URI có thể được cho trước vào phương thức này trong trường hợp đích đến dự kiến không có.

#### Specifying Additional Conditions

Nếu muốn, bạn cũng có thể thêm những điều kiện mở rộng vào truy vấn xác thực. Ví dụ, chúng ta có thể xác nhận xem user đã được đánh dấu như "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> **Ghi chú:** Trong những ví dụ này, `email` là không bắt buộc, nó chỉ được sử dụng như là một ví dụ. Bạn nên sử dụng những tên cột khác tương ứng với "username" trong database.

#### Accessing Specific Guard Instances

You may specify which guard instance you would like to utilize using the `guard` method on the `Auth` facade. This allows you to manage authentication for separate parts of your application using entirely separate authenticatable models or user tables.
Bạn có thể chỉ định các guard instance bạn thích để làm việc bằng cách dùng phương thức `guard` trên facade `Auth`. Điều này cho phép bạn quản lí việc xác thực cho những phần khác nhau của ứng dụng bằng cách sử dụng toàn bộ các model tách biệt có thể xác thực hoặc các table user.

The guard name passed to the `guard` method should correspond to one of the guards configured in your `auth.php` configuration file:
Tên của guard truyền vào phương thức `guard` nên tương ứng với một trong các guard được cấu hình trong file `auth.php`:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Logging Out
#### Đăng xuất

To log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:
Để đăng xuất người dùng khỏi ứng dụng của bạn, bạn có thể sử dụng phương thức `logout` trên facade `Auth`. Việc này sẽ xóa toàn bộ thông tin xác thực trong session của user:

    Auth::logout();

<a name="remembering-users"></a>
### Remembering Users
### Ghi nhớ người dùng

If you would like to provide "remember me" functionality in your application, you may pass a boolean value as the second argument to the `attempt` method, which will keep the user authenticated indefinitely, or until they manually logout. Of course, your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.
Nếu bạn muốn cung cấp chức năng "remember me" trong ứng dụng, bạn có thể truyền một giá trị boolean như tham số thứ 2 vào phương thức `attempt`, cái mà sẽ giữ cho người dùng đã được xác thực vô thời hạn, hoặc tới khi họ đăng xuất thủ công. Tất nhiên, table `users` phải có một cột tring `remember_token`, cái mà sẽ được dùng để lưu token "remember me".

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

If you are "remembering" users, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:
Nếu bạn "remembering" người dùng, bạn có thể dùng phương thức `viaRemember` để xác định nếu user đã được xác thực bằng cách dùng cookie "remember me":

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Other Authentication Methods
### Các phương thức xác thực khác

#### Authenticate A User Instance

If you need to log an existing user instance into your application, you may call the `login` method with the user instance. The given object must be an implementation of the `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts). Of course, the `App\User` model included with Laravel already implements this interface:
Nếu bạn cần đăng nhập một user instance đã tồn tại vào ứng dụng, bạn có thể gọi phương thức `login` với user instance. Đối tượng đã cho phải là một imlementation của `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts). Tất nhiên, model `App\User` của Laravel đã implement interface này rồi:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

Of course, you may specify the guard instance you would like to use:
Tất nhiên, bạn có thể chỉ định guard instance bạn muốn sử dụng:

    Auth::guard('admin')->login($user);

#### Authenticate A User By ID
### Xác thực người dùng bằng ID

To log a user into the application by their ID, you may use the `loginUsingId` method. This method simply accepts the primary key of the user you wish to authenticate:
Để đăng nhập một user vào ứng dụng bằng ID của họ, bạn có thể sử dụng phương thức `loginUsingId`. Phương thức này chấp nhận primary key của của user bạn muốn để xác thực:

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### Authenticate A User Once
#### Xác thực người dùng một lần duy nhất

You may use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized, which may be helpful when building a stateless API. The `once` method has the same signature as the `attempt` method:
Bạn có thể sử dụng phương thức `once` để đăng nhập một user vào ứng dụng cho một single request. Không có session hay cookie được tạo ra, cái có thể hữu ích khi xây dựng stateless API (khác với statefull API, stateless API không lưu trạng thái của từng người dùng truy cập vào ứng dụng). Phương thức `once` có cách dùng tương tự như phương thức `attempt`:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](http://en.wikipedia.org/wiki/Basic_access_authentication) provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` [middleware](/docs/{{version}}/middleware) to your route. The `auth.basic` middleware is included with the Laravel framework, so you do not need to define it:
[HTTP Basic Authentication](http://en.wikipedia.org/wiki/Basic_access_authentication) cung cấp một cách nhanh chóng để xác thực người dùng của ứng dụng của bạn mà không cần phải thiết lập một trang "login" tách biệt. Để bắt đầu, đính kèm `auth.basic` [middleware](/docs/{{version}}/middleware) vào route của bạn. Middleware `auth.basic` được bao gồm trong Laravel framework, vì vậy bạn không cần phải định nghĩa nó:

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // Only authenticated users may enter...
    }]);

Once the middleware has been attached to the route, you will automatically be prompted for credentials when accessing the route in your browser. By default, the `auth.basic` middleware will use the `email` column on the user record as the "username".
Một khi middleware đã được đính kèm vào route, bạn sẽ tự động được nhắc nhở về các thông tin khi truy cập vào route trên trình duyệt. Mặc định, middleware `auth.basic` sẽ dùng cột `email` trên các bản ghi user như là "username".

#### A Note On FastCGI
#### Một lưu ý về FastCGI

If you are using PHP FastCGI, HTTP Basic authentication may not work correctly out of the box. The following lines should be added to your `.htaccess` file:
Nếu bạn đang sử dụng PHP FastCGI, HTTP Basic authentication có thể không hoạt động chính xác. Những dòng sau nên được thêm vào trong file `.htaccess` của bạn:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, [define a middleware](/docs/{{version}}/middleware) that calls the `onceBasic` method. If no response is returned by the `onceBasic` method, the request may be passed further into the application:
Bạn cũng có thể sử dụng HTTP Basic Authentication mà không cần thiết lập một cookie định danh người dùng trong session, cái mà là một thành phần hữu ích cho API authentication. Việc tiếp theo, [Định nghĩa một middleware](/docs/{{version}}/middleware) cái mà gọi phương thức `onceBasic`. Nếu không có response nào được trả về bởi phương thức `onceBasic`, request có thể được chuyển vào trong ứng dụng:

    <?php

    namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

Next, [register the route middleware](/docs/{{version}}/middleware#registering-middleware) and attach it to a route:
Tiếp theo [Đăng kí route middleware](/docs/{{version}}/middleware#registering-middleware) và đính kèm nó vào một route:

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // Only authenticated users may enter...
    }]);

<a name="resetting-passwords"></a>
## Reset Mật Khẩu

<a name="resetting-database"></a>
### Database Considerations
### Những chú ý về database

Most web applications provide a way for users to reset their forgotten passwords. Rather than forcing you to re-implement this on each application, Laravel provides convenient methods for sending password reminders and performing password resets.
Hầu hết các ứng dụng web cung cấp một cách cho người dung thiết lập lại mật khẩu đã quên của mình. Hơn là việc tập trung bạn phải re-implement điều này trên mỗi ứng dụng, Laravel cung cấp các phương thức thuận tiện cho việc gửi các nhắc nhở về mật khẩu và thực hiện thiết lập lại nó.

To get started, verify that your `App\User` model implements the `Illuminate\Contracts\Auth\CanResetPassword` contract. Of course, the `App\User` model included with the framework already implements this interface, and uses the `Illuminate\Auth\Passwords\CanResetPassword` trait to include the methods needed to implement the interface.
Để bắt đầu, kiểm tra model `App\User` của bạn  implement `Illuminate\Contracts\Auth\CanResetPassword` contract. Tất nhiên, model `App\User` có sẵn trong framework đã implement interface này rồi, và sử dụng trait `Illuminate\Auth\Passwords\CanResetPassword` để bao hàm những phương thức cần thiết để implement interface.

#### Generating The Reset Token Table Migration
#### Tạo Reset Token Table Migration

Next, a table must be created to store the password reset tokens. The migration for this table is included with Laravel out of the box, and resides in the `database/migrations` directory. So, all you need to do is migrate:
Tiếp theo, một table phải được tạo để lưu các token password reset. Migration cho table này đã được tích hợp sẵn trong Laravel out of the box, và ở trong thư mục `database/migrations`. Vì vậy toàn bộ việc bạn cần là thực hiện migrate:

    php artisan migrate

<a name="resetting-routing"></a>
### Routing

Laravel includes an `Auth\PasswordController` that contains the logic necessary to reset user passwords. All of the routes needed to perform password resets may be generated using the `make:auth` Artisan command:
Laravel bao gồm một `Auth\PasswordController` mà gồm logic cần thiết để thiết lập lại mật khẩu của user. Toàn bộ các route cần thiết để thực hiện reset password có thể được sinh ra bằng cách sử dụng Artisan command `make:auth`:

    php artisan make:auth

<a name="resetting-views"></a>
### Views

Again, Laravel will generate all of the necessary views for password reset when the `make:auth` command is executed. These views are placed in `resources/views/auth/passwords`. You are free to customize them as needed for your application.
Một lần nữa, Laravel sẽ sinh toàn bộ các view cần thiết cho việc reset password khi command `make:auth` được thực thi. Những view này đặt tại `resources/views/auth/passwords. Bạn hoàn toàn có thể tùy biến chúng nếu cần thiết cho ứng dụng của mình.

<a name="after-resetting-passwords"></a>
### After Resetting Passwords
### Sau khi Reset Password

Once you have defined the routes and views to reset your user's passwords, you may simply access the route in your browser at `/password/reset`. The `PasswordController` included with the framework already includes the logic to send the password reset link e-mails as well as update passwords in the database.
Moojt khi bạn đã có các route và view để thiết lập lại password của user, bạn có thể đơn giản truy cập route trong trình duyệt tại `/password/reset`. `PasswordController` trong framework đã bao gồm toàn bộ logic để gửi email link reset password cũng như cập nhật password trong database.

After the password is reset, the user will automatically be logged into the application and redirected to `/home`. You can customize the post password reset redirect location by defining a `redirectTo` property on the `PasswordController`:
Sau khi password được reset, user sẽ tự động được đăng nhập vào ứng dụng và chuyển hướng tới `/home`. Bạn có thể tùy biến lại địa chỉ chuyển hướng của post reset passwrod bằng cách định nghĩa một thuộc tính `redirectTo` trong `PasswordController`:

    protected $redirectTo = '/dashboard';

> **Note:** By default, password reset tokens expire after one hour. You may change this via the password reset `expire` option in your `config/auth.php` file.
> **Ghi chú:** Mặc định, các token reset password hết hạn sau 1 giờ. Bạn có thể thay đổi điều này thông qua option `expire` trong file `config/auth.php`.

<a name="password-customization"></a>
### Customization
### Tùy biến

#### Authentication Guard Customization
#### Tùy biến Authentication Guard

In your `auth.php` configuration file, you may configure multiple "guards", which may be used to define authentication behavior for multiple user tables. You can customize the included `PasswordController` to use the guard of your choice by adding a `$guard` property to the controller:
Trong fiel cấu hình `auth.php`, bạn có thể cấu hình nhiết "guards", cái mà có thể được dùng để định nghĩa việc xác thực cho nhiều bảng user. Bạn có thể tùy biến `PasswordController` để dùng guard bạn chọn bằng cách thêm thuộc tính `$guard` vào controller:

    /**
     * The authentication guard that should be used.
     *
     * @var string
     */
    protected $guard = 'admins';

#### Password Broker Customization
#### Tùy biến Password Broker

In your `auth.php` configuration file, you may configure multiple password "brokers", which may be used to reset passwords on multiple user tables. You can customize the included `PasswordController` to use the broker of your choice by adding a `$broker` property to the controller:
Trong fiel cấu hình `auth.php`, bạn có thể cấu hình nhiều password "broker", cái mà có thể được sử dụng để reset password cho nhiều table user. Bạn có thể tùy biến `PasswordController` để sử dụng broker bạn chọn bằng cách thêm thuộc tính `$broker` vào controller:

    /**
     * The password broker that should be used.
     *
     * @var string
     */
    protected $broker = 'admins';

<a name="adding-custom-guards"></a>
## Adding Custom Guards
## Thêm các Custom Guards

You may define your own authentication guards using the `extend` method on the `Auth` facade. You should place this call to `provider` within a [service provider](/docs/{{version}}/providers):
Bạn có thể định nghĩa các authentication guard của bạn bằng cách sử dụng phương thức `extend` trên facade `Auth`. Bạn nên đặt lời gọi này tới `provider` cùng với một [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Auth;
    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Auth::extend('jwt', function($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

As you can see in the example above, the callback passed to the `extend` method should return an implementation of `Illuminate\Contracts\Auth\Guard`. This interface contains a few methods you will need to implement to define a custom guard.
Như bạn có thể thấy trong ví dụ trên, callback truyền vào phương thức `extend` trả về một implementation của `Illuminate\Contracts\Auth\Guard`. Interface này bao gồm vài phương thức bạn sẽ cần để implement để định nghĩa một custom guard.

Once your custom guard has been defined, you may use the guard in your `guards` configuration:
Một khi custom guard của bạn được định nghĩa, bạn có thể sử dụng guard trong cấu hình `guards`:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Adding Custom User Providers
## Thêm các Custom User Provider

If you are not using a traditional relational database to store your users, you will need to extend Laravel with your own authentication user provider. We will use the `provider` method on the `Auth` facade to define a custom user provider. You should place this call to `provider` within a [service provider](/docs/{{version}}/providers):
Nếu bạn đang không sử dụng các cơ sở dữ liệu quan hệ truyền thông để lưu trữ user, bạn sẽ cần phải mở rộng Laravel với authentication user provider của bạn. Chúng ta sẽ dùng phương thức `provider` trên facade `Auth` để định nghĩa một custom user provider. Bạn cần đặt lời gọi tới `provider` trong một [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Auth::provider('riak', function($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...
                return new RiakUserProvider($app['riak.connection']);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

After you have registered the provider with the `provider` method, you may switch to the new user provider in your `config/auth.php` configuration file. First, define a `provider` that uses your new driver:
Sau khi bạn đã đăng kí provider với phương thức `provider`, bạn có thể chuyển sang user provider mới trong file cấu hình `config/auth.php`. Đầu tiên, định nghĩa một `provider` mà sử dụng driver mới của bạn:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Then, you may use this provider in your `guards` configuration:
Sau đó bạn có thể sử dụng provider này trong cấu hình `guards`:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

### The User Provider Contract

The `Illuminate\Contracts\Auth\UserProvider` implementations are only responsible for fetching a `Illuminate\Contracts\Auth\Authenticatable` implementation out of a persistent storage system, such as MySQL, Riak, etc. These two interfaces allow the Laravel authentication mechanisms to continue functioning regardless of how the user data is stored or what type of class is used to represent it.
Các implementation `Illuminate\Contracts\Auth\UserProvider` chỉ chịu trách nhiệm cho việc lấy `Illuminate\Contracts\Auth\Authenticatable` implementation khỏi một persistent storage system, như là MySql, Riak, etc. 2 interface này cho phép các cơ chế Laravel authentication tiếp tục hoạt động bất kể dữ liệu user được lưu trữ như thế nào hoặc kiểu của các lớp sử dụng để đại diện nó.

Let's take a look at the `Illuminate\Contracts\Auth\UserProvider` contract:
Hãy nhìn qua contract `Illuminate\Contracts\Auth\UserProvider`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

The `retrieveById` function typically receives a key representing the user, such as an auto-incrementing ID from a MySQL database. The `Authenticatable` implementation matching the ID should be retrieved and returned by the method.
Hàm `retrieveById` thông thường nhận một key đại diện cho user, như là một auto-incrementing ID từ MySql database. Implementation `Authenticatable` tìm kiếm ID sẽ được lấy và trả về bởi phương thức.

The `retrieveByToken` function retrieves a user by their unique `$identifier` and "remember me" `$token`, stored in a field `remember_token`. As with the previous method, the `Authenticatable` implementation should be returned.
Hàm `retrieveByToken` truy xuất một user bằng `$identifier` của họ và `$token` "remember me", được lưu trong trường `remember_token`. Giống như với phương thức trước, implementation `Authenticatable` implementation sẽ được trả về.

The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. The new token can be either a fresh token, assigned on a successful "remember me" login attempt, or a null when the user is logged out.
Hàm `updateRememberToken` cập nhật `$user` trường `remember_token` với `$token` mới. Token mới có thể là một token hoàn toàn mới, được gán bởi một đăng nhập "remember me" thành công, hoặc null khi user đăng xuất.

The `retrieveByCredentials` method receives the array of credentials passed to the `Auth::attempt` method when attempting to sign into an application. The method should then "query" the underlying persistent storage for the user matching those credentials. Typically, this method will run a query with a "where" condition on `$credentials['username']`. The method should then return an implementation of `UserInterface`. **This method should not attempt to do any password validation or authentication.**
Hàm `retrieveByCredentials` nhận mảng các credentials truyền vào phương thức `Auth:attempt` khi xảy ra đăng nhập vào ứng dụng. Phương thức sau đó "query" underlying persistent storage cho việc tìm kiếm các credentials phù hợp. Cơ bạn, phương thức này sẽ chạy 1 truy vấn với điều kiện "where" trên `$credentials['username']`. Phương thức sau đó trả về một implementation của `UserInterface`. **Phương thức này không nên cố gắng validate hay xác thực mật khẩu.**

The `validateCredentials` method should compare the given `$user` with the `$credentials` to authenticate the user. For example, this method might compare the `$user->getAuthPassword()` string to a `Hash::make` of `$credentials['password']`. This method should only validate the user's credentials and return a boolean.
Phương thức `validateCredentials` so sánh `$user` với `$credentials` để xác thực user. Ví dụ, phương thức này có thể so sánh chuỗi `$user->getAuthPassword()` tới `Hash::make`  của `$credentials['password']`. Phương thức này chỉ validate user's credentials và trả về boolean.

### The Authenticatable Contract

Now that we have explored each of the methods on the `UserProvider`, let's take a look at the `Authenticatable` contract. Remember, the provider should return implementations of this interface from the `retrieveById` and `retrieveByCredentials` methods:
Bây giờ chúng ta đã khám phá từng phương thức trong `UserProvider`, hãy xem qua `Authenticatable` contract. Nhớ rằng, provider nên trả về các implementations của interface này từ phương thức `retrieveById` và `retrieveByCredentials`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

This interface is simple. The `getAuthIdentifierName` method should return the name of the "primary key" field of the user and the `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.
Interface này là đơn giản. Phương thức `getAuthIdentifierName` trả về tên của trường "primary key" của user và `getAuthIdentifier` trả về "primary key" của user. Trong MySQL back-end, một lần nữa, điều sẽ là auto-incrementing primary key. `getAuthPassword` trả về password đã được hashed. Interface này cho phép hệ thống xác thực làm việc với bất kì lớp User nào, bất kể ORM nào hay các lớp lưu trữ trừu tượng (storage abstraction layer) nào bạn đang sử dụng. Mặc định, Laravel bao gồm một class `User` trong thư mục `app` cái mà implement interface này, vì vậy bạn có thể tham khảo class này như một ví dụ.

<a name="events"></a>
## Events

Laravel raises a variety of [events](/docs/{{version}}/events) during the authentication process. You may attach listeners to these events in your `EventServiceProvider`:
Laravel xây dựng một loạt [events](/docs/{{version}}/events) khác nhau trong khi xử lí xác thực. Bạn có thể đính kèm các listener vào những event này trong `EventServiceProvider` của bạn:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
    ];
