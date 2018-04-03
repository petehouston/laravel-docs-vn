# Eloquent: Serialization

- [Giới thiệu](#introduction)
- [Sử dụng cơ bản](#basic-usage)
- [Giấu attributes khỏi JSON](#hiding-attributes-from-json)
- [Thêm giá trị vào JSON](#appending-values-to-json)

<a name="introduction"></a>
## Giới thiệu

Khi xây dựng JSON API, bạn sẽ cần phải convert model và relationship thành mảng hay JSON. Eloquent cung cấp sẵn các hàm tiện ích để thực hiện việc này, cũng như các thao tác xử lý attributes đi kèm trong serialization.

<a name="basic-usage"></a>
## Sử dụng cơ bản

#### Convert một model thành mảng

Khi convert một model và [relationships](/docs/{{version}}/eloquent-relationships) của nó thành một mảng, bạn có thể sử dụng hàm `toArray`. Hàm này thực hiện đệ quy, vì thế tất cả các attributes và relations (bao gồm relation của relations nữa) sẽ được convert thành mảng:

    $user = App\User::with('roles')->first();

    return $user->toArray();

Bạn cũng có thể convert [collections](/docs/{{version}}/eloquent-collections) thành mảng:

    $users = App\User::all();

    return $users->toArray();

#### Convert một model thành JSON

Để convert một model thành JSON, bạn có thể sử dụng hàm `toJson`. Giống như `toArray`, hàm `toJson` cũng đệ quy và tất cả attributes cùng với relations sẽ được convert thành JSON:

    $user = App\User::find(1);

    return $user->toJson();

Ngoài ra, bạn có thể cast model hay collection thành string, việc này sẽ tự động gọi tới hàm `toJson`:

    $user = App\User::find(1);

    return (string) $user;

Vì model và collection được convert thành JSON khi cast thành string, bạn có thể trả về đối tượng Eloquent trực tiếp từ trong route hay controller:

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## Giấu các attributes khỏi JSON

Sẽ có lúc bạn muốn giới hạn attributes như mật khẩu không được hiển thị trong kết quả array hay JSON sau khi convert. Để làm được điều đó, thêm vào thuộc tính `$hidden` vào trong model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> **Chú ý:** Khi giấu relationships, sử dụng tên **method** của relationship, chứ không phải thuộc tính động của nó.

Một cách khác, bạn có thể sử dụng thuộc tính `visible` để khai báo danh sách các attributes cần được thêm vào trong kết quả array và JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### Tạm thời mở attribute bị giấu

Nếu bạn muốn làm cho một số thuộc tính đã giấu có thể thấy được, sử dụng hàm `makeVisible`. Hàm này trả về một đối tượng của model làm cho việc móc nối tiện hơn:

    return $user->makeVisible('attribute')->toArray();

<a name="appending-values-to-json"></a>
## Thêm giá trị vào JSON

Bạn cũng có thể thêm vào thuộc tính mà không có trường lưu trong database. Để làm thế, đầu tiện cần phải khai báo một [accessor](/docs/{{version}}/eloquent-mutators):

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

Khi đã tạo được accessor, them vào tên của attribute vào thuộc tính `$appends` trên model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

Khi mà attribute được thêm vào trong danh sách `$appends`, nó sẽ được thêm vào khi convert thành thành array hay JSON. Attribute trong mảng `appends` cũng sẽ tuần tự theo cấu hình `$visible` và `$hidden` trong model.
