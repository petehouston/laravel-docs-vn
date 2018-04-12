# Eloquent: Getting Started

## Giới thiệu

Eloquent ORM đi kèm với Laravel cung cấp một API ActiveRecord đơn giản và tiện lợi cho giao tiếp với database. Mỗi database table sẽ có một "Model" tương ứng để tương tác với table đó. Model cho phép bạn query dữ liệu trong table, cũng như chèn thêm các dữ liệu mới.

Trước khi bắt đầu, hãy đảm bảo cấu hình kết nối database trong `config/database.php`. Để biết thêm thông tin chi tiết cho cấu hình database, hãy xem [mục cấu hình database](database.md#configuration).

## Tạo model

Để bắt đầu, hãy cùng tạo một Eloquent model. Model về cơ bản nằm trong thư mục `app`, nhưng bạn có thể tuỳ ý đặt chúng ở bất cứ đâu mà được cấu hình autoload trong `composer.json`. Tất cả các Eloquent model đều kế thừa từ class `Illuminate\Database\Eloquent\Model`.

Cách đơn giản nhất để tạo một model là sử dụng [câu lệnh Artisan](artisan.md) `make:model`:

```php
php artisan make:model User
```

Nếu bạn muốn tạo [migration](migrations.md) đi kèm với model khi tạo thì sử dụng thêm cờ `--migration` hay `-m`:

```php
php artisan make:model User --migration

php artisan make:model User -m
```

### Quy tắc cho Eloquent Model

Bây giờ, hãy cùng nhau coi ví dụ về class model `Flight`, mà chúng ta sẽ dùng để lấy và lưu thông tin vào trong bảng `flights`:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    //
}
```

#### Tên table

Để ý là chúng ta không hề cho Eloquent biết là bảng nào được sử dụng cho model `Flight`. Vì kiểu "snake case", tên class ở số nhiều sẽ được sử dụng như tên table trừ khi có một tên khác được khai báo. Vì thế, trong trường hợp này, Eloquent sẽ coi model `Flight` lưu dữ liệu vào trong table `flights`. Bạn có thể chỉ định tên table khác cho model bằng cách khai báo thuộc tình `$table`:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
      * The table associated with the model.
      *
      * @var string
      */
    protected $table = 'my_flights';
}
```

#### Primary Keys

Eloquent cũng coi mỗi table có một column là primary key tên là `id`. Bạn có thể khai báo thuộc tính `$primaryKey` để đổi tên column này.

Thêm nữa, Eloquent cũng coi primary key là một giá trị nguyên tăng dần, có nghĩa là về mặc định primary key sẽ được cast về kiểu `int` tự động. Nếu bạn muốn sử dụng primary không tăng dần hay không phải là dạng số, bạn cần thay đổi thuộc tính `$incrementing` trong model thành `false`.

#### Timestamps

Mặc định, Eloquent cần hai column `created_at` và `updated_at` có mặt trong các table. Nếu bạn không muốn những columns này tự động được quản lý bởi Eloquent, thiết lập thuộc tinh `$timestamps` thành `false`:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
      * Indicates if the model should be timestamped.
      *
      * @var bool
      */
    public $timestamps = false;
}
```

Nếu bạn muốn thay đổi định dạng của timestamp, thiết lập vào thuộc tính `$dateFormat` trong model. Thuộc tính này xác định cách mà các thuộc tính kiểu date được lưu trong database cũng như cách format khi được serialize thành array hay JSON:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
      * The storage format of the model's date columns.
      *
      * @var string
      */
    protected $dateFormat = 'U';
}
```

Nếu bạn cần tuỳ chỉnh tên của các cột được sử dụng để lưu trữ dấu thời gian, bạn có thể đặt  ``CREATED_AT`` và các ``UPDATED_AT`` trong mô hình của mình:

```php
<?php

class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'last_update';
}
```

#### Kết nối database

Tất cả các Eloquent model sẽ sử dụng kết nối database mặc định được cấu hình. Nếu bạn muốn sử dụng một kết nối khác cho model, sử dụng thuộc tính `$connection`:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
      * The connection name for the model.
      *
      * @var string
      */
    protected $connection = 'connection-name';
}
```

## Lấy nhiều model kết quả

Khi bạn đã tạo được một model và [table tương ứng](migrations.md#writing-migrations) của nó, bạn có thể sẵn sàng truy xuất dữ liệu từ database. Hãy coi mỗi Eloquent model như một [query builder](queries.md) mạnh mẽ cho phép bạn thực hiện query tới database một cách liền mạch. Ví dụ:

```php
<?php

