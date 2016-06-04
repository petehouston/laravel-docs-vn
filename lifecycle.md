# Request Lifecycle

- [Giới thiệu](#introduction)
- [Tổng quan về Lifecycle](#lifecycle-overview)
- [Tập trung vào Service Providers](#focus-on-service-providers)

<a name="introduction"></a>
## Giới thiệu

Khi sử dụng bất cứ công cụ nào trong "thế giới thực", bạn sẽ cảm thấy tự tin hơn nếu như bạn hiểu được công cụ đó hoạt động như thế nào. Phát triển ứng dụng cũng không khác gì mấy. Khi bạn hiểu được các công cụ phát triển hoạt động thế nào, bạn sẽ cảm thấy thoải mái và tự tin hơn khi sử dụng chúng.

Mục tiêu của tài liệu này là hướng dẫn cho bạn một cái nhìn lớn ở tầng cao tổng quát về "cách hoạt động" của Laravel framework. Bằng cách hiểu hơn về mọi thứ quanh framework, thì bạn sẽ cảm thấy ít "ma thuật" và tự tin hơn trong quá trình phát triển ứng dụng.

Nếu bạn không hiểu các thuật ngữ lúc nà, đừng buồn! Hãy thử cố gắng nắm bắt cơ bản và hiểu biết của bạn sẽ tốt dần khi mà bạn thực hiện khám phá các phần khác trong bộ tài liệu này.

<a name="lifecycle-overview"></a>
## Tổng quát về Lifecycle

### Điều đầu tiên

Đầu vào của mọi request tới Laravel nằm trong file `public/index.php`. Mọi request được đẩy tới file này là nhờ web server (Apache / Nginx). File `index.php` không chứa nhiều code lắm. Nó chỉ chứa điểm bắt đầu để thực hiện load các phần còn lại của framework.

File `index.php` load các cấu hình tự động load sinh ra từ Composer, và thực hiện trả về một đối tượng của Laravel nằm trong file `bootstrap/app.php`. Việc đầu tiên mà Laravel làm đó là tạo một đối tượng của ứng dụng, [service container](/docs/{{version}}/container).

### HTTP / Console Kernels

Tiếp đến, các request đến được gửi hoặc tới HTTP kernel hay console kernel, phụ thuộc vào kiểu request đi vào ứng dụng. Hai kernel này phục vụ như khu trung tâm điều phối dòng request. Bây giờ, hãy chỉ tập trung vào HTTP kernel, được đặt trong `app/Http/Kernel.php`.

HTTP kernel kết thừa từ class `Illuminate\Foundation\Http\Kernel`, class này định nghĩa một mảng các `bootstrappers` mà sẽ được thực thi khi mà request được xử lý. Các bootstrapper này cấu hình các việc xử lý lỗi, cấu hình log, [nhận biết môi trường ứng dụng](/docs/{{version}}/installation#environment-configuration), và thực hiện các công việc khác ngay trước khi request được xử lý.

HTTP kernel cũng định nghĩa danh sách các HTTP [middleware](/docs/{{version}}/middleware) mà các yêu cầu phải đi qua trước khi được xử lý bởi ứng dụng. Những middleware này xử lý việc đọc và ghi các [HTTP session](/docs/{{version}}/session), chỉ định nếu ứng dụng đang ở chế độ bảo trì, [xác nhận CSRF token](/docs/{{version}}/routing#csrf-protection), và nhiều nữa.

HTTP kernel chứa một phương thức là `handle` khá là đơn giản, nhận một `Request` và trả về một `Response`. Hãy coi Kernel như một hộp đen lớn tượng trưng cho ứng dụng của bạn. Cung cấp cho nó các HTTP request và nó sẽ trả về các HTTP response.

#### Service Providers

Một trong những thao tác khởi tạo kernel quan trọng nhất đó là thực hiện load các [service providers](/docs/{{version}}/providers). Tất cả các service providers được cấu hình trong file `config/app.php` ở mảng `$providers`. Đầu tiên, hàm `register` sẽ được gọi ở tất cả các providers, rồi sau đó, khi mà các providers đã được đăng kí đầy đủ, thì hàm `boot` sẽ được gọi.

Service providers chịu trách nhiệm khởi tạo tất cả các thành phần khác nhau của framework, ví dụ như database, queue, validation, và routing. Vì chúng thực hiện khởi tạo và cấu hình các chức năng được Laravel cung cấp, service providers là phần quan trọng nhất trong tiến trình khởi tạo của Laravel.

#### Điều phối Request

Khi mà ứng dụng đã được khởi tạo và các service providers đã được đăng kí, `Request` sẽ được đưa xuống cho router. Router sẽ thực hiện đưa các request này xuống một route hoặc controller để xử lý, cũng như thực thi các middleware cụ thể riêng cho từng route.

<a name="focus-on-service-providers"></a>
## Tập trung vào Service Providers

Service providers thực sự là chìa khoá để khởi tạo Laravel. Khi đối tượng của ứng dụng được tạo ra, các service providers đã được đăng kí, thì các request sẽ được đẩy tới phần đã được khởi tạo của ứng dụng. Đơn giản chỉ vậy thôi!

Nắm bắt rõ ràng cách mà Laravel được xây dựng và khởi tạo thông qua service providers rất là giá trị. Dĩ nhiên là các service providers mặc định được lưu trong thư mục `app/Providers`.

Mặc định, `AppServiceProvider` là không có gì cả. Provider này là nơi hợp lý để thêm vào các phần khởi tạo và liên kết riêng cho ứng dụng của bạn. Dĩ nhiên là đối với các ứng dụng lớn, bạn sẽ muốn tạo vài service providers, mỗi cái sẽ thực hiện khởi tạo một công việc khác nhau.