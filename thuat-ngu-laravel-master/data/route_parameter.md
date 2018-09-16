## Route Parameter

**Route pamareter**: là các tham số truyền trên URL, sẽ được chuyển đổi thành các tham số để thực hiện sử dụng trong controller. Các route parameter này được khai báo ở tham số thứ nhất trong khai báo route.

Ví dụ:

```
Route::get('/articles/{id}', 'ArticlesController@show');
```

Ở khai báo route trên thì `{id}` là một route parameter, và khi sử dụng trong controller thì sẽ sử dụng như một đối số vào trong hàm một cách bình thường.

```
public function show($id) {

}
```