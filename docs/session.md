# Session

- [Session](#session)
  - [Giới thiệu](#gii-thiu)
    - [Cấu hình](#cu-hinh)
    - [Điều kiện tiên quyểt của Driver](#iu-kin-tien-quyt-ca-driver)
    - [Những cân nhắc sử dụng Session](#nhng-can-nhc-s-dng-session)
  - [Các dùng cơ bản](#cac-dong-c-bn)
    - [Nhận dữ liệu](#nhn-d-liu)
    - [Lưu dữ liệu](#lu-d-liu)
    - [Flash Data](#flash-data)
    - [Xóa bỏ khỏi bộ nhớ Session](#xoa-b-khi-b-nh-session)
    - [Khởi tạo Session ID](#khi-to-session-id)
  - [Thêm một Session Drivers riêng](#them-mt-session-drivers-rieng)
    - [Implementing The Driver](#implementing-the-driver)
    - [Đăng ký The Driver](#ng-k-the-driver)

## Giới thiệu

Hệ thống HTTP không có chỗ lưu trữ, thế nên sessions cung cấp cho ta một cách để lưu trữ thông tin các yêu cầu từ người sử dụng. Laravel cung cấp đầy đủ hệ thống thống nhất thông qua API để hỗ trợ việc này. Hỗ trợ các back-ends nổi tiếng như [Memcached](http://memcached.org), [Redis](http://redis.io), và cơ sở dữ liệu đã được bao gồm sẵn trong gói.

### Cấu hình

Thông tin cấu hình của sessions được chứa tại `config/session.php`. Hãy chắc rằng bạn nắm rõ tất cả các thông tin cấu hình của session trước khi chỉnh sửa lại tập tin này. Theo mặc định, Laravel sẽ cấu hình sử dụng `file` cho session driver, nó sẽ hoạt động tốt trên mọi ứng dụng. Đối với các ứng dụng chạy thực tế, bạn có thể sử dụng `memcached` hoặc `redis` drivers để cho hiệu suất sử dụng đạt cao hơn.

Các session `driver` được định nghĩa là nơi lưu trữ và truy suất dữ liệu session thông qua các yêu cầu. Laravel đã tích hợp sẵn một số session driver sau:

- `file` - sessions sẽ chứa tại `storage/framework/sessions`.
- `cookie` - sessions sẽ lưu có bảo mật, mã hóa bởi cookies.
- `database` - sessions sẽ lưu trong CSDL được dùng trong ứng dụng của bạn.
- `memcached` / `redis` - session sẽ lưu và truy suất nhanh hơn, dựa trên cache.
- `array` - sessions sẽ được lưu trong mảng PHP thông thường khá đơn giản và tồn tại rất lâu.

> **Lưu ý:** Với array driver chỉ nên sử dụng khi chạy [tests](testing.md) để có các dữ liệu tồn tại trong thời gian dài.

### Điều kiện tiên quyểt của Driver

#### Cơ sở dữ liệu

Để sử dụng `database` session driver, bạn phải thiết lập bảng chứa các dữ liệu session trong cơ sở dữ liệu. Bên dưới là một ví dụ `Schema` dùng tạo bảng:

```php
Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->integer('user_id')->nullable();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity');
});
```

Bạn có thể sử dụng lệnh `session:table` trong Artisan command để tạo tự động Migration này!

```php
php artisan session:table

composer dump-autoload

php artisan migrate
```

#### Redis

Trước khi dùng Redis sessions cho Laravel, Bạn cần phải cài đặt gói `predis/predis` package (~1.0) thông qua Composer. Bạn cấu hình Redis của bạn kết nối trong file cấu hình `database`. Trong file cấu hình  `session`, thuộc tính `connection` có thể được sử dụng để xác định kết nối với Redis là sử dụng session.

### Những cân nhắc sử dụng Session

Laravel framework dùng `flash` session trong nội bộ, thế nên bạn không nên đặt tên của session trùng với tên đó.

Nếu bạn muốn tất cả các dữ liệu Session của bạn được mã hóa, hãy thiết lặp `encrypt` với giá trị là `true`.

## Các dùng cơ bản

### Nhận dữ liệu

#### Truy cập vào Session

Có hai cách chính để làm việc với dữ liệu session data trong Laravel: phương thức global `session` và qua thể hiện `Request`. Đầu tiên, Chúng ta nhìn cách truy cập session qua thể hiện `Request`, có thể được type-hinted trong phương thức controller. Hãy nhớ rằng, Các phương thức controller phụ thuộc vào Laravel [service container](container.md):

```php
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
```

Khi này bạn có thể nhận từ giá trị từ session, Bạn có thể thiết lập giá trị mặc định qua tham số thứ hai của phương thức `get`. Điều này có nghĩa là nếu như session này không có giá trị thì sẽ trả về giá trị mặc định. Nếu bạn truyền vào một `Closure`, thì `Closure` sẽ được thực thi và trả về giá trị `return`:

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function() {
    return 'default';
});
```

Nếu bạn muốn nhận tất cả các giá trong session, bạn hãy sử dụng phương thức `all`:

```php
$data = $request->session()->all();
```

Bạn cũng có thể sử dụng hàm `session` để truy suất hoặc lưu các session:

```php
Route::get('home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

#### Phương thức Global Session

Bạn cũng có thể sử dụng hàm global ``session`` của PHP và lưu dữ liệu trong session. Khi hàm `session` được gọi, chuỗi tham số, nó sẽ trà về giá trị của key session. Khi hàm được gọi với một cặp giá trị key / value, giá trị sẽ lưu trong session:

```php
Route::get('home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

>Có rất ít sự khác biệt giữa sử dụng session qua HTTP request và sử dụng hàm global  session. Cả hai phương thức là [testable](testing.md) qua phương thức assertSessionHas nó tồn tại trong tất cả các test cases của bạn.

#### Kiểm tra sự tồn tại của một Session

Phương thức `has` cho phép bạn kiểm tra sự tồn tại của một session. Phương thức này sẽ trả về `true` nếu session đấy tồn tại:

```php
if ($request->session()->has('users')) {
    //
}
```

### Lưu dữ liệu

Để lưu dữ liệu trong session, bạn sẽ thường sử dụng phương thức ``put`` hoặc hàmsession:

```php
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global helper...
session(['key' => 'value']);
```

#### Đẩy giá trị vào mảng Session

Với phương thức `push` bạn có thể đẩy một giá trị mới vào một biến mảng Session. Ví dụ, trong `user.teams` là một mảng chứa các tên nhóm, bạn có thể đẩy tên nhóm mới vào trong mảng theo cách như sau:

```php
$request->session()->push('user.teams', 'developers');
```

#### Truy xuất và xóa dữ liệu

Dùng phương thức `pull` dùng để lấy và xóa ngay session:

```php
$value = $request->session()->pull('key', 'default');
```

### Flash Data

Đôi khi có một vài dữ liệu mà bạn chỉ muốn nó lưu tại lần truy suất tiếp theo và sau đó xóa đi thì phương thức `flash` có thể giúp bạn. Dữ liệu sẽ được lưu lại và chỉ suất hiện một lần duy nhất trong lần phản hồi yêu cần tiếp theo, sau đó nó sẽ tự động xóa đi. Flash data thường dùng để biểu thị các trạng thái, thông báo, lời nhắn:

```php
$request->session()->flash('status', 'Task was successful!');
```

Nếu bạn muốn giữ dư liệu trong nhiều yêu cầu, bạn hãy sử dụng phương thức `reflash`, nó sẽ giữ lại các dữ liệu được thêm vào sau đó. Nếu bạn muốn giữ các nội dung `flash` cụ thể, thì lúc này hãy dùng phương thức `keep`:

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

### Xóa bỏ khỏi bộ nhớ Session

Phương thức `forget` sẽ xóa dữ liệu trong session. Mặc khác nếu bạn muốn xóa toàn bộ Session, bạn chỉ cần dùng phương thức `flush`:

```php
$request->session()->forget('key');

$request->session()->flush();
```

### Khởi tạo Session ID

Khởi tạo the session ID thường gặp khi ngăn một mã độc từ người dùng khai thác một session fixation tấn công ứng dụng của bạn.

Laravel tự động Khởi tạo Session ID trong khi xác thực nếu bạn sử dụng built-in ``LoginController``; tuy nhiên, nếu bạn cần tự tay Khởi tạo Session ID, bạn có thể sử dụng phương thức ``regenerate``.

```php
$request->session()->regenerate();
```

## Thêm một Session Drivers riêng

### Implementing The Driver

Tùy biến session driver của bạn nên thực hiện trong SessionHandlerInterface. Nó chứa một vài phương thức chúng ta cần để thực thi. Một stubbed MongoDB thực hiện giống như bên dưới:

```php
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
?>
```

>Laravel không cung cấp đường dẫn chứa extensions của bạn. bạn có thể thoải mái đặt ở đâu bạn thích. Trong ví dụ trên, chúng ta sẽ tạo một đường dẫn ``Extensions`` chứa  ``MongoHandler``.

Những phương thức này không hẳn khó hiểu và nó giống như `StoreInterface`, hãy tìm hiểu sơ về các phương thức nào:

- Phương thức `open` thường sẽ được sử dụng trong các tập tin. Và bạn hầu như không cần dùng phương thức này (ta có thể tìm hiểu sau) .
- Phương thức `close` , giống như phương thức `open`, nó cũng thường được bỏ qua. Hầu hết các Driver đều không cần sử dụng nó.
- Phương thức `read`  trả về chuỗi các dữ liệu phiên liên quan đến kiểm soát của `$sessionId`. Ở đây không phải làm bất kỳ serialization hoặc encoding khi nhận hoặc lưu dữ liệu session trong driver, Laravel sẽ làm serialization cho bạn.
- Phương thức `write` là ghi các giá trị chuỗi `$data` được kiểm soát với `$sessionId` đến một vài hệ thống persistent storage như MongoDB, Dynamo, etc. Một lần nữa, bạn không cần phải serialization - Laravel đã xử lý việc đó cho bạn.
- Phương thức `destroy` xóa các giá trị session với `$sessionId` từ bộ nhớ.
- Phương thức `gc` Xóa các session cũng mà tồn tại lâu hơn `$lifetime` bằng doạn UNIX timestamp. Đối với hệ thống tự hết hạn như Memcached và Redis, phương thức này có thể được bỏ trống.

### Đăng ký The Driver

Khi driver của bạn được thực hiện, bạn đã sẵn sàng đăng ký nó với framework. Bổ sung thêm drivers vào session backend của Laravel, bạn có thể sử dụng phương thức ``extend`` trong ``Session`` facade. Bạn nên gọi phương thức ``extend`` từ phương thức ``boot`` của [service provider](providers.md). Bạn có thể làm điều này từ ``AppServiceProvider`` hoặc tạo mới một provider:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Support\Facades\Session;
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
        Session::extend('mongo', function ($app) {
            // Return implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
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
```

Một khi sessions driver được đăng ký, bạn có thể sử dụng `mongo` driver ở trong cấu cấu hình `config/session.php`.
