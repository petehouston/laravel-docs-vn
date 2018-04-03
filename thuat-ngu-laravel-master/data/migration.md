## Migration

**Migration**: là một phương pháp xây dựng và quản lý version cho việc xây dựng cấu trúc cho database. Trong một migration, lập trình viên sẽ cần thực hiện khai báo việc tạo cấu trúc như thế nào thông qua các hàm được cung cấp trên `Schema` và `Blueprint` objects.

Việc sử dụng migration làm cho lập trình viên có thể dễ dàng tương tác với database hơn, thay vì phải nhớ và học các câu lệnh SQL để xây tạo các bảng, cột, index...lập trình viên có thể sử dụng OOP để tạo cấu trúc này một cách tiện lợi. Dưới đây là một ví dụ về migration,

```php

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->string('email')->unique();
        $table->string('password');
        $table->rememberToken();
        $table->timestamps();
    });

```

Ở ví dụ trên, hàm `Schema::create` sẽ tạo một bảng là `users` với các trường như `PRIMARY KEY id INT AUTO INCREMENT`, `name VARCHAR(255)`, `email VARCHAR(255) UNIQUE` ...

Tham khảo thêm về [Laravel migration tại đây](https://github.com/petehouston/laravel-docs-vn/blob/master/migrations.md).