# Cấu hình

- [Giới thiệu](#introduction)
- [Truy xuất các giá trị cấu hình](#accessing-configuration-values)
- [Cấu hình môi trường](#environment-configuration)
    - [Xác định môi trường hiện tại](#determining-the-current-environment)
- [Thực hiện cache cấu hình](#configuration-caching)
- [Chế độ bảo trì](#maintenance-mode)

<a name="introduction"></a>
## Giới thiệu

Tất cả các file cấu hình cho Laravel được lưu trong thư mục `config`. Mỗi thông số đều được ghi chú lại, vì thế hãy xem nội dung các file để làm quen với các thông số mà bạn có thể sử dụng.

<a name="accessing-configuration-values"></a>
## Truy xuất vào các giá trị cấu hình

Bạn có thể dễ dàng truy xuất vào các giá trị cấu hình sử dụng hàm helper `config` tại bất cứ đâu trong Laravel. Giá trị cấu hình có thể được lấy ra sử dụng cú pháp "dot", bao gồm tên file và tên thông số bạn muốn lấy ra. Giá trị mặc định có thể được trả về nếu thông số về cấu hình đó không tồn tại:

    $value = config('app.timezone');

Để thiết lập giá trị cấu hình lúc thực thi, truyền vào một mảng cho helper `config`:

    config(['app.timezone' => 'America/Chicago']);

<a name="environment-configuration"></a>
## Cấu hình môi trường

Thông thường, có nhiều giá trị cấu hình khác nhau tuỳ thuộc vào môi trường đang chạy sẽ khá là hữu ích. Ví dụ, bạn có thể sử dụng một cache driver khác với khi chạy trên server production. Việc sử dụng môi trường dựa trên cấu hình cũng khá là dễ dàng.

Laravel tận dụng thư viện [DotEnv](https://github.com/vlucas/phpdotenv) được phát triển Vance Lucas để thực hiện điều này. Với một ứng dụng cài mới, tại thư mục gốc sẽ có chứa một file `.env.example`. Nếu bạn cài Laravel thông qua Composer, file này sẽ được tự động đổi tên thành `.env`. Nếu không thì bạn cần tự chỉnh lại tên file.

Tất cả các biến liệt kê trong file sẽ được load lên vào trong biến super-global `$_ENV` của PHP khi ứng dụng nhận một request. Tuy nhiên, bạn có thể sử dụng hàm helper `env` để nhận các giá trị này từ các biến trong file cấu hình. Thực tế thì, nếu bạn xem nội dung file cấu hình của Laravel, bạn sẽ để ý thấy có vài thông số đã sử dụng hàm helper này rồi:

    'debug' => env('APP_DEBUG', false),

Giá trị thứ hai truyền vào trong `env` là "giá trị mặc định". Giá trị này sẽ được sử dụng nếu không có biến môi trường nào tồn tại cho một khoá.

File `.env` không nên được lưu trong source control của ứng dụng, vì mỗi lập trình viên hay server sử dụng có thể yêu cầu các thông số môi trường khác nhau.

Nếu bạn đang phát triển với một team, bạn có thể muốn tiếp tục lưu lại file `.env.example` trong ứng dụng. Bằng cách đặt các giá trị ví dụ vào trong file cấu hình, các lập trình viên khác trong team có thể thấy rõ ràng biến môi trường nào là cần thiết để chạy ứng dụng.

<a name="determining-the-current-environment"></a>
### Xác định môi trường hiện tại

Môi trường hiện tại của ứng dụng được xác định thông qua biến `APP_ENV` trong file `.env`. Bạn có thể lấy giá trị này thông qua hàm `environment` trong `App` [facade](/docs/{{version}}/facades):

    $environment = App::environment();

Bạn cũng có thể truyền tham số vào trong hàm `environment` để kiểm tra xem môi trường có khớp không. Nếu cần thiết, bạn có thể truyền vào nhiều giá trị cho hàm `environment`. Nếu môi trường đúng với giá trị truyền vào, hàm sẽ trả về `true`:

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment('local', 'staging')) {
        // The environment is either local OR staging...
    }

Đối tượng truy xuất ứng dụng cũng có thể được sử dụng thông qua helper `app`:

    $environment = app()->environment();

<a name="configuration-caching"></a>
## Thực hiện cache cấu hình

Để tăng tốc cho ứng dụng, bạn nên cache lại các file cấu hình sử dụng câu lệnh Artisan `config:cache`. Lệnh này sẽ gộp tất cả các thông số cấu hình trong ứng dụng vào trong một file để có thể được load lên nhanh chóng.

Bạn nên chạy lệnh `php artisan config:cache` trong quá trình triển khai môi trường production. Lệnh này không nên chạy ở môi trường phát triển vì các thông số cấu hình sẽ liên tục bị thay đổi trong khi phát triển.

<a name="maintenance-mode"></a>
## Chế độ bảo trì

Khi ứng dụng của bạn ở trong chế độ bảo trì, một view riêng được hiển thị cho tất cả các request tới ứng dụng. Điều này làm cho việc "disable" ứng dụng dễ dàng hơn khi bạn đang thực hiện cập nhật hay khi đang thực hiện bảo trì. Việc kiểm tra bảo trì nằm trong stack middleware mặc định. Nếu ứng dụng nằm trong chế độ bảo trì, thì một exception là `HttpException` sẽ được bắn ra với status code là 503.

Để kích hoạt chế độ bảo trì, đơn giản gõ lệnh `down`:

    php artisan down

Để tắt chế độ bảo trì, sử dụng lệnh `up`:

    php artisan up

#### Template trả lại ở chế độ bảo trì

Template sử dụng cho chế độ bảo trì nằm tại `resources/views/errors/503.blade.php`. Bạn hoàn toàn có quyền thay đổi view này theo ý riêng của bạn.

#### Chế độ bảo trì và queue

Khi ứng dụng của bạn ở trong trạng thái bảo trì, không có [queued jobs](/docs/{{version}}/queues) sẽ được thực hiện. Các jobs sẽ tiếp tục được xử lý bình thường khi nào mà ứng dụng không còn trong trạng thái bảo trì nữa.

#### Giải pháp khác cho chế độ bảo trì

Vì chế độ bảo trì yêu cầu ứng dụng của bạn bị down trong vài giây, bạn có thể sử dụng cách khác như [Envoyer](https://envoyer.io) để không có thời gian down khi triển khai với Laravel.
