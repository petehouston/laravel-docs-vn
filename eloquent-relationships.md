# Eloquent: Relationships

- [Giới thiệu](#introduction)
- [Định nghĩa các quan hệ](#defining-relationships)
    - [Một - Một](#one-to-one)
    - [Một - Nhiều](#one-to-many)
    - [Nhiều - Nhiều](#many-to-many)
    - [Has Many Through](#has-many-through)
    - [Quan hệ đa hình](#polymorphic-relations)
    - [Quan hệ đa hình nhiều nhiều](#many-to-many-polymorphic-relations)
- [Truy vấn quan hệ](#querying-relations)
    - [Eager Loading](#eager-loading)
    - [Constraining Eager Loads](#constraining-eager-loads)
    - [Lazy Eager Loading](#lazy-eager-loading)
- [Chèn các model liên quan](#inserting-related-models)
    - [Quan hệ nhiều nhiều](#inserting-many-to-many-relationships)
    - [Touching Parent Timestamps](#touching-parent-timestamps)

<a name="introduction"></a>
## Giới thiệu

Các bảng trong cơ sở dữ liệu thường có liên quan tới một bảng khác. Ví dụ một blog có thể có nhiều comment, hay một đơn hàng sẽ phải có thông tin liên quan của người dùng mà đã đặt nó. Eloquent giúp cho quản lý và làm việc với những quan hệ này một cách đơn giản và hỗ trợ nhiều kiểu quan hệ.

- [Một - Một](#one-to-one)
- [Một - Nhiều](#one-to-many)
- [Nhiều - Nhiều](#many-to-many)
- [Has Many Through](#has-many-through)
- [Quan hệ đa hình](#polymorphic-relations)
- [Quan hệ đa hình nhiều nhiều](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## Định nghĩa các quan hệ

Các quan hệ trong Eloquent được định nghĩa như các hàm trong class model. Từ đó, giống như là Eloquent mô tả chính mình, các quan hệ cũng phục vụ rất mạnh mẽ bởi [query builders](/docs/{{version}}/queries), định nghĩa các quan hệ như các hàm cũng cấp các ràng buộc mạnh mẽ và khả năng truy vấn. Ví dụ:

    $user->posts()->where('active', 1)->get();

Nhưng trước khi đi sâu vào việc sử dụng các mối quan hệ, hãy học cách định nghĩa mỗi loại:

<a name="one-to-one"></a>
### Một - Một


Quan hệ một - một một quan hệ đơn giản. Ví dụ, một `User` model có thể liên quan với một `Phone`. Để định nghĩa mối quan hệ này, chúng ta tạo một phương thức `phone` trong model `User`. phương thức `phone` sẽ trả về kết quả của phương thức `hasOne` dựa trên lớp Eloquent model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

Tham số đầu tiên truyền vào phương thức `hasOne` là tên của model liên quan. Một khi quan hệ đã được định nghĩa, chúng ta có thể truy xuất bản ghi liên quan bằng cách sử dụng các thuộc tính động của Eloquent. Các thuộc tính này cho phép bạn truy cập các hàm về mối quan hệ  như là các thuộc tính đã được định nghĩa trong model:

    $phone = User::find(1)->phone;

Eloquent giả sử khóa ngoại của quan hệ dựa trên tên model. Trong trường hợp này, `Phone` model sẽ tự động được giả sử có một khóa ngoại tên `user_id`. Nếu bạn muốn ghi đè quy tắc này, bạn có thể truyền vào một tham số thứ 2 vào phương thức `hasOne`:

    return $this->hasOne('App\Phone', 'foreign_key');

Thêm vào đó Eloquent giả sử rằng khóa ngoại có 1 giá trị tương ứng với cột `id` (hay khóa chính do bạn đặt) của bảng chứa khóa chính. Hay nói một cách khác, Eloquent sẽ tìm kiếm giá trị của user `id` trong cột `user_id` của bản ghi `Phone`. Nếu bạn muốn sử dụng một cột tên khác `id`, bạn sẽ phải truyền vào phương thức `hasOne` một tham số thứ 3 để chị định khóa chính này:

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### Định nghĩa đảo ngược của quan hệ

Chúng ta có thể truy cập model `Phone` từ model `User`. Bây giờ hãy định nghĩa một quan hệ trên model `Phone` để truy cập `User` có số điện thoại nào đó. Chúng ta có thể định nghĩa việc này bằng cách sử dụng phương thức `belongsTo` cho model `Phone`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

Trong ví dụ trên, Eloquent sẽ có gắng so sánh `user_id` của model `Phone` với `id` ở model `User`. Eloquent xác định tên khóa ngoại mặc định bằng tên của bảng chứa khóa chính kèm theo sau là `_id`. Tuy nhiên nếu khóa ngoại model `Phone` không phải là `user_id`, bạn phải truyền tên khóa ngoại này như là tham số thứ 2 của phương thức `belongsTo`:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

Nếu `User` model ông sử dụng `id` là khóa chính, hay bạn muốn join với một cột khác (câu lệnh JOIN), bạn phải truyền vào tham số thứ 3 để chỉ định khóa chính này:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="one-to-many"></a>
### Một - Nhiều

Quan hệ "một - nhiều" là quan hệ mà một model có nhiều model khác. Ví dụ một bài viết có vố số các bình luận. Giống như các mối quan hệ khác trong Eloquent, quan hệ này được định nghĩa bằng một hàm trong model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

Hãy nhớ, Eloquent sẽ tự động xác định cột khóa ngoại trên model `Comment` với định dạng "snake case" là tên của model chứa khóa chính + `_id`, trong trường hợp này khóa ngoại sẽ là `post_id`

Khi quan hệ đã được định nghĩa, chúng ta có thể truy cập danh sách comment bằng cách sử dụng thuộc tính `comments`. Hãy nhớ, Eloquent cung cấp các "thuộc tính động", chúng ta có thể truy cập các hàm quan hệ nếu như chúng được định nghĩa như thuộc tính của model:

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

Tất nhiên, tất cả các quan hệ cũng được dùng như query builders, bạn có thể thêm các ràng buộc khác cho những comments bằng cách gọi phương thức `comments` và tiếp tục thêm các điều kiện vào truy vấn:

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

Giống như phương thức `hasOne`, bạn cũng có thể định nghĩa khóa chính và khóa ngoại riêng cho mình bằng cách truyền tham số vào phương thức `hasMany`:

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

#### Định nghĩa đảo ngược của quan hệ

Chúng ta bây giờ đã có thể truy cập toàn bộ comment của bài post trên blog, tiếp theo hãy định nghĩa một quan hệ để cho phép comment có thể truy cập vào bài viết, chúng ta sẽ dùng phương thức `belongsTo`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Khi quan hệ đã được định nghĩa, chúng ta có thể lấy thông tin model `Post` cho một `Comment` bằng cách truy cập vào thuộc tính động `post`


    $comment = App\Comment::find(1);

    echo $comment->post->title;

Trong ví dụ trên, Eloquent sẽ cố gắng đối chiếu `post_id` từ model `Comments` đến `id` của model `Post`. Eloquent mặc định định nghĩa tên khóa ngoại bằng tên của phương thức quan hệ với đuôi `_id`. Tuy nhiên nếu bạn muốn tùy chọn một tên khác, hãy truyền nó vào như là tham số thứ 2 vào phương thức `belongsTo`:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

Nếu model của bạn không sử dụng `id` làm khóa chính, hoặc bạn muốn join với model con ở một cột khác, bạn có thể truyền nó vào như là tham số thứ 3 trong phương thức `belongsTo` để chỉ định key của bảng cha

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### Nhiều - Nhiều

Quan hệ nhiều nhiều có hơi phức tạp hơn quan hệ `hasOne` và `hasMany` một chút. Ví dụ như là mối quan hệ của 1 user với nhiều "role" (vai trò, quyền, kiểu như admin, mod,...), khi mà các role cũng được đảm nhận bởi nhiều user. Cụ thể hơn, nhiều user có thể có cùng role "Admin". Để định nghĩa mối quan hệ này, cần đến 3 bảng: `users`, `roles`, `role_user`. Bảng `role_user` xuất phát từ tên của những bảng hay model liên quan, và bao gồm các cột `user_id` và `role_id`.

Quan hệ nhiều nhiều được định nghĩa bởi phương thức `belongsToMany`. Ví dụ sau sẽ định nghĩa phương thức `roles` trong model `User`:


    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

Khi quan hệ được định nghĩa, bạn có thể truy cập vào các role của user thông quan thuộc tính `roles`:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

Tất nhiên cũng giống như tất cả các kiểu quan hệ khác, bạn cũng có thể gọi phương thức `roles` và thêm vào nó các ràng buộc:

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

Như đã nhắc đến từ trước, để xác định tên của table để join quan hệ này (ở đây là table `role_user`), Eloquent sẽ join 2 model tên liên quan theo thứ tự alphabetical. Tuy nhiên bạn có thể đặt tên tùy chọn bằng cách truyền vào tham số thứ 2 trong phương thức `belongsToMany`:

    return $this->belongsToMany('App\Role', 'role_user');

Để tùy chọn lựa chọn cột để join, bạn có thể truyền thêm tham số vào hàm `belongsToMany` với tham số thứ 3 là tên của khóa ngoại ứng với model định nghĩa quan hệ (ở đây là `User`) và tham số thứ 4 là tên cột của khóa ngoại ứng với model tham chiếu đến (ở đây là `Role`). 2 tham số `user_id` và `role_id` là 2 cột khóa ngoại của table `role_user`:

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### Defining The Inverse Of The Relationship

Để truy ngược lại của quan hệ nhiều nhiều, bạn chỉ đơn giản đặt hàm `belongsToMany` trong model được liên quan. Tiếp tục với ví dụ về user và role, hãy định nghĩa phương thức `users` trong model `Role`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

Bạn có thể thấy, quan hệ được định nghĩa giống hết như bên `User`, với khác biệt duy nhất là chúng ta tham chiếu tới model `App\User`. Khi chúng ta sử dụng lại phương thức `belongsToMany`, tất cả các bảng và khóa tùy chọn đều có sẵn khi định nghĩa "the inverse" (có thể hiểu như truy xuất ngược) của quan hệ nhiều - nhiều

#### Lấy các cột của bảng trung gian

Như chúng ta đã biết, khi làm việc với quan hệ nhiều nhiều, ta cần tới 1 bảng trung gian. Eloquent cung cấp nhiều cách rất hữu ích để tương tác với bảng này. Ví dụ hãy giả sử đối tượng `User` có nhiều đối tượng `Role` mà nó liên quan đến. Sau khi truy cập vào quan hệ này, chúng ta có thể muốn lấy thông tin bảng trung gian bằng cách sử dụng thuộc tính `pivot` trên model:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Chú ý rằng mỗi model `Role` chúng ta lấy ra sẽ được tự động gán cho một thuộc tính `pivot`. Thuộc tính này bao gồm 1 model đại diện cho bảng trung gian, và có thể được sử dụng dung như bất kì model Eloquent nào.

Mặc định, chỉ có các khóa của model tồn tại trong đối tượng `pivot`. Nếu bảng pivot của bạn có nhiều thuộc tính hơn, bạn phải chỉ định chúng khi định nghĩa quan hệ:

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

Nếu bạn muốn bảng pivot này cũng có các automatically maintained `created-at` và `updated_at` timestamps (đây là 2 trường lưu lại thời gian khi thực hiện query trên 1 field), sử dụng phương thức `withTimestamps` khi định nghĩa quan hệ.

    return $this->belongsToMany('App\Role')->withTimestamps();

#### Lọc quan hệ thông qua các cột của bảng trung gian

Bạn cũng có thể lọc các kết quả trả về bởi `belongsToMany` bằng cách sử dụng phương thức `wherePivot` và `wherePivotIn`:

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('approved', [1, 2]);

<a name="has-many-through"></a>
### Has Many Through

(*lời người dịch, mình ko biết dịch cái này sang tiếng Việt sẽ như nào nữa, đơn giản là có nhiều thông qua :v)
Quan hệ "has-many-through" cung cấp cho ta một "đường tăt" tiện lợi cho việc truy cập những sự quan hệ "xa" thông qua một quan hệ trung gian. Ví dụ model `Country` có thể có nhiều model `Post` thông qua model `User`. Trong ví dụ này, bạn có thể dễ dàng nhóm tất cả các bài viết blog cho một country đã cho. Hãy nhìn vào các bảng cần thiết để định nghĩa mối quan hệ này:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

Bạn có thể thấy `posts` không có cột `country_id`, sự quan hệ `hasManyThrough` cung cấp cho chúng ta tuy cập tới các country's post qua `$country-posts`. Để thực hiện truy vấn này, Eloquent xem xét `country_id` trên bảng trung gian `users`. Sau khi tìm thấy các user ID phù hợp, chúng sẽ được dùng để truy vấn bảng `posts`.

Chúng ta đã vừa xem qua cấu trúc bảng cho mối quan hệ này, bây giờ sẽ định nghĩa nó trong model `Country`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

Tham số đầu tiên truyền vào phương thức `hasManyThrough` là tên của model cuối nơi mà chúng ta muốn lấy data, còn tham số thứ 2 là tên của model trung gian. (*Không biết có trung gian qua nhiều table dc không nhỉ :v).

Thông thường các khóa ngoại Eloquent sẽ được sử dụng để thực hiện các truy vấn quan hệ. Nếu bạn muốn tùy chỉnh tên các khóa, bạn phải truyền chúng vào như là tham số thứ 3 và thứ 4 cho phương thức `hasManyThrough`. Tham số thứ 3 là tên của khóa ngoại trên model trung gian, tham số thứ 4 là tên của khóa ngoại trên model cuối và tham số thứ 5 là local key (key của model).

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post', 'App\User',
                'country_id', 'user_id', 'id'
            );
        }
    }

<a name="polymorphic-relations"></a>
### Quan hệ đa hình

#### Cấu trúc bảng

Quan hệ đa hình cho phép một model co thể thuộc về nhiều hơn 1 model khác với một sự liên kết đơn. Ví dụ, tưởng tượng người dùng của ứng dụng có thể "like" cả post và comment. Sử dụng mối quan hệ đa hình, bạn có thể sử dụng duy nhất một bảng `likes` cho cả 2 ngữ cảnh trên. Đầu tiên hãy xem qua về điều kiện cấu trúc bảng để tạo nên quan hệ này:

    posts
        id - integer
        title - string
        body - text

    comments
        id - integer
        post_id - integer
        body - text

    likes
        id - integer
        likeable_id - integer
        likeable_type - string

 Có hai cột quan trong cần ghi nhớ đó là `likeable_id` và `likeable_type` trên table `likes`. Cột `likeable_id` sẽ lưu giữ giá trị ID của post hoặc comment, trong khi đó cột `likeable_type` sẽ lưu giữ tên lớp của model sở hữu. Cột `likeablel_type` là cách mà ORM xác định "kiểu" của model sở hữu để trả về khi truy cập vào quan hệ `likeable`.

#### Cấu trúc Model

Tiếp theo, hãy xem xét các định nghĩa cần thiết cho model để tạo nên quan hệ này:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Like extends Model
    {
        /**
         * Get all of the owning likeable models.
         */
        public function likeable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get all of the post's likes.
         */
        public function likes()
        {
            return $this->morphMany('App\Like', 'likeable');
        }
    }

    class Comment extends Model
    {
        /**
         * Get all of the comment's likes.
         */
        public function likes()
        {
            return $this->morphMany('App\Like', 'likeable');
        }
    }

#### Lấy thông tin từ quan hệ đa hình

Một khi các các bảng dữ liệu và model được định nghĩa, bạn có thể truy cập vào quan hệ thông qua các model. Ví dụ để truy cập tất cả like của một post, chúng ta đơn giản chỉ cần sử dụng thuộc tính động `likes`:

    $post = App\Post::find(1);

    foreach ($post->likes as $like) {
        //
    }

Bạn cũng có thể lấy chính chủ của một quan hệ đa hình từ model đa hình bằng cách truy câp tên của phương thức mà gọi tới `morphTo`. Trong trường hợp này, đó là phương thức `likeable` trên model `Like`. Vì vậy chúng ta sẽ truy cập phương thức đó như là một thuộc tính động:

    $like = App\Like::find(1);

    $likeable = $like->likeable;

Quan hệ `likeable` trên model `Like` sẽ trả về 1 instance hoặc là `Post` hoặc `Comment`, dựa trên kiểu của model sở hữu like.

#### Tùy chỉnh các kiểu đa hình

Mặc định, Laravel sẽ sử dụng tên lớp đầy đủ để lưu giữ kiểu của model được liên quan. Cho một thể hiện(instance), đã có ở ví dụ trên nơi mà một `Like` có thể thuộc về một `Post` hoặc một `Comment`, mặc định `likeable_type` sẽ hoặc là `App\Post` hoặc `App\Comment` tương ứng. Tuy nhiên, bạn có thể muốn tách database từ cấu trúc bên trong của ứng dụng của bạn. Trong trường hợp đó, bạn phải định nghĩa một quan hệ "morph map" để thông báo cho Eloquent sử dụng tên bảng liên quan với mỗi model thay vì sử dụng tên class đầy đủ:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        App\Post::class,
        App\Comment::class,
    ]);

Hoặc bạn muốn chỉ đinh một chuỗi tùy chọn liên quan với mỗi model:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => App\Post::class,
        'likes' => App\Like::class,
    ]);

Bạn phải đăng kí `morphMap` trong hàm `boot` của `AppServiceProvider` hoặc tạo 1 service provider tách biệt nếu bạn muốn.

<a name="many-to-many-polymorphic-relations"></a>
### Quan hệ đa hình nhiều - nhiều

#### Cấu trúc bảng

Thêm vào mối quan hệ đa hình truyền thống, bạn cũng có thể định nghĩa quan hệ đa hình nhiều - nhiều. Ví dụ 1 blog model `Post` và `Video` có thể chia sẻ 1 liên kết đa hình tới model `Tag`. Sử dụng quan hệ đa hình nhiều - nhiều cho phép bạn có một danh sách các tag không trùng lặp mà được chia sẻ qua các blog post và video. Đầu tiên, hãy xem về cấu trúc bảng:

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### Cấu trúc Model

Tiếp theo, chúng ta sẽ định nghĩa các mối quan hệ trên model. Model `Post` và `Video` sẽ đều có một phương thức `tags` gọi tới phương thức `morphToMany` trên lớp Eloquent:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### Defining The Inverse Of The Relationship

Tiếp theo, trên model `Tag`, bạn nên định nghĩa một phương thức cho mỗi model liên quan tới nó. Vì vậy trong ví dụ này, chúng ta sẽ định nghĩa 1 phương thức `posts` và `videos`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### Lấy dữ liệu quan hệ

Khi các bảng và model đã được định nghĩa, bạn có thể truy cập vào các quan hệ qua model. Ví dụ để lấy toàn bộ các tag cho một post, bạn đơn giản chỉ cần sử dụng thuộc tính động `tags`:

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

Bạn cũng có thể lấy chủ thể của một quan hệ đa hình từ model đa hình bằng cách truy cập tên của phương thức mà thực hiện gọi tới `morphedByMany`. Trong trường hợp của chúng ta đó là phương thức `posts` hoặc `videos` trên model `Tag`. Vì vậy bạn sẽ sử dụng những phương thức này như là các thuộc tính động:

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## Các quan hệ truy vấn

Khi tất cả các kiểu của quan hệ Eloquent được định nghĩa qua các hàm, bạn có thể gọi những hàm này để lấy những instance của quan hệ mà không cần thực thi các truy vấn quan hệ. Thêm vào đó, tất cả kiểu của Eloquent relationships cũng có sẵn tại [query builders](/docs/{{version}}/queries), cho phép bạn tiếp tục gắn thêm các ràng buộc on truy vấn trước khi được thực thi SQL đến database.

Ví dụ, hãy tưởng tượng 1 blog mà một model `User` có nhiều model `Post` liên quan:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

Bạn có thể truy vấn quan hệ `posts` và thêm các ràng buộc vào như sau:

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

Ghi nhớ rằng bạn có thể sử dụng bất kì phương thức của [query builder](/docs/{{version}}/queries) trên mối quan hệ!

#### Phương thức quan hệ Vs. Thuộc tính động

Nếu bạn không cần thêm các ràng buộc vào truy vấn Eloquent relationship, bạn có thể đơn giản chỉ cần truy cập vào quan hệ nếu như nó là 1 thuộc tính. Tiếp tục ví dụ sử dụng model `User` và `Post` của chúng ta, có thể truy cập toàn bộ bài viết của 1 user như sau:

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Các thuộc tính động là "lazy loading", nghĩa là chúng sẽ chỉ load dữ liệu quan hệ của chúng khi bạn thực sự gọi. Bởi vì điều này, lập trình viên thường sử dụng [eager loading](#eager-loading) để pre-load các quan hệ mà họ biết sẽ được xử lý sau khi load model. Eager loading cung cấp một sự giảm bớt đáng kể trong các truy vấn SQL mà sẽ phải thực thi khi load các quan hệ của model.

#### Querying Relationship Existence

Khi truy cập các bản ghi một model, bạn có thể muốn giới hạn kết quả trả về dựa trên sự tồn tại của một quan hệ. Ví dụ, tưởng tượng bạn muốn lấy tất cả các post của blog mà có ít nhật một comment. Để làm việc này, bạn phải truyền tên của quan hệ vào phương thức `has`:

    // lấy toàn bộ các post có ít nhất 1 comment...
    $posts = App\Post::has('comments')->get();

Bạn cũng có thể chỉ định thêm toán tử và số lượng vào truy vấn:

    // Lấy tất cả các post có nhiều hơn hoặc bằng 3 comment...
    $posts = Post::has('comments', '>=', 3)->get();

Nested `has` statements may also be constructed using "dot" notation. For example, you may retrieve all posts that have at least one comment and vote:
Cú pháp `has` lồng cũng có thể sử dụng kí hiệu "dot". Ví dụ bạn muốn lấy tất cả các post có tối thiểu 1 comment và 1 vote:

    // lấy tất cả các post có tối thiểu 1 comment và vote...
    $posts = Post::has('comments.votes')->get();

Nếu bạn vấn muốn có những truy vấn mạnh hơn, hãy sử dụng phương thức `whereHas` và `orWhereHas` để đặt các điều kiện "where" bên trong truy vấn `has`. Những phương thức này cho phép bạn thêm những ràng buộc tùy chọn, như là kiểm tra nội dung của 1 comment:

    // Lấy tất cả các post có ít nhật 1 comment bao gồm từ foo% - tức là bắt đầu bằng foo
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

#### Đếm các kết quả của quan hệ

Nếu bạn muốn đếm số kết quả từ 1 quan hệ mà không muốn load chúng bạn có thể sử dụng phương thức `withCount`, nó sẽ đặt một cột `{relation}_count` vào model kết quả. Ví dụ:

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

Bạn có thể lấy "counts" cho nhiều quan hệ cũng giống như thêm ràng buộc vào truy vấn:

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

<a name="eager-loading"></a>
### Eager Loading

Khi bạn truy cập Eloquent relationship như là các thuộc tính, các dữ liệu này là "lazy loaded". Điều này có nghĩa là dữ liệu không thực sự load cho đến khi bạn truy cập lần đầu tiên tới thuộc tính. Tuy nhiên, Eloquent có thể "eager load" các quan hệ tại thời điểm bạn truy vấn và model cha. Eager loading làm giảm bớt vấn đề truy vấn N + 1. Để giải thích vấn đề truy vấn N + 1, ví dụ như model `Book` có liên quan đến `Author`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Lấy tác giả viết sách.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

Bây giờ, lấy toàn bộ sách và các tác giả của chúng:

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Vòng lặp sẽ thực hiện 1 truy vấn để lấy tất cả các sách trong bảng, sau đó là mỗi truy vấn cho 1 cuốn sách để lấy tác giả. Vì vậy nếu chúng ta có 25 quyển sách, vòng lặp sẽ thực hiện 26 truy vấn: 1 để lấy sách, 25 để lấy tác giả của mỗi quyển sách.

Rất may, chúng ta có thể sử dụng eager loading để giảm thiểu tính toán chỉ với 2 truy vấn. Khi thực hiện truy vấn, bạn chỉ định quan hệ này sẽ được eager load bằng cách dùng phương thức `with`:

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Với tính toán này chỉ có 2 truy vấn sẽ được thực thi:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager Loading Multiple Relationships

Đôi khi bạn có thể cần eager load vài mối quan hệ khác nhau trong 1 tính toán. Để làm việc này, chỉ cần truyền thêm các tham số vào phương thưcs `with`:

    $books = App\Book::with('author', 'publisher')->get();

#### Nested Eager Loading

Để lồng các eager load, bạn có thể sử dụng cú pháp "dot". Ví dụ eager load tất cả tác giả sách và tất cả thông tin liên hệ của từng tác giả trong một cú pháp Eloquent:

    $books = App\Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### Ràng buộc các eager load

Đôi khi bạn có thể muộn eager load một quan hệ, nhưng cũng muốn chỉ định những ràng buộc thêm vào trong truy vấn của eager load. Sau đây là 1 ví dụ:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');

    }])->get();

Trong ví dụ này, Eloquent sẽ chỉ eager load các post mà có cột `title` chứa từ `first`. Tất nhiên, bạn có thể gọi các phương thức [query builder](/docs/{{version}}/queries) khác để làm truy vấn linh động hơn:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');

    }])->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Đôi khi bạn muốn eager load một quan hệ sau khi model cha đã được lấy ra. Điều này có thể hữu ích nếu bạn muốn linh động quyết định sẽ load model liên quan nào:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

Nếu bạn muốn thêm các ràng buộc vào truy vấn của eager load, bạn có thể truyền một `Closure` vào phương thức `load`:

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-related-models"></a>
## Chèn các model liên quan

#### Phương thức save

Eloquent cung cấp nhiều phương thức tiện lợi cho việc thêm model vào quan hệ. Ví dụ, không chừng bạn muốn chèn một `Comment` mới cho model `Post`. Thay vì thủ công set thuộc tính `post_id` trên `Comment`, bạn có thể chèn `Comment` trực tiếp từ phương thức `save`:

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

Chú ý rằng chúng ta không truy cập vào quan hệ `comments` như một thuộc tính động. Thay vào đó chúng ta gọi phương thức `comments` để có được một instance của quan hệ. Phương thức `save` sẽ tự động thêm giá trị `post_id` phù hợp vào model `Comment` mới.

Nếu bạn cần lưu nhiều model liên quan, bạn có thể sử dụng phương thức `saveMany`:

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

#### Phương thức save và quan hệ nhiều - nhiều

Khi làm việc với quan hệ nhiều - nhiều, phương thức `save` chập nhận một mảng của các thuộc tính của bảng trung gian như là tham số thứ 2:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Phương thức create

Ngoài phương thức `save` và `saveMany`, bạn cũng có thể sử dụng phương thức `create`, cái mà cho phép một mảng của các thuộc tính, tạo ra 1 model, và chèn nó vào trong database. Một lần nữa, sự khác nhau giữa `save` và `create` đó là `save` chấp nhận một Eloquent model instance (một biến đã được gán giá trị) trong khi `create` chấp nhận một PHP `array`:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

Before using the `create` method, be sure to review the documentation on attribute [mass assignment](/docs/{{version}}/eloquent#mass-assignment).
Trước khi sử dụng phương thức `create`, hãy chắc chắn bạn đã xem qua doc [mass assignment](/docs/{{version}}/eloquent#mass-assignment). 

<a name="updating-belongs-to-relationships"></a>
#### Cập nhật quan hệ "Belongs To"

Khi bạn cập nhật một quan hệ `belongsTo`, bạn có thể sử dụng phương thức `associate`. Phương thức này sẽ thiết lập khóa ngoại trên model con.

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

Khi xóa quan hệ `belongsTo`, bạn có thể sử dụng phương thức `dissociate`. Phương thức này sẽ thiết lập lại khóa ngoại cũng như quan hệ trên model con:

    $user->account()->dissociate();

    $user->save();

<a name="inserting-many-to-many-relationships"></a>
### Quan hệ nhiều - nhiều

#### Đính kèm / Gỡ bỏ

Khi làm việc với quan hệ nhiều - nhiều, Eloquent cung cấp một số phương thức hữu ích để làm việc với các model có liên quan vô cùng tiện lợi. Ví dụ, tưởng tượng 1 user có thể có nhiều role và 1 role có thể có nhiều user. Để đính kèm 1 role cho 1 user bằng cách chèn thêm 1 bản ghi vào bảng trung gian cái mà join với model, sử dụng phương thức `attach`

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

Khi đính kèm 1 quan hệ vào model, bạn có thể truyền 1 mảng dữ liệu sẽ được chèn vào bảng trung gian:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Tất nhiên, đôi khi có thể là cần thiết khi xóa bỏ 1 role khỏi 1 user. Để xóa bỏ bản ghi quan hệ nhiều - nhiều, sử dụng phương thức `detach`. Phương thức này sẽ xóa bỏ bản ghi phù hợp khỏi bảng trung gian; tuy nhiên, cả hai model vẫn sẽ còn trong database:

    // xóa 1 role khỏi 1 user...
    $user->roles()->detach($roleId);

    // xóa toàn bộ role khỏi 1 user...
    $user->roles()->detach();

For convenience, `attach` and `detach` also accept arrays of IDs as input:
Cho thuận tiện hơn, `attach` và `detach` cũng chập nhận mảng các ID như là đầu vào:

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### Cập nhật bản ghi trên bảng trung gian

Nếu bạn cần cập nhật 1 hàng trên bảng trung gian, sử dụng phương thức `updateExistingPivot`:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);
	

#### Syncing For Convenience

Bạn có thể cũng sử dụng phương thức `sync` để khởi tạo tập hợp nhiều - nhiều. Phương thức `sync` chấp nhận 1 mảng ID để đưa vào bảng trung gian. Bất kì ID nào không ở trong mảng này sẽ bị xóa khỏi bảng trung gian. Vì vậy sau khi tính toán hoàn thành, chỉ có những id trong mảng sẽ tồn tại trong bảng trung gian:

    $user->roles()->sync([1, 2, 3]);

Bạn cũng có thể truyền thêm các giá trị cho bảng trung gian cùng với ID:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

<a name="touching-parent-timestamps"></a>
### Touching Parent Timestamps

Khi một model `belongsTo` hoặc `belongsToMany` những model khác, như là `Comment` sẽ thuộc về 1 `Post`, đôi khi rất hữu ích khi cập nhật lại timestamp của model cha khi model con được cập nhật. Ví dụ, khi 1 model `Comment` được cập nhật, bạn có thể muốn tự động "chạm" tới timestamp `update_at` của `Post`. Eloquent làm việc này dễ dàng. Chỉ cần thêm thuộc tính `touches` chứa tên của quan hệ đến model con:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Tất cả quan hệ sẽ được chạm tới :D.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Bây giờ, khi bạn cập nhật 1 `Comment`, model `Post` sở hữu nó sẽ được cập nhật tại cột `update_at`:

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();