namespace App\Http\Controllers;

use App\Flight;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
      * Show a list of all available flights.
      *
      * @return Response
      */
    public function index()
    {
        $flights = Flight::all();

        return view('flight.index', ['flights' => $flights]);
    }
}
```

#### Truy xuất giá trị của column

Nếu bạn đã có một model instance, bạn có thể lấy giá trị của column trong model bằng cách gọi tên thuộc tính tương ứng. Ví dụ, hãy cùng lặp qua mỗi instance `Flight` trả về bởi query và hiển thị giá trị của column `name`:

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

#### Thêm ràng buộc bổ sung

Hàm `all` sẽ trả về tất cả các kết quả trong table của model. Vì mỗi Eloquent model phục vụ như một [query builder](queries.md), nên bạn có thể tạo ràng buộc cho các query, và cuối cùng sử dụng hàm `get` để lấy kết quả:

```php
$flights = App\Flight::where('active', 1)
                ->orderBy('name', 'desc')
                ->take(10)
                ->get();
```

> **Chú ý:** Vì các Eloquent model là các query builder, bạn nên xem qua tất cả các hàm có thể sử dụng trên [query builder](queries.md). Bạn có thể áp dụng bất kì hàm nào trong này với Eloquent query.

### Collections

Vì các hàm của Eloquent như `all` và `get` đều trả về nhiều kết quả, một instance từ `Illuminate\Database\Eloquent\Collection` sẽ được trả về. Class `Collection` cung cấp [các hàm hữu ích](eloquent-collections.md#available-methods) để làm việc với tập kết quả của Eloquent. Tất nhiên, bạn có thể thực hiện lặp collection này như một array:

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

### Tạo khối kết quả

Nếu bạn muốn xử lý hàng ngàn kết quả từ Eloquent, sử dụng hàm `chunk`. Hàm này sẽ lấy từng "khối" kết quả của Eloquent model, cung cấp chúng thông qua `Closure` để xử lý. Sử dụng hàm này sẽ tiết kiệm được memory khi thao tác với tập dữ liệu kết quả lớn:

```php
Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
```

Tham số đầu truyền vào là số record bạn muốn lấy từng "khối" (chunk). Closure truyền vào ở tham số thứ hai sẽ được gọi cho mỗi chunk được lấy từ database.

#### Sử dụng Cursors

Hàm cursor cho phép bạn duyệt qua records bằng cách sử dụng một cursor, nó chỉ thực thi cho một truy vấn. Khi dữ liệu lớn, hàm cursor có thể được sử dụng để giảm memory sử dụng:

```php
foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
    //
}
```

## Lấy single model / aggregate

Ngoài việc lấy tất cả dữ liệu, bạn có thể lấy một kết quả sử dụng hàm `find` và `first`. Thay vì trả về một collection model, những hàm này trả về một model instance:

```php
// Retrieve a model by its primary key...
$flight = App\Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = App\Flight::where('active', 1)->first();
```

Bạn có thể gọi hàm `find` với một mảng các primary key, với kết quả trả về là một collection các kết quả tìm thấy:

```php
$flights = App\Flight::find([1, 2, 3]);
```

#### Not Found Exceptions

Sẽ có lúc bạn muốn bắn ra một exception nếu một model không được tìm thấy. Điều này thực sự hữu ích khi làm việc trên route hay controller. Hàm `findOrFail` và `firstOrFail` sẽ trả lại kết quả đầu tiên của query. Tuy nhiên, nếu không có kết quả, thì `Illuminate\Database\Eloquent\ModelNotFoundException` sẽ được bắn ra:

```php
$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();
```

Nếu exception mà không được bắt, một HTTP response `404` sẽ tự động được gửi lại cho user, vì thế, không cần thiết phải viết code riêng để kiểm tra để trả về `404` khi sử dụng những hàm này:

```php
Route::get('/api/flights/{id}', function ($id) {
    return App\Flight::findOrFail($id);
});
```

### Lấy kết quả tính toán

Bạn cũng có thể sử dụng các hàm như `count`, `sum`, `max` hay [các hàm tập hợp khác](queries.md#aggregates) được cung cấp bởi [query builder](queries.md). Những hàm này trả về một kết quả thay vì một model instance:

```php
$count = App\Flight::where('active', 1)->count();

