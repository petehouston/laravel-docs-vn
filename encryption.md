# Encryption

- [Cấu hình](#configuration)
- [Sử dụng cơ bản](#basic-usage)

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng phần mã hoá của Laravel, bạn nên set giá trị `key` trong file `config/app.php` là một chuỗi ngẫu nhiên có độ dài là 32 kí tự. Nếu giá trị này không được set, tất cả các giá trị được mã hoá bởi Laravel sẽ không được bảo mật.

<a name="basic-usage"></a>
## Sử dụng cơ bản

#### Mã hoá một giá trị

Bạn có thể mã hoá một giá trị sử dụng `Crypt` [facade](/docs/{{version}}/facades). Tất cả các giá trị được mã hoá sử dụng OpenSSL và thuật toán `AES-256-CBC`. Thêm nữa, các giá trị này được kí với mã MAC (Message Authetication Code) để phát hiện chỉnh sửa ở chuỗi đã được mã hoá.

Ví dụ, chúng ta có thể sử dụng phương thức `encrypt` để mã hoá một secret và lưu vào trong một [Eloquent model](/docs/{{version}}/eloquent):

    <?php

    namespace App\Http\Controllers;

    use Crypt;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => Crypt::encrypt($request->secret)
            ])->save();
        }
    }

> **Chú ý:** Các giá trị được mã hoá sẽ được áp dụng `serialize` trong quá trình mã hoá, đây là cách làm cho các đối tượng và mảng có thể "mã hoá được". Vì vậy, các phía nhận giá trị mã hoá không phải từ PHP thì sẽ cần phải `unserialize` dữ liệu.

#### Giải mã một giá trị

Dĩ nhiên là bạn có thể giải mã sử dụng `decrypt` trong `Crypt` facade. Nếu như giá trị không thể giải mã hoàn chỉnh, ví dụ như mã MAC là không hợp lệ, thì một giá trị ngoại lệ `Illuminate\Contracts\Encryption\DecryptException` sẽ được phát ra:

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = Crypt::decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
