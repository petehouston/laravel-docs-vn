# Views

- [Cách dùng cơ bản](#basic-usage)
    - [Truyền dữ liệu vào Views](#passing-data-to-views)
    - [Chia sẽ dữ liệu vào tất cả các Views](#sharing-data-with-all-views)
- [View Composers](#view-composers)

<a name="basic-usage"></a>
## Cách dùng cơ bản

Views chứa nội dung HTML phục vụ cho ứng dụng của bạn và tách ra riêng biệt từ bộ điều kiển controller / application. Các views được chứa tại thư mục `resources/views`.

Ví dụ đơn giản của  Views sẽ như thế này:

    <!-- View chứa tại resources/views/greeting.php -->

    <html>
        <body>
            <h1>Xin chào, <?php echo $name; ?></h1>
        </body>
    </html>

Nội dung này sẽ chứa tại `resources/views/greeting.php`, Chúng ta sẽ trả dữ liệu về hàm `view` như sau:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Như bạn đã thất, với tham số đầu tiên của hàm `view` đó là tên của tập tin view trong thư mục `resources/views`. Tham số thứ hai chính là một biến hoặc mảng với các dữ liệu cần dùng cho view. Trong trường hợp này, chúng ta đã truyền vào một biến `name`, nó sẽ hiển thị trong view bằng các thực hiện lệnh `echo` lên biến.

Tất nhiên, các views có thể chứa trong các thư mục con ở trong thư mục `resources/views`. Dấu "Chấm" sẽ ngăn cách cách thư mục con. Ví dụ như sau, nếu như view của bạn đặt tại `resources/views/admin/profile.php`, thì khi này nó sẽ trông như thế này:

    return view('admin.profile', $data);

#### Xác định một view tồn tại

Nếu bạn muốn xác đinh một view có tồn tại hay không, bạn có thể sử dụng phương thức `exists` được gọi từ `view` không có tham số. Với phương thức này sẽ trả về `true` nếu view này tồn tại:

    if (view()->exists('emails.customer')) {
        //
    }

Khi hàm `view` được gọi và không có tham số, thì nó chính là thể hiện của `Illuminate\Contracts\View\Factory`, điều này cho phép ta truy cập tất cả các phương thức của đối tượng factory.

<a name="view-data"></a>
### Dữ liệu của View

<a name="passing-data-to-views"></a>
#### Truyền dữ liệu vào Views

Như ở ví dụ trước, bạn có thể truyền vào một mảng giá trị vào views:

    return view('greetings', ['name' => 'Victoria']);

Khi truyền dữ liệu bằng cách này, `$data` sẽ thành một mảng có khóa/giá trị tương ứng. Bên trong view, bạn có thể sử dụng các giá trị bằng cách gọi biến với tên là khóa của mảng, ví dụ như `<?php echo $key; ?>`. Một các khác có thể truyền dữ liệu vào view `view`, bạn sử dụng phương thức `with` để truyền dữ liệu đến view:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Chia sẽ dữ liệu vào tất cả các Views

Thỉnh thoảng, bạn cần dùng một số dữ liệu nhất định với tất cả các views trong ứng dụng của bạn. Bạn có thể dùng phương thức `share` của view factory. Thông thông thường, bạn cần phải gọi phương thức `share` trong phương thức `boot` từ một service provider. Bạn có thể thêm chúng vào trong `AppServiceProvider` hoặc tự tạo ra một Service Provider khác:

    <?php

    namespace App\Providers;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->share('key', 'value');
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## View Composers

View composers sẽ gọi trực tiếp một phương thức hoặc một lớp khi một view được render (dịch). Nếu bạn có một dữ liệu mà bạn cần ràng buộc nó lại tại thời điểm view của bạn được render, một view composer sẽ giúp bạn xử lý sắp xếp các tư duy logic trong view đó.

Nào chúng ta hãy đăng ký một view composer cùng với [service provider](/docs/{{version}}/providers). Chúng ta sẽ sử dụng hàm `view` để truy cập vào mẫu `Illuminate\Contracts\View\Factory`. Hãy nhớ rằng, Laravel sẽ không có thư mục mặc định cho view composers. Bạn có quyền tạo chúng bất cứ ở đâu mà bạn muốn. Ví dụ, Bạn có thể tạo thư mục `App\Http\ViewComposers` chẳng hạn:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // sử dụng class based composers...
            view()->composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // dùng Closure based composers...
            view()->composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Hãy nhớ rằng, nếu bạn tạo ra một Service Provider chứa các đăng ký view composer, bạn cần thêm nó vào bên trong mảng `providers` chứa tại tập tin cấu hình `config/app.php`.

Bây giờ chúng ta đã đăng ký thành công một view composer, Phương thức `ProfileComposer@compose` sẽ thực thi tại thời điểm mà view `profile` được rendered. Vì vậy, nào chúng ta hãy định nghĩa lớp đó:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Tự động giải quyết từ service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Như vậy trước khi view đó được rendered, phương thức `compose` sẽ gọi `Illuminate\View\View` qua `$view`. Bạn có thể dụng phương thức `with` để ràng buộc dữ liệu đến view.

> **Chú ý:**  Tất các các view composers được xử lý thông qua [service container](/docs/{{version}}/container), vì vậy bạn có thể thêm các phụ thuộc vào bên trong phương thức khởi tạo contructor của view composer.

#### Đính kèm Composer vào nhiều Views

Bạn có thể đính kèm nhiều view vào view composer bằng cách truyền vào một mảng chứa tất cả cá view vào thương thức `composer`:

    view()->composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

Phương thức `composer` có thể dùng `*` để đinh kèm toàn bộ view của bạn:

    view()->composer('*', function ($view) {
        //
    });

### View Creators

View **creators** rất giống với view composers; Tuy nhiên, nó sẽ tác động ngay lập tức vào các view thay vì chờ các view cho tới khi chúng được rendered. Để đăng ký một view creator, sử dụng phương thức `creator`:

    view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
