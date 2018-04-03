# Service Providers

- [Giới thiệu](#introduction)
- [Viết Service Providers](#writing-service-providers)
    - [Phương thức Register](#the-register-method)
    - [Phương thức Boot](#the-boot-method)
- [Đăng kí Providers](#registering-providers)
- [Deferred Providers](#deferred-providers)

<a name="introduction"></a>
## Giới thiệu

Service providers là trung tâm của việc khởi tạo tất cả các ứng dụng Laravel. Ứng dụng của bạn, cũng như các thành phần core của Laravel được khởi tạo từ service providers.

Nhưng, "bootstrapped" nghĩa là sao? Đơn giản, ý là **đăng kí**, bao gồm đăng kí các liên kết tới service container, event listeners, middleware, và thậm chí các route. Service providers là trung tâm để cấu hình ứng dụng của bạn.

Nếu bạn mở file `config/app.php` nằm trong Laravel, bạn sẽ thấy một mảng `providers`. Tất cả những service provider class này sẽ được load vào trong ứng dụng. Một điều hiển nhiên là nhiều trong số đó được gọi là "deferred" providers, nghĩa là chúng không phải được load trong mọi request, chỉ khi có service nào yêu cầu thì mới thực hiện cung cấp.

Trong phần tổng quát này, bạn sẽ học cách viết service providers riêng của bạn và đăng kí chúng với Laravel.

<a name="writing-service-providers"></a>
## Viết Service Providers

Tất cả các service providers đều kế thừa từ class `Illuminate\Support\ServiceProvider`. Class cơ bản này yêu cầu bạn khai báo ít nhất một phương thức trong provider: `register`. Bên trong hàm `register`, bạn **chỉ nên đăng kí vào trong [service container](/docs/{{version}}/container)**. Bạn đừng bao giờ cố gắng đăng kí bất kì các event listeners, routes hay bất kì chắc năng nào khác vào trong hàm `register`.

Artisan CLI có thể dễ dàng sinh ra một provider mới thông qua câu lệnh `make:provider`:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### Phương thức Register

Như đã đề cập ở trước, bên trong hàm `register`, bạn chỉ nên thực hiện đăng kí vào trong [service container](/docs/{{version}}/container). Bạn không nên bao giờ cố gắng đăng kí bất kì event listeners, routes hay bất kì các chức năng nào khác vào trong hàm `register`. Nếu không, bạn có thể vô tình sử dụng một service được cung cấp bởi một service provider mà chưa được load.

Bây giờ, hãy cùng nhau xem thử một service provider cơ bản:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

Service provider này chỉ khai báo đúng một hàm `register`, và sử dụng nó để triển khai `Riak\Connection` trong service container. Nếu bạn không hiểu cách service container thực hiện như thế nào, hãy xem [tài liệu về service container](/docs/{{version}}/container).

<a name="the-boot-method"></a>
### Phương thức Boot

Vậy nếu như chúng ta muốn đăng kí một view composer vào trong service provider thì sao? Điều này có thể thực hiện bên trong hàm `boot`. **Hàm này được gọi sau khi tất cả các service providers đã được đăng kí**, nghĩa là bạn có thể truy cập vào trong tất cả các services đã được đăng kí vào trong framework:

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        // Other Service Provider Properties...

        /**
         * Register any other events for your application.
         *
         * @param  \Illuminate\Contracts\Events\Dispatcher  $events
         * @return void
         */
        public function boot(DispatcherContract $events)
        {
            parent::boot($events);

            view()->composer('view', function () {
                //
            });
        }
    }

#### Boot Method Dependency Injection

Bạn có thể type-hint dependencies cho service provider của bạn ở hàm `boot`. [Service container](/docs/{{version}}/container) sẽ tự động inject bất cứ dependencies nào bạn cần:

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $factory)
    {
        $factory->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Đăng kí Providers

Tất cả các service provider được đăng kí bên trong file cấu hình `config/app.php`. File này chứa một mảng các `providers` danh sách tên của các service providers. Mặc định, một tập hợp các core service provider của Laravel nằm trong mảng này. Những provider này làm nhiệm vụ khởi tạo các thành phần core của Laravel, ví dụ như mailer, queue, cache, và các thành phần khác.

Để đăng kí provider, đơn giản chỉ cần thêm vào trong mảng đó:

    'providers' => [
        // Other Service Providers

        App\Providers\AppServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Deferred Providers

Nếu bạn muốn provider **chỉ** đăng kí liên kết vào trong [service container](/docs/{{version}}/container), bạn có thể chọn trì hoãn việc đăng kí cho tới khi nào cần thiết. Việc trì hoãn quá trình load một provider sẽ cải thiện performance của ứng dụng, vì nó không load từ filesystem trong mọi yêu cầu.

Để trì hoàn việc load một provider, set thuộc tính `defer` thành `true` và khai báo một hàm `provides`. Hàm này sẽ trả về liên kết tới service container mà provider này đăng kí:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    }

Laravel đóng gói và chứa một danh sách các service thực hiện bởi các deferred service provider, cùng với tên của các service provider class. Sau đó, chỉ khi nào bạn cần resolve một trong những service này thì Laravel mới thực hiện load service provider.
