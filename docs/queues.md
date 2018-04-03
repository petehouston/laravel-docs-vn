# Queues

- [Giới thiệu](#introduction)
- [Viết Job classes](#writing-job-classes)
    - [Tạo Job Classes](#generating-job-classes)
    - [Cấu trúc Job Class](#job-class-structure)
- [Đưa Job vào trong Queue](#pushing-jobs-onto-the-queue)
    - [Trì hoãn Jobs](#delayed-jobs)
    - [Job Events](#job-events)
- [Thực thi Queue Listener](#running-the-queue-listener)
    - [Cấu hình Supervisor](#supervisor-configuration)
    - [Daemon Queue Listener](#daemon-queue-listener)
    - [Triển khai với Daemon Queue Listeners](#deploying-with-daemon-queue-listeners)
- [Xử lý các Failed Job](#dealing-with-failed-jobs)
    - [Failed Job Events](#failed-job-events)
    - [Thử lại Failed Jobs](#retrying-failed-jobs)

<a name="introduction"></a>
## Giới thiệu


<a name="configuration"></a>
### Cấu hình

File cấu hình được lưu trong `config/queue.php`. Trong file này bạn sẽ muốn tìm cấu hình kết nối cho mỗi queue drivers được đi kèm với framework, bao gồm database, [Beanstalkd](http://kr.github.com/beanstalkd), [Amazon SQS](http://aws.amazon.com/sqs), [Redis](http://redis.io), và synchronous driver (để sử dụng local).

Driver `null` cũng có để đơn giản là thực hiện bỏ queued jobs.

### Yêu cầu driver

#### Database

Để sử dụng driver `database`, bạn sẽ cần một bảng để lưu trũ thông tin về jobs. Để tạo file migration cho việc tạo bảng, thực thi câu lệnh Artisan `queue:table`. Một khi migration đã được tạo, bạn có thể migrate vào database sử dụng lệnh `migrate`:

    php artisan queue:table

    php artisan migrate

#### Các dependency khác cho queue

Các dependency dưới đây cần thiết cho từng queue driver tương ứng:

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`

<a name="writing-job-classes"></a>
## Viết Job Classes

<a name="generating-job-classes"></a>
### Tạo Job Classes

Về mặc định, tất cả các job có thể được queue được lưu trong thư mục `app/Jobs`. Bạn có thể tạo một job mới sử dụng Artisan:

    php artisan make:job SendReminderEmail

Câu lệnh này sẽ tạo ra một class mới trong thư mục `app/Jobs`, và class sẽ triển khai interface `Illuminate\Contracts\Queue\ShouldQueue`, cho Laravel biết job này cần được đẩy vào queue thay vì chạy đồng bộ.

<a name="job-class-structure"></a>
### Cấu trúc Job Class

Job class khá đơn giản, bình thường chỉ chứa một hàm duy nhất `handle` mà được gọi khi job được queue gọi để xử lý. Để bắt đầu, hãy cùng coi ví dụ dưới đây:

    <?php

    namespace App\Jobs;

    use App\User;
    use App\Jobs\Job;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendReminderEmail extends Job implements ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;

        protected $user;

        /**
         * Create a new job instance.
         *
         * @param  User  $user
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Execute the job.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function handle(Mailer $mailer)
        {
            $mailer->send('emails.reminder', ['user' => $this->user], function ($m) {
                //
            });

            $this->user->reminders()->create(...);
        }
    }

Ở ví dụ này, chú ý là chúng có thể truyền vào một [Eloquent model](/docs/{{version}}/eloquent) trực tiếp vào trong hàm khởi tạo của Job. Vì có trait `SerializesModels` trong Job class, nên các Eloquent model sẽ được serialize và unserialize khi job được xử lý. Nếu queue job nhận một model vào hàm khởi tạo, thì chỉ có id của model sẽ được serialize vào trong queue. Khi job thực sự được xử lý, hệ thống queue sẽ tự động lấy ra model đầy đủ từ database. Nó thực hiện tất cả khá rõ ràng trong chương trình và ngăn chặn các vấn đề phát sinh từ việc serialize nguyên một Eloquent model.

Hàm `handle` được gọi khi job được queue xử lý. Chú ý chúng ta có thể type-hint dependency trong hàm `handle` của job. Laravel [service container](/docs/{{version}}/container) sẽ tự động nhúng các dependency tương ứng vào.

#### Khi việc không như ý muốn

Nếu có exception khi đang thực hiện queue, nó sẽ tự động bị đẩy ngược lại vào queue để có thể thực hiện lại lần nữa. Job sẽ thực hiện như thế cho tới khi số lượng lần thử lại đạt tới số lần tối đa cho phép. Số lần thực thi tối đa này được truyền qua thông số `--tries` của câu lệnh `queue:listen` hay `queue:work`. Để biết thêm chi tiết về cách thực thi queue listener, [tham khảo mục dưới đây](#running-the-queue-listener).

#### Tự release job

Nếu bạn muốn `release` (gỡ, nhả) job thủ công, trait `InteractsWithQueue` có sẵn trong job class, cho phép gọi hàm `release` trong queue job. Hàm `release` nhận một tham số: số giây bạn muốn chờ cho tới khi job có hiệu lực trở lại lần nữa:

    public function handle(Mailer $mailer)
    {
        if (condition) {
            $this->release(10);
        }
    }

#### Kiểm tra số lần thực thi

Như ghi ở trên, nếu có exception xuất hiện trong khi job đang được thực thi, nó sẽ tự được bị release trở lại queue. Bạn có thể kiểm qua số lần thử được thực hiện bằng hàm `attempts`:

    public function handle(Mailer $mailer)
    {
        if ($this->attempts() > 3) {
            //
        }
    }

<a name="pushing-jobs-onto-the-queue"></a>
## Đưa job vào trong queue

Controller mặc định của Laravel nằm tại `app/Http/Controllers/Controller.php` sử dụng trait `DispatchesJobs`. Trait này cung cấp vài phương thức cho phép bạn đẩy job vào trong queue, ví dụ như hàm `dispatch`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $this->dispatch(new SendReminderEmail($user));
        }
    }

#### Trait `DispatchesJobs`

Dĩ nhiên, thi thoảng bạn muốn dispatch một job từ đâu đó trong ứng dụng ngoài route và controller. Vì lí do đó, bạn có thể thêm vào trait `DispatchJobs` trong bất cứ class nào để có thể sử dụng các phương thức dispatch. Ví dụ, đây là một class sử dụng trait:

    <?php

    namespace App;

    use Illuminate\Foundation\Bus\DispatchesJobs;

    class ExampleClass
    {
        use DispatchesJobs;
    }

#### Hàm `dispatch`

Hoặc bạn có thể muốn sử dùng hàm helper `dispatch`:

    Route::get('/job', function () {
        dispatch(new App\Jobs\PerformTask);

        return 'Done!';
    });

#### Chỉ định queue cho một job

Bạn cũng có thể chỉ định queue nào mà job sẽ được gửi tới.

Bằng cách đẩy job tới các quêu khác nhau, bạn có thể "categorize" các queued job, và ưu tien hoá bao nhiêu worker thiết lập cho các queue. Việc này không đẩy job tới các queue "connections" khác nhau như khai báo về queue trong file cấu hình, nhưng chỉ cho queue cụ thể trong một connection. Để chỉ rõ queue nào, sử dụng hàm `onQueue`. Hàm `onQueue` được cung cấp bởi `Illuminate\Bus\Queueable`, mà nằm sẵn trong class `App\Jobs\Job`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $job = (new SendReminderEmail($user))->onQueue('emails');

            $this->dispatch($job);
        }
    }

<a name="delayed-jobs"></a>
### Trì hoãn Jobs

Đôi lúc bạn sẽ muốn trì hoãn việc thực thi một queued job. Ví dụ, bạn muốn queue một job mà gửi customer một email nhắc nhở sau khi đăng kí tầm 5 phút. Bạn có thể làm việc này sử dụng hàm `delay` của job class, được cung cấp bởi trait `Illuminate\Bus\Queueable`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $job = (new SendReminderEmail($user))->delay(60 * 5);

            $this->dispatch($job);
        }
    }

Ở ví dụ này, chúng ta chỉ định job sẽ bị trì hoãn tầm 5 phút trước khi được đưa ra worker.

> **Lưu ý:** Amazon SQS service có thời gian trì hoãn tối đa là 15 phút.

<a name="job-events"></a>
### Job Events

#### Job Lifecycle Events

Hai hàm `Queue::before` và `Queue::after` cho phép bạn đăng kí một callback được thực hiện trước khi một queued job được khởi động hay khi nó thực thi thành công. Callback là các hữu hiệu để thực hiện các việc log bổ sung, queue một job con, hay cập nhật số liệu thống kê cho dashboard. Ví dụ, chúng ta có thể gán một callback cho event này từ `AppServiceProvider` đi kèm với Laravel:

    <?php

    namespace App\Providers;

    use Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->data
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="running-the-queue-listener"></a>
## Thực thi Queue Listener

Laravel có chứa một câu lệnh Artisan mà sẽ chạy các job mới khi chúng được đưa vào queue. Bạn có thể chạy listener sử dụng lệnh `queue:listen`:

    php artisan queue:listen

Bạn cũng có thể chỉ định queue connection nào mà listener nên xử lý:

    php artisan queue:listen connection-name

Chú ý là một khi task được bắt đầu thì nó sẽ thực hiện tiếp tục cho tới khi được dừng thủ công. Bạn có thể sử dụng process monitor như [Supervisor](http://supervisord.org/) để đảm bảo queue listener không bị dừng.

#### Độ ưu tiên của queue

Bạn có thể truyền vào danh sách các queue ngăn cách bằng dấu `,` tới job được `listen` để thiết lập độ ưu tiên:

    php artisan queue:listen --queue=high,low

In this example, jobs on the `high` queue will always be processed before moving onto jobs from the `low` queue.
Ở ví dụ này, job ở `high` queue sẽ luôn luôn được thực hiện trước các job `low`.

#### Chỉ định timeout

Bạn có thể set giới hạn thời gian mỗi job được phép chạy:

    php artisan queue:listen --timeout=60

#### Chỉ định thời gian sleep

Bạn có thể thiét lập thời gian sleep trước khi thực hiện job mới:

    php artisan queue:listen --sleep=5

Chú ý là queue chỉ "sleep" nếu không có job nào trong queue. Nếu còn nhiều job, thì queue sẽ thực hiện liên tục mà không sleep.

#### Thực thi job đầu tiên trên queue

Để chỉ thực hiện job đầu tiên trên queue, bạn có thể sử dụng `queue:work`:

	php artisan queue:work

<a name="supervisor-configuration"></a>
### Cấu hình Supervisor

Supervisoer là một process monitor cho hệ điều hành Linux, và sẽ tự động khởi động lại `queue:listen` hay `queue:work` nếu thất bại. Để cài Supervisor trên Ubuntu, bạn có thể sử dụng câu lệnh sau:

    sudo apt-get install supervisor

File cấu hình cho Supervisor cơ bản được lưu trong `/etc/supervisor/conf.d`. Bên trong thư mục này, bạn có thể tạo bao nhiêu file cấu hình tuỳ ý để chỉ dẫn supervisor nên monitor process nào. Ví dụ, hãy tạo file `laravel-work.conf` để khởi động và monitor `queue:work`:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --daemon
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

Ở ví dụ này, khoá `numprocs` sẽ chỉ Supervisor chạy 8 tiến trình `queue:work` và monitor tất cả chúng, tự động khởi động lại nếu thất bại. Dĩ nhiên, bạn nên đổi phần `queue:work sqs` của khoá `command` để tương ứng với driver mà bạn sử dụng.

Khi file cấu hình được tạo, bạn có thể cập nhật cấu hình Supervisor và khởi động lại tiến trình sử dụng câu lệnh sau:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

Để biết thêm chi tiết về cấu hình và sử dụng Supervisor, tham khảo [tài liệu của Supervisor](http://supervisord.org/index.html). Ngoài ra, bạn có thể sử dụng [Laravel Forge](https://forge.laravel.com) để tự động cấu hình và quản lý cấu hình Supervisor từ giao diện web.

<a name="daemon-queue-listener"></a>
### Daemon Queue Listener

Câu lệnh `queue:work` có tuỳ chọn là `--daemon` để làm cho queue worker tiếp tục xử lý các job mà không cần khởi động lại framework. Điều này giảm thiểu được CPU usage rõ rệt so với sử dụng `queue:listen`:

Để khởi động queue worker ở chế độ daemon, sử dụng cờ `--daemon`:

    php artisan queue:work connection-name --daemon

    php artisan queue:work connection-name --daemon --sleep=3

    php artisan queue:work connection-name --daemon --sleep=3 --tries=3

Như bạn thấy, job `queue:work` hỗ trợ hầu hết các thông số giống với `queue:listen`. Bạn có thể gõ lệnh `php artisan help queue:work` để xem các tuỳ chọn có thể sử dụng.

Daemon queue worker không khởi động lại framework trước khi xử lý mỗi job. Vì thế, bạn cần phải chú ý giải phóng bất cứ tài nguyên nặng nào trước khi job kết thúc. Ví dụ, nếu ban jđang xử lý ảnh với thư viện GB, bạn nên giải phóng bộ nhớ với `imagedestroy` khi hoàn thiện.

<a name="deploying-with-daemon-queue-listeners"></a>
### Triển khai Daemon Queue Listeners

Vì daemon queue workers là các tiến trình tồn tại lâu, chúng sẽ không chọn thay đổi trong code bạn mà không cần khởi động lại. Vì vậy, đơn giản nhất để triển khai một chương trình sử dụng daemon queue worker là khởi động lại worker trong deployment script. Bạn có thể dễ dàng khởi động lại tất cả các worker bằng cách thêm vào câu lệnh sau trong deployment script:

    php artisan queue:restart

Câu lệnh này sẽ nhẹ nhàng chỉ dẫn các queue worker "die" sau khi hoàn thiện job hiện tại để đảm bảo không có job nào bị mất. Nhớ là, queue workers sẽ "die" khi lệnh `queue:restart` được thực thi, vì thế bạn nên chạy một trình quản lý process như Supervisor để tự động khởi động lại các queue driver.

> **Chú ý:** Câu lệnh này phụ thuộc vào hệ thống cache để đặt lịch khởi động. Về mặc định, APCu không làm việc với CLI job. Nếu bạn đang sử dụng APCu, thêm vào `apc.enable_cli=1` trong cấu hình APCu.

<a name="dealing-with-failed-jobs"></a>
## Xử lý các failed job

Vì mọi thứ không phải lúc nào cũng như mong muốn, sẽ có lúc queued job thất bại. Đừng lo, đó là điều tốt! Laravel có một cấu hình giúp chúng ta chỉ định số lần muốn thử lại cho failed job. Sau khi job vượt quá số lần thử tối đa, nó sẽ chèn vào trong bảng `failed_jobs`. Tên của bảng có thể thay đổi trong file cấu hình `config/queue.php`.

Để tạo migration cho bảng `failed_jobs`, bạn có thể sử dụng lệnh `queue:failed-table`:

    php artisan queue:failed-table

Khi chạy các [queue listener](#running-the-queue-listener), bạn có thể chỉ định số lần thử lại tối đa cho một job bằng tuỳ chọn `--tries` trên câu lệnh `queue:listen`:

    php artisan queue:listen connection-name --tries=3

<a name="failed-job-events"></a>
### Failed Job Events

Nếu bạn muốn đăng kí event lắng nghe khi một queued job thất bại, bạn có thể sử dụng hàm `Queue::failing`. Event này là một cách tốt để thông báo cho team của bạn qua email hoặc [HipChat](https://www.hipchat.com). Ví dụ, chúng ta có thể gắn một callback cho event này từ `AppServiceProvider` có sẵn trong Laravel:

    <?php

    namespace App\Providers;

    use Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->data
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

#### Các hàm xử lý thất bại trong job

Để điều khiển tốt hơn, bạn có thể thêm hàm `failed` trực tiếp trong queue job class, cho phép bạn thực hiện một hành động cụ thể khi thất bại:

    <?php

    namespace App\Jobs;

    use App\Jobs\Job;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendReminderEmail extends Job implements ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;

        /**
         * Execute the job.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function handle(Mailer $mailer)
        {
            //
        }

        /**
         * Handle a job failure.
         *
         * @return void
         */
        public function failed()
        {
            // Called when the job is failing...
        }
    }

<a name="retrying-failed-jobs"></a>
### Thử lại Failed Jobs

Để xem tất cả các failed jobs đã insert vào trong bảng `failed_jobs`, bạn có thể sử dụng câu lệnh `queue:failed`:

    php artisan queue:failed

Câu lệnh `queue:failed` sẽ liệt kế Job ID, connection, queue và thời gian thất bại. Job ID có thể được dùng để thử chạy lại. Ví dụ để thử chạy lại một failed job có ID là 5, gõ lệnh như sau:

    php artisan queue:retry 5

Để thực thi lại tất cả các failed jobs, sử dụng `queue:retry` với `all` thay cho ID:

    php artisan queue:retry all

Nếu bạn muốn xoá một failed job, sử dụng `queue:forget`:

    php artisan queue:forget 5

Để xoá tất cả các failed jobs, bạn có thể sử dụng `queue:flush`:

    php artisan queue:flush
