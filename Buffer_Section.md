# Buffer Section
Các plugin đầu ra của Fluentd hỗ trợ phần bộ đệm để chỉ định cách đệm các sự kiện (được xử lý bởi lõi Fluentd) và các tham số khác cho các plugin đệm.

## Overview
Phần đệm phải nằm trong phần <match>. Nó được kích hoạt cho các plugin đầu ra hỗ trợ các tính năng đệm cho đầu ra.
```
<match tag.*>
  @type file

  # ... parameters for output plugin

  <buffer>
    # buffer section parameters ...
  </buffer>

  # <buffer> section can be configured just once
</match>
```
## buffer plugin type
<buffer> chấp nhận tham số `@type` để chỉ định loại plugin đệm.
```
<buffer>
  @type file
</buffer>
```
Fluentd chứa `memory` và `file`: `buf_file Plugin`, `buf_memory Plugin`

`@type` có thể được bỏ qua. Khi tham số `@type` không được định cấu hình, bộ đệm plugin mặc định được chỉ định bởi plugin đầu ra sẽ được sử dụng (nếu có thể) hoặc plugin `memory` đệm theo mặc định.

Sử dụng bộ đệm file giúp cho độ bền cao hơn.

## Chunk keys
Các plugin đầu ra tạo ra các khối đệm bằng cách thu thập các sự kiện. Các `chunk key`, được chỉ định làm đối số của <buffer>, kiểm soát cách thu thập các sự kiện thành các chunks.
```
<buffer ARGUMENT_CHUNK_KEYS>
  # ...
</buffer>
```
Giá trị hợp lệ của các khóa chunk đối số là các chuỗi được phân tách bằng dấu phẩy hoặc để trống.

## Blank chunk keys
Khi các chunk key trống được chỉ định (và plugin đầu ra không chỉ định các chunk key mặc định), plugin đầu ra ghi tất cả các sự kiện phù hợp vào một chunk, cho đến khi kích thước của nó đầy.
```
<match tag.**>
  # ...
  <buffer>
    # ...
  </buffer>
</match>

# No chunk keys: All events will be appended into the same chunk.

11:59:30 web.access {"key1":"yay","key2":100}  --|
                                                 |
12:00:01 web.access {"key1":"foo","key2":200}  --|---> CHUNK_A
                                                 |
12:00:25 ssh.login  {"key1":"yay","key2":100}  --|
```

## Tag
Khi `tag` được chỉ định làm buffer chunk key, plugin đầu ra ghi các sự kiện thành các chunks riêng biệt cho mỗi tags. Các sự kiện với các tags khác nhau sẽ được viết thành nhiều chunks khác nhau.
```
<match tag.**>
  # ...
  <buffer tag>
    # ...
  </buffer>
</match>

# Tag chunk key: events will be separated per tags

11:59:30 web.access {"key1":"yay","key2":100}  --|
                                                 |---> CHUNK_A
12:00:01 web.access {"key1":"foo","key2":200}  --|

12:00:25 ssh.login  {"key1":"yay","key2":100}  ------> CHUNK_B
```
## Time
Khi `time` và `timekey` trong phần đệm (bắt buộc) được chỉ định, plugin đầu ra ghi các sự kiện thành các khối riêng biệt cho mỗi time key. Time key được tính là time(unix time)/timekey(seconds).


+ timekey 60: ["12:00:00", ..., "12:00:59"], ["12:01:00", ..., "12:01:59"], ...
+ timekey 180: ["12:00:00", ..., "12:02:59"], ["12:03:00", ..., "12:05:59"], ...
+ timekey 3600: ["12:00:00", ..., "12:59:59"], ["13:00:00", ..., "13:59:59"], ...

Vì vậy, các sự kiện sẽ được phân tách thành các khối theo phạm vi thời gian và sẽ được xóa (được ghi) bởi plugin đầu ra sau khi hết hạn cho phạm vi khóa thời gian.
```
<match tag.**>
  # ...
  <buffer time>
    timekey      1h # chunks per hours ("3600" also available)
    timekey_wait 5m # 5mins delay for flush ("300" also available)
  </buffer>
</match>

# Time chunk key: events will be separated for hours (by timekey 3600)

11:59:30 web.access {"key1":"yay","key2":100}  ------> CHUNK_A

12:00:01 web.access {"key1":"foo","key2":200}  --|
                                                 |---> CHUNK_B
12:00:25 ssh.login  {"key1":"yay","key2":100}  --|
```
`timekey_wait` là để xác định khi nào các chunks sẽ được làm sạch. Thời gian sự kiện thường là thời gian trễ từ dấu thời gian hiện tại, vì vậy Fluentd sẽ chờ để xóa các khối đệm cho các sự kiện bị trì hoãn. Độ trễ được định cấu hình qua `timekey_wait`. Ví dụ, hình dưới đây cho thấy khi các khối (timekey: 3600) thực sự sẽ bị xóa, đối với một số giá trị `timekey_wait`. Giá trị mặc định của `timekey_wait` là 600 (10 phút).

