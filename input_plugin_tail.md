# Input Plugins
Fluentd có 8 loại plugin: `Input`, `Parser`, `Filter`, `Output`, `Formatter`, `Storage`, `Service Discovery` và `Buffer`.

## Overview
Các plugin đầu vào mở rộng Fluentd để lấy và pull event logs từ các nguồn bên ngoài. Một plugin đầu vào thường tạo ra một thread socket và một listen socket. Nó cũng có thể được viết để định kỳ pull dữ liệu từ các nguồn dữ liệu.

## Danh sách các Input Plugins
`in_tail`, `in_forward`, `in_udp`, `in_tcp`, `in_unix`, `in_http`, `in_syslog`, `in_exec`, `in_dummy`, `in_windows_eventlog`.

## in_tail

`in_tail` cho phép Fluentd đọc các sự kiện từ đuôi tệp văn bản. Hành vi của nó tương tự như lệnh `tail -F`.

### Example Configuration
`in_tail` được bao gồm trong lõi của Fluentd. Không có quá trình cài đặt bổ sung được yêu cầu.
```
<source>
  @type tail
  path /var/log/httpd-access.log
  pos_file /var/log/td-agent/httpd-access.log.pos
  tag apache.access
  <parse>
    @type apache2
  </parse>
</source>
```
### How it Works
Khi Fluentd được cấu hình lần đầu tiên bằng `in_tail`, nó sẽ bắt đầu đọc từ đuôi của log đó, không phải từ đầu. Khi log được xoay, Fluentd bắt đầu đọc tệp mới từ đầu. Nó theo dõi số inode hiện tại.

Nếu td-agent khởi động lại, nó bắt đầu đọc từ vị trí cuối cùng td-agent đọc trước khi khởi động lại. Vị trí này được ghi trong tệp vị trí được chỉ định bởi tham số `pos_file`.

# Parameters
#### @type (required)
Giá trị phải là `tail`.

**tag**

+ type: string
+ default: required pararameter

Các thẻ của sự kiện.

`*` có thể được sử dụng như một trình "giữ chỗ" mở rộng đến đường dẫn tệp thực tế, thay thế "/" bằng ".". Ví dụ: nếu bạn có cấu hình sau:
```
path /path/to/file
tag foo.*
```
`in_tail` phát ra các sự kiện được phân tích cú pháp bằng thẻ "foo.path.to.file".

**path**
+ type: string
+ default: required parameter

Các đường dẫn để đọc. Nhiều đường dẫn có thể được chỉ định, phân tách bằng ','.

Định dạng `*` và `strftime` có thể được bao gồm để add/remove tệp một cách linh hoạt. Tại khoảng thời gian `refresh_interval`, Fluentd làm mới danh sách tệp theo dõi.

ví dụ:
```
path /path/to/%Y/%m/%d/*
```
Đối với nhiều đường dẫn:
```
path /path/to/a/*,/path/to/b/c.log
```

**path_timezone**

+ type: string
+ default: nil

Tham số này dành cho `strftime` đường dẫn được định dạng như /path/to/%Y/%m/%d/.

`in_tail` sử dụng múi giờ hệ thống theo mặc định. Tham số này thay đổi nó.
```
path_timezone "+00"
```

**exclude_path**

+ type: array
+ default: [] (empty)

Các đường dẫn để loại trừ các tập tin từ danh sách theo dõi. Ví dụ: nếu bạn muốn xóa các tệp nén, bạn có thể sử dụng mẫu sau.

```
path /path/to/*
exclude_path ["/path/to/*.gz", "/path/to/*.zip"]
```
`exclude_path` lấy đầu vào của nó là một mảng, không giống như `path`, nó lấy nó làm một chuỗi.

**refresh_interval**
+ type: time
+ default: 60s

Khoảng thời gian làm mới danh sách theo dõi các tập tin. Điều này được sử dụng khi đường dẫn bao gồm `*`.

