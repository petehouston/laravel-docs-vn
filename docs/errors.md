# Errors & Logging

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [The Exception Handler](#the-exception-handler)
    - [Phương thức Report](#report-method)
    - [Phương thức Render](#render-method)
- [HTTP Exceptions](#http-exceptions)
    - [Tuỳ biến HTTP Error Pages](#custom-http-error-pages)
- [Logging](#logging)

<a name="introduction"></a>
## Giới thiệu

Khi bạn bắt đầu một project Laravel mới, việc xử lý error và exception đã được cấu hình sẵn cho bạn. Thêm vào đó, Laravel được tích hợp với thư viện [Monolog](https://github.com/Seldaek/monolog), đây là thư viện hỗ trợ các xử lý log rất hữu hiệu.

<a name="configuration"></a>
## Cấu hình

#### Chi tiết Error

Các nội dung chi tiết lỗi trong ứng dụng của bạn hiển thị trên trình duyệt được điều khiển bởi cấu hình `debug` trong file cấu hình `config/app.php`. Mặc định, cấu hình này thiết lập dựa trên giá trị biến môi trường `APP_DEBUG`, lưu trong file `.env`.

Trong môi trường phát triển nội bộ, bạn nên set giá trị `APP_DEBUG` thành `true`. Trong môi trường production, giá trị này luôn luôn phải là `false`.

#### Các chế độ log

Cơ bản, Laravel hỗ trợ các chế độ log này `single`, `daily`, `syslog`, và `errorlog`. Ví dụ, nếu bạn muốn ghi log file hàng ngày, thay vì ghi vào một file, bạn đơn giản chỉ cần thiết lập giá trị `log` trong file `config/app.php`:

    'log' => 'daily'

Khi sử dụng chế độ `daily`, Laravel sẽ chỉ lưu trữ log files của 5 ngày. Nếu bạn muốn điều chỉnh số lượng file lưu trữ, bạn có thể thêm vào cấu hình `log_max_files` vào trong `app.php`:

    'log_max_files' => 30

#### Tuỳ chọn cấu hình Monolog

Nếu bạn muốn điều khiển toàn bộ quy trình Monolog cấu hình trên ứng dụng, bạn có thể sử dụng phương thức `configureMonologUsing`. Bạn nên gọi xử lý này trong file `bootstrap/app.php` ngay trước khi biến `$app` được trả về:

    $app->configureMonologUsing(function($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## The Exception Handler

Tất cả các exception được xử lý bởi `App\Exception\Handler` class. Class này chứa hai phương thức: `report` và `render`. Chúng ta sẽ tìm hiểu các phương thức này.

<a name="report-method"></a>
### Phương thức Report

Phương thức `report` được sử dụng để log các exception hoặc gửi chúng tới các dịch vụ bên ngoài như [BugSnag](https://bugsnag.com) hay [Sentry](https://github.com/getsentry/sentry-laravel). Mặc định, `report` đơn giản chỉ đẩy exception về class cơ sở nơi mà exception được log lại. Tuy nhiên, bạn có thể tuỳ ý log exception theo cách bạn muốn.

Ví dụ, nếu bạn cần report nhiều kiểu exception bằng nhiều cách khác nhau, bạn có thể sử dụng toán tử kiểm tra `instanceof` của PHP:

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $e
     * @return void
     */
    public function report(Exception $e)
    {
        if ($e instanceof CustomException) {
            //
        }

        return parent::report($e);
    }

#### Loại bỏ exception theo kiểu

Thuộc tính `$dontReport` của handler xử lý exception chứa một mảng các kiểu exception sẽ không cần log. Mặc định, exception của lỗi 404 sẽ không được lưu vào trong log file. Bạn có thể thêm các kiểu exception vào trong mảng này nếu thấy cần thiết.

<a name="render-method"></a>
### Phương thức Render

Phương thức `render` chịu trách nhiệm chuyển đổi một exception thành một mẫu HTTP response để trả lại cho trình duyệt. Mặc định, exception được đẩy tới class cơ sở để tạo response cho bạn. Tuy nhiên, bạn hoàn toàn tự do trong việc kiểm tra kiểu exception hoặc trả về response tuỳ ý riêng của bạn:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $e
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $e)
    {
        if ($e instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $e);
    }

<a name="http-exceptions"></a>
## HTTP Exceptions

Một số exception mô tả mã lỗi HTTP từ server. Ví dụ, đó có thể là một lỗi "page not found" (404), một lỗi "unauthorized error" (401) hoặc lỗi 500. Để sinh ra response cho mã lỗi tại bất kì vị trí trên ứng dụng, sử dụng:

    abort(404);

Phương thức `abort` sẽ lập tức đẩy ra một exception sẽ được render bởi exception handler. Bạn có thể tuỳ chọn cung cấp thêm nội dung response:

    abort(403, 'Unauthorized action.');

Phương thức này có thể được sử dụng bất cứ lúc nào trong lifecycle của request.

<a name="custom-http-error-pages"></a>
### Tuỳ biến HTTP Error Pages

Laravel làm cho việc trả về trang lỗi tuỳ chọn cho các mã lỗi HTTP một cách dễ dàng. Ví dụ, nếu bạn muốn chỉnh sửa trang lỗi riêng cho trang HTTP 404, tạo một file `resources/views/errors/404.blade.php`. File này sẽ được gọi ra khi có lỗi 404 được sinh ra trong ứng dụng của bạn.

Các views nằm trong thư mục này nên được đặt tên trùng với mã lỗi HTTP tương ứng.

<a name="logging"></a>
## Logging

Công cụ log của Laravel cung cấp một layer đơn giản dựa trên thư viện [Monolog](http://github.com/seldaek/monolog). Mặc định, Laravel được cấu hình để tạo log file cho ứng dụng trong thư mục `storage/logs`. Bạn có thể viết thêm nội dung vào trong logs sử dụng `Log` [facade](/docs/{{version}}/facades):

    <?php

    namespace App\Http\Controllers;

    use Log;
    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Logger cung cấp 8 level log cơ bản theo định nghĩa của [RFC 5424](http://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** and **debug**.

    Log::emergency($error);
    Log::alert($error);
    Log::critical($error);
    Log::error($error);
    Log::warning($error);
    Log::notice($error);
    Log::info($error);
    Log::debug($error);

#### Thông tin theo ngữ cảnh

Một mảng các dữ liệu theo ngữ cảnh cũng có thể được truyền vào phương thức log. Các dữ liệu này sẽ được format và hiển thị cùng với nội dung log:

    Log::info('User failed to login.', ['id' => $user->id]);

#### Truy cập vào đối tượng phía dưới của Monolog

Monolog có một số handler bổ sung mà bạn có thể sử dụng cho việc log. Nếu cần thiết, bạn có thể truy cập vào instance phía dưới của Monolog mà đang được dùng bởi Laravel:

    $monolog = Log::getMonolog();
