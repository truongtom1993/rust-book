## Cài Đặt

Bước đầu tiên là cài đặt Rust. Chúng ta sẽ tải Rust thông qua `rustup`, một công cụ dòng lệnh để quản lý các phiên bản Rust và các công cụ liên quan. Bạn cần có kết nối internet để tải xuống.

> Lưu ý: Nếu bạn không muốn dùng `rustup` vì lý do nào đó, hãy xem [trang Other Rust Installation Methods][otherinstall] để biết thêm các lựa chọn khác.

Các bước dưới đây sẽ cài đặt phiên bản stable mới nhất của Rust compiler. Rust đảm bảo tính ổn định, nghĩa là tất cả các ví dụ trong sách có thể compile được sẽ tiếp tục compile được với các phiên bản Rust mới hơn. Output có thể khác nhau đôi chút giữa các phiên bản vì Rust thường xuyên cải thiện thông báo lỗi và cảnh báo. Nói cách khác, bất kỳ phiên bản stable mới hơn nào bạn cài đặt theo các bước này đều sẽ hoạt động đúng với nội dung của cuốn sách này.

> ### Quy Ước Ký Hiệu Dòng Lệnh
>
> Trong chương này và xuyên suốt cuốn sách, chúng ta sẽ hiển thị một số lệnh được dùng trong terminal. Các dòng bạn cần nhập vào terminal đều bắt đầu bằng `$`. Bạn không cần gõ ký tự `$`; đó là dấu nhắc lệnh (command prompt) được hiển thị để đánh dấu đầu mỗi lệnh. Các dòng không bắt đầu bằng `$` thường là output của lệnh trước đó. Ngoài ra, các ví dụ dành riêng cho PowerShell sẽ dùng `>` thay vì `$`.

### Cài Đặt `rustup` trên Linux hoặc macOS

Nếu bạn đang dùng Linux hoặc macOS, mở terminal và nhập lệnh sau:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Lệnh này sẽ tải về một script và bắt đầu cài đặt công cụ `rustup`, từ đó cài đặt phiên bản stable mới nhất của Rust. Bạn có thể được yêu cầu nhập mật khẩu. Nếu cài đặt thành công, dòng sau sẽ xuất hiện:

```text
Rust is installed now. Great!
```

Bạn cũng sẽ cần một _linker_, là chương trình mà Rust dùng để ghép các file đã compile thành một file duy nhất. Có khả năng bạn đã có sẵn rồi. Nếu gặp lỗi linker, bạn nên cài đặt C compiler, thường đã bao gồm linker. C compiler cũng hữu ích vì một số Rust package phổ biến phụ thuộc vào C code và sẽ cần C compiler.

Trên macOS, bạn có thể cài C compiler bằng lệnh:

```console
$ xcode-select --install
```

Người dùng Linux nên cài GCC hoặc Clang theo hướng dẫn của bản phân phối mình đang dùng. Ví dụ, nếu dùng Ubuntu, bạn có thể cài package `build-essential`.

### Cài Đặt `rustup` trên Windows

Trên Windows, truy cập [https://www.rust-lang.org/tools/install][install]<!-- ignore --> và làm theo hướng dẫn để cài đặt Rust. Tại một thời điểm trong quá trình cài đặt, bạn sẽ được nhắc cài Visual Studio. Đây cung cấp linker và các native library cần thiết để compile chương trình. Nếu bạn cần thêm trợ giúp ở bước này, hãy xem [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!-- ignore -->.

Phần còn lại của cuốn sách sử dụng các lệnh hoạt động trên cả _cmd.exe_ và PowerShell. Nếu có sự khác biệt, chúng ta sẽ giải thích nên dùng cái nào.

### Kiểm Tra Lỗi (Troubleshooting)

Để kiểm tra xem Rust đã được cài đặt đúng chưa, mở shell và nhập dòng lệnh này:

```console
$ rustc --version
```

Bạn sẽ thấy số phiên bản, commit hash và ngày commit của phiên bản stable mới nhất đã được release, theo định dạng sau:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Nếu bạn thấy thông tin này, Rust đã được cài đặt thành công! Nếu không thấy, hãy kiểm tra xem Rust có trong biến hệ thống `%PATH%` của bạn không theo các cách sau.

Trong Windows CMD, dùng:

```console
> echo %PATH%
```

Trong PowerShell, dùng:

```powershell
> echo $env:Path
```

Trên Linux và macOS, dùng:

```console
$ echo $PATH
```

Nếu tất cả đều đúng mà Rust vẫn không hoạt động, có một số nơi bạn có thể tìm kiếm sự trợ giúp. Tìm hiểu cách liên hệ với cộng đồng Rustaceans (biệt danh vui của chúng ta) trên [trang cộng đồng][community].

### Cập Nhật và Gỡ Cài Đặt

Sau khi đã cài Rust qua `rustup`, việc cập nhật lên phiên bản mới release rất đơn giản. Từ shell của bạn, chạy lệnh update sau:

```console
$ rustup update
```

Để gỡ cài đặt Rust và `rustup`, chạy lệnh uninstall sau từ shell:

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### Đọc Tài Liệu Offline

Bản cài đặt Rust cũng bao gồm một bản sao tài liệu cục bộ để bạn có thể đọc offline. Chạy `rustup doc` để mở tài liệu cục bộ trong trình duyệt.

Bất cứ khi nào một type hoặc function được cung cấp bởi standard library và bạn không chắc nó làm gì hoặc cách dùng như thế nào, hãy dùng tài liệu API (application programming interface) để tìm hiểu!

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### Sử Dụng Text Editor và IDE

Cuốn sách này không giả định bạn dùng công cụ nào để viết Rust code. Hầu hết bất kỳ text editor nào cũng có thể làm được việc! Tuy nhiên, nhiều text editor và IDE (integrated development environments) đã có hỗ trợ Rust tích hợp sẵn. Bạn luôn có thể tìm thấy danh sách khá cập nhật của nhiều editor và IDE trên [trang tools][tools] của website Rust.

### Làm Việc Offline Với Cuốn Sách Này

Trong một số ví dụ, chúng ta sẽ dùng các Rust package ngoài standard library. Để thực hành những ví dụ đó, bạn cần có kết nối internet hoặc đã tải trước các dependency đó. Để tải dependency trước, bạn có thể chạy các lệnh sau. (Chúng ta sẽ giải thích `cargo` là gì và từng lệnh làm gì ở phần sau.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Lệnh này sẽ cache các bản tải xuống của các package này để bạn không cần tải lại sau. Sau khi chạy lệnh này, bạn không cần giữ lại folder `get-dependencies`. Nếu đã chạy lệnh này, bạn có thể dùng flag `--offline` với tất cả các lệnh `cargo` trong phần còn lại của cuốn sách để dùng các phiên bản đã cache thay vì cố gắng kết nối mạng.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
