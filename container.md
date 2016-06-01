# Service Container

- [Giới thiệu](#introduction)
- [Liên kết](#binding)
    - [Liên kết interfaces vào triển khai](#binding-interfaces-to-implementations)
    - [Liên kết theo ngữ cảnh](#contextual-binding)
    - [Tagging](#tagging)
- [Resolving](#resolving)
- [Events của container](#container-events)

<a name="introduction"></a>
## Giới thiệu

Laravel service container là một công cụ rất mạnh trong việc quản lý các dependencies và thực hiện xử lý dependency injection. Dependency injection là một cụm từ mỹ miều cơ bản thể hiện ý như này: các dependencies của class được "injected" vào trong class thông qua hàm khởi tạo, hoặc, trong một số trường hợp là quả các phương thức "setter".

Cùng nhau xem ví dụ đơn giản dưới đây:

    <?php

    namespace App\Jobs;

    use App\User;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Contracts\Bus\SelfHandling;

    class PurchasePodcast implements SelfHandling
    {
        /**
         * The mailer implementation.
         */
        protected $mailer;

        /**
         * Create a new instance.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function __construct(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        /**
         * Purchase a podcast.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

Trong ví dụ này, cái job `PurchasePodcase` cần thực hiện việc gửi emails khi một podcast được mua. Vì thế, chúng ta cần phải **inject** vào một service có thể thực hiện việc gửi emails. Khi mà service được inject vào rồi, thì chúng ta có thể dễ dàng thay đổi các phương pháp triển khai khác nhau. Chúng ta cũng có thể dễ dàng thực hiện "mock", hay tạo nhiều phương pháp thực thi dummy của phần gửi mail khi thực hiện kiểm thử ứng dụng.

Việc hiểu sâu về Laravel service container là một điều cơ bản để xây dựng ứng dụng mạnh mẽ và lớn hơn, và các bạn cũng có thể đóng góp vào Laravel core nữa.

<a name="binding"></a>
## Liên kết (Binding)

Hầu như tất cả việc liên kết service container được đăng kí bên trong [service providers](/docs/{{version}}/providers), vì thế tất cả ví dụ trong này đều sử dụng container trong hoàn cảnh đó. Tuy nhiên, sẽ không thực sự cần thiết là phải bind class vào trong container nếu như chúng không phụ thuộc vào interface nào cả. Container không cần thiết được chỉ định cách tạo objects như thế nào, vì nó có thể tự động tìm ra các objects "cụ thể" sử dụng reflection services của PHP.

Bên trong một service provider, bạn luôn luôn có quyền truy cập vào trong container thông qua biến `$this->app`. Chúng ta có thể đăng kí liên kết sử dụng phương thức `bind`, và truyền vào tên của class hay interface mà chúng ta muốn đăng kí cùng với `closure` thực hiện trả về instance của class đó:

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app['HttpClient']);
    });

Chú ý là chúng ta nhận được container nhưng một đối số truyền vào cho resolver. Sau đó thì chúng ta có thể thực hiện resolve các dependencies con của đối tượng mà đang được xây dựng.

#### Liên kết một singleton

Phương thức `singleton` thực hiện liên kết một class hay interface vào container mà chỉ cần thực hiện duy nhất một lần, và sau đó cùng một đối tượng sẽ được trả về trong các lần gọi tiếp theo vào trong container.

    $this->app->singleton('FooBar', function ($app) {
        return new FooBar($app['SomethingElse']);
    });

#### Liên kết các instances

Bạn cũng có thể liên kết một instance đang tồn tại vào trong container sử dụng phương thức `instance`. Instance đó sẽ luôn luôn được trả về trong các lần gọi sau vào container:

    $fooBar = new FooBar(new SomethingElse);

    $this->app->instance('FooBar', $fooBar);

<a name="binding-interfaces-to-implementations"></a>
### Liên kết interfaces vào triển khai

Một điểm rất mạnh của service container đó là khả năng liên kết một interface tới một mẫu triển khải. Ví dụ, giả sử là chúng ta có một interface là `EventPusher` và có một triển khai là `RedisEventPusher`, thì chúng ta có thể đăng kí qua service container như thế này:

    $this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');

Câu lệnh đó sẽ bảo container luôn luôn inject `RedisEventPusher` khi một class nào đó cần một triển khai từ interface `EventPusher`. Lúc này, chúng ta có thể đánh dấu interface `EventPusher` vào trong một hàm khởi tạo hay bất cứ vị trí nào mà dependencies có thể được inject bởi service container:

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Liên kết theo ngữ cảnh

Đôi khi bạn sẽ có hai classes triển khai từ cùng một interface nhưng bạn muốn inject các triển khai khác nhau vào các class. Ví dụ, khi mà hệ thống nhận được một Order mới, chúng ta có muốn gửi một event thông qua [PubNub](http://www.pubnub.com/) hơn là Pusher. Laravel cung cấp một interface đơn giản và liền mạch cho việc khai báo hành vi này:

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give('App\Services\PubNubEventPusher');

Thậm chí có thể truyền Closure vào trong `give`:

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give(function () {
                      // Resolve dependency...
                  });

#### Liên kết vào các giá trị

Đôi khi bạn có một class mà nhận các giá trị inject vào không phải là class mà là kiểu giá trị ví dụ như integer. Bạn có thể dễ dàng liên kết theo ngữ cảnh để inject giá trị mà bạn cần:

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('$maxOrderCount')
              ->give(10);

<a name="tagging"></a>
### Tagging

Sẽ có lúc bạn cần tới việc resolve một nhóm liên kết. Ví dụ, bạn đang xây dụng một tập báo cáo mà sẽ nhận một mảng danh sách các triển khải khác nhau của interface `Report`. Sau khi đăng kí xong triển khai của `Report`, bạn có thể gán chúng vào một tag thông qua phương thức `tag`:

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Một khi service được tag, chúng có thể được dễ dàng resolve thông qua phương thức `tagged`:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## Resolving

Có vài cách để resolve một cái gì đó ra khỏi container. Thứ nhất, bạn có thể sử dụng `make`, phương thức này nhận tên class hay interface bạn muốn thực hiện resolve:

    $fooBar = $this->app->make('FooBar');

Thứ hai, bạn có thể truy cập vào trong container như một mảng, vì nó được triển khai từ interface `ArrayAccess` của PHP:

    $fooBar = $this->app['FooBar'];

Cuối cùng, và quan trọng nhất, bạn cần phải đánh dấu các dependency trong hàm khởi tạo của một class để thực hiện resolve bởi container, bao gồm [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [queue jobs](/docs/{{version}}/queues), [middleware](/docs/{{version}}/middleware), và còn nữa. Thực tế thì đây là cách mà hầu hết các object của bạn được resolve từ container.

Container sẽ tự động inject dependencies cho class mà nó xử lý. Ví dụ, bạn có thể đánh dấu một repository được khai báo trong ứng dụng qua hàm khởi tạo của controller. Repository này sẽ tự động được resolve và inject vào trong class:

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## Events của container

Service container bắn ra các event mỗi khi nó thực hiện resolve một object. Bạn có thể listen các event này thông qua phương thức `resolving`:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(FooBar::class, function (FooBar $fooBar, $app) {
        // Called when container resolves objects of type "FooBar"...
    });

Như bạn thấy, object đang được resolve sẽ được truyền lại vào trong callback, cho phép bạn thiết lập các thuộc tính bổ sung nào vào trong object trước khi được trả lại cho bên sử dụng nó.
