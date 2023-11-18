# How To Allow Remote Access to MySQL

Một trong những vấn đề phổ biến hơn mà người dùng gặp phải khi cố gắng thiết lập cơ sở dữ liệu MySQL từ xa là phiên bản MySQL của họ chỉ được định cấu hình để lắng nghe các kết nối cục bộ. Đây là cài đặt mặc định của MySQL, nhưng nó sẽ không hoạt động khi thiết lập cơ sở dữ liệu từ xa vì MySQL phải có khả năng lắng nghe địa chỉ IP bên ngoài nơi có thể truy cập máy chủ. Để kích hoạt tính năng này, hãy mở `mysqld.cnf` tệp của bạn:

```java
$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
