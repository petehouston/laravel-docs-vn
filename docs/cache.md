# Cache

- [Cấu hình](#configuration)
- [Sử dụng cache](#cache-usage)
    - [Lấy một đối tượng cache](#obtaining-a-cache-instance)
    - [Lấy các items lưu trong cache](#retrieving-items-from-the-cache)
    - [Lưu các items vào trong cache](#storing-items-in-the-cache)
    - [Xoá các items ra khỏi cache](#removing-items-from-the-cache)
- [Cache Tags](#cache-tags)
    - [Lưu trữ các cache items được tag](#storing-tagged-cache-items)
    - [Truy cập các cache items được tag](#accessing-tagged-cache-items)
- [Thêm một cache driver tự chọn](#adding-custom-cache-drivers)
- [Events](#events)

<a name="configuration"></a>
## Cấu hình

Laravel cung cấp một API thống nhất cho các hệ thống cache khác nhau. Cấu hình cho cache được đặt trong file `config/cache.php`. Trong file này bạn có thể chỉ định cache driver nào bạn muốn sử dụng mặc định trong ứng dụng. Laravel hỗ trợ sẵn các hệ thông cache phía backends phổ biến như [Memcached](http://memcached.org) và [Redis](http://redis.io).

File cấu hình cache cũng chứa các tuỳ chọn khác, đều được ghi chú đầy đủ bên trong, vì thế hãy nhớ đọc kĩ chỉ dẫn trong đó. Mặc định, Laravel được cấu hình để sửa dụng cache driver là `file`, để lưu trữ các cache object đã được serialized trong filesystem. Với các ứng dụng lớn hơn, khuyên các bạn sử dụng in-memory cache như Memcached hay APC. Bạn thậm chí có thẻ cấu hình để sử dụng nhiều cấu hình cache cho cùng một driver.

### Yêu cầu cho cache

#### Cơ sở dữ liệu

Khi sử dụng cache driver `database`, bạn sẽ cần thiết lập một bảng để lưu trữ các cache items. Bạn sẽ thấy một ví dụ về khai báo `Schema` cho bảng dưới đây:

    Schema::create('cache', function($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

Bạn cũng có thể sử dụng câu lệnh Artisan `php artisan cache:table` để tạo ra một migration với schema chuẩn xác.

#### Memcached

Sử dụng Memcached yêu cầu [thư viện Memcached PECL](http://pecl.php.net/package/memcached) phải được cài đặt.

[Cấu hình](#configuration) mặc định sử dụng TCP_IP dựa trên [Memcached::addServer](http://php.net/manual/en/memcached.addserver.php):

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

Bạn cũng có thể thiết lập thông số `host` tới một đường dẫn tới UNIX socket. Nếu làm thế này thì `port` cần được set về `0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

Trước khi sử dụng Redis cache với Laravel, bạn cần phải cài đặt thư viện Composer `predis/predis` (~1.0).

Thông tin thêm chi tiết về cấu hình cho Redis, hãy tham khảo [mục cấu hình cho Redis](/docs/{{version}}/redis#configuration)

<a name="cache-usage"></a>
## Sử dụng cache

<a name="obtaining-a-cache-instance"></a>
### Lấy một đối tượng cache

Hai [contracts](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Factory` và `Illuminate\Contracts\Cache\Repository` cung cấp truy xuất tới Laravel cache services. Contract `Factory` cung cấp tuy cập tới tất cả các cache drivers được khai báo cho ứng dụng. Contract `Repository` về cơ bản là triển khai của cache driver mặc định cho ứng dụng mà bạn chỉ định trong file cấu hình `cache`.

Tuy nhiên, bạn cũng có thể sử dụng `Cache` facade, mà bạn sẽ thấy trong suốt tài liệu này. `Cache` facade cung cấp cách truy xuất tiện và ngắn gọn tới các phần triển khai của các Laravel contracts.

Ví dụ, cùng nhau thêm vào `Cache` facade vào trong controller:

    <?php

    namespace App\Http\Controllers;

    use Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### Truy xuất tới nhiều cache stores

Sử dụng `Cache` facade cho phép bạn có thể truy xuất tới nhiều cache store thông quan hàm `store`. Giá trị khoá truyền vào hàm `store` cần ứng với một trong những store danh sách các `stores` của mảng cấu hình trong file cấu hình `cache`:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### Lấy các items trong cache

Hàm `get` trong `Cache` facade được sử dụng để lấy các items trong cache. Nếu như item không tồn tại trong cache, giá trị `null` sẽ được trả về. Nếu muốn, bạn có thể truyền vào tham số thứ hai để hàm `get` chỉ định giá trị mặc định nếu như item không tồn tại:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');


Bạn thậm chí có thể truyền vào một `Closure` như một giá trị mặc định. Kết quả của `Closure` sẽ được trả về nếu item cần lấy không tồn tại trong cache. Truyền vào một Closure cho phép bạn trì hoãn lại việc lấy giá trị mặc định từ trong một database hay từ một dịch vụ bên ngoài:

    $value = Cache::get('key', function() {
        return DB::table(...)->get();
    });

#### Kiểm tra item có tồn tại hay không

Hàm `has` có thể được sử dụng để kiểm tra xem một item có tồn tại trong cache hay không:

    if (Cache::has('key')) {
        //
    }

#### Tăng / Giảm giá trị

Hai hàm `increment` và `decrement` có thể được sử dụng để điều chỉnh giá trị của các items số nguyên nằm trong cache. Cả hai hàm này có tuỳ chọn cho phép tham số thứ hai chỉ định giá trị tăng giảm bao nhiêu cho cache item.

    Cache::increment('key');

    Cache::increment('key', $amount);

    Cache::decrement('key');

    Cache::decrement('key', $amount);

#### Lấy ra hay cập nhật

Đôi lúc bạn muốn lấy ra một item trong cache, nhưng cũng muốn lưu giá trị mặc định cho item nếu như không tồn tại. Ví dụ, bạn muốn lấy tất cả users nằm trong cache hoặc, nếu chúng không tồn tại, thì sẽ lấy từ database và thêm vào trong cache. Bạn có thể thực hiện việc này bằng cách sử dụng hàm `Cache::remember`:

    $value = Cache::remember('users', $minutes, function() {
        return DB::table('users')->get();
    });

Nếu item không tồn tại trong cache, thì `Closure` truyền vào trong hàm `remember` sẽ được thực thi và kết quả sẽ được lưu lại vào trong cache.

Bạn cũng có thể kết hợp hai hàm `remember` và `forever` với nhau:

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### Lấy ra và xoá

Nếu bạn muốn lấy ra một item trong cache và xoá đi, bạn có thể sử dụng hàm `pull`. Giống như hàm `get` `null` sẽ được trả về nếu item không tồn tại trong cache từ trước đó:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### Lưu trữ items vào trong cache

Bạn có thể sử dụng hàm `put` của `Cache` facade để lưu items vào trong cache. Khi bạn thêm một item vào trong cache, bạn sẽ cần phải chỉ rõ số phút mà giá trị sẽ được lưu:

    Cache::put('key', 'value', $minutes);

Thay vì truyền vào số phút cho tới khi item bị hết hạn, bạn cũng có thể truyền vào một đối tượng PHP kiểu `DateTime` để thể hiện thời gian hết hạn của item được cache:

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

Hàm `add` sẽ chỉ thêm item vào cache nếu như nó chưa tồn tại sẵn. Hàm sẽ trả về `true` nếu item thực sự được thêm vào cache. Còn ngược lại thì hàm sẽ trả về `false`:

    Cache::add('key', 'value', $minutes);

Hàm `forever` có thể được sử dụng để lưu một item trong cache vĩnh viễn. Những giá trị này cần phải được gỡ ra một cách thủ công sử dụng hàm `forget`:

    Cache::forever('key', 'value');

<a name="removing-items-from-the-cache"></a>
### Xoá các items ra khỏi cache

Bạn có thể xoá items khỏi cache sử dụng hàm `forget` trong `Cache` facade:

    Cache::forget('key');

Bạn có thể xoá toàn bộ cache sử dụng hàm `flush`:

    Cache::flush();

Việc xoá toàn bộ cache **không hề** tuân theo tiền tố cache nào, mà sẽ thực hiện xoá toàn bộ tất cả trong cache. Vì thế hãy thực sự cẩn trọng khi xoá một giá trị cache mà được sử dụng chung giữa các ứng dụng.

<a name="cache-tags"></a>
## Cache Tags

> **Lưu ý:** Cache tag không hỗ trợ khi sử dụng cache driver là `file` và `database`. Thêm nữa, khi sử dụng nhiều tag với cache được lưu dưới dạng "forever", hiệu năng sẽ đạt tốt nhất ở driver có khả năng tự động xoá các danh sách đã lưu quá lâu như `memcached`.

<a name="storing-tagged-cache-items"></a>
### Lưu các items đã được tag

Cache tag cho phép bạn tag các item liên quan tới nhau trong cache và có thể xoá hết các giá trị cache mà có chung một tag. Bạn có thể truy xuất vào một cache được tag bằng cách truyền vào một mảng các tên tag:

	Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

	Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

Tuy nhiên, bạn không hề bị hạn chế khi sử dụng hàm `put`. Bạn có thể sử dụng bất cứ hàm lưu trữ cache nào khi làm việc với tag.

<a name="accessing-tagged-cache-items"></a>
### Truy xuất vào cache item được tag

Để lấy một cache item được tag, truyền vào danh sách tương tự các tag trong hàm `tags`:

	$john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

Bạn có thể xoá tất cả các item có chung một tag hoặc danh sách các tag. Ví dụ, mã lệnh sau sẽ xoá tất cả các cache được tag với giá trị là `people`, `authors`, hoặc cả hai. Vì thế, cả hai khoá là `Anne` và `John` sẽ bị xoá khỏi cache:

	Cache::tags(['people', 'authors'])->flush();

Ngược lại, câu lệnh này sẽ chỉ xoá các cache có tag là `authors`, vì thế `Anne` sẽ bị xoá, còn `John` thì không.

	Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## Thêm một cache driver tự chọn

Để mở rộng Laravel với một cache driver tuỳ chọn, chúng ta sẽ sử dụng hàm `extend` trong `Cache` facade, để có thể thực hiện liên kết tới phần quản lý. Về cơ bản, việc này sẽ thông qua một [service provider](/docs/{{version}}/providers).

Ví dụ, để đăng kí một cache driver mới là "mongo":

    <?php

    namespace App\Providers;

    use Cache;
    use App\Extensions\MongoStore;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function($app) {
                return Cache::repository(new MongoStore);
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

Tham số thứ nhất truyền vào hàm `extend` là tên của driver. Cái này tương ứng với thông số `driver` trong file cấu hình `config/cache.php`. Tham số thứ hai là một Closure mà sẽ trả về một đối tượng kiểu `Illuminate\Cache\Repository`. Closure này sẽ được truyền vào đối tượng `$app`, chính là [service container](/docs/{{version}}/container).

Việc gọi tới `Cache::extend` có thể được gọi trong hàm `boot` của `App\Providers\AppServiceProvider` được đi kèm trong một ứng dụng Laravel mới cài đặt, hoặc bạn có thể tạo service provider riêng để quản lý việc mở rộng - đừng quên việc đăng kí provider vào trong danh sách providers nằm trong file cấu hình `config/app.php`.

Để tạo một cache driver riêng, trước tiên chúng ta cần phải triển khai `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts). Do đó, việc triển khải MongoDB cache sẽ kiểu như thế này:

    <?php

    namespace App\Extensions;

    class MongoStore implements \Illuminate\Contracts\Cache\Store
    {
        public function get($key) {}
        public function put($key, $value, $minutes) {}
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Chúng ta chỉ cần triển khai cho mỗi hàm này sử dụng một connection tới MongoDB. Khi mà triển khai hoàn thiện, chúng ta sẽ hoàn thiện việc đăng kí cache driver:

    Cache::extend('mongo', function($app) {
        return Cache::repository(new MongoStore);
    });

Khi mà việc mở rộng hoàn thiện, chỉ cần cập nhất tên giá trị `driver` trong file cấu hình `config/cache.php`.

Nếu bạn băn khoăn việc đặt code cho cache driver riêng ở đâu, hãy nghĩ tới việc tạo thành thư viện trên Packagist! Hoặc là, bạn có thể tạo một namespace `Extensions` trong thư mục `app`. Tuy nhiên, nên nhớ là Laravel không có cứng nhắc trong cấu trúc ứng dụng và bạn hoàn toàn thoải mái trong việc quản lý ứng dụng theo cách bạn muốn.

<a name="events"></a>
## Events

Để thực thi code mỗi khi có một thao tác làm việc với cache xảy ra, bạn có thể lắng nghe các [events](/docs/{{version}}/events) từ cache. Về cơ bản, bạn nên đặt những phần này vào trong `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
