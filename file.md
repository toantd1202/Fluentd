# file

Plugin `out_file` ghi các sự kiện vào các tệp (khoảng 00:10). Điều này có nghĩa là khi bạn nhập bản ghi đầu tiên bằng cách sử dụng plugin, không có tệp nào được tạo ngay lập tức.

Tập tin sẽ được tạo khi điều kiện `timekey` được đáp ứng. Để thay đổi tần số đầu ra, phải sửa đổi giá trị `timekey`.

## Example Configuration
`Out_file` có trong lõi của Fluentd. Không có quá trình cài đặt bổ sung được yêu cầu.

```
<match pattern>
  @type file
  path /var/log/fluent/myapp
  compress gzip
  <buffer>
    timekey 1d
    timekey_use_utc true
    timekey_wait 10m
  </buffer>
</match>
```

### Parameters

**@type**
Giá trị phải là `file`.

**path**
+ type: string
+ default: yêu cầu tham số đầu vào

Đường dẫn của tệp. Đường dẫn thực tế là đường dẫn + thời gian + '.log' theo mặc định. Tham số đường dẫn hỗ trợ giữ chỗ, vì vậy bạn có thể nhúng các trường thời gian, thẻ và ghi lại trong đường dẫn. Đây là một ví dụ:
```
path /path/to/${tag}/${key1}/file.%Y%m%d
<buffer tag,time,key1>
  # buffer parameters
</buffer>
```
Tham số `path` được sử dụng làm đường dẫn của <buffer> trong plugin này.

Ban đầu, bạn có thể thấy một tệp trông giống như /path/to/file.%Y%m%d/buffer.b5692238db04045286097f56f361028db.log. Đây là một tệp bộ đệm trung gian (b5692238db04045286097f56f361028db xác định bộ đệm). Khi nội dung của bộ đệm đã được xóa hoàn toàn, bạn sẽ thấy tệp đầu ra mà không cần định danh theo dõi.

**append**
+ type: bool
+ default: false
Đoạn dữ liệu được gắn vào tệp tồn tại hay không. Mặc định không được thêm vào. Theo mặc định, `out_file` đẩy ra từng đoạn đến đường dẫn khác nhau.
```
# append false
file.20140608.log_0
file.20140608.log_1
file.20140609.log_0
file.20140609.log_1
```
Điều này giúp cho việc xử lý tập tin song song dễ dàng. Nhưng nếu bạn muốn vô hiệu hóa hành vi này, bạn có thể vô hiệu hóa nó bằng cách đặt `append true`.
```
# append true
file.20140608.log
file.20140609.log
```

**<format> directive**
Các định dạng của nội dung tập tin. `@type` mặc định là `out_file`.
ví dụ: json
```
<format>
  @type json
</format>
```

**<inject> section**
Thêm sự kiện `time` và `tag` sự kiện để ghi lại.

**compress**
Nén các tệp được đẩy ra bằng `gzip`. Không có nén được thực hiện theo mặc định.

**recompress**
Thực hiện nén lại ngay cả khi phần đệm đã được nén. Mặc định là `false`.

**symlink_path**
Tạo symlink đến tập tin đệm tạm thời khi `buffer_type` là `file`. Không có symlink được tạo theo mặc định. Điều này rất hữu ích cho việc theo dõi nội dung tệp để kiểm tra log.

**@log_level option**
Tùy chọn `@log_level` cho phép người dùng đặt các log levela khác nhau cho mỗi plugin. Các log level được hỗ trợ là: fatal, error, warn, info, debug, and trace.
