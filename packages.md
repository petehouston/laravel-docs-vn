# Phát triển package

- [Giới thiệu](#introduction)
- [Service Providers](#service-providers)
- [Routing](#routing)
- [Resources](#resources)
    - [Views](#views)
    - [Translations](#translations)
    - [Cấu hình](#configuration)
- [Public Assets](#public-assets)
- [Xuất File Groups](#publishing-file-groups)

<a name="introduction"></a>
## Giới thiệu

Packages là cách chính thức để thêm chức năng vào trong Laravel. Package có thể là bất cứ thứ gì từ việc xử lý ngày giờ như [Carbon](https://github.com/briannesbitt/Carbon), hay nguyên bộ framework testing như [Behat](https://github.com/Behat/Behat).

Dĩ nhiên là có nhiều loại package. Một số package thì stand-alone, nghĩa là chúng có thể hoạt động với bất kì framework nào, không chỉ là Laravel. Cả Carbon và Behat là ví dụ về stand-alone package.Bất cứ package nào có thể sử dụng với Laravel chỉ đơn giản là yêu cầu trong file `composer.json`.

Mặt khác, một số package lại được tạo ra với mục đích sử dụng trong Laravel. Những package nào có route, controller, view và các cấu hình nhằm mục đích cải thiện ứng dụng Laravel. Hướng dẫn này phần lớn sẽ nói về việc phát triển các package dành cho Laravel.

<a name="service-providers"></a>
## Service Providers

[Service providers](/docs/{{version}}/providers) là điểm két nối giữa package của bạn và Laravel. Một service provider chịu trách nhiệm kết nối các thành phần vào trong [container](/docs/{{version}}/container) của Laravel và thông báo cho Laravel vị trí để load các tài nguyên của package như views, cấu hình, và các file localization.

Một service provider kế thừa từ class `Illuminate\Support\ServiceProvider` và chứa hai phương thức chính là `register` và `boot`. Class cơ sở `ServiceProvider` nằm trong Composer package là `illuminate/support` mà bạn cần phải thêm vào trong dependencies trong package của bạn.

Để biết thêm về cấu trúc và mục đích của service providers, hãy tham khảo [tài liệu về chúng](/docs/{{version}}/providers).

<a name="routing"></a>
## Routing

Để khai báo các routes cho package của bạn, đơn giản chỉ cần `require` các route file trong hàm `boot` của service provider nằm trong package của bạn. Từ trong route file, bạn có thể sử dụng `Route` facade để [đăng kí routes](/docs/{{version}}/routing) như là làm với một ứng dụng Laravel bình thường:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        if (! $this->app->routesAreCached()) {
            require __DIR__.'/../../routes.php';
        }
    }

<a name="resources"></a>
## Resources

<a name="views"></a>
### Views

Để đăng kí [views](/docs/{{version}}/views) trong package của bạn vào Laravel, bạn cần phải cho Laravel biết vị trí của các views ở đâu. Bạn có thể làm việc này thông qua hàm `loadViewsFrom` của service provider. Hàm `loadViewsFrom` nhận hai tham số: đương dẫn tới view template và tên package của bạn. Ví dụ, nếu tên package của bạn là "courier", thêm phần dưới đây vào trong hàm `boot` của service provider của bạn:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

Thư viện views được tham chiếu sử dụng hai dấu hai chấm `package::view`. Vì thế, bạn có thể load `admin` view từ package `courier` như sau:

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### Ghi đè lên views trong package

Khi bạn sử dụng hàm `loadViewsFrom`, Laravel hay đăng kí tại **hai** vị trí cho views của bạn: một nằm trong thư mục `resources/views/vendor` của ứng dụng và một nằm trong thư mục mà bạn chỉ định. Vì thế, với ví dụ `courier` thì: khi có yêu cầu một package view, Laravel sẽ đầu tiên thực hiện kiểm tra nếu một version riêng của view được cung cấp bởi lập trình viên trong `resources/views/vendor/courier`. Rồi sau đó, nếu view chưa được tuỳ chỉnh, Laravel sẽ tìm trong thư mục view của package mà bạn chỉ định trong hàm `loadViewsFrom`. Điều này làm cho người sử dụng dễ dàng có thể tuỳ chỉnh hay ghi đè lên thay thế views trong package của bạn.

#### Publishing Views

Nếu bạn muốn làm cho views có thể sẵn sàng cho việc publish tới thư mục `resources/views/vendor` của ứng dụng, bạn có thể sử dụng hàm `publishes` của service provider. Hàm `publishes` nhận một mảng bao gồm các đường dẫn tới view của package và đường đãn publish tương ứng.

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

Lúc này khi người dùng thực hiện câu lệnh `vendor:publish` với package của bạn, các view trong package sẽ được copy tới vị trí được thiết lập sẵn trong service provider.

<a name="translations"></a>
### Các file dịch ngôn ngữ

Nếu package của bạn chứa [các file dịch cho ngôn ngữ](/docs/{{version}}/localization), bạn có thể sử dụng hàm `loadTranslationsFrom` để cho Laravel biết cách load lên như thế nào, nếu package của bạn tên là "courier" bạn nên thêm như thế này vào trong hàm `boot` của service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

File dịch trong package được tham chiếu sử dụng hai dấu hai chấm `packacage::file.line`. Vì thế, bạn có thể load dòng `welcome` từ file `messages` trong package `courier` như sau:

    echo trans('courier::messages.welcome');

#### Publishing các file dịch

Nếu bạn muốn publish các file dịch trong package tới thư mục `resources/lang/vendor` của ứng dụng, bạn có thể sử dụng hàm `publishes` trong service provider. Hàm `publishes` nhận một mảng các đường dẫn của package và vị trí publish tương ứng. Ví dụ, để publish các file dịch cho package `courier` thì làm như sau:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }

Lúc này khi người dùng sử dụng câu lệnh `vendor:publish` cho package của bạn, thì các file dịch sẽ được publish tới vị trí được thiết lập sẵn.

<a name="configuration"></a>
### Cấu hình

Về cơ bản, bạn sẽ muốn publish các file cấu hình của package vào trong thư mục `config` của ứng dụng. Điều này cho phép người dùng package của bạn sẽ dễ dàng ghi đè thông số cấu hình mặc định của bạn. Để publish một file cấu hình, chỉ cần sử dụng hàm `publishes` trong hàm `boot` của service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

Lúc này, khi người dùng package của bạn thực thi câu lệnh `vendor:publish`, các file sẽ được copy tới thư mục được thiết lập sẵn. Dĩ nhiên là khi file cấu hình được publish, nó có thể được truy cập để sử dụng như các file cấu hình khác:

    $value = config('courier.option');

#### Cấu hình mặc định cho package

Bạn cũng có thể chọn để gộp cấu hình trong package của bạn với cấu hình của ứng dụng. Điều này cho phép người dùng có thể thêm vào những option mà họ muốn ghi đè lên trong bản copy của file cấu hình được publish. Để gộp file cấu hình, sử dụng hàm `mergeConfigFrom` bên trong hàm `register` của service provider:

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

<a name="public-assets"></a>
## Public Assets

Package của bạn có thể chứa các assets như file Javascript, CSS, và ảnh. Để publish những asset này tới thư mục `public`, sử dụng hàm `publishes` trong service provider. Ở ví dụ này, chúng ta cũng sẽ thêm vào một tag `public` cho nhóm asset, để có thể được sử dụng khi publish các nhóm asset liên quan:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

Lúc này, khi mà người dùng package của bạn thực thi câu lệnh `vendor:publish`, asset của bạn sẽ được copy tới vị trí được chỉ định. Khi mà bạn cần ghi đè lên asset mỗi khi package được cập nhật, bạn có thể sử dụng thêm cờ `--force`:

    php artisan vendor:publish --tag=public --force

Nếu bạn muốn đảm bảo là các file public asset luôn được cập nhật, bạn có thể thêm câu lệnh này vào trong khoá `post-update-cmd` trong file `composer.json`.

<a name="publishing-file-groups"></a>
## Publishing file theo nhóm

Bạn có thể muốn publish các nhóm file assets và resources một cách riêng rẽ. Ví dụ như này, bạn có thể muốn người dụng của bạn có thể publish file cấu hình mà không cần ép buộc phải publish file assets của package cùng một lúc. Bạn có thể làm vậy bằng cách "đặt tag" cho chúng khi gọi hàm `publishes`. Ví dụ, cùng nhau khai báo hai nhóm publish trong hàm `boot` của service provider trong package:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }

Lúc này, người dùng có thể publish các nhóm riêng rẽ bằng cách tham chiếu tới tên tag của chúng sử dụng option `--tag` khi gọi câu lệnh `vendor:publish`:

    php artisan vendor:publish --provider="Vendor\Providers\PackageServiceProvider" --tag="config"