$max = App\Flight::where('active', 1)->max('price');
```

## Thêm và cập nhật model

### Thao tác thêm dữ liệu cơ bản

Để thêm dữ liệu mới vào database, đơn giản hãy tạo một model instance mới, thiết lập các attributes vào model rồi gọi hàm `save`:

```php
<?php

namespace App\Http\Controllers;

use App\Flight;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
      * Create a new flight instance.
      *
      * @param  Request  $request
      * @return Response
      */
    public function store(Request $request)
    {
        // Validate the request...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();
    }
}
```

Ở ví dụ này, chúng ta có thể đơn giản chỉ gán tham số `name` từ HTTP request vào thuộc tính `name` của model `App\Flight`. Khi gọi hàm `save`, một record sẽ được thêm vào database. Timestamp `created_at` và `updated_at` cũng tự động được tạo, nên không cần thiết phải thêm vào thủ công.

### Thao tác cập nhật dữ liệu cơ bản

Hàm `save` cũng được dùng để cập nhật model đã tồn tại sẵn trong database. Để update, bạn cần lấy model instance ra trước, thay đổi các attribute ban muốn, rồi gọi hàm `save`. Lúc này, giá trị của `updated_at` cũng sẽ tự động được cập nhật, và bạn không cần thay đổi thủ công giá trị này:

```php
$flight = App\Flight::find(1);

$flight->name = 'New Flight Name';

$flight->save();
```

#### Mass Updates

Update cũng có thể được thực hiện cho nhiều model mà thoả mãn một điều kiện quẻy. Ở ví dụ này, tất cả các flights mà `active` và có `destination` là `San Diego` sẽ bị đánh dấu là delayed:

```php
App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);
```

Hàm `update` nhận một mảng các column và cặp giá trị tương ứng column nào cần được update.

>Khi thực hiện một mass update bằng Eloquent, hàm ``saved`` và ``updated`` model events sẽ không được bắn ra cho các model updated. Vì các model thực ra chưa bao giờ lấy khi mass update.

### Mass Assignment

Bạn cũng có thể sử dụng hàm `create` để tạo một model mới chỉ trong một dòng. Model instance được thêm mới sẽ được trả lại từ hàm. Tuy nhiên, để làm được điều đó, bạn cần thiết phải chỉ định thuộc tinh `$fillable` hoặc `$guarded` trong model, để Eloquent model được bảo vệ trước mass-assignment.

Lỗi bảo mật mass-assignment xảy ra khi một user truyền vào một tham số HTTP không mong muốn trong request, và tham số đó sẽ có thể thay đổi một column trong database mà bạn không ngờ tới. Ví dụ, một user xấu có thể gửi một tham số `is_admin` qua HTTP request, và khi giá trị này được map vào trong model qua hàm `create`, sẽ cho phép user thay đổi để biến thành một admin.

Vì thế, để bắt đầu, bạn cần khai báo thuộc tính bạn muốn cho phép mass-assignment. Bạn có thể thiết lập qua thuộc tính `$fillable`. Ví dụ, hãy làm cho thuộc tính `name` trong model `Flight` có thể được sử dụng qua mass-assignment:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
      * The attributes that are mass assignable.
      *
      * @var array
      */
    protected $fillable = ['name'];
}
```

Sau đó, chúng ta có thể sử dụng hàm `create` để tạo một record mới trong database:

```php
$flight = App\Flight::create(['name' => 'Flight 10']);
```

Nếu như `$fillable` dùng để lưu danh sách các thuộc tính "được phép" (white list) mass-assign, bạn có thể sử dụng `$guarded` để lưu các thuộc tính mà **không được phép** mass-assign. Các thuộc tính khác không lưu trong `$guarded` sẽ được mass-assign. Vì thế, `$guarded` được coi như là một "black list". Bạn có thể sử dụng một trong hai, hoặc `$fillable` hoặc `$guarded`, chứ không được dùng cả hai cùng một lúc:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
      * The attributes that aren't mass assignable.
      *
      * @var array
      */
    protected $guarded = ['price'];
}
```

Sau đó, chúng ta có thể sử dụng hàm create để tạo một record mới trong database. Hàm create sẽ trả về một model instance được lưu:

```php
$flight = App\Flight::create(['name' => 'Flight 10']);
```

##### Thuộc tính guarding

Trong khi `$fillable` dùng để lưu danh sách các thuộc tính "white list" được mass assignable, bạn có thể sử dụng `$guarded`. Thuộc tính `$guarded` để lưu các thuộc tính mà không được phép mass assignable. Các thuộc tính khác không lưu trong nó sẽ dượcd mass assignable. Vì vậy, `$guarded` giống như là một "black list". Tất nhiên, Bạn chỉ có thể sử dụng một trong hai, `$fillable` hoặc  `$guarded` - không cả hai. Trong ví dụ dưới, tất cả các thuộc tính ngoại trừ price sẽ được mass assignable:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The attributes that aren't mass assignable.
     *
     * @var  array
     */
    protected $guarded = ['price'];
}
```

