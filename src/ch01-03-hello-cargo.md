## Hello, Cargo!

Cargo là build system và package manager của Rust. Hầu hết các Rustaceans dùng công cụ này để quản lý dự án Rust vì Cargo xử lý nhiều tác vụ cho bạn, chẳng hạn như build code, tải xuống các thư viện mà code của bạn phụ thuộc vào, và build những thư viện đó. (Chúng ta gọi các thư viện mà code của bạn cần là _dependencies_.)

Các chương trình Rust đơn giản nhất, như chương trình chúng ta vừa viết, không có bất kỳ dependency nào. Nếu chúng ta đã build dự án "Hello, world!" với Cargo, nó chỉ dùng phần Cargo xử lý việc build code. Khi bạn viết các chương trình Rust phức tạp hơn, bạn sẽ thêm dependency, và nếu bạn bắt đầu dự án bằng Cargo, việc thêm dependency sẽ dễ dàng hơn nhiều.

Vì phần lớn các dự án Rust đều dùng Cargo, phần còn lại của cuốn sách này giả định rằng bạn cũng đang dùng Cargo. Cargo được cài đặt kèm với Rust nếu bạn dùng các trình cài đặt chính thức đã thảo luận trong phần ["Cài Đặt"][installation]<!-- ignore -->. Nếu bạn cài Rust thông qua một phương thức khác, hãy kiểm tra xem Cargo có được cài đặt không bằng cách nhập lệnh sau vào terminal:

```console
$ cargo --version
```

Nếu bạn thấy số phiên bản, bạn đã có nó! Nếu bạn thấy lỗi như `command not found`, hãy xem tài liệu cho phương thức cài đặt của bạn để biết cách cài Cargo riêng.

### Tạo Dự Án Với Cargo

Hãy tạo một dự án mới bằng Cargo và xem nó khác với dự án "Hello, world!" ban đầu của chúng ta như thế nào. Quay lại thư mục _projects_ (hoặc bất cứ đâu bạn đã quyết định lưu code). Sau đó, trên bất kỳ hệ điều hành nào, chạy lệnh sau:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

Lệnh đầu tiên tạo một thư mục và dự án mới có tên _hello_cargo_. Chúng ta đặt tên dự án là _hello_cargo_, và Cargo tạo các file của nó trong một thư mục cùng tên.

Vào thư mục _hello_cargo_ và liệt kê các file. Bạn sẽ thấy Cargo đã tạo hai file và một thư mục cho chúng ta: một file _Cargo.toml_ và một thư mục _src_ chứa file _main.rs_ bên trong.

Nó cũng đã khởi tạo một Git repository mới cùng với file _.gitignore_. Các file Git sẽ không được tạo ra nếu bạn chạy `cargo new` trong một Git repository đã tồn tại; bạn có thể ghi đè hành vi này bằng cách dùng `cargo new --vcs=git`.

> Lưu ý: Git là một version control system phổ biến. Bạn có thể thay đổi `cargo new` để dùng version control system khác hoặc không dùng version control system nào bằng flag `--vcs`. Chạy `cargo new --help` để xem các tùy chọn có sẵn.

Mở _Cargo.toml_ trong text editor của bạn. Nó sẽ trông tương tự như code trong Listing 1-2.

<Listing number="1-2" file-name="Cargo.toml" caption="Nội dung của *Cargo.toml* được tạo bởi `cargo new`">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

