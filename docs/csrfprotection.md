# Bảo mật CSRF

## Giới thiệu

Laravel làm cho nó dễ dàng để bảo vệ các ứng dụng của bạn từ tấn công giả mạo cross-site request forgery (CSRF). Cross-site request forgery (CSRF) là một loại mã độc, theo đó các lệnh trái phép được thực thi thay cho một người dùng đã xác thực.

Laravel tự động tạo ra một CSRF "token" cho mỗi người dùng hoạt động quản lý bởi ứng dụng. Mã này dử dụng để xác minh rằng người dùng là một trong những người thực sự gửi yêu cầu với ứng dụng.

Bất cứ khi nào bạn tạo một biểu mẫu HTML trong ứng dụng của bạn, bạn nên thêm một trường ẩn CSRF token để bảo mật CSRF có thể xác nhận yêu cầu. Để tạo ra `_token` bạn có thể sử dụng hàm `csrf_fiel` helper function:

```PHP
// Vanilla PHP
<?php echo csrf_field(); ?>

// Blade Template Syntax
{{ csrf_field() }}
```

`csrf_field` được tạo thông qua HTML:

```PHP
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```

Bạn không cần phải tự xác minh CSRF token trên các yêu cầu POST, PUT, DELETE. `VerifyCsrfToken` middleware, được bao gồm trong `web` middleware sẽ tự động xác minh token có phù hợp hay không.

## Loại bỏ URIs khỏi bảo mật CSRF

Đôi khi, bạn có thể muốn loại bỏ URIs khỏi bảo mật CSRF. Ví dụ, nếu bạn đang sử dụng Stripe để xử lý thanh toán và được sử dụng hệ thông  webhook của họ, bạn sẽ cần phải loại trừ đi các router xử lý của bạn khỏi bảo mật CSRF của Laravel.

Bạn có thể loại bỏ các bảo mật CSRF bằng cách xác router đường bên ngoài middleware web bên trong tệp tin `routes.php` hoặc bằng cách thêm thuộc tính `$except` trong middleware `VerifyCsrfToken`:

```PHP
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

class VerifyCsrfToken extends BaseVerifier
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
    ];
}
```

## X-CSRF-TOKEN

Ngoài việc kiểm tra CSRF như một tham số POST, middleware `VerifyCsrfToken` cũng sẽ kiểm tra các yêu cầu X-CSRF-TOKEN. Bạn có thể, ví dụ, lưu trữ token trong thẻ meta:

```PHP
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Một khi bạn đã tạo ra các thẻ `meta`, bạn có thể chỉ định một thư viện như jQuery để thêm các thẻ cho tất cả các yêu cầu (request headers). Điều này cung cấp thật đơn giản, thuận tiện đẻ bảo vệ các ứng dụng AJAX của bạn:

```PHP
$.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
});
```

## X-XSRF-TOKEN

Laravel cũng lưu X-XSRF-TOKEN trong một cookie `XSRF-TOKEN`. Bạn có thể sử dụng cookie để thiết lập các yêu cầu `X-XSRF-TOKEN`. Một số thư viện Javascript, như Agular, làm điều này tự động cho bạn. Nó không chác chắn rằng bạn phải tự tay làm điều này.