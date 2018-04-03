# Database: Bắt đầu

- [Giới thiệu](#introduction)
- [Thực thi câu lệnh SQL thuần](#running-queries)
    - [Listen các query events](#listening-for-query-events)
- [Database Transactions](#database-transactions)
- [Sử dụng nhiều database connections](#accessing-connections)

<a name="introduction"></a>
## Giới thiệu

Laravel làm cho việc kết nối tới các database và thực thi các query cực kì đơn giản với nhiều database back-ends thông qua sử dụng raw SQL, [fluent query builder](/docs/{{version}}/queries), và [Eloquent ORM](/docs/{{version}}/eloquent). Hiện tại, Laravel hỗ trợ sẵn bốn database sau:

- MySQL
- Postgres
- SQLite
- SQL Server

<a name="configuration"></a>
### Cấu hình

Laravel xử lý việc kết nối và thực thi các query rất đơn giản. Cấu hình cho database nằm trong file `config/database.php`. Trong fil enayf, bạn có thể khai báo tất cả các database connections, cũng như chỉ định connection nào là mặc định.

Về mặc định, [cấu hình môi trường](/docs/{{version}}/installation#environment-configuration) ví dụ của Laravel đã có sẵn để dùng với [Laravel Homestead](/docs/{{version}}/homestead), đây là một môi trường máy ảo hoàn hảo cho phát triển. Dĩ nhiên là bạn có thể hoàn toàn thoải mái thay đổi cấu hình này ứng với database trên máy bạn.

#### Cấu hình SQLite

Sau khi tạo một SQLite database sử dụng câu lệnh như `touch database/database.sqlite`, bạn có thể dễ dàng cấu hình biến môi trường để trỏ tới file được tạo này sử dụng đường dẫn tuyệt đối:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

#### Cấu hình SQL Server

Laravel hỗ trợ sẵn cho SQL Server; tuy nhiên bạn vẫn cần thêm vào thông số cấu hình cho database:

    'sqlsrv' => [
        'driver' => 'sqlsrv',
        'host' => env('DB_HOST', 'localhost'),
        'database' => env('DB_DATABASE', 'forge'),
        'username' => env('DB_USERNAME', 'forge'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => 'utf8',
        'prefix' => '',
    ],

<a name="read-write-connections"></a>
#### Đọc / Ghi các connection

Thi thoảng bạn muốn sử dụng một database connection cho câu lệnh SELECT, một connection khác cho việc INSERT, UPDATE và DELETE. Laravel làm cho điều này quá dễ dàng và các connections sẽ luôn được sử dụng nếu như bạn muốn thực thi raw query, query builder hay Eloquent ORM.

Để biết cấu hình cho các connection đọc / ghi thế nào, hãy coi ví dụ dưới đây:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
    ],

Chú ý là có hai khoá được thêm vào trong mảng cấu hình: `read` và `write`. Những khoá này có mảng giá trị chứa một key duy nhất: `host`. Các thông số còn lại cho `read` và `write` sẽ được gộp từ mảng chính `mysql`.

Vì thế, chúng ta chỉ cần đặt các items vào trong mảng `read` và `write` nếu chúng ta muốn ghi đè các giá trị này ở mảng chính. Do đó, ở trường hợp này, `192.168.1.1` sẽ được sử dụng cho kết nối `read`, còn `192.168.1.2` được sử dụng cho kết nối `write`. Các thông số về database như credentials, prefix, character set và các thông số khác trong mảng chính `mysql` sẽ được dùng chung giữa các connection.

<a name="running-queries"></a>
## Thực thi SQL thuần

Khi mà bạn đã cấu hình cho database, bạn có thể chạy các query sử dụng `DB` facade. `DB` facade cung cấp các hàm để thực hiện các kiểu query: `select`, `update`, `insert`, `delete`, và `statement`.

#### Thực thi lệnh select

Để thực thi một query cơ bản, chúng ta cần sử dụng hàm `select`:

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
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

Tham số thứ nhất truyền vào cho hàm `select` là câu query thuần, trong khi tham số thứ hai là các giá trị binding cần được liên kết vào câu query. Về cơ bản, đây là những giá trị rằng buộc của mệnh đề `where`. Liên kết parameter cung cấp việc bảo vệ chống lại SQL injection.

Hàm `select` luôn trả về một `array` kết quả. Mỗi kết quả trong mảng sẽ là một đối tượng PHP kiểu `StdClass`, cho phép bạn truy xuất vào giá trị bên trong kết quả:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Sử dụng liên kết đặt tên

Thay vì sử dụng `?` để tượng trưng cho liên kết parameter, bạn có thể thực thi câu query sử dụng liên kết đặt tên:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### Thực thi câu lệnh insert

Để thực hiện việc insert, bạn có thể sử dụng hàm `insert` trong `DB` facade. Giống như `select`, hàm này nhận câu raw SQL query ở tham số đầu tiên, và liên kết ở tham số thứ hai:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### Thực thi câu lệnh update

Hàm `update` sẽ được dùng để update các records đang có trong database. Số lượng row ảnh hưởng bởi câu lệnh sẽ được trả về qua hàm này:

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### Thực thi câu lệnh delete

Hàm `delete` cần được sử dụng để xoá các records khỏi database. Giống như `update`, số lượng dòng bị xoá sẽ được trả về:

    $deleted = DB::delete('delete from users');

#### Thực thi một câu lệnh chung

Một vài câu lệnh database không trả về giá trị gì cả. Với những thao tác kiểu này, bạn có thể sử dụng hàm `statement` trong `DB` facade:

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### Listen tới các query events

Nếu bạn muốn nhận câu query SQL thực thi bởi ứng dụng, bạn có thể sử dụng hàm `listen`. Hàm này hữu ích khi thực hiện log các query hay debug. Bạn có thể đăng kí query listener trong một [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use DB;
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
            DB::listen(function ($query) {
                // $query->sql
                // $query->bindings
                // $query->time
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

<a name="database-transactions"></a>
## Database Transactions

Để thực thi một tập các xử lý trong một database transaction, bạn có thể sử dụng hàm `transaction`. Nếu một exception bị bắn ra từ trong transaction `Closure`, transaction sẽ tự động được rollback lại. Nếu `Closure` thực thi thành công, transaction sẽ tự động được commit. Bạn không cần phải lo lắng về việc thực hiện thủ công các thao tác roll back hay commit khi sử dụng hàm `transaction`:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### Thực hiện transaction thủ công

Nếu bạn muốn thực hiện transaction thủ công và muốn quản lý việc rollback và commit, bạn có thể sử dụng hàm `beginTransaction`:

    DB::beginTransaction();

Sử dụng hàm `rollback` để rollback transaction:

    DB::rollBack();

Sử dụng hàm `commit` để commit một transaction:

    DB::commit();

> **Chú ý:** Sử dụng các hàm transaction của `DB` facade cũng có thể quản lý được transaction cho [query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent).

<a name="accessing-connections"></a>
## Sử dụng nhiều database connection

Khi sử dụng nhiều connection, bạn có thể truy cập vào mỗi connection thông qua hàm `connection` trên `DB` facade. Giá trị `name` truyền vào hàm `connection` cần tương ứng với tên của connections trong file cấu hình `config/database.php`:

    $users = DB::connection('foo')->select(...);

Bạn cũng có thể truy cập vào trong phần raw, đối tượng PDO bên dưới sử dụng hàm `getPdo`:

    $pdo = DB::connection()->getPdo();