```
 timekey: 3600
 -------------------------------------------------------
 time range for chunk | timekey_wait | actual flush time
  12:00:00 - 12:59:59 |           0s |          13:00:00
  12:00:00 - 12:59:59 |     60s (1m) |          13:01:00
  12:00:00 - 12:59:59 |   600s (10m) |          13:10:00
```

## Other keys

Khi các khóa khác (không phải thời gian/tag) được chỉ định, chúng được xử lý như tên trường của các bản ghi. Plugin đầu ra sẽ phân tách các sự kiện thành các khối theo giá trị của các trường này.

```
<match tag.**>
  # ...
  <buffer key1>
    # ...
  </buffer>
</match>

# Chunk keys: events will be separated by values of "key1"

11:59:30 web.access {"key1":"yay","key2":100}  --|---> CHUNK_A
                                                 |
12:00:01 web.access {"key1":"foo","key2":200}  -)|(--> CHUNK_B
                                                 |
12:00:25 ssh.login  {"key1":"yay","key2":100}  --|
```

Bạn có thể sử dụng cú `record_accessor syntax` để sử dụng trường lồng nhau. Đây là một ví dụ:
```
match tag.**>
  # ...
  <buffer $.nest.field> # access record['nest']['field']
    # ...
  </buffer>
</match>
```

## Combination of chunk keys

Các chunk k đệm có thể được chỉ định 2 hoặc nhiều khóa - các sự kiện sẽ được phân tách thành các khối bằng cách kết hợp các giá trị của các chunk keys.

```
# <buffer tag,time>

11:58:01 ssh.login  {"key1":"yay","key2":100}  ------> CHUNK_A

11:59:13 web.access {"key1":"yay","key2":100}  --|
                                                 |---> CHUNK_B
11:59:30 web.access {"key1":"yay","key2":100}  --|

12:00:01 web.access {"key1":"foo","key2":200}  ------> CHUNK_C

12:00:25 ssh.login  {"key1":"yay","key2":100}  ------> CHUNK_D
```
Không có giới hạn chắc chắn về số lượng các chunk key, nhưng quá nhiều chunk key có thể làm giảm hiệu suất I/O và / hoặc tăng tổng mức sử dụng tài nguyên.
## Empty keys
 
Các chunk keys đệm có thể được empy bằng cách chỉ định [] trong phần đệm.
```
<match tag.**>
  # ...
  <buffer []>
    # ...
  </buffer>
</match>
```
Điều này rất hữu ích khi plugin đầu ra có các chunk key mặc định và vô hiệu hóa nó.

## Parameters

### Argument

Đối số là một mảng các chunk key, chuỗi này được phân tách bằng dấu phẩy. Blank cũng có sẵn.

```
<buffer>
  # ...
</buffer>

<buffer tag, time, key1>
  # ...
</buffer>
```
`tag` và `time` là thẻ và thời gian, không phải tên trường của bản ghi. Những thứ khác là để tham khảo các bản ghi.

Khi `time` được chỉ định, các tham số bên dưới có sẵn:
+ `timekey` [time]: Plugin đầu ra sẽ xóa các chunks theo thời gian được chỉ định (được bật khi `time` được chỉ định trong các chunk keys)

+ `timekey_wait`: mặc định 600s.
  + Plugin đầu ra ghi các chunks sau `timekey_wait` giây sau khi hết `timekey`
  + Nếu người dùng định cấu hình `timekey 60m`, plugin đầu ra sẽ chờ các sự kiện bị trì hoãn cho time key bị xóa và viết đoạn mã vào lúc 10 phút mỗi giờ.
+timekey_use_utc[bool], timekey_zone[sting]

## @type

`@type` là chỉ định loại plugin đệm. Loại mặc định là `memory`, nhưng nó có thể bị ghi đè bởi các cài đặt plugin đầu ra. Ví dụ: plugin bộ đệm mặc định là `file` cho plugin đầu ra tệp.
```
<buffer>
  @type file
  # ...
</buffer>
```

