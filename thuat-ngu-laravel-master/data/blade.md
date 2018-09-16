## Blade

**Blade**: là một bộ máy được sử dụng để tối ưu hoá việc tạo mẫu [template cho các view trong Laravel](https://github.com/petehouston/laravel-docs-vn/blob/master/blade.md). Các Blade view trong Laravel có đuôi file là `.blade.php` để nhận biết. Nội dung của một Blade view khá là cơ bản, đơn giản chỉ là 1 file mã HTML có sử dụng các từ khoá riêng của Blade. Trong Blade, các từ khoá sử dụng có kí hiệu `@` ở đầu. Ví dụ:

```
@if (isset($error))
    <p>{{ $error->getErrors() }}</p>
@else
    <p>No error found.</p>
@endif
```

Ngoài việc Blade sử dụng để tối ưu hoá view, Blade còn là một cú pháp rất hữu dụng để làm các việc khác, ví dụ như tạo ra các task tự động hoá trong [Laravel Envoy](https://github.com/petehouston/laravel-docs-vn/blob/master/envoy.md).