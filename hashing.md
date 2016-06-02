# Hashing

- [Giới thiệu](#introduction)
- [Sử dụng cơ bản](#basic-usage)

<a name="introduction"></a>
## Giới thiệu

Laravel `Hash` [facade](/docs/{{version}}/facades) cung cấp phương thức hash an toàn với Bcrypt để lưu mật khẩu của người dùng. Nếu bạn sử dụng `AuthController` có sẵn trong Laravel, nó đã tự động thiết lập sử dụng sẵn Bcrypt cho việc đăng kí và chứng thực.

Bcrypt là một sự lựa chọn tốt cho hashing mật khẩu bởi vì **chỉ số hoạt động* của nó có thể điều chỉnh được, có nghĩa là thời gian tốn để tạo ra một hash có thể tăng lên nếu như công suất của phần cứng tăng lên.

<a name="basic-usage"></a>
## Sử dụng cơ bản

Bạn có thể hash mật khẩu bằng cách gọi hàm `make` trong `Hash` facade:

    <?php

    namespace App\Http\Controllers;

    use Hash;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function updatePassword(Request $request, $id)
        {
            $user = User::findOrFail($id);

            // Validate the new password length...

            $user->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

Cách khác, bạn có thể dùng helper `bcrypt`:

    bcrypt('plain-text');

#### So sánh mật khẩu với một hash

Hàm `check` cho phép bạn so sánh một chuỗi với một hash. Tuy nhiên, nếu bạn đang sử dụng `AuthController` [đi kèm với Laravel](/docs/{{version}}/authentication), thì bạn không cần sử dụng trực tiếp, vì nó việc xác nhận này đã được xử lý sắn trong controller:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### Kiểm tra nếu mật khẩu cần được hash lại

Hàm `needsRehash` cho phép bạn xác định nếu chỉ số hoạt động của máy hash đã thay đổi khi mật khẩu thay đổi:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
