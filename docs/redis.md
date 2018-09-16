# Redis

- [Giới thiệu](#introduction)
- [Sử dụng cơ bản](#basic-usage)
    - [Các câu lệnh pipelining](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Giới thiệu

[Redis](http://redis.io) là một dự án store mã nguồn mở để lưu các cặp dữ liệu kiểu key-value. Nó thường được gọi là server cấu trúc dữ liệu vì các khoá có thử chứa dữ liệu kiểu [strings](http://redis.io/topics/data-types#strings), [hashes](http://redis.io/topics/data-types#hashes), [lists](http://redis.io/topics/data-types#lists), [sets](http://redis.io/topics/data-types#sets), và [sorted sets](http://redis.io/topics/data-types#sorted-sets). Trước khi sử dụng Redis với Laravel, bạn sẽ cần cài thư viện `predis/predis` (~1.0) bằng Composer.

<a name="configuration"></a>
### Cấu hình

Cấu hình cho Redis nằm tại file `config/database.php`. Trong file này, bạn sẽ thấy một mảng `redis` có chứa thông số tới Redis server:

    'redis' => [

        'cluster' => false,

        'default' => [
            'host'     => '127.0.0.1',
            'port'     => 6379,
            'database' => 0,
        ],

    ],

Cấu hình server mặc định cần cung cấp đầy dủ để phát triển. Tuy nhiên, bạn hoàn toàn thoải mái để thay đổi các giá trị này dựa theo môi trường của bạn. Đơn giản chỉ cần cung cấp tên Redis server và chỉ định host và port mà server sử dụng.

Thông số `cluster` sẽ cho Laravel Redis client biết để thực hiện phân mảnh trên các node của Redis, cho phép bạn tạo pool cho các node và tạo một lượng lớn RAM có thể sử dụng. Tuy nhiên, chú ý là việc phân mảnh phía client không xử lý khi thất bại; dó đó, cái này phù hợp với dữ liệu đã được cache có thể dùng được trong các data store chính khác.

Thêm vào đó, bạn có thể khai báo một mảng `options` trong kết nối tới Redis, cho phép bạn chỉ định một set các [thông số client](https://github.com/nrk/predis/wiki/Client-Options) của Predis.

Nếu Redis server yêu cầu đăng nhập, bạn cần phải cung cấp mật khảu bằng cách thêm thông số `password` vào trong mảng cấu hình cho Redis server.

> **Lưu ý:** Nếu bạn có Redis PHP extension cài thông qua PECL, bạn cần phải đổi tên của Redis trong `config/app.php`.

<a name="basic-usage"></a>
## Sử dụng cơ bản

Bạn có thể tương tác với Redis bằng cách gọi một số hàm trong `Redis` [facade](/docs/{{version}}/facades). `Redis` facade hỗ trợ các hàm động, nghĩa là bạn có thể gọi bất cứ [câu lệnh Redis](http://redis.io/commands) trong facade và nó sẽ được truyền trực tiếp tới Redis. Ở ví dụ này, chúng ta sẽ gọi `GET` trên Redis bằng cách gọi hàm `get` trong `Redis` facade:

    <?php

    namespace App\Http\Controllers;

    use Redis;
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
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

Dĩ nhiên, như đã nói ở trước, bạn có thể gọi bất cứ câu lệnh Redis nào trên `Redis` facade. Laravel sử dụng các hàm magic để truyền câu lệnh tới Redis server, vì thế, đơn giản chỉ cần truyền tham số mà lệnh Redis muốn:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

Một cách khác, bạn cũng có thể truyền câu lệnh tới server sử dụng hàm `command`, hàm này nhận tên trong tham số đầu tiên và một mảng các giá trị ở tham số thứ hai:

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Sử dụng nhiều Redis connection

Bạn có thể lấy một đối tượng kết nối Redis bằng cách gọi tới hàm `Redis::connection`:

    $redis = Redis::connection();

Lệnh này sẽ cho bạn đối tượng kết nối tới Redis server mặc định. Nếu bạn không sử dụng server clustering nào, bạn có thể truyền vào tên server trong hàm `connection` để lấy một server chỉ định được khai báo trong cấu hình Redis:

    $redis = Redis::connection('other');

<a name="pipelining-commands"></a>
### Các câu lệnh pipelining

Pipelining nên được sử dụng khi bạn cần gửi nhiều câu lệnh tới server trong một yêu cầu. Hàm `pipeline` nhận một tham số: một `Closure` nhận đối tượng Redis. Bạn có thể cung cấp tất cả các câu lệnh tới đối tượng Redis này và chúng sẽ được thực thi trong một yêu cầu:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel cũng cung cấp một interface tiện ích cho hai lệnh của Redis là `publish` và `subscribe`. Những câu lệnh này cho phép bạn listen bản tin trên một "channel". Bạn có thể publish bản tin tới một channel từ ứng dụng khác, hoặc thậm chí sử dụng một ngôn ngữ lập trình khác, cho phép trao đổi dễ dàng giữa các ứng dụng và processes.

Đầu tiện, hãy cùng nhau thiết lập một listener trên một channel thông qua Redis sử dụng hàm `subscribe`. Chúng ta sẽ đặt hàm gọi này bên trong [câu lệnh Artisan](/docs/{{version}}/artisan) vì việc gọi hàm `subscribe` bắt đầu một tiến trình chạy lâu:

    <?php

    namespace App\Console\Commands;

    use Redis;
    use Illuminate\Console\Command;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function($message) {
                echo $message;
            });
        }
    }

Lúc này, chúng ta có thể publish bản tin tới channel sử dụng `publish`:

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### Đăng kí subscription sử dụng wildcard

Sử dụng hàm `psubscribe`, bạn có thể subscribe tới một channel dạng wildcard, khá hữu dụng để bắt tất cả các bản tin trên mọi channel. Tên `$channel` sẽ được truyền ở tham số thứ hai trong `Closure`:

    Redis::psubscribe(['*'], function($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function($message, $channel) {
        echo $message;
    });
