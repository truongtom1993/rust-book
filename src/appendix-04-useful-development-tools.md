## Phụ Lục D: Các Công Cụ Phát Triển Hữu Ích

Trong phụ lục này, chúng ta nói về một số công cụ phát triển hữu ích mà dự án Rust cung cấp. Chúng ta sẽ xem xét định dạng tự động, cách nhanh để áp dụng các sửa lỗi cảnh báo, một linter, và tích hợp với các IDE.

### Định Dạng Tự Động với `rustfmt`

Công cụ `rustfmt` định dạng lại code của bạn theo phong cách code của cộng đồng. Nhiều dự án cộng tác sử dụng `rustfmt` để tránh tranh luận về style nào cần sử dụng khi viết Rust: Mọi người đều định dạng code của họ bằng công cụ.

Các bản cài đặt Rust bao gồm `rustfmt` theo mặc định, vì vậy bạn đã có các chương trình `rustfmt` và `cargo-fmt` trên hệ thống của mình. Hai lệnh này tương tự như `rustc` và `cargo` theo nghĩa `rustfmt` cho phép kiểm soát chi tiết hơn và `cargo-fmt` hiểu các quy ước của một dự án sử dụng Cargo. Để định dạng bất kỳ dự án Cargo nào, nhập lệnh sau:

```console
$ cargo fmt
```

Chạy lệnh này định dạng lại tất cả code Rust trong crate hiện tại. Điều này chỉ nên thay đổi style code, không phải ngữ nghĩa code. Để biết thêm thông tin về `rustfmt`, xem [tài liệu của nó][rustfmt].

### Sửa Code của Bạn với `rustfix`

Công cụ `rustfix` được bao gồm trong các bản cài đặt Rust và có thể tự động sửa các cảnh báo của trình biên dịch có cách rõ ràng để sửa vấn đề mà có thể là điều bạn muốn. Bạn có thể đã thấy các cảnh báo của trình biên dịch trước đây. Ví dụ, hãy xem xét code này:

<span class="filename">Tên file: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

Ở đây, chúng ta đang định nghĩa biến `x` là mutable, nhưng chúng ta thực sự không bao giờ thay đổi nó. Rust cảnh báo chúng ta về điều đó:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

Cảnh báo gợi ý rằng chúng ta nên xóa từ khóa `mut`. Chúng ta có thể tự động áp dụng gợi ý đó bằng công cụ `rustfix` bằng cách chạy lệnh `cargo fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Khi chúng ta nhìn vào _src/main.rs_ một lần nữa, chúng ta sẽ thấy rằng `cargo fix` đã thay đổi code:

<span class="filename">Tên file: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

Biến `x` bây giờ là immutable, và cảnh báo không còn xuất hiện nữa.

Bạn cũng có thể sử dụng lệnh `cargo fix` để chuyển đổi code của bạn giữa các Rust edition khác nhau. Các edition được đề cập trong [Phụ Lục E][editions]<!-- ignore -->.

### Thêm Lint với Clippy

Công cụ Clippy là một bộ sưu tập các lint để phân tích code của bạn để bạn có thể phát hiện các lỗi thường gặp và cải thiện code Rust của mình. Clippy được bao gồm trong các bản cài đặt Rust tiêu chuẩn.

Để chạy các lint của Clippy trên bất kỳ dự án Cargo nào, nhập lệnh sau:

```console
$ cargo clippy
```

Ví dụ, giả sử bạn viết một chương trình sử dụng xấp xỉ của một hằng số toán học, chẳng hạn như pi, như chương trình này:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Chạy `cargo clippy` trên dự án này dẫn đến lỗi sau:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Lỗi này cho bạn biết rằng Rust đã có hằng số `PI` chính xác hơn được định nghĩa, và chương trình của bạn sẽ chính xác hơn nếu bạn sử dụng hằng số thay thế. Sau đó bạn sẽ thay đổi code để sử dụng hằng số `PI`.

Code sau đây không dẫn đến bất kỳ lỗi hoặc cảnh báo nào từ Clippy:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Để biết thêm thông tin về Clippy, xem [tài liệu của nó][clippy].

### Tích Hợp IDE Sử Dụng `rust-analyzer`

Để giúp tích hợp IDE, cộng đồng Rust khuyến nghị sử dụng [`rust-analyzer`][rust-analyzer]<!-- ignore -->. Công cụ này là một bộ các tiện ích tập trung vào trình biên dịch hỗ trợ [Language Server Protocol][lsp]<!-- ignore -->, đây là đặc tả để các IDE và ngôn ngữ lập trình giao tiếp với nhau. Các client khác nhau có thể sử dụng `rust-analyzer`, chẳng hạn như [plugin Rust analyzer cho Visual Studio Code][vscode].

Truy cập [trang chủ][rust-analyzer]<!-- ignore --> của dự án `rust-analyzer` để biết hướng dẫn cài đặt, sau đó cài đặt hỗ trợ language server trong IDE cụ thể của bạn. IDE của bạn sẽ có được các khả năng như tự động hoàn thành, nhảy đến định nghĩa, và hiển thị lỗi trực tiếp.

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
