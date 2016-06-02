# Session

- [Giới thiệu](#introduction)
- [Cách dùng cơ bản](#basic-usage)
    - [Flash Data](#flash-data)
- [Thêm một Session Drivers riêng](#adding-custom-session-drivers)

<a name="introduction"></a>
## Giới thiệu

Hệ thống HTTP không có chỗ lưu trữ, thế nên sessions cung cấp cho ta một cách để lưu trữ thông tin các yêu cầu từ người sử dụng. Laravel cung cấp đầy đủ hệ thống thống nhất thông qua API để hỗ trợ việc này. Hỗ trợ các back-ends nổi tiếng như [Memcached](http://memcached.org), [Redis](http://redis.io), và cơ sở dữ liệu đã được bao gồm sẵn trong gói.

<a name="configuration"></a> 
### Cấu hình

Thông tin cấu hình của sessions được chứa tại `config/session.php`. Hãy chắc rằng bạn nắm rõ tất cả các thông tin cấu hình của session trước khi chỉnh sửa lại tập tin này. Theo mặc định, Laravel sẽ cấu hình sử dụng `file` cho session driver, nó sẽ hoạt động tốt trên mọi ứng dụng. Đối với các ứng dụng chạy thực tế, bạn có thể sử dụng `memcached` hoặc `redis` drivers để cho hiệu suất sử dụng đạt cao hơn.

Các session `driver` được định nghĩa là nơi lưu trữ và truy suất dữ liệu session thông qua các yêu cầu. Laravel đã tích hợp sẵn một số session driver sau:

<div class="content-list" markdown="1">
- `file` - sessions sẽ chứa tại `storage/framework/sessions`.
- `cookie` - sessions sẽ lưu có bảo mật, mã hóa bởi cookies.
- `database` - sessions sẽ lưu trong CSDL được dùng trong ứng dụng của bạn.
- `memcached` / `redis` - session sẽ lưu và truy suất nhanh hơn, dựa trên cache.
- `array` - sessions sẽ được lưu trong mảng PHP thông thường khá đơn giản và tồn tại rất lâu.
</div>

> **Lưu ý:** Với array driver chỉ nên sử dụng khi chạy [tests](/docs/{{version}}/testing) để có các dữ liệu tồn tại trong thời gian dài.

### Điều kiện tiên quyểt của Driver

#### Cơ sở dữ liệu

Để sử dụng `database` session driver, bạn phải thiết lập bảng chứa các dữ liệu session trong cơ sở dữ liệu. Bên dưới là một ví dụ `Schema` dùng tạo bảng:

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->integer('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

Bạn có thể sử dụng lệnh `session:table` trong Artisan command để tạo tự động Migration này!

    php artisan session:table

    composer dump-autoload

    php artisan migrate

#### Redis

Trước khi dùng Redis sessions cho Laravel, Bạn cần phải cài đặt gói `predis/predis` package (~1.0) thông qua Composer.

### Những cân nhắc sử dụng Session

Laravel framework dùng `flash` session trong nội bộ, thế nên bạn không nên đặt tên của session trùng với tên đó.

Nếu bạn muốn tất cả các dữ liệu Session của bạn được mã hóa, hãy thiết lặp `encrypt` với giá trị là `true`.

<a name="basic-usage"></a>
## Các dùng cơ bản

#### Truy cập vào Session

First, let's access the session. We can access the session instance via the HTTP request, which can be type-hinted on a controller method. Remember, controller method dependencies are injected via the Laravel [service container](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function showProfile(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

When you retrieve a value from the session, you may also pass a default value as the second argument to the `get` method. This default value will be returned if the specified key does not exist in the session. If you pass a `Closure` as the default value to the `get` method, the `Closure` will be executed and its result returned:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function() {
        return 'default';
    });

If you would like to retrieve all data from the session, you may use the `all` method:

    $data = $request->session()->all();

You may also use the global `session` PHP function to retrieve and store data in the session:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');

        // Store a piece of data in the session...
        session(['key' => 'value']);
    });

#### Determining If An Item Exists In The Session

The `has` method may be used to check if an item exists in the session. This method will return `true` if the item exists:

    if ($request->session()->has('users')) {
        //
    }

#### Storing Data In The Session

Once you have access to the session instance, you may call a variety of functions to interact with the underlying data. For example, the `put` method stores a new piece of data in the session:

    $request->session()->put('key', 'value');

#### Pushing To Array Session Values

The `push` method may be used to push a new value onto a session value that is an array. For example, if the `user.teams` key contains an array of team names, you may push a new value onto the array like so:

    $request->session()->push('user.teams', 'developers');

#### Retrieving And Deleting An Item

The `pull` method will retrieve and delete an item from the session:

    $value = $request->session()->pull('key', 'default');

#### Deleting Items From The Session

The `forget` method will remove a piece of data from the session. If you would like to remove all data from the session, you may use the `flush` method:

    $request->session()->forget('key');

    $request->session()->flush();

#### Regenerating The Session ID

If you need to regenerate the session ID, you may use the `regenerate` method:

    $request->session()->regenerate();

<a name="flash-data"></a>
### Flash Data

Sometimes you may wish to store items in the session only for the next request. You may do so using the `flash` method. Data stored in the session using this method will only be available during the subsequent HTTP request, and then will be deleted. Flash data is primarily useful for short-lived status messages:

    $request->session()->flash('status', 'Task was successful!');

If you need to keep your flash data around for even more requests, you may use the `reflash` method, which will keep all of the flash data around for an additional request. If you only need to keep specific flash data around, you may use the `keep` method:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="adding-custom-session-drivers"></a>
## Adding Custom Session Drivers

To add additional drivers to Laravel's session back-end, you may use the `extend` method on the `Session` [facade](/docs/{{version}}/facades). You can call the `extend` method from the `boot` method of a [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Session;
    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionStore;
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

Note that your custom session driver should implement the `SessionHandlerInterface`. This interface contains just a few simple methods we need to implement. A stubbed MongoDB implementation looks something like this:

    <?php

    namespace App\Extensions;

    class MongoHandler implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

Since these methods are not as readily understandable as the cache `StoreInterface`, let's quickly cover what each of the methods do:

<div class="content-list" markdown="1">
- The `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will almost never need to put anything in this method. You can leave it as an empty stub. It is simply a fact of poor interface design (which we'll discuss later) that PHP requires us to implement this method.
- The `close` method, like the `open` method, can also usually be disregarded. For most drivers, it is not needed.
- The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
- The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB, Dynamo, etc.
- The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
- The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.
</div>

Once the session driver has been registered, you may use the `mongo` driver in your `config/session.php` configuration file.
