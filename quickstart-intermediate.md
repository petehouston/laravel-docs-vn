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

Chúng ta sẽ học thêm về cách sử dụng các model của Eloquent khi chúng ta thêm các route (định tuyến) và ứng dụng. Tất nhiên, bạn có thể tự do tham khảo thêm chi tiết tại [complete Eloquent documentation](/docs/{{version}}/eloquent).

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

In the [basic version](/docs/{{version}}/quickstart) of our task list application, we defined all of our logic using Closures within our `routes.php` file. For the majority of this application, we will use [controllers](/docs/{{version}}/controllers) to organize our routes. Controllers will allow us to break out HTTP request handling logic across multiple files for better organization.

<a name="displaying-a-view"></a>
### Displaying A View

We will have a single route that uses a Closure: our `/` route, which will simply be a landing page for application guests. So, let's fill out our `/` route. From this route, we want to render an HTML template that contains the "welcome" page:

In Laravel, all HTML templates are stored in the `resources/views` directory, and we can use the `view` helper to return one of these templates from our route:

    Route::get('/', function () {
        return view('welcome');
    });

Of course, we need to actually define this view. We'll do that in a bit!

<a name="authentication-routing"></a>
### Authentication

Remember, we also need to let users create accounts and login to our application. Typically, it can be a tedious task to build an entire authentication layer into a web application. However, since it is such a common need, Laravel attempts to make this procedure totally painless.

First, notice that there is already a `app/Http/Controllers/Auth/AuthController` included in your Laravel application. This controller uses a special `AuthenticatesAndRegistersUsers` trait which contains all of the necessary logic to create and authenticate users.

#### Authentication Routes & Views

So, what's left for us to do? Well, we still need to create the registration and login templates as well as define the routes to point to the authentication controller. We can do all of this using the `make:auth` Artisan command:

    php artisan make:auth

> **Note:** If you would like to view complete examples for these views, remember that the entire application's source code is [available on GitHub](https://github.com/laravel/quickstart-intermediate).

Now, all we have to do is add the authentication routes to our routes file. We can do this using the `auth` method on the `Route` facade, which will register all of the routes we need for registration, login, and password reset:

    // Authentication Routes...
    Route::auth();

Once the `auth` routes are registered, verify that the `$redirectTo` property on the `app/Http/Controllers/Auth/AuthController` controller is set to '/tasks':

    protected $redirectTo = '/tasks';

It is also necessary to update the `app/Http/Middleware/RedirectIfAuthenticated.php` file with the proper redirect path:

    return redirect('/tasks');

<a name="the-task-controller"></a>
### The Task Controller

Since we know we're going to need to retrieve and store tasks, let's create a `TaskController` using the Artisan CLI, which will place the new controller in the `app/Http/Controllers` directory:

    php artisan make:controller TaskController

Now that the controller has been generated, let's go ahead and stub out some routes in our `app/Http/routes.php` file to point to the controller:

    Route::get('/tasks', 'TaskController@index');
    Route::post('/task', 'TaskController@store');
    Route::delete('/task/{task}', 'TaskController@destroy');

#### Authenticating All Task Routes

For this application, we want all of our task routes to require an authenticated user. In other words, the user must be "logged into" the application in order to create a task. So, we need to restrict access to our task routes to only authenticated users. Laravel makes this a cinch using [middleware](/docs/{{version}}/middleware).

To require an authenticated users for all actions on the controller, we can add a call to the `middleware` method from the controller's constructor. All available route middleware are defined in the `app/Http/Kernel.php` file. In this case, we want to assign the `auth` middleware to all actions on the controller:

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
## Building Layouts & Views

The primary part of this application only has a single view which contains a form for adding new tasks as well as a listing of all current tasks. To help you visualize the view, here is a screenshot of the finished application with basic Bootstrap CSS styling applied:

