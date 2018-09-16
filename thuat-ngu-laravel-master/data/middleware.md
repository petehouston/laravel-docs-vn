## Middleware

**Middleware**: là các thao tác cần thực hiện bổ sung cho các HTTP request tới chương trình, có thể coi như là bộ lọc các dữ liệu trước khi đến và sau khi thực hiện xong của HTTP request. Ví dụ như, trước khi user truy cập vào trang nội bộ, thì sẽ có thao tác kiểm tra user đã đăng nhập hay chưa, hoặc kiểm tra xem nếu user đã đăng nhập thì có đủ quyền hạn để thực hiện xoá bài viết hay không ... Như vậy, việc sử dụng middleware đem lại lợi ích để kiểm tra dữ liệu vào ra của request, để đảm bảo được luồng dữ liệu trước khi vào xử lý trong controller hay trước khi trả về response cho trình duyệt là an toàn hay hợp lệ.

Tham khảo thêm về middleware [trong mục tài liệu](https://github.com/petehouston/laravel-docs-vn/blob/master/middleware.md).