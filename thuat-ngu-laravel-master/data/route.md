# Route

**Route**: mỗi route tương ứng với một URL trên trình duyệt. Mỗi route sẽ đảm nhiệm việc quản lý URL nào và thực hiện điều phối xử lý như thế nào, có thể là qua một Closure, hoặc qua một Controller.

Trong Laravel các route được khai báo trong file: `app/Http/routes.php`.

Dưới đây là một ví dụ về route:

```PHP
Route::get('/', function () {
    return "Welcome";
});

Route::get('/about', 'PagesController@about');
```