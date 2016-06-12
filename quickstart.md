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

Chúng ta sẽ học cách sử dụng Eloquent models cùng với thêm các routes cho ứng dụng này. Tất nhiên, bạn cần phải đoc [hoàn tất tài liệu Eloquent](/docs/{{version}}/eloquent) để hiểu hơn về nó.

<a name="routing"></a>
## Routing

<a name="stubbing-the-routes"></a>
### Chuẩn bị Routes

Tiếp theo, chúng ta đã sẳn sàng thêm các routes cho ứng dụng này. Routes sử dụng các tham chiếu của URLs trỏ đến các controllers hoặc các hàm không định danh thực hiện các yêu cầu và xuất ra view cụ thể. Theo mặc định, Laravel routes được định nghĩa ở tập tin `app/Http/routes.php`.

Trong ứng dụng này, chúng ta cần ba routes chính đó là: route hiển thị tất cả các tasks, route tạo tasks, và route để xóa tasks. Nào chúng ta cùng định nghĩa chúng trong `app/Http/routes.php`:

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

> **Lưu ý**: Các dự án của bạn có sẵn `RouteServiceProvider` thì chúng đã bao gồm `web` middleware group, thế nên bạn không cần thêm chúng vào trong `routes.php`.

<a name="displaying-a-view"></a>
### Hiển thị một View

Tiếp tục, chúng ta cần truyền vào `/` vào một view. Với các route, chúng ta sẽ tạo ra một HTML template nơi hiển thị các trường nhập tạo task, hay hiển thị các task tồn tại.

Trong Laravel, tất cả các HTML templates được lưu tại `resources/views`, chúng dễ dàng gọi chúng bằng các gọi hàm `view`:

    Route::get('/', function () {
        return view('tasks');
    });

Khi truyền `tasks` vào hàm `view` chúng sẽ gọi các thành phần từ `resources/views/tasks.blade.php`. Tất nhiên, chúng ta cần phải định nghĩa các view trước, và bây giờ chúng ta sẽ thực hiện điều đó!

<a name="building-layouts-and-views"></a>
## Xây dựng Layouts & Views

Chúng ta chỉ cần hai thành phần view chính đó là view hiển thị các task tồn tại và view dùng để tạo các task. Để dễ hình dung, đây là một bức ảnh chụp một view khi hoàn thành đã được làm đẹp bằng Bootstrap CSS:

