# HTTP Requests

- [HTTP Requests](#http-requests)
  - [Truy cập vào Request](#truy-cp-vao-request)
    - [Dependency Injection & Route Parameters](#dependency-injection-route-parameters)
    - [Đường dẫn Request & Phương thức](#ng-dn-request-phng-thc)
  - [PSR-7 Requests](#psr-7-requests)
  - [Input Trimming & Normalization](#input-trimming-normalization)
  - [Nhận input](#nhn-input)
    - [Nhận toàn bộ giá trị input](#nhn-toan-b-gia-tr-input)
    - [Nhận giá trị input](#nhn-gia-tr-input)
    - [Nhận dữ liệu từ câu truy vấn](#nhn-d-liu-t-cau-truy-vn)
    - [Nhận dữ liệu với Thuộc tính động](#nhn-d-liu-vi-thuc-tinh-ng)
    - [Nhận giá trị input từ JSON](#nhn-gia-tr-input-t-json)
    - [Nhận một phần của dữ liệu input](#nhn-mt-phn-ca-d-liu-input)
    - [Kiểm tra một giá trị input có tồn tại](#kim-tra-mt-gia-tr-input-co-tn-ti)
    - [Input cũ](#input-c)
    - [Cookies](#cookies)
    - [Tạo Cookie Instances](#to-cookie-instances)
  - [Files](#files)
    - [Lấy file được upload](#ly-file-c-upload)
    - [Kiểm tra upload thành công](#kim-tra-upload-thanh-cong)
    - [Đường dẫn File & Extensions](#ng-dn-file-extensions)
    - [Phương thức khác của File](#phng-thc-khac-ca-file)
    - [Chuyển vị trí file upload](#chuyn-v-tri-file-upload)
    - [Các phương thức khác với file](#cac-phng-thc-khac-vi-file)
  - [Configuring Trusted Proxies](#configuring-trusted-proxies)
    - [Trusting All Proxies](#trusting-all-proxies)

## Truy cập vào Request

Để lấy đối tượng của HTTP request hiện tại thông qua dependency injection, bạn phải type-hint `Illuminate\Http\Request` vào trong hàm khởi tạo của controller hay phương thức trong controller. Đối tượng của request hiện tại sẽ được tự động inject vào bởi [service container](container.md):

```PHP
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}
```

### Dependency Injection & Route Parameters

Nếu như hàm của controller cũng cần đầu vào từ các tham số của route, đơn giản chỉ cần ghi danh sách các đối số vào sau các dependencies. Ví dụ, nếu route được khai báo như sau:

```PHP
Route::put('user/{id}', 'UserController@update');
```

Bạn vẫn có thể type-hint `Illuminate\Http\Request` và truy cập vào tham số `id` của route bằng cách khai báo trong controller như sau:

```PHP
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
      * Update the specified user.
      *
      * @param  Request  $request
      * @param  string  $id
      * @return Response
      */
    public function update(Request $request, $id)
    {
        //
    }
}
```

#### Truy cập vào Request qua Route Closures

Bạn cũng có thể type-hint class ``Illuminate\Http\Request`` trong route Closure. Service sẽ tự động inject các request Closure khi nó sẽ được thực thi:

```PHP
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    //
});
```

### Đường dẫn Request & Phương thức

Đối tượng ``Illuminate\Http\Request`` cung cập một số phương thức để kiểm tra HTTP request cho ứng dụng và kế thừa class ``Symfony\Component\HttpFoundation\Request`` . Chúng ta sẽ thảo luận một số phương thức quan trọng dưới đây.

#### Nhận đường dẫn Request

Phương thức path trả về thông tin đường dẫn của request. Vì vậy, Nếu request gửi đến là  `http://domain.com/foo/bar`, phương thức path sẽ trả về `foo/bar`:

```PHP
$uri = $request->path();
```

Phương thức ``is`` sẽ cho phép bạn xác nhận những request gửi đến có đường dẫn khớp với pattern hay không. Bạn có thể sử dụng ký tự ``*`` khi sử dụng phương thức này:

```PHP
if ($request->is('admin/*')) {
    //
}
```

#### Nhận Request URL

Để nhận đường dẫn đầy đủ URL từ request gửi đến bạn có thể sử dụng phương thức ``url`` or  ``fullUrl``. Phương thức ``url`` sẽ trả về URL không có string query, trong khi phương thức ``fullUrl`` bao gồm cả string query:

```PHP
// Without Query String...
$url = $request->url();

// With Query String...
$url = $request->fullUrl();
```

#### Nhận phương thức Request

Phương thức ``method`` sẽ trả về phương thức HTTP tương ứng với request. Bạn có thể sử dụng phương thức ``isMethod`` để xác thực phương thức HTTP khớp với string:

```PHP
$method = $request->method();

if ($request->isMethod('post')) {
    //
}
```

## PSR-7 Requests

Tiêu chuẩn của PSR-7 đặt ra các interface cho HTTP messages, bao gồm cả request và response. Nếu bạn muốn lấy một đối tượng chuẩn của PSR-7 request, bạn cần phải cài đặt một số thư viện. Laravel sử dụng component Symfony HTTP Message Bridge để convert một Laravel request và response sang mẫu tương thích với PSR-7 chuẩn:

```PHP
composer require symfony/psr-http-message-bridge

composer require zendframework/zend-diactoros
```

Khi bạn đã cài đặt các thư viện này, bạn có thể lấy một request PSR-7 bằng cách type-hint kiểu request trên route hay controller:

```PHP
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    //
});
```

>Nếu bạn trả lại một response PSR-7 từ một route hoặc controller, nó sẽ tự động convert lại thành một response của Laravel và hiển thị.

## Input Trimming & Normalization

Theo mặc định, laravel bao gồm ``TrimStrings`` và ``ConvertEmptyStringsToNull`` ở trong ngăn xếp trung gian toàn cục của ứng dụng của bạn. Những phần mềm trung gian này được liệt kê trong ngăn xếp của lớp ``App\Http\Kernel``. Các phần mềm trung gian này sẽ tự động cắt tất cả các trường chuỗi đến theo yêu cầu, cũng như chuyển đổi bất kỳ trường chuỗi rỗng nào thành ``null`` . Điều này cho phép bạn không phải lo lắng về những mối quan tâm bình thường hóa trong các tuyến đường và bộ điều khiển của bạn.

Nếu bạn muốn vô hiệu hóa hành vi này, bạn có thể xóa hai phần mềm trung gian khỏi ngăn xếp phần mềm trung gian của ứng dụng bằng cách xóa chúng khỏi thuộc tính ``$middleware``của lớp ``App\Http\Kernel`` của bạn .

## Nhận input

### Nhận toàn bộ giá trị input

Bạn có thể lấy toàn bộ giá trị input dưới dạng mảng bằng cách sử dụng phương thức ``all``:

```PHP
$input=$request->all()
```

### Nhận giá trị input

Sử dụng một vài hàm cơ bản, bạn có thể lấy input từ người dùng qua `Illuminate\Http\Request`. Bạn không cần quan tâm về các hành động của HTTP sử dụng cho request.Bất kể nó là phương thức HTTP nào, phương thức input sử dụng có thể lấy được input từ người dùng:

```PHP
$name = $request->input('name');
```

Bạn có thể truyền vào giá trị mặc định ở đối số thứ hai trong hàm `input`. Giá trị này sẽ được trả lại nếu như giá trị input không có trong request:

```PHP
$name = $request->input('name', 'Sally');
```

Khi thực hiện trên form với mảng input, bạn có thể sử dụng kí hiệu "dot" để truy cập vào mảng:

```PHP
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

### Nhận dữ liệu từ câu truy vấn

Khi phương thức ``input`` lấy các giá trị từ toàn bộ câu lệnh yêu cầu(bao gồm cả chuỗi truy vấn), thì phương thức ``query`` chỉ nhận giá trị từ câu truy vấn

```PHP
$name=$request->query('name')
```

Nếu như câu truy vấn yêu cầu không có giá trị thì đối số thứ 2 của phương thức sẽ được trả về:

```PHP
$name = $request->query('name', 'Helen');
```

Bạn cũng có thể gọi phương thức ``query`` mà không có đối số nào để lấy toàn bộ giá trị của câu truy vấn dưới dạng mảng:

```PHP
$query = $request->query();
```

### Nhận dữ liệu với Thuộc tính động

Bạn cũng có thể truy xuất vào input sử dụng thuộc tính động trên đối tượng `Illuminate\Http\Request`. Ví dụ, nếu một trong các form có chứa trường là `name`, bạn có thể lấy giá trị được post lên như thế này:

```PHP
$name = $request->name;
```

Khi sử dụng thuộc tính động, Laravel đầu tiên sẽ tìm giá trị của tham số trên dữ liệu của request và rồi trong route parameter.

### Nhận giá trị input từ JSON

Khi gửi request kiểu JSON lên, bạn có thể truy xuất dữ liệu trong JSON thông qua hàm `input` với điều kiện là header của request `Content-Type` phải được set là `application/json`. Bạn có thể sử dụng cú pháp "dot" để truy xuất sâu hơn vào trong mảng JSON:

```PHP
$name = $request->input('user.name');
```

### Nhận một phần của dữ liệu input

Nếu bạn muốn lấy một tập nhỏ của input, bạn có thể sử dụng hai hàm `only` hay `except`. Cả hai hàm đều nhận một `array` hoặc một danh sách các đối số:

```PHP
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

### Kiểm tra một giá trị input có tồn tại

Để kiểm tra một giá trị có tồn tại trên request hay không, bạn có thể sử dụng hàm `has`. Hàm `has` trả về `true` nếu như giá trị tồn tại **và** không phải là chuỗi rỗng:

```PHP
if ($request->has('name')) {
    //
}
```

Khi đưa vào một mảng, phương thức ``has`` sẽ xác định trong tất cả dữ liệu có trong biến:

```PHP
if ($request->has(['name', 'email'])) {
    //
}
```

Nếu bạn muốn kiểm tra xem giá trị có rỗng không, có thể sử dụng phương thức ``filled``:

```PHP
if ($request->filled('name')) {
    //
}
```

### Input cũ

Laravel cho phép bạn giữ giá trị input từ một request sang request tiếp theo. Đặc điểm này đặc biệt hữu dụng khi bạn muốn thiết lập lại form sau khi phát hiện có lỗi. Tuy nhiên, nếu bạn sử dụng [validation services](validation.md) của Laravel, thì bạn không cần phải làm việc này vì các công cụ xử lý validation sẵn của Laravel đã tự động thực hiện rồi.

#### Flash input tới session

Hàm `flash` trong `Illuminate\Http\Request` sẽ flash input hiện tại vào trong [session](session.md) nên nó có thể sử dụng trong request tiếp theo của user tới ứng dụng:

```php
$request->flash();
```

Bạn cũng có thể sử dụng `flashOnly` và `flashExcept` để flash một tập nhỏ của dữ liệu request vào trong session:

```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

#### Flash input vào trong session rồi chuyển trang

Vì bạn thường muốn flash input cùng với chuyển trang vào trang trước đó, bạn có thể dễ dàng tạo móc nối input vào trong một redirect sử dụng hàm `withInput`:

```php
return redirect('form')->withInput();

return redirect('form')->withInput$request->except('password'));
```

#### Lấy dữ liệu cũ

Để lấy dữ liệu đã flash từ request trước đó, sử dụng hàm `old` của `Request`. Hàm `old` cung cấp một helper tiện ích cho việc lấy dữ liệu đã flash ra khỏi [session](session.md):

```php
$username = $request->old('username');
```

Laravel cũng cung cấp một helper toàn cục `old`. Nếu như bạn muốn hiển thị giá trị input cũ bên trong [Blade template](blade.md), thì sử dụng helper `old` sẽ tiện hơn. Nếu không có input cũ nào tìm thấy thì giá trị `null` sẽ được trả về:

```php
<input type="text" name="username" value="{{ old('username') }}">
```

### Cookies

#### Lấy cookies từ Request

Tất cả các cookies tạo bởi Laravel đều được mã hoá và kí với một mã xác thực, nghĩa là chúng sẽ bị coi là không hợp lệ nếu chúng bị thay đổi phía client. Để lấy cookie từ request, bạn có thể sử dụng hàm `cookie` từ `Illuminate\Http\Request`:

```php
$value = $request->cookie('name');
```

#### Gắn một cookie mới vào Response

Laravel cung cấp hàm `cookie` helper phục vụ như một factory đơn giản để tạo ra một đối tượng từ class `Symfony\Component\HttpFoundation\Cookie`. Những cookies này có thể được gắn với một đối tượng `Illuminate\Http\Response` sử dụng hàm `withCookie`:

```php
$response = new Illuminate\Http\Response('Hello World');

$response->withCookie('name', 'value', $minutes);

return $response;
```

Để tạo một cookie tồn tại lâu (long-lived), tồn tại trong vòng 5 năm, bạn có thể sử dụng hàm `forever` trong factory bằng cách gọi hàm `cookie` helper không có tham số nào, và rồi gọi móc nối với `forever` để tạo:

```php
$response->withCookie(cookie()->forever('name', 'value'));
```

### Tạo Cookie Instances

Nếu bạn muốn tạo một ``Symfony\Component\HttpFoundation\Cookie`` có thể response sau một khoảng thời gian, bạn có thể sử dụng helper global ``cookie``. Khi đó cookie sẽ không gửi lại cho client trừ khi nó được gán vào response instance:

```PHP
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

## Files

### Lấy file được upload

Bạn có thể lấy files uploaded từ một ``Illuminate\Http\Request`` bằng cách sử dụng phương thức  file hoặc sử dụng thuộc tính động. Phương thức ``file`` sẽ trả về một class  ``Illuminate\Http\UploadedFile``, nó kế thừa từ ``SplFileInfo`` class của PHP và cung cấp một số phương thức để tương tác với fiel:

```PHP
$file = $request->file('photo');

$file = $request->photo;
```

Bạn có thể kiểm tra nếu một file tồn tại trên request sử dụng hàm `hasFile`:

```PHP
if ($request->hasFile('photo')) {
  //
}
```

### Kiểm tra upload thành công

Ngoài việc kiểm tra nếu file upload tồn tại, bạn có thể kiểm tra xem có vấn đề gì trong quá trình file upload lên không thông qua việc sử dụng hàm `isValid`:

```PHP
if ($request->file('photo')->isValid()) {
    //
}
```

### Đường dẫn File & Extensions

Class ``UploadedFile`` ngoài ra còn chứa phương thức lấy đường dẫn đầy đủ và extension của file. Phương thức extension sẽ cho phép đoán extension trên dựa nội dung của file. Extension này có thể khác với extension được cung cấp bởi client:

```PHP
$path = $request->photo->path();

$extension = $request->photo->extension();
```

### Phương thức khác của File

Có một số phương thức tồn tại trong class ``UploadedFile``. Chi tiết xem tại [tài liệu API](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) của class để biết thêm chi tiết các phương thức đấy.

### Chuyển vị trí file upload

Để lưu một file uploaded, thông thường sử dụng một trong những cấu hình filesystems. Class  ``UploadedFile`` có phương thức  ``store`` nó sẽ chuyển file upload từ ổ cứng của bạn đến một nơi có thể là trên local của bạn hoặc ngay cả trên cloud storage như Amazon S3.

Phương thức ``store`` chấp nhận đường dẫn file nên được lưu trữ đường dẫn tương đối so với thư mục gốc cấu hình của filesystem. Đường dẫn không được chứa tên file, tên sẽ tự động được sinh ra bằng cách sử dụng mã hóa MD5 của nội dung file.

Phương thức ``store`` ngoài ra còn chấp nhận tham số thứ hai có tên của nơi mà bạn sử dụng để lưu file. Phương thức sẽ trả về đường dẫn tương đối của file đối với thư mục gốc:

```PHP
$request->file('photo')->move($destinationPath);

$request->file('photo')->move($destinationPath, $fileName);
```

Nếu bạn không muốn tên file được tự động tạo ra, bạn có thể sử dụng phương thứcstoreAs, nó sẽ chấp nhận các đối số như đường dẫn, tên file, và tên nơi lưu:

```PHP
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

### Các phương thức khác với file

Có nhiều hàm khác hỗ trợ cho việc xử lý file trong `UploadedFile`. Hãy tham khảo [tài liệu](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) để biết thêm thông tin về các hàm đó.

## Configuring Trusted Proxies

When running your applications behind a load balancer that terminates TLS / SSL certificates, you may notice your application sometimes does not generate HTTPS links. Typically this is because your application is being forwarded traffic from your load balancer on port 80 and does not know it should generate secure links.

To solve this, you may use the ``App\Http\Middleware\TrustProxies`` middleware that is included in your Laravel application, which allows you to quickly customize the load balancers or proxies that should be trusted by your application. Your trusted proxies should be listed as an array on the $proxies property of this middleware. In addition to configuring the trusted proxies, you may configure the proxy $headers that should be trusted:

```PHP
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Request;
use Fideloper\Proxy\TrustProxies as Middleware;

class TrustProxies extends Middleware
{
    /**
     * The trusted proxies for this application.
     *
     * @var array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    /**
     * The headers that should be used to detect proxies.
     *
     * @var string
     */
    protected $headers = Request::HEADER_X_FORWARDED_ALL;
}
```

If you are using AWS Elastic Load Balancing, your $headers value should be  Request::HEADER_X_FORWARDED_AWS_ELB. For more information on the constants that may be used in the $headers property, check out Symfony's documentation on trusting proxies.

### Trusting All Proxies

If you are using Amazon AWS or another "cloud" load balancer provider, you may not know the IP addresses of your actual balancers. In this case, you may use * to trust all proxies:

```PHP
/**
 * The trusted proxies for this application.
 *
 * @var array
 */
protected $proxies = '*';
```