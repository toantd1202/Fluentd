# Copy
Các `copy` plugin đầu ra sao chép các sự kiện để taọ nhiều đầu ra.

## Example Configuration
`out_copy` có trong lõi của Fluentd. Không có quá trình cài đặt bổ sung được yêu cầu.
```
<match pattern>
  @type copy
  <store>
    @type file
    path /var/log/fluent/myapp1
    ...
  </store>
  <store>
    ...
  </store>
  <store>
    ...
  </store>
</match>
```

Dưới đây là một ví dụ được thiết lập để gửi các sự kiện tới cả tệp trong local `/var/log/fluent/myapp` và bộ sưu tập `fluentd.test` trong một ví dụ Elaticsearch.

## Parameters
**@type**
Giá trị phải là `copy`

**copy_mode**
+ type: enum
+ default: no_copy
  + `no_copy`: Chia sẻ sự kiện giữa các `<store>` plugin. Đây là chế độ mặc định.
  + `shallow`: Bỏ qua các sự kiện sao chép mức thấp cho mỗi `store` plugin. Chế độ này sử dụng `dup` method của ruby. Chế độ này hữu ích khi bạn không sửa đổi các trường lồng nhau sau `out_copy`, ví dụ: Loại bỏ các trường mức cao nhất.
  + `deep`: Truyền các sự kiện sao chép sâu cho từng `store` plugin. Chế độ này sử dụng `msgpack-ruby` trong nội miền. Chế độ này hữu ích khi bạn sửa đổi trường lồng nhau sau out_copy, ví dụ: Các lĩnh vực liên quan đến kubernetes.

**deep_copy**
+ type: bool
+ default:false 
Out_copy chia sẻ một bản ghi giữa các plugin `store` theo mặc định.
Khi `deep_copy` là true, `out_copy` chuyển bản ghi được sao chép vào từng plugin của `<store>`. Đây là hành vi tương tự với `copy_mode shallow.

**<store> section**
Chỉ định các điểm lưu trữ. Các định dạng giống như chỉ thị <match>.

**ignore_error argument**
Nếu một `store` phát sinh lỗi, nó sẽ ảnh hưởng đến `store` khác. Ví dụ,
```
<match app.**>
  @type copy
  <store>
    @type plugin1
  </store>
  <store>
    @type plugin2
  </store>
</match>
```

Nếu emit/format của plugin1 phát sinh lỗi, plugin2 sẽ không được thực thi. Nếu bạn muốn bỏ qua một lỗi từ 1 <store> ít quan trọng hơn, bạn có thể chỉ định ignor_error trong `<store>`.
