# Envoy Task Runner

- [Giới thiệu](#introduction)
- [Viết Tasks](#writing-tasks)
    - [Task Variables](#task-variables)
    - [Nhiều Servers](#envoy-multiple-servers)
    - [Task Macros](#envoy-task-macros)
- [Thực thi Tasks](#envoy-running-tasks)
- [Thông báo](#envoy-notifications)
    - [HipChat](#hipchat)
    - [Slack](#slack)

<a name="introduction"></a>
## Giới thiệu

[Laravel Envoy](https://github.com/laravel/envoy) cung cấp một cú pháp đơn giản và gọn gàng cho việc khai báo các tác vụ thường được thực thi trên servers. Bằng việc sử dụng cú pháp của Blade, bạn có thể dễ dàng thiết lập các tác vụ cho triển khai dự án, chạy câu lệnh Artisan và nhiều nữa. Ở thời điểm hiện tại, Envoy chỉ hỗ trợ trên hệ điều hành của Mac và Linux.

<a name="envoy-installation"></a>
### Cài đặt

Trước tiên, bạn sử dụng Composer `global` để cài Envoy:

    composer global require "laravel/envoy=~1.0"

Cần phải chắc chắn đã đặt thư mục `~/.composer/vendor/bin` vào trong biến môi trường PATH để `envoy` có thể được tìm thấy khi được gọi trên terminal.

#### Cập nhật Envoy

Bạn cũng có thể sử dụng Composer để đảm bảo Envoy luôn là bản mới nhất:

    composer global update

<a name="writing-tasks"></a>
## Viết Tasks

Tất cả các Envoy task phải được khai báo trong một file `Envoy.blade.php` tại root của project. Đây là một ví dụ để bạn làm quen:

    @servers(['web' => 'user@192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

Như bạn thế, một mảng các `@servers` được khai báo ở ngay đầu file, cho phép bạn tham chiếu tới các server này trong tuỳ chọn `on` của khai báo task. Bên trong khai báo `@task`, bạn nên đặt đoạn mã Bash sẽ được thực thi trên server khi mà task được gọi.

#### Các tasks nội bộ

Bạn có thể khai báo một script để chạy local bằng cách khai báo tham chiếu tới server local:

    @servers(['localhost' => '127.0.0.1'])

#### Khởi tạo

Đôi lúc, bạn cần thực thi một số đoạn mã PHP trước khi thực thi Envoy task. Bạn có thể sử dụng ```@setup``` để khai báo các biến và thực thi các mã PHP bên trong:

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

Bạn cũng có thể sử dụng ```@include``` để thêm vào các file PHP ở bên ngoài:

    @include('vendor/autoload.php')

#### Xác nhận thực hiện task

Nếu bạn muốn yêu cầu xác nhận thực thi trước khi chạy một task trên server, bạn có thể thêm vào `confirm` trong khai báo task:

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="task-variables"></a>
### Task Variables

Khi cần thiết, bạn có thể truyền biến vào trong file Envoy, cách này sẽ cho phép bạn tuỳ biến tasks:

    envoy run deploy --branch=master

Bạn có thể sử dụng tuỳ chọn trong task thông qua cú pháp "echo" của Blade:

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="envoy-multiple-servers"></a>
### Nhiều Servers

Bạn có thể dễ dàng thực thi một task trên nhiều servers. Đầu tiên, khai báo các server trong khai báo `@servers`. Mỗi server phải được gán một tên khác biệt. Khi mà bạn đã khai báo đầy đủ, thì đơn giản chỉ cần đưa ra tên các server muốn thực thi trong khai báo task ở mảng `on`:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

Mặc định, task sẽ được thực thi trên các server một cách lần lượt. Nghĩa là, khi task thực thi xong ở server thì mới tiếp tục chuyển sang thực thi ở server tiếp theo.

#### Thực thi song song

Nếu bạn muốn thực thi task trên nhiều servers một cách song song, thêm vào tuỳ chọn `parallel` vào trong khai báo task:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="envoy-task-macros"></a>
### Task Macros

Macros cho phép bạn định nghĩa một tập hợp task được chạy lần lượt sử dụng một câu lệnh. Ví dụ, một macro `deploy` có thể chạy hai task `git` và `composer`:

    @servers(['web' => '192.168.1.1'])

    @macro('deploy')
        git
        composer
    @endmacro

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

Khi mà macro được định nghĩa, bạn có thể chạy qua câu lệnh đơn giản:

    envoy run deploy

<a name="envoy-running-tasks"></a>
## Thực thi Tasks

Để thực thi task từ `Envoy.blade.php`, thực thi câu lệnh `run`, chỉ cần truyền vào tên của task hay macro bạn muốn thực thi. Envoy sẽ thực thi task và hiển thị kết quả từ servers khi task đang chạy:

    envoy run task

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
## Thông báo

<a name="hipchat"></a>
### HipChat

Sau khi thực thi một task, bạn có thể gửi thông báo tới team của bạn qua HipChat sử dụng `@hipchat`. Directive này nhận API token, tên room và username để hiển thị:

    @servers(['web' => '192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

    @after
        @hipchat('token', 'room', 'Envoy')
    @endafter

Nếu muốn, bạn cũng có thể gửi một message tuỳ ý khi gửi nội dung gới phòng HipChat. Bất cứ biến nào có thể sử dụng trong Envoy task cũng có thể sử dụng được khi tạo message:

    @after
        @hipchat('token', 'room', 'Envoy', "$task ran in the $env environment.")
    @endafter

<a name="slack"></a>
### Slack

Ngoài HipChat, Envoy cũng hỗ trợ gửi thông báo tới [Slack](https://slack.com). Directive `@slack` nhận link hook, tên channel và nội dung tin bạn muốn gửi tới channel:

    @after
        @slack('hook', 'channel', 'message')
    @endafter

Bạn có thể nhận webhook URL bằng cách tạo một tích hợp `Incoming WebBooks` trên trang chủ của Slack. Đối số `hook` phải là giá trị đầy đủ của webhook URL được tạo ra qua Incoming WebHooks của Slack. Ví dụ:

    https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX

Bạn có thể cung cấp một trong những đối số channel sau:

- To send the notification to a channel: `#channel`
- To send the notification to a user: `@user`