File này ở định dạng [_TOML_][toml]<!-- ignore --> (_Tom's Obvious, Minimal Language_), là định dạng cấu hình của Cargo.

Dòng đầu tiên, `[package]`, là một tiêu đề section cho biết các câu lệnh tiếp theo đang cấu hình một package. Khi chúng ta thêm thông tin vào file này, chúng ta sẽ thêm các section khác.

Ba dòng tiếp theo thiết lập thông tin cấu hình mà Cargo cần để compile chương trình của bạn: tên, phiên bản và edition của Rust sẽ dùng. Chúng ta sẽ nói về key `edition` trong [Phụ lục E][appendix-e]<!-- ignore -->.

Dòng cuối cùng, `[dependencies]`, là phần đầu của section nơi bạn liệt kê các dependency của dự án. Trong Rust, các package code được gọi là _crates_. Chúng ta sẽ không cần thêm crate nào cho dự án này, nhưng chúng ta sẽ cần trong dự án đầu tiên ở Chương 2, vì vậy chúng ta sẽ dùng section dependencies này sau.

Bây giờ mở _src/main.rs_ và xem:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo đã tạo sẵn chương trình "Hello, world!" cho bạn, giống như chương trình chúng ta đã viết trong Listing 1-1! Cho đến nay, sự khác biệt giữa dự án của chúng ta và dự án Cargo tạo ra là Cargo đặt code trong thư mục _src_, và chúng ta có file cấu hình _Cargo.toml_ ở thư mục gốc.

Cargo kỳ vọng các file source của bạn nằm trong thư mục _src_. Thư mục gốc của dự án chỉ dành cho các file README, thông tin giấy phép, file cấu hình, và bất cứ thứ gì không liên quan đến code. Dùng Cargo giúp bạn tổ chức dự án gọn gàng hơn. Mọi thứ đều có chỗ của nó, và mọi thứ đều đúng chỗ.

Nếu bạn bắt đầu dự án không dùng Cargo, như chúng ta đã làm với dự án "Hello, world!", bạn có thể chuyển đổi nó thành dự án dùng Cargo. Di chuyển code dự án vào thư mục _src_ và tạo file _Cargo.toml_ phù hợp. Một cách dễ dàng để tạo file _Cargo.toml_ đó là chạy `cargo init`, nó sẽ tự động tạo cho bạn.

### Build và Chạy Dự Án Cargo

Bây giờ hãy xem điều gì khác biệt khi chúng ta build và chạy chương trình "Hello, world!" với Cargo! Từ thư mục _hello_cargo_ của bạn, build dự án bằng lệnh sau:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Lệnh này tạo một file thực thi trong _target/debug/hello_cargo_ (hoặc _target\debug\hello_cargo.exe_ trên Windows) thay vì trong thư mục hiện tại. Vì build mặc định là debug build, Cargo đặt binary trong một thư mục tên là _debug_. Bạn có thể chạy file thực thi bằng lệnh sau:

```console
$ ./target/debug/hello_cargo # hoặc .\target\debug\hello_cargo.exe trên Windows
Hello, world!
```

Nếu mọi thứ ổn, `Hello, world!` sẽ được in ra terminal. Chạy `cargo build` lần đầu tiên cũng khiến Cargo tạo một file mới ở cấp độ gốc: _Cargo.lock_. File này theo dõi các phiên bản chính xác của các dependency trong dự án. Dự án này không có dependency, nên file khá thưa thớt. Bạn sẽ không bao giờ cần thay đổi file này thủ công; Cargo quản lý nội dung của nó cho bạn.

Chúng ta vừa build dự án với `cargo build` và chạy nó với `./target/debug/hello_cargo`, nhưng chúng ta cũng có thể dùng `cargo run` để compile code và sau đó chạy file thực thi kết quả trong một lệnh duy nhất:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Dùng `cargo run` thuận tiện hơn là phải nhớ chạy `cargo build` và sau đó dùng toàn bộ đường dẫn đến binary, vì vậy hầu hết các developer dùng `cargo run`.

Lưu ý rằng lần này chúng ta không thấy output cho biết Cargo đang compile `hello_cargo`. Cargo nhận ra rằng các file không thay đổi nên không build lại mà chỉ chạy binary. Nếu bạn đã chỉnh sửa source code, Cargo sẽ build lại dự án trước khi chạy và bạn sẽ thấy output này:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo cũng cung cấp một lệnh gọi là `cargo check`. Lệnh này kiểm tra nhanh code của bạn để đảm bảo nó compile được nhưng không tạo ra file thực thi:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Tại sao bạn không muốn tạo file thực thi? Thường thì `cargo check` nhanh hơn nhiều so với `cargo build` vì nó bỏ qua bước tạo file thực thi. Nếu bạn liên tục kiểm tra công việc trong khi viết code, dùng `cargo check` sẽ tăng tốc quá trình thông báo cho bạn biết dự án có còn compile được không! Vì vậy, nhiều Rustaceans chạy `cargo check` định kỳ khi họ viết chương trình để đảm bảo nó compile. Sau đó, họ chạy `cargo build` khi đã sẵn sàng dùng file thực thi.

Hãy tóm tắt những gì chúng ta đã học về Cargo cho đến nay:

- Chúng ta có thể tạo dự án bằng `cargo new`.
- Chúng ta có thể build dự án bằng `cargo build`.
- Chúng ta có thể build và chạy dự án trong một bước bằng `cargo run`.
- Chúng ta có thể build dự án mà không tạo binary để kiểm tra lỗi bằng `cargo check`.
- Thay vì lưu kết quả build trong cùng thư mục với code, Cargo lưu nó trong thư mục _target/debug_.

Một ưu điểm bổ sung khi dùng Cargo là các lệnh đều giống nhau dù bạn đang làm việc trên hệ điều hành nào. Vì vậy, từ thời điểm này, chúng ta sẽ không còn cung cấp hướng dẫn riêng cho Linux và macOS so với Windows.

### Build cho Release

Khi dự án của bạn đã sẵn sàng để release, bạn có thể dùng `cargo build --release` để compile nó với tối ưu hóa. Lệnh này sẽ tạo file thực thi trong _target/release_ thay vì _target/debug_. Các tối ưu hóa làm cho Rust code chạy nhanh hơn, nhưng bật chúng sẽ kéo dài thời gian compile chương trình. Đây là lý do tại sao có hai profile khác nhau: một cho development, khi bạn muốn build lại nhanh và thường xuyên, và một cho việc build chương trình cuối cùng bạn sẽ đưa cho người dùng, không được build lại nhiều lần và sẽ chạy nhanh nhất có thể. Nếu bạn đang đo hiệu năng runtime của code, hãy chắc chắn chạy `cargo build --release` và benchmark với file thực thi trong _target/release_.

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### Tận Dụng Các Quy Ước Của Cargo

Với các dự án đơn giản, Cargo không cung cấp nhiều giá trị hơn so với chỉ dùng `rustc`, nhưng nó sẽ chứng tỏ giá trị khi chương trình của bạn trở nên phức tạp hơn. Khi chương trình phát triển thành nhiều file hoặc cần một dependency, việc để Cargo điều phối build sẽ dễ dàng hơn nhiều.

Mặc dù dự án `hello_cargo` đơn giản, nhưng bây giờ nó đã dùng nhiều công cụ thực tế mà bạn sẽ dùng trong suốt sự nghiệp Rust của mình. Thực tế, để làm việc với bất kỳ dự án hiện có nào, bạn có thể dùng các lệnh sau để checkout code bằng Git, chuyển đến thư mục của dự án đó, và build:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Để biết thêm thông tin về Cargo, hãy xem [tài liệu của nó][cargo].

## Tóm Tắt

Bạn đã có một khởi đầu tuyệt vời trong hành trình Rust của mình! Trong chương này, bạn đã học cách:

- Cài đặt phiên bản stable mới nhất của Rust bằng `rustup`.
- Cập nhật lên phiên bản Rust mới hơn.
- Mở tài liệu được cài đặt cục bộ.
- Viết và chạy chương trình "Hello, world!" bằng cách dùng `rustc` trực tiếp.
- Tạo và chạy dự án mới bằng các quy ước của Cargo.

Đây là thời điểm tuyệt vời để xây dựng một chương trình thực chất hơn để làm quen với việc đọc và viết Rust code. Vì vậy, trong Chương 2, chúng ta sẽ xây dựng một chương trình trò chơi đoán số. Nếu bạn muốn bắt đầu bằng cách học cách các khái niệm lập trình phổ biến hoạt động trong Rust, hãy xem Chương 3 trước rồi quay lại Chương 2.

[installation]: ch01-01-installation.html#cài-đặt
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
