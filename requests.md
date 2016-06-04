# HTTP Requests

- [Truy cập vào Request](#accessing-the-request)
    - [Thông tin Request cơ bản](#basic-request-information)
    - [PSR-7 Requests](#psr7-requests)
- [Nhận input từ Request](#retrieving-input)
    - [Input cũ](#old-input)
    - [Cookies](#cookies)
    - [Files](#files)

<a name="accessing-the-request"></a>
## Truy cập vào Request

Để lấy đối tượng của HTTP request hiện tại thông qua dependency injection, bạn phải type-hint `Illuminate\Http\Request` vào trong hàm khởi tạo của controller hay phương thức trong controller. Đối tượng của request hiện tại sẽ được tự động inject vào bởi [service container](/docs/{{version}}/container):

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

Nếu như hàm của controller cũng cần input từ route parameter, đơn giản chỉ cần ghi danh sách các đối số vào sau các dependencies. Ví dụ, nếu route được khai báo như sau:

    Route::put('user/{id}', 'UserController@update');

Bạn vẫn có thể type-hint `Illuminate\Http\Request` và truy cập vào route parameter `id` bằng cách khai báo trong controller như sau:

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

<a name="basic-request-information"></a>
### Thông tin Request cơ bản

Đối tượng `Illuminate\Http\Request` cung cấp một số phương thức để kiểm tra HTTP request và kế thừa từ class cơ sở là `Symfony\Component\HttpFoundation\Request`. Sau đây là một vài hàm hữu dùng của class:

#### Lấy Request URI

Hàm `path` trả về URI của request. Vì vậy, nếu request đến ở mục tiêu là `http://domain.com/foo/bar`, hàm `path` sẽ trả về `foo/bar`:

    $uri = $request->path();

Hàm `is` cho phép bạn xác nhận URI của request đến có khớp với một pattern nào đó không. Bạn có thể sử dụng kí tự `*` khi sử dụng hàm này:

    if ($request->is('admin/*')) {
        //
    }

Để lấy URL đầy đủ, không chỉ là đường dẫn, bạn có thể sử dụng `url` hoặc `fullUrl`:

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

Bạn cũng có thể lấy URL đầy đủ và thêm vào các query parameters. Ví dụ, nếu request có mục tiêu là `http://domain.com/foo`, thì hàm sau sẽ trả về `http://domain.com/foo?bar=baz`:

    $url = $request->fullUrlWithQuery(['bar' => 'baz']);

#### Lấy tên hàm được gọi của Request

Hàm `method` sẽ trả về hành động HTTP của request. Bạn cũng có thể sử dụng `isMethod` để xác nhận hành động của HTTP request có khớp một chuỗi hay không:

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### PSR-7 Requests

Tiêu chuẩn của PSR-7 đặt ra các interface cho HTTP messages, bao gồm cả request và response. Nếu bạn muốn lấy một đối tượng chuẩn của PSR-7 request, bạn cần phải cài đặt một số thư viện. Laravel sử dụng component Symfony HTTP Message Bridge để convert một Laravel request và response sang mẫu tương thích với PSR-7 chuẩn:

    composer require symfony/psr-http-message-bridge

    composer require zendframework/zend-diactoros

Khi bạn đã cài đặt các thư viện này, bạn có thể lấy một request PSR-7 bằng cách type-hint kiểu request trên route hay controller:

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

Nếu bạn trả lại một response PSR-7 từ một route hoặc controller, nó sẽ tự động convert lại thành một response của Laravel và hiển thị.

<a name="retrieving-input"></a>
## Nhận input từ request

#### Nhận giá trị input

Sử dụng một vài hàm cơ bản, bạn có thể lấy input từ người dùng qua `Illuminate\Http\Request`. Bạn không cần quan tâm về các hành động của HTTP sử dụng cho request:

    $name = $request->input('name');

Bạn có thể truyền vào giá trị mặc định ở đối số thứ hai trong hàm `input`. Giá trị này sẽ được trả lại nếu như giá trị input không có trong request:

    $name = $request->input('name', 'Sally');

Khi thực hiện trên form với mảng input, bạn có thể sử dụng kí hiệu "dot" để truy cập vào mảng:

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### Nhận giá trị input từ JSON

Khi gửi request kiểu JSON lên, bạn có thể truy xuất dữ liệu trong JSON thông qua hàm `input` với điều kiện là header của request `Content-Type` phải được set là `application/json`. Bạn có thể sử dụng cú pháp "dot" để truy xuất sâu hơn vào trong mảng JSON:

    $name = $request->input('user.name');

#### Kiểm tra một giá trị input có tồn tại

Để kiểm tra một giá trị có tồn tại trên request hay không, bạn có thể sử dụng hàm `has`. Hàm `has` trả về `true` nếu như giá trị tồn tại **và** không phải là chuỗi rỗng:

    if ($request->has('name')) {
        //
    }

#### Nhận tất cả dữ liệu input

Bạn cũng có thể nhận tất cả giá trị input dưới dạng một `array` sử dụng hàm `all`:

    $input = $request->all();

#### Nhận một phần của dữ liệu input

Nếu bạn muốn lấy một tập nhỏ của input, bạn có thể sử dụng hai hàm `only` hay `except`. Cả hai hàm đều nhận một `array` hoặc một danh sách các đối số:

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

#### Thuộc tính động

Bạn cũng có thể truy xuất vào input sử dụng thuộc tính động trên đối tượng `Illuminate\Http\Request`. Ví dụ, nếu một trong các form có chứa trường là `name`, bạn có thể lấy giá trị được post lên như thế này:

    $name = $request->name;

Khi sử dụng thuộc tính động, Laravel đầu tiên sẽ tìm giá trị của tham số trên dữ liệu của request và rồi trong route parameter.

<a name="old-input"></a>
### Input cũ

Laravel cho phép bạn giữ giá trị input từ một request sang request tiếp theo. Đặc điểm này đặc biệt hữu dụng khi bạn muốn thiết lập lại form sau khi phát hiện có lỗi. Tuy nhiên, nếu bạn sử dụng [validation services](/docs/{{version}}/validation) của Laravel, thì bạn không cần phải làm việc này vì các công cụ xử lý validation sẵn của Laravel đã tự động thực hiện rồi.

#### Flash input tới session

Hàm `flash` trong `Illuminate\Http\Request` sẽ flash input hiện tại vào trong [session](/docs/{{version}}/session) nên nó có thể sử dụng trong request tiếp theo của user tới ứng dụng:

    $request->flash();

Bạn cũng có thể sử dụng `flashOnly` và `flashExcept` để flash một tập nhỏ của dữ liệu request vào trong session:

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### Flash input vào trong session rồi chuyển trang

Vì bạn thường muốn flash input cùng với chuyển trang vào trang trước đó, bạn có thể dễ dàng tạo móc nối input vào trong một redirect sử dụng hàm `withInput`:

    return redirect('form')->withInput();

    return redirect('form')->withInput($request->except('password'));

#### Lấy dữ liệu cũ

Để lấy dữ liệu đã flash từ request trước đó, sử dụng hàm `old` của `Request`. Hàm `old` cung cấp một helper tiện ích cho việc lấy dữ liệu đã flash ra khỏi [session](/docs/{{version}}/session):

    $username = $request->old('username');

Laravel cũng cung cấp một helper toàn cục `old`. Nếu như bạn muốn hiển thị giá trị input cũ bên trong [Blade template](/docs/{{version}}/blade), thì sử dụng helper `old` sẽ tiện hơn. Nếu không có input cũ nào tìm thấy thì giá trị `null` sẽ được trả về:

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Cookies

#### Lấy cookies từ Request

Tất cả các cookies tạo bởi Laravel đều được mã hoá và kí với một mã xác thực, nghĩa là chúng sẽ bị coi là không hợp lệ nếu chúng bị thay đổi phía client. Để lấy cookie từ request, bạn có thể sử dụng hàm `cookie` từ `Illuminate\Http\Request`:

    $value = $request->cookie('name');

#### Gắn một cookie mới vào Response

Laravel cung cấp hàm `cookie` helper phục vụ như một factory đơn giản để tạo ra một đối tượng từ class `Symfony\Component\HttpFoundation\Cookie`. Những cookies này có thể được gắn với một đối tượng `Illuminate\Http\Response` sử dụng hàm `withCookie`:

    $response = new Illuminate\Http\Response('Hello World');

    $response->withCookie('name', 'value', $minutes);

    return $response;

Để tạo một cookie tồn tại lâu (long-lived), tồn tại trong vòng 5 năm, bạn có thể sử dụng hàm `forever` trong factory bằng cách gọi hàm `cookie` helper không có tham số nào, và rồi gọi móc nối với `forever` để tạo:

    $response->withCookie(cookie()->forever('name', 'value'));

<a name="files"></a>
### Files

#### Lấy file được upload

Bạn có thể truy xuất vào file được upload lên sử dụng hàm `file`. Hàm `file` này sẽ trả về một đối tượng từ class `Symfony\Component\HttpFoundation\File\UploadedFile`, mà được kế thừa từ `SplFileInfo` và cung cấp một số phương thức để giao tiếp với file:

    $file = $request->file('photo');

Bạn có thể kiểm tra nếu một file tồn tại trên request sử dụng hàm `hasFile`:

    if ($request->hasFile('photo')) {
        //
    }

#### Kiểm tra upload thành công

Ngoài việc kiểm tra nếu file upload tồn tại, bạn có thể kiểm tra xem có vấn đề gì trong quá trình file upload lên không thông qua việc sử dụng hàm `isValid`:

    if ($request->file('photo')->isValid()) {
        //
    }

#### Chuyển vị trí file upload

Để chuyển vị trí file đã được upload tới một vị trí mới, bạn nên sử dụng hàm `move`. Hàm này sẽ chuyển một file từ vị trí tạm thời (được thiết lập trong cấu hình PHP của máy bạn) tới một vị trí mà bạn muốn:

    $request->file('photo')->move($destinationPath);

    $request->file('photo')->move($destinationPath, $fileName);

#### Các phương thức khác với file

There are a variety of other methods available on `UploadedFile` instances. Check out the [API documentation for the class](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) for more information regarding these methods.
Có nhiều hàm khác hỗ trợ cho việc xử lý file trong `UploadedFile`. Hãy tham khảo [tài liệu](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) để biết thêm thông tin về các hàm đó.
