# Database: Query Builder

- [Giới thiệu](#introduction)
- [Retrieving Results](#retrieving-results)
    - [Aggregates](#aggregates)
- [Selects](#selects)
- [Joins](#joins)
- [Unions](#unions)
- [Where Clauses](#where-clauses)
    - [Các mệnh đề Where nâng cao](#advanced-where-clauses)
    - [Các mệnh đề JSON Where](#json-where-clauses)
- [Ordering, Grouping, Limit, & Offset](#ordering-grouping-limit-and-offset)
- [Conditional Statements](#conditional-statements)
- [Inserts](#inserts)
- [Updates](#updates)
- [Deletes](#deletes)
- [Pessimistic Locking](#pessimistic-locking)

<a name="introduction"></a>
## Giới thiệu

Query builder cung cấp một giao thức thuận tiện, linh hoạt cho việc tạo và thực thi các truy vấn dữ liệu. Nó có thể sử dụng để thực hiện hầu hết các tính toán dữ liệu trong ứng dụng của bạn, và làm việc trên tất các các hệ cơ sở dữ liệu được hỗ trợ.

> **Ghi chú:** Laravel query builder sử dụng PDO parameter binding để bảo vệ ứng dụng của bạn khỏi SQL injection. Vì vậy không cần phải xử lí các chuỗi khi truyền vào.

<a name="retrieving-results"></a>
## Retrieving Results

#### Trả về toàn bộ dòng từ một table

Bắt đầu với một fluent query, sử dụng phương thức `table` trên facade `DB`. Phương thức `table` trả về một fluent query builder instance với bảng đã cho, cho phép bạn thêm nhiều ràng buộc vào truy vấn và cuối cùng là lấy kết quả. Trong ví dụ này, hãy `get` toàn bộ bản ghi từ 1 table:

    <?php

    namespace App\Http\Controllers;

    use DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

Giống như [raw queries](/docs/{{version}}/database), phương thức `get` trả về một `array` các kết quả mà mỗi kết quả là một instance của đối tượng PHP `StdClass`. Bạn có thể truy cập mỗi cột giá trị bằng cách coi cột như là thuộc tính của một object.

    foreach ($users as $user) {
        echo $user->name;
    }

#### Trả về một dòng / cột từ một bảng

Nếu bạn chỉ cần lấy một dòng từ bảng dữ liệu, bạn có thể sử dụng phương thức `first`. Phương thức này sẽ trả về một đối tượng `StdClass`:

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

Nếu bạn không cần lấy toàn bộ dòng, bạn có thể lấy ra một giá trị từ một bản ghi sử dụng phương thức `value`. Phương thức này sẽ trả về giá trị của cột:

    $email = DB::table('users')->where('name', 'John')->value('email');

#### Chia nhỏ kết quả một bảng

Nếu bạn phải làm việc với hàng nghìn bản ghi dữ liệu, hãy xem xét việc sử dụng phương thức `chunk`. Phương thức này sẽ lấy một "chunk" các kết quả tại một thời điểm, và đưa mỗi chunk vào một `Closure` để xử lí. Phương thức này vô cùng hữu ích cho việc viết [Artisan commands](/docs/{{version}}/artisan) để xử lí hàng nghìn bản ghi. Ví dụ hãy làm việc với toàn bộ table `users` với các chunks 100 bản ghi một lúc:

    DB::table('users')->orderBy('id')->chunk(100, function($users) {
        foreach ($users as $user) {
            //
        }
    });

Bạn có thể dừng các chunk khác khỏi việc xử lí bằng cách trả về `false` từ `Closure`:

    DB::table('users')->orderBy('id')->chunk(100, function($users) {
        // Process the records...

        return false;
    });

#### Trả về danh sách giá trị của một cột

Nếu bạn thích lấy một mảng gồm các giá trị của một cột, bạn có thể sử dụng phương thức `pluck`. Trong ví dụ này, chúng ta sẽ lấy một mảng gồm các role titles:

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

 Bạn cũng có thể chỉ định một cột key cho mảng trả về:

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="aggregates"></a>
### Aggregates

Query builder cũng cung cấp một tập hợp các phương thức khác nhau, như là `count`, `max`, `min`, `avg` và `sum`. Bạn có thể gọi bất kì phương thức nào sau cấu trúc truy vấn:

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Tất nhiên, bạn có thể gộp những phương thức này với các mệnh đề khác để tạo truy vấn:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects

#### Chỉ định một mệnh đề select

Tất nhiên, bạn có thể không phải lúc nào cũng muốn lấy toàn bộ các cột từ một bảng. Sử dụng phương thức `select`, bạn có thể chỉ định tùy chọn một mệnh đề `select` cho truy vấn:

    $users = DB::table('users')->select('name', 'email as user_email')->get();

Phương thức `distinct` cho phép bạn bắt buộc truy vấn trả về các kết quả phân biệt:

    $users = DB::table('users')->distinct()->get();

Nếu bạn đã có sẵn một query builder instance và bạn muốn thêm một cột vào mệnh đề select, bạn có thể sử dụng phương thức `addSelect`:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

#### Raw Expressions

Đôi khi bạn có thể cần sử dụng một biểu thức trong truy vấn. Những expression này sẽ được đưa vào truy vấn như các chuỗi, vì vậy hãy cẩn thận đừng tạo bất kì lỗi SQL injection nào. Để tạo một raw expression, bạn có thể sử dụng phương thức `DB:raw`:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## Joins

#### Cú pháp Inner Join

Query builder cũng có thể được sử dụng để viết các cú pháp join. Để thực hiện một "inner join" SQL đơn giản, bạn có thể sử dụng phương thức `join` cho một query builder instance. Tham số được truyền vào đầu tiên trong phương thức `join` là tên của bảng bạn join đến, trong khi tham số còn lại chỉ định các cột ràng buộc cho việc join. Tất nhiên như bạn có thể thấy, bạn có thể join nhiều bảng trong một truy vấn:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Cú pháp Left Join

Nếu bạn thích thực hiện một "left join" thay vì "inner join", sử dụng phương thức `leftJoin`. Phương thức này có cú pháp giống phương thức `join`: 

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cú pháp Cross Join

Để thực hiện một "cross join", sử dụng phương thức `crossJoin` với tên của bảng bạn muốn cross join đến. Cross join sinh ra một cartesion product (tích Descartes - là một tập hợp AxB của A và B) giữa bảng đầu tiên và bảng bị join:

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### Các cú pháp Join nâng cao

Bạn cũng có thể chỉ định nhiều mệnh đề join nâng cao. Để bắt đầu, truyền một `Closure` như là tham số thứ 2 vào phương thức `join`. `Closure` sẽ nhận một đối tượng `JoinClause` cái mà cho phép bạn chỉ định các ràng buộc trong mệnh đề `join`:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

Nếu bạn thích sử dụng mệnh đề "where" trong join, bạn có thể sử dụng phương thức `where` và `orWhere` trong join. Thay vì so sánh 2 cột, các phương thức này sẽ so sánh cột với giá trị:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions

Query builder cũng cung cấp một cách nhanh chóng để "union" 2 truy vấn với nhau. Ví dụ, bạn có thể tạo một truy vấn khởi tạo, và sau đó sử dụng phương thức `union` để nối nó vào truy vấn thứ 2:

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

Phương thức `unionAll` cũng có và có cách sử dụng như `union`.

<a name="where-clauses"></a>
## Mệnh đề Where

### Các mệnh đề Wherer đơn giản

Để thêm mệnh đề `where` vào truy vấn, sử dụng phương thức `where` trong query builder instance. Hầu hết cách gọi cơ bản `where` yêu cầu 3 tham số. Tham số đầu tiên là tên của cột. Tham số thứ 2 là một toán tử, cái mà có thể là bất kì toán tử nào mà được hỗ trợ bởi database. Tham số thứ 3 là giá trị để so sánh với cột.

Ví dụ, đây là một truy vấn mà kiểm tra giá trị của cột "votes" bằng 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

Để thuận tiện, bạn có thể đơn giản chỉ muốn lấy một cột có giá trị bằng giá trị đã cho, bạn chỉ cần truyền giá trị trực tiếp vào như là tham số thứ 2 vào phương thức `where`:

    $users = DB::table('users')->where('votes', 100)->get();

Tất nhiên, bạn có thể sử dụng nhiều toán tử khác khi viết mệnh đề `where`:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

Bạn có thể truyền vào một mảng điều kiện vào hàm `where`:

    $users = DB::table('users')->where([
        ['status','1'],
        ['subscribed','<>','1'],
    ])->get();

#### Cú pháp Or

Bạn có thể nối tiếp các ràng buộc where cùng nhau, cũng như thêm mệnh đề `or` vào truy vấn. Phương thức `orWhere` chấp nhận các đối số giống như `where`:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### Các mệnh đề Where khác

**whereBetween**

Phương thức `whereBetween` kiểm tra giá trị một cột có ở giữa 2 giá trị:

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

Phương thức `whereNotBetween` kiểm tra giá trị của cột có nằm bên ngoài hai giá trị:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

Phương thức `whereIn` kiểm tra giá trị của cột đã có có thuộc về mảng:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

Phương thức `whereNotIn` kiểm tra giá trị của cột có **không** thuộc về mảng:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

Phương thức `whereNull` kiểm tra giá trị của cột đã có là `NULL`:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

Phương thức `whereNotNull` kiểm tra giá trị của cột có là **không** `NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereColumn**

Phương thức `whereColumn` có thể sử dụng để kiểm tra 2 cột bằng nhau:

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name');

Bạn cũng có thể truyền một toán tử so sánh vào phương thức:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at');

Phương thức `whereColumn` có thể được truyền vào một mảng các điều kiện. Những điều kiện này sẽ được nối với nhau sử dụng toán tử `and`:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ]);

<a name="advanced-where-clauses"></a>
## Mệnh đề Where nâng cao

#### Nhóm tham số

Đôi khi bạn có thể cần tạo nhiều mệnh đề where nâng cao như "where exists" hoặc nhóm các tham số lồng nhau. Laravel query builder có thể xử lí việc này ok. Để bắt đầu, hãy xem ví dụ sau về việc nhóm các ràng buộc trong ngoặc:

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

Như bạn có thể thấy, truyền một `Closure` vào trong phương thức `orWhere` chỉ cho query builder bắt đầu một nhóm ràng buộc. `Closure` sẽ nhận một query builder instance cái mà bạn có thể sử dụng để thiết lập các ràng buộc mà sẽ được đặt trong trong ngoặc. Ví dụ trên sẽ tạo ra một câu lệnh SQL như sau:

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

#### Các cú pháp Exist

Phương thức `whereExists` cho phép bạn viết các mệnh đề `where exists`. Phương thức `whereExists` chấp nhận tham số là một `Closure`, cái mà sẽ nhận một query builder instance cho phép bạn định nghĩa truy vấn mà sẽ được đặt trong mệnh đề "exists":

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

Truy vấn trên sẽ sinh ra đoạn SQL sau:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
## Mệnh đề JSON Where

Laravel hỗ trợ truy vấn với cột kiểu JSON trên database hỗ trợ JSON. Hiện tại, các hỗ trợ này có trong MySQL 5.7 và Postgres. Để truy vấn một cột JSON, sử dụng toán tử `->`:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit, & Offset

#### orderBy

Phương thức `orderBy` cho phép bạn sắp xếp kết quả của truy vấn bởi một cột cho trước. Tham số đầu tiên của phương thức `orderBy` nên là cột bạn muốn sắp xếp, trong khi tham số thứ 2 là chiểu của sắp xếp và có thể là `asc` hoặc `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### inRandomOrder

Phương thức `inRandomOrder` có thể được sử dụng để sắp xếp kết quả truy vấn một cách ngẫu nhiên. Ví dụ bạn có thể sử dụng phương thức này để lấy một user ngẫu nhiên:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having / havingRaw

Phương thức `groupBy` và `having` có thể được sử dụng để nhóm kết quả truy vấn. Phương thức `having` có cách sử dụng tương tự phương thức `where`:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Phương thức `havingRaw` có thể được sử dụng thiết lập các chuỗi vào mệnh đề `having`. Ví dụ chúng ta có thể tìm toàn bộ các department mà có sale lớn hơn 2,500$:

    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### skip / take

Để giới hạn số kết quả trả về từ truy vấn, hoặc bỏ qua một số cho trước các kết quả trong truy vấn (`OFFSET`), bạn có thể sử dụng phương thức `skip` và `take`:

    $users = DB::table('users')->skip(10)->take(5)->get();

<a name="conditional-statements"></a>
## Các cú pháp điều kiện

Đôi khi bạn có thể muốn các cú pháp áp dụng vào truy vấn chỉ khi cái đéo gì đấy đúng. Ví dụ bạn chỉ muốn áp dụng mệnh đề `where` khi có giá trị nhập vào ở trong trong request đến. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức `when`:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


Phương thức `when` chỉ thực hiện Closure khi tham số đầu tiên là `true`. Nếu tham số đầu tiên là `false`, Closure sẽ không được thực thi.

<a name="inserts"></a>
## Inserts

Query builder cũng cung cấp phương thức `insert` cho việc chèn các bản ghi vào trong bảng. Phương thức `insert` chấp nhận một mảng tên các cột và giá trị để thêm vào:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

Bạn có thể chèn các bản ghi riêng biệt vào bảng với một lần gọi `insert` bằng cách truyền vào một mảng các mảng. Mỗi mảng con đại diện cho một dòng sẽ được chèn vô table:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### Auto-Incrementing IDs

Nếu bảng có một id auto-incrementing, sử dụng phương thức `insertGetId` để thêm vào một bản ghi vào sau đó lấy ID:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> **Ghi chú:** Khi sử dụng PostgreSQL phương thức insertGetId coi như cột auto-incrementing sẽ được đặt tên là `id`. Nếu bạn thích lấy giá trị ID từ một "sequence" khác, bạn có thể truyền vào the sequence name như là tham số thứ 2 trong phương thức `insertGetId`.

<a name="updates"></a>
## Updates

Tất nhiên, ngoài việc chèn thêm bản ghi vào database, query builder cũng có thể cập nhật bản ghi có sẵn bằng cách sử dụng phương thức `update`. Phương thức `update` giống như `insert`, chấp nhận một mảng các cặp cột và giá trị có trong cột để cập nhật. Bạn có thể ràng buộc truy vấn `update` sử dụng mệnh đề `where`:

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

#### Increment / Decrement

Query builder cũng cung cấp các phương thức thuận tiện cho việc tăng hay giảm giá trị của một cột. Đây chỉ đơn giản là một short-cut, cung cấp một interface nhanh chóng và ngắn gọn so với việc viết cú pháp `update`.

Cả hai phương thức trên đều chấp nhận ít nhất 1 tham số: cột để thay đổi. Một tham số thứ 2 có thể tùy chọn được truyền vào để điều khiển giá trị tăng hay giảm cho cột.

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

Bạn cũng có thể chỉ định thêm các cột để cập nhật trong khi thực hiện:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

Tất nhiên, query builder cũng có thể được sử dụng để xóa các bản ghi từ bảng thông qua phương thức `delete`:

    DB::table('users')->delete();

Bạn có thể ràng buộc cú pháp `delete` bằng cách thêm mệnh đề `where` trước khi gọi phương thức `delete`:

    DB::table('users')->where('votes', '>', 100)->delete();

Nếu bạn muốn truncate toàn bộ bảng, cái mà sẽ xóa toàn bộ các dòng và reset lại auto-incrementing ID về 0, bạn có thể sử dụng phương thức `truncate`:

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## Pessimistic Locking

Query builder cũng bao gồm các hàm nhỏ để giúp bạn thực hiện "pessimistic locking" trên cú pháp `select`. Để chạy cú pháp với một "shared lock", bạn có thể sử dụng phương thức `sharedLock` trên truy vấn. Một shared lock bảo về các dòng được chọn khỏi việc bị thay đổi tới khi transaction của bạn được ủy thác (nghe như game - commit):

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Ngoài ra, bạn có thể sử dụng phương thức `lockForUpdate`. Một "for update" lock bảo về các dòng khỏi việc thay đổi hoặc bị selected bởi các shared lock khác

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();