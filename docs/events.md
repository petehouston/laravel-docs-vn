# Events

- [Giới thiệu](#introduction)
- [Đắng kí Events / Listeners](#registering-events-and-listeners)
- [Tạo Events](#defining-events)
- [Tạo Listeners](#defining-listeners)
    - [Queued Event Listeners](#queued-event-listeners)
- [Firing Events](#firing-events)
- [Broadcasting Events](#broadcasting-events)
    - [Cấu hình](#broadcast-configuration)
    - [Đánh dấu Events cho Broadcast](#marking-events-for-broadcast)
    - [Broadcast Data](#broadcast-data)
    - [Tuỳ chỉnh Event Broadcasting](#event-broadcasting-customizations)
    - [Sử dụng Event Broadcasts](#consuming-event-broadcasts)
- [Event Subscribers](#event-subscribers)

<a name="introduction"></a>
## Giới thiệu

Events của Laravel cung cấp một triển khai observer đơn giản, cho phép bạn subscribe và listen tới các events trong ứng dụng. Các event class về cơ bản được lưu trong thư mục `app/Events`, còn các listener lại được lưu trong `app/Listeners`.

<a name="registering-events-and-listeners"></a>
## Đăng kí Events / Listeners

`EventServiceProvider` đi kèm trong Laravel cung cấp một vị trí tiện ích cho việc đăng kí tất cả các event listener. Thuộc tính `listen` chứa một mảng tất cả các events (khoá) và listeners của chúng (values). Dĩ nhiên, bạn có thể thêm vào bao nhiêu events tuỳ ý trong mảng này nếu như ứng dụng yêu cầu. Ví dụ, hãy cùng thêm vào event `PodcastWasPurchased`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\PodcastWasPurchased' => [
            'App\Listeners\EmailPurchaseConfirmation',
        ],
    ];

### Tạo class cho event và listener

Việc tạo file thủ công cho từng event và listener khá là vướng víu. Vì thế, bạn có thể thêm event với listener vào trong `EventServiceProvider` và sử dụng câu lệnh `event:generate` để tạo các file cho event và listener cho những event, và listener được khai vào trong `EventServiceProvider`. Những event và listener nào đã tồn tại trước đó rồi thì sẽ được để nguyên:

    php artisan event:generate

### Đăng kí event

Về cơ bản, event cần được đăng kí vào trong `EventServiceProvider` trong mảng `$listen`; tuy nhiên, bạn cũng có thể tự đăng kí với event dispatcher bằng cách sử dụng `Event` facade hoặc contract `Illuminate\Contracts\Events\Dispatcher`:

    /**
     * Register any other events for your application.
     *
     * @param  \Illuminate\Contracts\Events\Dispatcher  $events
     * @return void
     */
    public function boot(DispatcherContract $events)
    {
        parent::boot($events);

        $events->listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### Wildcard Event Listeners

Bạn có thể đăng kí các listener sử dụng dấu wildcard `*`, cho phép bạn bắt nhiều event trong cùng một listener. Wildcard listener nhận toàn bộ mảng dữ liueej trong một đối số truyền vào:

    $events->listen('event.*', function (array $data) {
        //
    });

<a name="defining-events"></a>
## Tạo Events

Một event class đơn giản chỉ là một data container chứa thông tin liên quan tới event. Ví dụ, giả dụ chúng ta có tạo ra event `PodcastWasPurchased` và nhận vào một [Eloquent ORM](/docs/{{version}}/eloquent):

    <?php

    namespace App\Events;

    use App\Podcast;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;

    class PodcastWasPurchased extends Event
    {
        use SerializesModels;

        public $podcast;

        /**
         * Create a new event instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    }

Như bạn thấy, event class này không có chứa logic nào. Nó đơn giản chỉ là một container cho đối tượng `Podcast` được mua. Trait `SerializeModels` sử dụng bởi event sẽ thực hiện serialize bất cứ Eloquent model nào nếu như đối tượng event được serialize sử dụng hàm `serialize` của PHP.

<a name="defining-listeners"></a>
## Tạo Listeners

Tiếp đến, hãy cùng nhau xem listener cho ví dụ về event ở trên. Event listener nhận một instance event trong hàm `handle`. Câu lệnh `event:generate` sẽ tự động import vào các event class cần thiết và type-hint event trong hàm `handle`. Bên trong hàm `handle`, bạn có thể thực hiện bất cứ logic xử lý cần thiết tương ứng cho event.

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;

    class EmailPurchaseConfirmation
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  PodcastWasPurchased  $event
         * @return void
         */
        public function handle(PodcastWasPurchased $event)
        {
            // Access the podcast using $event->podcast...
        }
    }

Event listener cũng có thể được type-hint các dependency cần thiết trong hàm khởi tạo. Tất cả các event listener được resolve trên Laravel [service container](/docs/{{version}}/container), vì thế các dependency sẽ được inject vào tự động:

    use Illuminate\Contracts\Mail\Mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

#### Ngừng việc chuyển tiếp event

Thi thoảng, bạn muốn dừng chuyển tiếp event tới các listener khác. Bạn có thể thực hiện bằng cách trả về `false` trong hàm `handle` của listener.

<a name="queued-event-listeners"></a>
### Queued Event Listeners

Bạn muốn [queue](/docs/{{version}}/queues) một event listener? Không còn gì đơn giản hơn. Đơn giản chỉ cần thêm vào `ShouldQueue` interface vào trong listener class. Listener được tạo bởi câu lệnh `event:generate` đã kèm sẵn interface import vào trong namespace, vì thế bạn chỉ cần sử dụng ngay và luôn:

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation implements ShouldQueue
    {
        //
    }

Chỉ thế thôi. Lúc này, khi mà listener này được gọi cho một event, nó sẽ tự động được queue bởi event dispatcher sử dụng [hệ thống queue](/docs/{{version}}/queues) của Laravel. Nếu không có exception nào bị bắn ra khi listener được xử lý bởi queue, thì queued job sẽ tự động được xoá sau khi được xử lý.

#### Tự truy xuất vào trong queue

Nếu bạn cần truy xuất vào queue qua hai hàm `delete` và `release`, bạn có thể thêm vào trait `Illuminate\Queue\InteractsWithQueue` đã được import sẵn, và bạn có thể sử dụng hai hàm này:

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(PodcastWasPurchased $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="firing-events"></a>
## Firing Events

Để bắn event, bạn có thể sử dụng `Event` [facade](/docs/{{version}}/facades), bằng cách truyền vào một instance của event vào trong hàm `fire`. Hàm `fire` sẽ dispatch event tới tất cả các listener đã được đăng kí:

    <?php

    namespace App\Http\Controllers;

    use Event;
    use App\Podcast;
    use App\Events\PodcastWasPurchased;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $userId
         * @param  int  $podcastId
         * @return Response
         */
        public function purchasePodcast($userId, $podcastId)
        {
            $podcast = Podcast::findOrFail($podcastId);

            // Purchase podcast logic...

            Event::fire(new PodcastWasPurchased($podcast));
        }
    }

Ngoài ra, bạn có thể sử dụng hàm helper `event` để bắn event:

    event(new PodcastWasPurchased($podcast));

<a name="broadcasting-events"></a>
## Broadcasting Events

Trong nhiều ứng dụng web hiện đại ngày nay, web socket được sử dụng để tạo cập nhật động và real-time trên UI. Khi có dữ liệu cập nhật trên server, một bản tin được gửi qua socket và được xử lý bởi client.

Để hỗ trọ bạn tạo ứng dụng kiểu này, Laravel làm cho việc đó đơn giản để "broadcast" các event qua một kết nối websocket. Broadcast event cho phép bạn chia sẻ cùng tên event giữa code phía server và code Javascript phía client.

<a name="broadcast-configuration"></a>
### Cấu hình

Cấu hình để broadcast event được lưu trong `config/broadcasting.php`. Laravel hỗ trợ một số broadcast driver như [Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis), và `log` driver cho môi trường phát triển và debug. Cấu hình ví dụ cho mỗi driver này đều kèm sẵn trong mỗi ứng dụng Laravel.

#### Yêu cầu cho broadcast

Các dependency sau cần thiết cho sử dụng event broadcasting:

- Pusher: `pusher/pusher-php-server ~2.0`
- Redis: `predis/predis ~1.0`

#### Yêu cầu về queue

Trước khi broadcast event, bạn cũng cần cấu hình và chạy một [queue listener](/docs/{{version}}/queues). Tất cả các event broadcasting được thực hiện thông qua queued jobs vì thế thời gian response của ứng dụng không bị ảnh hưởng lớn lắm.

<a name="marking-events-for-broadcast"></a>
### Đánh dấu event cho Broadcast

Để cho Laravel biết nếu một event cần được broadcase, triển khai interface `Illuminate\Contracts\Broadcasting\ShouldBroadcast` trong event class. Interface này yêu cầu bạn triển khai một hàm là `broadcastOn`, hàm này trả về một mảng tên "các channel" mà event cần được broadcast tới:

    <?php

    namespace App\Events;

    use App\User;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated extends Event implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should be broadcast on.
         *
         * @return array
         */
        public function broadcastOn()
        {
            return ['user.'.$this->user->id];
        }
    }

Sau đó, bạn chỉ cần [bắn event](#firing-events) như thông thường. Khi mà event đã được bắn ra, một [queued job](/docs/{{version}}/queues) sẽ tự động broadcast event qua broadcast driver đã cấu hình.

<a name="broadcast-data"></a>
### Broadcast Data

Khi một event được broadcase, tất cả các thuộc tính `public` được tự động serialize và broadcast cùng event payload, điều này cho phép bạn lấy bất cứ dữ liệu public nào từ Javascript. Vì thế, ví dụ như, nếu event có một thuộc tính public là `$user` có chứa thông tin về Eloquent model tương ứng, thì payload của broadcast sẽ kiểu thế này:

    {
        "user": {
            "id": 1,
            "name": "Jonathan Banks"
            ...
        }
    }

Tuy nhiên, nếu bạn muốn có quyền kiểm soát tốt hơn với payload này, bạn có thể thêm vào hàm `broadcastWith` trong event. Hàm này sẽ trả về một mảng dữ liệu mà bạn muốn gửi đi cùng event:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['user' => $this->user->id];
    }

<a name="event-broadcasting-customizations"></a>
### Tuỳ chỉnh event broadcasting

#### Thay đổi tên event

Mặc định, tên của broadcast event sẽ là tên class đầy đủ của event. Vì thế nếu như tên class là `App\Events\ServerCreated`, thì tên của broadcast event sẽ là `App\Events\ServerCreated`. Bạn cần thay đổi tên của event bằng cách khai báo trong hàm `broadcastAs` trong event class:

    /**
     * Get the broadcast event name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'app.server-created';
    }

#### Thay đổi queue

Mặc định, mỗi event được broadcase được đặt vào trong queue mặc định khai báo trong `queue.php`. Bạn có thể tuỳ chọn thay đổi queue bằng cách thêm vào hàm `onQueue` trong event class. Hàm này sẽ trả về tên của queue mà bạn muốn sử dụng:

     /**
     * Set the name of the queue the event should be placed on.
     *
     * @return string
     */
    public function onQueue()
    {
        return 'your-queue-name';
    }

<a name="consuming-event-broadcasts"></a>
### Nhận và sử dụng Event Broadcasts

#### Pusher

Bạn có thể nhận event broadcast sử dụng [Pusher](https://pusher.com) driver thông qua Pusher Javascript SDK. Ví dụ, hãy bắt event `App\Events\ServerCreated` ở ví dụ trước:

    this.pusher = new Pusher('pusher-key');

    this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

    this.pusherChannel.bind('App\\Events\\ServerCreated', function(message) {
        console.log(message.user);
    });

#### Redis

Nếu bạn sử dụng Redis broadcaster, bạn sẽ cần viết consumer riêng để nhận và gửi thông qua công nghệ websocket mà bạn muốn. Ví dụ bạn có thể sử dụng thư viện nổi tiếng [Socket.io](http://socket.io) được viết bằng Node.

Sử dụng thư viện `socket.io` và `ioredis`, bạn có thể nhanh chóng tạo một event broadcaster tới tất cả các events được Laravel broadcast ra:

    var app = require('http').createServer(handler);
    var io = require('socket.io')(app);

    var Redis = require('ioredis');
    var redis = new Redis();

    app.listen(6001, function() {
        console.log('Server is running!');
    });

    function handler(req, res) {
        res.writeHead(200);
        res.end('');
    }

    io.on('connection', function(socket) {
        //
    });

    redis.psubscribe('*', function(err, count) {
        //
    });

    redis.on('pmessage', function(subscribed, channel, message) {
        message = JSON.parse(message);
        io.emit(channel + ':' + message.event, message.data);
    });

<a name="event-subscribers"></a>
## Event Subscribers

Event subscriber là class mà bạn có thể dùng để đăng kí nhiều event bên trong class, và bạn có thể tạo ra các event handler khác nhau chỉ trong một class. Subscriber cần khai báo một hàm `subscribe`, mà sẽ được truyền vào trong event dispatcher:

    <?php

    namespace App\Listeners;

    class UserEventListener
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'App\Events\UserLoggedIn',
                'App\Listeners\UserEventListener@onUserLogin'
            );

            $events->listen(
                'App\Events\UserLoggedOut',
                'App\Listeners\UserEventListener@onUserLogout'
            );
        }

    }

#### Đăng kí một Event Subscriber

Khi mà subscriber được tạo, nó sẽ được đăng kí với event dispatcher. Bạn có thể đăng kí subscriber sử dụng thuộc tính `$subscribe` trong `EventServiceProvider`. Ví dụ, hãy thêm vào `UserEventListener`.

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventListener',
        ];
    }