Nếu bạn muốn tất cả các thuộc tính mass assignable, bạn định nghĩa thuộc tính `$guarded` là một mảng rỗng:

```php
/**
 * The attributes that aren't mass assignable.
 *
 * @var  array
 */
protected $guarded = [];
```

### Các hàm tạo khác

#### firstOrCreate/ firstOrNew

Còn hai hàm khác bạn có thể sử dụng để model bằng cách mass-assignment các attributes: `firstOrCreate` và `firstOrNow`. Hàm `firstOrCreate` sẽ cố gắng tìm trong database sử dụng cặp column và giá trị truyền vào. Nếu model không được tìm thấy trong database, một dòng record mới sẽ được thêm vào với các attributes được truyền vào.

Hàm `firstOrNew`, giống như `firstOrCreate` sẽ cố gắng tìm record trong database khớp với các attribute truyền vào. Tuy nhiên, nếu model không tìm tháy, một model instance mới sẽ được trả về. Chú ý là model được trả về bởi `firstOrNew` vẫn chưa được lưu vào database. Bạn cần gọi hàm `save` để lưu nó lại:

```php
// Retrieve the flight by the attributes, or create it if it doesn't exist...
$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

// Retrieve the flight by the attributes, or instantiate a new instance...
$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);
```

#### updateOrCreate

Bạn cũng có thể gặp những tình huống mà bạn muốn cập nhật một mô hình hiện tại hoặc tạo một mô hình mới nếu không tồn tại. Laravel cung cấp một phương pháp `updateOrCreate` để làm điều này trong một bước. Giống như phương pháp `firstOrCreate`, `updateOrCreate` vẫn tồn tại mô hình, vì vậy không cần phải gọi :save()

```php
// If there's a flight from Oakland to San Diego, set the price to $99.
// If no matching model exists, create one.
$flight = App\Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99]
);
```

## Xoá model

Để thực hiện xoá model, gọi hàm `delete` trên model instance:

```php
$flight = App\Flight::find(1);

$flight->delete();
```

#### Xoá model tồn tại bằng một key

Ở ví dụ trên, chúng ta lấy model từ database trước khi gọi hàm `delete`. Tuy nhiên, nếu bạn đã biết primary key của model, bạn có thể xoá model mà không cần lấy nó ra. Để làm được việc này, bạn chỉ cần gọi hàm `destroy`:

```php
App\Flight::destroy(1);

App\Flight::destroy([1, 2, 3]);

App\Flight::destroy(1, 2, 3);
```

#### Xoá nhiều model sử dụng query

Bạn cũng có thể thực hiện gọi một query để xoá một tập hợp các model. Ở ví dụ này, chúng ta sẽ xoá tất cả các flights được đánh dấu là inactive:

```php
$deletedRows = App\Flight::where('active', 0)->delete();
```

### Soft Deleting

