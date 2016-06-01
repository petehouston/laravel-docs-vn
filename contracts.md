# Contracts

- [Giới thiệu](#introduction)
- [Vì sao cần Contract?](#why-contracts)
- [Tham khảo về Contract](#contract-reference)
- [Sử dụng Contract như thế nào?](#how-to-use-contracts)

<a name="introduction"></a>
## Giới thiệu

Laravel Contracts là một set các interfaces khai báo các core services cung cấp bởi framework. Ví dụ, contract `Illuminate\Contracts\Queue\Queue` khai báo các phương thức cần thiết cho các job xử lý hàng đợi, còn `Illuminate\Contracts\Mail\Mailer` khai báo các phương thức cần thiết để gửi email.

Mỗi contract đều có sẵn một triển khải tương ứng bởi framework. Ví dụ, Laravel cung cấp một triển khai về hàng đợi với các drivers khác nhau, và một triển khải cho mailer được cung cấp bởi [SwiftMailer](http://swiftmailer.org/).

Tất cả các Laravel Contracts đều nằm trong [repository riêng của chúng trên Github](https://github.com/illuminate/contracts). Điều này làm cho việc tìm hiểu tất cả các Contracts có sẵn một cách tiện lợi, cũng như việc áp dụng để phát triển riêng bởi các nhà phát triển package.

### Contracts Vs. Facades

Laravel [facades](/docs/{{version}}/facades) cung cấp một cách thuận tiện để sử dụng Laravel services mà không cần tới type-hint và resolve contracts khỏi container. Tuy nhiên, sử dụng contact cũng cho phép bạn có thể khai báo các dependencies ngoài vào trong class. Trong hầu hết các ứng dụng, chỉ sử dụng facade không có vấn đề gì. Nhưng nếu bạn thực sự cần tạo ra sự liên kết lỏng lẻo mà các contract cung cấp, hãy đọc thêm ở phần dưới đây.

<a name="why-contracts"></a>
## Vì sao cần Contracts?

Có thể bạn đang đặt vài câu hỏi về Contracts. Tại sao sử dụng interfaces? Không phải là việc sử dụng interfaces làm cho mọi thứ trở nên phức tạp? Vậy hãy cùng nhau đi khai phá lý do để sử dụng interface theo hai tựa đề sau: mối liên kết lỏng lẻo và tính đơn giản.

### Mối liên kết lỏng lẻo

Đầu tiên, cùng nhau review mã nguồn bị liên kết chặt cho một triển khai cho cache. Xem đoạn code dưới đây:

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

Trong class này, mã nguồn bị liên kết khá chặt vào một phương thức triển khai cache. Nó bị liên kết chặt là do chúng ta đang phụ thuộc vào lớp Cache từ một package. Nếu API của package này thay đổi, mã nguồn của chúng ta cũng bắt buộc phải thay đổi theo.

Cũng như vậy, nếu chúng ta muốn thay đổi nền tảng cache ở phía dưới (Memcached) bằng một nền tảng khác (Redis), chúng ta lại phải thay đổi mã nguồn trong repository. Mã nguồn của repository không nên biết quá rõ về việc ai cung cấp dữ liệu và cung cấp như thế nào.

**Thay vì làm như vậy, chúng ta có thể cải thiện mã nguồn bằng cách sử dụng interface.**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

Lúc này mã nguồn không bị phụ thuộc quá nhiều vào bất cứ package nào cả, thậm chí cả Laravel. Vì contracts của package không có chứa bất kì triển khai và dependencies nào, bạn có thể dễ dàng viết một triển khai của bất kì contract nào, điều này cho phép bạn thay đổi triển khai cache mà không phải thay đổi mã nguồn của đoạn sử dụng cache nữa.

### Tính đơn giản

Khi mà tất cả Laravel services đều được khai báo trong các interface đơn giản, mọi thứ se trở nên rất dễ dàng khi tìm chức năng cung của một service. **Contract được sử dụng như một bản tài liệu cho các chức năng của framework.**

Thêm vào đó, khi bạn phụ thuộc vào các interface đơn giản, mã nguồn sẽ trở nên dễ hiểu và dễ bảo trì hơn. Thay vì theo dõi các phương thức nào có thể sử dụng trong một class lớn và phức tạp, bạn có thể tham khảo tới cấu trúc đơn giản của interface.

<a name="contract-reference"></a>
## Tham khảo về Contract

Đây là bản tham chiếu tới các Contract của Laravel cũng như các phần "facade" tương ứng:

Contract  |  Facade Tương ứng
------------- | -------------
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/master/Auth/Factory.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/master/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## Sử dụng Contract như thế nào?

Vậy thì làm thế nào để triển khai một contract? Sự thực là rất đơn giản.

Nhiều kiểu class trong Laravel được resolve qua các [service container](/docs/{{version}}/container), bao gồm controllers, event listeners, middleware, queued jobs, và thậm chí các route trong Closures. Vì thế, để lấy một triển khai của một contract, bạn chỉ cần "type-hint" interface trong hàm khởi tạo của class đang được resolve.

Xem ví dụ dưới đây về event listener:

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\NewUserRegistered;
    use Illuminate\Contracts\Redis\Database;

    class CacheUserInformation
    {
        /**
         * The Redis database implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  NewUserRegistered  $event
         * @return void
         */
        public function handle(NewUserRegistered $event)
        {
            //
        }
    }

Khi mà event listener được resolve, thì service container sẽ đọc phần type-hint trên hàm khởi tạo của class, và inject giá trị phù hợp vào. Để tìm hiểu thêm về đăng kí vào trong service container, hãy xem mục [service container](/docs/{{version}}/container).
