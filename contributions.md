# Hướng dẫn đóng góp

- [Thông báo lỗi](#bug-reports)
- [Thảo luận phát triển phần core](#core-development-discussion)
- [Branch nào?](#which-branch)
- [Các lỗi bảo mật](#security-vulnerabilities)
- [Coding Style](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Thông báo lỗi

Để hỗ trợ hợp tác hiệu quả, Laravel rất khuyến khích các pull requests, không chỉ là thông báo lỗi. "Thông báo lỗi" cũng có thể được gửi dưới dạng một pull request có chứa một kết quả test thất bại.

Tuy nhiên, nếu bạn muốn viết một bug report, issue của bạn cần phải có tiêu đề và mô tả rõ ràng về issue đó. Bạn cũng nên đính kèm các thông tin liên quan càng chi tiết càng tốt và code mẫu để minh chứng cho issue đó. Mục đích của một bug report là để làm cho bạn - hay người khác - có thể tái hiện lại bug và tìm cách khắc phục một cách dễ dàng.

Hãy nhớ là, các bug report được tạo ra để người khác với lỗi tương tự có thể hợp tác với bạn để giải quyết nó. Đừng nên mong một bug report sẽ tự động có bất cứ hành động này người khác sẽ nhảy vào để giúp đỡ. Việc tạo bug report nhằm để hỗ trợ bạn và người khác có thể bắt đầu tìm cách để sửa lỗi.

Mã nguồn của Laravel được quản lý trên Github, và có các repository cho mỗi Laravel project:

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## Thảo luận phát triển phần core

Bạn có thể đề xuất tính năng mới hay cải thiện các hành vi hiện tại của Laravel bên trong [board thảo luận issue của Laravel Internals](https://github.com/laravel/internals/issues). Nếu bạn đề xuất một tính năng mới, hãy sẵn sàng triển khai một ít code có thể cần thiết để thoàn thiện.

Các thảo luận liên quan tới bug, tính năng mới và triển khai của các tính năng hiện tại đều thông qua channel `#internals` của [LaraChat](http://larachat.co). Taylor Otwell là người phát triển và quản lý chính của Laravel, về cơ bản là sẽ có mặt trong channel vào các ngày làm việc từ 8am-5pm (UTC-06:00 or America/Chicago).

<a name="which-branch"></a>
## Branch nào?

**Tất cả** các bản lỗi cần được đẩy tới branch stable mới nhật hoặc tới phiên bản LTS hiện tại (5.1). Các bản vá lỗi **không bao giờ** được phép gửi tới branch `master` trừ khi là vá lỗi cho tính năng chỉ xuất hiện trong phiên bản sắp phát hành.

Các tính năng **minor** mà **tương thích hoàn toàn với các phiên bản trước** với phiên bản hiện tại của Laravel cần được gửi tới branch stable mới nhất.

Các tính năng **major** luôn luôn được gửi tới branch `master`, mà có chứa trong phiên bản Laravel sắp phát hành.

Nếu bạn không chắc chắn nếu tính năng của bạn phù hợp với minor hay major, hãy hỏi Taylor Otwell trong channel `#internal` của [LaraChat](http://larachat.co).

<a name="security-vulnerabilities"></a>
## Các lỗi bảo mật

Nếu bạn phát hiện lỗi bảo mật trong Laravel, hãy gửi một email tới Taylor Otwell tại địa chỉ <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Tất cả các lỗi bảo mật sẽ được giải quyết nhanh chóng.

<a name="coding-style"></a>
## Coding Style

Laravel sử dụng tiêu chuẩn lập trình [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) và tiêu chuẩn [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) cho autoload.

<a name="phpdoc"></a>
### PHPDoc

Dưới đây là một ví dụ về khối tài liệu hợp lệ cho Laravel. Chú ý là thuộc tính `@param` được nối tiếp bởi hai dấu cách, kiểu dữ liệu, hai dấu cách nữa, và cuối cùng là tên biến:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

Nếu kiểu code của bạn chưa hoàn hảo, đừng lo lắng! [StyleCI](https://styleci.io/) sẽ tự động merge bất cứ bản vá style nào vào trong repository của Laravel sau khi có pull request được thực hiện merge. Việc này cho phép chúng ta tập trung vào việc xây dựng Laravel hơn là code style.
