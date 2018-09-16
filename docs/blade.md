# Blade Templates

- [Blade Templates](#blade-templates)
  - [Giới thiệu](#gi%E1%BB%9Bi-thi%E1%BB%87u)
  - [Kế thừa template](#k%E1%BA%BF-th%E1%BB%ABa-template)
    - [Tạo một layout](#t%E1%BA%A1o-m%E1%BB%99t-layout)
    - [Mở rộng layout](#m%E1%BB%9F-r%E1%BB%99ng-layout)
  - [Components & Slots](#components-slots)
  - [Hiển thị dữ liệu](#hi%E1%BB%83n-th%E1%BB%8B-d%E1%BB%AF-li%E1%BB%87u)
    - [Hiện dữ liệu chưa Unescaped](#hi%E1%BB%87n-d%E1%BB%AF-li%E1%BB%87u-ch%C6%B0a-unescaped)
    - [Hiển thị dữ liệu nếu tồn tại](#hi%E1%BB%83n-th%E1%BB%8B-d%E1%BB%AF-li%E1%BB%87u-n%E1%BA%BFu-t%E1%BB%93n-t%E1%BA%A1i)
    - [Hiện thị Json](#hi%E1%BB%87n-th%E1%BB%8B-json)
    - [HTML Entity Encoding](#html-entity-encoding)
  - [Blade & JavaScript Frameworks](#blade-javascript-frameworks)
    - [Chỉ thị ``@verbatim``](#ch%E1%BB%89-th%E1%BB%8B-verbatim)
  - [Cấu trúc điều khiển](#c%E1%BA%A5u-tr%C3%BAc-%C4%91i%E1%BB%81u-khi%E1%BB%83n)
    - [Cấu trúc điều kiện](#c%E1%BA%A5u-tr%C3%BAc-%C4%91i%E1%BB%81u-ki%E1%BB%87n)
      - [Section Directives](#section-directives)
      - [Chỉ thị xác thực](#ch%E1%BB%89-th%E1%BB%8B-x%C3%A1c-th%E1%BB%B1c)
    - [Điều kiện Switch](#%C4%91i%E1%BB%81u-ki%E1%BB%87n-switch)
    - [Vòng lặp](#v%C3%B2ng-l%E1%BA%B7p)
    - [Biến vòng lặp](#bi%E1%BA%BFn-v%C3%B2ng-l%E1%BA%B7p)
    - [Comments](#comments)
    - [PHP](#php)
  - [Chèn thêm view con](#ch%C3%A8n-th%C3%AAm-view-con)
    - [Render views cho collection](#render-views-cho-collection)
  - [Stacks](#stacks)
  - [Service Injection](#service-injection)
  - [Mở rộng Blade](#m%E1%BB%9F-r%E1%BB%99ng-blade)
    - [Tùy chỉnh điều kiện if](#t%C3%B9y-ch%E1%BB%89nh-%C4%91i%E1%BB%81u-ki%E1%BB%87n-if)

## Giới thiệu

Blade là một templating engine đơn giản nhưng rất mạnh mẽ được tạo ra và đi cùng với Laravel. Không giống các templating engine khác, Blade không cấm bạn sử dụng PHP thuần trong view. Tất cả các views của Blade được biên dịch thành mã PHP thuần và được lưu trữ trong cache cho tới khi bị chỉnh sửa, nghĩa là Blade về cơ bản không làm tăng thêm chi phí ban đầu nào trong ứng dụng. Các file của Blade view sử dụng đuôi là `.blade.php` và cơ bản được lưu trong thư mục `resources/views`.

## Kế thừa template

### Tạo một layout

Hai lợi ích chính của việc sử dụng Blade là _kế thừa template_ và _sections_. Để bắt đầu, hãy cùng nhau xem ví dụ đơn giản dưới đây.

Đầu tiên, chúng ta cùng xem một trang layout "master". Vì hầu hết các ứng dụng web bảo trì một mẫu layout chung giữa các trang với nhau, nên sẽ rất là tiện nếu tạo ra layout này thành một Blade view riêng biệt:

```php
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
```

Như bạn thấy, file này có chứa mã HTML cơ bản. Tuy nhiên, hãy chú ý ở hai chỉ thị`@section` và `@yield`. Về `@section` , như cái tên đã thể hiện, thực hiện khai báo một khối dữ liệu, trong khi `@yield` lại được sử dụng để hiển thị dữ liệu ở một vị trí đặt trước.

Lúc này chúng ta đã tạo xong một layout, hãy cùng nhau tạo ra các trang con kế thừa từ layout này.

### Mở rộng layout

Khi tạo một trang con, bạn có thể sử dụng `@extends` để cho biết là layout của trang con này sẽ thực hiện "kế thừa" từ đâu. View mà `@extends` một Blade layout có thể inject nội dung vào trong mục `@section`. Nhớ lại ở ví dụ trước, nội dung của những section này sẽ được hiển thị sử dụng `@yield`:

```PHP
<!-- Stored in resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

Ở ví dụ này, phần `sidebar` có sử dụng `@parent` để thực hiện thêm nội dung vào trong sidebar (thay vì ghi đè toàn bộ). `@parent` sẽ được thay thế bởi nội dung của layout khi view được render.

>Khác với ví dụ trước, phần ``sidebar`` kết thúc với ``@endsection`` thay vì ``@show``. ``@endsection`` chỉ định nghĩa phần được chọn trong khi ``@show`` sẽ định nghĩa và hiện thị ngay phần đó

Dĩ nhiên, tương tự như view PHP thuần, Blade view có thể được trả về từ route sử dụng `view`:

```PHP
Route::get('blade', function () {
        return view('child');
});
```

## Components & Slots

## Hiển thị dữ liệu

Bạn có thể hiển thị dữ liệu truyền vào trong Blade views bằng cách đặt biến vào trong cặp **dấu ngoặc nhọn**. Ví dụ, với route dưới đây:

```php
Route::get('greeting', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

thì bạn có thể hiển thị nội dung của biến `name` như này:

```PHP
Hello, {{ $name }}.
```

Dĩ nhiên là bạn không hề bị giới hạn trong việc hiển thị nội dung của biến truyền vào trong view. Bạn cũng có thể hiển thị kết quả của bất cứ hàm PHP nào. Chính xác hơn, bạn có thể đặt bất cứ mã PHP nào bạn muốn vào trong một mệnh đề hiển thị của Blade:

```PHP
The current UNIX timestamp is {{ time() }}.
```

> **Chú ý:** Cặp dấu `{{ }}` của Blade được tự động gửi tới hàm `htmlentities` của PHP để ngăn chặn các hành vi tấn công XSS.

### Hiện dữ liệu chưa Unescaped

Mặc định, cặp {{ }} được tự động gửi qua hàm htmlentities của PHP để ngăn chặn tấn công XSS. Nếu bạn không muốn dự liệu bị escaped,bạn có thể sử dụng cú pháp:

```php
Hello, {!! $name !!}.
```

### Hiển thị dữ liệu nếu tồn tại

Thỉnh thoảng bạn muốn hiện giá trị một biến, nhưng bạn không chắc nếu biết đó có giá trị.Chúng ta có thể thể hiện theo kiểu code PHP như sau:

```php
{{ isset($name) ? $name : 'Default' }}
```

Tuy nhiên, thay vì viết kiểu ternary, Blade provides cung cấp cho bạn một cách ngắn gọn hơn:

```php
{{ $name or 'Default' }}
```

Trong ví dụ trên, nếu biến $name tồn tại, giá trị sẽ được hiện thị. Tuy nhiên, nếu nó không tồn tại, Từ Default sẽ được hiển thị.

### Hiện thị Json

Đôi lúc bạn truyền một mảng lưu dưới dạng json và muốn chuyển nó dạng biến của javascrip, chúng ta có thể thực hiện như sau

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

Tuy nhiên thay vì gọi ``json_encode``, bạn có thể sử dụng ``@json`` thay thế

```PHP
<script>
    var app = @json($array);
</script>
```

### HTML Entity Encoding

Theo mặc định, Blade (và Laravel ``e`` helper) sẽ mã hóa các thực thể html. Nếu bạn muốn bỏ, gọi phương thức ``Bale:Blade::withoutDoubleEncoding`` ở trong phương thức ``boot`` của ``AppServiceProvider``:

```PHP
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::withoutDoubleEncoding();
    }
}
```

## Blade & JavaScript Frameworks

Vì nhiều framework Javascript cũng sử dụng cặp dấu **ngoặc nhọn** để cho biết một biểu thức cần được hiển thị lên trình duyệt, bạn có thể sử dụng dấu `@` để báo cho Blade biết là biểu thức này cần được giữ lại. Ví dụ:

```PHP
<h1>Laravel</h1>

Hello, @{{ name }}.
```

Ở ví dụ này, kí hiệu `@` sẽ bị Blade xoá đi; vì thế `{{ name }}` sẽ được giữ lại và cho phép nó được render tiếp bởi Javascript khác của bạn.

### Chỉ thị ``@verbatim``

Nếu bạn muốn hiển thị biến JavaScript trong phần lớn template của bạn, bạn có thể bọc chúng trong  ``@verbatim`` khi đó bạn sẽ không cần tiền tố ``@`` trước biểu thức điều kiện:

```PHP
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## Cấu trúc điều khiển

Ngoài việc kế thừa template và hiển thị dữ liệu, Blade cũng cung cấp các tiện ích cho các cấu trúc điều khiển cơ bản của PHP, như mệnh đề điều kiện hay vòng lặp. Những tiện ích này rất hữu ích khi làm việc với cấu trúc điều khiển của PHP.

### Cấu trúc điều kiện

Bạn có thể sử dụng `if` với các directive `@if`, `@elseif`, `@else`, và `@endif`. Những directive này tương ứng giống hệt các từ khoá của PHP:

```php
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Để tiện hơn, Blade cũng cung cấp điều kiện `@unless`:

```php
@unless (Auth::check())
    You are not signed in.
@endunless
```

Ngoài các điều kiện đã nói trên, còn có ``@isset`` và ``@empty`` có thể sử đụng:

```PHP
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

#### Section Directives

Bạn cũng có thể kiểm tra nếu một phần layout có chứa nội dung nào không sử dụng `@hasSection`:

```php
<title>
    @hasSection('title')
        @yield('title') - App Name
    @else
        App Name
    @endif
</title>
```

#### Chỉ thị xác thực

Chỉ thị ``@auth`` và ``@guest`` có thể dùng để kiểm tra người dùng hiện giờ là khách hay là đã đăng ký

```PHP
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

Nếu cần, bạn có thể chỉ định[bảo vệ xác thực](authentication.md) cần được kiểm tra khi sử dụng @auth và  @guest:

```PHP
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

### Điều kiện Switch

Có thể sử dụng bằng các chỉ thị @swtich,@case,@break,@default,@endswitch

```php
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

### Vòng lặp

Ngoài cấu trúc điều kiện, Blade cũng cung cấp các phương thức hỗ trợ cho việc xử lý vòng lặp, và các phần này tương tự với PHP:

```PHP
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
```

Khi sử dụng vòng lặp, bạn có thể cần kết thúc hay bỏ qua vòng lặp hiện tại:

```php
@foreach ($users as $user)
    @if($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if($user->number == 5)
        @break
    @endif
@endforeach
```

Bạn cũng có thể thêm điều kiện vào khai trên cùng một dòng:

```php
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

### Biến vòng lặp

Trong vòng lặp, một biến ``$loop`` sẽ tồn tại bên trong vòng lặp. Biến này cho phép ta truy cập một số thông tin hữu ích của vòng lặp như index của vòng lặp hiện tại và vòng lặp đầu hoặc vòng lặp cuối của nó:

```php
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

Nếu bạn có vòng lặp lồng nhau, bạn có thể truy cập biên ``$loop`` của vòng lặp tra qua thuộc tính ``parent``:

```php
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

Biến $loopcòn chứa một số thông tin hữu ích:

| Thuộc tính       | Miêu tả                                              |
| ---------------- | ---------------------------------------------------- |
| $loop->index     | Chỉ số index hiện tại của vòng lặp (starts at 0).    |
| $loop->iteration | Các vòng lặp hiện tại (starts at 1).                 |
| $loop->remaining | Số vòng lặp còn lại.                                 |
| $loop->count     | Tổng số vòng lặp.                                    |
| $loop->first     | Vòng lặp đầu tiên.                                   |
| $loop->last      | Vòng lặp cuối cùng.                                  |
| $loop->depth     | Độ sâu của vòng lặp hiện tại.                        |
| $loop->parent    | Biến parent loop của vòng lặp trong 1 vòng lặp lồng. |

### Comments

Blade cũng cho phép bạn viết comments trong view. Tuy nhiên, không giống như HTML comment, Blade comment không đi kèm trong nội dung HTML được trả về:

```PHP
{{-- This comment will not be present in the rendered HTML --}}
```

### PHP

Trong một số trường hợp, chèn PHP code vào view sẽ có ích hơn, Bạn có thể sử dụng chỉ thị của blade ``@php`` để thực hiện một khối lệnh php đơn giản trong mẫu của bạn:

```PHP
@php
//
@endphp
```

## Chèn thêm view con

Sử dụng `@include`, cho phép bạn dễ dàng chèn thêm một Blade view vào trong một view có sẵn. Tất cả các biến sử dụng được ở view cha đều có thể sử dụng được ở view chèn thêm:

```php
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Mặc dù, các view được chèn thêm kế thừa tất cả dữ liệu từ view cha, bạn cũng có thể truyền vào một mảng các dữ liệu bổ sung:

```php
@include('view.name', ['some' => 'data'])
```

> **Chú ý:** Bạn nên tránh sử dụng `__DIR__` và `__FILE__` ở trong Blade view, vì chúng sẽ tham chiếu tới vị trí của file view bị cache.

### Render views cho collection

Bạn có thể kết hợp vòng lặp và view chèn thêm trong một dòng với `@each`:

```php
@each('view.name', $jobs, 'job')
```

Tham số thứ nhất là tên của view partial để render các element trong mảng hay collection. Tham số thứ hai là mảng hay collection bạn muốn lặp, còn tham số thứ ba là tên của biến sẽ được gán vào trong vòng lặp bên trong view. Vì thế, nếu như bạn muốn lặp qua một mảng tên `jobs`, bạn sẽ cần phải truy xuất vào mỗi job thông qua biến tên là `job` ở bên trong view partial.

Bạn cũng có thể truyền vào tham số thứ tư vào trong `@each`. Tham số này chỉ định view sẽ được render nếu như mảng bị rỗng.

```php
@each('view.name', $jobs, 'job', 'view.empty')
```

## Stacks

Blade cũng cho phép bạn đẩy vào stack để có thể được render ở một vị trí nào trong view hay layout khác:

```php
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

Bạn có thể đẩy vào stack tuỳ ý bao nhiêu lần bạn muốn. Để render, bạn sử dụng `@stack`:

```PHP
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

Nếu bạn muốn thềm vào phần đầu của stack, bạn có thể sự dụng chỉ thị ``@prepend``

```PHP
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

## Service Injection

Để nhúng vào một service trong Laravel [service container](container.md), sử dụng `@inject`. Tham số thứ nhất truyền vào là tên của biến mà service sẽ được đặt vào, còn tham số thứ hai là tên của class hay interface của service bạn muốn resolve:

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

## Mở rộng Blade

Blade thậm chí cho phép bạn tạo directive riêng. Bạn có thể sử dụng hàm `directive` để đăng kí một directive. Khi trình biên dịch của Blade gặp directive, nó sẽ gọi tới callback được cung cấp với tham số tương ứng.

Ví dụ sau đây tạo một directive `@datetime($var)` để thực hiện format một biến `$var`:

```php
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
```

Như bạn thấy, hàm helper `with` được sử dụng trong directive này. Hàm `with` đơn giản chỉ trả về đối tượng hay giá trị được cung cấp và cho phép móc nối hàm một cách tiện lợi. Mã PHP được sinh ra cuối cùng bởi directive này sẽ là:

```php
<?php echo with($var)->format('m/d/Y H:i'); ?>
```

>Sau khi cập nhật logic của một Blade directive, bạn cần xoá hết tất cả các Blade views đã bị cache bằng cách sử dụng câu lệnh Artisan `view:clear`.

### Tùy chỉnh điều kiện if

Lập trình tùy chỉnh chỉ thị đôi khi phức tạp hơn mức cần thiết khi xác định các câu lệnh điều kiện đơn giản, tùy chỉnh. Vì lý do đó, Blade cung cấp một phương pháp cho phép bạn nhanh chóng xác định các chỉ thị có điều kiện tùy chỉnh bằng cách sử dụng Closures.

Ví dụ, định nghĩa điều kiện tùy chỉnh để kiểm tra môi trường ứng dụng hiện tại. Chúng tôi có thể làm điều này ở phương thức ``boot`` ở ``AppServiceProvider``

```php
use Illuminate\Support\Facades\Blade;

/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

Khi điều kiện tùy chỉnh đã được xác định, chúng tôi có thể dễ dàng sử dụng nó trên mẫu của chúng tôi:

```php
@env('local')
    // The application is in the local environment...
@elseenv('testing')
    // The application is in the testing environment...
@else
    // The application is not in the local or testing environment...
@endenv
```