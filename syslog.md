# Syslog
Plugin In_syslog Input cho phép Fluentd truy xuất các bản ghi thông qua giao thức syslog trên UDP hoặc TCP.

## Example Configuration
```
<source>
  @type syslog
  port 5140
  bind 0.0.0.0
  tag system
</source>
```

Điều này nói với Fluentd để tạo một socket nghe trên cổng 5140. Cần thiết lập deamon syslog để gửi tin nhắn đến socket. Ví dụ: nếu bạn đang sử dụng rsyslogd, hãy thêm các dòng sau vào `/etc/rsyslog.conf`.
```
# Send log messages to Fluentd
*.* @127.0.0.1:5140
```
## Example Usage
Các dữ liệu lấy được tổ chức như sau. Thẻ của Fluentd được tạo bởi tham số `tag` (tiền tố thẻ), `facility level` và `priority`. Bản ghi được phân tích cú pháp bởi các regexp.

```
tag = "#{@tag}.#{facility}.#{priority}"
time = 1353436518,
record = {
  "host": "host",
  "ident": "ident",
  "pid": "12345",
  "message": "text"
}
```
Nếu bạn muốn giữ facility và priority trong bản ghi, hãy đặt các tham số liên quan.

### Parameters 
**@type**
Giá trị phải là `syslog`.

**tag**
Thẻ được tạo bởi tiền tố `tag`, facility level và priority.

**port**
+ type: interger
+ default: 5140

**bind**
+ type: string
+ default: 0.0.0.0

**protocol_type**
+ type: enum
+ default: udp/tcp

**<transport>section**
+ type: enum
+ default: udp (udp/tcp/tls)
```
<source>
  @type syslog
  tag system
  <transport tcp>
  </transport>
  # other parameters
</source>
```

**message_length_limit**
+ type: side
+ default: 2048

### format
**<parse>**
Các định dạng của log. Tùy chọn này được sử dụng để phân tích các định dạng syslog không chuẩn bằng các trình phân tích cú pháp.
```
<source>
  @type syslog
  tag system
  <parse>
    @type FORMAT_PARAMETER
  </parse>
</source>
```
Ví dụ: nếu in_syslog nhận được nhật ký bên dưới:
```
 <1>Feb 20 00:00:00 192.168.0.1 fluentd[11111]: [error] hogehoge
```
Sau đó trình phân tích cú pháp định dạng nhận được log sau:

```
 Feb 20 00:00:00 192.168.0.1 fluentd[11111]: [error] hogehoge
```

**with_priority**
+ type: bool
+ default: true
Tham số này được sử dụng bên trong chỉ thị `<parse>`.
Nếu `with_priority` là true, thì các thông báo syslog được giả sử là có tiền tố với thẻ prority như <3>. Tùy chọn này tồn tại do một số log đầu ra syslog daemon không có thẻ priority trước nội dung thư.

Nếu bạn muốn phân tích các thông báo syslog có định dạng tùy ý, `in_tcp` hoặc `in_udp` được khuyến nghị.

### ký hệ thống trong một kết nối TCP theo mặc định. 
Plugin này giả định `\n` cho ký tự phân cách giữa các bản syslog trong một kết nối TCP theo mặc định. Nếu sử dụng thư viện syslog trong ứng dụng của mình với phương tiện truyền thông tcp, hãy thêm \n vào tin nhắn nhật ký hệ thống của bạn.

Nếu syslog của bạn sử dụng chế độ đếm octet, hãy đặt `frame_type octet_count` trong cấu hình `in_syslog`.
