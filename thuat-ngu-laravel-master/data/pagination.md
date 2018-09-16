## Pagination

**Pagination**: hay còn gọi là "phân trang" là một kĩ thuật để phân tách một mảng kết quả thành các trang khác nhau. Ngoài ra, việc phân trang còn bao gồm sinh ra mã HTML và trỏ liên kết tới các trang tiếp theo.

**Paginator**: ám chỉ đối tượng trong Laravel được dùng để xử lý việc phân trang. Thông thường, các Paginator kế thừa từ class `Illuminate\Pagination\Paginator` hoặc `Illuminate\Pagination\LengthAwarePaginator`.

Mã HTML sinh ra từ Paginator trong Laravel tương thích với [Bootstrap CSS Framework](http://getbootstrap.com).