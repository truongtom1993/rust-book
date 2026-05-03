<!-- Các tiêu đề cũ. Không được xóa nếu không liên kết có thể bị hỏng. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## Chuyển hướng lỗi sang luồng lỗi chuẩn (Standard Error)

Hiện tại, chúng ta đang ghi toàn bộ đầu ra ra terminal bằng macro
`println!`. Trong hầu hết các terminal, có hai loại đầu ra: _đầu ra chuẩn_
(`stdout`) cho thông tin chung và _lỗi chuẩn_ (`stderr`) cho các thông báo lỗi.
Sự phân tách này cho phép người dùng lựa chọn chuyển hướng phần đầu ra thành
công của chương trình vào một tệp nhưng vẫn in thông báo lỗi ra màn hình.

Macro `println!` chỉ có khả năng in ra đầu ra chuẩn, do đó chúng ta phải dùng
một cơ chế khác để in ra lỗi chuẩn.

### Kiểm tra nơi lỗi được ghi ra

Đầu tiên, hãy quan sát cách nội dung được in bởi `minigrep` hiện đang được
ghi ra đầu ra chuẩn, bao gồm cả những thông báo lỗi mà chúng ta muốn ghi ra
lỗi chuẩn. Chúng ta sẽ làm điều đó bằng cách chuyển hướng luồng đầu ra chuẩn
sang một tệp trong khi cố tình gây ra lỗi. Chúng ta sẽ không chuyển hướng
luồng lỗi chuẩn, do đó mọi nội dung gửi đến lỗi chuẩn sẽ vẫn được hiển thị
trên màn hình.

Các chương trình dòng lệnh được kỳ vọng sẽ gửi thông báo lỗi đến luồng lỗi
chuẩn để chúng ta vẫn có thể thấy thông báo lỗi trên màn hình ngay cả khi
chuyển hướng luồng đầu ra chuẩn vào một tệp. Chương trình của chúng ta hiện
chưa hoạt động đúng: chúng ta sắp thấy rằng nó ghi thông báo lỗi vào tệp thay
vì hiển thị trên màn hình!

Để minh họa hành vi này, chúng ta sẽ chạy chương trình với ký hiệu `>` và
đường dẫn tệp _output.txt_, là tệp mà chúng ta muốn chuyển hướng luồng đầu ra
chuẩn tới. Chúng ta sẽ không truyền bất kỳ tham số nào, điều này sẽ gây ra
lỗi:

```console
$ cargo run > output.txt
```

Cú pháp `>` báo cho shell ghi nội dung của đầu ra chuẩn vào _output.txt_
thay vì lên màn hình. Chúng ta đã không thấy thông báo lỗi mong đợi được in
ra màn hình, vì vậy điều đó có nghĩa là nó đã được ghi vào tệp. Đây là nội
dung của _output.txt_:

```text
Problem parsing arguments: not enough arguments
```

Đúng vậy, thông báo lỗi của chúng ta đang được in ra đầu ra chuẩn. Sẽ hữu ích
hơn nhiều nếu những thông báo lỗi như thế này được in ra lỗi chuẩn để chỉ dữ
liệu từ một lần chạy thành công mới được ghi vào tệp. Chúng ta sẽ thay đổi
điều đó.

### In lỗi ra luồng lỗi chuẩn

Chúng ta sẽ sử dụng đoạn mã trong Liệt kê 12-24 để thay đổi cách in thông báo
lỗi. Do việc tái cấu trúc (refactoring) mà chúng ta đã thực hiện trước đó
trong chương này, toàn bộ mã in thông báo lỗi đều nằm trong một hàm, `main`.
Thư viện chuẩn cung cấp macro `eprintln!` để in ra luồng lỗi chuẩn, vì vậy hãy
thay đổi hai chỗ mà chúng ta đang gọi `println!` để in lỗi sang dùng
`eprintln!`.

<Listing number="12-24" file-name="src/main.rs" caption="Ghi thông báo lỗi ra luồng lỗi chuẩn thay vì đầu ra chuẩn bằng `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Bây giờ hãy chạy lại chương trình theo cách cũ, không có tham số và chuyển
hướng đầu ra chuẩn bằng `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Giờ đây chúng ta thấy thông báo lỗi trên màn hình và _output.txt_ không chứa
bất kỳ nội dung nào, đây là hành vi mong đợi của các chương trình dòng lệnh.

Hãy chạy lại chương trình với các tham số không gây lỗi nhưng vẫn chuyển
hướng đầu ra chuẩn vào tệp, như sau:

```console
$ cargo run -- to poem.txt > output.txt
```

Chúng ta sẽ không thấy bất kỳ đầu ra nào trên terminal, và _output.txt_ sẽ
chứa kết quả:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Điều này minh họa rằng chúng ta hiện đang sử dụng đầu ra chuẩn cho kết quả
thành công và lỗi chuẩn cho đầu ra lỗi một cách phù hợp.

## Tổng kết

Chương này đã hệ thống lại một số khái niệm chính mà bạn đã học cho đến nay
và trình bày cách thực hiện các thao tác I/O thường gặp trong Rust. Thông qua
việc sử dụng tham số dòng lệnh, tệp, biến môi trường, và macro `eprintln!` để
in lỗi, giờ đây bạn đã sẵn sàng viết các ứng dụng dòng lệnh. Kết hợp với các
khái niệm trong những chương trước, mã của bạn sẽ được tổ chức tốt, lưu trữ
dữ liệu hiệu quả trong những cấu trúc dữ liệu thích hợp, xử lý lỗi một cách
hợp lý và được kiểm thử đầy đủ.

Tiếp theo, chúng ta sẽ khám phá một số tính năng của Rust chịu ảnh hưởng từ
các ngôn ngữ hàm: closures và iterators.
