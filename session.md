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

Trước tiên, để truy cập session. Bạn cần phải truy cập ứng dụng thông qua các yêu cầu HTTP, đó là các phương thức Controller mà bạn muốn hướng đến. Hãy nhớ rằng, Các phương thức controller phụ thuộc vào Laravel [service container](/docs/{{version}}/container):

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

Khi này bạn có thể nhận từ giá trị từ session, Bạn có thể thiết lập giá trị mặc định qua tham số thứ hai của phương thức `get`. Điều này có nghĩa là nếu như session này không có giá trị thì sẽ trả về giá trị mặc định. Nếu bạn truyền vào một `Closure`, thì `Closure` sẽ được thực thi và trả về giá trị `return`:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function() {
        return 'default';
    });

Nếu bạn muốn nhận tất cả các giá trong session, bạn hãy sử dụng phương thức `all`:

    $data = $request->session()->all();

Bạn cũng có thể sử dụng hàm `session` để truy suất hoặc lưu các session:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');

        // Store a piece of data in the session...
        session(['key' => 'value']);
    });

#### Kiểm tra sự tồn tại của một Session

Phương thức `has` cho phép bạn kiểm tra sự tồn tại của một session. Phương thức này sẽ trả về `true` nếu session đấy tồn tại:

    if ($request->session()->has('users')) {
        //
    }

#### Lưu giá trị và Session

Một trong những cách khác, bạn có thể gọi phương thức `put` từ phương thức `session` đươc trỏ thông qua thuộc tính `request`. Ví dụ, dùng phương thức `put` để lưu giá trị mới vào session:

    $request->session()->put('key', 'value');

#### Đẩy giá trị vào mảng Session

Với phương thức `push` bạn có thể đẩy một giá trị mới vào một biến mảng Session. Ví dụ, trong `user.teams` là một mảng chứa các tên nhóm, bạn có thể đẩy tên nhóm mới vào trong mảng theo cách như sau:

    $request->session()->push('user.teams', 'developers');

#### Truy xuất và xóa dữ liệu

Dùng phương thức `pull` dùng để lấy và xóa ngay session:

    $value = $request->session()->pull('key', 'default');

#### Xóa bỏ khỏi bộ nhớ Session

Phương thức `forget` sẽ xóa dữ liệu trong session. Mặc khác nếu bạn muốn xóa toàn bộ Session, bạn chỉ cần dùng phương thức `flush`:

    $request->session()->forget('key');

    $request->session()->flush();

#### Khởi tạo Session ID

Nếu bạn muốn tạo ra một session ID, hãy dùng phương thức `regenerate`:

    $request->session()->regenerate();

<a name="flash-data"></a>
### Flash Data

Đôi khi có một vài dữ liệu mà bạn chỉ muốn nó lưu tại lần truy suất tiếp theo và sau đó xóa đi thì phương thức `flash` có thể giúp bạn. Dữ liệu sẽ được lưu lại và chỉ suất hiện một lần duy nhất trong lần phản hồi yêu cần tiếp theo, sau đó nó sẽ tự động xóa đi. Flash data thường dùng để biểu thị các trạng thái, thông báo, lời nhắn:

    $request->session()->flash('status', 'Task was successful!');

Nếu bạn muốn giữ dư liệu trong nhiều yêu cầu, bạn hãy sử dụng phương thức `reflash`, nó sẽ giữ lại các dữ liệu được thêm vào sau đó. Nếu bạn muốn giữ các nội dung `flash` cụ thể, thì lúc này hãy dùng phương thức `keep`:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="adding-custom-session-drivers"></a>
## Thêm một Session Drivers riêng

Để thêm một Laravel's session back-end driver, bạn hãy dùng phương thức `extend` từ `Session` [facade](/docs/{{version}}/facades). Bạn cần gọi phương thức `extend` từ phương thức `boot` thông qua một [service provider](/docs/{{version}}/providers):

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

Chú ý Session Driver của bạn phải có cấu trúc kế thừa giao diện `SessionHandlerInterface`. Và đây là một mẫu ví dụ về kế thừa từ mẫu giao diện. Một driver sử dụng MongoDB cho Session:

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

Những phương thức này không hẳn khó hiểu và nó giống như `StoreInterface`, hãy tìm hiểu sơ về các phương thức nào:

<div class="content-list" markdown="1">
- Phương thức `open` thường sẽ được sử dụng trong các tập tin. Và bạn hầu như không cần dùng phương thức này (ta có thể tìm hiểu sau) .
- Phương thức `close` , giống như phương thức `open`, nó cũng thường được bỏ qua. Hầu hết các Driver đều không cần sử dugj nó.
- Phương thức `read`  trả về chuỗi các dữ liệu phiên liên quan đến kiểm soát của `$sessionId`. Bạn cần serialization hay các cách mã hóa khác khi lưu và nhận giá trị session, với Laravel thì sẽ dùng serialization.
- Phương thức `write` là ghi các giá trị chuỗi `$data` được kiểm soát `$sessionId`, một số hệ thống ghi liên tục trong bộ nhớ, như MongoDB, Dynamo, khác nữa.
- Phương thức `destroy` xóa các giá trị session với `$sessionId` từ bộ nhớ.
- Phương thức `gc` Xóa các session cũng mà tồn tại lâu hơn `$lifetime` hoặc ó giá trị mới, Bằng mộ đoạn UNIX timestamp. Đối với hệ thống tự hết hạn như Memcached và Redis, phương thức này có thể được bỏ trống.
</div>

Một khi sessions driver được đăng ký, bạn có thể sử dụng `mongo` driver ở trong cấu cấu hình `config/session.php`.
