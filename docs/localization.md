# Localization

- [Giới thiệu](#introduction)
- [Sử dụng cơ bản](#basic-usage)
    - [Tạo số nhiều](#pluralization)
- [Ghi đè ngôn ngữ của vendor](#overriding-vendor-language-files)

<a name="introduction"></a>
## Giới thiệu

Chức năng localization của Laravel cung cấp một cách tiện lợi cho việc lấy các chuỗi dữ liệu từ các ngôn ngữ khác nhau cho phép hỗ trợ ứng dụng đa ngôn ngữ.

Các chuỗi ngôn ngữ được lưu trong file nằm trong thư mục `resources/lang`. Bên trong thư mục này có chứa các thư mục con cho mỗi ngôn ngữ được hỗ trợ bởi ứng dụng:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Tất cả các file ngôn ngữ đơn giản chỉ cần trả về mảng các chuỗi khoá. Ví dụ:

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

#### Cấu hình ngôn ngữ

Ngôn ngữ mặc định được thiết lập trong file cấu hình `config/app.php`. Dĩ nhiên, bạn có thể thay đổi giá trị này sao cho phù hợp với yêu cầu của bạn. Bạn cũng có thể thay đổi ngôn ngữ sử dụng mặc định bằng cách sử dụng hàm `setLocale` của `App` facade:

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

Bạn cũng có thể cấu hình "ngôn ngữ thay thế" khi mà ngôn ngữ đang được sử dụng không có chứa file ngôn ngữ tương ứng. Giống như ngôn ngữ mặc định, ngôn ngữ thay thế cũng có thể được cấu hình trong file `config/app.php`:

    'fallback_locale' => 'en',

Bạn có thể kiểm tra nếu một ngôn ngữ đang được sử dụng bằng cách gọi hàm `isLocale` trong `App` [facade](/docs/{{version}}/facades):

    if (App::isLocale('en')) {
        //
    }

Để lấy ngôn ngữ đang sử dụng hiện tại, gọi hàm `getLocale` trong `App` [facade](/docs/{{version}}/facades):

    return App::getLocale();

<a name="basic-usage"></a>
## Sử dụng cơ bản

Bạn có thể lấy các dòng trong file ngôn ngữ bằng cách sử dụng `trans` helper. Hàm `trans` nhận tên file và khoá trong file ngôn ngữ ở đối số đầu tiên. Ví dụ, để lấy dòng có khoá `welcome` trong file `resources/lang/messages.php`:

    echo trans('messages.welcome');

Nếu bạn sử dụng [Blade](/docs/{{version}}/blade) thì bạn có thể thực hiện output sử dụng `@lang` directive

    {{ trans('messages.welcome') }}

    @lang('messages.welcome')

Nếu dòng ngôn ngữ không tồn tại, thì hàm `trans` sẽ trả về khoá của dòng đó. Vì thế, ở ví dụ trên, hàm `trans` sẽ trả về `messages.welcome` nếu dòng này không tồn tại.

#### Đổi tham số trong dòng ngôn ngữ

Nếu muốn, bạn có thể khai báo các giá trị thay thế trong file ngôn ngữ. Tất cả các giá trị thay thế được tiền tố với dấu `:`. Ví dụ, bạn có thể khai báo một nội dung welcome với tên của giá trị thay thế:

    'welcome' => 'Welcome, :name',

Để thực hiện thay thế khi truy xuất lấy ngôn ngữ, truyền vào một mảng các giá trị thay thay thế vào tham số thứ hai của hàm `trans`:

    echo trans('messages.welcome', ['name' => 'dayle']);

Nếu giá trị thay thế chứa toàn kí tự in hoa, hoặc chỉ chữ cái đầu viết hoa, thì giá trị được truyền vào sẽ được viết hoa tương ứng:

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle


<a name="pluralization"></a>
### Tạo số nhiều

Tạo số nhiều là một vấn đề khá phức tạp vì các ngôn ngữ khác nhau đều có các quy tắc phức tạp để chuyển từ sang số nhiều. Bằng cách sử dụng dấu `|`, bạn có thể phân biệt được thể số ít hay thể số nhiều của một chuỗi:

    'apples' => 'There is one apple|There are many apples',

Sau đó, bạn có thể sử dụng hàm `trans_choice` để lấy dòng ngôn ngữ cho một giá trị count. Trong ví dụ này, khi mà giá trị count lớn hơn một, thì thể số nhiều của dòng ngôn ngữ được trả về:

    echo trans_choice('messages.apples', 10);

Vì phần phiên dịch của Laravel dựa trên thành phần phiên dịch của Symfony, bạn thậm chí có thể tạo các quy tắc phức tạp cho số nhiều:

    'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',

<a name="overriding-vendor-language-files"></a>
## Ghi đề ngôn ngữ của vendor

Vài packages có thể đính kèm file ngôn ngữ riêng của nó. Thay vì chỉnh sửa các file ngôn ngữ core, bạn có thể thực hiện ghi đè chúng bằng file riêng của bạn bằng cách đặt vào trong vị trí `resources/lang/vendor/{package}/{locale}`.

Vì thế, ví dụ, nếu bạn muốn ghi đè nội dung tiếng Anh trong file `messages.php` của một package tên là `skyrim/hearthfire`, bạn sẽ đặt file ngôn ngữ tại: `resources/lang/vendor/hearthfire/en/messages.php`. Trong file này bạn chỉ nên định nghĩa các dòng ngôn ngữ bạn muốn thay thế. Bất cứ dòng nào bạn không ghi đè thì nó vẫn lấy từ file gốc.
