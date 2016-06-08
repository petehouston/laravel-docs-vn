# Collections

- [Giới thiệu](#introduction)
- [Tạo Collections](#creating-collections)
- [Available Methods](#available-methods)

<a name="introduction"></a>
## Giới thiệu

Lớp `Illuminate\Support\Collection` cung cấp một gói làm việc dễ dàng, thuận lợi với các mảng dữ liệu. Ví dụ, hãy xem qua đoạn code sau. Chúng ta sẽ sử dụng helper(helper là các hàm được viết và có thể gọi dùng lại nhiều lần) `collect` để tạo mới một collection instance(instance là một biến mang giá trị) từ mảng, chạy hàm `strtoupper` trên mỗi phần tử, và sau đó xóa tất cả các phần tử trống:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

Như bạn có thể thấy, lớp `Collection` cho phép bạn đính kèm các phương thức để dễ dàng thực hiện xử lý và xóa phần từ mảng. Nói chung, mỗi phương thức `Collection` trả về một instance `Collection`.

<a name="creating-collections"></a>
## Tạo Collections

Như đã đề cập bên trên, helper `collect` trả về một instance `Illuminate\Support\Collection` mới cho một mảng. Do đó, tạo một collection đơn giản như sau:

    $collection = collect([1, 2, 3]);

Mặc định, các collections của các model [Eloquent](/docs/{{version}}/eloquent) luôn luôn trả về instances `Collection`; tuy nhiên, hãy thoải mái sử dụng lớp `Collection` bất cứ nơi nào nếu nó thuận tiện cho ứng dụng của bạn.

<a name="available-methods"></a>
## Các phương thức có sẵn

Trong tài liệu này, chúng ta sẽ thảo luận về từng phương thức có sẵn trong lớp `Collection`. Nhớ rằng tất cả những phương thức có thể được liên kết cho việc xử lý mảng một cách thuận lợi nhất. Hơn nữa, hầu hết mỗi phương thức đều trả về một instance `Collection` mới, cho phép bạn giữ lại bản gốc của collection khi cần thiết. 

Bạn có thể lựa chọn bất kì phương thức từ bảng dưới đây và xem ví dụ sử dụng nó:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">
[all](#method-all)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[contains](#method-contains)
[count](#method-count)
[diff](#method-diff)
[diffKeys](#method-diffkeys)
[each](#method-each)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[isEmpty](#method-isempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[max](#method-max)
[merge](#method-merge)
[min](#method-min)
[only](#method-only)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[sum](#method-sum)
[take](#method-take)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[values](#method-values)
[where](#method-where)
[whereLoose](#method-whereloose)
[whereIn](#method-wherein)
[whereInLoose](#method-whereinloose)
[zip](#method-zip)
</div>

<a name="method-listing"></a>
## Danh sách phương thức

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

Phương thức `all` đơn giản trả về mảng cơ bản được miêu tả bởi collection:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-avg"></a>
#### `avg()` {#collection-method}

Phương thức `avg` trả về trung bình cộng của tất cả phần tử trong collection:

    collect([1, 2, 3, 4, 5])->avg();

    // 3

Nếu collection bao gồm nhiều mảng hoặc đối tượng lồng nhau, bạn có thể truyền vào 1 key để phân biết giá trị nào dùng để tính trung bình:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->avg('pages');

    // 636

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

Phương thức `chunk` chia collection thành nhiều phần, các collection nhỏ hơn với kích thước đã cho:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

Phương thức này đặc biệt hữu ích trong [views](/docs/{{version}}/views) khi làm việc với grid system như [Bootstrap](http://getbootstrap.com/css/#grid). Tưởng tượng bạn có 1 collectioni của các model [Eloquent](/docs/{{version}}/eloquent) mà bạn muốn hiển thị dạng lưới:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

Phương thức `collapse` gộp một mảng collection thành một collection:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

Phương thức `combine` hợp nhất key của một collection với value của một mảng hoặc collection khác:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

Phương thức `contains` xác định xem colletion có bao gồm item đã cho hay không:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

Bạn cũng có thể truyền 1 cặp key / value vào phương thức `contains`, nó sẽ xác định sự tồn tại cặp đó trong collection:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

Cuối cùng, bạn cũng có thể truyền vào một callback vào phương thức `contains` để thực hiện test kiểm tra:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($key, $value) {
        return $value > 5;
    });

    // false

<a name="method-count"></a>
#### `count()` {#collection-method}

Phương thức `count` trả về tổng số phần tử trong collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

Phương thức `diff` so sánh collection với một collection khác hoặc một mảng php đơn giản dựa trên value của nó:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

Phương thức `diffKeys` so sánh collection với một collection khác hoặc mảng php đơn giản dựa trên key của nó:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-each"></a>
#### `each()` {#collection-method}

Phương thức `each` lặp qua các phần tử trong collection và truyền mỗi phần tử vào trong 1 callback:

    $collection = $collection->each(function ($item, $key) {
        //
    });

Return `false` trong callback để thoát khỏi vòng lặp:

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

Phương thức `every` tạo ra 1 collection mới bao gồm mỗi phần tử thứ n:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->every(4);

    // ['a', 'e']

Bạn có thể tùy chọn truyền offset(phần bỏ qua) như là tham số thứ 2:

    $collection->every(4, 1);

    // ['b', 'f']

<a name="method-except"></a>
#### `except()` {#collection-method}

Phương thức `except` trả về toàn bộ phần tử trọng collection trừ những phần từ với key đã được chỉ định:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Trái ngược với `except`, xem phương thức [only](#method-only)

<a name="method-filter"></a>
#### `filter()` {#collection-method}

Phương thức `filter` lọc qua collection bằng một callback, chỉ giữ lại những phần tử mà thông qua điều kiện đã cho:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Ngược lại của `filter`, xem phương thức [reject](#method-reject).

<a name="method-first"></a>
#### `first()` {#collection-method}

Phương thức `first` trả về phần từ đầu tiên trong collection mà thông qua điều kiện:

    collect([1, 2, 3, 4])->first(function ($key, $value) {
        return $value > 2;
    });

    // 3

Bạn cũng có thể gọi phương thức `first` không tham số để lấy về phần từ đầu tiên trong collection. Nếu collection rỗng, trả về `null`:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

The `flatMap` method iterates through the collection and passes each value to the given callback. The callback is free to modify the item and return it, thus forming a new collection of modified items. Then, the array is flattened by a level:
Phương thức `flatMap` lặp qua collection và truyền từng value vào callback. Callback sẽ thay đổi và trả lại nó, do vậy tạo ra 1 collection mới từ các phần tử được thay đổi. Sau đó mảng sẽ bị flatten bởi 1 level (flatten là gì các bạn xem bên dưới nhé).

    $collection = collect(
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return strtoupper($values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => 28];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

Phương thức `flatten` làm phẳng một collection nhiều chiều thành một chiều:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

You may optionally pass the function a "depth" argument:
Bạn có thể tùy chọn truyền vào hàm một tham số "depth":

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

Ở đây, nếu gọi `flatten` mà không cung cấp "depth" (độ sau) có thể cũng sẽ flatten cả những mảng lồng nhau, kết quả là `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Cung cấp "depth" cho phép bạn hạn chế mức độ của các mảng lồng nhau mà sẽ bị flatten.

<a name="method-flip"></a>
#### `flip()` {#collection-method}

Phương thức `flip` đảo ngược vị trí key và value của collection:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

Phương thức `forget` xóa một phần từ từ collection bởi key:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> **Note:** Không như hầu hết các phương thức collection, `forget` không trả về một collection mới được mà đã được thay đổi, nó thay đổi chính collection mà nó được gọi tới.

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

Phương thức `forPage` trả về một collection mới bao gồm các phần tử mà sẽ được xuất hiện với số trang đa cho:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

Phương thức này yêu cầu số trang và số phần từ mỗi trang, tương ứng với các tham số truyền vào.

<a name="method-get"></a>
#### `get()` {#collection-method}

Phương thức `get` trả về phần tử tại key đã cho. Nếu key không tồn tại, trả về `null`:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Bạn có thể tùy chọn truyền vào một giá trị mặc định như là tham số thứ 2:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

Bạn có thể truyền một callback như là giá trị mặc định. Giá trị của callback sẽ được trả về nếu key chỉ định không tồn tại:

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

Phương thức `groupBy` nhóm các phần tử của collection bởi một key:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

Thêm vào đó để truyền một `key` là chuỗi, bạn cũng có thể truyền vào một callback. Callback sẽ trả về giá trị bạn muốn làm key để nhóm:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

The `has` method determines if a given key exists in the collection:
Phương thức `has` xác định nếu key đã cho tồn tại trong collection:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('email');

    // false

<a name="method-implode"></a>
#### `implode()` {#collection-method}

Phương thức `implode` nối các phần tử thành trong collection. Tham số của nó dựa trên kiểu của các phần tử trong collection.

Nếu collection bào gồm các mảng hoặc object, bạn  phải truyền vào key hoặc thuộc tính bạn muốn join, và "glue" mà bạn muốn đặt giữa các giá trị:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Nếu collection chỉ đơn giản  bao gồm các chuỗi hoặc giá trị số, chỉ cần truyền "glue" như là tham số duy nhất cho phương thức:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

Phương thức `intersect` xóa các giá trị mà không xuất hiện trong `array` hoặc collection đã cho:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

Như bạn thấy, giá trị collection trả về sẽ giữ nguyên key của collection cũ.

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

Phương thức `isEmpty` trả về `true` nếu collection là rỗng, ngược lại trả về `false`:

    collect([])->isEmpty();

    // true

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

Keys the collection by the given key:
Tạo key cho collection bởi key đã cho (Keys the collection by the given key):

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'desk'],
        ['product_id' => 'prod-200', 'name' => 'chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

Nếu nhiều phần từ có chung key, chỉ có phần từ cuối cùng sẽ xuất hiện trong collection mới.

Bạn cũng có thể truyền vào một callback, cái mà sẽ trả về giá trị để làm khóa cho collection:

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */


<a name="method-keys"></a>
#### `keys()` {#collection-method}

Phương thức `keys` trả về oàn bộ key của collection:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

Phương thức `last` trả về phần thử cuối cùng trong collection mà thông qua điều kiện đã cho:

    collect([1, 2, 3, 4])->last(function ($key, $value) {
        return $value < 3;
    });

    // 2

Bạn có thể gọi phương thức `last` không tham số để lấy phần tử cuối cùng trong collection. Nếu collection rỗng, trả về `null`:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

Phương thức `map` lặp qua collection và truyền từng giá trị vào callback. Callback thay đổi phần từ và trả về, do vậy hình thành một collection mới từ các phần tử đã được thay đổi:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> **Ghi chú:** Giống như các phương thức khác, `map` trả về một instance collection mới; nó không thay đổi collection mà nó được gọi, Nếu bạn muốn thay đổi collection gốc, sử dụng phương thức [`transform](#method-transform).

<a name="method-max"></a>
#### `max()` {#collection-method}

Phương thức `max` trả về giá trị lớn nhất của một key đã cho:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

Phương thức `merge` gộp mảng đã cho vào collection. Bất kì string key trong mảng trùng với string key trong collection sẽ bị ghi đè bởi giá trị của collection:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $merged = $collection->merge(['price' => 100, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]

Nếu key của mảng đã cho là số, giá trị sẽ được xuất hiện tại cuối collection:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

Phương thức `min` trả về giá trị nhỏ nhất của key đã cho:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-only"></a>
#### `only()` {#collection-method}

Phương thức `only` trả về các phần từ trong collection với key xác định:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Ngược lại với `only`, xem phương thức [except](#method-except).

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

Phương thức `pluck` lấy toàn bộ các value của collection cho một key cho trước:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

Bạn cũng có thể chỉ định bạn muốn collection kết quả được chỉ mục như thế nào:

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

Phương thức `pop` xóa và trả lại phần tử cuối cùng từ collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

Phương thức `prepend` thêm một phân từ vào đầu của collection: 

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

Bạn có thể tùy chọn truyền tham số thứ 2 để set key cho giá trị được thêm:

    $collection = collect(['one' => 1, 'two', => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two', => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

Phương thức `pull` xóa và trả về một giá trị từ collection bởi key của nó:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

Phương thức `push` thêm một phần từ vào cuối của collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

The `put` method sets the given key and value in the collection:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

The `random` method returns a random item from the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

You may optionally pass an integer to `random`. If that integer is more than `1`, a collection of items is returned:

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

The `reduce` method reduces the collection to a single value, passing the result of each iteration into the subsequent iteration:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

The value for `$carry` on the first iteration is `null`; however, you may specify its initial value by passing a second argument to `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

The `reject` method filters the collection using the given callback. The callback should return `true` for any items it wishes to remove from the resulting collection:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

For the inverse of the `reject` method, see the [`filter`](#method-filter) method.

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

The `reverse` method reverses the order of the collection's items:

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

The `search` method searches the collection for the given value and returns its key if found. If the item is not found, `false` is returned.

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

The search is done using a "loose" comparison. To use strict comparison, pass `true` as the second argument to the method:

    $collection->search('4', true);

    // false

Alternatively, you may pass in your own callback to search for the first item that passes your truth test:

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

The `shift` method removes and returns the first item from the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

The `shuffle` method randomly shuffles the items in the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] // (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

The `slice` method returns a slice of the collection starting at the given index:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

If you would like to limit the size of the returned slice, pass the desired size as the second argument to the method:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

The returned slice will have new, numerically indexed keys. If you wish to preserve the original keys, pass `true` as the third argument to the method.

<a name="method-sort"></a>
#### `sort()` {#collection-method}

The `sort` method sorts the collection:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

The sorted collection keeps the original array keys. In this example we used the [`values`](#method-values) method to reset the keys to consecutively numbered indexes.

For sorting a collection of nested arrays or objects, see the [`sortBy`](#method-sortby) and [`sortByDesc`](#method-sortbydesc) methods.

If your sorting needs are more advanced, you may pass a callback to `sort` with your own algorithm. Refer to the PHP documentation on [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters), which is what the collection's `sort` method calls under the hood.

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

The `sortBy` method sorts the collection by the given key:

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

The sorted collection keeps the original array keys. In this example we used the [`values`](#method-values) method to reset the keys to consecutively numbered indexes.

You can also pass your own callback to determine how to sort the collection values:

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

This method has the same signature as the [`sortBy`](#method-sortby) method, but will sort the collection in the opposite order.

<a name="method-splice"></a>
#### `splice()` {#collection-method}

The `splice` method removes and returns a slice of items starting at the specified index:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

You may pass a second argument to limit the size of the resulting chunk:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

In addition, you can pass a third argument containing the new items to replace the items removed from the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

The `sum` method returns the sum of all items in the collection:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

If the collection contains nested arrays or objects, you should pass a key to use for determining which values to sum:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

In addition, you may pass your own callback to determine which values of the collection to sum:

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

The `take` method returns a new collection with the specified number of items:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

You may also pass a negative integer to take the specified amount of items from the end of the collection:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

The `toArray` method converts the collection into a plain PHP `array`. If the collection's values are [Eloquent](/docs/{{version}}/eloquent) models, the models will also be converted to arrays:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> **Note:** `toArray` also converts all of its nested objects to an array. If you want to get the underlying array as is, use the [`all`](#method-all) method instead.

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

The `toJson` method converts the collection into JSON:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk","price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

The `transform` method iterates over the collection and calls the given callback with each item in the collection. The items in the collection will be replaced by the values returned by the callback:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> **Note:** Unlike most other collection methods, `transform` modifies the collection itself. If you wish to create a new collection instead, use the [`map`](#method-map) method.

<a name="method-union"></a>
#### `union()` {#collection-method}

The `union` method adds the given array to the collection. If the given array contains keys that are already in the collection, the collection's values will be preferred:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], [3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

The `unique` method returns all of the unique items in the collection:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

The returned collection keeps the original array keys. In this example we used the [`values`](#method-values) method to reset the keys to consecutively numbered indexes.

When dealing with nested arrays or objects, you may specify the key used to determine uniqueness:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

You may also pass your own callback to determine item uniqueness:

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

<a name="method-values"></a>
#### `values()` {#collection-method}

The `values` method returns a new collection with the keys reset to consecutive integers:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */
<a name="method-where"></a>
#### `where()` {#collection-method}

The `where` method filters the collection by a given key / value pair:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
    */

The `where` method uses strict comparisons when checking item values. Use the [`whereLoose`](#method-whereloose) method to filter using "loose" comparisons.

<a name="method-whereloose"></a>
#### `whereLoose()` {#collection-method}

This method has the same signature as the [`where`](#method-where) method; however, all values are compared using "loose" comparisons.

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

The `whereIn` method filters the collection by a given key / value contained within the given array.

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
    [
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Desk', 'price' => 200],
    ]
    */

The `whereIn` method uses strict comparisons when checking item values. Use the [`whereInLoose`](#method-whereinloose) method to filter using "loose" comparisons.

<a name="method-whereinloose"></a>
#### `whereInLoose()` {#collection-method}

This method has the same signature as the [`whereIn`](#method-wherein) method; however, all values are compared using "loose" comparisons.

<a name="method-zip"></a>
#### `zip()` {#collection-method}

The `zip` method merges together the values of the given array with the values of the collection at the corresponding index:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]