# Các hàm trợ giúp

- [Giới thiệu](#introduction)
- [Danh sách các hàm](#available-methods)

<a name="introduction"></a>
## Giới thiệu

Laravel có chứa danh sách các hàm PHP "trợ giúp". Trong số này, nhiều hàm được sử dụng bên trong framework; tuy nhiên, bạn có thể tuỳ ý sử dụng chúng trong ứng dụng nếu bạn cảm thấy tiện.

<a name="available-methods"></a>
## Danh sách các hàm

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### Arrays

<div class="collection-method-list" markdown="1">
[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_pull](#method-array-pull)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[head](#method-head)
[last](#method-last)
</div>

### Paths

<div class="collection-method-list" markdown="1">
[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[elixir](#method-elixir)
[public_path](#method-public-path)
[storage_path](#method-storage-path)
</div>

### Strings

<div class="collection-method-list" markdown="1">
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[studly_case](#method-studly-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)
</div>

### URLs

<div class="collection-method-list" markdown="1">
[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[url](#method-url)
</div>

### Các hàm khác

<div class="collection-method-list" markdown="1">
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[collect](#method-collect)
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[dispatch](#method-dispatch)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[method_field](#method-method-field)
[old](#method-old)
[redirect](#method-redirect)
[request](#method-request)
[response](#method-response)
[session](#method-session)
[value](#method-value)
[view](#method-view)
[with](#method-with)
</div>

<a name="method-listing"></a>
## Giới thiệu các hàm

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## Arrays

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

Hàm `array_add` thêm một cặp key / value vào trong mảng nếu key đó chưa tồn tại trong array:

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

Hàm `array_collapse` làm thu nhỏ lại mảng của các mảng thành một mảng đơn:

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

Hàm `array_divide` trả về hai mảng, một mảng chứa các key, mảng còn lại chứa các values của mảng gốc:

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

Hàm `array_dot` làm flat các mảng đa chiều thành mảng một chiều sử dụng kí hiệu "dot" để đánh đấu độ sâu:

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

Hàm `array_except` loại bỏ các cặp key / value khỏi mảng:

    $array = ['name' => 'Desk', 'price' => 100];

    $array = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

Hàm `array_first` trả về phần tử đầu tiên của mảng theo một điều kiện:

    $array = [100, 200, 300];

    $value = array_first($array, function ($key, $value) {
        return $value >= 150;
    });

    // 200

Giá trị mặc định cũng có thể được truyền vào ở tham số thứ ba. Giá trị này sẽ được trả lại nếu không có giá trị nào thoả mãn điều kiện:

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

Hàm `array_flatten` sẽ làm flat mảng đa chiều thành mảng một chiều.

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

Hàm `array_forget` xoá một cặp key / value từ một mảng con nằm sâu bên trong sử dụng kí hiệu "dot":

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

Hàm `array_get` lấy giá trị từ mảng con sâu bên trong sử dụng kí hiệu "dot":

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

Hàm `array_get` cũng nhận một giá trị mặc định, và trả lại nếu như một khoá không tìm thấy:

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

Hàm `array_has` kiểm tra xem một item có tồn tại trong mảng hay không sử dụng kí hiệu "dot":

    $array = ['products' => ['desk' => ['price' => 100]]];

    $hasDesk = array_has($array, 'products.desk');

    // true

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

Hàm `array_only` sẽ trả lại giá trị của các cặp key / value từ một mảng cho trước:

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

Hàm `array_pluck` sẽ trả lại danh sách giá trị với key cho trước:

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

Bạn cũng có thể chỉ định đâu là khoá trong danh sách kết quả:

    $array = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail'];

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

Hàm `array_prepend` sẽ thêm một item vào đầu mảng:

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // $array: ['zero', 'one', 'two', 'three', 'four']

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

Hàm `array_pull` trả lại và xoá một cặp key / value khỏi mảng:

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

Hàm `array_set` thiết lập giá trị sâu trong mảng con sử dụng kí hiệu "dot":

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

Hàm `array_sort` thực hiện sắp xếp mảng theo kết quả của một Closure truyền vào:

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Chair'],
    ];

    $array = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

Hàm `array_sort_recursive` thực hiện sắp xếp mảng sử dụng hàm `sort` một cách đệ quy:

    $array = [
        [
            'Roman',
            'Taylor',
            'Li',
        ],
        [
            'PHP',
            'Ruby',
            'JavaScript',
        ],
    ];

    $array = array_sort_recursive($array);

    /*
        [
            [
                'Li',
                'Roman',
                'Taylor',
            ],
            [
                'JavaScript',
                'PHP',
                'Ruby',
            ]
        ];
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

Hàm `array_where` lọc mảng theo một Closure cho trước:

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function ($key, $value) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-head"></a>
#### `head()` {#collection-method}

Hàm `head` đơn giản chỉ trả về phần tử đầu tiên của mảng:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

Hàm `last` trả về phần tử cuối cùng của mảng:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## Paths

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

Hàm `app_path` trả về đường dẫn đầy đủ tới thư mục `app`:

    $path = app_path();

Bạn cũng có thể sử dụng hàm `app_path` để tạo ra đường dẫn đầy đủ tới một file relative với thư mục của ứng dụng:

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

Hàm `base_path` trả về đường dẫn đầy đủ của project root:

    $path = base_path();

Bạn cũng có thể sử dụng hàm `base_path` để tạo đường dẫn đầy đủ tới một file relative tới thư mục ứng dụng:

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

Hàm `config_path` trả về đường dẫn đầy đủ tới thư mục cấu hình:

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

Hàm `database_path` trả về đường dẫn đầy đủ tới thư mục database của ứng dụng:

    $path = database_path();

<a name="method-elixir"></a>
#### `elixir()` {#collection-method}

Hàm `elixir` trả về đường dẫn của file [Elixir](/docs/{{version}}/elixir) được đánh version:

    elixir($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

Hàm `public_path` trả về đường dẫn đầy dủ tới thư mục `public`:

    $path = public_path();

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

Hàm `storage_path` trả về đường dẫn đầy đủ tới thư mục `storage`:

    $path = storage_path();

Bạn cũng có thể sử dụng hàm `storage_path` để sinh ra đường dẫn đầy đủ tới một file relative tới thư mục storage:

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Strings

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

Hàm `camel_case` convert một chuỗi thành kiểu `camelCase`:

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

Hàm `class_basename` trả về tên của class với namespace được gỡ đi:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

Hàm `e` thực hiện gọi hàm `htmlentities` trên một chuỗi:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

Hàm `ends_with` cho biết nếu một chuỗi kết thúc bằng một chuỗi con hay không:

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

Hàm `snake_case` convert một chuỗi thành kiểu `snake_case`:

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

Hàm `str_limit` giới hạn số lượng character trong một chuỗi. Hàm nhận vào một chuỗi ở tham số đầu và số lượng kí tự tối đa ở tham số thứ hai:

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

Hàm `starts_with` cho biết nếu một chuỗi bắt đầu bằng một chuỗi con cho trước hay không:

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

Hàm `str_contains` cho biết nếu một chuỗi có chứa một chuỗi con khác hay không:

    $value = str_contains('This is my name', 'my');

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

Hàm `str_finish` thêm một kí tự vào cuối một chuỗi:

    $string = str_finish('this/string', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

Hàm `str_is` cho biết nếu một chuỗi có khớp với một pattern nào không. Dấu `*` có thể đươc dùng để đánh dấu wildcards:

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

Hàm `str_plural` convert một chuỗi sang số nhiều. Hàm này hiện tại chỉ hỗ trợ tiếng Anh:

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

Bạn có thể cung cấp một số nguyên vào tham số thứ hai để lấy giá trị số ít hay số nhiều của chuỗi:

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

Hàm `str_random` sinh ra một chuỗi ngẫu nhiên với độ dài cho trước:

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

Hàm `str_singular` convert một chuỗi sang số ít. Hàm này hiện tại chỉ hỗ trợ tiếng Anh:

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

Hàm `str_slug` sinh ra một "slug" thân thiện cho URL:

    $title = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

Hàm `studly_case` convert một chuỗi cho trước thành kiểu `StudlyCase`:

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-trans"></a>
#### `trans()` {#collection-method}

Hàm `trans` dịch một dòng ngôn ngữ khai báo trong [localization files](/docs/{{version}}/localization):

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

Hàm `trans_choice` dịch một dòng ngôn ngữ với một chút biến đổi:

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

Hàm `action` sinh ra một URL cho một controller action. Bạn không cần truyền vào đầy đủ namespace tới controller. Thay vào đó, truyền tên class của controller relative với namespace `App\Http\Controllers`:

    $url = action('HomeController@getIndex');

Nếu hàm nhận route parameters, bạn có thể truyền chúng vào tham số thứ hai:

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

Sinh ra một URL cho asset sử dụng scheme hiện tại của request (HTTP hoặc HTTPS):

	$url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

Sinh ra một URL cho asset sử dụng HTTPS:

	echo secure_asset('foo/bar.zip', $title, $attributes = []);

<a name="method-route"></a>
#### `route()` {#collection-method}

Hàm `route` sinh ra một URL cho một route có tên:

    $url = route('routeName');

Nếu route này có nhận parameters, bạn có thể truyền chúng vào tham số thứ hai:

    $url = route('routeName', ['id' => 1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

Hàm `url` sinh ra URL đầy đủ cho một path cho trước:

    echo url('user/profile');

    echo url('user/profile', [1]);

Nếu path không được cung cấp, một instance của `Illuminate\Routing\UrlGenerator` sẽ được trả về:

    echo url()->current();
    echo url()->full();
    echo url()->previous();

<a name="miscellaneous"></a>
## Các hàm khác

<a name="method-auth"></a>
#### `auth()` {#collection-method}

Hàm `auth` trả về một instance của authenticator. Bạn có thể sử dụng nó thay vì `Auth` facade:

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

Hàm `back()` sinh ra một response để redirect lại location trước của user:

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

Hàm `bcrypt` thực hiện hash một giá trị sử dụng Bcrypt. Bạn có thể sử dụng nó như một cách thay thế cho `Hash` facade:

    $password = bcrypt('my-secret-password');

<a name="method-collect"></a>
#### `collect()` {#collection-method}

Hàm `collect` tạo một instance của [collection](/docs/{{version}}/collections) từ items được cung cấp:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

Hàm `config` lấy giá trị của một biến cấu hình. Giá trị cấu hình có thể truy xuất sử dụng kí hiệu "dot", bao gồm tên file và thông số muốn lấy ra. Giá trị mặc định có thể được truyền vào và trả lại nếu giá trị cấu hình không tồn tại:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Hàm `config` cũng có thể được dùng để đặt giá trị biến cấu hình lúc chạy bằng cách truyền vào một mảng các cặp key / value:

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

Hàm `csrf_field` sinh ra một trường HTML input `hidden` có chứa giá trị token CSRF. Ví dụ, khi sử dụng [Blade syntax](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

Hàm `csrf_token` trả về giá trị của token CSRF hiện tại:

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

Hàm `dd` được dùng để dump một biến ra và dừng việc thực thi script:

    dd($value);

Nếu bạn không muốn làm treo việc thực thi script, có thể sử dụng hàm `dump`:

    dump($value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

Hàm `dispatch` đẩy một job mới vào trong Laravel [job queue](/docs/{{version}}/queues):

    dispatch(new App\Jobs\SendEmails);

<a name="method-env"></a>
#### `env()` {#collection-method}

Hàm `env` lấy giá trị của biến môi trường hoặc trả lại giá trị mặc định:

    $env = env('APP_ENV');

    // Return a default value if the variable doesn't exist...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

Hàm `event` đẩy một [event](/docs/{{version}}/events) tới các listeners:

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

Hàm `factory` tạo một builder cho model factory cho một class, với tên và số lượng. Nó có thể sử dụng trong [testing](/docs/{{version}}/testing#model-factories) hoặc [seeding](/docs/{{version}}/seeding#using-model-factories):

    $user = factory(App\User::class)->make();

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

The `method_field` function generates an HTML `hidden` input field containing the spoofed value of the form's HTTP verb. For example, using [Blade syntax](/docs/{{version}}/blade):
Hàm `method_field` tạo ra một trường HTML input `hidden` chứa giá trị của động từ HTTP. Ví dụ, sử dụng [cú pháp Blade](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-old"></a>
#### `old()` {#collection-method}

Hàm `old` [lấy giá trị](/docs/{{version}}/requests#retrieving-input) input cũ flash vào trong session:

    $value = old('value');

    $value = old('value', 'default');

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

Hàm `redirect` trả về một instance của redirector để thực hiện [redirects](/docs/{{version}}/responses#redirects):

    return redirect('/home');

<a name="method-request"></a>
#### `request()` {#collection-method}

Hàm `request` trả về instance của [request](/docs/{{version}}/requests) hiện tại hay lấy item input:

    $request = request();

    $value = request('key', $default = null)

<a name="method-response"></a>
#### `response()` {#collection-method}

Hàm `response` tạo một instance của [response](/docs/{{version}}/responses) instance hoặc lấy instance của response factory:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-session"></a>
#### `session()` {#collection-method}

Hàm `session` có thể được dùng để get / set giá trị cho session:

    $value = session('key');

Bạn có thể set giá trị bằng cách truyền một mảng key / value vào hàm:

    session(['chairs' => 7, 'instruments' => 3]);

Session store sẽ được trả lại nếu không có giá trị nào truyền vào hàm:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-value"></a>
#### `value()` {#collection-method}

Hàm `value` đơn giản chỉ trả về giá trị được cung cấp. Tuy nhiên, nếu bạn truyền vào một `Closure`, thì `Closure` sẽ được thực thi và trả lại giá trị trong đó:

    $value = value(function() { return 'bar'; });

<a name="method-view"></a>
#### `view()` {#collection-method}

Hàm `view` lấy giá trị từ instance của [view](/docs/{{version}}/views):

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

Hàm `with` trả về giá trị nó được cung cấp. Hàm này mục đích chính hữu ích cho việc thực hiện móc nối các hàm:

    $value = with(new Foo)->work();
