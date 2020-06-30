# Fluentd là gì?
Với những hệ thống lớn việc quản lý log và phân loại log bằng việc xem file log của server để xác định thông tin của log, phân loại log là khá khó khăn. Cần thiết phải có một công cụ quản lý log một cách tốt hơn, sớm phát hiện những lỗi phát sinh của server hoặc kiểm tra các thông tin về log. Hiện nay cũng có khá nhiều công cụ để quản lý log khác nhau. 

+ "Fluentd" is an open-source tool to collect events and logs.
+ Fluentd là một phần mềm trung gian tuyệt vời dựa trên Ruby dùng để đọc, xử lý và gửi log.
+ Fluentd cũng có thể được mở rộng bằng cách viết plugin, hiện nay có nhiều plugin đã được develop và public.

Fluentd được cấp phép theo các điều khoản của Apache License v2.0. Dự án này được thực hiện và tài trợ bởi Treasure Data.

# Kiến trúc
Fluentd coi các bản ghi là JSON, một định dạng phổ biến có thể đọc được bằng máy. Nó được viết chủ yếu bằng C với các phần về hiệu năng, Ruby cũng được sử dụng chút ít giúp người dùng linh hoạt.
![Screenshot from 2020-06-23 15-37-19](https://user-images.githubusercontent.com/61723456/85389819-203d2a00-b572-11ea-87a4-f0d43cd50999.png)

### Hoạt động của bộ công cụ: Fluentd, Elasticsearch, Kibana
![mô hình EFK](https://user-images.githubusercontent.com/61723456/86105185-3a828500-bae9-11ea-811c-472c1ea4d344.png)

+ Đầu tiên, log sẽ được đưa đến Fluentd. (Ví dụ như log access server nginx/apache, log do develop setting trong source php/java vv. miễn là có ghi ra file log).
+ Fluentd sẽ đọc những log này, thêm những thông tin như thời gian, IP, parse dữ liệu từ log (server nào, độ nghiêm trọng, nội dung log) ra, sau đó ghi xuống database là Elasticsearch.
+ Khi muốn xem log, người dùng vào URL của Kibana. Kibana sẽ đọc thông tin log trong Elasticsearch, hiển thị lên giao diện cho người dùng query và xử lý.
+ Log có nhiều loại (tag), do developer định nghĩa chẳng hạn access_log error_log, peformance_log, api_log. Khi Fluentd đọc log và phân loại từng loại log rồi gửi đến Elasticsearch, ở giao diện Kibana chúng ta chỉ cần add một số filter như access_log chẳng hạn và query.


# Fluentd

# Cài đặt Fluentd với Docker
- Download and install Docker

**Step 1: Pull Fluentd's Docker image**
 ```
	$ docker pull fluent/fluentd:v0.12-debian
 ```

**Step 2: Launch Fluentd Container**
Thử nghiệm đơn giản, tạo cấu hình ví dụ tại `/tmp/fluentd.conf`. Ví dụ này chấp nhận các bản ghi từ http và xuất ra thiết bị xuất chuẩn.
```
# /tmp/fluentd.conf
<source>
  @type http
  port 9880
  bind 0.0.0.0
</source>
<match **>
  @type stdout
</match>
```
Khởi chạy Fluentd với lệnh `docker run`.
```
$ docker run -d \
  -p 9880:9880 -v /tmp:/fluentd/etc -e FLUENTD_CONF=fluentd.conf \
  fluent/fluentd
```

**Step3: Post Sample Logs via HTTP**
Hãy post sample logs qua HTTP và xác nhận nó đang hoạt động. Sử dụng lệnh `curl`
```
$ curl -X POST -d 'json={"json":"message"}' http://localhost:9880/sample.test
```
![Screenshot from 2020-06-23 14-09-21](https://user-images.githubusercontent.com/61723456/85390257-c0934e80-b572-11ea-97f9-07adebf2d6cb.png)

Sử dụng lệnh `docker ps` để truy xuất ID container và sử dụng lệnh `docker logs` để kiểm tra nhật ký của container cụ thể.

```
	$ docker logs container id | tail -n 1
```

**docker-compose**
```
version: '3'
services:
  fluentd:
    image: fluent/fluentd:v0.12-debian-1
    volumes:
      - ./fluentd/conf:/fluentd/etc
    ports:
      - "9880:9880"
      - "9880:9880/udp"
```
![Screenshot from 2020-06-23 14-08-59](https://user-images.githubusercontent.com/61723456/85390287-cf7a0100-b572-11ea-8cda-af5465e8a6df.png)

# Cấu hình của Fluentd
## Config File Syntax
Thời gian tồn tại của một Fluentd Event

Tệp cấu hình cho phép người dùng kiểm soát hành vi đầu vào và đầu ra của Fluentd bằng cách (1) chọn các plugin đầu vào và đầu ra, (2) chỉ định các tham số plugin. Tập tin được yêu cầu để Fluentd hoạt động đúng.

Với Docker container, vị trí mặc định được đặt tại /fluentd/etc/fluent.conf. Để mount tệp cấu hình từ bên ngoài Docker, sử dụng bind-mount.
```
docker run -ti --rm -v /path/to/dir:/fluentd/etc fluentd -c /fluentd/etc/<conf-file> -v
```

Tệp cấu hình bao gồm các chỉ thị sau:
1. **source** xác định các nguồn đầu vào.
2. **match** xác định các điểm đến của output.
3. **filter** xác định các pipelines xử lý sự kiện.
4. **system** chỉ thị thiết lập cấu hình hệ thống rộng.
5. **label** chỉ thị nhóm đầu ra và bộ lọc cho định tuyến nội miền.
6. **@include** chỉ thị bao gồm các tập tin khác.

### "source": where all the data come from.
Các nguồn đầu vào của Fluentd được bật bằng cách chọn và định cấu hình các plugin đầu vào mong muốn bằng cách sử dụng các chỉ thị nguồn. Các plugin đầu vào tiêu chuẩn của Fluentd bao gồm `http` và `forward`. `http` biến fluentd thành điểm cuối HTTP để chấp nhận các tin nhắn HTTP đến trong khi `forword` biến fluentd thành điểm cuối TCP để chấp nhận các gói TCP.

```
# Nhận các events từ 24224/tcp
# Sử dụng log forwarding và fluent-cat command
<source>
  @type forward
  port 24224
</source>

# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>
```
![Screenshot from 2020-06-23 17-04-54](https://user-images.githubusercontent.com/61723456/85390952-baea3880-b573-11ea-8326-47234a571593.png)

Mỗi **source** phải bao gồm một tham số `@type`. Tham số `@type` chỉ định plugin đầu vào nào sẽ sử dụng.

**Interlude: Routing**
`source` gửi các sự kiện vào công cụ định tuyến của Fluentd. Một sự kiện bao gồm ba thực thể: **tag**, **time** và **record**. `Tag` là một chuỗi được phân tách bằng `.` (Ví dụ: myapp.access) và được sử dụng làm chỉ đường cho công cụ định tuyến nội bộ của Fluentd. Trường thời gian được chỉ định bởi các plugin đầu vào và nó phải ở định dạng thời gian Unix. Bản ghi là một đối tượng JSON.

Fluentd chấp nhận tất cả các ký tự không phải là một phần của `tag`. Tuy nhiên, vì đôi khi `tag` được sử dụng trong một ngữ cảnh khác bởi các đích đầu ra (ví dụ: tên bảng, tên cơ sở dữ liệu, tên khóa, v.v.), nên sử dụng bảng chữ cái chữ thường, chữ số và dấu gạch dưới, vd: `^[a-z0-9 _]+$`.

Trong ví dụ trên, plugin đầu vào HTTP gửi sự kiện sau:
```
# generated by http://this.host:9880/myapp.access?json={"event":"data"}
tag: myapp.access
time: (current time)
record: {"event":"data"}
```
Có thể thêm các nguồn đầu vào mới bằng cách viết các plugin  riêng. Để biết thêm thông tin về các nguồn đầu vào của Fluentd, vui lòng tham khảo bài viết Tổng quan về Plugin đầu vào.

### "match": Tell fluentd what to do!
Lệnh 'match' tìm kiếm các sự kiện với các matching tags và xử lý chúng. Việc sử dụng phổ biến nhất của lệnh match là để xuất các sự kiện cho các hệ thống khác (vì lý do này, các plugin tương ứng với lệnh khớp được gọi là 'plugin đầu ra'). Các plugin đầu ra tiêu chuẩn của Fluentd bao gồm `file` và `forward`. Nó được  thêm vào tập tin cấu hình.
```
# Match events tagged with "myapp.access" and
# store them to /var/log/fluent/access.%Y-%m-%d
# Of course, you can control how you partition your data
# with the time_slice_format option.
<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```
Mỗi `match` phải bao gồm một match pattern (mẫu) và tham số @type. Chỉ các sự kiện có tag matching với mẫu sẽ được gửi đến đích đầu ra (trong ví dụ trên, chỉ các sự kiện có thẻ 'myapp.access' được matched. Tham số @type chỉ định plugin đầu ra sẽ sử dụng.

Giống như các nguồn đầu vào, ta có thể thêm các đích đầu ra mới bằng cách viết các plugin của riêng.

### "filter": Event processing pipeline
Lệnh `filter` có cú pháp tương tự như `match` nhưng `filter` có thể được kết nối để xử lý pipeline. Sử dụng filters, luồng sự kiện như dưới đây:
```
Input -> filter 1 -> ... -> filter N -> Output
```
Ví dụ thêm bộ lọc `record_transformer` vào ví dụ `match`.
```
# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>

<filter myapp.access>
  @type record_transformer
  <record>
    host_param "#{Socket.gethostname}"
  </record>
</filter>

<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```
Sự kiện đã nhận, `{'event': 'data'}`, trước tiên chuyển đến bộ lọc `record_transformer`. `Record_transformer` thêm trường 'host_param' vào sự kiện và sự kiện được lọc, {'event': 'data', 'host_param': 'webserver1'}, đi đến đầu ra tệp.

### Đặt cấu hình toàn hệ thống: `system`
Các cấu hình toàn hệ thống được đặt theo chỉ thị `system`. Hầu hết trong số chúng cũng có sẵn thông qua các tùy chọn dòng lệnh. Ví dụ: các cấu hình sau đây có sẵn:
+ log_level
+ suppress_repeated_stacktrace
+ emit_error_log_interval
+ suppress_config_dump
+ without_source
+ process_name (only available in system directive. No fluentd
option)

ví dụ:  process_name
Nếu được đặt tham số này, tên fluentd's supervisor và worker sẽ  được thay đổi.

### Group filter and output: the "label" directive
Nhóm chỉ thị `label` lọc và đầu ra cho định tuyến nội miền. `label` làm giảm sự phức tạp của việc xử lý tag.

`lable` hữu ích cho phân tách luồng sự kiện mà không có tiền tố thẻ.

### Re-use your config: the "@include" directive
Chỉ thị trong các tệp cấu hình riêng biệt có thể được nhập bằng chỉ thị @include:
```
# Include config files in the ./config.d directory
@include config.d/*.conf
```
Lệnh @include hỗ trợ đường dẫn tệp thông thường, mẫu toàn cục và các quy ước URL http:
```
# absolute path
@include /path/to/config.conf

# if using a relative path, the directive will use
# the dirname of this config file to expand the path
@include extra.conf

# glob match pattern
@include config.d/*.conf

# http
@include http://example.com/fluent.conf
```

## Check configuration file
Có thể kiểm tra cấu hình của mình mà không cần plugin bắt đầu bằng cách chỉ định tùy chọn --dry-run.
```
$ fluentd --dry-run -c fluent.conf
```