## Buffering parameters

Các tham số dưới đây là để cấu hình các plugin đệm và chunks của nó.
+ `chunk_limit_side`[side]: mặc định 8MB(memory)/256MB(file)
+ `chunk_limit_records`[side]: mặc định 512MB(memory)/64GB(file), khi tổng kích thước của bộ đệm được lưu trữ đạt đến ngưỡng này, tất cả các hoạt động chắp thêm sẽ không có lỗi (và dữ liệu sẽ bị mất)
+ `queue_limit_length`[interger], mặc định `nil`
+ `chunk_full_threshold`[float]: mặc định 0.95. Tỷ lệ ngưỡng kích thước khối để xả. Plugin đầu ra sẽ đưa ra chunk khi kích thước thực tế đạt tới `chunk_limit_size * chunk_full_threshold(== 8MB * 0.95)`
+ queued_chunks_limit_size [interger]: mặc định 1, giới hạn số lượn hàng, nếu bạn đặt flush_interval nhỏ hơn, ví dụ: 1s, có rất nhiều khối nhỏ xếp hàng trong bộ đệm. Điều này không tốt với bộ đệm tệp vì nó tiêu tốn nhiều tài nguyên fd khi đích đầu ra có vấn đề. Tham số này giảm nhẹ các tình huống như vậy.
+ compress[enum: text/gzip]: default text, nếu đặt tùy chọn này thành gzip, ta có thể lấy Fluentd để nén các bản ghi dữ liệu trước khi ghi vào các phần đệm.
Fluentd sẽ tự động giải nén các khối được nén này trước khi chuyển chúng đến plugin đầu ra (Trường hợp đặc biệt là khi plugin đầu ra có thể chuyển dữ liệu ở dạng nén. Trong trường hợp này, dữ liệu sẽ được chuyển đến plugin như hiện tại).

## Flushing parameters

Các tham số này là để định cấu hình cách xóa các khối để tối ưu hóa hiệu suất (độ trễ và thông lượng).
+ flush_at_shutdown[bool]: Mặc định là false cho các bộ đệm ổn định (ví dụ: buf_file), true cho các bộ đệm không ổn định (ví dụ: buf_memory). Giá trị để chỉ định flush/write tất cả các khối đệm khi tắt máy, hoặc không.
+ flush_mode [enum: default/lazy/interval/immediate].
  + Mặc định: default (bằng với `lazy` nếu `time` được chỉ định là chunk key, `interval` trường hợp khác).
  + `lazy`: flush/write chunk một lần mỗi timekey.
  + `interval`: flush/write chunks trên mỗi thời gian được chỉ định thông qua. `flush_interval`
  + `immediate`: flush/write chunk ngay sau khi các sự kiện được thêm vào khối.
+ flush_interval [time]: mặc định 60s
+ flush_thread_count [integer]: mặc định 1, số lượng luồng của các plugin đầu ra, được sử dụng để viết các đoạn song song.
+ flush_thread_interval [float]: default 1, Khoảng thời gian sleep giây của các luồng để chờ bản dùng thử tiếp theo (khi không có đoạn nào đang chờ)
## Retries parameters

+ retry_timeout [time]: mặc định 72h, Số giây tối đa để thử lại để xóa trong khi không thành công, cho đến khi plugin loại bỏ các phần đệm.
+ retry_forever [bool]: mặc định false, nếu đúng, plugin sẽ bỏ qua các tùy chọn retry_timeout và retry_max_times và retry flushing forever
+ retry_max_times [integer]: mặc định none, số lần tối đa để thử lại để xả trong khi không thành công.
+ retry_secondary_threshold [float]: default 0.8
+ retry_type [enum: exponential_backoff/periodic]: Default: exponential_backoff
+ retry_wait [time]: mặc định 1s.
+ retry_exponential_backoff_base [float]: default 2
+ retry_max_interval [time]: mặc định none. Khoảng thời gian tối đa giây để lùi lại theo cấp số nhân giữa các lần thử lại trong khi thất bại
+ retry_randomize [bool]: mặc định true. Nếu đúng, plugin đầu ra sẽ retry sau khoảng thời gian ngẫu nhiên không thực hiện thử lại cụm.
+ disable_chunk_backup [bool]: mặc định false. Thay vì lưu trữ các khối không thể phục hồi trong thư mục sao lưu, chỉ cần loại bỏ chúng.
