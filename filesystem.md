# Filesystem / Cloud Storage

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
- [Sử dụng cơ bản](#basic-usage)
    - [Lấy disk instances](#obtaining-disk-instances)
    - [Lấy files](#retrieving-files)
    - [Lưu files](#storing-files)
    - [File Visibility](#file-visibility)
    - [Xoá files](#deleting-files)
    - [Các thư mục](#directories)
- [Custom Filesystems](#custom-filesystems)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một lớp abstraction cho filesystem khá mạnh mẽ nhờ có sử dụng tới PHP package tuyệt vời [Flysystem](https://github.com/thephpleague/flysystem) tạo bởi Frank de Jonge. Việc tích hợp Flysystem vào Laravel cung cấp cách sử dụng rất đơn giản để giao tiếp với các local filesystem, Amazon S3, và Rackspace Cloud Storage. Thậm chi còn hay hơn, đó là có thể thay đổi các storage này cực kì dễ dàng mà sử dụng chung API cho các hệ thống.

<a name="configuration"></a>
## Cấu hình

File cấu hình cho filesystem được đặt tại `config/filesystems.php`. Trong file này, bạn có thể cấu hình tất cả "disks" của bạn. Mỗi diesk tượng trưng cho một storage driver cụ thể và vị trí storage. Cấu hình ví dụ cho mỗi driver nằm trong file cấu hình. Vì thế, hãy thay đổi giá trị cấu hình sao cho hợp với ý của bạn.

Tất nhiên, bạn có thể cấu hình bao nhiêu disk tuỳ thích, và thậm chí có thể sử dụng nhiều disks mà cùng sử dụng chung driver.

<a name="the-public-disk"></a>
#### The Public Disk

Disk `public` dùng cho các files có thể được truy cập công khai từ bên ngoài. Về mặc định, `public` sử dụng `local` driver và lưu các file này trong `storage/app/public`. Để chúng có thể truy cập từ web, bạn nên tạo một symbolic link từ `publis/storage` sang `storage/app/public`. Cách này sẽ làm cho các file public trong một thư mục mà có thể dễ dàng chia sẻ qua các lần triển khai khi sử dụng hệ thống triển khai không mất thời gian như [Envoyer](https://envoyer.io).

Dĩ nhiên, khi mà file đã được lưu và symbolic link được tạo, bạn có thể tạo một URL tới file sử dụng helper `asset`:

    echo asset('storage/file.txt');

#### Local Driver

Khi sử dụng `local` driver, chú ý là tất cả các thao tác tới file là relative tới thư mục `root` mà được khai báo trong file cấu hình. Về mặc định, giá trị này được trỏ tới `storage/app`. Vì thế, hàm sau đây sẽ lưu file vào trong `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

#### Các yêu cầu driver khác

Trước khi sử dụng S3 hay Rackspace drivers, bạn sẽ cần cài đặt các package phù hợp thông qua Composer:

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### Cấu hình cho FTP Driver

Sự tích hợp với Flysystem làm cho việc giao tiếp FTP trở nên tốt hơn; tuy nhiên, file cấu hình ví dụ lại không đi kèm với cấu hình mặc định `filesystems.php`. Nếu bạn cần cấu hình tới FTP, bạn có thể sử dụng ví dụ cấu hình dưới đây:

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Cấu hình cho Rackspace Driver

Các thông số cấu hình mẫu cho Rackspace không có sẵn trong file `filesystems.php`. Nếu muốn sử dụng, bạn có thể dùng mẫu cấu hình dưới đây:

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="basic-usage"></a>
## Sử dụng cơ bản

<a name="obtaining-disk-instances"></a>
### Lấy Disk Instances

`Storage` facade có thể được dùng để tương tác với bất kì disks đã cấu hình nào. Ví dụ, bạn có thể sử dụng hàm `put` để lưu một avatar vào trong disk mặc định. Nếu bạn gọi hàm này mà không gọi tới hàm `disk` trước, thì nó sẽ tự động lấy disk mặc định:

    <?php

    namespace App\Http\Controllers;

    use Storage;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Update the avatar for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function updateAvatar(Request $request, $id)
        {
            $user = User::findOrFail($id);

            Storage::put(
                'avatars/'.$user->id,
                file_get_contents($request->file('avatar')->getRealPath())
            );
        }
    }

Khi sử dụng nhiều disk, bạn có thể truy xuất tới một disk cụ thể sử dụng hàm `disk` trong `Storage` facade. Dĩ nhiên là bạn vẫn có thể tiếp tục móc nối các hàm để thực thi trên disk:

    $disk = Storage::disk('s3');

    $contents = Storage::disk('local')->get('file.jpg')

<a name="retrieving-files"></a>
### Lấy Files

Hàm `get` có thể được dung để lấy nội dung của một file. Chuỗi nội dung của file sẽ được trả lại:

    $contents = Storage::get('file.jpg');

Hàm `exists` được dùng để kiểm tra xem một file có tồn tại trên disk hay không:

    $exists = Storage::disk('s3')->exists('file.jpg');

### File URLs

Khi sử dụng driver là `local` hay `s3`, bạn có thể dùng hàm `url` để lấy URL cho file. Nếu bạn sử dụng `local` driver, nó sẽ tự động thêm vào `/storage` cho path và trả về một relative URL của file. Nếu bạn sử dụng `s3` driver, URL đầy đủ sẽ được trả về.

    $url = Storage::url('file1.jpg');

> **Chú ý:** Khi sử dụng `local` driver, hãy chắc chắn [tạo một symbolic link tại `public/storage`](#the-public-disk) trỏ tới thư mục `storage/app/public`.

#### Thông tin meta của file

Hàm `size` trả về kích thước của file, đơn vị là bytes:

    $size = Storage::size('file1.jpg');

Hàm `lastModified` trả về giá trị UNIX timestamp của lần cuối cùng file bị thay đổi:

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
### Lưu Files

Hàm `put` có thể được dùng để lưu file lên disk. Bạn cũng có thể truyền một PHP `resource` cho hàm `put`, nó sẽ sử dụng stream của Flysystem. Khuyến khích sử dụng stream khi phải làm việc với file lớn:

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

Hàm `copy` được dùng để copy một file đang tồn tại sang một vị trí mới trên disk:

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

Hàm `move` được dùng để đổi tên hay di chuyển một file đang tồn tại tới vị trí mới:

    Storage::move('old/file1.jpg', 'new/file1.jpg');

#### Chèn thêm nội dung vào file

Hàm `prepend` và `append` cho phép bạn dễ dàng chèn thêm nội dung vào đầu hay cuối file:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="file-visibility"></a>
### File Visibility

Thuộc tính file visibility có thể lấy và set thông qua hai hàm `getVisibility` và `setVisibility`. Visibility là lớp abstraction của file permission giữa các nền tảng với nhau:

    Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

Ngoài ra, bạn có thể set visibility khi sử dụng hàm `put`. Giá trị hợp lệ cho visibility là `public` và `private`:

    Storage::put('file.jpg', $contents, 'public');

<a name="deleting-files"></a>
### Xoá Files

Hàm `delete` nhận tên file hay một mảng các file để xoá khỏi disk:

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
### Các thư mục

#### Lấy các files trong một thư mục

Hàm `files` trả về một mảng các files trong một thư mục. Nếu bạn muốn lấy danh sách tất cả các file trong một thư mục bao gồm các thư mục con, bạn có thể sử dụng hàm `allFiles`:

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### Lấy tất cả các thư mục trong một thư mục

Hàm `directories` trả về một mảng tất cả các thư mục bên trong một thư mục cho trước. Thêm vào đó, bạn có thể sử dụng hàm `allDirectories` để lấy danh sách tất cả các thư mục trong một thư mục và các thư mục con của nó:

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### Tạo một thư mục mới

Hàm `makeDirectory` sẽ tạo một thư mục mới, bao gồm các thư mục con cần thiết:

    Storage::makeDirectory($directory);

#### Xoá thư mục

Cuối cùng, hàm `deleteDirectory` được dùng để xoá một thư mục, bao gồm tất cả các file của nó:

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## Tạo Filesystems riêng

Flysystem của Laravel cung cấp sẵn vài "drivers"; tuy nhiên, Flysystem không hề bị giới hạn bởi những drivers này mà có adapters để hỗ trợ cho các hệ thống storage khác. Bạn có thể tạo một driver riêng nếu bạn muốn sử dụng một trong những adapter bổ sung này trong ứng dụng Laravel của bạn.

Để cài đặt filesystem riêng, bạn sẽ cần tạo một [service provider](/docs/{{version}}/providers) ví dụ như `DropboxServiceProvider`. Trong hàm `boot`, bạn có thể sử dụng `Storage` facade với hàm `extend` để khai báo driver riêng:

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Dropbox\Client as DropboxClient;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Dropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function($app, $config) {
                $client = new DropboxClient(
                    $config['accessToken'], $config['clientIdentifier']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Tham số đầu tiên của hàm `method` là tên của driver còn tham số thứ hai là một Closure nhận vào hai biến là `$app` và `$config`. Closure phải trả về một instance của `League\Flysystem\Filesystem`. Biến `$config` chứa các giá trị khai báo trong `config/filesystems.php` cho disk được chỉ định.

Khi mà bạn đã tạo service provider để đăng kí extension, bạn có thể sử dụng driver `dropbox` trong file cấu hình `config/filesystem.php`.