![Application Image](https://laravel.com/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### Defining The Layout

Almost all web applications share the same layout across pages. For example, this application has a top navigation bar that would be typically present on every page (if we had more than one). Laravel makes it easy to share these common features across every page using Blade **layouts**.

As we discussed earlier, all Laravel views are stored in `resources/views`. So, let's define a new layout view in `resources/views/layouts/app.blade.php`. The `.blade.php` extension instructs the framework to use the [Blade templating engine](/docs/{{version}}/blade) to render the view. Of course, you may use plain PHP templates with Laravel. However, Blade provides convenient short-cuts for writing cleaner, terse templates.

Our `app.blade.php` view should look like the following:

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

Note the `@yield('content')` portion of the layout. This is a special Blade directive that specifies where all child pages that extend the layout can inject their own content. Next, let's define the child view that will use this layout and provide its primary content.

<a name="defining-the-child-view"></a>
### Defining The Child View

Great, our application layout is finished. Next, we need to define a view that contains a form to create a new task as well as a table that lists all existing tasks. Let's define this view in `resources/views/tasks/index.blade.php`, which will correspond to the `index` method in our `TaskController`.

We'll skip over some of the Bootstrap CSS boilerplate and only focus on the things that matter. Remember, you can download the full source for this application on [GitHub](https://github.com/laravel/quickstart-intermediate):

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

#### A Few Notes Of Explanation

Before moving on, let's talk about this template a bit. First, the `@extends` directive informs Blade that we are using the layout we defined at `resources/views/layouts/app.blade.php`. All of the content between `@section('content')` and `@endsection` will be injected into the location of the `@yield('content')` directive within the `app.blade.php` layout.

The `@include('common.errors')` directive will load the template located at `resources/views/common/errors.blade.php`. We haven't defined this template, but we will soon!

Now we have defined a basic layout and view for our application. Let's go ahead and return this view from the `index` method of our `TaskController`:

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

Next, we're ready to add code to our `POST /task` route's controller method to handle the incoming form input and add a new task to the database.

<a name="adding-tasks"></a>
## Adding Tasks

<a name="validation"></a>
### Validation

Now that we have a form in our view, we need to add code to our `TaskController@store` method to validate the incoming form input and create a new task. First, let's validate the input.

For this form, we will make the `name` field required and state that it must contain less than `255` characters. If the validation fails, we want to redirect the user back to the `/tasks` URL, as well as flash the old input and errors into the [session](/docs/{{version}}/session):

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

If you followed along with the [basic quickstart](/docs/{{version}}/quickstart), you'll notice this validation code looks quite a bit different! Since we are in a controller, we can leverage the convenience of the `ValidatesRequests` trait that is included in the base Laravel controller. This trait exposes a simple `validate` method which accepts a request and an array of validation rules.

We don't even have to manually determine if the validation failed or do manual redirection. If the validation fails for the given rules, the user will automatically be redirected back to where they came from and the errors will automatically be flashed to the session. Nice!

#### The `$errors` Variable

Remember that we used the `@include('common.errors')` directive within our view to render the form's validation errors. The `common.errors` view will allow us to easily show validation errors in the same format across all of our pages. Let's define the contents of this view now:

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


> **Note:** The `$errors` variable is available in **every** Laravel view. It will simply be an empty instance of `ViewErrorBag` if no validation errors are present.

<a name="creating-the-task"></a>
### Creating The Task

Now that input validation is handled, let's actually create a new task by continuing to fill out our route. Once the new task has been created, we will redirect the user back to the `/tasks` URL. To create the task, we are going to leverage the power of Eloquent's relationships.

Most of Laravel's relationships expose a `create` method, which accepts an array of attributes and will automatically set the foreign key value on the related model before storing it in the database. In this case, the `create` method will automatically set the `user_id` property of the given task to the ID of the currently authenticated user, which we are accessing using `$request->user()`:

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

Great! We can now successfully create tasks. Next, let's continue adding to our view by building a list of all existing tasks.

<a name="displaying-existing-tasks"></a>
## Displaying Existing Tasks

First, we need to edit our `TaskController@index` method to pass all of the existing tasks to the view. The `view` function accepts a second argument which is an array of data that will be made available to the view, where each key in the array will become a variable within the view. For example, we could do this:

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

However, let's explore some of the dependency injection capabilities of Laravel to inject a `TaskRepository` into our `TaskController`, which we will use for all of our data access.

<a name="dependency-injection"></a>
### Dependency Injection

Laravel's [service container](/docs/{{version}}/container) is one of the most powerful features of the entire framework. After reading this quickstart, be sure to read over all of the container's documentation.

#### Creating The Repository

As we mentioned earlier, we want to define a `TaskRepository` that holds all of our data access logic for the `Task` model. This will be especially useful if the application grows and you need to share some Eloquent queries across the application.

So, let's create an `app/Repositories` directory and add a `TaskRepository` class. Remember, all Laravel `app` folders are auto-loaded using the PSR-4 auto-loading standard, so you are free to create as many extra directories as needed:

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

Once our repository is defined, we can simply "type-hint" it in the constructor of our `TaskController` and utilize it within our `index` route. Since Laravel uses the container to resolve all controllers, our dependencies will automatically be injected into the controller instance:

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
### Displaying The Tasks

Once the data is passed, we can spin through the tasks in our `tasks/index.blade.php` view and display them in a table. The `@foreach` Blade construct allows us to write concise loops that compile down into blazing fast plain PHP code:

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

Our task application is almost complete. But, we have no way to delete our existing tasks when they're done. Let's add that next!

<a name="deleting-tasks"></a>
## Deleting Tasks

<a name="adding-the-delete-button"></a>
### Adding The Delete Button

We left a "TODO" note in our code where our delete button is supposed to be. So, let's add a delete button to each row of our task listing within the `tasks/index.blade.php` view. We'll create a small single-button form for each task in the list. When the button is clicked, a `DELETE /task` request will be sent to the application which will trigger our `TaskController@destroy` method:

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
#### A Note On Method Spoofing

Note that the delete button's form `method` is listed as `POST`, even though we are responding to the request using a `Route::delete` route. HTML forms only allow the `GET` and `POST` HTTP verbs, so we need a way to spoof a `DELETE` request from the form.

We can spoof a `DELETE` request by outputting the results of the `method_field('DELETE')` function within our form. This function generates a hidden form input that Laravel recognizes and will use to override the actual HTTP request method. The generated field will look like the following:

    <input type="hidden" name="_method" value="DELETE">

<a name="route-model-binding"></a>
### Route Model Binding

Now, we're almost ready to define the `destroy` method on our `TaskController`. But, first, let's revisit our route declaration and controller method for this route:

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

Since the `{task}` variable in our route matches the `$task` variable defined in our controller method, Laravel's [implicit model binding](/docs/{{version}}/routing#route-model-binding) will automatically inject the corresponding Task model instance.

<a name="authorization"></a>
### Authorization

Now, we have a `Task` instance injected into our `destroy` method; however, we have no guarantee that the authenticated user actually "owns" the given task. For example, a malicious request could have been concocted in an attempt to delete another user's tasks by passing a random task ID to the `/tasks/{task}` URL. So, we need to use Laravel's authorization capabilities to make sure the authenticated user actually owns the `Task` instance that was injected into the route.

#### Creating A Policy

Laravel uses "policies" to organize authorization logic into simple, small classes. Typically, each policy corresponds to a model. So, let's create a `TaskPolicy` using the Artisan CLI, which will place the generated file in `app/Policies/TaskPolicy.php`:

    php artisan make:policy TaskPolicy

Next, let's add a `destroy` method to the policy. This method will receive a `User` instance and a `Task` instance. The method should simply check if the user's ID matches the `user_id` on the task. In fact, all policy methods should either return `true` or `false`:

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

Finally, we need to associate our `Task` model with our `TaskPolicy`. We can do this by adding a line in the `app/Providers/AuthServiceProvider.php` file's `$policies` property. This will inform Laravel which policy should be used whenever we try to authorize an action on a `Task` instance:

    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Task' => 'App\Policies\TaskPolicy',
    ];


#### Authorizing The Action

Now that our policy is written, let's use it in our `destroy` method. All Laravel controllers may call an `authorize` method, which is exposed by the `AuthorizesRequest` trait:

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

Let's examine this method call for a moment. The first argument passed to the `authorize` method is the name of the policy method we wish to call. The second argument is the model instance that is our current concern. Remember, we recently told Laravel that our `Task` model corresponds to our `TaskPolicy`, so the framework knows on which policy to fire the `destroy` method. The current user will automatically be sent to the policy method, so we do not need to manually pass it here.

If the action is authorized, our code will continue executing normally. However, if the action is not authorized (meaning the policy's `destroy` method returned `false`), a 403 exception will be thrown and an error page will be displayed to the user.

> **Note:** There are several other ways to interact with the authorization services Laravel provides. Be sure to browse the complete [authorization documentation](/docs/{{version}}/authorization).

<a name="deleting-the-task"></a>
### Deleting The Task

Finally, let's finish adding the logic to our `destroy` method to actually delete the given task. We can use Eloquent's `delete` method to delete the given model instance in the database. Once the record is deleted, we will redirect the user back to the `/tasks` URL:

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
