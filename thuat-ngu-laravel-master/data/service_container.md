## Service Container

**Service Container**: là phương pháp mà Laravel sử dụng để quản lý các dependencies trong ứng dụng. Khi các dependencies được đăng kí vào trong container, thì bất cứ khi nào có một dependency injection được thực hiện thì container sẽ thực hiện xử lý để lấy ra đối tượng được yêu cầu.

**Resolve**: chỉ việc container thực hiện query để lấy ra đối tượng liên kết đã được đăng kí vào trong container.