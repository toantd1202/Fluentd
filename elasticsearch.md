# elasticsearch
`Elasticsearch` là một công cụ tìm kiếm dựa trên nền tảng `Apache Lucene`. Nó cung cấp một bộ máy tìm kiếm dạng phân tán, có đầy đủ công cụ với một giao diện web HTTP có hỗ trợ dữ liệu JSON.
`Elasticsearch` được phát triển bằng Java và được phát hành dạng nguồn mở theo giấy phép Apache.

Plugin `out_elaticsearch`, ghi các bản ghi vào Elaticsearch. Theo mặc định, nó tạo các bản ghi bằng thao tác ghi số lượng lớn. Điều này có nghĩa là khi bạn lần đầu tiên nhập bản ghi bằng plugin, thì sẽ không có bản ghi nào được tạo ngay lập tức.

Bản ghi sẽ được tạo khi điều kiện `chunk_keys` được đáp ứng. Để thay đổi tần số đầu ra, vui lòng chỉ định `time` trong `chunk_keys` và chỉ định giá trị  `timekey` trong conf.

## Parameters
+ **@type**: Tùy chọn luôn là `elasticsearch`

+ **host(option)**: hostname của Elaticsearch được dùng (mặc định: localhost)

+ **port**: Số hiệu cổng của ES (mặc định là 9200)

+ **hosts**: Được dùng khi có nhiều hơn một node ES:
```
hosts host1:port1,host2:port2,host3:port3
# or
hosts https://customhost.com:443/path,https://username:password@host-failover.com:443
```
Nếu dùng option nay thì các option `host`, `port` sẽ bị bỏ qua.

+ **user,password**: Thông tin đăng nhập để kết nối với nút Elaticsearch (mặc định: nil)

+ **path**: Điểm cuối REST API của Elaticsearch để gửi yêu cầu ghi (mặc định: nil)

+ **index_name**: Tên chỉ mục để viết các sự kiện (mặc định: `fluentd`).

Tùy chọn này hỗ trợ cú pháp "giữ chỗ" của API plugin Fluentd. Ví dụ: nếu bạn muốn phân vùng chỉ mục theo thẻ, ta có thể chỉ định như sau:
```
index_name fluentd.${tag}
```
Ví dụ phân vùng chỉ mục Elaticsearch theo thẻ và timestamps:
```
index_name fluentd.${tag}.%Y%m%d
```

+ **@log_level**: Tùy chọn `@log_level` cho phép người dùng đặt các levels logging khác nhau cho mỗi plugin. Các mức nhật ký được hỗ trợ là: `fatal`, `error`, `warn`, `info`, `debug`, and `trace`.

+ **logstash_prefix**

### Miscellaneous
Ta có thể sử dụng trình giữ chỗ kiểu `%{}`  để mã hóa URL, các ký tự cần thiết.
```
user %{demo+}
password %{@secret}
```
hoặc:
```
hosts https://%{j+hn}:%{passw@rd}@host1:443/elastic/,http://host2
```
