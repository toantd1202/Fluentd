# format section
## overview
Phần định dạng có thể nằm trong phần <match> hoặc <filter>.
```
<match tag.*>
  @type file
  # parameters for output plugin
  <format>
    # format section parameters
  </format>
</match>
```
## formatter plugin type
<format> yêu cầu tham số `@type` để chỉ định loại định dạng plugin . Fluentd chứa một số plugin định dạng hữu ích.
```
<format>
  @type json
</format>
```
Đây là danh sách các plugin định dạng tích hợp: out_file, json, ltsv, csv, msgpack, hash, single_value
## parameters
### Parameters
**@type**
@type là chỉ định loại định dạng plugin.
```
<format>
  @type csv
  # ...
</format>
```
## Time parameters


+ Time_type(enum)(tùy chọn): giá trị parse/format theo loại này.

Mặc định: float

+ Các giá trị khả dụng: float, unixtime, str
  + Float: giây từ Epoch + nano giây (ví dụ: 1510544836.154709804)
  + Unixtime: giây từ Epoch (ví dụ: 1510544815)
  + string: sử dụng định dạng được chỉ định bởi `time_format`, giờ địa phương hoặc múi giờ

+ **Time_format**(string)(optional): giá trị quá trình sử dụng định dạng được chỉ định. Điều này chỉ khả dụng khi `time_type` là string. Mặc định: `nil`

+ **timezone**(string)(optional): sử dụng múi giờ được chỉ định. Người ta có thể parse/format giá trị thời gian trong múi giờ được chỉ định.
  + Mặc định: `nil`
  + Định dạng múi giờ có sẵn:
[+ -]HH:MM (ví dụ: '+09: 00')
[+ -]HHMM (ví dụ: '+0900')
[+ -]HH (ví dụ: '+09')
Region/Zone (ví dụ: 'Asia/Tokyo')
Region/Zone/Zone (ví dụ: 'America/Argentina/Buenos_Aires')