![Application Image](https://laravel.com/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### Đinh nghĩa Layout

Hầu hết các ứng dụng đều có một layout và các trang con được kế thừa từ chúng. Ví dụ, một ứng dụng cần có một thanh menu ở phía trên và ta muốn có ở mọi trang mà không cần phải thêm chúng ở nhiều trang. Laravel dễ dàng thực hiện việc này bằng các Blade **layouts**.

Như các chúng ta đã thảo luận trước đó, tất cả các Laravel views được lưu trữ tại `resources/views`. Vì thế, ta tiến hành tạo một layout mới `resources/views/layouts/app.blade.php`. Đuôi mở rộng `.blade.php` được giải thích ở [Blade templating engine](/docs/{{version}}/blade) nếu bạn không hiểu hãy đọc chúng. Và tất nhiên, chúng ta cũng có thể sử dụng các mã PHP thuần bên trong các view Laravel. Tuy vậy, Blade đã cung câp cho ta cách viết các view đơn giản và đẹp hơn, gọn gàng hơn trong các templates.

`app.blade.php` sẽ trông như thế này:

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

Chú ý các `@yield('content')` ở trong layout. là các thẻ Blade nơi mà tất cả các trang con kế thừa từ chúng có thể hiển thị các nội dung mong muốn ở các phần khác nhau. Tiếp tục, chúng ta sẽ định nghĩa các view con để hiển thị các task tồn tại cũng như việc tạo task.

<a name="defining-the-child-view"></a>
### Định nghĩa các view con

Nào, chúng ta hãy tạo view để hiển thị các task tồn tại. Chúng ta sẽ định nghĩa chúng bên trong `resources/views/tasks.blade.php`.

Chúng ta sẽ bỏ qua bước cài đặt Bootstrap CSS vì không cần thiết. Hãy nhớ rằng, bạn dễ dàng tải về toàn bộ dự án này ở [GitHub](https://github.com/laravel/quickstart-basic):

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

#### Một vài ghi chú

Trước khi qua bước tiếp theo, chúng ta sẽ nói về template. Đầu tiên, thẻ `@extends` sẽ định nghĩa Blade hiện tại sẽ kế thừa từ Blade chính tức là layout `resources/views/layouts/app.blade.php`. Các thẻ tiếp theo chính là `@section('content')` và `@endsection` chúng sẽ truyền đến các bị trí `@yield('content')` trên Blade `app.blade.php` layout.

Với `@include('common.errors')` chúng sẽ chèn trực tiếp tập tin `resources/views/common/errors.blade.php`. Chúng ta chưa nói về phần này, nhưng bạn sẽ sớm biết thôi!

Bây giờ chúng ta đã hoàn tất định nghĩa các view và layout. Hãy nhớ rằng, chúng ta trả về view từ route `/` giống như thế này:

    Route::get('/', function () {
        return view('tasks');
    });

Tiếp theo, chúng ta sẵn sàng chỉnh sửa `POST /task` route để chuẩn bị việc thêm các task đế thêm chúng vào CSDL.

<a name="adding-tasks"></a>
## Tạo Tasks

<a name="validation"></a>
### Validation

Chúng ta đã có các trường nhập cho view, chúng ta cần thêm phương thức `POST /task` vào bên trong `app/Http/routes.php` và kiểm tra các ràng buộc của các trường nhập trước khi thêm chúng vào CSDL.

Với các trường nhập này, chúng ta muốn trường `name` phải bắt buộc nhập và tồn tại ít nhất `255` ký tự. Nếu không đáp ứng đúng điều kiện đưa ra, Chúng ta sẽ chuyển hướng về `/`, cùng với đó sẽ thêm vào [session](/docs/{{version}}/session) các thông báo lỗi dưới dạng flash. Chúng ta dễ dàng thực hiện việc đó như sau:

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

#### Biến `$errors`

Chúng nắm bắt các lỗi bằng phương thức `->withErrors($validator)`. Và `->withErrors($validator)` sẽ truyền vào flash lỗi (đã đề cập ở bài Session) chúng sẽ truyền các thông báo lỗi vào trong biến `$errors` của view.

Hãy nhớ rằng hãy sử dụng `@include('common.errors')` để đính kèm vào bên trong các view chính cần hiển thị lỗi. `common.errors` sẽ giúp ta tách ra dễ dàng phần hiển thị các thông báo lỗi ở các view cần thiết mà không cần phải lập đi lập lại nhiều lần. Và tập tin ấy sẽ trông như thế này:

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


> **Lưu ý:** Biến `$errors` xuất hiện **thường xuyên** trong Laravel view. Trường hợp biến `ViewErrorBag` rỗng nếu như không có bất kỳ lỗi gì xảy ra.

<a name="creating-the-task"></a>
### Tạo Task

Bây giờ các trường nhập đã được ràng buộc, chúng ta đã sẳn sàng tạo ra Task bằng cách sử dụng route. Một khi task mới được tạo ra, chúng ta sẽ chuyển hướng chúng đến URL `/`. Để tạo ra Task, chúng ta dùng phương thức `save` bằng cách tạo ra một đối tượng Eloquent model mới rồi truyền vào đó các thuộc tính cần thiết:

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

Tuyệt! Chúng ta đã tạo ra tasks thành công. Kê tiếp, chúng ta sẽ hiển thị các task tồn tại.

<a name="displaying-existing-tasks"></a>
### Hiển thị các Tasks tồn tại

Đầu tiên, ta cần chỉnh sửa route `/` để chuyển tất cả dữ liệu vào view. hàm `view` có sẵn hai tham số và ta sẽ truyền dữ liệu vào tham số thứ hai, chúng là biến dữ liệu của chúng ta:

    Route::get('/', function () {
        $tasks = Task::orderBy('created_at', 'asc')->get();

        return view('tasks', [
            'tasks' => $tasks
        ]);
    });

Một khi dữ liệu được truyền, chúng ta quay lại `tasks.blade.php` và hiển thị chúng dưới dạng bảng. `@foreach` Blade cho phép chúng ta hiển thị dữ liệu theo vòng lập, các mảng và ta có PHP code như sau:

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
