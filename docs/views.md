# Views

- [Views](#views)
  - [Tạo Views](#to-views)
  - [Truyền dữ liệu vào Views](#truyn-d-liu-vao-views)
    - [Chia sẻ dữ liệu vào tất cả các Views](#chia-s-d-liu-vao-tt-c-cac-views)
  - [View Composers](#view-composers)

## Tạo Views

>Tìm kiếm thông tìn về tạo bản mẫu Blade ? xem ở mục [Blade](blade.md)

Views chứa nội dung HTML phục vụ cho ứng dụng của bạn và tách ra riêng biệt từ bộ điều kiển controller / application. Các views được chứa tại thư mục `resources/views`.

Ví dụ đơn giản của  Views sẽ như thế này:

```HTML
<!-- View chứa tại resources/views/greeting.php -->

<html>
    <body>
        <h1>Xin chào, <?php echo $name; ?></h1>
    </body>
</html>
```

Nội dung này sẽ chứa tại `resources/views/greeting.php`, Chúng ta sẽ trả dữ liệu về hàm `view` như sau:

```PHP
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

Như bạn đã thấy, với tham số đầu tiên của hàm `view` đó là tên của tập tin view trong thư mục `resources/views`. Tham số thứ hai chính là một biến hoặc mảng với các dữ liệu cần dùng cho view. Trong trường hợp này, chúng ta đã truyền vào một biến `name`, nó sẽ hiển thị trong view bằng các thực hiện lệnh `echo` lên biến.

Tất nhiên, các views có thể chứa trong các thư mục con ở trong thư mục `resources/views`. Dấu "Chấm" sẽ ngăn cách cách thư mục con. Ví dụ như sau, nếu như view của bạn đặt tại `resources/views/admin/profile.php`, thì khi này nó sẽ trông như thế này:

```PHP
return view('admin.profile', $data);
```

#### Xác định một view tồn tại

Nếu bạn muốn xác đinh một view có tồn tại hay không, bạn có thể sử dụng phương thức `exists` được gọi từ `view` không có tham số. Với phương thức này sẽ trả về `true` nếu view này tồn tại:

```PHP
use Illuminate\Support\Facades\View;

if (View::exists('emails.customer')) {
    //
}
```

Khi hàm `view` được gọi và không có tham số, thì nó chính là thể hiện của `Illuminate\Contracts\View\Factory`, điều này cho phép ta truy cập tất cả các phương thức của đối tượng factory.

#### Creating The First Available View

Sử dụng phương thức ``first``, bạn có thể tạo ra view đầu tiện mà tồn tại trong các mảng view cho sẵn . Điều này hữu ích nếu ứng dụng hoặc gói của bạn cho phép tùy chỉnh hoặc ghi đè các chế độ xem:

```PHP
return view()->first(['custom.admin', 'admin'], $data);
```

Tất nhiên, bạn có thể gọi phương thức này trong View:

```PHP
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

## Truyền dữ liệu vào Views

Như ở ví dụ trước, bạn có thể truyền vào một mảng giá trị vào views:

```PHP
return view('greetings', ['name' => 'Victoria']);
```

Khi truyền dữ liệu bằng cách này, `$data` sẽ thành một mảng có khóa/giá trị tương ứng. Bên trong view, bạn có thể sử dụng các giá trị bằng cách gọi biến với tên là khóa của mảng, ví dụ như `<?php echo $key; ?>`. 

Một các khác có thể truyền dữ liệu vào view `view`, bạn sử dụng phương thức `with` để truyền dữ liệu đến view:

```PHP
return view('greeting')->with('name', 'Victoria');
```

### Chia sẻ dữ liệu vào tất cả các Views

Thỉnh thoảng, bạn cần dùng một số dữ liệu nhất định với tất cả các views trong ứng dụng của bạn. Bạn có thể dùng phương thức `share` của view factory. Thông thông thường, bạn cần phải gọi phương thức `share` trong phương thức `boot` từ một service provider. Bạn có thể thêm chúng vào trong `AppServiceProvider` hoặc tự tạo ra một Service Provider khác:

```PHP
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
```

## View Composers

View composers sẽ gọi trực tiếp một phương thức hoặc một lớp khi một view được render (dịch). Nếu bạn có một dữ liệu mà bạn cần ràng buộc nó lại tại thời điểm view của bạn được render, một view composer sẽ giúp bạn xử lý sắp xếp các tư duy logic trong view đó.

Nào chúng ta hãy đăng ký một view composer cùng với [service provider](providers.md). Chúng ta sẽ sử dụng hàm `view` để truy cập vào mẫu `Illuminate\Contracts\View\Factory`. Hãy nhớ rằng, Laravel sẽ không có thư mục mặc định cho view composers. Bạn có quyền tạo chúng bất cứ ở đâu mà bạn muốn. Ví dụ, Bạn có thể tạo thư mục `App\Http\ViewComposers` chẳng hạn:

```PHP
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
```

>Hãy nhớ rằng, nếu bạn tạo ra một Service Provider chứa các đăng ký view composer, bạn cần thêm nó vào bên trong mảng `providers` chứa tại tập tin cấu hình `config/app.php`.

Bây giờ chúng ta đã đăng ký thành công một view composer, Phương thức `ProfileComposer@compose` sẽ thực thi tại thời điểm mà view `profile` được rendered. Vì vậy, nào chúng ta hãy định nghĩa lớp đó:

```PHP
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
```

Như vậy trước khi view đó được rendered, phương thức `compose` sẽ gọi `Illuminate\View\View` qua `$view`. Bạn có thể dụng phương thức `with` để ràng buộc dữ liệu đến view.

> **Chú ý:**  Tất các các view composers được xử lý thông qua [service container](container.md), vì vậy bạn có thể thêm các phụ thuộc vào bên trong phương thức khởi tạo contructor của view composer.

#### Đính kèm Composer vào nhiều Views

Bạn có thể đính kèm nhiều view vào view composer bằng cách truyền vào một mảng chứa tất cả các view vào thương thức `composer`:

```PHP
view()->composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```

Phương thức `composer` có thể dùng `*` để đinh kèm toàn bộ view của bạn:

```PHP
view()->composer('*', function ($view) {
        //
    });
```

#### View Creators

View **creators** rất giống với view composers; Tuy nhiên, nó sẽ tác động ngay lập tức vào các view thay vì chờ các view cho tới khi chúng được rendered. Để đăng ký một view creator, sử dụng phương thức `creator`:

```PHP
view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
```