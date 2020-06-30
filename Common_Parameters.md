# common Parameters
## @type
Tham số @type là chỉ định loại plugin cho một phần.
```
<source>
  @type my_plugin_type
</source>

<filter>
  @type my_filter
</filter>
```

## @id
Tham số `@id` được sử dụng để thêm unique name của cấu hình plugin, được sử dụng cho các đường dẫn của buffer/storage, logging và các mục đích khác.
```
<match>
  @type file
  @id   service_www_accesslog
  path  /path/to/my/access.log
  # ...
</match>
```
Tham số này phải được chỉ định cho tất cả các plugin để kích hoạt tính năng `root_dir` và `workers` trên globally.

## @log_level
Tham số này là để xác định logging level dành riêng cho plugin. Log level mặc định là `info`. Global log có thể được chỉ định bởi `log_level` trong `<system>` hoặc tùy chọn `-v/-q`.
```
<system>
  log_level info
</system>

<source>
  # ...
  @log_level debug  # Hiển thị debug log chỉ cho plugin này

</source>
```
Mục đích chính của tham số này là:

+ Bỏ qua nhiều logs chỉ cho plugin đó.
+ Hiển thị debug logs trong khi gỡ lỗi plugin đó.

**Các tham số cho các plugin phát ra các sự kiện
**
## @label
Tham số `@label` dùng để định tuyến các sự kiện đầu vào đến các phần `<label>`, tập hợp các `<filter>` và các `<match>`.
```
<source>
  @type  ...
  @label @access_logs
  # ...
</source>

<source>
  @type  ...
  @label @system_metrics
  # ...
</source>

<label @access_log>
  <match **>
    @type file
    path  ...
  </match>
</label>

<label @system_metrics>
  <match **>
    @type file
    path  ...
  </match>
</label>
```
Các giá trị của tham số `@label` phải bắt đầu bằng ký tự `@`.

Chỉ định `@label` được khuyến nghị để định tuyến các sự kiện đến bất kỳ plugin nào mà không sửa đổi tags. Nó có thể làm cho toàn bộ cấu hình đơn giản.
