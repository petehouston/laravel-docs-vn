# Authorization

- [Giới thiệu](#introduction)
- [Định Nghĩa Abilities](#defining-abilities)
- [Kiểm Tra Abilities](#checking-abilities)
    - [Thông Qua Gate Facade](#via-the-gate-facade)
    - [Thông Qua The User Model](#via-the-user-model)
    - [Trong Blade Templates](#within-blade-templates)
    - [Trong Form Requests](#within-form-requests)
- [Policies](#policies)
    - [Tạo Policies](#creating-policies)
    - [Viết Policies](#writing-policies)
    - [Kiểm Tra Policies](#checking-policies)
- [Controller Authorization](#controller-authorization)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc cung cấp các service [authenticatioin](/docs/{versioin}/authentication), Laravel cũng cung cấp một cách đơn giản để tổ chức các logic cấp quyền và điều khiển việc truy cập vào tài nguyên. Có nhiều methods và helpers hỗ trợ bạn trong việc tổ chức việc cấp quyền của bạn và chúng ta sẽ đi qua từng phần của chúng trong tài liệu này.

<a name="defining-abilities"></a>
## Định nghĩa các Abilities

Cách đơn giản nhất để xác định nếu một user có thể thực hiện một hành động đã cho là định nghĩa ra một "ability" bằng cách sử dụng class `Illuminate\Auth\Access\Gate`. `AuthServiceProvider` cái mà cùng với Laravel phục vụ như một nơi để định nghĩa tất cả các abilities cho ứng dụng của bạn. Ví dụ, hãy định nghĩa một `update-post` ability nhận `User` hiện tại và một `Post` [model](/docs/{versioni}/eloquent). Với ability này, chúng ta sẽ xác định nếu `id` của user trùng với `user_id` của post.

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Auth\Access\Gate as GateContract;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
         * @return void
         */
        public function boot(GateContract $gate)
        {
            $this->registerPolicies($gate);

            $gate->define('update-post', function ($user, $post) {
                return $user->id == $post->user_id;
            });
        }
    }

Chú ý rằng chúng ta đã không kiểm tra nếu `$user` đã cho là không `NULL`. `Gate` sẽ tự động trả về `false` cho **tất cả abilities** khi có một user chưa được xác thực hoặc một user được chỉ định mà không sử dụng `forUser` method.

#### Class Based Abilities

Ngoài ra để đăng kí `Closures` như là authorization callbacks, bạn có thể đăng kí các class methods bằng cách truyền vào một string gồm tên class và method. Khi cần thiết, class sẽ được resolved thông qua [service container](/docs/{{version}}/container):

    $gate->define('update-post', 'Class@method');

<a name="intercepting-all-checks"></a>
<a name="intercepting-authorization-checks"></a>
#### Bỏ Qua Authorization Checks

Đôi khi, bạn có thể muốn cấp toàn bộ abilities cho một user nào đó. Trong trường hợp này, sử dụng method `before` để định nghĩa một callback mà được chạy trước tất cả các authorization checks:

    $gate->before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Nếu callback `before` trả về kết quả non-null, kết quả đó sẽ được đại diện cho kết quả của việc kiểm tra.

Bạn có thể sử dụng method `after` để định nghĩa một callback thực thi sau mỗi authorization check. Tuy nhiên bạn không thể thay đổi kết quả của việc kiểm tra authorization từ `after` callback:

    $gate->after(function ($user, $ability, $result, $arguments) {
        //
    });

<a name="checking-abilities"></a>
## Kiểm Tra Abilities

<a name="via-the-gate-facade"></a>
### Thông Qua Gate Facade

Một khi ability đã được định nghĩa, chúng ta có thể "kiểm tra" nó bằng nhiều cách khác nhau. Đầu tiên, chúng ta có thể sử dụng `check`, `allows` hoặc `denies` methods trong `Gate` [facade](/docs/{{version}}/facades). Tất cả những phương thức này nhận tên của ability và các đối số mà sẽ được truyền vào ability's callback. Bạn **không cần** truyền vào user hiện tại vào các methods này, khi mà `Gate` sẽ tự động thêm user vào trước các đối số được truyền vào callback. Vì vậy khi kiểm tra `update-post` ability chúng ta đã định nghĩa lúc trước, chúng ta chỉ cần truyền một `Post` instance vào `denies` method:

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
            $post = Post::findOrFail($id);

            if (Gate::denies('update-post', $post)) {
                abort(403);
            }

            // Update Post...
        }
    }

Tất nhiên, method `allows` đơn giản là ngược lại của method `denies`, và trả về `true` nếu hành động được cấp quyền. Method `check` là một alias của method `allows`.

#### Kiểm Tra Abilities Cho User Xác Định

Nếu bạn muốn sử dụng `Gate` facade để kiểm tra một user **không phải là user hiện tại đã được xác thực** có quyền nào đó hay không, bạn có thể sử dụng `forUser` method:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        //
    }

#### Truyền Nhiều Đối Số

Tất nhiên, các ability callback có thể nhận nhiều đối số:

    Gate::define('delete-comment', function ($user, $post, $comment) {
        //
    });

Nếu ability của bạn cần nhiều đối số, đơn giản chỉ cần truyền chúng dưới dạng mảng vào trong `Gate` methods:

    if (Gate::allows('delete-comment', [$post, $comment])) {
        //
    }

<a name="via-the-user-model"></a>
### Thông Qua User Model

Ngoaì ra, bạn có thể kiểm tra abilities thông qua instance của `User` model. Mặc định, `App\User` model của Laravel sử dụng `Authorizable` trait cái mà cung cấp cho chúng ta 2 methods: `can` và `cannot`. Những methods này có cách sử dụng tương tự như `allows` và `denies` trong `Gate` facade.
Vì vậy, trong cách ví dụ chúng ta sử dụng trước, có thể thay đổi như sau:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            $post = Post::findOrFail($id);

            if ($request->user()->cannot('update-post', $post)) {
                abort(403);
            }

            // Update Post...
        }
    }

Of course, the `can` method is simply the inverse of the `cannot` method:
Tất nhiên, `can` method chỉ đơn giản là ngược lại của `cannot` method:

    if ($request->user()->can('update-post', $post)) {
        // Update Post...
    }

<a name="within-blade-templates"></a>
### Trong Blade Templates

Để thuận tiện, Laravel cung cấp cho chúng ta `@can` Blade directive để nhanh chóng kiểm tra user đã xác thực hiện tại có ability nào đó hay không. Ví dụ:

    <a href="/post/{{ $post->id }}">View Post</a>

    @can('update-post', $post)
        <a href="/post/{{ $post->id }}/edit">Edit Post</a>
    @endcan

Bạn cũng có thể kết hợp `@can` với `@else`:

    @can('update-post', $post)
        <!-- The Current User Can Update The Post -->
    @else
        <!-- The Current User Can't Update The Post -->
    @endcan

<a name="within-form-requests"></a>
### Trong Form Requests

Bạn cũng có thể chọn việc sử dụng các abilities đã được định nghĩa trong `Gate` từ [form request's](/docs/{{version}}/validation#form-request-validation) `authorize` method. Ví dụ:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $postId = $this->route('post');

        return Gate::allows('update', Post::findOrFail($postId));
    }

<a name="policies"></a>
## Policies

<a name="creating-policies"></a>
### Tạo Policies

Khi mà việc định nghĩa toàn bộ authorization logic trong `AuthServiceProvider` có thể thành trở ngại trong các ứng dụng lớn, Laravel cho phép bạn tách các authorization login thành cách class "Policy". Policies là các class thuần PHP mà nhóm các authorization logic dựa trên tài nguyền chúng cấp quyền.

Đầu tiên, tạo một policy để quản lí việc cấp quyền cho `Post` model. Bạn có thể tạo một policy thông quan `make;policy` [artisan command](/docs/{{version}}/artisan). Policy được tạo ra sẽ ở trong thư mục `app/Policies`:

    php artisan make:policy PostPolicy

#### Đăng Kí Policies

Khi đã có policy, chúng ta cần đăng kí nó với `Gate` class. `AuthServiceProvider` bao gồm một thuộc tính `policies` dùng để map nhiều thực thể policies để quản lý chúng. Vì vậy, chúng ta sẽ chỉ định policy của `Post` model là `PostPolicy` class:

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
         * @return void
         */
        public function boot(GateContract $gate)
        {
            $this->registerPolicies($gate);
        }
    }

<a name="writing-policies"></a>
### Viết Policies

Khi policy được sinh ra và đăng kí, chúng ta cần thêm các method cho mỗi ability mà nó cấp quyền. ví dụ, định nghĩa một `update` method trong `PostPolicy` để xác định `User` có thể "update" `Post` hay không:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Bạn có thể tiếp tục định nghĩa thêm các method vào policy nếu thấy cần thiết. Ví dụ bạn có thể định nghĩa `show`, `destroy` hoặc `addComment` method để cấp quyền cho nhiều hành động `Post`.

> **Ghi chú:** Toàn bộ các policies được mang đến thông qua Laravel [service container](/docs/{{version}}/container), nghĩa là bạn có thể type-hint bất kì các dependencies trong policy constructor và chúng sẽ tự động được injected.

#### Bỏ Qua Bộ Kiểm Tra

Đôi khi, bạn có thể muốn cấp toàn bộ abilities cho một người dùng nào đó. Trong trường hợp này, định nghĩa `before` method trong policy. Method này sẽ trả chạy trước toàn bộ các kiểm tra cấp quyền trong policy:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Nếu `before` method trả về một kết quả non-null thì kết quả đó sẽ được đại diện cho kết quả của kiểm tra.

<a name="checking-policies"></a>
### Kiểm Tra Policies

Các policy methos được gọi chính xác giống như cách `Closure` based authorization callbacks. Bạn có thể sử dụng `Gate` facade, `User` model, `@can` Blade directive hoặc `policy` helper.

#### Thông Qua Gate Facade

`Gate` sẽ tự động xác định policy nào để sử dụng kiểm tra với class của các đối số truyền vào method của nó. Vì vậy, nếu chúng ta truyền một `Post` instance vào `denies` method, `Gate` sẽ tự động lựa chọn đúng `PostPolicy` tương ứng thực hiện các hành động cấp quyền:

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
            $post = Post::findOrFail($id);

            if (Gate::denies('update', $post)) {
                abort(403);
            }

            // Update Post...
        }
    }

#### Thông Qua User Model

Các phương thực `can` và `cannot` của `User` model sẽ tự động sử dụng các policies khi chúng có sẵn với các đối số đã cho. Những method này cho chúng ta thuận tiện trong việc cấp quyền các hành động cho bất kì `User` instance nào được lấy ra bởi ứng dụng của bạn:

    if ($user->can('update', $post)) {
        //
    }

    if ($user->cannot('update', $post)) {
        //
    }

#### Trong Blade Templates

Tương tự, `@can` Blade directive sẽ sử dụng policies khi chúng có sẵn cho các đối số:

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @endcan

#### Thông Qua Policy Helper

Global `policy` helper function có thể được sử dụng để lấy `Policy` class cho một class instance đã cho. Ví dụ, chúng ta có thể truyền một `Post` instance vào `policy` helper để nhận được một instance tương ứng `PostPolicy` class:

    if (policy($post)->update($user, $post)) {
        //
    }

<a name="controller-authorization"></a>
## Controller Authorization

Mặc định, `App\Http\Controllers\Controller` class trong Laravel sử dụng `AuthorizesRequests` trait. Trait này cung cấp `authorize` method, mà có thể được sử dụng để nhanh chóng cấp quyền cho một hành động và throw một `AuthorizationException` nếu hành động không được cấp quyền.

`authorize` method giống các phương thức cấp quyền khác như `Gate::allows` và `$user->can()`. Vì vậy, hãy sử dụng `authorize` method để nhanh chóng cấp quyền cho một request thực hiện cập nhật một `Post`:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
            $post = Post::findOrFail($id);

            $this->authorize('update', $post);

            // Update Post...
        }
    }

Nếu hành động được cấp quyền, controller sẽ tiếp tục thực thi bình thường; tuy nhiên, nếu `authorize` method xác định rằng hành động không được cấp quyền, một `AuthorizationException` sẽ tự động được throw cái mà sẽ sinh ra một HTTP response với `403 Not Authorized` status code. Như bạn có thể thấy, `authorize` method là cách thuận tiện, nhanh chóng để cấp quyền một hành động hoặc throw một exception với duy nhất một dòng code.

`AuthorizesRequests` trait cũng cung cấp `authorizeForUser` method để cấp quyền một hành động cho một user mà hiện tại chưa được xác thực:

    $this->authorizeForUser($user, 'update', $post);

#### Tự Động Xác Định Policy Methods

Các method của một policy sẽ tương ứng với các method trong controller. Ví dụ, trong `update` method trên, controller method và policy method cùng sử dụng chung tên: `update`.

Vì lí do này, Laravel cho phép bạn đơn giản truyền vào các đối số instance vào `authorize` method, và ability để cấp quyền sẽ tự động được xác định dựa trên tên của hàm gọi. Trong ví dụ này, khi `authorize` được gọi từ controller `update` method, `update` method cũng sẽ được gọi trên `PostPolicy`:

    /**
     * Update the given post.
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
        $post = Post::findOrFail($id);

        $this->authorize($post);

        // Update Post...
    }