Thay vì thực sự xoá các record khỏi database, Eloquent cũng cung cấp kiểu "soft delete" (xoá mềm) model. Khi model được soft deleted, chúng chưa thực sự bị xoá khỏi database. Thay vì thế, một trường là `deleted_at` sẽ được thiết lập trong model và chèn vào trong database. Nếu model có giá trị `deleted_at` khác NULL, tức là model đã bị soft deleted. Để kích hoạt xoá mềm cho một model, sử dụng trait `Illuminate\Database\Eloquent\SoftDeletes` trên model và thêm vào column `deleted_at` vào trong thuộc tính `$dates` của model:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;

    /**
      * The attributes that should be mutated to dates.
      *
      * @var array
      */
    protected $dates = ['deleted_at'];
}
```

Tất nhiên là bạn cần phải thêm column `deleted_at` vào trong table. Điều này có thể thực hiện qua việc sử dụng một helper được cung cấp để tạo trên [schema builder](migrations.md):

```php
Schema::table('flights', function ($table) {
    $table->softDeletes();
});
```

Lúc này, khi bạn gọi hàm `delete` trên model, column `deleted_at` sẽ được set vào current date và time. Và, khi thực hiện query một model có sử dụng soft delete, thì model đó sẽ tự động bị loại khỏi tất cả các kết qủa query.

Để xác định nếu một model instance bị soft delete, sử dụng hàm `trashed`:

```php
if ($flight->trashed()) {
    //
}
```

### Thực hiện query các soft delete model

#### Thêm các soft delete model vào kết quả

Như đã ghi chú ở trên, soft delete mode sẽ tự động bị tách khỏi các kết quả query. Tuy nhiên, bạn có thể ép các soft delete model xuất hiện trên tập kết quả sử dụng hàm `withTrashed`:

```php
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
```

Hàm `withTrashed` cũng có thể được dùng trong các câu query có dùng [relationship](eloquent-relationships.md):

```php
$flight->history()->withTrashed()->get();
```

#### Chỉ lấy các soft delete model vào kết quả

Hàm `onlyTrashed` sẽ **chỉ** lấy các soft delete model:

```php
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```

#### Phục hồi các soft delete model

Thi thoảng bạn cũng muốn **không xoá** một soft delete model. Để khôi phục lại một soft delete model về trạng thái active, hãy sử dụng hàm `restore`:

```php
$flight->restore();
```

Bạn cũng có thể dùng hàm `restore` trên một query để nhanh chóng khôi phục nhiều model:

```php
App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
```

Tương tự hàm `withTrashed`, hàm `restore` cũng có thể áp dụng trong [relationships](eloquent-relationships.md):

```php
$flight->history()->restore();
```

#### Xoá các model vĩnh viễn

Bạn có thể cần thực xoá một model khỏi database. Để xoá vĩnh viễn một soft delete model, hãy sử dụng hàm `forceDelete`:

```php
// Force deleting a single model instance...
$flight->forceDelete();

// Force deleting all related models...
$flight->history()->forceDelete();
```

## Query Scopes

### Global Scopes

Global scope cho phép bạn thêm các constraint vào **tất cả** các query cho một model. Chức năng [soft deleting](#soft-deleting) của Laravel thực hiện trên global scope chỉ với các model "chưa bị xoá" trong database. Viết global scope riêng của bạn có thể tạo một cách dễ dàng để đảm bảo mỗi query cho một model nhận đúng constraint.

#### Viết global scope

Viết một global scope khá đơn giản. Tạo một class triển khai từ interface `Illuminate\Database\Eloquent\Scope`. Interface này yêu cầu bạn viết mã cho một hàm `apply`. Hàm này có thể nhận constraint `where` vào query khi cần thiết:

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Eloquent\Scope;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class AgeScope implements Scope
{
    /**
      * Apply the scope to a given Eloquent query builder.
      *
      * @param  \Illuminate\Database\Eloquent\Builder  $builder
      * @param  \Illuminate\Database\Eloquent\Model  $model
      * @return void
      */
    public function apply(Builder $builder, Model $model)
    {
        return $builder->where('age', '>', 200);
    }
}
```

Không có thư mục định sẵn để lưu các scope trong Laravel, vì thế bạn có thể vô tư đặt hay tạo thư mục `Scopes` bên trong thư mục `app`.

#### Áp dụng global scope

Để gán một global scope cho một model, bạn cần ghi đè lại hàm `boot` của model đó và sử dụng hàm `addGlobalScope`:

```php
<?php

namespace App;

use App\Scopes\AgeScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
      * The "booting" method of the model.
      *
      * @return void
      */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new AgeScope);
    }
}
```

Sau khi thêm vào scope, thì câu query `User::all()` sẽ tạo ra câu SQL như sau:

```php
select * from `users` where `age` > 200
```

#### Các global scope vô danh

Eloquent cũng cho phép bạn tạo các global scope sử dụng Closure, điều này khá hữu ích cho các scope đơn giản mà không cần tạo class riêng:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class User extends Model
{
    /**
      * The "booting" method of the model.
      *
      * @return void
      */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope('age', function(Builder $builder) {
            $builder->where('age', '>', 200);
        });
    }
}
```

Tham số đầu tiên truyền vào `addGlobalScope()` là identifier để loại bỏ scope khi cần thiết:

```php
User::withoutGlobalScope('age')->get();
```

#### Xoá các global scope

Nếu bạn muốn bỏ một global scope cho một câu query, bạn có thể sử dụng `withoutGlobalScope`:

```php
User::withoutGlobalScope(AgeScope::class)->get();
```

Nếu bạn muốn bỏ một vài hoặc tất cả các global scope, bạn có thể dùng `withoutGlobalScopes`:

```php
User::withoutGlobalScopes()->get();

