# Mail

- [Giới thiệu](#introduction)
- [Gửi mail](#sending-mail)
    - [Attachments](#attachments)
    - [Inline Attachments](#inline-attachments)
    - [Thực hiện queue gửi mail](#queueing-mail)
- [Mail & Local Development](#mail-and-local-development)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp API rất gọn và đơn giản thông qua thư viện [SwiftMailer](http://swiftmailer.org) nổi tiếng. Laravel về cơ bản cung cấp driver cho SMTP, Mailgun, Mandrill, SparkPost, Amazon SES, hàm `mail` của PHP, và `sendmail`, cho phép bạn nhanh chóng bắt đầu gửi mail qua dịch vụ mail local hay cloud tuỳ theo lựa chọn của bạn.

### Yêu cầu cho driver

Các driver dựa trên API như Mailgun hay Mandrill thường đơn giản và nhanh hơn SMTP server. Tất cả các API driver yêu cầu sử dụng thư viện Guzzle HTTP được cài đặt trong ứng dụng của bạn. Bạn có thể cài đặt vào project bằng cách thêm dòng dưới đây vào trong file `composer.json`:

    "guzzlehttp/guzzle": "~5.3|~6.0"

#### Mailgun Driver

Để sử dụng Mailgun driver, đầu tiên cài Guzzle, rồi set giá trị `driver` ở trong file `config/mail.php` thành `mailgun`. Tiếp đó, xác nhận thông tin sau trong file `config/services.php`:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### Mandrill Driver

Để sử dụng Mandrill driver, đầu tiên cài Guzzle, rồi set giá trị `driver` trong `config/mail.php` thành `mandrill`. Sau đó, xác nhận thông tin sau trong file `config/services.php`:

    'mandrill' => [
        'secret' => 'your-mandrill-key',
    ],

#### SparkPost Driver

Để sử dụng SparkPost driver, đầu tiên cài Guzzle, rồi set giá trị `driver` trong file `config/mail.php` thành `sparkpost`. Tiếp đó, xác nhận thông tin sau trong file cấu hình `config/services.php`:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SES Driver

Để sử dụng Amazon SES driver, cài Amazon AWS SDK cho PHP. Bạn có thể cài thư viện này bằng cách thêm dòng dưới đây vào trong file `composer.json` ở mục `require`:

    "aws/aws-sdk-php": "~3.0"

Sau đó, set giá trị `driver` trong file `config/mail.php` thành `ses`. Sau đó, xác nhận các thông số dưới đây trong file cấu hình `config/services.php`:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="sending-mail"></a>
## Sending Mail

Laravel cho phép bạn lưu bản tin email trong [views](/docs/{{version}}/views). Ví dụ, để quản lý emails, bạn có thể tạo thư mục `emails` bên trong thư mục `resources/views`.

Để gửi một bản tin, sử dụng hàm `send` trong `Mail` [facade](/docs/{{version}}/facades). Hàm `send` nhận ba tham số. Tham số đầu tiên, là tên của [view](/docs/{{version}}/views) mà chứa nội dung của email. Tham số thứ hai là một mảng data mà bạn muốn truyền vào trong view. Cuối cùng là một `Closure` trong đó có nhận một instance của message, cho phép bạn tuỳ chỉnh thông tin recipients, subject và các thông tin khác của một bản tin email:

    <?php

    namespace App\Http\Controllers;

    use Mail;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send an e-mail reminder to the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendEmailReminder(Request $request, $id)
        {
            $user = User::findOrFail($id);

            Mail::send('emails.reminder', ['user' => $user], function ($m) use ($user) {
                $m->from('hello@app.com', 'Your Application');

                $m->to($user->email, $user->name)->subject('Your Reminder!');
            });
        }
    }

Khi chúng ta truyền vào một mảng chứa khoá `user` trong ví dụ ở trên, chúng ta có thể hiển thị tên của usser bên trong view của email sử dụng đoạn mã PHP sau:

    <?php echo $user->name; ?>

> **Lưu ý:** Một biến `$message` luôn được truyền vào views của emails, và cho phép [nhúng trực tiếp file đính kèm](#attachments). Vì thế, bạn nên tránh truyền biến `$message` vào trong view payload.

#### Tạo bản tin

Như bàn ở trước, tham số thứ ba truyền vào trong hàm `send` là một `Closure` cho phép bạn chỉ định nhiều thông số cho email. Với Closure này, bạn có thể truyền vào các thuộc tính cho bản tin như CC, BCC...

    Mail::send('emails.welcome', $data, function ($message) {
        $message->from('us@example.com', 'Laravel');

        $message->to('foo@example.com')->cc('bar@example.com');
    });

Đây là danh sách các hàm có thể sử dụng trong biến `$message`:

    $message->from($address, $name = null);
    $message->sender($address, $name = null);
    $message->to($address, $name = null);
    $message->cc($address, $name = null);
    $message->bcc($address, $name = null);
    $message->replyTo($address, $name = null);
    $message->subject($subject);
    $message->priority($level);
    $message->attach($pathToFile, array $options = []);

    // Attach a file from a raw $data string...
    $message->attachData($data, $name, array $options = []);

    // Get the underlying SwiftMailer message instance...
    $message->getSwiftMessage();

> **Lưu ý:** Instance `$message` được truyền cho `Mail::send` Closure mở rộng class message của SwiftMailer, cho phép bạn gọi bất cứ hàm nào của class đó để tạo nội dung bản tin cho email.

#### Gửi mail dạng text

Mặc định, view được truyền vào hàm `send` là dạng HTML. Tuy nhiên, bằng cách truyền vào một mảng vào tham số đầu của hàm `send`, bạn có thể chỉ định một view text để gửi bổ sung với HTML view:

    Mail::send(['html.view', 'text.view'], $data, $callback);

Hoặc, nếu bạn chỉ muốn email dạng text, bạn chỉ cần sử dụng khoá `text` trong mảng:

    Mail::send(['text' => 'view'], $data, $callback);

#### Gửi mail dạng chuỗi

Bạn có thể sử dụng hàm `raw` nếu muốn gửi một email dạng chuỗi trực tiếp:

    Mail::raw('Text to e-mail', function ($message) {
        //
    });

<a name="attachments"></a>
### Các file đính kèm

Để thêm đính kèm vào một email, sử dụng hàm `attach` của `$message` truyền trong Closure. Hàm `attach` nhận đường dẫn đầy đủ tới file ở tham số đầu:

    Mail::send('emails.welcome', $data, function ($message) {
        //

        $message->attach($pathToFile);
    });

Khi đính kèm files tới một bản tin, bạn cũng có thể chỉ định tên hiển thị hay / hoặc MIME type bằng cách truyền vào một `array` ở tham số thứ hai trong hàm `attach`:

    $message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);

<a name="inline-attachments"></a>
### Đính kèm inline

#### Nhúng ảnh vào trong view của email

Nhúng ảnh vào trong email về cơ bản là khá rườm rà; tuy nhiên, Laravel cung cấp sẵn một khá tiện lợi để nhúng ảnh vào trong mail và lấy ra sử dụng CID hơp lí. Để nhúng vào một ảnh inline, sử dụng hàm `embed` của biến `$message` bên trong email view. Nhớ rằng, Laravel tự động làm cho biến `$message` có thể sử dụng được trong tất cả các view của email:

    <body>
        Here is an image:

        <img src="<?php echo $message->embed($pathToFile); ?>">
    </body>

#### Nhúng dữ liệu thô vào trong email view

Nếu bạn đã có sẵn chuỗi dữ liệu thô và muốn nhúng vào trong bản tin email, bạn có thể sử dụng hàm `embedData` cho biến `$message`:

    <body>
        Here is an image from raw data:

        <img src="<?php echo $message->embedData($data, $name); ?>">
    </body>

<a name="queueing-mail"></a>
### Sử dụng queue gửi mail

#### Thực hiện queue một bản tin email

Vì việc gửi email sẽ làm tăng thời gian response của ứng dụng, nhiều lập trình viên chọn cách sử dụng queue cho bản tin email để gửi ở background. Laravel làm cho điều này khá dễ dàng sử dụng [API thống nhất cho queue](/docs/{{version}}/queues). Để queue một bản tin mail, sử dụng hàm `queue` trong `Mail` facade:

    Mail::queue('emails.welcome', $data, function ($message) {
        //
    });

Hàm này sẽ tự động lo việc đẩy job vào trong queue để gửi tin nhắn trong background. Dĩ nhiên, bạn sẽ cần thực hiện [cấu hình queue](/docs/{{version}}/queues) trước khi sử dụng tính năng này.

#### Trì hoãn việc queue mail

Nếu bạn muốn trì hoãn việc gửi các bản tin email đã được queue, bạn có thể sử dụng hàm `later`. Để bắt đầu, đơn giản chỉ cần truyền vào số giây mà bạn muốn trì hoãn việc gửi bản tin trong tham số đâu tiên của hàm:

    Mail::later(5, 'emails.welcome', $data, function ($message) {
        //
    });

#### Đẩy vào một queue cụ thể

Nếu bạn muốn đẩy vào một queue cụ thể, bạn sử dụng hàm `queueOn` và `laterOn`:

    Mail::queueOn('queue-name', 'emails.welcome', $data, function ($message) {
        //
    });

    Mail::laterOn('queue-name', 5, 'emails.welcome', $data, function ($message) {
        //
    });

<a name="mail-and-local-development"></a>
## Mail & Local Development

Khi phát triển ứng dụng có gửi email, bạn có thể không muốn thực sự gửi email tới địa chỉ thực. Laravel hỗ trợ vài cách để "vô hiệu hoá" việc gửi email này.

#### Log Driver

One solution is to use the `log` mail driver during local development. This driver will write all e-mail messages to your log files for inspection. For more information on configuring your application per environment, check out the [configuration documentation](/docs/{{version}}/installation#environment-configuration).
Một phương án là sử dụng driver `log` trong môi trường phát triển local. Driver này sẽ viết các bản tin emails vào trong log file để kiểm tra. Để tìm hiểu thêm thông tin về cấu hình ứng dụng cho từng môi trường, xem [mục cấu hình](/docs/{{version}}/installation#environment-configuration).

#### Universal To

Một cách khác được Laravel cung cấp đó là thiết lập một recipient toàn cục cho tất cả các email được framework gửi đi. Với cách này, tất cả các mail được tạo bởi ứng dụng sẽ được gửi tới một địa chỉ cụ thể, thay vì địa chỉ được ghi khi gửi mail. Việc này có thể thực hiện thông qua tuỳ chọn `to` trong file cấu hình `config/mail.php`:

    'to' => [
        'address' => 'dev@domain.com',
        'name' => 'Dev Example'
    ],

#### Mailtrap

Cuối cùng, bản có thể sử dụng dịch vụ bên ngoài như [Mailtrap](https://mailtrap.io) với driver là `smtp` để gửi email tới một hộp thư "dummy" mà bạn có thể coi nội dung trên một email client thật. Phương pháp này có lợi ích là cho phép bạn xem email cuối cùng được gửi đi và nhìn nội dung như thế nào trên Mailtrap.

<a name="events"></a>
## Events

Laravel bắn ra một event trước khi gửi bản tin mail. Hãy nhớ là, event này được bắn ra khi mail **đã được gửi đi**, chứ không phải lúc được queue. Bạn có thể đăng kí event listener tới event này trong `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSentMessage',
        ],
    ];

