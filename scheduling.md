# Task Scheduling

- [Giới thiệu](#introduction)
- [Định nghĩa Schedules](#defining-schedules)
    - [Tuỳ chọn tần suất Schedule](#schedule-frequency-options)
    - [Ngăn chặn task chồng chéo](#preventing-task-overlaps)
- [Task Output](#task-output)
- [Task Hooks](#task-hooks)

<a name="introduction"></a>
## Giới thiệu

Trước đây, lập trình viên phải tạo ra các dòng cron cho mỗi task cần được schedule. Tuy nhiên, việc này khá đau đầu. Việc đặt lịch cho task không nằm trong source control, và bạn phải SSH vào trong server và thêm vào các nội dung cron. Bộ lệnh đặt lịch của Laravel cho phép bạn định nghĩa các lệnh đặt lịch một cách liền mạch và rõ ràng, và bạn chỉ cần thực hiện thêm đúng một dòng Cron cần thiết vào trong server.

Việc đặt lịch task được định nghĩa bên trong hàm `schedule` của file `app/Console/Kernel.php`. Để giúp bạn bắt đầu, một ví đụ đơn giản được ghi sẵn bên trong hàm đó. Bạn tuỳ ý thêm bao nhiêu task tuỳ thích vào trong đối tượng `Schedule`.

### Khởi động Scheduler

Đây là dòng Cron duy nhất mà bạn cần thêm vào trong server:

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

Dòng Cron này sẽ gọi tới bộ lệnh thực hiện lịch của Laravel mỗi phút. Sau đó, Laravel sẽ kiểm tra các task đã được lên lịch và thực thi các task cần thực hiện.

<a name="defining-schedules"></a>
## Định nghĩa Schedules

Bạn có thể định nghĩa các task bên trong hàm `schedule` của class `App\Console\Kernel`. Để bắt đầu, hãy cùng nhìn vào ví dụ sau về lên lịch cho một task. Trong ví dụ này, chúng ta sẽ lên lịch cho một `Closure` được gọi mỗi ngày vào lúc nửa đêm. Bên trong `Closure` chúng ta sẽ thực thi một database query để xoá một bảng:

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * The Artisan commands provided by your application.
         *
         * @var array
         */
        protected $commands = [
            \App\Console\Commands\Inspire::class,
        ];

        /**
         * Define the application's command schedule.
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

Ngoài việc đặt lịch cho `Closure`, bạn cũng có thể đặt lịch cho [câu lệnh Artisan](/docs/{{version}}/artisan) và các lệnh hệ thống. Ví dụ, bạn có thể sử dụng hàm `command` để đặt lịch cho một câu lệnh Artisan như sau:

    $schedule->command('emails:send --force')->daily();

Lệnh `exec` có thể được sử dụng để thực thi một câu lệnh trong hệ điều hành:

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### Tuỳ chọn tần suất Schedule

Dĩ nhiên, có vài kiểu đặt lịch bạn có thể thiết lập cho task:

Hàm  | Mô tả
------------- | -------------
`->cron('* * * * * *');`  |  Chạy task với lịch Cron tuỳ chọn
`->everyMinute();`  |  Chạy từng phút
`->everyFiveMinutes();`  |  Cứ 5 phút chạy một lần
`->everyTenMinutes();`  |  Cứ 10 phút chạy một lần
`->everyThirtyMinutes();`  |  Cứ 30 phút chạy một lần
`->hourly();`  |  Cứ mỗi tiếng chạy một lần
`->daily();`  |  Chạy hàng ngày lúc nửa đêm
`->dailyAt('13:00');`  |  Chạy hàng ngày lúc 13:00
`->twiceDaily(1, 13);`  |  Chạy ngày hai lần tại hai thời điểm 1:00 và 13:00
`->weekly();`  |  Chạy hàng tuần
`->monthly();`  |  Chạy hàng tháng
`->monthlyOn(4, '15:00');`  |  Chạy vào ngày mùng 4 hàng tháng lúc 15:00
`->quarterly();` |  Cứ mỗi quý chạy một lần
`->yearly();`  |  Mỗi năm chạy một lần
`->timezone('America/New_York');` | Thiết lập timezone

Các hàm này có thể phối hợp nhau để tạo ràng buộc chạy các kiểu lịch phức tạp hơn. Ví dụ như chạy một câu lệnh hàng tuần vào thứ hai lúc 13:00:

    $schedule->call(function () {
        // Runs once a week on Monday at 13:00...
    })->weekly()->mondays()->at('13:00');

Dưới đây là danh sách các ràng buộc đặt lịch bổ sung:

Hàm  | Mô tả
------------- | -------------
`->weekdays();`  |  Chỉ chạy vào ngày thường
`->sundays();`  |  Chỉ chạy vào Chủ Nhật
`->mondays();`  |  Chỉ chạy vào thứ Hai
`->tuesdays();`  |  Chỉ chạy vào thứ Ba
`->wednesdays();`  |  Chỉ chạy vào thứ Tư
`->thursdays();`  |  Chỉ chạy vào thứ Năm
`->fridays();`  |  Chỉ chạy vào thứ Sáu
`->saturdays();`  |  Chỉ chạy vào thứ Bảy
`->when(Closure);`  |  Chạy phụ thuộc vào điều kiện trong `Closure`

#### Kiểm tra ràng buộc

Hàm `when` có thể được sử dụng để giới hạn thực thi của một task dựa trên kết quả kiểm tra mệnh đề trong `Closure`. Nói cách khác, nếu như `Closure` trả về `true`, task sẽ thực thi nếu như không có điều kiện ràng buộc nào ngăn chặn:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

Hàm `skip` có thể coi là ngược lại đối với `when`. Nếu `skip` trả về `true`, task sẽ không được thực thi:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

Khi sử dụng phối hợp với `when`, câu lệnh đặt lịch sẽ chỉ thực thi khi mà các điều kiện `whene` trả về `true`.

<a name="preventing-task-overlaps"></a>
### Ngăn chặn task chồng chéo

Mặc định, task được đặt lịch sẽ thực hiện thậm chí khi mà lần thực thi trước vẫn còn đang thực hiện. Để chặn việc này, bạn có thể sử dụng hàm `withoutOverlapping`:

    $schedule->command('emails:send')->withoutOverlapping();

Trong ví dụ này, [câu lệnh](/docs/{{version}}/artisan) `emails:send` sẽ được thực hiện mỗi phút nếu như chưa được chạy. Hàm `withoutOverlapping` đặc biệt hữu dụng nếu bạn có task thực hiện không đồng đều về thời gian, hạn chế việc bạn phải tính toán xem một task tốn mất bao nhiêu thời gian để thực hiện tiếp lần sau.

<a name="task-output"></a>
## Task Output

Bộ đặt lịch của Laravel cung cấp vài hàm tiện tích để làm việc với output sinh ra bởi các task. Đầu tiên là hàm `sendOutputTo`, bạn có thể đẩy output tới một file để tìm hiểu sau:

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

Nếu bạn muốn chèn thêm output vào một file, bạn có thể sử dụng `appendOutputTo`:

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

Sử dụng hàm `emailOutputTo`, bạn có thể gửi email về output tới địa chỉ bạn muốn. Chú ý là output phải được lưu vào một file sử dụng `sendOutputTo` trước. Thêm nữa, trước khi gửi mail, bạn phải cấu hình [dịch vụ email](/docs/{{version}}/mail) của Laravel:

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> **Chú ý:** Hai hàm `emailOutputTo` và `sendOutputTo` không sử dụng được với hàm `command` và không hỗ trợ cho hàm `call`.

<a name="task-hooks"></a>
## Task Hooks

Sử dụng hàm `before` và `after`, bạn có thể chỉ định đoạn code cần thực hiện trước vào sau khi task hoàn thiện:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

#### Pinging URLs

Sử dụng hàm `pingBefore` và `thenPing`, bộ đặt lịch có thể tự động gửi ping tới một URL trước và sau khi một task hoàn thiện. Hàm này khá hữu ích cho việc gửi thông báo tới một dịch vụ bên ngoài, ví dụ như là Laravel Envoyer](https://envoyer.io), để cho biết là task của bạn đã bắt đầu hay hoàn thiện.

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

Sử dụng `pingBefore($url)` hoặc `thenPing($url)` yêu cầu thư viện Guzzle HTTP. Vì thế, bạn cần thêm Guzzle vào trong project bằng cách thêm dòng sau vào trong `composer.json`:

    "guzzlehttp/guzzle": "~5.3|~6.0"
