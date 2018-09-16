# Cấu trúc thư mục

- [Cấu trúc thư mục](#cu-truc-th-mc)
  - [Giới thiệu](#gii-thiu)
  - [Thư mục gốc](#th-mc-gc)
    - [Thư mục app](#th-mc-app)
    - [Thư mục Bootstrap](#th-mc-bootstrap)
    - [Thư mục Config](#th-mc-config)
    - [Thư mục Database](#th-mc-database)
    - [Thư mục Public](#th-mc-public)
    - [Thư mục Resources](#th-mc-resources)
    - [Thư mục Routes](#th-mc-routes)
    - [Thư mục Storage](#th-mc-storage)
    - [Thư mục Tests](#th-mc-tests)
    - [Thư mục Vendor](#th-mc-vendor)
  - [Thư mục App](#th-mc-app)
    - [Thư mục Broadcasting](#th-mc-broadcasting)
    - [Thư mục Console](#th-mc-console)
    - [Thư mục Events](#th-mc-events)
    - [Thư mục Exceptions](#th-mc-exceptions)
    - [Thư mục Http](#th-mc-http)
    - [Thư mục Jobs](#th-mc-jobs)
    - [Thư mục Listeners](#th-mc-listeners)
    - [Thư mục Mail](#th-mc-mail)
    - [Thư mục Notifications](#th-mc-notifications)
    - [Thư mục Policies](#th-mc-policies)
    - [Thư mục Providers](#th-mc-providers)
    - [Thư mục Rules](#th-mc-rules)

## Giới thiệu

Mặc định cấu trúc thư mục ứng dụng Laravel được thiết kế để xây dựng cả ứng dụng nhỏ và ứng dụng lớn. Tất nhiên, bạn có thể hoàn toàn tổ trức lại cấu trúc thư mục theo ý bạn muốn. Laravel hầu như không áp đặt những hạn chế về nơi các class nằm ở thư mục nào - miễn là Composer có thể tự động tải được các class.

#### Thư mục Models ở đâu

Khi bắt đầu làm việc với Laravel, rất nhiều developer bị bối rối vì cảm thấy thiếu thiếu một thư mục ``models``. Tuy nhiên, Laravel đã cố ý không tạo một folder models đó. Chúng ta thấy từ "models" là mơ hồ vì nó có rất nhiều nghĩa khác nhau từ nhiều người khác nhau. Một số developer xem "model" như là toàn bộ logic của ứng dụng, trong khi những sô khác thì xem "models" như là các class có thể tương tác với database.

Chính vì lý do đấy, Laravel chọn đặt Eloquent model mặc định ở thư mục app, và cho phép các developer có thể đặt chúng ở nơi khác mà họ muốn.

## Thư mục gốc

### Thư mục app

Thư mục ``app``, như bạn mong đợi, nó sẽ chứa tất cả các cốt lõi trong ứng dụng của bạn. Chúng ta sẽ khám phá vhi tiết về nó sớm thôi; tuy nhiên, hầu hết các class trong ứng dụng của bạn đều ở trong đây.

### Thư mục Bootstrap

Thư mục ``bootstrap`` chứa những file khởi động của frameword và những file cấu hình autoloading. Ngoài ra nó còn có thư mục cache chứa những file mà framework sinh ra để tăng hiệu năng như route và services cache files.

### Thư mục Config

Thư mục ``config``, đúng như cái tên của nó, chứa tất cả những file cấu hình. Thật tuyệt vời khi bạn lướt qua tất cả các file của nó với những cấu hình có sẵn cho bạn.

### Thư mục Database

Thư mục ``database`` chứa những file database migration, model factories và seeds. Nếu bạn muốn, bạn cũng có thể sử dụng nó để tổ chức một cơ sử dữ liệu SQLite.

### Thư mục Public

Thư mục public chứa file index.php, đó là điểm mấu chốt cho tất cả các request vào ứng dụng của bạn. Ngoài ra nó còn chứa một số tài nguyên như ảnh, JavaScript, và CSS.

### Thư mục Resources

Thư mục ``resources`` chứa những view và raw, những tài nguyên chưa compiled như LESS, SASS, hoặc JavaScript. Nó còn chứa tất cả các file ngôn ngữ trong ứng dụng của bạn.

### Thư mục Routes

Thư mục ``routes`` chứa tất cả các định nghĩa route trong ứng dụng của bạn. Mặc định, có ba file route được thêm vào với Laravel: ``web.php``, ``api.php``, ``console.php`` và ``channels.php``.

File ``web.php`` chứa những routes là ``RouteServiceProvider`` ở trong ``web`` nhóm middleware, Nó cung cấp các trạng thái session, CSRF protection, và cookie encryption. Nếu ứng dụng của bạn không có stateless, RESTful API, thì hầu hết các routes bạn định nghĩa nằm ở file ``web.php``.

File ``api.php`` chứa những routes là ``RouteServiceProvider`` ở trong api nhóm middleware, nó cung cấp rate limiting. Nhữn routes đã được xác định stateless, vì vậy những request gửi đến ứng dụng của bạn thông qua routes sẽ được xác thực bằng tokens và sẽ không có quyền truy cập trạng thái session.

File ``console.php`` là nơi để bạn định nghĩa tất cả các Closure dựa trên câu lệnh console. Mỗi Closure bị bị ràng buộc bởi thể hiện lệnh cho phép tiếp cận đơn giản để tương tác với các phương thức IO của mỗi lệnh. Mặc dù nó không định nghĩa HTTP routes, nó định nghĩa console based entry points (routes) trong ứng dụng của bạn.

File ``channels.php`` là nơi bạn đăng ký tất cả các kênh broadcasting mà ứng dụng của bạn hỗ trợ

### Thư mục Storage

Thư mục storage chứa các file biên dịch Blade templates của bạn, file based sessions, file caches, và những file sinh ra từ framework. Bên trong nó bao gồm các thư mục ``app``, ``framework``, và ``logs``. Thư mục ``app`` dùng để chứa những file sinh ra bởi ứng dụng của bạn. Thư mục ``framework`` chứa những file sinh ra từ framework và caches. Cuối cùng, thư mục ``logs`` chứa những file logs.

Thư mục ``storage/app/public`` lưu những file người dùng tạo ra như ảnh đại diện, nó phải được để công khai. Bạn nên tạo một liên kết tại ``public/storage`` đến thư mục này bằng cách sử dụng lệnh  ``php artisan storage:link``.

### Thư mục Tests

Thư mục ``tests`` chứa những file tests của bạn. Ví dụ PHPUnit nó cung cấp rất đầy đủ. Mỗi một class test nên chứa hậu tố với từ ``Test``. Bạn có thể chạy class test của bạn bằng lệnh ``phpunit`` hoặc ``php vendor/bin/phpunit``.

### Thư mục Vendor

The ``vendor`` chứa các dependencies của Composer.

## Thư mục App

Phần lớn các ứng dụng của bạn đặt ở thư mục ``app``. Mặc định, thư mục này có namespaced là ``App`` và được load tự động bởi Composer sử dụng PSR-4 autoloading standard.

Thư mục ``app`` chứa một vài thư mục con bên trong như ``Console``, ``Http``, và ``Providers``. Hãy nghĩ về  ``Console`` và ``Http`` là các thư mục cung cấp API cho phần code lỗi ứng dụng của bạn. Giao thức HTTP và CLI là hai cơ chế tương tác với ứng dụng của bạn, nhưng nó không thật sự chứa logic của ứng dụng. Bạn có thể hiểu, Chúng ta có hai cách để thực thi lệnh đến ứng dụng của bạn. Thư mục ``Console`` chứa tất cả các lệnh Artisan của bạn, trong khi thư mục ``Http`` chứa controllers, middleware, và requests.

Khi bạn sử dụng lệnh make Artisan thì sẽ có một loạt các thư mục sẽ được tạo ra bên trong thư mục  app. Ví dụ, thư mục app/Jobs sẽ không tồn tại cho đến khi bạn thực thi lệnh ``make:job`` Artisan sinh ra một class job.

>Rất nhiều class được sinh ra bên trong thư mục app bằng lệnh Artisan . Bạn có thể xem lại những lệnh nào đang tồn tại bằng lệnh ``php artisan list make`` trong terminal.

### Thư mục Broadcasting

Thư mục ``Broadcasting`` chứa tất cả các lớp kênh phát sóng cho ứng dụng của bạn. Các lớp này được tạo ra bằng cách sử dụng lệnh ``make:channel``. Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn khi bạn tạo kênh đầu tiên của mình. Để tìm hiểu thêm về các kênh, hãy xem tài liệu về [sự kiện phát sóng](broadcasting.md) .

### Thư mục Console

Thư mục ``Console`` chứa tất cả những file Artisan commands ứng dụng của bạn. Đó là những lệnh được sinh ra bằng lện ``make:command``. Ngoài ra nó còn chứa console kernel của ứng dụng, nó là nơi bạn có thể chỉnh đăng ký Artisan commands và [scheduled tasks](scheduling.md) được định nghĩa.

### Thư mục Events

Mặc định thư mục này không tồn tại, nhưng nó sẽ được sinh ra bằng lệnh ``event:generate`` và lệnh  ``make:event``. Thư mục ``Events``, như bạn mong đợi, nó sẽ chứa các event classes. Events có thể được sử dụng để thông báo các phần khác của ứng dụng của bạn mà hạnh động đấy nhất định xảy ra, ngoài ra nó khả năng của nó rất linh hoạt và decoupling.

### Thư mục Exceptions

Thư mục ``Exceptions`` chứa các xử lý exception trong ứng dụng của bạn ngoài ra nó còn là nơi tốt để bắn ra nhiều exception bởi ứng dụng. Nếu bạn muốn tùy chỉnh những exception hoặc rendered, bạn nên chỉnh lại class ``Handler`` bên trong thư mục.

### Thư mục Http

Thư mục ``Http`` chứa các controllers, middleware, và form requests. Tất cả các logic xử lý requests vào ứng dụng của bạn sẽ nằm ở trong thư mục này.

### Thư mục Jobs

Mặc định thư mục này không tồn tại, nhưng nó sẽ được sinh ra nếu bạn thực thi lệnh ``make:job Artisan``. Thư mục ``Jobs`` chứa các [queueable jobs](queues.md) ứng dụng của bạn. Jobs có thể được xếp hàng bởi ứng dụng hoặc để chạy đồng bộ bên trong vòng đời request hiện tại. Jobs có thể chạy đồng bộ trong khi request hiện tại là một "commands" khi thực hiện trong command pattern.

### Thư mục Listeners

Mặc định thư mục này không tồn tại, nhưng nó sẽ được sinh ra nếu bạn thực thi lệnh  event:generate hoặc ``make:listener`` Artisan. Thư mục ``Listeners`` chứa các class xử lý các [events](events.md). Event listeners nhận được một thể hiện event và thực hiện logic khi hồi đáp cho sự kiện đã được bắn ra. Ví dụ, một ``UserRegistered`` event phải được sử lý bởi một ``SendWelcomeEmail`` listener.

### Thư mục Mail

Mặc định thư mục này không tồn tại, nhưng nó sẽ được sinh ra nếu bạn thực thi lệnh ``make:mail`` Artisan. Thư mục ``Mail`` chứa tất cả các class đại diện cho mail được gửi bởi ứng dụng của bạn. Mail objects cho phép bạn đóng gói tất cả các logic thành một, một class có thể được gửi bởi lệnh  ``Mail::send``.

### Thư mục Notifications

Mặc định thư mục này không tồn tại, nhưng nó sẽ được sinh ra nếu bạn thực thi lệnh  ``make:notification`` Artisan. Thư mục ``Notifications`` chứa tất cả các "giao dịch" thông báo đã gửi bởi ứng dụng, ví dụ đơn giải là thông báo về event xảy ra trong ứng dụng của bạn. Ngoài ra notifications của Laravel còn có thể thông báo cho email, Slack, SMS, hoặc lưu trong database.

### Thư mục Policies

Mặc định thư mục này không tồn tại, nhưng nó sẽ được sinh ra nếu bạn thực thi lệnh ``make:policy`` Artisan. Thư mục ``Policies`` chứa các lớp quy ước cấp quyền ứng dụng của bạn. Các quy ước được dùng để quyết định user có thể thực hiện hành động đối với một tài nguyên nào đó. Để xem chi tiếp, bạn có thể xem tại authorization documentation.

### Thư mục Providers

Thư mục ``Providers`` chứa tất cả các service providers ứng dụng của bạn. Service providers khởi động ứng dụng của bạn bằng các services trong service container, đăng ký events, hoặc thực hiện bất kỳ một công việc khác để chuẩn bị cho request đến ứng dụng của bạn.

Khi bạn mới cài xong project, thư mục đã chứa một số providers. Bạn có thể thoải mái thêm providers của bạn vào nếu cần.

### Thư mục Rules

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn nếu bạn thực hiện lệnh ``make:rule`` Artisan. Thư mục chứa các đối tượng quy tắc hợp lệ tùy chỉnh cho ứng dụng của bạn. Quy tắc được sử dụng để đóng gói logic xác nhận phức tạp trong một đối tượng đơn giản. Để biết thêm thông tin, hãy xem [Validation.md](validation.md)