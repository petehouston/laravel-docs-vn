# Facades

- [Giới thiệu](#introduction)
- [Sử dụng Facades](#using-facades)
- [Tham khảo các Facade](#facade-class-reference)

<a name="introduction"></a>
## Giới thiệu

Facades cung cấp một interface "tĩnh" cho các class sử dụng trong [service container](/docs/{{version}}/container) của ứng dụng. Laravel cung cấp sẵn nhiều facade, và bạn có thể đã sử dụng chúng mà không hề biết! Laravel "facade" phục vụ như "proxies tĩnh" cho các class bên dưới ở trong service container, cung cấp lợi ích của việc sử dụng cú pháp vừa ngắn gọn vừa có thể bảo trì có thể thoải mái hơn là sử dụng các phương thức tĩnh truyền thống.

<a name="using-facades"></a>
## Sử dụng Facades

Trong một ứng dụng Laravel, facade là một class cung cấp truy cập vào một đối tượng từ container. Bộ máy làm cho cách này hoạt động được là nhờ vào class `Facade`. Các facade của Laravel, và bất cứ facade mà bạn tạo ra đều được kế thừa từ class cơ sở `Illuminate\Support\Facades\Facade`.

Một class facade chỉ cần triển khai một phương thức duy nhất: `getFacadeAccessor`. Việc của `getFacadeAccessor` là định nghĩa cái gì cần được resolve từ container. Class `Facade` cơ sở tận dụng phương thức magic `__callStatic()` để điều việc thực thi từ facade tới object đã được resolve.

Trong ví dụ dưới đây, một lệnh thực thi được gọi tới hệ thống cache của Laravel. Nhìn qua đoạn code này, bạn có thể nghĩ là phương thức tĩnh `get` đang được gọi trong class `Cache`:

    <?php

    namespace App\Http\Controllers;

    use Cache;
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
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Chú ý ở gần đầu file chúng ta có thực hiện "importing" vào facade `Cache`. Facade này đóng vai trò như một proxy để truy cập vào phần triển khai phía dưới của interface `Illuminate\Contracts\Cache\Factory`. Bất cứ việc gọi nào mà chúng ta sử dụng từ facade sẽ được đẩy tới instance phía dưới của Laravel cache service.

Nếu nhìn vào class `Illuminate\Support\Facades\Cache`, bạn sẽ thấy không hề có phương thức tĩnh `get` nào cả:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

Thay vào đó, facade `Cache` mở rộng class cơ sở `Facade` và định nghĩa phương thức `getFacadeAccessor()`. Nhớ là nhiệm vụ của phương thức này là trả về tên của liên kết trong service container. Khi mà người dùng tham chiếu tới bất kì phương thức tĩnh nào trong facade `Cache`, Laravel thực hiện việc Resolve liên kết `cache` từ [service container](/docs/{{version}}/container) và thực thi phương thức được gọi (trong trường hợp này, `get`) đối với object.

<a name="facade-class-reference"></a>
## Tham khảo các Facade

Dưới đây, bạn có thể tìm các facade và lớp dưới của nó, rất hữu ích khi bạn muốn tìm hiểu nhanh tài liệu API cho một facade nào đó. Các tên khoá liên kết vào trong [service container](/docs/{{version}}/container) cũng được ghi kèm theo nếu có.

Tên Facade  |  Tên Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](http://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](http://laravel.com/api/5.1/Illuminate/Contracts/Auth/Access/Gate.html)  |
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue`
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |
Storage  |  [Illuminate\Contracts\Filesystem\Factory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/{{version}}/Illuminate/View/View.html)  |
