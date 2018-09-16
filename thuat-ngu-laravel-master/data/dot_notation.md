## Dot Notation

**Dot Notation**: là một phương pháp hay được sử dụng trong Laravel để truy cập vào các thành phần lồng ghép vào nhau, ví dụ, lấy một giá trị trong mảng, lấy một views trong một thư mục... Như tên gọi, dấu `.` được sử dụng để phân cách các level truy cập.

Ví dụ sau đây minh hoạ cho việc lấy một view tại `resources/views/pages/common/contact.blade.php`,

```
return view('pages.common.contact');
```