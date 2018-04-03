# Installation

- [Cài đặt](#installation)
    - [Yêu cầu Server](#server-requirements)
    - [Cài đặt Laravel](#installing-laravel)
    - [Cấu hình](#configuration)

<a name="installation"></a>
## Cài đặt

<a name="server-requirements"></a>
### Yêu cầu Server

Laravel framework có một vài yêu cầu về hệ thống. Hiển nhiên là các yêu cầu này đã được đầy đủ trong [Laravel Homestead](/docs/{{version}}/homestead), vì thế, rất khuyến khích các bạn sử dụng Homestead cho môi trường phát triển.

Tuy nhiên, nếu bạn không sử dụng Homestead, bạn cần đảm bảo server đáp ứng các yêu cầu sau:

<div class="content-list" markdown="1">
- PHP >= 5.5.9
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

<a name="installing-laravel"></a>
### Cài đặt Laravel

Laravel tận dụng [Composer](http://getcomposer.org) để quản lý các dependencies. Vì thế, trước khi sử dụng Laravel, hãy chắc chắn là bạn đã cài Composer trên máy.

#### Sử dụng Laravel Installer

Đầu tiên, download tool cài đặt Laravel installer sử dụng Composer:

    composer global require "laravel/installer"

Hãy đảm bảo đặt thư mục `~/.composer/vendor/bin` vào trong biến môi trường PATH để `laravel` có thể được tìm thấy.

Sau khi cài đặt, sử dụng câu lệnh `laravel new` để cài đặt một project Laravel trong thư mục bạn muốn. Ví dụ, `laravel new blog` sẽ tạo một thư mục tên là `blog` và có chứa toàn bộ mã nguồn đầy đủ của Laravel cùng với tất cả các dependencies cần thiết. Phương pháp này thực hiện cài đặt nhanh hơn nhiều so với sử dụng Composer:

    laravel new blog

#### Sử dụng Composer Create-Project

Một cách khác, bạn có thể sử dụng Composer `create-project` trên terminal:

    composer create-project --prefer-dist laravel/laravel blog

<a name="configuration"></a>
### Cấu hình

Tất cả các file cấu hình của Laravel được đặt trong thư mục `config`. Mỗi tuỳ chọn được ghi tài liệu đầu đủ, vì thế, bạn cứ thoải mái xem các file và làm quen với các tuỳ chọn.

#### Quyền hạn thư mục

Sau khi cài Laravel, bạn có thể cần tiến hành cấu hình về quyền hạn. Các thư mục bên trong `storage` và `bootstrap/cache` cần được thiết lập quyền viết writable bởi web server nếu không là Laravel sẽ không chạy được. Nếu bạn đang sử dụng [Homestead](/docs/{{version}}/homestead), các quyền hạn này đã được thiết lập sẵn rồi.

#### Application Key

Điều tiếp theo bạn cần làm sau khi cài Laravel đó là thiết lập khoá ứng dụng (**application key**), là một chuỗi ngẫu nhiên. Nếu bạn cài Laravel sử dụng Laravel hay Laravel installer, khoá này đã được thiết lập luôn cho bạn thông qua thực thi câu lệnh `php artisan key:generate`. Về cơ bản, chuỗi này có độ dài là 32 kí tự. Khoá này có thể được thiết lập trong file `.env`. Nếu bạn vẫn chưa đổi tên file `.env.example` thành `.env` thì bạn nên làm ngay bây giờ. **Nếu application key không được thiết lập, các user sessions và các dữ liệu mã hoá khác sẽ không được bảo mật an toàn!**.

#### Cấu hình bổ sung

Laravel hầu như không cần cấu hình thêm nào khác. Bạn hoàn toàn thoải mái bắt đầu quá trình phát triển! Tuy nhiên, bạn có thể xem lại file cấu hình `config/app.php` và tài liệu của nó. Nó có chứa một số tuỳ chọn như `timezone` và `locale` mà bạn muốn thay đổi tuỳ thuộc vào yêu cầu ứng dụng của bạn.

Bạn cũng có thể muốn cầu hình thêm một vài thành phần bổ sung cho Laravel, chẳng hạn như:

- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)

Một khi Laravel được cài đặt, bạn cũng có thể tiến hành việc [cấu hình môi trường phát triển](/docs/{{version}}/configuration#environment-configuration).
