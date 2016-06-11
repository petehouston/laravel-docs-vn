# Intermediate Task List

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Chuẩn bị cơ sở dữ liệu](#prepping-the-database)
    - [Database Migrations](#database-migrations)
    - [Eloquent Models](#eloquent-models)
    - [Eloquent Relationships](#eloquent-relationships)
- [Routing](#routing)
    - [Hiển thị view](#displaying-a-view)
    - [Authentication](#authentication-routing)
    - [Task Controller](#the-task-controller)
- [Xây dựng Layouts & Views](#building-layouts-and-views)
    - [Xác định Layout](#defining-the-layout)
    - [Xác định Child View](#defining-the-child-view)
- [Thêm Tasks](#adding-tasks)
    - [Validation](#validation)
    - [Tạo Task](#creating-the-task)
- [Hiển thị những Task đã tồn tại](#displaying-existing-tasks)
    - [Dependency Injection](#dependency-injection)
    - [Hiển thị các Task](#displaying-the-tasks)
- [Xoá Tasks](#deleting-tasks)
    - [Thêm button Delete](#adding-the-delete-button)
    - [Lắp ghép Route Model](#route-model-binding)
    - [Authorization](#authorization)
    - [Xoá Task](#deleting-the-task)

<a name="introduction"></a>
## Giới thiệu

Bài hướng dẫn nhanh này là lời giới thiệu nâng cao hơn về Laravel framework, bao gồm các nội dung database migrations, Eloquent ORM, routing, authentication, authorization, dependency injection, validation, views, và Blade templates. Đây là một điểm khởi đầu tốt nếu như bạn đã quen với Laravel cơ bản hay là PHP framework nói chung.

Để lựa chọn ví dụ về các tính năng của Laravel, chúng ta sẽ xây dựng một ứng dụng "task list" mà ta cần để theo dõi những công việc muốn hoàn thành. Nói cách khác, là ví dụ về "to-do" list. Ngược lại so với hướng dẫn "cơ bản", hướng dẫn này sẽ cho phép người dùng có thể tạo các tài khoản và xác thực với ứng dụng. Source code đầy đủ cho project này là [có sẵn trên GitHub](https://github.com/laravel/quickstart-intermediate).

<a name="installation"></a>
## Cài đặt

#### Cài đặt Laravel

Tất nhiên, việc đầu tiên chúng ta cần là một bản cài đặt hoàn toàn mới của Laravel framework. Bạn có thể sử dụng [máy ảo Homestead](/docs/{{version}}/homestead) hoặc sử dụng môi trường PHP cục bộ để chạy framework. Sau khi môi trường cục bộ sẵn sàng, bạn có thể cài đặt Laravel framework bằng cách sử dụng Composer:

    composer create-project laravel/laravel quickstart --prefer-dist

#### Cài đặt bản Quickstart (Tùy chọn)

Bạn hoàn toàn tự do để đọc phần còn lại của hướng dẫn nhanh này; tuy nhiên, nếu như bạn muốn download source code của hướng dẫn nhanh này và chạy thử trên máy local của bạn thì bạn có thể clone Git repository của nó và cài đặt các thành phần phụ thuộc:

    git clone https://github.com/laravel/quickstart-intermediate quickstart
    cd quickstart
    composer install
    php artisan migrate

Để có thêm thông tin về việc xây dựng môi trường phát triển Laravel cục bộ, hãy xem qua document về [Homestead](/docs/{{version}}/homestead) và [installation](/docs/{{version}}/installation) documentation.

<a name="prepping-the-database"></a>
## Chuẩn bị cơ sở dữ liệu

<a name="database-migrations"></a>
### Database Migrations

Đầu tiên, hãy sử dụng migration để xác định database table chứa các task của chúng ta. Database migrations của Laravel cung cấp phương pháp đơn giản để định nghĩa cấu trúc database và chỉnh sửa database sử dụng các đoạn code PHP trơn tru, dễ hiểu. Thay vì bảo thành viên trong nhóm của bạn thêm cột vào trong database cục bộ của họ, thì thành viên trong nhóm của bạn đơn giản chỉ cần chạy migration mà bạn đã đấy lên source control.

#### Bảng `users`

Kể từ khi chúng ta cho phép người dùng tạo tài khoản của họ trong ứng dụng, thì chúng ta cần phải có một bảng để chứa tất cả các tài khoản của người dùng. Thật may mắn, Laravel đã có sẵn một migration để tạo bảng `users` cơ bản, do đó chúng ta không cần phải tạo một bảng khác một cách thủ công. Migration mặc định cho bảng `users` được đặt trong thư mục `database/migrations`.

#### Bảng `tasks`

Tiếp theo, hãy xây dụng một bảng cơ sở dữ liệu mà chứa các task của chúng ta. [Artisan CLI](/docs/{{version}}/artisan) có thể được sử dụng để tạo ra nhiều kiểu class và sẽ tiết kiệm được kha khá thời gian viết code khi xây dụng dự án dựa vào Laravel. Với trường hợp này, hãy sử dụng câu lệnh `make:migration` để tạo ra một database migration mới cho bảng `tasks`.

    php artisan make:migration create_tasks_table --create=tasks

The migration will be placed in the `database/migrations` directory of your project. As you may have noticed, the `make:migration` command already added an auto-incrementing ID and timestamps to the migration file. Let's edit this file and add an additional `string` column for the name of our tasks, as well as a `user_id` column which will link our `tasks` and `users` tables:

Migration sẽ được để trong thư mục `database/migration`. Có thể bạn đã để ý, câu lệnh `make:migration` đã thêm thêm auto-íncrementing ID và timestamps vào file migration. Hãy chỉnh sửa file này và thêm cột `string` cho tên của các task, cũng như là cột `user_id` để kết nối 2 bảng `tasks` và `users`

    <?php

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateTasksTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('tasks', function (Blueprint $table) {
                $table->increments('id');
                $table->integer('user_id')->index();
                $table->string('name');
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('tasks');
        }
    }

Để chạy migration, chúng ta sẽ sử dụng câu lệnh `migrate` của Artisan. Nếu bạn dùng Homestead, bạn nên chạy command này ở trong máy ảo(dùng ssh để kết nối), vì máy của bạn không kết nối trực tiếp đến database.

    php artisan migrate

Câu lệnh này sẽ tạo tất cả các bảng trong cơ sở dữ liệu. Nếu bạn kiểm tra bảng trong cơ sở dữ liệu, bạn có thể thấy bảng mới `tasks` và `users` có những cột mà đã được định nghĩa trong migration của ta. Tiếp theo, chúng ta đã sẵn sàng để định nghĩa những model Eloquent ORM.

<a name="eloquent-models"></a>
### Eloquent Models

[Eloquent](/docs/{{version}}/eloquent) là ORM (ánh xạ quan hệ đối tượng) mặc định của Laravel. Eloquent làm cho việc lấy và lưu trữ dữ liệu trong cơ sở dữ liệu trở nên đỡ đau đầu hơn bằng cách định nghĩa ra các "mô hình-model" rõ ràng. Thường thì, mỗi một mô hình Eloquent sẽ tương ứng với một bảng trong cơ sở dữ liệu.

#### Model `User`

Đầu tiên, chúng ta cần một model tương ứng với bảng `users` trong database, nếu bạn nhìn qua thư mục `app` trong project thì bạn sẽ thấy rằng Laravel đã có sẵn model `User`, do đó chúng ta không cần tạo nó nữa.

#### Model `Task`

Vậy, hãy định nghĩa model `Task` tương ứng với bảng `tasks` trong cơ sở dữ liệu mà ta đã tạo. Một lần nữa, chúng ta có thể sử dụng câu lệnh Artisan để tạo ra model này. Trong trường hợp này, chúng ta sử dụng lệnh `make:model`:

    php artisan make:model Task

Model sẽ được đặt trong thư mục `app` của application. Mặc định thì class model là trống. Chúng ta không cần phải mô tả rõ ràng cho Eloquent model rằng bảng nào được tương ứng với model, bởi vì nó sẽ giả định cơ sở dữ liệu là dạng số nhiều của tên model. Do đó, trong trường hợp này, model `Task` được giả định tương ứng với bảng `tasks` trong cơ sở dữ liệu.

Hãy thêm một vài thứ vào model này. Đầu tiên, chúng ta sẽ chỉ ra rằng thuộc tính `name` trong model nên là `mass-assignable`. Việc này sẽ cho phép chúng ta điền thuộc tính `name` khi sử dụng câu lệnh `create` của Eloquent.

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Task extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Chúng ta sẽ học thêm về cách sử dụng các model của Eloquent khi chúng ta thêm các route (định tuyến) và ứng dụng. Tất nhiên, bạn có thể tự do tham khảo thêm chi tiết tại [tài liệu Eloquent đầy đủ](/docs/{{version}}/eloquent).

<a name="eloquent-relationships"></a>
### Eloquent Relationships

Bây giờ các model của chúng ta đã được định nghĩa, chúng ta cần kết nối chúng. Ví dụ như, một `User` của chúng ta có thể có nhiều `Task`, khi một `Task` chỉ được gán cho một `User`. Định nghĩa một mối quan hệ sẽ cho phép chúng ta dễ dàng di chuyển qua các quan hệ tương tự như sau:

    $user = App\User::find(1);

    foreach ($user->tasks as $task) {
        echo $task->name;
    }

#### Mối quan hệ của `tasks`

Đầu tiên, hãy định nghĩa mối quan hệ `tasks` trên model `User` của chúng ta. Mối quan hệ trong Eloquent được định nghĩa như là các method trong các model. Eloquent hỗ trợ vài kiểu khác nhau của các mối quan hệ, do đó nên chắc chắn rằng tham khảo [tài liệu Eloquent đầy đủ](/docs/{{version}}/eloquent-relationships) để có thêm thông tin. Trong trường hợp này , chúng ta sẽ định nghĩa function `tasks` trong model `User`, đây là function gọi method `hasMany` được cung cấp bởi Eloquent.

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        // Other Eloquent Properties...

        /**
         * Get all of the tasks for the user.
         */
        public function tasks()
        {
            return $this->hasMany(Task::class);
        }
    }

#### Mối quan hệ của `user`

Tiếp theo, hãy định nghĩa mối quan hệ của `user` trong Model `Task`. Chúng ta sẽ định nghĩa mối quan hệ như là method của model. Trong trường hợp này, chúng ta sẽ sử dụng method `belongsTo`, được cung cấp bởi Eloquent, để định nghĩa mối quan hệ.

    <?php

    namespace App;

    use App\User;
    use Illuminate\Database\Eloquent\Model;

    class Task extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];

        /**
         * Get the user that owns the task.
         */
        public function user()
        {
            return $this->belongsTo(User::class);
        }
    }

Tuyệt vời! Giờ các mối quan hệ của chúng ta đã hoàn thanh, chúng ta có thể bắt đầu xây dụng các controller!

<a name="routing"></a>
## Routing

Trong [version cơ bản](/docs/{{version}}/quickstart) của ứng dụng task list của chúng ta, chúng ta đã định nghĩa tất cả logic sử dụng Closures trong file `routes.php`. Với phần lớn ứng dụng như này, chúng ta sẽ sử dụng [controllers](/docs/{{version}}/controllers) để tổ chức các route của mình. Các controller sẽ cho phép chúng ta phân tách logic việc xử lý các request HTTP ra nhiều file khác nhau để quản lý tốt hơn.

<a name="displaying-a-view"></a>
### Hiển thị View

Chúng ta sẽ có một route đơn lẻ sử dụng Closures: route `/`, cái mà sẽ dùng để hiển thị trang đích đơn giản cho ứng dụng khách. Từ route này, chúng ta sẽ render một trang template HTML bao gồm trang "welcome".

Trong Laravel, tất cả các template HTML được lưu trong thư mục `resources/views`, và chúng ta có thể sử dụng `view` helper để trả về một trong những template đó từ route của chúng ta:

    Route::get('/', function () {
        return view('welcome');
    });

Tất nhiên, chúng ta cần định nghĩa view này. Chúng ta sẽ làm điều đó sau một chút nữa.

<a name="authentication-routing"></a>
### Authentication

Hãy nhớ rằng, chúng ta cũng cần cho phép người dùng tạo tài khoản và đăng nhập vào ứng dụng của chúng ta. Thông thường, điều đó có thể khá là buồn chán để xây dụng nguyên một layer dùng để authentication vào trong ứng dụng web. Tuy nhiên, kể từ lúc mà điều đó trở thành một thứ cần thiết phổ biến, thì Laravel đã chủ đích để làm cho điều này hoàn toàn đỡ mệt mỏi. 

Đầu tiên, chú ý rằng trong ứng dụng đã có sẵn `app/Http/Controllers/Auth/AuthController`. Controller này sử dụng một trait khá đặc biệt `AuthenticatesAndRegistersUsers` cái mà đã bao gồm tất cả những logic cần thiết để tạo và xác thực người dùng.

#### Authentication Routes & Views

Vậy, còn lại chúng ta phải làm những gì? À vâng, chúng ta vẫn cần phải tạo template để đăng ký và đăng nhập cũng như là định nghĩa các route chỉ đến controller authentication. Chúngta có thể làm tất cả điều đó sử dụng câu lệnh Artisan `make:auth`:

    php artisan make:auth

> **Note:** Nếu bạn muốn xem toàn bộ ví dụ cho các views này, nhớ rằng toàn bộ source code của ứng dụng là [sẵn có trên GitHub](https://github.com/laravel/quickstart-intermediate).

Bây giờ, tất cả những gì chúng ta phải làm là thêm route xác thực (authentication) vào file route của chúng ta. Chúng ta có thể làm điều đó sử dụng method `auth` trên facade `Route`, cái mà sẽ đăng ký tất cả các route mà chúng ta cần cho việc đăng ký, đăng nhập và lấy lại pass:

    // Authentication Routes...
    Route::auth();

Một khi route `auth` đã được đăng ký, đảm bảo rằng thuộc tính `$redirectTo` trong `app/Http/Controllers/Auth/AuthController` được gán đến `/tasks`:

    protected $redirectTo = '/tasks';

Cũng cần thiết để update file `app/Http/Middleware/RedirectIfAuthenticated.php` đúng path cần chuyển hướng:

    return redirect('/tasks');

<a name="the-task-controller"></a>
### Controller Task

Chúng ta biết rằng sẽ phải nhận và lưu các task, hãy tạo `TaskController` sử dụng Artisan CLI, nó sẽ tạo một controller mới trong thư mục `app/Http/Controllers:

    php artisan make:controller TaskController

Bây giờ controller đã được tạo ra, tiếp theo chúng ta sẽ chuyển vài route từ `app/Http/routes.php` để trỏ sang controller:

    Route::get('/tasks', 'TaskController@index');
    Route::post('/task', 'TaskController@store');
    Route::delete('/task/{task}', 'TaskController@destroy');

#### Authenticating All Task Routes

Với ứng dụng này, chúng ta muốn tất cả các task route đều yêu cầu người dùng đã xác thực, người dùng phải "đã đăng nhập" vào ứng dụng để có thể tạo task. Do đó, chúng ta cần phải hạn chế truy cập vào các task route thành chỉ cho phép người dùng đã xác thực. Laravel làm điều đó trở nên chắc chắn bằng cách sử dụng [middleware](/docs/{{version}}/middleware).

Để yêu cầu người dùng đã xác thực cho tất cả các actions trong controller, chúng ta thêm một lời gọi đến `middleware` từ constructor của controller. Tất cả các route middleware có sẵn được định nghĩa trong `app/Http/Kernel.php`. Trong trường hợp này, chúng ta muốn gán middleware `auth` đến tất cả các action trong controller:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Requests;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class TaskController extends Controller
    {
        /**
         * Create a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');
        }
    }

<a name="building-layouts-and-views"></a>
## Xây dựng Layouts & Views

Phần chính của ứng dụng này chỉ có một view đơn có chứa form để thêm task mới cũng như là hiển thị tất cả các task hiện tại. Để giúp bạn hình dung ra được view này, thì đây là một ảnh chụp màn hình của ứng dụng đã hoàn thành với Bootsrap CSS cơ bản được áp dụng:

![Application Image](https://laravel.com/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### Defining The Layout

Hầu hết các ứng dụng web chia sẻ layout giống nhau xuyên suốt các trang. Ví dụ, ứng dụng này có thanh navigation phía trên thường sẽ xuất hiện ở tất cả các page (nếu chúng ta có nhiều hơn 1 trang). Laravel làm chi việc chia sẻ các tính năng chung giữa các page trở nên dễ dàng bằng cách sử dụng Blade **layout**.

Như là chúng ta đã bàn luận trước đây, tất cả các view của Laravel được lưu trong `resources/views`. Do đó, hãy định nghĩa layout mới trong `resources/views/layouts/app.blade.php`. Phần mở rộng `.blade.php` chỉ cho framework sử dụng [Blade templating engine](/docs/{{version}}/blade) để tạo ra view. Tất nhiên, bạn có thể sử dụng template PHP thuần với Laravel. Tuy nhiên, Blade cung cấp các lối tắt tiện lợi để viết những template ngắn gọn.

File `app.blade.php` nên được nhìn thấy như sau:

    <!-- resources/views/layouts/app.blade.php -->

    <!DOCTYPE html>
    <html lang="en">
        <head>
            <title>Laravel Quickstart - Intermediate</title>

            <!-- CSS And JavaScript -->
        </head>

        <body>
            <div class="container">
                <nav class="navbar navbar-default">
                    <!-- Navbar Contents -->
                </nav>
            </div>

            @yield('content')
        </body>
    </html>

Chú ý phần `@yield('content')` của layout. Đây là phần chỉ thị Blade đặc biệt quy định cụ thể tất cả các view con muốn mở rộng layout có thể thêm nội dung của chúng vào. Tiếp theo, hãy định nghĩa view con sẽ sử dụng layout này và cung cấp các nội dung cơ bản.

<a name="defining-the-child-view"></a>
### Định nghĩa View con

Layout của ứng dụng của chúng ta đã hoàn thành. Tiếp theo, chúng ta cần định nghĩa một view sẽ chứa form để tạo task mới cũng như là chứa 1 bảng các danh sách những task tồn tại. Hãy định nghĩa view này trong `resources/views/tasks/index.blade.php`, cái mà sẽ tương ứng với method `index` của `TaskController` của chúng ta.

Chúng ta sẽ bỏ qua một vài bản mẫu của Bootstrap CSS và chỉ tập trung vào những điều quan trọng. Nhớ rằng, bạn có thể tải xuống bản source đầy đủ cho ứng dụng này tại [GitHub](https://github.com/laravel/quickstart-intermediate): 

    <!-- resources/views/tasks/index.blade.php -->

    @extends('layouts.app')

    @section('content')

        <!-- Bootstrap Boilerplate... -->

        <div class="panel-body">
            <!-- Display Validation Errors -->
            @include('common.errors')

            <!-- New Task Form -->
            <form action="{{ url('task') }}" method="POST" class="form-horizontal">
                {{ csrf_field() }}

                <!-- Task Name -->
                <div class="form-group">
                    <label for="task-name" class="col-sm-3 control-label">Task</label>

                    <div class="col-sm-6">
                        <input type="text" name="name" id="task-name" class="form-control">
                    </div>
                </div>

                <!-- Add Task Button -->
                <div class="form-group">
                    <div class="col-sm-offset-3 col-sm-6">
                        <button type="submit" class="btn btn-default">
                            <i class="fa fa-plus"></i> Add Task
                        </button>
                    </div>
                </div>
            </form>
        </div>

        <!-- TODO: Current Tasks -->
    @endsection

#### Vài ghi chú giải thích

Trước khi đi đến phần tiếp theo, hãy nói một chút về template này. Đầu tiên, chỉ thị `@extends` cho Blade biết rằng chúng ta sử dụng layout được định nghĩa tại `resources/views/layouts/app.blade.php`. Tất cả các nội dung giữa `section('content')` và `@endsection` sẽ được thêm vào vị trí của chỉ thị `@yield('content')` trong file layout `app.blade.php`.

Chỉ thị `@include('common.errors')` sẽ load bản mẫu tại `resources/views/common/errors.blade.php`. Chúng ta vẫn chưa định nghĩa template này, nhưng sẽ sớm định nghĩa thôi!

Bây giờ chúng ta đã định nghĩa xong layout cơ bản và view cho ứng dụng của chúng ta. Hãy tiếp tục và trả cái view này từ method `index` của `TaskController`:

    /**
     * Display a list of all of the user's task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
        return view('tasks.index');
    }

Tiếp theo, chúng ta đã sẵn sàng để thêm code vào method route controller `POST /task` để xử lý input từ form và thêm task mới vào database.

<a name="adding-tasks"></a>
## Thêm các Task

<a name="validation"></a>
### Validation

Giờ chúng ta đã có một form trong view, chúng ta cần thêm code vào method `TaskController@store` để validate đầu vào từ form và tạo task mới. Đầu tiên, hãy phê duyệt (validate) đầu vào.

Đối với form này, chúng ta sẽ làm cho trường `name` là bắt buộc và đặt điều kiện là chỉ bao gồm ít hơn `255` ký tự. Nếu như validate không được thông qua, chúng ta muốn chuyển hướng user quay lại URL `/tasks`, cũng như là làm nhấp nháy những input cũ và các lỗi vào trong [session](/docs/{{version}}/session):

    /**
     * Create a new task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|max:255',
        ]);

        // Create The Task...
    }

Nếu bạn đã đọc qua phần [basic quickstart](/docs/{{version}}/quickstart), thì bạn sẽ để ý rằng đoạn code validation này có một chút khác biệt! Kể từ khi chúng ta sử dụng controller, chúng ta có thể tận dụng sự tiện lợi của trait (đặc điểm) `ValidatesRequest` được kèm theo trong controller base của Laravel. Trait này cung cấp một phương thức `validate` đơn giản chấp nhận một request và một chuỗi các quy tắc validation.

Chúng ta thậm chí không cần phải xác nhận nếu validation thất bại hay phải xử lý điều hướng. Nếu như validation thất bại với những quy tắc đã được đặt ra, user sẽ được chuyển hướng tự động quay lại trang trước và các lỗi sẽ được tự động nhấp nháy. Tuyệt vãi!

#### Biến `$errors`

Nhớ rằng chúng ta sử dụng chỉ thị `@include('common.errors')` trong view để tạo ra validation error của form. View `common.errors` sẽ cho phép chúng ta dễ dàng hiển thị các lỗi validation với cùng một format xuyên suốt tất cả các trang của chúng ta. Hãy định nghĩa nội dung của view này:

    <!-- resources/views/common/errors.blade.php -->

    @if (count($errors) > 0)
        <!-- Form Error List -->
        <div class="alert alert-danger">
            <strong>Whoops! Something went wrong!</strong>

            <br><br>

            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif


> **Note:** Biến `$errors` là sẵn có trong **mỗi** view của Laravel. Nó đơn giản là một instance rỗng của `ViewErrorBag` nếu như không có lỗi validation nào được hiển thị.

<a name="creating-the-task"></a>
### Tạo Task

Bây giờ thì việc validate data đầu vào đã được xử lý, hãy thực sự tạo một task mới bằng cách tiếp tục hoàn thành route của chúng ta. Một khi task mới đã được tạo, chúng ta sẽ điều hướng user quay lại URL `/tasks`. Để tạo task mới, chúng ta sẽ dựa vào sức mạnh của các mối quan hệ (relationships) của Eloquent.

Đa số các mối quan hệ (relationships) của Laravel đều cung cấp một method `create`, nó nhận một loạt các thuộc tính và sẽ tự động thiết lập các giá trị khoá ngoại trên các mô hình có liên quan trước khi lưu trữ nó trong cơ sở dữ liệu. Trong trường hợp này, những method `create` sẽ tự động thiết lập các thuộc tính `user_id` của task đến ID của user đang được xác thực, cái mà chúng ta có thể truy cập bằng cách sử dụng `$request->user()`:

    /**
     * Create a new task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|max:255',
        ]);

        $request->user()->tasks()->create([
            'name' => $request->name,
        ]);

        return redirect('/tasks');
    }

Ngon! Bây giờ chúng ta đã có thể tạo các task thành công. Tiếp theo, hãy tiếp tục hoàn thiện view của chúng ta bằng cách xây dựng một danh sách của các task đã tồn tại.

<a name="displaying-existing-tasks"></a>
## Hiển thị các Task đã tồn tại

Đầu tiên, chúng ta cần chỉnh sửa method `TaskController@index` để truyền tất cả các task đã tồn tại đến view. Function `view` chấp nhận đối số thứ hai là chuỗi các dữ liệu sẽ được hiển thị trên view, mỗi key trong array sẽ trở thành một biến trong view. Ví dụ ta có thể làm như sau:

    /**
     * Display a list of all of the user's task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
        $tasks = $request->user()->tasks()->get();

        return view('tasks.index', [
            'tasks' => $tasks,
        ]);
    }

Tuy nhiên, hãy tìm hiểu mốt vài khả năng 'dependency injection' của Laravel để 'inject' một `TaskRespository` vào `TaskController` của chúng ta, cái mà chúng ta sẽ sử dụng cho tất cả các truy cập dữ liệu.

<a name="dependency-injection"></a>
### Dependency Injection

[service container](/docs/{{version}}/container) của Laravel là một trong những chức năng mạnh nhất của toàn bộ framework. Sau khi đọc xong quickstart, hãy chắc chắn rằng bạn đã đọc qua toàn bộ tài liệu của container.

#### Tạo Repository

Như chúng ta đã đề cập trước đây, chúng ta muốn định nghĩa một `TaskRespository` chứa tất cả các logic truy cập data cho model `Task`. Nó sẽ đặc biệt có ích nếu ứng dụng lớn lên và bạn cần chia sẻ một vài câu truy vấn Eloquent xuyên suốt toàn ứng dụng.

Do đó, hãy tạo một thư mục `app/Respositories` và thêm class `TaskRespository`. Nhớ rằng, tất cả các folder trong `app` được load tự động sử dụng tiêu chuẩn PSR-4 auto-loading, do đó bạn thoải mái tạo thêm nhiều thư mục nếu cần:

    <?php

    namespace App\Repositories;

    use App\User;

    class TaskRepository
    {
        /**
         * Get all of the tasks for a given user.
         *
         * @param  User  $user
         * @return Collection
         */
        public function forUser(User $user)
        {
            return $user->tasks()
                        ->orderBy('created_at', 'asc')
                        ->get();
        }
    }

#### Injecting The Repository

Một khi respository của chúng ta đã được định nghĩa, chúng ta đơn giản "type-hint" nó vào constructor của `TaskController` và sử dụng nó trong route `index` của chúng ta. Kể từ khi Laravel sử dụng container để xử lý tất cả các controller, các dependency sẽ được tự động inject vào instance của controller:

    <?php

    namespace App\Http\Controllers;

    use App\Task;
    use App\Http\Requests;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    use App\Repositories\TaskRepository;

    class TaskController extends Controller
    {
        /**
         * The task repository instance.
         *
         * @var TaskRepository
         */
        protected $tasks;

        /**
         * Create a new controller instance.
         *
         * @param  TaskRepository  $tasks
         * @return void
         */
        public function __construct(TaskRepository $tasks)
        {
            $this->middleware('auth');

            $this->tasks = $tasks;
        }

        /**
         * Display a list of all of the user's task.
         *
         * @param  Request  $request
         * @return Response
         */
        public function index(Request $request)
        {
            return view('tasks.index', [
                'tasks' => $this->tasks->forUser($request->user()),
            ]);
        }
    }

<a name="displaying-the-tasks"></a>
### Hiển thị các Task

Một khi data đã được thông qua, chúng ta có thể quay qua các task trong view `tasks/index.blade.php` và hiển thị chúng lên table. Blade construct `@foreach` cho phép chúng ta viết một vòng lặp ngắn gọn biên dịch nhanh chóng các code PHP thuần:

    @extends('layouts.app')

    @section('content')
        <!-- Create Task Form... -->

        <!-- Current Tasks -->
        @if (count($tasks) > 0)
            <div class="panel panel-default">
                <div class="panel-heading">
                    Current Tasks
                </div>

                <div class="panel-body">
                    <table class="table table-striped task-table">

                        <!-- Table Headings -->
                        <thead>
                            <th>Task</th>
                            <th>&nbsp;</th>
                        </thead>

                        <!-- Table Body -->
                        <tbody>
                            @foreach ($tasks as $task)
                                <tr>
                                    <!-- Task Name -->
                                    <td class="table-text">
                                        <div>{{ $task->name }}</div>
                                    </td>

                                    <td>
                                        <!-- TODO: Delete Button -->
                                    </td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                </div>
            </div>
        @endif
    @endsection

Ứng dụng task của chúng ta đã gần như hoàn thành. Nhưng, chúng ta chưa có cách nào xoá các task đã tồn tại khi chúng đã xong. Hãy làm điều đó tiếp theo!    

<a name="deleting-tasks"></a>
## Xoá các Task

<a name="adding-the-delete-button"></a>
### Thêm button Delete

Chúng tôi đã để lại một note "TODO" trong source code, nơi mà nên để button Delete ở đó. Do đó, hãy thêm button Delete vào mỗi hàng của danh sách các task trong view `tasks/index.blade.php`. Chúng ta sẽ tạo một button nhỏ cho mỗi task trong danh sách. Khi button được click, request `DELETE /task` sẽ được gửi đến application và trigger method `TaskController@destroy`:

    <tr>
        <!-- Task Name -->
        <td class="table-text">
            <div>{{ $task->name }}</div>
        </td>

        <!-- Delete Button -->
        <td>
            <form action="{{ url('task/'.$task->id) }}" method="POST">
                {{ csrf_field() }}
                {{ method_field('DELETE') }}

                <button type="submit" id="delete-task-{{ $task->id }}" class="btn btn-danger">
                    <i class="fa fa-btn fa-trash"></i>Delete
                </button>
            </form>
        </td>
    </tr>

<a name="a-note-on-method-spoofing"></a>
#### Ghi chú về phương thức giả mạo (Spoofing)

Chú ý rằng method của button được liệt kê dưới dạng `POST`, mặc dù chúng ta đáp ứng cho request bằng route `Route:delete. HTML form chỉ cho phép các HTTP verb là `GET` và `POST`, do đó chúng ta cần một cách để giả mạo request `DELETE` từ form.

Chúng ta có thể giả mạo request `DELETE` bằng cách xuất ra các kết quả của function `method_field('DELETE')` trong form của chúng ta, Function này sẽ tạo ra một form input ẩn và Laravel sẽ nhận diện và sử dụng để ghi đè request HTTP thực tế. Field được tạo ra sẽ có dạng như sau:

    <input type="hidden" name="_method" value="DELETE">

<a name="route-model-binding"></a>
### Route Model Binding

Bây giờ, chúng ta gần như sẵn sàng để định nghĩa method `destroy` vào `TaskController`. Tuy nhiên, đầu tiên chúng ta phải xem lại kiểm tra lại khai báo route và controller method cho route này:

    Route::delete('/task/{task}', 'TaskController@destroy');

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        //
    }

Kể từ khi biến `{task}` trong route của chúng ta khớp với biến `$task` được định nghĩa trong method của controller. [implicit model binding](/docs/{{version}}/routing#route-model-binding) của Laravel sẽ tự động inject Task model tương ứng vào.

<a name="authorization"></a>
### Authorization

Bây giờ chúng ta có một instance `Task` đã được inject vào trong method `destroy`; tuy nhiên, chúng ta không được đảm bảo rằng những user đã được xác nhận "sở hữu" những task đó. Ví dụ, một request độc hại có thể được chủ định để xoá đi task của user khác bằng cách gửi một task ID bất kỳ đén URL `/tasks/{task}`. Do đó, chúng ta cần sử dụng khả năng authorization của Laravel để đảm bảo rằng user đã được xác thực là chủ sở hữu của instance `Task` đã được inject vào route.

#### Tạo quy ước

Laravel sử dụng các "quy ước" để sắp xếp các logic authorization thành các lớp nhỏ, đơn giản. Thông thường, mỗi quy ước tương ứng với một model. Do đó, hãy tạo một `TaskPolicy` sử dụng CLI Artisan, cái mà sẽ tạo ra file trong `app/Policies/TaskPolicy.php`:

    php artisan make:policy TaskPolicy

Tiếp, hãy thêm method `destroy` vào policy. Method này sẽ nhận các instance của `User` và `Task`. Method sẽ đơn giản kiểm tra nếu user's ID tương ứng với `user_id` trong task. Trên thực tế, tất cả các method policy sẽ return `true` hoặc `false`:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Task;
    use Illuminate\Auth\Access\HandlesAuthorization;

    class TaskPolicy
    {
        use HandlesAuthorization;

        /**
         * Determine if the given user can delete the given task.
         *
         * @param  User  $user
         * @param  Task  $task
         * @return bool
         */
        public function destroy(User $user, Task $task)
        {
            return $user->id === $task->user_id;
        }
    }

Cuối cùng, chúng ta cần liên kết `Task` model với `TaskPolicy`. Chúng ta có thể làm d diều này bằng cách thêm một dòng thuộc tính `$policies` trong  file `app/Providers/AuthServiceProvider.php`. Điều này sẽ thông báo cho Laravel rằng policy nào sẽ được dùng mỗi khi chúng ta cố gắng để authorize một hành động tròng instance `Task`:

    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Task' => 'App\Policies\TaskPolicy',
    ];


#### Authorizing The Action

Bây giờ, policy của chúng ta đã được viết, hãy sử dụng nó trong method `destroy` của chúng ta. Tất cả các controller của Laravel có thể gọi method `authorize`, cái mà được cung cấp bởi trait (đặc tính) `AuthorizesRequest`:

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        // Delete The Task...
    }

Hãy kiểm tra này gọi phương thức trong một lúc. Đối số đầu tiên được chuyển đến method 'authorize` là tên của method policy mà chúng ta muốn gọi. Đối số thứ hai là các instance của model mà chúng ta đang quan tâm. Nhớ rằng, gần đây chúng ta đã bảo Laravel rằng model `Task` tương ứng với `TaskPolicy`, do đó framework biết policy nào để gọi method `destroy`. User hiện tại sẽ được tự động gửi đến method policy, do đó chúng ta không câng phải gửi nó thủ công.

Nếu action đã được xác thực, code của chúng ta sẽ tiếp tục thực thi bình thường. Tuy nhiên, nếu action không được xác thực (nghĩa là method destroy của policy trả về `false`), một ngoại lệ 403 sẽ được bắn ra và trang lỗi sẽ hiển thị đến người dùng.

> **Note:** Có vài cách khác để tương tác với dich vụ authorization được cung cấp bởi Laraveel. Hãy chắc rằng xem hết tài liệu [authorization documentation](/docs/{{version}}/authorization).

<a name="deleting-the-task"></a>
### Deleting The Task

Cuối cùng, hãy hoàn thành logic vào method `destroy` để thực sự xoá được task đã cho. Chúng ta có thể sử dụng method `delete` của Eloquent để xoá instance của model đã cho trong cơ sở dữ liệu. Một khi bản ghi đã bị xoá, chúng ta sẽ chuyển hướng user quay lại URL `/tasks`:

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        $task->delete();

        return redirect('/tasks');
    }