**limit_recently_modified**
+ type: time
+ default: nil

Giới hạn các tệp đang xem rằng thời gian sửa đổi nằm trong phạm vi thời gian được chỉ định khi sử dụng `*` trong tham số `path`.

**skip_refresh_on_startup**
+ type: bool
+ default: false

Bỏ qua việc làm mới danh sách khi khởi động. Điều này giúp giảm thời gian khởi động khi sử dụng `*` trong đường dẫn.

**read_from_head**
+ type: bool
+ default: false

Bắt đầu đọc các bản ghi từ đầu tệp, không phải dưới cùng.

Nếu bạn muốn theo đuôi tất cả nội dung bằng đường dẫn động `*` hoặc strftime, hãy đặt tham số này thành `true`. Thay vào đó, bạn nên đảm bảo rằng xoay vòng log sẽ không xảy ra trong thư mục `*`.

Khi điều này là true, `in_tail` cố đọc một tệp trong giai đoạn khởi động. Nếu tệp mục tiêu lớn, sẽ mất nhiều thời gian và bắt đầu các plugin khác không được thực thi cho đến khi đọc xong tệp.

**multiline_flush_interval**
+ type: time
+ default: nil

Khoảng thời gian xả bộ đệm cho định dạng đa dòng.

Nếu bạn đặt `multiline_flush_interval 5s`, `in_tail` sẽ xóa sự kiện được đệm sau 5 giây kể từ lần phát cuối cùng. Tùy chọn này hữu ích khi bạn sử dụng option `format_firstline`.

**pos_file (highly recommended)**

+ type: string
+ default: nil

Thông số này rất được khuyến khích. Fluentd sẽ ghi lại vị trí mà nó đọc lần cuối vào tập tin này.

ví dụ:
```
pos_file /var/log/td-agent/tmp/access.log.pos
```

`pos_file` xử lý nhiều vị trí trong một tệp, do đó không cần nhiều tham số `pos_file` cho mỗi `source`.

Không chia sẻ `pos_file` giữa các cấu hình `in_tail`. Nó gây ra hành vi bất ngờ, ví dụ: Nội dung `pos_file` bị hỏng.

`in_tail` loại bỏ vị trí tệp không bị theo dõi trong giai đoạn khởi động. Điều đó có nghĩa là nội dung của `pos_file` đang phát triển cho đến khi khởi động lại khi bạn nối đuôi nhiều tệp với cài đặt đường dẫn động.

**<parse>**
Các định dạng của log. `in_tail` sử dụng parser plugin để phân tích log.

ví dụ:
```
# json
<parse>
  @type json
</parse>

# regexp
<parse>
  @type regexp
  expression ^(?<name>[^ ]*) (?<user>[^ ]*) (?<age>\d*)$
</parse>
```
Nếu `@type` chứa `multiline`, `in_tail` hoạt động như chế độ đa dòng.

## FAQ
### What happens when <parse> type is not matched for logs.

`in_tail` in thông báo cảnh báo. Ví dụ: nếu bạn chỉ định `@type json` trong `<parse>` và dòng log của bạn là `123,456, str, true`, thì bạn sẽ thấy thông báo sau trong fluentd log.

```
2018-04-19 02:23:44 +0900 [warn]: #0 pattern not match: "123,456,str,true"
```

**in_tail doesn't start to read log file, why?**

`in_tail` theo dõi hành vi lệnh `tail -F` theo mặc định, vì vậy `in_tail` chỉ đọc các bản ghi mới hơn. Nếu bạn muốn đọc các dòng hiện có cho trường hợp sử dụng hàng loạt, hãy đặt `read_from_head true`.

**What happens when in_tail receives BufferOverflowError?**

`in_tail` dừng đọc các dòng mới và cập nhật tệp pos cho đến khi BufferOverflowError được giải quyết. Sau khi giải quyết BufferOverflowError, hãy khởi động lại phát ra các dòng mới và cập nhật tệp pos.
