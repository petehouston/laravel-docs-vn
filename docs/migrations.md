# Database: Migrations

- [Database: Migrations](#database-migrations)
  - [Giới thiệu](#gi%E1%BB%9Bi-thi%E1%BB%87u)
  - [Tạo Migrations](#t%E1%BA%A1o-migrations)
  - [Cấu trúc Migration](#c%E1%BA%A5u-tr%C3%BAc-migration)
  - [Thực thi Migrations](#th%E1%BB%B1c-thi-migrations)
    - [Ép buộc thực thi migration chạy cho môi trường production](#%C3%A9p-bu%E1%BB%99c-th%E1%BB%B1c-thi-migration-ch%E1%BA%A1y-cho-m%C3%B4i-tr%C6%B0%E1%BB%9Dng-production)
    - [Rolling Back Migrations](#rolling-back-migrations)
    - [Rollback / Migrate trong một câu lệnh](#rollback-migrate-trong-m%E1%BB%99t-c%C3%A2u-l%E1%BB%87nh)
  - [Bảng](#b%E1%BA%A3ng)
    - [Tạo Tables](#t%E1%BA%A1o-tables)
      - [Kiểm tra xem bảng hay cột có tồn tại hay không](#ki%E1%BB%83m-tra-xem-b%E1%BA%A3ng-hay-c%E1%BB%99t-c%C3%B3-t%E1%BB%93n-t%E1%BA%A1i-hay-kh%C3%B4ng)
      - [Connection & Storage Engine](#connection-storage-engine)
    - [Đổi tên hay drop tables](#%C4%91%E1%BB%95i-t%C3%AAn-hay-drop-tables)
      - [Đổi tên table với foreign keys](#%C4%91%E1%BB%95i-t%C3%AAn-table-v%E1%BB%9Bi-foreign-keys)
  - [Cột](#c%E1%BB%99t)
    - [Tạo columns](#t%E1%BA%A1o-columns)
      - [Các kiểu column](#c%C3%A1c-ki%E1%BB%83u-column)
      - [Column Modifiers](#column-modifiers)
    - [Chỉnh sửa Columns](#ch%E1%BB%89nh-s%E1%BB%ADa-columns)
      - [Yêu cầu](#y%C3%AAu-c%E1%BA%A7u)
      - [Thay đổi thuộc tính của column](#thay-%C4%91%E1%BB%95i-thu%E1%BB%99c-t%C3%ADnh-c%E1%BB%A7a-column)
      - [Đổi tên của columns](#%C4%91%E1%BB%95i-t%C3%AAn-c%E1%BB%A7a-columns)
    - [Drop columns](#drop-columns)
    - [Available Command Aliases](#available-command-aliases)
  - [Indexes](#indexes)
    - [Tạo indexes](#t%E1%BA%A1o-indexes)
      - [Các kiểu index được hỗ trợ](#c%C3%A1c-ki%E1%BB%83u-index-%C4%91%C6%B0%E1%BB%A3c-h%E1%BB%97-tr%E1%BB%A3)
      - [Chiều dài chỉ mục và MySQL / MariaDB](#chi%E1%BB%81u-d%C3%A0i-ch%E1%BB%89-m%E1%BB%A5c-v%C3%A0-mysql-mariadb)
    - [Drop indexes](#drop-indexes)
    - [Foreign Key Constraints](#foreign-key-constraints)

## Giới thiệu

Migration được coi như là version control cho database, cho phép team có thể dễ dàng thay đổi và chia sẻ schema của database trong chương trình với nhau. Migratiom cơ bản được sử dụng cùng với schema builder để dễ dàng xây dựng cấu trúc cho database schema.

`Schema` [facade](facades.mf) hỗ trợ việc tạo và thao tác trên các bảng mà không cần biết về database bằng việc sử dụng các API rõ ràng, tương ứng và mạch lạc khi giao tiếp với các hệ thống database khác nhau mà Laravel hỗ trợ.

## Tạo Migrations

Để tạo một migration, sử dụng câu lệnh `make:migration`:

```PHP
php artisan make:migration create_users_table
```

File migration mới sẽ được đặt trong thư mục `database/migrations`. Mỗi file migration được đặt tên bao gồm timestamp để xác định thứ tự các migration với nhau.

Hai tham số truyền trong câu lệnh `--table` và `--create` có thể được sử dụng để cho biết table nào và migration có cần tạo table mới hay không. Hai tham số này đơn giản được dùng để cho mã stub được tạo ra sẽ thực hiện trên table nào:

```PHP
php artisan make:migration add_votes_to_users_table --table=users

php artisan make:migration create_users_table --create=users
```

Nếu bạn muốn đưa migration vào trong một thư mục khác, bạn có thể truyền vào tham số `--path` khi thực hiện câu lệnh `make:migration`. Đường dẫn đưa vào phải relative với đường dẫn cơ bản của chương trình.

## Cấu trúc Migration

Một migration class chứa hai hàm cơ bản là: `up` và `down`. Hàm `up` được dùng để tạo table, cột hay index mới vào trong database, trong khi hàm `down` đơn giản chỉ dùng để làm ngược lại những thao tác ở hàm `up`.

Bên trong hai hàm này bạn có thể sử dụng schema builder để tạo và chỉnh sửa table rõ ràng hơn. Để tìm hiểu tất cả các hàm được cung cấp trong `Schema` builder, [hãy đọc phần này](#creating-tables). Ví dụ, hãy cùng nhau làm một ví dụ về migration bằng việc tạo ra một bảng tên là `flights`:

```php
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
```

## Thực thi Migrations

Để thực thi tất cả các migration trong chương trình, sử dụng hàm `migrate`. Nếu bạn đang sử dụng [Homestead](homestead.md), bạn cần chạy câu lệnh này bên trong đó:

```php
php artisan migrate
```

Nếu bạn nhận lỗi "class not found" khi thực thi migration, hãy thử thực thi câu lệnh này trước `composer dump-autoload` và sau đó gõ lại câu lệnh migrate.

### Ép buộc thực thi migration chạy cho môi trường production

Một số thao tác migration khá là nguy hiểm, vì chúng có thể gây ra mất mát dữ liệu. Để tránh khỏi điều này khi thực thi các câu lệnh migration trong môi trường production, bạn sẽ nhận được yêu cầu xác nhận trước khi thực chi chúng. Để ép các câu lệnh này thực thi mà không cần xác nhận, sử dụng cờ `--force`:

```PHP
php artisan migrate --force
```

### Rolling Back Migrations

Để rollback lại thao tác migration cuối cùng, bạn có thể sử dụng câu lệnh `rollback`. Chú ý là việc rollback này sẽ thực hiện lại "nhóm" những migration được chạy lần gần nhất, có thể là một hay nhiều files:

```PHP
php artisan migrate:rollback
```

Bạn có thế giới hạn số file migrations rollback bằng thêm step vào lệnh rollback. Ví dụ, lệnh sau sẽ rollback 5 file sau cùng của bảng migrations có "batch" lớn nhất:

```php
php artisan migrate:rollback --step=5
```

Câu lệnh `migrate:reset` sẽ thực hiện rollback lại toàn bộ migration của chương trình:

```PHP
php artisan migrate:reset
```

### Rollback / Migrate trong một câu lệnh

Câu lệnh `migrate:refresh` sẽ đầu tiên rollback lại toàn bộ migration của chương trình, và thực hiện câu lệnh `migrate`. Câu lệnh sẽ thực hiện tái cấu trúc toàn bộ database:

```php
php artisan migrate:refresh

php artisan migrate:refresh --seed
```

Bạn có thẻ rollback & migrate lại với giới hạn migrations bởi cờ step vào lệnh refresh. Ví dụ, lệnh sau sẽ rollback & migrate lại 5 file mới nhất của migrations:

```php
php artisan migrate:refresh --step=5
```

## Bảng

### Tạo Tables

Để tạo một bảng mới, sử dụng hàm `create` của `Schema` facacde. Hàm `create` nhận hai tham số. Tham số đầu là tên của bảng, còn tham số thứ hai là một `Closure` mà sẽ nhận vào một `Blueprint` object để khai báo cấu trúc của bảng mới:

```php
Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});
```

Khi tạo bảng mới, bạn có thể sử dụng bất cứ [hàm tạo column](#tạo-columns) nào để tạo các trường cho bảng.

#### Kiểm tra xem bảng hay cột có tồn tại hay không

Bạn có thể dễ dàng kiểm tra sự tồn tại của một bảng hay cột sử dụng hàm `hasTable` và `hasColumn`:

```php
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
```

#### Connection & Storage Engine

Nếu bạn muốn thực hiện thao tác schema trên một kết nối database không phải mặc định, sử dụng hàm `connection`:

```php
Schema::connection('foo')->create('users', function ($table) {
    $table->increments('id');
});
```

Để thiết lập storage engine cho bảng, sử dụng thuộc tính `engine` trên schema builder:

```php
Schema::create('users', function ($table) {
    $table->engine = 'InnoDB';

    $table->increments('id');
});
```

Bạn có thể sử dụng các lệnh sau đây trên schema builder để xác định các tùy chọn của bảng:

| Chỉ thị                                | Miêu tả                                           |
| -------------------------------------- | ------------------------------------------------- |
| $table->engine = 'InnoDB';             | Chỉ định công cụ lưu trữ bảng (MySQL).            |
| $table->charset = 'utf8';              | Chỉ định một bộ ký tự mặc định cho bảng (MySQL).  |
| $table->collation = 'utf8_unicode_ci'; | Chỉ định một collation mặc định cho bảng (MySQL). |
| $table->temporary();                   | Tạo một bảng tạm (trừ SQL Server).                |

### Đổi tên hay drop tables

Để đổi tên một bảng đã tồn tại trong database, sử dụng hàm `rename`:

```php
Schema::rename($from, $to);
```

Để drop một bảng trong database, bạn có thể sử dụng hàm `drop` hoặc `dropIfExists`:

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

#### Đổi tên table với foreign keys

Trước khi thay đổi tên bảng, bạn nên kiểm tra xem có foreign key constraints nào trên bảng có tên khác trong migration file hay không thay vì để Laravel tự gán tên. Nếu không, tên của foreign key constraint sẽ trỏ tới tên cũ của table.

## Cột

### Tạo columns

Để cập nhật một bảng, chúng ta sẽ sử dụng hàm `table` của `Schema` facade. Tương tự hàm `create`, hàm `table` nhận vào hai tham số: tên của bảng và một `Closure` nhận một `Blueprint` object để thực hiện thao tác với table:

```php
Schema::table('users', function ($table) {
    $table->string('email');
});
```

#### Các kiểu column

Schema builder về cơ bản có chứa một danh sách các kiểu column mà bạn có thể sử dụng để xây dựng cấu trúc table:

| Hàm                                        | Mô tả                                                                                          |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| `$table->bigIncrements('id');`             | Tăng ID (primary key) tương đương với "UNSIGNED BIG INTEGER".                                  |
| `$table->bigInteger('votes');`             | tương đương với BIGINT.                                                                        |
| `$table->binary('data');`                  | tương đương với BLOB.                                                                          |
| `$table->boolean('confirmed');`            | tương đương với BOOLEAN.                                                                       |
| `$table->char('name', 4);`                 | tương đương với CHAR và có độ dài thiết lập trước.                                             |
| `$table->date('created_at');`              | tương đương với DATE.                                                                          |
| `$table->dateTime('created_at');`          | tương đương với DATETIME.                                                                      |
| `$table->dateTimeTz('created_at');`        | tương đương với DATETIME (cùng timezone).                                                      |
| `$table->decimal('amount', 5, 2);`         | tương đương với DECIMAL và phần thập phân.                                                     |
| `$table->double('column', 15, 8);`         | tương đương với DOUBLE có độ dài là 15 chữ số và 8 chữ số phần thập phân.                      |
| `$table->enum('choices', ['foo', 'bar']);` | tương đương với ENUM.                                                                          |
| `$table->float('amount');`                 | tương đương với FLOAT.                                                                         |
| `$table->increments('id');`                | Tăng ID (primary key) tương đương với "UNSIGNED INTEGER".                                      |
| `$table->integer('votes');`                | tương đương với INTEGER.                                                                       |
| `$table->ipAddress('visitor');`            | tương đương với IP address.                                                                    |
| `$table->json('options');`                 | tương đương với JSON.                                                                          |
| `$table->jsonb('options');`                | tương đương với JSONB.                                                                         |
| `$table->longText('description');`         | tương đương với LONGTEXT.                                                                      |
| `$table->macAddress('device');`            | tương đương với MAC address.                                                                   |
| `$table->mediumInteger('numbers');`        | tương đương với MEDIUMINT.                                                                     |
| `$table->mediumText('description');`       | tương đương với MEDIUMTEXT.                                                                    |
| `$table->morphs('taggable');`              | thêm INTEGER `taggable_id` và STRING `taggable_type`.                                          |
| `$table->multiLineString('positions');`    | MULTILINESTRING equivalent column.                                                             |
| `$table->multiPoint('positions');`         | MULTIPOINT equivalent column.                                                                  |
| `$table->multiPolygon('positions');`       | MULTIPOLYGON equivalent column.                                                                |
| `$table->nullableTimestamps();`            | giống với `timestamps()`, ngoại trừ việc cho phép sử dụng NULLs.                               |
| `$table->point('position');`               | POINT equivalent column.                                                                       |
| `$table->polygon('positions');`            | POLYGON equivalent column.                                                                     |
| `$table->rememberToken();`                 | thêm `remember_token` như VARCHAR(100) NULL.                                                   |
| `$table->smallInteger('votes');`           | tương đương với SMALLINT.                                                                      |
| `$table->softDeletes();`                   | thêm `deleted_at` column để soft deletes.                                                      |
| `$table->softDeletesTz();`                 | Adds a nullable deleted_at TIMESTAMP (with timezone) equivalent column for soft deletes.       |
| `$table->string('email');`                 | tương đương với VARCHAR.                                                                       |
| `$table->string('name', 100);`             | tương đương với VARCHAR có độ dài.                                                             |
| `$table->text('description');`             | tương đương với TEXT.                                                                          |
| `$table->time('sunrise');`                 | tương đương với TIME.                                                                          |
| `$table->timeTz('sunrise');`               | tương đương với TIME (với timezone).                                                           |
| `$table->tinyInteger('numbers');`          | tương đương với TINYINT.                                                                       |
| `$table->timestamp('added_on');`           | tương đương với TIMESTAMP.                                                                     |
| `$table->timestampTz('added_on');`         | tương đương với TIMESTAMP.                                                                     |
| `$table->timestamps();`                    | thêm vào hai column `created_at` và `updated_at`.                                              |
| `$table->tinyIncrements('id');`            | Auto-incrementing UNSIGNED TINYINT (primary key) equivalent column.                            |
| `$table->tinyInteger('votes');`            | TINYINT equivalent column.                                                                     |
| `$table->unsignedBigInteger('votes');`     | UNSIGNED BIGINT equivalent column.                                                             |
| `$table->unsignedDecimal('amount', 8, 2);` | UNSIGNED DECIMAL equivalent column with a precision (total digits) and scale (decimal digits). |
| `$table->unsignedInteger('votes');`        | UNSIGNED INTEGER equivalent column.                                                            |
| `$table->unsignedMediumInteger('votes');`  | UNSIGNED MEDIUMINT equivalent column.                                                          |
| `$table->unsignedSmallInteger('votes');`   | UNSIGNED SMALLINT equivalent column.                                                           |
| `$table->unsignedTinyInteger('votes');`    | UNSIGNED TINYINT equivalent column.                                                            |
| `$table->uuid('id');`                      | UUID equivalent column.                                                                        |
| `$table->year('birth_year');`              | YEAR equivalent column.                                                                        |

#### Column Modifiers

Ngoài các kiểu column liệt kê ở trên, có một số kiểu column "modifier" khác mà bạn có thể sử dụng. Ví dụ để làm cho column "nullable", bạn có thể sử dụng hàm `nullable`:

```php
Schema::table('users', function ($table) {
    $table->string('email')->nullable();
});
```

Dưới đây là một số column modifier mà bạn có thể sử dụng, không bao gồm các [index modifiers](#tạo-indexes):

| Modifier                  | Mô tả                                                      |
| ------------------------- | ---------------------------------------------------------- |
| `->first()`               | Đặt column "first" trong table (chỉ áp dụng với MySQL)     |
| `->after('column')`       | Đặt column "after" một column khác (chỉ áp dụng với MySQL) |
| `->nullable()`            | Cho phép dữ liệu kiểu NULL có thể chèn vào column.         |
| `->default($value)`       | Thiết lập giá trị mặc định cho column.                     |
| `->unsigned()`            | Thiết lập kiểu `unsigned` cho column.                      |
| `->comment('my comment')` | Thêm một comment cho column.                               |

### Chỉnh sửa Columns

#### Yêu cầu

Trước khi chỉnh sửa column, hãy chắc chắn là bạn thêm vào `doctrine/dbal` dependency vào trong file `composer.json`. Thư viện Doctrine DBAL được dùng để xác định trạng thái hiện tại của column và tạo câu SQL query cần thiết để chỉnh sửa column.

#### Thay đổi thuộc tính của column

Hàm `change` cho phép bạn thay đổi một column sang một kiểu mới, hoặc thay đổi thuộc tính của column. Ví dụ, bạn có thể muốn tăng kích thước của string. Xem ví dụ dưới đây trong việc sử dụng `change` để thay đổi kích thước của column `name` từ 25 thành 50:

```php
Schema::table('users', function ($table) {
    $table->string('name', 50)->change();
});
```

Chúng ta cũng có thể thay đổi một column sang `nullable`:

```php
Schema::table('users', function ($table) {
    $table->string('name', 50)->nullable()->change();
});
```

> **Chú ý:** Chỉ các loại cột sau có thể được "thay đổi": bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json, longText, mediumText, smallInteger, string, text, time, unsignedBigInteger, unsignedInteger và unsignedSmallInteger.

#### Đổi tên của columns

Để thay đổi tên column, bạn có thể sử dụng hàm `renameColumn` trong Schema builder:

```php
Schema::table('users', function ($table) {
    $table->renameColumn('from', 'to');
});
```

> **Chú ý:** Hiện chưa hỗ trợ đổi tên cho column có kiểu là `enum`.

### Drop columns

Để drop một column, sử dụng hàm `dropColumn`:

```php
Schema::table('users', function ($table) {
    $table->dropColumn('votes');
});
```

Bạn có thể drop nhiều column trong một bảng bằng cách truyền vào một mảng tên các column vào hàm `dropColumn`:

```php
Schema::table('users', function ($table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

> **Chú ý:** Trước khi drop column trong SQLite, bạn cần phải thêm vào `doctrine/dbal` vào trong `composer.json` và thực thi câu lệnh `composer update` để cài thư viện.
> **Chú ý:** Drop hay thay đổi nhiều columns trong một file migration trong khi sử dụng SQLite chưa được hỗ trợ.

### Available Command Aliases

| Chỉ thị                      | Miêu tả                                   |
| ---------------------------- | ----------------------------------------- |
| $table->dropRememberToken(); | Xóa cột remember_token.                   |
| $table->dropSoftDeletes();   | Xóa cột deleted_at.                       |
| $table->dropSoftDeletesTz(); | Bí danh của phương pháp.dropSoftDeletes() |
| $table->dropTimestamps();    | Xóa cột created_at và cột updated_at.     |
| $table->dropTimestampsTz();  | Bí danh của phương pháp.dropTimestamps()  |

## Indexes

### Tạo indexes

Schema builder hỗ trợ vài kiểu index. Đầu tiên, cùng xem một ví dụ tạo column có giá trị là unique. Để tạo index, chúng ta sẽ sử dụng hàm `unique` khi tạo column:

```php
$table->string('email')->unique();
```

Ngoài ra bạn có thể tạo index ngay sau khi tạo column, ví dụ:

```php
$table->unique('email');
```

Bạn có thể truyền vào một mảng column cho hàm `index`:

```php
$table->index(['account_id', 'created_at']);
```

Laravel sẽ tự thực hiện tạo tên phù hợp cho index, nhưng bạn có thể tự đặt tên cho index bằng cách truyền vào tham số thứ hai:

```php
$table->index('email', 'my_index_name');
```

#### Các kiểu index được hỗ trợ

| Hàm                                         | Mô tả                        |
| ------------------------------------------- | ---------------------------- |
| `$table->primary('id');`                    | Thêm vào một primary key.    |
| `$table->primary(['first', 'last']);`       | Thêm vào composite keys.     |
| `$table->unique('email');`                  | Thêm một unique index.       |
| `$table->unique('state', 'my_index_name');` | Thêm một index có tên riêng. |
| `$table->index('state');`                   | Thêm vào một index.          |

#### Chiều dài chỉ mục và MySQL / MariaDB

Laravel sử dụng bộ ký tự  ''utf8mb4'' mặc định, bao gồm hỗ trợ lưu trữ "emojis" trong cơ sở dữ liệu. Nếu bạn đang chạy một phiên bản của MySQL cũ hơn bản phát hành 5,7.7 hoặc MariaDB cũ hơn phiên bản 10.2.2, bạn có thể cần phải tự cấu hình độ dài chuỗi mặc định được tạo ra bởi di chuyển để cho MySQL tạo các chỉ mục cho chúng. Bạn có thể định cấu hình bằng cách gọi phương thức ``Schema::defaultStringLength`` trong ``AppServiceProvider``:

```php
use Illuminate\Support\Facades\Schema;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Schema::defaultStringLength(191);
}
```

Ngoài ra, bạn có thể kích hoạt ``innodb_large_prefix`` tùy chọn cho cơ sở dữ liệu của bạn. Tham khảo tài liệu của cơ sở dữ liệu của bạn để có hướng dẫn về cách bật đúng tùy chọn này.

### Drop indexes

Để drop một index, bạn cần truyền vào tên của index. Về mặc định, Laravel sẽ tự động gán một tên phù hợp cho index. Đơn giản chỉ là nối tên bảng, tên của column được index và kiểu index. Dưới đây là một vài ví dụ:

| Hàm                                         | Mô tả                                    |
| ------------------------------------------- | ---------------------------------------- |
| `$table->dropPrimary('users_id_primary');`  | Drop một primary key khỏi table "users". |
| `$table->dropUnique('users_email_unique');` | Drop một unique key khỏi table "users".  |
| `$table->dropIndex('geo_state_index');`     | Drop một index khỏi table "geo".         |

Nếu bạn truyền vào một mảng các column, thì Laravel sẽ thực hiện tìm tên index tương ứng dựa theo quy tắc đặt để drop.

```php
Schema::table('geo', function ($table) {
    $table->dropIndex(['state']); // Drops index 'geo_state_index'
});
```

### Foreign Key Constraints

Laravel cũng hỗ trợ cung cấp việc tạo foreign key constraint một cách dễ dàng. Ví dụ, cùng tạo một column `user_id` trên table `posts` tham chiếu tới column `id` trên bảng `users`:

```php
Schema::table('posts', function ($table) {
    $table->integer('user_id')->unsigned();

    $table->foreign('user_id')->references('id')->on('users');
});
```

Bạn cũng có thể chỉ định thao tác cho thuộc tính "on delete" và "on update" của constraint:

```php
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
```

Để drop một foreign key, bạn có thể sử dụng hàm `dropForeign`. Foreign key constraints sử dụng chung quy tắc đặt tên như index. Vì thế, chúng ta tên của foreign key constraints sẽ bao gồm tên bảng, tên cột và hậu tố là "_foreign":

```php
$table->dropForeign('posts_user_id_foreign');
```

Hay bạn có thể truyền vào một mảng các giá trị để thực hiện drop theo quy tắc đặt tên:

```php
$table->dropForeign(['user_id']);
```

Bạn có thể kích hoạt hay bỏ kích hoạt việc sử dụng foreign key constraint trong migration sử dụng hai hàm sau:

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();
```