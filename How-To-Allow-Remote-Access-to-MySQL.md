# How To Allow Remote Access to MySQL

<br />

## I. Config MySQL

Một trong những vấn đề phổ biến hơn mà người dùng gặp phải khi cố gắng thiết lập cơ sở dữ liệu MySQL từ xa là phiên bản MySQL của họ chỉ được định cấu hình để lắng nghe các kết nối cục bộ. Đây là cài đặt mặc định của MySQL, nhưng nó sẽ không hoạt động khi thiết lập cơ sở dữ liệu từ xa vì MySQL phải có khả năng lắng nghe địa chỉ IP bên ngoài nơi có thể truy cập máy chủ. Để kích hoạt tính năng này, hãy mở `mysqld.cnf` tệp của bạn:

```java
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

<br />

Tìm kiếm dòng lệnh `bind-address`:

*/etc/mysql/mysql.conf.d/mysqld.cnf*
```sql
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
```sql
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

<br />

> Lưu ý: Trong một số phiên bản nhất định của MySQL, lệnh `bind-address` có thể không có trong tệp `mysqld.cnf` theo mặc định. Trong trường hợp này, hãy thêm dòng `bind-address = 0.0.0.0` vào cuối tệp.
> 
<br />

Sau khi thay đổi dòng này, hãy lưu và đóng tệp (`CTRL + X`, `Y`, sau đó `ENTER` nếu bạn chỉnh sửa nó bằng `nano`).

<br />

Sau đó khởi động lại dịch vụ MySQL để những thay đổi bạn đã thực hiện `mysqld.cnf` có hiệu lực:
```sql
sudo systemctl restart mysql
```

<br />

## II. Thêm tài khoản kết nối với MySQL từ xa

Nếu bạn hiện có tài khoản người dùng MySQL mà bạn dự định sử dụng để kết nối với cơ sở dữ liệu từ máy chủ từ xa, bạn sẽ cần phải định cấu hình lại tài khoản đó để kết nối từ máy chủ từ xa thay vì __localhost__. Để làm như vậy, hãy mở ứng dụng khách MySQL với tư cách là người dùng MySQL __root__ của bạn hoặc bằng một tài khoản người dùng đặc quyền khác:
```sql
sudo mysql
```

Nếu bạn đã bật xác thực mật khẩu cho __root__, thay vào đó, bạn sẽ cần sử dụng lệnh sau để truy cập shell MySQL:
```sql
mysql -u root -p
```

<br />

Bạn có thể tạo một tài khoản người dùng mới sẽ chỉ kết nối từ máy chủ từ xa bằng lệnh sau:
```sql
CREATE USER 'sammy'@'remote_server_ip' IDENTIFIED BY 'password';
```

- Thay `sammy` bằng tên tài khoản muốn tạo
- Thay `password` bằng mật khẩu muốn tạo
- Thay `remote_server_ip` bằng địa chỉ IP bạn muốn cho phép truy cập từ xa đến MySQL, nếu bạn không muốn giới hạn địa chỉ IP truy cập đến MySQL thì sử dụng `%`

<br />

Sau đó cấp cho người dùng mới những đặc quyền phù hợp với nhu cầu cụ thể của bạn. 
Ví dụ sau cấp cho người dùng các đặc quyền chung đối với:
- `CREATE`, `ALTER`, và `DROP` cơ sở dữ liệu, bảng và người dùng
- `INSERT`, `UPDATE` và `DELETE` dữ liệu từ bất kỳ bảng nào trên máy chủ.
- `SELECT`, tạo khóa ngoại bằng `REFERENCES` từ khóa và thực hiện `FLUSH` các thao tác với `RELOAD`.
  
Tuy nhiên, bạn chỉ nên cấp cho người dùng những quyền mà họ cần, vì vậy bạn có thể thoải mái điều chỉnh các đặc quyền của người dùng nếu cần.
```sql
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD ON *.* TO 'sammy'@'remote_server_ip' WITH GRANT OPTION;
```

Bạn cũng có thể cấp đầy đủ quyền `ALL PRIVILEGES` cho người dùng mới như sau:
```sql
GRANT ALL PRIVILEGES ON *.* TO 'sammy'@'remote_server_ip' WITH GRANT OPTION;
```

> Không nên cấp đầy đủ quyền cho người dùng mới một cách dễ dàng, vì bất kỳ ai có quyền truy cập vào MySQL này sẽ có toàn quyền kiểm soát mọi cơ sở dữ liệu trên máy chủ.

<br />

Theo đó, tốt nhất là chạy lệnh `FLUSH PRIVILEGES`. Điều này sẽ giải phóng mọi bộ nhớ mà máy chủ đã lưu vào bộ nhớ đệm do các câu lệnh `CREATE USER` trước đó `GRANT`:
```sql
FLUSH PRIVILEGES;
```

<br />

Sau đó, bạn có thể thoát khỏi MySQL client:
```sql
exit
```

<br />

Nguồn tham khảo: [How To Allow Remote Access to MySQL](https://www.digitalocean.com/community/tutorials/how-to-allow-remote-access-to-mysql)

<br />
<br />
