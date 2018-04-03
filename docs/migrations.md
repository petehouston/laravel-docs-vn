# Database: Migrations

- [Giới thiệu](#introduction)
- [Tạo migrations](#generating-migrations)
- [Cấu trúc migration](#migration-structure)
- [Thực thi migrations](#running-migrations)
    - [Rollback Migrations](#rolling-back-migrations)
- [Cách viết migrations](#writing-migrations)
    - [Tạo tables](#creating-tables)
    - [Thay đổi tên hay Drop tables](#renaming-and-dropping-tables)
    - [Tạo columns](#creating-columns)
    - [Thay đổi columns](#modifying-columns)
    - [Drop columns](#dropping-columns)
    - [Tạo indexes](#creating-indexes)
    - [Drop indexes](#dropping-indexes)
    - [Foreign key constraints](#foreign-key-constraints)

<a name="introduction"></a>
## Giới thiệu

Migration được coi như là version control cho database, cho phép team có thể dễ dàng thay đổi và chia sẻ schema của database trong chương trình với nhau. Migratiom cơ bản được sử dụng cùng với schema builder để dễ dàng xây dựng cấu trúc cho database schema.

`Schema` [facade](/docs/{{version}}/facades) hỗ trợ việc tạo và thao tác trên các bảng mà không cần biết về database bằng việc sử dụng các API rõ ràng, tương ứng và mạch lạc khi giao tiếp với các hệ thống database khác nhau mà Laravel hỗ trợ.

<a name="generating-migrations"></a>
## Tạo Migrations

Để tạo một migration, sử dụng câu lệnh `make:migration`:

    php artisan make:migration create_users_table

File migration mới sẽ được đặt trong thư mục `database/migrations`. Mỗi file migration được đặt tên bao gồm timestamp để xác định thứ tự các migration với nhau.

Hai tham số truyền trong câu lệnh `--table` và `--create` có thể được sử dụng để cho biết table nào và migration có cần tạo table mới hay không. Hai tham số này đơn giản được dùng để cho mã stub được tạo ra sẽ thực hiện trên table nào:

    php artisan make:migration add_votes_to_users_table --table=users

    php artisan make:migration create_users_table --create=users

Nếu bạn muốn đưa migration vào trong một thư mục khác, bạn có thể truyền vào tham số `--path` khi thực hiện câu lệnh `make:migration`. Đường dẫn đưa vào phải relative với đường dẫn cơ bản của chương trình.

<a name="migration-structure"></a>
## Cấu trúc Migration

Một migration class chứa hai hàm cơ bản là: `up` và `down`. Hàm `up` được dùng để tạo table, cột hay index mới vào trong database, trong khi hàm `down` đơn giản chỉ dùng để làm ngược lại những thao tác ở hàm `up`.

Bên trong hai hàm này bạn có thể sử dụng schema builder để tạo và chỉnh sửa table rõ ràng hơn. Để tìm hiểu tất cả các hàm được cung cấp trong `Schema` builder, [hãy đọc phần này](#creating-tables). Ví dụ, hãy cùng nhau làm một ví dụ về migration bằng việc tạo ra một bảng tên là `flights`:

    <?php

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }


<a name="running-migrations"></a>
## Thực thi Migrations

Để thực thi tất cả các migration trong chương trình, sử dụng hàm `migrate`. Nếu bạn đang sử dụng [Homestead](/docs/{{version}}/homestead), bạn cần chạy câu lệnh này bên trong đó:

    php artisan migrate

Nếu bạn nhận lỗi "class not found" khi thực thi migration, hãy thử thực thi câu lệnh này trước `composer dump-autoload` và sau đó gõ lại câu lệnh migrate.

#### Ép buộc thực thi migration chạy cho môi trường production

Một số thao tác migration khá là nguy hiểm, vì chúng có thể gây ra mất mát dữ liệu. Để tránh khỏi điều này khi thực thi các câu lệnh migration trong môi trường production, bạn sẽ nhận được yêu cầu xác nhận trước khi thực chi chúng. Để ép các câu lệnh này thực thi mà không cần xác nhận, sử dụng cờ `--force`:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### Rolling Back Migrations

Để rollback lại thao tác migration cuối cùng, bạn có thể sử dụng câu lệnh `rollback`. Chú ý là việc rollback này sẽ thực hiện lại "nhóm" những migration được chạy lần gần nhất, có thể là một hay nhiều files:

    php artisan migrate:rollback

Câu lệnh `migrate:reset` sẽ thực hiện rollback lại toàn bộ migration của chương trình:

    php artisan migrate:reset

#### Rollback / Migrate trong một câu lệnh

Câu lệnh `migrate:refresh` sẽ đầu tiên rollback lại toàn bộ migration của chương trình, và thực hiện câu lệnh `migrate`. Câu lệnh sẽ thực hiện tái cấu trúc toàn bộ database:

    php artisan migrate:refresh

    php artisan migrate:refresh --seed

<a name="writing-migrations"></a>
## Viết Migrations

<a name="creating-tables"></a>
### Tạo Tables

Để tạo một bảng mới, sử dụng hàm `create` của `Schema` facacde. Hàm `create` nhận hai tham số. Tham số đầu là tên của bảng, còn tham số thứ hai là một `Closure` mà sẽ nhận vào một `Blueprint` object để khai báo cấu trúc của bảng mới:

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Khi tạo bảng mới, bạn có thể sử dụng bất cứ [hàm tạo column](#creating-columns) nào để tạo các trường cho bảng.

#### Kiểm tra xem bảng hay cột có tồn tại hay không

Bạn có thể dễ dàng kiểm tra sự tồn tại của một bảng hay cột sử dụng hàm `hasTable` và `hasColumn`:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### Connection & Storage Engine

Nếu bạn muốn thực hiện thao tác schema trên một kết nối database không phải mặc định, sử dụng hàm `connection`:

    Schema::connection('foo')->create('users', function ($table) {
        $table->increments('id');
    });

Để thiết lập storage engine cho bảng, sử dụng thuộc tính `engine` trên schema builder:

    Schema::create('users', function ($table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### Đổi tên hay drop tables

Để đổi tên một bảng đã tồn tại trong database, sử dụng hàm `rename`:

    Schema::rename($from, $to);

Để drop một bảng trong database, bạn có thể sử dụng hàm `drop` hoặc `dropIfExists`:

    Schema::drop('users');

    Schema::dropIfExists('users');

#### Đổi tên table với foreign keys

Trước khi thay đổi tên bảng, bạn nên kiểm tra xem có foreign key constraints nào trên bảng có tên khác trong migration file hay không thay vì để Laravel tự gán tên. Nếu không, tên của foreign key constraint sẽ trỏ tới tên cũ của table.

<a name="creating-columns"></a>
### Tạo columns

Để cập nhật một bảng, chúng ta sẽ sử dụng hàm `table` của `Schema` facade. Tương tự hàm `create`, hàm `table` nhận vào hai tham số: tên của bảng và một `Closure` nhận một `Blueprint` object để thực hiện thao tác với table:

    Schema::table('users', function ($table) {
        $table->string('email');
    });

#### Các kiểu column

Schema builder về cơ bản có chứa một danh sách các kiểu column mà bạn có thể sử dụng để xây dựng cấu trúc table:

Hàm  | Mô tả
------------- | -------------
`$table->bigIncrements('id');`  |  Tăng ID (primary key) tương đương với "UNSIGNED BIG INTEGER".
`$table->bigInteger('votes');`  |  tương đương với BIGINT.
`$table->binary('data');`  |  tương đương với BLOB.
`$table->boolean('confirmed');`  |  tương đương với BOOLEAN.
`$table->char('name', 4);`  |  tương đương với CHAR và có độ dài thiết lập trước.
`$table->date('created_at');`  |  tương đương với DATE.
`$table->dateTime('created_at');`  |  tương đương với DATETIME.
`$table->dateTimeTz('created_at');`  |  tương đương với DATETIME (cùng timezone).
`$table->decimal('amount', 5, 2);`  |  tương đương với DECIMAL và phần thập phân.
`$table->double('column', 15, 8);`  |  tương đương với DOUBLE có độ dài là 15 chữ số và 8 chữ số phần thập phân.
`$table->enum('choices', ['foo', 'bar']);` | tương đương với ENUM.
`$table->float('amount');`  |  tương đương với FLOAT.
`$table->increments('id');`  |  Tăng ID (primary key) tương đương với "UNSIGNED INTEGER".
`$table->integer('votes');`  |  tương đương với INTEGER.
`$table->ipAddress('visitor');`  |  tương đương với IP address.
`$table->json('options');`  |  tương đương với JSON.
`$table->jsonb('options');`  |  tương đương với JSONB.
`$table->longText('description');`  |  tương đương với LONGTEXT.
`$table->macAddress('device');`  |  tương đương với MAC address.
`$table->mediumInteger('numbers');`  |  tương đương với MEDIUMINT.
`$table->mediumText('description');`  |  tương đương với MEDIUMTEXT.
`$table->morphs('taggable');`  |  thêm INTEGER `taggable_id` và STRING `taggable_type`.
`$table->nullableTimestamps();`  |  giống với `timestamps()`, ngoại trừ việc cho phép sử dụng NULLs.
`$table->rememberToken();`  |  thêm `remember_token` như VARCHAR(100) NULL.
`$table->smallInteger('votes');`  |  tương đương với SMALLINT.
`$table->softDeletes();`  |  thêm `deleted_at` column để soft deletes.
`$table->string('email');`  |  tương đương với VARCHAR.
`$table->string('name', 100);`  |  tương đương với VARCHAR có độ dài.
`$table->text('description');`  |  tương đương với TEXT.
`$table->time('sunrise');`  |  tương đương với TIME.
`$table->timeTz('sunrise');`  |  tương đương với TIME (với timezone).
`$table->tinyInteger('numbers');`  |  tương đương với TINYINT.
`$table->timestamp('added_on');`  |  tương đương với TIMESTAMP.
`$table->timestampTz('added_on');`  |  tương đương với TIMESTAMP.
`$table->timestamps();`  |  thêm vào hai column `created_at` và `updated_at`.
`$table->uuid('id');`  |  tương đương với UUID.

#### Column Modifiers

Ngoài các kiểu column liệt kê ở trên, có một số kiểu column "modifier" khác mà bạn có thể sử dụng. Ví dụ để làm cho column "nullable", bạn có thể sử dụng hàm `nullable`:

    Schema::table('users', function ($table) {
        $table->string('email')->nullable();
    });

Dưới đây là một số column modifier mà bạn có thể sử dụng, không bao gồm các [index modifiers](#creating-indexes):

Modifier  | Mô tả
------------- | -------------
`->first()`  |  Đặt column "first" trong table (chỉ áp dụng với MySQL)
`->after('column')`  |  Đặt column "after" một column khác (chỉ áp dụng với MySQL)
`->nullable()`  |  Cho phép dữ liệu kiểu NULL có thể chèn vào column.
`->default($value)`  |  Thiết lập giá trị mặc định cho column.
`->unsigned()`  |  Thiết lập kiểu `unsigned` cho column.
`->comment('my comment')`  |  Thêm một comment cho column.

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### Chỉnh sửa Columns

#### Yêu cầu

Trước khi chỉnh sửa column, hãy chắc chắn là bạn thêm vào `doctrine/dbal` dependency vào trong file `composer.json`. Thư viện Doctrine DBAL được dùng để xác định trạng thái hiện tại của column và tạo câu SQL query cần thiết để chỉnh sửa column.

#### Thay đổi thuộc tính của column

Hàm `change` cho phép bạn thay đổi một column sang một kiểu mới, hoặc thay đổi thuộc tính của column. Ví dụ, bạn có thể muốn tăng kích thước của string. Xem ví dụ dưới đây trong việc sử dụng `change` để thay đổi kích thước của column `name` từ 25 thành 50:

    Schema::table('users', function ($table) {
        $table->string('name', 50)->change();
    });

Chúng ta cũng có thể thay đổi một column sang `nullable`:

    Schema::table('users', function ($table) {
        $table->string('name', 50)->nullable()->change();
    });

> **Chú ý:** Hiện tại chưa hỗ trợ thay đổi cột có kiểu là `enum`.

<a name="renaming-columns"></a>
#### Đổi tên của columns

Để thay đổi tên column, bạn có thể sử dụng hàm `renameColumn` trong Schema builder:

    Schema::table('users', function ($table) {
        $table->renameColumn('from', 'to');
    });

> **Chú ý:** Hiện chưa hỗ trợ đổi tên cho column có kiểu là `enum`.

<a name="dropping-columns"></a>
### Drop columns

Để drop một column, sử dụng hàm `dropColumn`:

    Schema::table('users', function ($table) {
        $table->dropColumn('votes');
    });

Bạn có thể drop nhiều column trong một bảng bằng cách truyền vào một mảng tên các column vào hàm `dropColumn`:

    Schema::table('users', function ($table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> **Chú ý:** Trước khi drop column trong SQLite, bạn cần phải thêm vào `doctrine/dbal` vào trong `composer.json` và thực thi câu lệnh `composer update` để cài thư viện.

> **Chú ý:** Drop hay thay đổi nhiều columns trong một file migration trong khi sử dụng SQLite chưa được hỗ trợ.

<a name="creating-indexes"></a>
### Tạo indexes

Schema builder hỗ trợ vài kiểu index. Đầu tiên, cùng xem một ví dụ tạo column có giá trị là unique. Để tạo index, chúng ta sẽ sử dụng hàm `unique` khi tạo column:

    $table->string('email')->unique();

Ngoài ra bạn có thể tạo index ngay sau khi tạo column, ví dụ:

    $table->unique('email');

Bạn có thể truyền vào một mảng column cho hàm `index`:

    $table->index(['account_id', 'created_at']);

Laravel sẽ tự thực hiện tạo tên phù hợp cho index, nhưng bạn có thể tự đặt tên cho index bằng cách truyền vào tham số thứ hai:

    $table->index('email', 'my_index_name');

#### Các kiểu index được hỗ trợ

Hàm  | Mô tả
------------- | -------------
`$table->primary('id');`  |  Thêm vào một primary key.
`$table->primary(['first', 'last']);`  |  Thêm vào composite keys.
`$table->unique('email');`  |  Thêm một unique index.
`$table->unique('state', 'my_index_name');`  |  Thêm một index có tên riêng.
`$table->index('state');`  |  Thêm vào một index.

<a name="dropping-indexes"></a>
### Drop indexes

Để drop một index, bạn cần truyền vào tên của index. Về mặc định, Laravel sẽ tự động gán một tên phù hợp cho index. Đơn giản chỉ là nối tên bảng, tên của column được index và kiểu index. Dưới đây là một vài ví dụ:

Hàm  | Mô tả
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  Drop một primary key khỏi table "users".
`$table->dropUnique('users_email_unique');`  |  Drop một unique key khỏi table "users".
`$table->dropIndex('geo_state_index');`  |  Drop một index khỏi table "geo".

Nếu bạn truyền vào một mảng các column, thì Laravel sẽ thực hiện tìm tên index tương ứng dựa theo quy tắc đặt để drop.

    Schema::table('geo', function ($table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### Foreign Key Constraints

Laravel cũng hỗ trợ cung cấp việc tạo foreign key constraint một cách dễ dàng. Ví dụ, cùng tạo một column `user_id` trên table `posts` tham chiếu tới column `id` trên bảng `users`:

    Schema::table('posts', function ($table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

Bạn cũng có thể chỉ định thao tác cho thuộc tính "on delete" và "on update" của constraint:

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

Để drop một foreign key, bạn có thể sử dụng hàm `dropForeign`. Foreign key constraints sử dụng chung quy tắc đặt tên như index. Vì thế, chúng ta tên của foreign key constraints sẽ bao gồm tên bảng, tên cột và hậu tố là "_foreign":

    $table->dropForeign('posts_user_id_foreign');

Hay bạn có thể truyền vào một mảng các giá trị để thực hiện drop theo quy tắc đặt tên:

    $table->dropForeign(['user_id']);

Bạn có thể kích hoạt hay bỏ kích hoạt việc sử dụng foreign key constraint trong migration sử dụng hai hàm sau:

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();
