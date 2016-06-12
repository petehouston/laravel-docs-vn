# Basic Task List

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Chuẩn bị CSDL](#prepping-the-database)
    - [Migrations](#database-migrations)
    - [Eloquent Models](#eloquent-models)
- [Routing](#routing)
    - [Chuẩn bị Routes](#stubbing-the-routes)
    - [Hiển thị một View](#displaying-a-view)
- [Xây dựng Layouts & Views](#building-layouts-and-views)
    - [Định nghĩa Layout](#defining-the-layout)
    - [Định nghĩa các View con](#defining-the-child-view)
- [Tạo Tasks](#adding-tasks)
    - [Validation](#validation)
    - [Tạo Task](#creating-the-task)
    - [Hiển thị các Tasks tồn tại](#displaying-existing-tasks)
- [Xóa Tasks](#deleting-tasks)
    - [Thêm nút xóa](#adding-the-delete-button)
    - [Xóa Task](#deleting-the-task)

<a name="introduction"></a>
## Giới thiệu

Quickstart là một bài hướng dẫn dẫn đơn giản để giới thiệu Laravel framework cùng với việc sử dụng nội dung từ CSDL, Eloquent ORM, routing, validation, views, và cả Blade templates. Thật tuyệt vời nếu bạn dùng Laravel framework hoặc PHP frameworks để thực hiện. Nếu bạn đã sử dụng Laravel hoặc PHP frameworks, thì chắc hẳn bạn đã sẳn sàng với những quickstarts nâng cao hơn.

Đây là một ví dụ cơ bản về tính năng của Laravel, chúng ta sẽ xây dựng hệ thống task list (tiến trình công việc) chúng ta có thể sử dụng để theo dõi tất cả các công việc mà chúng ta muốn đạt được. Nói các khác, ta có thể gọi là "to-do" list. Chúng tôi đã hoàn thành nó và lưu trữ sourse code [tại GitHub](https://github.com/laravel/quickstart-basic).

<a name="installation"></a>
## Cài đặt

#### Cài đặt Laravel

Hiển nhiên rồi, công việc đầu tiên mà chúng ta cần làm đó chính là cài đặt Laravel framework. Bạn có thể dùng [máy ảo Homestead](/docs/{{version}}/homestead) hoặc một môi trường PHP khác để sử dụng framework. Khi môi trường đã sẳn sàng, bạn có thể cài đặt framework bằng cách sử dụng Composer:

    composer create-project laravel/laravel quickstart --prefer-dist

#### Cài đặt Quickstart (tùy chọn)

Để tiết kiệm thời gian bạn có thể cài đặt luôn cái quickstart này; tất nhiên, nếu bạn muốn tải sourse code này và dùng nó trên môi trường của bạn, bạn hãy clone gói Git và cài đặt các thức cần thiết:

    git clone https://github.com/laravel/quickstart-basic quickstart
    cd quickstart
    composer install
    php artisan migrate

Và để xây dựng môi trường phát triền Laravel, bạn hãy xem bài viết [Homestead](/docs/{{version}}/homestead) và [hướng dẫn cài đặt](/docs/{{version}}/installation).

<a name="prepping-the-database"></a>
## Chuẩn bị CSDL

<a name="database-migrations"></a>
### Migrations

Tiếp theo, hãy tạo một bảng chứa tất cả các tasks. Laravel's database migrations cung cấp cho bạn cách đơn giản để xây dựng cấu trúc và tùy chỉnh CSDL cần thiết cho việc truy vấn, cùng như là code PHP. Thay vì nói cho các thành viên nhóm của bạn để tự thêm bảng và cột của họ vào cơ sở dữ liệu theo cách thông thường, đồng đội của bạn chỉ đơn giản là có thể chạy một lệnh cơ bản để kiểm soát nguồn dữ liệu ấy.

Nào, chúng ta sẽ bắt đầu tạo bản chứa các tasks. [Artisan CLI](/docs/{{version}}/artisan) được dùng để tạo các class cần thiết phục vụ cho dự án Laravel. Trong trường hợp này, hãy sử dụng lệnh `make:migration` để tạo ra một class database migration cho bản `tasks`:

    php artisan make:migration create_tasks_table --create=tasks

Migration sẽ chứa tại thư mục `database/migrations` của dự án của bạn. Bạn nên biết là, lệnh `make:migration` tự động thêm các cột auto-incrementing ID và timestamps vào trong tập tin migration. Nào cùng thêm vào đó một cột dạng `string` dùng để chứa tên của các tasks:

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

Để chạy migration, chúng ta sử dụng lệnh `migrate` trong Artisan. Nếu bạn đang dùng Homestead, bạn sẽ rất dễ dàng sử dụng lệnh này, nếu ở môi trường khác nó sẽ kết nối trực tiếp đến CSDL:

    php artisan migrate

Lệnh này sẽ tạo ra tất cả các bảng CSDL. Nếu bạn dùng CSDL của khác hàng, bạn nên kiểm tra bảng `tasks` để định nghĩa lại migration cho đúng. Tiếp theo, chúg ta sẽ bắt đầu tạo Eloquent ORM model cho tasks của chúng ta!

<a name="eloquent-models"></a>
### Eloquent Models

[Eloquent](/docs/{{version}}/eloquent) là một Laravel's default ORM (object-relational mapper). Eloquent sẽ tạo ra cách truy vấn và thao tác với CSDL một cách dễ dàng hay được gọi là các "models". Thường thì, mỗi một Eloquent model chỉ kết nối đến một bảng duy nhất.

Chúng ta bắt đầu tạo ra một `Task` model để kết nối tới bảng `tasks` mà chúng ta đã tạo trước đó. Thêm một lần nữa, chúng ta sẽ sử dụng các lệnh Artisan để tạo ra model. Trong trường hợp này, ta sẽ dùng lệnh `make:model`:

    php artisan make:model Task

Model này sẽ chứa tại thư mục `app`. Mặc định, class model sẽ trống. Chúng ta không cần phải nói cho rõ ràng mô hình mà bảng tương ứng với nó bởi vì nó sẽ tự động dùng tên của Model để kết nối đến CSDL. Nào, với trường hợp này thì `Task` model sẽ tự động kết nối với bảng `tasks`. Và đây là khi model vừa tạo:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Task extends Model
    {
        //
    }

Chúng ta sẽ học cách sử dụng Eloquent models as we add routes to our application. Of course, feel free to consult the [complete Eloquent documentation](/docs/{{version}}/eloquent) for more information.

<a name="routing"></a>
## Routing

<a name="stubbing-the-routes"></a>
### Stubbing The Routes

Next, we're ready to add a few routes to our application. Routes are used to point URLs to controllers or anonymous functions that should be executed when a user accesses a given page. By default, all Laravel routes are defined in the `app/Http/routes.php` file that is included in every new project.

For this application, we know we will need at least three routes: a route to display a list of all of our tasks, a route to add new tasks, and a route to delete existing tasks. So, let's stub all of these routes in the `app/Http/routes.php` file:

    <?php

    use App\Task;
    use Illuminate\Http\Request;

    /**
     * Show Task Dashboard
     */
    Route::get('/', function () {
        //
    });

    /**
     * Add New Task
     */
    Route::post('/task', function (Request $request) {
        //
    });

    /**
     * Delete Task
     */
    Route::delete('/task/{task}', function (Task $task) {
        //
    });

> **Note**: If your copy of Laravel has a `RouteServiceProvider` that already includes the default routes file within the `web` middleware group, you do not need to manually add the group to your `routes.php` file.

<a name="displaying-a-view"></a>
### Displaying A View

Next, let's fill out our `/` route. From this route, we want to render an HTML template that contains a form to add new tasks, as well as a list of all current tasks.

In Laravel, all HTML templates are stored in the `resources/views` directory, and we can use the `view` helper to return one of these templates from our route:

    Route::get('/', function () {
        return view('tasks');
    });

Passing `tasks` to the `view` function will create a View object instance that corresponds to the template in `resources/views/tasks.blade.php`. Of course, we need to actually define this view, so let's do that now!

<a name="building-layouts-and-views"></a>
## Building Layouts & Views

This application only has a single view which contains a form for adding new tasks as well as a listing of all current tasks. To help you visualize the view, here is a screenshot of the finished application with basic Bootstrap CSS styling applied:

![Application Image](https://laravel.com/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### Defining The Layout

Almost all web applications share the same layout across pages. For example, this application has a top navigation bar that would be typically present on every page (if we had more than one). Laravel makes it easy to share these common features across every page using Blade **layouts**.

As we discussed earlier, all Laravel views are stored in `resources/views`. So, let's define a new layout view in `resources/views/layouts/app.blade.php`. The `.blade.php` extension instructs the framework to use the [Blade templating engine](/docs/{{version}}/blade) to render the view. Of course, you may use plain PHP templates with Laravel. However, Blade provides convenient short-cuts for writing clean, terse templates.

Our `app.blade.php` view should look like the following:

    <!-- resources/views/layouts/app.blade.php -->

    <!DOCTYPE html>
    <html lang="en">
        <head>
            <title>Laravel Quickstart - Basic</title>

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

Next, we need to define a view that contains a form to create a new task as well as a table that lists all existing tasks. Let's define this view in `resources/views/tasks.blade.php`.

We'll skip over some of the Bootstrap CSS boilerplate and only focus on the things that matter. Remember, you can download the full source for this application on [GitHub](https://github.com/laravel/quickstart-basic):

    <!-- resources/views/tasks.blade.php -->

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
                    <label for="task" class="col-sm-3 control-label">Task</label>

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

Before moving on, let's talk about this template a bit. First, the `@extends` directive informs Blade that we are using the layout we defined in `resources/views/layouts/app.blade.php`. All of the content between `@section('content')` and `@endsection` will be injected into the location of the `@yield('content')` directive within the `app.blade.php` layout.

The `@include('common.errors')` directive will load the template located at `resources/views/common/errors.blade.php`. We haven't defined this template, but we will soon!

Now we have defined a basic layout and view for our application. Remember, we are returning this view from our `/` route like so:

    Route::get('/', function () {
        return view('tasks');
    });

Next, we're ready to add code to our `POST /task` route to handle the incoming form input and add a new task to the database.

<a name="adding-tasks"></a>
## Adding Tasks

<a name="validation"></a>
### Validation

Now that we have a form in our view, we need to add code to our `POST /task` route in `app/Http/routes.php` to validate the incoming form input and create a new task. First, let's validate the input.

For this form, we will make the `name` field required and state that it must contain less than `255` characters. If the validation fails, we will redirect the user back to the `/` URL, as well as flash the old input and errors into the [session](/docs/{{version}}/session). Flashing the input into the session will allow us to maintain the user's input even when there are validation errors:

    Route::post('/task', function (Request $request) {
        $validator = Validator::make($request->all(), [
            'name' => 'required|max:255',
        ]);

        if ($validator->fails()) {
            return redirect('/')
                ->withInput()
                ->withErrors($validator);
        }

        // Create The Task...
    });

#### The `$errors` Variable

Let's take a break for a moment to talk about the `->withErrors($validator)` portion of this example. The `->withErrors($validator)` call will flash the errors from the given validator instance into the session so that they can be accessed via the `$errors` variable in our view.

Remember that we used the `@include('common.errors')` directive within our view to render the form's validation errors. The `common.errors` will allow us to easily show validation errors in the same format across all of our pages. Let's define the contents of this view now:

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
### Tạo ra

Now that input validation is handled, let's actually create a new task by continuing to fill out our route. Once the new task has been created, we will redirect the user back to the `/` URL. To create the task, we may use the `save` method after creating and setting properties on a new Eloquent model:

    Route::post('/task', function (Request $request) {
        $validator = Validator::make($request->all(), [
            'name' => 'required|max:255',
        ]);

        if ($validator->fails()) {
            return redirect('/')
                ->withInput()
                ->withErrors($validator);
        }

        $task = new Task;
        $task->name = $request->name;
        $task->save();

        return redirect('/');
    });

Great! We can now successfully create tasks. Next, let's continue adding to our view by building a list of all existing tasks.

<a name="displaying-existing-tasks"></a>
### Displaying Existing Tasks

First, we need to edit our `/` route to pass all of the existing tasks to the view. The `view` function accepts a second argument which is an array of data that will be made available to the view, where each key in the array will become a variable within the view:

    Route::get('/', function () {
        $tasks = Task::orderBy('created_at', 'asc')->get();

        return view('tasks', [
            'tasks' => $tasks
        ]);
    });

Once the data is passed, we can spin through the tasks in our `tasks.blade.php` view and display them in a table. The `@foreach` Blade construct allows us to write concise loops that compile down into blazing fast plain PHP code:

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

Vậy là dự án của chúng ta sắp hoàn thành rồi đấy. Nhưng, hình như chúng ta không có cách nào xóa cá task này, yên tâm chúng ta sẽ làm việc đó vào bước tiếp theo!

<a name="deleting-tasks"></a>
## Xóa Tasks

<a name="adding-the-delete-button"></a>
### Thêm nút xóa

Chúng ta đã xong việc với "TODO" và cần một nút xóa là cần thiết. Nào, chúng ta sẽ thêm nút xóa vào view `tasks.blade.php`. Cúng ta sẽ gắn vào đó một nút nhỏ. Và khi chúng ta bấm vào thì, một yêu cầu `DELETE /task` sẽ được gửi tới ứng dụng:

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

                <button type="submit" class="btn btn-danger">
                    <i class="fa fa-trash"></i> Delete
                </button>
            </form>
        </td>
    </tr>

<a name="a-note-on-method-spoofing"></a>
#### Lưu ý mở phương thức giả

Một vài lưu ý khi sử dụng gửi dữ liệu bằng `method` với giá trị là `POST`, và nếu chúng ta gọi route bằng cách `Route::delete` sẽ không dùng được. HTML forms chỉ cho phép hai phương thức gửi đó là dạng `GET` và `POST`, vì vậy chúng ta nên tạo một phương thức`DELETE` giả cho form.

Chúng ta sử dụng phương thức`DELETE` yêu cầu của các kết quả xuất ra từ hàm `method_field('DELETE')`. hàm này sẽ tạo ra một trường nhập ẩn của Laravel để thay thế một định dạng method khác trên HTTP. Nó sẽ tạo ra một chuỗi như thế này:

    <input type="hidden" name="_method" value="DELETE">

<a name="deleting-the-task"></a>
### Xóa Task

Cuối cùng, hãy tạo ra một route để thực việc xóa đi task. Chúng ta có thể sử dụng [implicit model binding](/docs/{{version}}/routing#route-model-binding) để tự động lấy `Task` tương ứng với các tham số của`{task}` route.

Và đây, chúng ta sẽ dùng phương thức `delete` để xóa các cột. Sau khi task được xóa, chúng ta sẽ chuyển hướng về URL `/`:

    Route::delete('/task/{task}', function (Task $task) {
        $task->delete();

        return redirect('/');
    });
