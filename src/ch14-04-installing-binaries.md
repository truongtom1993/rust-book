<!-- Các tiêu đề cũ. Không được xóa nếu không các liên kết có thể bị hỏng. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Cài đặt các chương trình nhị phân với `cargo install`

Lệnh `cargo install` cho phép bạn cài đặt và sử dụng các crate nhị phân ở môi trường cục bộ. Lệnh này không nhằm thay thế các gói của hệ thống; mục đích của nó là cung cấp một cách thuận tiện để các lập trình viên Rust cài đặt các công cụ mà người khác đã chia sẻ trên [crates.io](https://crates.io/)<!-- ignore -->. Lưu ý rằng bạn chỉ có thể cài đặt các gói có **binary target** (mục tiêu nhị phân). **Binary target** là chương trình có thể thực thi được tạo ra nếu crate có tệp _src/main.rs_ hoặc một tệp khác được chỉ định là binary, trái ngược với **library target** (mục tiêu thư viện) vốn không thể tự chạy mà chỉ phù hợp để được nhúng vào các chương trình khác. Thông thường, các crate sẽ có thông tin trong tệp README về việc crate đó là thư viện, có binary target, hay có cả hai.

Tất cả các chương trình nhị phân được cài đặt bằng `cargo install` đều được lưu trong thư mục _bin_ của thư mục gốc cài đặt. Nếu bạn cài đặt Rust bằng _rustup.rs_ và không có cấu hình tùy chỉnh nào, thì thư mục này sẽ là *$HOME/.cargo/bin*. Hãy đảm bảo rằng thư mục này nằm trong biến môi trường `$PATH` để có thể chạy được các chương trình bạn đã cài đặt bằng `cargo install`.

Ví dụ, trong Chương 12 chúng ta đã đề cập rằng có một bản triển khai Rust của công cụ `grep` tên là `ripgrep` dùng để tìm kiếm trong tệp. Để cài đặt `ripgrep`, chúng ta có thể chạy lệnh sau:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

Dòng áp chót của phần xuất (output) hiển thị vị trí và tên của chương trình nhị phân đã được cài đặt, trong trường hợp của `ripgrep` là `rg`. Miễn là thư mục cài đặt nằm trong `$PATH` của bạn như đã đề cập ở trên, bạn có thể chạy `rg --help` và bắt đầu sử dụng một công cụ tìm kiếm tệp nhanh hơn, được viết bằng Rust!