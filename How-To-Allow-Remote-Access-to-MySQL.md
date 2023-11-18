# How To Allow Remote Access to MySQL

<br />

Một trong những vấn đề phổ biến hơn mà người dùng gặp phải khi cố gắng thiết lập cơ sở dữ liệu MySQL từ xa là phiên bản MySQL của họ chỉ được định cấu hình để lắng nghe các kết nối cục bộ. Đây là cài đặt mặc định của MySQL, nhưng nó sẽ không hoạt động khi thiết lập cơ sở dữ liệu từ xa vì MySQL phải có khả năng lắng nghe địa chỉ IP bên ngoài nơi có thể truy cập máy chủ. Để kích hoạt tính năng này, hãy mở `mysqld.cnf` tệp của bạn:

```java
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

<br />

Tìm kiếm dòng lệnh `bind-address`:

*/etc/mysql/mysql.conf.d/mysqld.cnf*
```java
. . .
# If MySQL is running as a replication slave, this should be
# changed. Ref https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_tmpdir
# tmpdir                = /tmp
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 127.0.0.1
. . .
```

Theo mặc định, giá trị này được đặt thành `127.0.0.1`, nghĩa là máy chủ sẽ chỉ tìm kiếm các kết nối cục bộ. Bạn sẽ cần thay đổi chỉ thị này để tham chiếu địa chỉ IP bên ngoài. Với mục đích khắc phục sự cố, bạn có thể đặt lệnh này thành địa chỉ IP ký tự đại diện, `*`, `::` hoặc `0.0.0.0`:

*/etc/mysql/mysql.conf.d/mysqld.cnf*
```java
. . .
# If MySQL is running as a replication slave, this should be
# changed. Ref https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_tmpdir
# tmpdir                = /tmp
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0
. . .
```

> Lưu ý: Trong một số phiên bản nhất định của MySQL, lệnh ` bind-address` có thể không có trong tệp `mysqld.cnf` theo mặc định. Trong trường hợp này, hãy thêm dòng `bind-address = 0.0.0.0` vào cuối tệp.
