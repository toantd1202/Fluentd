# Parse section
## Overview
Phần phân tích có thể trong các phần <source>, <match> hoặc <filter>. Nó được kích hoạt cho các plugin hỗ trợ các tính năng của trình phân tích cú pháp.
```
<source>
  @type tail
  # parameters for input plugin
  <parse>
    # parse section parameters
  </parse>
</source>
```
## parser plugin type
`<parse>` yêu cầu tham số `@type` để chỉ định loại. Fluentd hỗ trợ rất nhiều plugin phân tích cú pháp hữu ích. Plugin bên thứ 3 cũng có sẵn khi cài đặt.
```
<parse>
  @type apache2
</parse>
```

## Parameters
`@type` là chỉ định loại plugin phân tích cú pháp.
```
<parse>
  @type regexp
  # ...
</parse>
```
Các trình phân tích cú pháp được tích hợp theo mặc định như: `regexp`, `apache2`, `apache_error`, `nginx`, `syslog`, `csv`, `tsv`, `ltsv`, `json`, `multiline`, `none`.

### Parse parameters
+ **types**(hash)(optional): Chỉ định loại để chuyển đổi trường thành loại khác.
  + Mặc định: `nil`
  + string-based hash:
	`field1:type, field2:type, field3:type:option, field4:type:option`
  + JSON format:
	`{"field1":"type", "field2":"type", "field3":"type:option", "field4":"type:option"}`
example: types user_id:integer,paid:bool,paid_usd_amount:float

+ **time_key**(string)(optional): Chỉ định trường thời gian cho thời gian sự kiện. Nếu sự kiện không có trường này, thời gian hiện tại được sử dụng.
  + Mặc định: `nil`
  + Lưu ý rằng `json`, `ltsv` và `regexp` ghi đè giá trị mặc định của tham số này và đặt nó thành `time` theo mặc định.
+ **null_value_pattern**(string)(optional): Chỉ định mẫu giá trị null.
  + Mặc định: `nil`
+ **null_empty_string**(bool)(optional): Nếu `true`, trường chuỗi rỗng được thay thế bằng nil.
  + Mặc định: `false`
+ **keep_time_key**(bool)(optional): Nếu `true`, giữ trường thời gian trong bản ghi.
  + Mặc định: `false`
+ **timeout**(time)(optional): Chỉ định thời gian chờ để xử lý `pasrse`. Điều này chủ yếu là để phát hiện mẫu regrexp sai.
  + Mặc định: `nil`

### The detail of types parameter
Danh sách các loại được hỗ trợ được hiển thị dưới đây:
+ string: Chuyển đổi trường thành string type. Sử dụng method `to_s` để chuyển đổi.
+ `bool`: Chuyển đổi chuỗi 'true', 'yes' hoặc '1' thành `true`. Ngược lại, `false`.
+ `integer`: dùng method `to_i`
+ `fload`: dùng method to_f
+ `time`: Chuyển đổi trường thành loại `Fluent:: EventTime`. Điều này sử dụng trình phân tích time của Fluentd để chuyển đổi. Đối với loại thời gian, trường thứ ba chỉ định định dạng thời gian bạn sẽ có trong `time_format`.
```
date:time:%d/%b/%Y:%H:%M:%S %z # for string with time format
date:time:unixtime             # for integer time
date:time:float                # for float time
```
+ `array`: Chuyển đổi trường chuỗi thành kiểu mảng. Đối với loại "array", trường thứ ba chỉ định dấu phân cách (mặc định là ','). Ví dụ: nếu một trường có tên 'item_ids' chứa giá trị "3,4,5", hãy dùng `types item_ids:array` cú pháp đó được chuyển thành ["3", "4", "5"]. Ngoài ra, nếu giá trị là `"Adam|Alice|Bob", hãy nhập `types item_ids:array:| Phân tích nó thành ["Adam", "Alice", "Bob"].

### Time parameters
+ time_type(enum)(optional): parse/format giá trị theo loại này
  + Mặc định: string
  + Giá trị khả dụng: float, unixtime, string
+ **time_format**(string)(optional): Giá trị quá trình sử dụng định dạng được chỉ định.Điều này chỉ khả dụng khi `time_type` là string.
  + Mặc định: `nil`
+ **localtime**(bool)(optional): Nếu `true`, sử dụng giờ địa phương. Nếu `false`, UTC được sử dụng.
  + Mặc định: true
+ **utc**(bool)(optional): Nếu `true`, sử dụng UTC. Nếu `false`, giờ địa phương được sử dụng.(mặc định true)
+ **timezone**(string)(optional): Sử dụng múi giờ quy định. Người ta có thể phân tích/định dạng giá trị thời gian trong múi giờ được chỉ định.
