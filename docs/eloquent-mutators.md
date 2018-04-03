# Eloquent: Mutators

- [Giới thiệu](#introduction)
- [Accessors & Mutators](#accessors-and-mutators)
- [Date Mutators](#date-mutators)
- [Attribute Casting](#attribute-casting)

<a name="introduction"></a>
## Giới thiệu

Accessor và mutator cho phép bạn format các attributes của Eloquent khi lấy ra từ một model hay đặt giá trị cho chúng. Ví dụ, bạn có thể muốn sử dụng [Laravel encrypter](/docs/{{version}}/encryption) để mã hoá một giá trị khi được lưu vào trong database, và tự động giải mã attribute đó khi truy xuất từ trong Eloquent model.

Ngoài việc hỗ trợ tạo accessor và mutator riêng, Eloquent cũng tự động chuyển các trường date thành [Carbon](https://github.com/briannesbitt/Carbon) instance hoặc thậm chí [chuyển trường text thành JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>
## Accessors & Mutators

#### Khai báo một accessor

Để khai báo một accessor, tạo một hàm `getFooAttribute` trong model với `Foo` là tên của cột bạn muốn truy cập sử dụng kiểu "camel". Ở ví dụ này, chúng ta sẽ khai báo một accessor cho `first_name`. Accessor sẽ tự động được gọi bởi Eloquent khi lấy giá trị của `first_name`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

Như bạn thấy, giá trị gốc của column được truyền vào accessor, cho phép bạn thay đổi và trả về giá trị. Để lấy gía trị này, chỉ cần truyền vào tên attribute là `first_name`:

    $user = App\User::find(1);

    $firstName = $user->first_name;

#### Khai báo một mutator

Để khai báo một mutator, khai báo một hàm `setFooAttribute` với `Foo` là tên của cột theo kiểu "camel". Lần nữa, chúng ta cùng khai báo một mutator cho trường `first_name`. Mutator này sẽ tự động được gọi khi chúng ta lấy giá trị `first_name`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

Mutator sẽ nhận giá trị được gán vào, cho phép bạn thay đổi tuỳ ý trong thuộc tính `$attributes` của Eloquent model. Ví dụ, chúng ta có thể gán giá trị `Sally` cho `first_name`:

    $user = App\User::find(1);

    $user->first_name = 'Sally';

Ở ví dụ này, hàm `setFirstNameAttribute` sẽ được gọi với giá trị là `Sally`. Mutator rồi sẽ thực hiện hàm `strtolower` để chỉnh lại tên và gán giá trị vào trong mảng `$attributes`.

<a name="date-mutators"></a>
## Date Mutators

Mặc định, Eloquent sẽ convert `created_at` và `updated_at` thành instances của [Carbon](https://github.com/briannesbitt/Carbon), một thư viện cung cấp rất nhiều hàm hữu ích và mở rộng class `DateTime` của PHP.

Bạn có thể tuỳ chỉnh trường nào sẽ được tự động mutate, và thậm chí có thể disable việc mutation này bằng cách ghi đè lên thuộc tính `$dates` của model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['created_at', 'updated_at', 'deleted_at'];
    }

Khi một column là kiểu date, bạn có thể đặt giá trị của nó là một UNIX timestamp, date string (`Y-m-d`), date-time string và tất nhiên là một instance của `DateTime` / `Carbon`, và giá trị của date sẽ tự động được lưu vào trong database:

    $user = App\User::find(1);

    $user->deleted_at = Carbon::now();

    $user->save();

Như ghi ở trên, khi lấy các attributes liệt kê trong thuộc tính `$date`, chúng sẽ tự động được cast thành instance của [Carbon](https://github.com/briannesbitt/Carbon), cho phép bạn sử dụng các hàm Carbon trong attributes:

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

Mặc định, timestamps được format dưới dạng `'Y-m-d H:i:s'`. Nếu bạn muốn tuỳ chỉnh format của timestamp, thiết lập vào thuộc tính `$dateFormat`. Thuộc tính này xác định các cách date attributes được lưu như thế nào trong database, cũng như format chúng khi model được serialize thành một array hay JSON:

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

<a name="attribute-casting"></a>
## Attribute Casting

Thuộc tính `$casts` cung cấp một phương thức convert attributes thành các kiểu dữ liệu khác nhau khá tiện lợi. Thuộc tính `$casts` là một mảng có key là tên của attribute được cast, còn giá trị là kiểu dữ liệu bạn muốn cast. Các kiểu dữ liệu để cast được hỗ trợ bao gồm: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, và `timestamp`.

For example, let's cast the `is_admin` attribute, which is stored in our database as an integer (`0` or `1`) to a boolean value:
Ví dụ, hãy cast attribute `is_admin`, được lưu trong database là integer (`0` hoặc `1`) thành boolean:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be casted to native types.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

Lúc này, attribute `is_admin` sẽ luôn luôn được cast thành boolean khi bạn truy cập nó, thậm chí nếu giá trị nó được lưu trong database là kiểu integer:

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

#### Array Casting

Cast kiểu `array` hữu dụng khi làm việc với các columns được lưu thành JSON. Ví dụ, nếu database có một trường kiểu `TEXT` có chứa serialized JSON, thêm vào `array` cast lên attribute sẽ tự động de-serialize attribute thành mảng PHP khi bạn access trong Eloquent model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be casted to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Khi mà cast đã được khai báo, bạn có thể lấy giá trị `options` và nó sẽ tự dộng deserialize từ JSON thành PHP array. Khi bạn gán giá trị vào `options`, nó sẽ tự động được serialize trở lại thành JSON để lưu vào trong cơ sở dữ liệu:

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