User::withoutGlobalScopes([FirstScope::class, SecondScope::class])->get();
```

### Local Scopes

Local scope cho phép bạn tạo tập hợp các ràng buộc thường dùng mà bạn có thể tái sử dụng trong chương trình. Ví dụ, bạn có thể hay lấy tất cả các "popular" users. Để tạo một scope, chỉ cần đặt tiền tố `scope` trong một hàm của Eloquent model:

Scope luôn luôn trả về một instance của query builder:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
      * Scope a query to only include popular users.
      *
      * @return \Illuminate\Database\Eloquent\Builder
      */
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }

    /**
      * Scope a query to only include active users.
      *
      * @return \Illuminate\Database\Eloquent\Builder
      */
    public function scopeActive($query)
    {
        return $query->where('active', 1);
    }
}
```

#### Sử dụng query scope

Khi scope được khai báo, bạn có thể sử dụng hàm của scope khi thực hiện query model. Tuy nhiên, bạn không cần thêm vào tiền tố `scope` khi gọi hàm. Bạn thậm chí có thể gọi móc nối các scope liên tiếp, ví dụ:

```php
$users = App\User::popular()->active()->orderBy('created_at')->get();
```

#### Scope động

Bạn có thể muốn tạo scope có nhận tham số. Tham số của scope cần được thêm vào sau tham số `$query`:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
      * Scope a query to only include users of a given type.
      *
      * @return \Illuminate\Database\Eloquent\Builder
      */
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}
```

Lúc này, bạn có thể truyền vào tham số khi gọi scope:

```php
$users = App\User::ofType('admin')->get();
```

## Events

Eloquent model bắn ra một số các events, cho phép bạn có thể hook vào nhiều điểm của model lifecycle sử dụng các hàm sau: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

Bất cứ khi nào một model mới được lưu lần đầu tiên, hai event `creating` và `created` sẽ được bắn ra. Nếu model đã tồn tại trong database và hàm `save` được gọi thì hai event `updating` / `updated` sẽ được bắn ra. Tuy nhiên, trong cả hai trường hợp thì `saving` / `saved` cũng đều được bắn ra.

Ví dụ, cùng tạo một listener cho Eloquent event trong một [service provider](providers.md). Bên trong event listener, chúng ta sẽ gọi hàm `isValid` trên model, và `false` sẽ được trả về nếu model không hợp lệ. Việc trả về giá trị `false` từ một Eloquent event listener sẽ huỷ bọ hai thao tác `save` / `update`:

```php
<?php

namespace App\Providers;

use App\User;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
      * Bootstrap any application services.
      *
      * @return void
      */
    public function boot()
    {
        User::creating(function ($user) {
            if ( ! $user->isValid()) {
                return false;
            }
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

### Observers

Nếu bạn đang listening nhiều events trong model, bạn có thể sử dụng observers để nhóm tất cả các listeners thành một class. Observers classes có tên phương thức, nó sẽ tương tác với Eloquent events mà bạn muốn listen. Mỗi một phương thức nhận đối số duy nhất là tên model. Laravel không thêm thư mục mặc định cho observers, bạn có thể tạo chúng ở bất kỳ đâu để chứa observer classes:

```php
<?php

namespace App\Observers;

use App\User;

class UserObserver
{
    /**
     * Listen to the User created event.
     *
     * @param    User  $user
     * @return  void
     */
    public function created(User $user)
    {
        //
    }

    /**
     * Listen to the User deleting event.
     *
     * @param    User  $user
     * @return  void
     */
    public function deleting(User $user)
    {
        //
    }
}
```

Để đăng ký một observer, sử dụng hàm observe trong model bạn muốn observe. Bạn có thể đăng ký observers trong hàm boot của service providers. Trong ví dụ này, chúng ta sẽ đăng ký observer trong AppServiceProvider:

```php
<?php

namespace App\Providers;

use App\User;
use App\Observers\UserObserver;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return  void
     */
    public function boot()
    {
        User::observe(UserObserver::class);
    }

    /**
     * Register the service provider.
     *
     * @return  void
     */
    public function register()
    {
        //
    }
}
```