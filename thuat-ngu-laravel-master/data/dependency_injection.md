## "Dependency Injection"

**Dependency Injection (viết tắt, DI)**: là một phương pháp xử lý sự phụ thuộc vào các đối tượng khác bằng cách thêm đối tượng vào trong hàm khởi tạo hoặc trong các hàm khác.

Dưới đây là một ví dụ thương thấy về **DI** thực hiện trong controllers của Laravel:

```
class PagesController extends from Controller {

    public function home(HomeRepositoryInterface $homeRepo) {
        // ...
    }
}
```

