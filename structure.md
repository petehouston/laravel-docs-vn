# Application Structure

- [Giới thiệu](#introduction)
- [Thư mục Root](#the-root-directory)
- [Thư mục App](#the-app-directory)

<a name="introduction"></a>
## Giới thiệu

Cấu trúc mặc định của ứng dụng Laravel được thiết kế để có thể cung cấp điểm khởi đầu tốt cho cả các ứng dụng lớn và nhỏ. Tất nhiên là bạn có thể tự do sắp xếp ứng dụng của bạn theo cách mà bạn muốn. Laravel hầu như không có hạn chế việc class nào đó phải nằm ở đâu - miến là Composer có thể autoload class đó.

<a name="the-root-directory"></a>
## Thư mục Root

Thư mục gốc của một ứng dụng Laravel cài đặt mới bao gồm các loại thư mục:

Thư mục `app`, như bạn mong đợi, chứa các đoạn mã cốt lõi cho ứng dụng của bạn. Chúng ta sẽ sớm tìm hiểu thư mục này.

Thư mục `bootstrap` chứa vài file dùng để tự khởi động framework và cấu hình tự động load, cũng như là thư mục `cache` chứa vài file framework tạo ra để tối ưu hoá hiệu suất tự khởi động.

Thư mục `config`, như ngụ ý của tên, chứa toàn bộ file cấu hình của ứng dụng.

Thư mục `database` chứa các file migration và seeds của cơ sở dữ liệu. Nếu bạn muốn, bạn có thể sử dụng thư mục này để chứa cơ sở dữ liệu SQLite.

Thư mục `public` chứa các controller mặt trước và tài nguyên (ảnh, JavaScript, CSS...)

Thư mục `resources` chứa các view, tài nguyên thô (LESS, SASS, CoffeeScript), và các file ngôn ngữ đã được địa phương hoá.

Thư mục `storage` chứa các file Blade mẫu đã được biên dịch, các file session, và các file khác được tạo ra bởi framework. Thư mục này được chia ra các thư mục `app`, `framework`, và `log`. Thư mục `app` chứa file được dùng bởi ứng dụng của bạn. Thư mục `framework` dùng để lưu các file tạo ra bởi framework và file cache. Cuối cùng, thư mục `logs` chứa các file log của ứng dụng.

Thư mục `test` chứa các file kiểm thử. Xem [PHPUnit](https://phpunit.de/) như một ví dụ để mở rộng tìm hiểu. 

Thư mục `vendor` chứa các file phụ thuộc cho [Composer](https://getcomposer.org)

<a name="the-app-directory"></a>
## Thư mục 

Phần cốt lõi của ứng dụng của bạn nằm trong thư mục `app`. Theo mặc định thì thư mục này được không gian tên hoá (namespaced) bên trong `App` và được load một cách tự động bởi Composer, sử dụng [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/).

Thư mục `app` được kèm theo mội vài thư mục khắc như `Console`, `Http`, và `Providers` bên trong. Hãy nghĩ về `Console` và `Http` như là các thư mục cung cấp API cho phần "cốt lõi" của ứng dụng. Giao thức HTTP và CLI là 2 cơ chế để tương tác với ứng dụng của bạn, nhưng không thực sự chứa logic của ứng dụng. Hay nói một cách khác, đơn giản là có 2 cách để thực thi lệnh đến ứng dụng của bạn. Thư mục `Console` chứa các câu lệnh Artisan, trong khi `Http` là thư mục chứa các controllers, bộ lọc request (middleware) và các yêu cầu (request) gửi lên ứng dụng.

Thư mục `Events`, như bạn muốn, chứa các [lớp sự kiện](/docs/{{version}}/events). Sự kiện có thể dùng để thông báo các phần khác của ứng dụng rằng một hành động nào đó đã xảy ra, cung cấp khả năng xử lý một cách linh hoạt và riêng biệt.

Thư mục `Exception` chứa các bộ xử lý ngoại lệ và cũng là nơi tốt để gán các ngoại lệ được bắn ra bởi ứng dụng của bạn.

Thư mục `Jobs`, tất nhiên, là nơi lưu trũ các [công việc có thể cho vào hàng](/docs/{{version}}/queues) cho ứng dụng. Các công việc này có thể xếp hàng, đợi ứng dụng xử lý, hoặc chạy đồng bộ trong vòng đời của yêu cầu (request) hiện tại.

Thư mục `Listeners` chứa các lớp xử lý cho các sự kiện của bạn. Các bộ xử lý sẽ nhận sự kiện và thực thi logic khi hồi đáp cho sự kiện đã được bắn ra. Ví dụ, một sự kiện `UserRegistered` có thể được xử lý bởi một bộ lắng nghe `SendWelcomeEmail` chẳng hạn.

Thư mục `Policies` chứa các lớp quy ước cấp quyền cho ứng dụng của bạn. Các quy ước được dùng để quyết định user có thể thực hiện hành động đối với mội tài nguyên nào đó. Xem thêm tại [tài liệu cấp quyền](/docs/{{version}}/authorization)

> **Ghi chú:** Rấr nhiều lớp trong thư mục `app` được tạo tự động bởi các lệnh. Để xem những lệnh có sẵn, thực thi lệnh `php artisan list make` trong terminal của bạn.
