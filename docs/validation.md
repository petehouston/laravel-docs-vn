# Validation

- [Validation](#validation)
  - [Giới thiệu](#gii-thiu)
  - [Khởi đầu nhanh](#khi-u-nhanh)
    - [Xác định routes](#xac-nh-routes)
    - [Tạo Controller](#to-controller)
    - [Viết logic validation](#vit-logic-validation)
    - [Hiển thị validation lỗi](#hin-th-validation-li)
    - [Ghi chú trên các trường tùy chọn](#ghi-chu-tren-cac-trng-toy-chn)
  - [Form Request Validation](#form-request-validation)
    - [Tạo Form Requests](#to-form-requests)
    - [Authorizing Form Requests](#authorizing-form-requests)
    - [Tùy chỉnh định dạng lỗi](#toy-chnh-nh-dng-li)
    - [Tùy biến nội dung lỗi](#toy-bin-ni-dung-li)
  - [Tự tạo validator](#t-to-validator)
    - [Redirection tự động](#redirection-t-ng)
    - [Named Error Bags](#named-error-bags)
    - [After Validation Hook](#after-validation-hook)
  - [Làm việc với nội dung lỗi](#lam-vic-vi-ni-dung-li)
    - [Tùy chỉnh nội dung lỗi](#toy-chnh-ni-dung-li)
  - [Những quy định validation có sẵn](#nhng-quy-nh-validation-co-sn)
  - [Thêm quy định có điều kiện](#them-quy-nh-co-iu-kin)
  - [Validating mảng](#validating-mng)
  - [Tùy biến quy định validation](#toy-bin-quy-nh-validation)
    - [Sử dụng đối tượng Rule](#s-dng-i-tng-rule)
    - [Sử dụng Closure](#s-dng-closure)
    - [Sử dụng Extensions](#s-dng-extensions)

## Giới thiệu

Laravel cung cấp một vài cách tiếp cận để xác thực dữ liệu đến ứng dụng của bạn. Mặc định, class base controller của Laravel sử dụng ``ValidatesRequests`` trait cung cấp phương thức khá thuận tiện cho việc xác nhận yêu cầu HTTP đến hợp lệ với một loạt các quy tắc xác thực mạnh mẽ.

## Khởi đầu nhanh

Để học tính năng xác thực của Laravel, Hãy xem một ví dụ hoàn chỉnh xác thực một form và hiển thị nội dung lỗi trả về cho người dùng.

### Xác định routes

Đầu tiên, giả sử chúng ta có route được định nghĩa trong routes/web.php:

```php
Route::get('post/create', 'PostController@create');

Route::post('post', 'PostController@store');
```

Tất nhiên, phương thức ``GET`` route sẽ hiển thị một form cho người dùng tạo mới một bài viết, trong khi phương thức ``POST`` route sẽ lưu bài viết đấy vào cơ sở dữ liệu.

### Tạo Controller

Tiếp theo, tạo một controller đơn giản xử lý các routes.Bây giờ, chúng ta sẽ để phương thức đấy  store rỗng:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     *
     * @return  Response
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * Store a new blog post.
     *
     * @param    Request  $request
     * @return  Response
     */
    public function store(Request $request)
    {
        // Validate and store the blog post...
    }
}
```

### Viết logic validation

Bây giờ chúng ta đã sẵn sàng viết logic vào phương thức ``store`` để validate tạo mới bài viết. Nếu bạn kiểm tra class base controller (``App\Http\Controllers\Controller``) của Laravel, bạn sẽ thấy class sử dụng một ``ValidatesRequests`` trait. nó cung cấp một phương thức validate cho tất cả controllers.

Phương thức ``validate`` chấp nhận một HTTP request đến và đặt quy định validation. Nếu quy định validationthành công, code của bạn sẽ thực thi bình thường; tuy nhiên, nếu validation thất bại, mội exception sẽ được ném và tích hợp lỗi response sẽ được tự động gửi cho người dùng. Trong trường hợp là HTTP request, một response chuyển trang sẽ được tạo ra, trong khi một JSON response sẽ được gửi cho AJAX requests.

Để có thể hiểu rõ hơn về phương thức ``validate`` , hãy quay lại phương thức ``store``:

```php
/**
 * Store a new blog post.
 *
 * @param    Request  $request
 * @return  Response
 */
public function store(Request $request)
{
    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // The blog post is valid, store in database...
}
```

Như bạn có thể thấy, chúng ta có thể truyền qua HTTP request đến và yêu cầu quy định validation vào phương thức ``validate``. Một lần nữa, nếu validation thất bại, Một proper response sẽ tự động được tạo ra. Nếu validation thành công, controller sẽ được thực thi bình thường.

#### Dừng khi sai ở xác thực đầu tiềni

Thình thoảng bạn muốn dừng việc kiếm tra thuộc tính sau khi validation đầu tiên đã sai. Để làm việc đó, gán quy định ``bail`` cho thuộc tính:

```php
$this->validate($request, [
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

Trong ví dụ này, nếu quy định ``required`` trên thuộc tính title thất bại, quy định ``unique`` sẽ không cần kiểm tra. Quy tắc sẽ được xác thực theo thứ tự được chỉ định.

#### Chú ý thuộc tính lồng nhau

Nếu HTTP request chứa tham số "lồng nhau", bạn có thể chỉ định chúng trong quy định validate bằng cách sử dụng cú pháp "dấu chấm":

```php
$this->validate($request, [
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

### Hiển thị validation lỗi

Cái gì sẽ xảy ra khi có một tham số request gửi đến không thành không với quy định ? Như đã đề cập ở trước, Laravel sẽ tự động chuyển trang lại cho người dùng về trang trước đó. Ngoài ra, tất cả các lỗi validation sẽ tự động [flashed vào session](session.md#flash-data).
Một lần nữa, chú ý rằng chúng ta sẽ không có một cách rõ ràng bind nội dung lỗi vào view của GET route. Bời vì Laravel sẽ tự động kiểm tra lỗi trong dữ liệu session, và tự động bind chúng vào view nếu chúng tồn tại. Biến ``$errors`` sẽ là một thể hiện của ``Illuminate\Support\MessageBag``. Để biết thêm chi tiết về nó, có thể xem dưới phần ``Working With Error Messages``.

>Biến ``$errors`` là bound vào view bởi middleware  ``Illuminate\View\Middleware\ShareErrorsFromSession``, nó được cung cấp bởi nhóm middleware ``web`` middleware. Khi middleware này được áp dụng, một biến $errors sẽ luôn luôn tồn tại tronh view của bạn, cho phép bạn thuận tiện để giả định biến  $errors luôn luôn được định nghĩa và có thể sử dụng.

Vì vậy, trong ví dụ trên, người dùng sẽ chuyển trang đến phương thức ``create`` của controller khi validation thất bại, cho phép chúng ta hiển thị nội dung lỗi trên view:

```php
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if (count($errors) > 0)
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

### Ghi chú trên các trường tùy chọn

Theo mặc định, Laravel bao gồm ``TrimStrings`` và ``ConvertEmptyStringsToNull`` middleware. Chúng được liệt kê trong lớp ``App\Http\Kernel``. Bởi vậy, bạn sẽ cần chỉnh lại phần tùy chọn cho yêu cầu nếu như bạn không muốn giá trị ``null`` là không hợp lệ:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

Trong ví dụ này, chúng tôi xác định rằng ``publish_at`` có thể là null hoặc ngày tháng hợp lệ. Nếu ``nullable`` không được thêm vào định nghĩa quy tắc, trình xác thực sẽ xem xét ``null`` là giá trị không hợp lệ.

#### AJAX Requests & Validation

Trong ví dụ này, chúng ta sử dụng form để gửi dữ liệu vào ứng dụng. Tuy nhiên, nhiều ứng dụng sử dụng AJAX requests. Khi sử dụng phương thức validate trong AJAX request, Laravel sẽ không tạo ra redirect response. Thay vì, Laravel tạo một JSON response chứa tất cả lỗi validation. JSON response này sẽ được gửi với mã 422 HTTP status.

## Form Request Validation

### Tạo Form Requests

Với những trường hợp validation phức tạp, bạn có thể tạo một "form request". Form requests là tùy chỉnh class request chứa logic validation. Để tạo class form request, sử dụng lệnh ``make:request`` Artisan CLI:

```php
php artisan make:request StoreBlogPost
```

class được tạo sẽ nằm ở thư mục ``app/Http/Requests``. Nếu thư mục đó không tồn tại, nó sẽ được tạo khi bạn chạy lệnh ``make:request``. Chúng ta sẽ thêm một vài quy định validation vào trong phương thưc ``rules``:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return  array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

Bạn đánh giá thế nào về quy định validation? tất cả bạn cần làm là type-hint request vào trong phương thức controller. Form request được validated trước khi phương thức controller được gọi, nghĩa là bạn không cần viết một mớ hỗn độn logic trong controller:

```php
/**
 * Store the incoming blog post.
 *
 * @param  StoreBlogPost  $request
 * @return Response
 */
public function store(StoreBlogPost $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();
}
```

Nếu validation thất bại, một chuyển trang response sẽ được tạo ra để gửi cho lại người dùng đến trang trước. Ngoài ra lỗi sẽ được flashed vào session, vì vậy chúng ta có thể hiển thị nó. Nếu request là AJAX request, một HTTP response với mã 422 status sẽ được trả về cho người dùng gồm JSON representation chứa lỗi validation.

#### Adding After Hooks To Form Requests

Nếu bạn muốn thêm móc "sau" vào yêu cầu biểu mẫu, bạn có thể sử dụng phương thức ``withValidator``. Phương thức này nhận được trình xác nhận hợp lệ được xây dựng đầy đủ, cho phép bạn gọi bất kỳ phương thức nào của nó trước khi các quy tắc xác thực thực sự được đánh giá:

```php
/**
 * Configure the validator instance.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```

### Authorizing Form Requests

Class form request ngoài ra còn chứa một phương thức ``authorize``. Bên trong phương thức, bạn có thể xác thực người dùng thực sự đã có quyền cập nhật dữ liệu. Ví dụ, nếu một người dùng cố gắng cập nhật comment của một một bài viết, họ thật sự sở hữa comment đấy? Ví dụ:

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return  bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

Khi tất cả các form requests kế thừa từ class base Laravel request, chúng ta có thể sử dụng phương thức ``user`` để truy cập xác thức người dùng. Ngoài ra cũng cần gọi phương thức ``route`` trong ví dụ trên. Phương thức này cho phép bạn truy cập đến tham số của URI được định nghĩa trong route được gọi, như tham số ``{comment}`` trong ví dụ trên:

```php
Route::post('comment/{comment}');
```

Nếu phương thức authorize trả về false, một HTTP response với mã 403 status sẽ được tự động trả về và phương thức controller sẽ không được thực hiện.

Nếu bạn có kế hoạch cho phép logic trong một phần khác ứng dung của bạn, đơn giản trả về true từ phương thức authorize:

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return  bool
 */
public function authorize()
{
    return true;
}
```

### Tùy chỉnh định dạng lỗi

Nếu bạn muốn tùy biến định dạng lỗi validation được flashed vào session khi validation thất bại, ghi đè phương thức formatErrors trong base request (``App\Http\Requests\Request``). Đừng quên import class ``Illuminate\Contracts\Validation\Validator`` class ở trên đầu file:

```php
/**
 * {@inheritdoc}
 */
protected function formatErrors(Validator $validator)
{
    return $validator->errors()->all();
}
```

### Tùy biến nội dung lỗi

Bạn có thể muốn tùy biến nội dung lỗi bằng cách sử dụng bởi form request bằng cách ghi đè phương thức ```messages```. Phương thức này trả về một mảng các cặp thuộc tính / quy định tương ứng với nội dung lỗi:

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return  array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
    ];
}
```

## Tự tạo validator

Nếu bạn không muốn sử dụng phương thức ``validate``, bạn có thể tự tạo một thể hiện validator bằng Validator [[facade]](facade.md). Phương thức ``make`` trong facade sinh ra một thể hiện mới validator:

```php
<?php

namespace App\Http\Controllers;

use Validator;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * Store a new blog post.
     *
     * @param    Request  $request
     * @return  Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // Store the blog post...
    }
}
```

Đối số đầu tiên truyền vào phương thức ``make`` là dữ liệu cần validation. Đối số thứ hai là mảng quy định validation được áp dụng vào dữ liệu.

Sau khi kiểm tra request validation không thành công, bạn có thể dùng phương thức ``withErrors`` để flash nội dung lỗi vào session. Khi sử dụng phương thức này, Biến ``$errors`` sẽ tự động được qhia sẻ đến views sau khi chuyển trang, cho phép bạn dễ dàng hiển thị chúng cho người dùng. Phương thức ``withErrors`` chấp nhận một validator, ``MessageBag``, hoặc một ``array`` PHP.

### Redirection tự động

Nếu bạn muốn tạo một cá thể trình duyệt hợp lệ theo cách thủ công nhưng vẫn tận dụng lợi thế của chuyển hướng tự động được cung cấp theo validatephương thức của yêu cầu, bạn có thể gọi phương thức ``validate`` trong một thể hiện hiện tại validator. Nếu validation thất bạn, người dùng sẽ tự động được chuyển trang hoặc trong trường hợp là một AJAX request, một JSON response sẽ được trả về:

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

### Named Error Bags

Nếu bạn có nhiều form trên một trang, bạn có thể sử dụng phương thức MessageBag, cho phép bạn nhận nội dung lỗi từ form cụ thể. Đơn giản chỉ là truyền thêm một tham số thứ hai của phương thức  withErrors:

```php
return redirect('register')
            ->withErrors($validator, 'login');
```

Bạn cũng có thể lấy một thể hiện MessageBag từ biến $errors:

```php
{{ $errors->login->first('email') }}
```

### After Validation Hook

Ngoài ra validator còn cho phép bạn thêm callbacks để chạy sau khi validation thành công. Điều này cho phép bạn dễ dàng thực hiện các validation và thêm nội dung lỗi cho message collection. Để bắt đầu, sử dụng phương thức ``after`` trong một thể hiện validator:

```php
$validator = Validator::make(...);

$validator->after(function($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```

## Làm việc với nội dung lỗi

Sau khi gọi phương thức errors trong một thể hiện Validator, bạn sẽ nhận được một thể hiện ``Illuminate\Support\MessageBag``, sẽ có một số phương thức làm việc với nội dung lỗi. Biến ``$errors`` sẽ tự động được tạo cho tất cả các view, ngoài ra nó cũng là một thể hiện của class ``MessageBag``.

#### Nhận về nội dung lỗi đầu tiên của một trường

Để nhận về lỗi đầu tiên của một trường, sử dụng phương thức first:

```php
$errors = $validator->errors();

echo $errors->first('email');
```

#### Nhận về tất cả nội dung lỗi của một trường

Nếu bạn cần nhận một mảng nội dung của tất cả lỗi của một trường, sử dụng phương thức get:

```php
foreach ($errors->get('email') as $message) {
    //
}
```

Nếu bạn validating một mảng các trường của form, bạn có thể nhận về tất cả nội dung cho mỗi phần tử của mảng bằng cách sử dụng ký tự *:

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

#### Nhận về tất cả các lỗi của tất cả các trường

Để nhận một mảng tất cả các nội dung của tất cả các trường, sử dụng phương thức all:

```php
foreach ($errors->all() as $message) {
    //
}
```

#### Xác định nội dung của một trường có tồn tại

Phương thức has có thể sử dụng để xác định bất kỳ nội dung lỗi tồn tại của một trường:

```php
if ($errors->has('email')) {
    //
}
```

### Tùy chỉnh nội dung lỗi

Nếu bạn cần, bạn có thể tùy biến nội dung lỗi cho thể hiện validation mặc định. Có một vài cách để làm việc này. Đầu tiên, bạn có thể truyền tùy biến nội dung như là tham số thứ ba của hàm  ``Validator::make``:

```php
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```

Trong ví dụ trên, thuộc tính ``:attribute`` place-holder sẽ thay thế bởi tên thực tế của trường validation. Ngoài ra bạn có thể sử dụng place-holders trong nội dung validation. Ví dụ:

```php
$messages = [
    'same'    => 'The :attribute and :other must match.',
    'size'    => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute must be between :min - :max.',
    'in'      => 'The :attribute must be one of the following types: :values',
];
```

#### Tùy biến nội dung của thuộc tính cụ thể

Thỉnh thoảng bạn có thể tùy biến nội dung lỗi chỉ duy nhất một trường. Bạn có thể sử dụng "dấu chấm". Chỉ định tên của thuộc tính đầu tiên, theo bởi quy định:

```php
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```

#### Tùy biến nội dung trong file ngôn ngữ

Trong hầu hết các trương hợp, bạn có thể tùy biến nội dung trong một file ngôn ngữ thay vì truyền chúng trực tiếp vào phương thức Validator. Đề làm điều này, bạn thêm nội dung vào mảng  custom trong file ngôn ngữ resources/lang/xx/validation.php.

```php
'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```

#### Tùy biến thuộc tính trong file ngôn ngữ

Nếu bạn muốn thuộc tính :attribute phần nội dung validation sẽ được thay đổi bởi tên thuộc tính tùy chỉnh, bạn có thể tùy biến trong mảng attributes của file ngôn ngữ  resources/lang/xx/validation.php:

```php
'attributes' => [
    'email' => 'email address',
],
```

## Những quy định validation có sẵn

Bên dưới là danh sách những quy định có sẵn và hàm của nó:

- Accepted
- Active URL
- After (Date)
- Alpha
- Alpha Dash
- Alpha Numeric
- Array
- Before (Date)
- Between
- Boolean
- Confirmed
- Date
- Date Format
- Different
- Digits
- Digits Between
- Dimensions (Image Files)
- Distinct
- E-Mail
- Exists (Database)
- File
- Filled
- Image (File)
- In
- In Array
- Integer
- IP Address
- JSON
- Max
- MIME Types
- MIME Type By File Extension
- Min
- Nullable
- Not In
- Numeric
- Present
- Regular Expression
- Required
- Required If
- Required Unless
- Required With
- Required With All
- Required Without
- Required Without All
- Same
- Size
- String
- Timezone
- Unique (Database)
- URL

#### accepted

Giá trị phải là yes, on, 1, or true. Rất hữu dụng cho việc chấp nhận "Terms of Service".

#### active_url

Giá trị phải là URL theo hàm checkdnsrr của PHP.

#### after:date

Giá trị phải là một ngày sau ngày đã cho. Giá trị ngày phải hợp lệ theo hàm strtotime của PHP:

```php
'start_date' => 'required|date|after:tomorrow'
```

Thay vì truyền giá trị ngày vào một chuỗi vào hàm strtotime, bạn có thể chỉ định một trường khác để so sánh ngày:

```php
'finish_date' => 'required|date|after:start_date'
```

#### alpha

Giá trị phải là chữ cái.

#### alpha_dash

Giá trị phải là chữ cái hoặc chữ số, gồm cả dấu gạch ngang và dấu gạch dưới.

#### alpha_num

Giá trị phải là chữ số.

#### array

Giá trị phải là một array PHP.

#### before:date

Giá trị phải là ngày trước ngày đã cho. Giá trị ngày sẽ được truyền vào hàm strtotime của PHP.

#### between:min,max

Giá trị phải nằm trong khoảng min and max. Chuỗi, số, và file là giống kiểu size với nhau.

#### boolean

Giá trị phải là kiểu boolean có thể là true, false, 1, 0, "1", và "0".

#### confirmed

Giá trị phải khớp với trường foo_confirmation. Ví dụ, nếu trường là password, thì giá trị  password_confirmation phải khớp với trương mật khẩu.

#### date

Giá trị phải là ngày hợp lệ theo hàm strtotime của PHP.

#### date_format:format

Giá trị phải giống format truyền vào. Định dạng phải hợp lệ với hàm date_parse_from_format của PHP. Bạn nên sử dụng either date hoặc date_format khi validate một trường.

#### different:field

Giá trị phải khác giá trị của field.

#### digits:value

Giá trị phải là numeric và phải chính xác độ dài là value.

#### digits_between:min,max

Giá trị phải có độ dài nằm trong khoảng min and max.

#### dimensions

Giá trị phải là một ảnh có kích thước giống rule's parameters:

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

Tồn tại một số thuộc tính: min_width, max_width, min_height, max_height, width, height, ratio.

A ratio biểu diễn tỷ lệ chiều rộng chia chiều cao.Có thể được xác định như 3/2 hoặc 1.5:

```php
'avatar' => 'dimensions:ratio=3/2'
```

#### distinct

Khi làm việc với mảng, Mảng phải không có giá trị lặp lại.

```php
'foo.*.id' => 'distinct'
```

#### email

Giá trị phải là một địa chỉ email.

#### exists:table,column

Giá trị phải có trong bảng cơ sở dữ liệu.

##### Basic Usage Of Exists Rule

```php
'state' => 'exists:states'
```

##### Specifying A Custom Column Name

```php
'state' => 'exists:states,abbreviation'
```

Thỉnh thoảng, bạn cần kiểm tra kết nối database sử dụng cho exists query. Bạn có thể làm điều này bằng cách thêm "dấu chấm" vào trước tên kết nối:

```php
'email' => 'exists:connection.staff,email'
```

Nếu bạn muốn tùy biến thực thi query , bạn có thể sử dụng class Ruleđể định nghĩa quy định. Trong ví dụ này, chúng ta chỉ định quy tắc validation như là một mảng thay vì sử dụng ký tự |:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function ($query) {
            $query->where('account_id', 1);
        }),
    ],
]);
```

#### file

Giá trị phải là một file tải lên thành công.

#### filled

Giá trị không được phép trống.

#### image

Giá trị phải là ảnh có định dạng (jpeg, png, bmp, gif, or svg)

#### in:foo,bar,...

Giá trị phải thuộc danh sách các giá trị.

#### in_array:anotherfield

Giá trị phải tồn tại trong giá trị của anotherfield's.

#### integer

Giá trị phải là kiểu integer.

#### ip

Giá trị phải là địa chỉ IP.

#### json

Giá trị phải là một string JSON.

#### max:value

Giá trị phải nhỏ hơn hoặc bằng value. Chuỗi, số, và file là kiểu giống  size với nhau.

#### mimetypes:text/plain,...

Giá trị phải khớp với MIME types:

```php
'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
```

Xác định MIME type của file upload, nội dung file sẽ được đọc framework sẽ đoán MIME type, có thể sẽ khác MIME type của người dùng.

#### mimes:foo,bar,...

Giá trị phải khơp với MIME type ứng với một danh sách extensions.

##### Basic Usage Of MIME Rule

'photo' => 'mimes:jpeg,bmp,png'
Mặc dù bạn chỉ cần xác định extensions, thực ra quy định validates này lại là validate MIME type file bằng các đọc nội dung và đoán MIME type.

Tất cả danh sách MIME types và extensions có thể tìm thấy ở: http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types

#### min:value

Giá trị phải nhỏ hơn value. Chuỗi, số, và file là giống size với nhau.

#### nullable

Giá trị có thể  null. Nó rất hữu dụng khi validate string hoặc integer chứa giá trị null.

#### not_in:foo,bar,...

Giá trị phải không thuộc danh sách giá trị.

#### numeric

Giá trị phải là số.

#### present

Giá trị hiện tại phải xuất hiện trong input nhưng thể được trống.

#### regex:pattern

Giá trị phải khớp với regular expression.

Note: Khi sử dụng regex pattern, nó cần được xác định quy định trong mảng thay vì sử dụng pipe delimiter, đặc biệt nếu regular expression chứa pipe ký tự.

#### required

Giá trị phải xuất hiện trong input và không được phép trống. Một trường được coi là "empty" nếu một trong số điều kiện dưới đây đúng:

Giá trị là null.
Giá trị là một chuỗi rỗng.
Giá trị là mảng rỗng hoặc object Countable rỗng.
Giá trị là file upload không có đường dẫn.

#### required_if:anotherfield,value,...

Giá trị phải xuất hiện và không được trống nếu trường anotherfield bằng bất kỳ value.

#### required_unless:anotherfield,value,...

Giá trị phải xuất hiện và không được phép trống trừ khi trường anotherfield bằng bất kỳ value.

#### required_with:foo,bar,...

Giá trị phải xuất hiện và không được trống only if bất kỳ một trường khác xác định xuất hiện.

#### required_with_all:foo,bar,...

Giá trị phải xuất hiện và không được trống only if tất cả các trường khác xác định xuất hiện.

#### required_without:foo,bar,...

Giá trị phải xuất hiện và không được trống only when bất cứ trường xác định không xuất hiện.

#### required_without_all:foo,bar,...

The field under validation must be present and not empty only when all of the other specified fields are not present.

#### same:field

Giá trị field phải khớp với trường này.

#### size:value

Giá trị phải có kích thước khớp với value. Đối với chuỗi, value tương ứng là số ký tự. Đối với só, value tương ứng là giá trị integer. Đối với mảng, size tương ứng là count phần tử của mảng. Đối với file, size tương ứng là kích thước file kiểu kilobytes.

#### string

Giá trị phải là chuỗi. Nếu bạn muốn cho phép trường đó null, bạn có thể gán nullable vào trường đó.

#### timezone

Giá trị phải là timezone identifier hợp lệ với hàm timezone_identifiers_list của PHP.

#### unique:table,column,except,idColumn

Giá trị phải là unique trong bảng cơ sở dữ liệu. Nếu tên  column không được chỉ định, trường name sẽ được sử dụng.

##### Specifying A Custom Column Name

```php
'email' => 'unique:users,email_address'
```

##### Tùy biến kết nối cơ sở dữ liệu

Thỉnh thoảng, có thể bạn muốn tủy chỉnh kết nối query cơ sở dữ liệu bởi Validator. Như ở trên, cài đặt unique:users như một quy định validation sẽ sử dụng kết nối mặc định database để query đến cơ sở dữ liệu. Để ghi đè nó, xác định kết nối và tên bảng sử dụng "dấu chấm":

```php
'email' => 'unique:connection.users,email_address'
```

##### Validate Uniquebỏ qua ID

Thỉnh thoảng, bạn có thể muốn bỏ qua id trong khi kiểm tra unique. Ví dụ, cân nhắc "cập nhận hồ sơ" sẽ bao gồm name, địa chỉ e-mail, và địa điểm của người dùng.Tất nhiên, bạn sẽ muốn xác định email là unique. Tuy nhiên, nếu người dùng chỉ thay đổi tên và không thay đổi email, bạn không muốn validation lỗi được ném ra bởi vì người dùng đó đã sử dụng cái email đấy rồi.

Chỉ dẫn validator bỏ qua ID của người dùng, chúng ta sử dụng class Rule định nghĩa quy tắc. Trong ví dụ này, chúng ta sẽ chỉ định quy tắc validation như một mảng thay thế sử dụng ký tự để phân cách | quy định:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

Nếu bản user của bạn có a primary key không phải là id, bạn có thể chỉ định name của cột khi gọi phương thức ignore:

```php
'email' => Rule::unique('users')->ignore($user->id, 'user_id')
```

##### Thêm điều kiện bổ sung

Bạn cũng có thể thêm query chứa tùy chỉnh query bằng cách sử dụng phương thức where. Ví dụ, chúng ta thêm một hạn chế để kiểm tra account_id là 1:

```php
'email' => Rule::unique('users')->where(function ($query) {
    $query->where('account_id', 1);
})
```

#### url

Giá trị phải là đúng định dạng URL.

## Thêm quy định có điều kiện

#### Validating khi xuất hiện

Trong một số trường hợp, bạn có thể muốn chạy validation kiểm tra lại trường only nếu trường đó xuất hiện trong mảng input. Để nhanh chóng làm điều này, thêm sometimes vào trong danh sách quy tắc rule:

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

Trong ví dụ trên, trường email sẽ chỉ được validated nếu nó xuất hiện trong mảng $data.

#### Thêm quy định phức tạp

Thỉnh thoảng bạn muốn thêm quy định trong logic. Ví dụ, bạn có thể muốn yêu cầu một trường chỉ nếu trường khác có giá trị lớn hơn 100. Hoặc, Bạn muốn 2 trường có giá trị chỉ khi trường khác xuất hiện. Để làm việc đó không có gì khó khăn cả. Đầu tiên, tạo một thể hiện Validator với static rules sẽ không bao giờ thay đổi:

```php
$v = Validator::make($data, [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

Giả sử bây giờ ứng dụng web của bạn là sưu tầm game.Nếu một người sưu tầm game đăng ký ứng dụng của bạn và họ có nhỏ hơn 100 games, chúng ta muốn họ giải thích tại sao chọ có quá nhiều game. Ví dụ, có thể họ chạy một shop bán game, hoặc có thể họ thích sư tầm. Để có thể yêu cầu này, chúng ta có thể sử dụng phương thức sometimes trong thể hiện Validator.

```php
$v->sometimes('reason', 'required|max:500', function($input) {
    return $input->games >= 100;
});
```

Tham số thứ nhất truyền vào phương thức sometimes là tên của trường chúng ta muốn validate.Tham số thứ hai là quy định chúng ta muốn thêm. Nếu truyền Closure như là tham số thứ ba trả về  true, quy định sẽ được thêm. Phương thức này làm cho việc thêm quy định validate phức tạp trở lên dễ dàng hơn, ngay cả khi bạn muốn thêm nhiều validate cho nhiều trường:

```php
$v->sometimes(['reason', 'cost'], 'required', function($input) {
    return $input->games >= 100;
});
```

>Tham số ``$input`` truyền vào trong Closure là một thể hiện của  ``Illuminate\Support\Fluent`` và bạn có thể truy cập input và file.

## Validating mảng

Validating mảng các trường của form không có gì khó khăn. Ví dụ, để validate mỗi email trong mảng trường input là unique, bạn có thể làm như sau:

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

Tương tự như vậy, bạn có thể sử dụng ký tự * khi muốn chỉ định nội dung validation trong file ngôn ngữ, làm cho việc dễ dàng sử dụng một file nội dung validate cho mảng:

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique e-mail address',
    ]
],
```

## Tùy biến quy định validation

### Sử dụng đối tượng Rule

Laravel cung cấp một số quy định validation rất hữu ích; tuy nhiên, có thể bạn muốn tạo validate bởi chính bạn.
Để tạo một quy định mới, bạn có thể sử dụng câu lệnh artisan ``make:rule``, Laravel sẽ đặt quy định mới ở trong thư mục ``app\Rules``:

```php
php artisan make:rule Uppercase
```

Khi quy tắc đã được tạo, chúng tôi sẵn sàng xác định hành vi của quy tắc. Một đối tượng quy tắc có chứa hai phương thức: ``passes`` và ``message``. Phương thức ``passes`` nhận giá trị thuộc tính và tên, và phải trả lại truehoặc falsetuỳ thuộc vào việc các giá trị thuộc tính là hợp lệ hay không. Phương thức ``message`` nên trả lại thông báo lỗi xác nhận rằng nên được sử dụng khi xác nhận thất bại:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    /**
     * Determine if the validation rule passes.
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be uppercase.';
    }
}
```

Tất nhiên, bạn có thể gọi ``trans`` từ phương thức ``message`` của bạn nếu bạn muốn trả lại một thông báo lỗi từ các tệp dịch của bạn:

```php
/**
 * Get the validation error message.
 *
 * @return string
 */
public function message()
{
    return trans('validation.uppercase');
}
```

Khi quy tắc đã được xác định, bạn có thể đính kèm quy tắc đó vào trình xác thực bằng cách chuyển một phiên bản của đối tượng quy tắc với các quy tắc xác thực khác của bạn:

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

### Sử dụng Closure

Nếu bạn chỉ cần các chức năng của một quy tắc tùy chỉnh một lần trong suốt ứng dụng của bạn, bạn có thể sử dụng một Closure thay vì một đối tượng quy tắc.
Tùy biến validator Closure nhận bốn đối số: tên của $attribute được validate, giá trị $value của thuộc tính, và lời gọi ``$faill`` sẽ được gọi khi nếu xác thực sai.

```php
$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function($attribute, $value, $fail) {
            if ($value === 'foo') {
                return $fail($attribute.' is invalid.');
            }
        },
    ],
]);
```

### Sử dụng Extensions

Một phương pháp khác để đăng ký quy tắc xác thực tùy chỉnh là sử dụng extendphương pháp trên Validator faceade. Sử dụng phương thức này trong [service provider](providers.md) để đăng kí một quy định tùy chỉnh

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Validator;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
            return $value == 'foo';
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
```

Tùy biến validator Closure nhận bốn đối số: tên của ``$attribute`` được validate, giá trị ``$value`` của thuộc tính, một mảng quy định ``$parameters``, và một thể hiện Validator.

Bạn cũng có thể truyền một class và method vào phương thức extend thay vì một Closure:

```php
Validator::extend('foo', 'FooValidator@validate');
```

#### Định nghĩa nội dung lỗi

Bạn có thể định nghĩa một nội dung lỗi cho quy định tùy biến của bạn. Bạn có thể làm như vậy hoặc một mảng nội dung tùy biến nội dung inline hoặc thêm vào validation file ngôn ngữ. Nội dung này sẽ được đặt ở trên đầu của mảng, không ở bên trong mảng custom,nó chỉ dành cho những nội dung lỗi attribute-specific:

```php
"foo" => "Your input was invalid!",

"accepted" => "The :attribute must be accepted.",

// The rest of the validation error messages...
```

Khi bạn tùy biến quy định validation, thỉnh thảng bạn cần định nghĩa tùy chỉnh place-holder thay thế nội dung lỗi. Bạn cũng có thể tạo một Validator như miểu tả ở trên sau đó gọi phương thức  replacertrong Validator facade. Bạn có thể sử dụng trong phương thức boot của service provider:

```php
/**
 * Bootstrap any application services.
 *
 * @return  void
 */
public function boot()
{
    Validator::extend(...);

    Validator::replacer('foo', function($message, $attribute, $rule, $parameters) {
        return str_replace(...);
    });
}
```

#### Implicit Extensions

Mặc định, khi một thuộc tính đã được validated là không xuất hiện hoặc chứa một giá trị rỗng như định nghĩa bởi quy tắc required, quy tắc validation thường, bao gồm cả phần extensions, là không hoạt động. Ví dụ, quy định unique sẽ không hoạt động lần nữa nếu giá trị null:

```php
$rules = ['name' => 'unique'];

$input = ['name' => null];

Validator::make($input, $rules)->passes(); // true
```

Đối với quy tắc validate hoạt động ngay cả khi thuộc tính là rỗng, quy định phải ngụ ý rằng các thuộc tính là bắt buộc. Như tạo một "implicit" extension, sử dụng phương thức  ``Validator::extendImplicit()``:

```php
Validator::extendImplicit('foo', function($attribute, $value, $parameters, $validator) {
    return $value == 'foo';
});
```

>Một "implicit" extension chỉ implies ngụ ý là các thuộc tính là bắt buộc. Cho dù nó thực sự invalidates thuộc tính là lỗi hoặc rộng là phụ thuộc vào bạn.