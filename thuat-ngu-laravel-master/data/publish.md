## Publish

**Publish**: là một thao tác của Laravel dùng để cung cấp tài nguyên được cung cấp từ các package bên ngoài vào trong ứng dụng để sử dụng. Ví dụ như file cấu hình của một package ngoài, muốn sử dụng được thì cần phải được đưa vào trong thư mục `config`, chính vì thế, lập trình viên Laravel cần phải thực hiện publish.

Để publish resources từ trong package vào trong ứng dụng, bạn sẽ cần phải sử dụng câu lệnh `artisan vendor:publish`.

Tham khảo mục [phát triển package cho Laravel](https://github.com/petehouston/laravel-docs-vn/blob/master/packages.md) để biết thêm chi tiết.