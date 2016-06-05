# Eloquent: Relationships

- [Giới thiệu](#introduction)
- [Định nghĩa các quan hệ](#defining-relationships)
    - [Một - Một](#one-to-one)
    - [Một - Nhiều](#one-to-many)
    - [Nhiều - Nhiều](#many-to-many)
    - [Has Many Through](#has-many-through)
    - [Polymorphic Relations](#polymorphic-relations)
    - [Many To Many Polymorphic Relations](#many-to-many-polymorphic-relations)
- [Querying Relations](#querying-relations)
    - [Eager Loading](#eager-loading)
    - [Constraining Eager Loads](#constraining-eager-loads)
    - [Lazy Eager Loading](#lazy-eager-loading)
- [Inserting Related Models](#inserting-related-models)
    - [Many To Many Relationships](#inserting-many-to-many-relationships)
    - [Touching Parent Timestamps](#touching-parent-timestamps)

<a name="introduction"></a>
## Giới thiệu

Các bảng trong cơ sở dữ liệu thường có liên quan tới một bảng khác. Ví dụ một blog có thể có nhiều comment, hay một đơn hàng sẽ phải có thông tin liên quan của người dùng mà đã đặt nó. Eloquent giúp cho quản lý và làm việc với những quan hệ này một cách đơn giản và hỗ trợ nhiều kiểu quan hệ.

- [Một - Một](#one-to-one)
- [Một - Nhiều](#one-to-many)
- [Nhiều - Nhiều](#many-to-many)
- [Has Many Through](#has-many-through)
- [Polymorphic Relations](#polymorphic-relations)
- [Many To Many Polymorphic Relations](#many-to-many-polymorphic-relations)

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

The "has-many-through" relationship provides a convenient short-cut for accessing distant relations via an intermediate relation. For example, a `Country` model might have many `Post` models through an intermediate `User` model. In this example, you could easily gather all blog posts for a given country. Let's look at the tables required to define this relationship:

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

Though `posts` does not contain a `country_id` column, the `hasManyThrough` relation provides access to a country's posts via `$country->posts`. To perform this query, Eloquent inspects the `country_id` on the intermediate `users` table. After finding the matching user IDs, they are used to query the `posts` table.

Now that we have examined the table structure for the relationship, let's define it on the `Country` model:

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

The first argument passed to the `hasManyThrough` method is the name of the final model we wish to access, while the second argument is the name of the intermediate model.

Typical Eloquent foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the third and fourth arguments to the `hasManyThrough` method. The third argument is the name of the foreign key on the intermediate model, the fourth argument is the name of the foreign key on the final model, and the fifth argument is the local key:

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
### Polymorphic Relations

#### Table Structure

Polymorphic relations allow a model to belong to more than one other model on a single association. For example, imagine users of your application can "like" both posts and comments. Using polymorphic relationships, you can use a single `likes` table for both of these scenarios. First, let's examine the table structure required to build this relationship:

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

Two important columns to note are the `likeable_id` and `likeable_type` columns on the `likes` table. The `likeable_id` column will contain the ID value of the post or comment, while the `likeable_type` column will contain the class name of the owning model. The `likeable_type` column is how the ORM determines which "type" of owning model to return when accessing the `likeable` relation.

#### Model Structure

Next, let's examine the model definitions needed to build this relationship:

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

#### Retrieving Polymorphic Relations

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the likes for a post, we can simply use the `likes` dynamic property:

    $post = App\Post::find(1);

    foreach ($post->likes as $like) {
        //
    }

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphTo`. In our case, that is the `likeable` method on the `Like` model. So, we will access that method as a dynamic property:

    $like = App\Like::find(1);

    $likeable = $like->likeable;

The `likeable` relation on the `Like` model will return either a `Post` or `Comment` instance, depending on which type of model owns the like.

#### Custom Polymorphic Types

By default, Laravel will use the fully qualified class name to store the type of the related model. For instance, given the example above where a `Like` may belong to a `Post` or a `Comment`, the default `likable_type` would be either `App\Post` or `App\Comment`, respectively. However, you may wish to decouple your database from your application's internal structure. In that case, you may define a relationship "morph map" to instruct Eloquent to use the table name associated with each model instead of the class name:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        App\Post::class,
        App\Comment::class,
    ]);

Or, you may specify a custom string to associate with each model:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => App\Post::class,
        'likes' => App\Like::class,
    ]);

You may register the `morphMap` in the `boot` function of your `AppServiceProvider` or create a separate service provider if you wish.

<a name="many-to-many-polymorphic-relations"></a>
### Many To Many Polymorphic Relations

#### Table Structure

In addition to traditional polymorphic relations, you may also define "many-to-many" polymorphic relations. For example, a blog `Post` and `Video` model could share a polymorphic relation to a `Tag` model. Using a many-to-many polymorphic relation allows you to have a single list of unique tags that are shared across blog posts and videos. First, let's examine the table structure:

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

#### Model Structure

Next, we're ready to define the relationships on the model. The `Post` and `Video` models will both have a `tags` method that calls the `morphToMany` method on the base Eloquent class:

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

Next, on the `Tag` model, you should define a method for each of its related models. So, for this example, we will define a `posts` method and a `videos` method:

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

#### Retrieving The Relationship

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the tags for a post, you can simply use the `tags` dynamic property:

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphedByMany`. In our case, that is the `posts` or `videos` methods on the `Tag` model. So, you will access those methods as dynamic properties:

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## Querying Relations

Since all types of Eloquent relationships are defined via functions, you may call those functions to obtain an instance of the relationship without actually executing the relationship queries. In addition, all types of Eloquent relationships also serve as [query builders](/docs/{{version}}/queries), allowing you to continue to chain constraints onto the relationship query before finally executing the SQL against your database.

For example, imagine a blog system in which a `User` model has many associated `Post` models:

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

You may query the `posts` relationship and add additional constraints to the relationship like so:

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

Note that you are able to use any of the [query builder](/docs/{{version}}/queries) methods on the relationship!

#### Relationship Methods Vs. Dynamic Properties

If you do not need to add additional constraints to an Eloquent relationship query, you may simply access the relationship as if it were a property. For example, continuing to use our `User` and `Post` example models, we may access all of a user's posts like so:

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Dynamic properties are "lazy loading", meaning they will only load their relationship data when you actually access them. Because of this, developers often use [eager loading](#eager-loading) to pre-load relationships they know will be accessed after loading the model. Eager loading provides a significant reduction in SQL queries that must be executed to load a model's relations.

#### Querying Relationship Existence

When accessing the records for a model, you may wish to limit your results based on the existence of a relationship. For example, imagine you want to retrieve all blog posts that have at least one comment. To do so, you may pass the name of the relationship to the `has` method:

    // Retrieve all posts that have at least one comment...
    $posts = App\Post::has('comments')->get();

You may also specify an operator and count to further customize the query:

    // Retrieve all posts that have three or more comments...
    $posts = Post::has('comments', '>=', 3)->get();

Nested `has` statements may also be constructed using "dot" notation. For example, you may retrieve all posts that have at least one comment and vote:

    // Retrieve all posts that have at least one comment with votes...
    $posts = Post::has('comments.votes')->get();

If you need even more power, you may use the `whereHas` and `orWhereHas` methods to put "where" conditions on your `has` queries. These methods allow you to add customized constraints to a relationship constraint, such as checking the content of a comment:

    // Retrieve all posts with at least one comment containing words like foo%
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

#### Counting Relationship Results

If you want to count the number of results from a relationship without actually loading them you may use the `withCount` method, which will place a `{relation}-count` column on your resulting models. For example:

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

You may add retrieve the "counts" for multiple relations as well as add constraints to the queries:

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

<a name="eager-loading"></a>
### Eager Loading

When accessing Eloquent relationships as properties, the relationship data is "lazy loaded". This means the relationship data is not actually loaded until you first access the property. However, Eloquent can "eager load" relationships at the time you query the parent model. Eager loading alleviates the N + 1 query problem. To illustrate the N + 1 query problem, consider a `Book` model that is related to `Author`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

Now, let's retrieve all books and their authors:

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries: 1 for the original book, and 25 additional queries to retrieve the author of each book.

Thankfully, we can use eager loading to reduce this operation to just 2 queries. When querying, you may specify which relationships should be eager loaded using the `with` method:

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

For this operation, only two queries will be executed:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager Loading Multiple Relationships

Sometimes you may need to eager load several different relationships in a single operation. To do so, just pass additional arguments to the `with` method:

    $books = App\Book::with('author', 'publisher')->get();

#### Nested Eager Loading

To eager load nested relationships, you may use "dot" syntax. For example, let's eager load all of the book's authors and all of the author's personal contacts in one Eloquent statement:

    $books = App\Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### Constraining Eager Loads

Sometimes you may wish to eager load a relationship, but also specify additional query constraints for the eager loading query. Here's an example:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');

    }])->get();

In this example, Eloquent will only eager load posts where the post's `title` column contains the word `first`. Of course, you may call other [query builder](/docs/{{version}}/queries) methods to further customize the eager loading operation:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');

    }])->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Sometimes you may need to eager load a relationship after the parent model has already been retrieved. For example, this may be useful if you need to dynamically decide whether to load related models:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

If you need to set additional query constraints on the eager loading query, you may pass a `Closure` to the `load` method:

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-related-models"></a>
## Inserting Related Models

#### The Save Method

Eloquent provides convenient methods for adding new models to relationships. For example, perhaps you need to insert a new `Comment` for a `Post` model. Instead of manually setting the `post_id` attribute on the `Comment`, you may insert the `Comment` directly from the relationship's `save` method:

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

Notice that we did not access the `comments` relationship as a dynamic property. Instead, we called the `comments` method to obtain an instance of the relationship. The `save` method will automatically add the appropriate `post_id` value to the new `Comment` model.

If you need to save multiple related models, you may use the `saveMany` method:

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

#### Save & Many To Many Relationships

When working with a many-to-many relationship, the `save` method accepts an array of additional intermediate table attributes as its second argument:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### The Create Method

In addition to the `save` and `saveMany` methods, you may also use the `create` method, which accepts an array of attributes, creates a model, and inserts it into the database. Again, the difference between `save` and `create` is that `save` accepts a full Eloquent model instance while `create` accepts a plain PHP `array`:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

Before using the `create` method, be sure to review the documentation on attribute [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

<a name="updating-belongs-to-relationships"></a>
#### Updating "Belongs To" Relationships

When updating a `belongsTo` relationship, you may use the `associate` method. This method will set the foreign key on the child model:

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

When removing a `belongsTo` relationship, you may use the `dissociate` method. This method will reset the foreign key as well as the relation on the child model:

    $user->account()->dissociate();

    $user->save();

<a name="inserting-many-to-many-relationships"></a>
### Many To Many Relationships

#### Attaching / Detaching

When working with many-to-many relationships, Eloquent provides a few additional helper methods to make working with related models more convenient. For example, let's imagine a user can have many roles and a role can have many users. To attach a role to a user by inserting a record in the intermediate table that joins the models, use the `attach` method:

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

When attaching a relationship to a model, you may also pass an array of additional data to be inserted into the intermediate table:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Of course, sometimes it may be necessary to remove a role from a user. To remove a many-to-many relationship record, use the `detach` method. The `detach` method will remove the appropriate record out of the intermediate table; however, both models will remain in the database:

    // Detach a single role from the user...
    $user->roles()->detach($roleId);

    // Detach all roles from the user...
    $user->roles()->detach();

For convenience, `attach` and `detach` also accept arrays of IDs as input:

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### Updating A Record On A Pivot Table

If you need to update an existing row in your pivot table, you may use `updateExistingPivot` method:

    $user = App\User::find(1);

	$user->roles()->updateExistingPivot($roleId, $attributes);

#### Syncing For Convenience

You may also use the `sync` method to construct many-to-many associations. The `sync` method accepts an array of IDs to place on the intermediate table. Any IDs that are not in the given array will be removed from the intermediate table. So, after this operation is complete, only the IDs in the array will exist in the intermediate table:

    $user->roles()->sync([1, 2, 3]);

You may also pass additional intermediate table values with the IDs:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

<a name="touching-parent-timestamps"></a>
### Touching Parent Timestamps

When a model `belongsTo` or `belongsToMany` another model, such as a `Comment` which belongs to a `Post`, it is sometimes helpful to update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically "touch" the `updated_at` timestamp of the owning `Post`. Eloquent makes it easy. Just add a `touches` property containing the names of the relationships to the child model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
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

Now, when you update a `Comment`, the owning `Post` will have its `updated_at` column updated as well:

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
