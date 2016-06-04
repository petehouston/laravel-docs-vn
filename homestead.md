# Laravel Homestead

- [Giới thiệu](#introduction)
- [Cài đặt & Thiết lập](#installation-and-setup)
    - [Khởi đầu](#first-steps)
    - [Cấu hình Homestead](#configuring-homestead)
    - [Khởi động Vagrant Box](#launching-the-vagrant-box)
    - [Cài đặt cho từng dự án Project](#per-project-installation)
    - [Cài đặt MariaDB](#installing-mariadb)
- [Cách sử dụng thường nhật](#daily-usage)
    - [Truy cập Homestead trên toàn hệ thống](#accessing-homestead-globally)
    - [Kết nối thông qua SSH](#connecting-via-ssh)
    - [Kết nối đến cơ sở dữ liệu](#connecting-to-databases)
    - [Thêm trang bổ sung](#adding-additional-sites)
    - [Cấu hình lịch Cron](#configuring-cron-schedules)
    - [Cổng](#ports)
- [Giao thức mạng](#network-interfaces)

<a name="introduction"></a>
## Giới thiệu

Laravel cố gắng để làm cho toàn bộ trải nghiệm phát triển PHP trở nên thú vị, bao gồm cả môi trường phát triển của bạn. [Vagrant](http://vagrantup.com) đưa ra một phương pháp đơn giản để cung cấp và quản lý máy ảo.

Laravel là Vagrant box chính thức, được đóng gói sẵn cung cấp cho bạn một môi trường phát triển tuyệt vời mà không yêu cầu bạn phải cài đặt PHP, HHVM, web server, hay bất cứ software nào khác trên máy trạm của bạn. Không còn mối bận tâm về việc làm rối tung hệ điều hành của bạn lên! Vargrant boxes là hoàn toàn đủ. Nếu có lỗi thì bạn có thể hủy và tạo lại một box khác trong vài phút!

Homestead chạy trên bất cứ hệ thống Windows, Mac, hay Linux, nó bao gồm cả Nginx web server, PHP 7.0, MySQL, Postgres, Redis, Memcached, Node, và bất cứ thứ gì khác bạn cần để phát triển một ứng dụng Laravel tuyệt vời.

> **Note:** Nếu bạn dùng Windows, bạn cần phải bật tính năng ảo hoá phần cứng (VT-x). Nó thường được bật thông qua BIOS. Nếu bạn dùng Hyper-V trên một hệ thống UEFI bạn có thể cần phải tắt Hyper-V trước khi bật VT-x.

<a name="included-software"></a>
### Phần mềm gồm có 

- Ubuntu 14.04
- Git
- PHP 7.0
- HHVM
- Nginx
- MySQL
- MariaDB
- Sqlite3
- Postgres
- Composer
- Node (With PM2, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd

<a name="installation-and-setup"></a>
## Cài đặt & Thiết lập

<a name="first-steps"></a>
### Khởi đầu

Trước khi chạy môi trường Homestead của bạn, bạn cần phải cài đặt [VirtualBox 5.x](https://www.virtualbox.org/wiki/Downloads) hoặc [VMWare](http://www.vmware.com), và [Vagrant](http://www.vagrantup.com/downloads.html). Tất cả những phần mềm trên đều cung cấp giao diện cài đặt trực quan và dễ sử dụng cho tất cả các hệ điều hành phổ biến.

Để sử dụng VMWare, bạn cần phải mua cả VMWare Fusion / Workstation và cả [VMware Vagrant plug-in](http://www.vagrantup.com/vmware). Dù không miễn phí nhưng VMWare có thể cung cấp hiệu suất truy cập thư mục chia sẻ nhanh hơn.

#### Cài đặt Homestead Vagrant

Sau khi VirtualBox / VMWare và Vagrant đã cài đặt xong, bạn cần thêm hộp `laravel/homestead` vào Vagrant bằng cách sử dụng dòng lệnh sau ở terminal (cửa sổ dòng lệnh). Bạn sẽ mất vài phút để tải hộp, tuỳ thuộc vào tốc độ Internet của bạn:

    vagrant box add laravel/homestead

Nếu dòng lệnh này thất bại, hãy chắc chắn rằng Vagrant là bản mới nhất.

#### Cài đặt Homestead

Bạn có thể cài đặt Homestead bằng cách đơn giản là sao chép kho lưu trữ. Hãy cân nhắc sao chép vào `Homestead` folder bên trong thư mục "home" của bạn, như thế thì Homestead sẽ có thể hoạt động như là máy chủ cho tất cả dự án Laravel cúa bạn:

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead

Sau khi bạn đã sao chếp Homestead, vào thư mục Homestead và chạy lệnh `bash init.sh` để tạo file cấu hình `Homestead.yaml`. File `Homestead.yaml` sẽ được đặt trong thư mục ẩn `~/.homestead`:

    bash init.sh

<a name="configuring-homestead"></a>
### Cấu hình Homestead

#### Cài đặt nhà cung cấp

Từ khoá `provider` trong file `~/.homestead/Homestead.yaml` chỉ ra nhà cung cấp Vagrant nào sẽ được sử dụng: `virtualbox`, `vmware_fusion`, hay là `vmware_workstation`. Bạn có thể thiết lập nhà cung cấp mà bạn muốn:

    provider: virtualbox

#### Cấu hình Folder chia sẻ

Từ khoá `folders` trong file `Homestead.yaml` liệt kê tất cả các folder mà bạn muốn chia sẻ với môi trường Homestead. Nếu như file nào đó trong folder này thay đổi thì nó sẽ được đồng bộ hoá giữa máy trạm của bạn với môi trường Homestead. Bạn có thể cấu hình nhiều folder chia sẻ nếu cần:

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

Để bật [NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html), bạn chỉ cần thêm một cờ đơn giản vào cấu hình chia sẻ folder của bạn:

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

#### Cấu hình Nginx

Bạn không quen thuộc với Nginx? Không sao cả. Thuộc tính `sites` co phép bạn dễ dàng map một "domain" đến một folder trong môi trường Homestead của bạn. File `Homestead.yaml` đã bao gồm một cấu hình ví dụ. Một lần nữa, bạn có thể thêm nhiều trang vào môi trường Homestead của bạn nếu cần. Homestead có thể hoạt động như một môi trường ảo hoá thuận tiện cho từng dự án Laravel mà bạn làm việc:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

Bạn có thể cho phép bất cứ trang Homestead nào sử dụng [HHVM](http://hhvm.com) bằng cách cài đặt tuỳ chọn `hhvm` thành `true`:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

Nếu bạn thay đổi thuộc tính `sites` sau khi cung cấp hộp Homestead, bạn cần chạy lại lệnh `vagrant reload --provision` để làm mới cấu hình Nginx trên máy ảo.

#### File Hosts

Bạn cần phải thêm "domain" cho trang Nginx vào file `host` trong máy của bạn. File `host` sẽ chuyển hướng requests của bạn đến trang Homestead sang máy chủ Homestead. Trên Mac và Linux, file này nằm ở `/etc/hosts` (với một vài version của Mac thì nằm ở `/private/etc/hosts`). Trên Windows, nó nằm ở `C:\Windows\System32\drivers\etc\hosts`. Dòng bạn cần thêm vào file này sẽ có dạng như dưới:

    192.168.10.10  homestead.app

Hãy chắc chắn rằng IP address được liệt kê là địa chỉ có trong file `~/.homestead/Homestead.yaml` của bạn. Sau khi bạn đã thêm tên miền vào file `host` và chạy Vagrant, bạn sẽ có thể kết nối đến trang của bạn thông qua web browser:

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### Chạy Vagrant Box

Sau khi chỉnh sửa `Homestead.yaml` theo ý muốn của bạn, chạy lệnh `vagrant up` từ trong thư mục Homestead. Vagrant sẽ khởi động máy ảo và tự động config folder chia sẻ và trang Ngix.

Để huỷ máy ảo, bạn có thể sử dụng lệnh `vagrant destroy --force`.

<a name="per-project-installation"></a>
### Cài đặt cho từng dự án

Thay vì cài đặt Homestead một cách toàn cục và chia sẻ Homestead giống nhau cho tất cả các dự án, thì bạn có thể cấu hình từng Homestead cho từng dự án. Cài đặt Homested cho từng dự án có lợi ích nếu bạn muốn gửi kèm `Vagrantfile` với dự án của bạn, cho phép người khác làm việc trên dự án đơn giản với lệnh `vagrant up`.

Để cài Homestead trực tiếp vào dự án, sử dụng Composer:

    composer require laravel/homestead --dev

Sau khi Homestead đã được cài đặt, sử dụng lệnh `make` để tạo file `Vagrantfile` và `Homestead.yaml` trong thư mục gốc của dự án. Lệnh `make` sẽ tự động config `sites` và `folders` vào file `Homestead.yaml`.

Mac / Linux:

    php vendor/bin/homestead make

Windows:

	vendor\\bin\\homestead make

Tiếp theo, chạy lệnh `vagrant up` từ terminal và truy cập vào dự án tại `http://homestead.app` từ trình duyệt. Nhớ rằng, bạn sẽ vẫn cần phải thêm `homestead.app` hoặc tên miền mà bạn muốn vào file `/etc/hosts`.

<a name="installing-mariadb"></a>
### Cài đặt MariaDB

Nếu bạn muốn dùng MariaDB thay vì MySQL, bạn có thể thêm tuỳ chọn `mariadb` vào file `Homestead.yaml`. Lựa chọn này sẽ bỏ MySQL và cài MariaDB. MariaDB sẽ hoạt động thay thế cho MySQL nên bạn vẫn có thể dùng `mysql` database driver trong cấu hình database của ứng dụng:

    box: laravel/homestead
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="daily-usage"></a>
## Cách sử dụng thường nhật

<a name="accessing-homestead-globally"></a>
### Truy cập Homestead một cách toàn cục

Đôi khi bạn muốn chạy `vagrant up` để khởi động máy ảo Homestead tại bất cứ đâu trong hệ thống. Bạn có thể làm điều đó bằng cách thêm function Bash đơn giản vào Bash profile. Function này cho phép bạn chạy bất cứ lệnh Vagrant nào tại bất cứ đâu trong hệ thống và nó sẽ tự động chuyển command đấy về nơi mà Homestead được cài đặt:

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

Hãy chắc chắn rằng bạn đã chỉnh đường dẫn `~/Homestead` đến nơi thực tế Homestead được cài đặt. Một khi function đã được cài đặt, bạn có thể chạy lệnh như `homestead up` hoặc `homestead ssh` tại bất cứ đâu trong hệ thống của bạn.

<a name="connecting-via-ssh"></a>
### Kết nối thông qua SSH

Bạn có thể kết nối SSH vào máy ảo bằng cách sử dụng lệnh `vagrant ssh` từ thư mục Homestead.

Tuy nhiên, có thể bạn sẽ muốn connect SSH đến máy ảo thường xuyên, nên bạn hãy cân nhắc việc thêm "function" được mô tả phía trên đến máy của bạn để có thể kết nối SSH đến hộp Homestead một cách nhanh chóng.

<a name="connecting-to-databases"></a>
### Kết nối đến cơ sở dữ liệu

Một cơ sở dữ liệu `homestead` đã được config sẽ cho cả MySQL và Postgres. Để thuận tiện hơn, file `.env` của Laravel cấu hình framework để có thể kết nối đến cơ sở dữ liệu.

Để kết nối đến cơ sở dữ liệu MySQL hay Postgres từ máy của bạn thông qua Navicat hay Sequel Pro, bạn nên kết nối đến `127.0.0.1` và port `33060` (MySQL) hoặc `54320` (Postgres). Username và password cho cả 2 loại database là `homestead` / `secret`.

> **Note:** Bạn chỉ nên sử dụng những cổng không tiêu chuẩn khi kết nối đến cơ sở dữ liệu từ máy của bạn. Bạn sẽ dùng những cổng mặc định 3306 và 5432 trong cấu hình database cho dự án Laravel bởi vì Laravel thường được chạy trong máy ảo.

<a name="adding-additional-sites"></a>
### Thêm trang bổ sung

Một khi môi trường Homestead của bạn đã được cung cấp và hoạt động, bạn có thể muốn thêm các trang bổ sung Nginx cho các ứng dụng Laravel. Bạn có thể chạy nhiều ứng dụng Laravel như bạn muốn trên cùng một môi trường Homestead. Để thêm các trang bổ sung, bạn đơn giản thêm trang vào file `~/.homestead/Homestead.yaml` sau đó chạy lệnh `vagrant provision` từ terminal trong thư mục Homestead.

<a name="configuring-cron-schedules"></a>
### Cấu hình lịch Cron

Laravel cung cấp phương thức thuận tiện để [lên lịch các công việc Cron](/docs/{{version}}/scheduling) bằng cách lên lịch một câu lệnh Artisan `schedule:run` để chạy mỗi phút. Câu lệnh `schedule:run` sẽ kiểm tra công việc đã được lên lịch được khai báo trong lớp `App\Console\Kernel` để quyết định xem công việc nào sẽ được thực thi.

Nếu muộn muốn lệnh `schedule:run` thực thi cho trang Homestead, bạn có thể thay đổi tuỳ chọn `schedule` thành `true` khi khai báo trang:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

Con job của trang sẽ được định nghĩa trong folder `/ect/cron/d` của máy ảo.

<a name="ports"></a>
### Cổng

Mặc định, những cổng sau sẽ được chuyển tiếp đến môi trường Homestead của bạn:

- **SSH:** 2222 &rarr; Forwards To 22
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **Postgres:** 54320 &rarr; Forwards To 5432

#### Chuyển tiếp các cổng bổ sung

Nếu bạn muốn, bạn có thể chuyển tiếp các cổng bổ sung đến hộp Vagrant, miễn là xác định được giao thức của chúng:

    ports:
        - send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp

<a name="network-interfaces"></a>
## Giao thức mạng

Thuộc tính `networks` của `Homestead.yaml` cấu hình giao thức mạng cho môi trường Homestead của bạn. Bạn có thể cấu hình nhiều giao thức theo nhu cầu:

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

Để bật giao thức [bridged](https://www.vagrantup.com/docs/networking/public_network.html), cấu hình cài đặt `bridge` và đổi loại của network sang `public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

Để bật giao thức [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), chỉ cần bỏ tuỳ chọn `ip` khỏi cấu hình:

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"
