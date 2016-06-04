# Blade Templates

- [Giới thiệu](#introduction)
- [Kế thừa template](#template-inheritance)
    - [Tạo một layout](#defining-a-layout)
    - [Mở rộng một layout](#extending-a-layout)
- [Hiển thị data](#displaying-data)
- [Cấu trúc điều khiển](#control-structures)
- [Stacks](#stacks)
- [Service Injection](#service-injection)
- [Mở rộng Blade](#extending-blade)

<a name="introduction"></a>
## Giới thiệu

Blade là một templating engine đơn giản nhưng rất mạnh mẽ được tạo ra và đi cùng với Laravel. Không giống các templating engine khác, Blade không cấm bạn sử dụng PHP thuần trong view. Tất cả các views của Blade được compile thành mã PHP thuần và được cache lại cho tới khi bị chỉnh sửa, nghĩa là Blade về cơ bản không làm tăng thêm chi phí ban đầu nào trong ứng dụng. Các file của Blade view sử dụng đuôi là `.blade.php` và cơ bản được lưu trong thư mục `resources/views`.

<a name="template-inheritance"></a>
## Kế thừa template

<a name="defining-a-layout"></a>
### Tạo một layout

Hai lợi ích chính của việc sử dụng Blade là _kế thừa template_ và _sections_. Để bắt đầu, hãy cùng nhau xem ví dụ đơn giản dưới đây. Đầu tiên, chúng ta cùng xem một trang layout "master". Vì hầu hết các ứng dụng web bảo trì một mẫu layout chung giữa các trang với nhau, nên sẽ rất là tiện nếu tạo ra layout này thành một Blade view riêng biệt:

    <!-- Stored in resources/views/layouts/master.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

Như bạn thấy, file này có chứa mã HTML cơ bản. Tuy nhiên, hãy chú ý ở hai directive `@section` và `@yield`. Về `@section` directive, như cái tên đã thể hiện, thực hiện khai báo một khối dữ liệu, trong khi `@yield` lại được sử dụng để hiển thị dữ liệu ở một vị trí đặt trước.

Lúc này chúng ta đã tạo xong một layout, hãy cùng nhau tạo ra các trang con kế thừa từ layout này.

<a name="extending-a-layout"></a>
### Mở rộng layout

Khi tạo một trang con, bạn có thể sử dụng `@extends` để cho biết là layout của trang con này sẽ thực hiện "kế thừa" từ đâu. View mà `@extends` một Blade layout có thể inject nội dung vào trong mục `@section`. Nhớ lại ở ví dụ trước, nội dung của những section này sẽ được hiển thị sử dụng `@yield`:

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.master')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

Ở ví dụ này, phần `sidebar` có sử dụng `@@parent` để thực hiện thêm nội dung vào trong sidebar (thay vì ghi đè toàn bộ). `@@parent` sẽ được thay thế bởi nội dung của layout khi view được render.

Dĩ nhiên, tương tự như view PHP thuần, Blade view có thể được trả về từ route sử dụng hàm helper `view`:

    Route::get('blade', function () {
        return view('child');
    });

<a name="displaying-data"></a>
## Hiển thị data

Bạn có thể hiển thị data truyền vào trong Blade views bằng cách đặt biến vào trong cặp **dấu ngoặc nhọn**. Ví dụ, với route dưới đây:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

thì bạn có thể hiển thị nội dung của biến `name` như này:

    Hello, {{ $name }}.

Dĩ nhiên là bạn không hề bị giới hạn trong việc hiển thị nội dung của biến truyền vào trong view. Bạn cũng có thể hiển thị kết quả của bất cứ hàm PHP nào. Chính xác hơn, bạn có thể đặt bất cứ mã PHP nào bạn muốn vào trong một mệnh đề hiển thị của Blade:

    The current UNIX timestamp is {{ time() }}.

> **Chú ý:** Cặp dấu `{{ }}` của Blade được tự động gửi tới hàm `htmlentities` của PHP để ngăn chặn các hành vi tấn công XSS.

#### Blade & JavaScript Frameworks

Vì nhiều framework Javascript cũng sử dụng cặp dấu **ngoặc nhọn** để cho biết một biểu thức cần được hiển thị lên trình duyệt, bạn có thể sử dụng dấu `@` để báo cho Blade biết là biểu thức này cần được giữ lại. Ví dụ:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

Ở ví dụ này, kí hiệu `@` sẽ bị Blade xoá đi; vì thế `{{ name }}` sẽ được giữ lại và cho phép nó được render tiếp bởi Javascript khác của bạn.

#### Hiển thị dữ liệu nếu tồn tại

Đôi lúc bạn muốn hiển thị giá trị một biến, nhưng bạn không chắc nếu như biến được đã được set. Chúng ta có thể thể hiện theo kiểu mã nguồn PHP như này:

    {{ isset($name) ? $name : 'Default' }}

Tuy nhiên, thay vì viết thay kiểu ternary, Blade cung cấp bạn một cách ngắn gọn:

    {{ $name or 'Default' }}

Ở ví dụ này, nếu biến `$name` tồn tại, giá trị của nó sẽ được hiển thị. Còn nếu như nó không tồn tại, thì từ `Default` sẽ được hiển thị.

#### Hiển thị dữ liệu chưa unescape

Mặc định, cặp dấu `{{ }}` được tự động gửi qua hàm `htmlentities` để ngăn chặn tấn công XSS. Nếu bạn không muốn dữ liệu bị escape, bạn có thể sử dụng cú pháp sau:

    Hello, {!! $name !!}.

> **Chú ý:** Phải thật cẩn thận khi hiển thị nội dung được người dùng cung cấp. Luôn luôn sử dụng cặp dấu ngoặc nhọn để escape các thẻ HTML trong nội dung.

<a name="control-structures"></a>
## Cấu trúc điều khiển

Ngoài việc kế thừa template và hiển thị dữ liệu, Blade cũng cung cấp các tiện ích cho các cấu trúc điều khiển cơ bản của PHP, như mệnh đề điều kiện hay vòng lặp. Những tiện ích này rất hữu ích khi làm việc với cấu trúc điều khiển của PHP.

#### Sử dụng If

Bạn có thể sử dụng `if` với các directive `@if`, `@elseif`, `@else`, và `@endif`. Những directive này tương ứng giống hệt các từ khoá của PHP:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

Để tiện hơn, Blade cũng cung cấp điều kiện `@unless`:

    @unless (Auth::check())
        You are not signed in.
    @endunless

Bạn cũng có thể kiểm tra nếu một phần layout có chứa nội dung nào không sử dụng `@hasSection`:

    <title>
        @hasSection('title')
            @yield('title') - App Name
        @else
            App Name
        @endif
    </title>

#### Vòng lặp

Ngoài cấu trúc điều kiện, Blade cũng cung cấp các phương thức hỗ trợ cho việc xử lý vòng lặp, và các phần này tương tự với PHP:

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

Khi sử dụng vòng lặp, bạn có thể cần kết thúc hay bỏ qua vòng lặp hiện tại:

    @foreach ($users as $user)
        @if($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if($user->number == 5)
            @break
        @endif
    @endforeach

Bạn cũng có thể thêm điều kiện vào khai trên cùng một dòng:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

#### Chèn thêm view con

Sử dụng `@include`, cho phép bạn dễ dàng chèn thêm một Blade view vào trong một view có sẵn. Tất cả các biến sử dụng được ở view cha đều có thể sử dụng được ở view chèn thêm:

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

Mặc dù, các view được chèn thêm kế thừa tất cả dữ liệu từ view cha, bạn cũng có thể truyền vào một mảng các dữ liệu bổ sung:

    @include('view.name', ['some' => 'data'])

> **Chú ý:** Bạn nên tránh sử dụng `__DIR__` và `__FILE__` ở trong Blade view, vì chúng sẽ tham chiếu tới vị trí của file view bị cache.

#### Render views cho collection

Bạn có thể kết hợp vòng lặp và view chèn thêm trong một dòng với `@each`:

    @each('view.name', $jobs, 'job')

Tham số thứ nhất là tên của view partial để render các element trong mảng hay collection. Tham số thứ hai là mảng hay collection bạn muốn lặp, còn tham số thứ ba là tên của biến sẽ được gán vào trong vòng lặp bên trong view. Vì thế, nếu như bạn muốn lặp qua một mảng tên `jobs`, bạn sẽ cần phải truy xuất vào mỗi job thông qua biến tên là `job` ở bên trong view partial.

Bạn cũng có thể truyền vào tham số thứ tư vào trong `@each`. Tham số này chỉ định view sẽ được render nếu như mảng bị rỗng.

    @each('view.name', $jobs, 'job', 'view.empty')

#### Comments

Blade cũng cho phép bạn viết comments trong view. Tuy nhiên, không giống như HTML comment, Blade comment không đi kèm trong nội dung HTML được trả về:

    {{-- This comment will not be present in the rendered HTML --}}

<a name="stacks"></a>
## Stacks

Blade cũng cho phép bạn đẩy vào stack để có thể được render ở một vị trí nào trong view hay layout khác:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

Bạn có thể đẩy vào stack tuỳ ý bao nhiêu lần bạn muốn. Để render, bạn sử dụng `@stack`:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## Service Injection

Để nhúng vào một service trong Laravel [service container](/docs/{{version}}/container), sử dụng `@inject`. Tham số thứ nhất truyền vào là tên của biến mà service sẽ được đặt vào, còn tham số thứ hai là tên của class hay interface của service bạn muốn resolve:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Mở rộng Blade

Blade thậm chí cho phép bạn tạo directive riêng. Bạn có thể sử dụng hàm `directive` để đăng kí một directive. Khi trình biên dịch của Blade gặp directive, nó sẽ gọi tới callback được cung cấp với tham số tương ứng.

Ví dụ sau đây tạo một directive `@datetime($var)` để thực hiện format một biến `$var`:

    <?php

    namespace App\Providers;

    use Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function($expression) {
                return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Như bạn thấy, hàm helper `with` được sử dụng trong directive này. Hàm `with` đơn giản chỉ trả về đối tượng hay giá trị được cung cấp và cho phép móc nối hàm một cách tiện lợi. Mã PHP được sinh ra cuối cùng bởi directive này sẽ là:

    <?php echo with($var)->format('m/d/Y H:i'); ?>

Sau khi cập nhật logic của một Blade directive, bạn cần xoá hết tất cả các Blade views đã bị cache bằng cách sử dụng câu lệnh Artisan `view:clear`.
