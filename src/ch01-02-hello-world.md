## Hello, World!

Bây giờ bạn đã cài đặt Rust, đến lúc viết chương trình Rust đầu tiên rồi. Theo truyền thống khi học một ngôn ngữ mới, người ta thường viết một chương trình nhỏ in ra dòng chữ `Hello, world!` lên màn hình, nên chúng ta cũng sẽ làm vậy!

> Lưu ý: Cuốn sách này giả định bạn đã quen cơ bản với dòng lệnh (command line). Rust không có yêu cầu cụ thể về việc bạn dùng editor hay công cụ nào, hoặc code của bạn nằm ở đâu, vì vậy nếu bạn muốn dùng IDE thay vì dòng lệnh, hãy thoải mái dùng IDE yêu thích của bạn. Nhiều IDE hiện nay đã hỗ trợ Rust ở một mức độ nhất định; hãy xem tài liệu của IDE để biết chi tiết. Nhóm Rust đang tập trung vào việc hỗ trợ IDE tốt thông qua `rust-analyzer`. Xem [Phụ lục D][devtools]<!-- ignore --> để biết thêm chi tiết.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### Thiết Lập Thư Mục Dự Án

Bắt đầu bằng cách tạo một thư mục để lưu trữ Rust code. Đối với Rust, code của bạn nằm ở đâu không quan trọng, nhưng cho các bài tập và dự án trong cuốn sách này, chúng ta đề xuất tạo một thư mục _projects_ trong home directory và lưu tất cả các dự án ở đó.

Mở terminal và nhập các lệnh sau để tạo thư mục _projects_ và một thư mục cho dự án "Hello, world!" bên trong thư mục _projects_.

Với Linux, macOS và PowerShell trên Windows, nhập:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Với Windows CMD, nhập:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

### Cơ Bản Về Chương Trình Rust

Tiếp theo, tạo một file source mới và đặt tên là _main.rs_. File Rust luôn có phần mở rộng _.rs_. Nếu tên file có nhiều hơn một từ, quy ước là dùng dấu gạch dưới để ngăn cách chúng. Ví dụ, dùng _hello_world.rs_ thay vì _helloworld.rs_.

Bây giờ mở file _main.rs_ vừa tạo và nhập code trong Listing 1-1.

<Listing number="1-1" file-name="main.rs" caption="Chương trình in ra `Hello, world!`">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

Lưu file và quay lại cửa sổ terminal trong thư mục _~/projects/hello_world_. Trên Linux hoặc macOS, nhập các lệnh sau để compile và chạy file:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

Trên Windows, nhập lệnh `.\main` thay vì `./main`:

```powershell
> rustc main.rs
> .\main
Hello, world!
```

Dù bạn dùng hệ điều hành nào, chuỗi `Hello, world!` đều sẽ được in ra terminal. Nếu bạn không thấy output này, hãy quay lại phần ["Troubleshooting"][troubleshooting]<!-- ignore --> trong phần Cài Đặt để tìm cách khắc phục.

Nếu `Hello, world!` đã được in ra, xin chúc mừng! Bạn đã chính thức viết được một chương trình Rust. Bạn là một Rust programmer rồi — chào mừng!

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### Cấu Trúc Của Một Chương Trình Rust

Hãy xem xét chi tiết chương trình "Hello, world!" này. Đây là mảnh ghép đầu tiên:

```rust
fn main() {

}
```

Những dòng này định nghĩa một function có tên `main`. Function `main` rất đặc biệt: nó luôn là đoạn code đầu tiên chạy trong mọi chương trình Rust có thể thực thi. Ở đây, dòng đầu tiên khai báo một function tên `main`, không có parameter và không trả về gì. Nếu có parameter, chúng sẽ nằm bên trong dấu ngoặc đơn (`()`).

Phần thân của function được bao bọc trong `{}`. Rust yêu cầu dấu ngoặc nhọn xung quanh tất cả các phần thân function. Phong cách tốt là đặt dấu ngoặc nhọn mở trên cùng dòng với khai báo function, thêm một khoảng trắng ở giữa.

> Lưu ý: Nếu bạn muốn tuân theo phong cách chuẩn trong các dự án Rust, bạn có thể dùng công cụ tự động định dạng tên `rustfmt` để định dạng code theo một phong cách nhất định (thêm về `rustfmt` trong [Phụ lục D][devtools]<!-- ignore -->). Nhóm Rust đã bao gồm công cụ này trong bản phân phối Rust tiêu chuẩn, cùng với `rustc`, nên nó đã được cài đặt trên máy tính của bạn rồi!

Phần thân của function `main` chứa đoạn code sau:

```rust
println!("Hello, world!");
```

Dòng này làm tất cả mọi việc trong chương trình nhỏ này: nó in văn bản ra màn hình. Có ba chi tiết quan trọng cần lưu ý ở đây.

Thứ nhất, `println!` gọi một Rust macro. Nếu nó gọi một function thay vào đó, sẽ được viết là `println` (không có `!`). Rust macro là cách để viết code sinh ra code nhằm mở rộng cú pháp Rust, và chúng ta sẽ thảo luận chi tiết hơn trong [Chương 20][ch20-macros]<!-- ignore -->. Hiện tại, bạn chỉ cần biết rằng dùng `!` nghĩa là bạn đang gọi một macro thay vì một function thông thường, và macro không phải lúc nào cũng tuân theo các quy tắc giống function.

Thứ hai, bạn thấy chuỗi `"Hello, world!"`. Chúng ta truyền chuỗi này như một argument vào `println!`, và chuỗi được in ra màn hình.

Thứ ba, chúng ta kết thúc dòng bằng dấu chấm phẩy (`;`), biểu thị rằng expression này đã kết thúc và cái tiếp theo đã sẵn sàng bắt đầu. Hầu hết các dòng Rust code đều kết thúc bằng dấu chấm phẩy.

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### Biên Dịch và Thực Thi

Bạn vừa chạy một chương trình mới tạo, vậy hãy xem xét từng bước trong quá trình đó.

Trước khi chạy một chương trình Rust, bạn phải compile nó bằng Rust compiler bằng cách nhập lệnh `rustc` và truyền cho nó tên file source của bạn, như sau:

```console
$ rustc main.rs
```

Nếu bạn có nền tảng về C hoặc C++, bạn sẽ nhận ra điều này tương tự với `gcc` hay `clang`. Sau khi compile thành công, Rust xuất ra một file thực thi nhị phân (binary executable).

Trên Linux, macOS và PowerShell trên Windows, bạn có thể thấy file thực thi bằng cách nhập lệnh `ls` trong shell:

```console
$ ls
main  main.rs
```

Trên Linux và macOS, bạn sẽ thấy hai file. Với PowerShell trên Windows, bạn sẽ thấy ba file giống như khi dùng CMD. Với CMD trên Windows, bạn sẽ nhập:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

Đây hiển thị file source code với phần mở rộng _.rs_, file thực thi (_main.exe_ trên Windows, nhưng là _main_ trên tất cả các nền tảng khác), và khi dùng Windows, một file chứa thông tin debug với phần mở rộng _.pdb_. Từ đây, bạn chạy file _main_ hoặc _main.exe_, như sau:

```console
$ ./main # hoặc .\main trên Windows
```

Nếu file _main.rs_ của bạn là chương trình "Hello, world!", dòng này sẽ in `Hello, world!` ra terminal.

Nếu bạn quen với ngôn ngữ động như Ruby, Python hoặc JavaScript, bạn có thể không quen với việc compile và chạy chương trình là hai bước riêng biệt. Rust là ngôn ngữ _biên dịch trước (ahead-of-time compiled)_, nghĩa là bạn có thể compile một chương trình và đưa file thực thi cho người khác, và họ có thể chạy nó ngay cả khi không có Rust được cài đặt. Nếu bạn đưa cho ai đó một file _.rb_, _.py_ hoặc _.js_, họ cần có triển khai Ruby, Python hoặc JavaScript được cài đặt (tương ứng). Nhưng trong những ngôn ngữ đó, bạn chỉ cần một lệnh để compile và chạy chương trình. Mọi thứ đều có sự đánh đổi trong thiết kế ngôn ngữ.

Chỉ compile bằng `rustc` là ổn với các chương trình đơn giản, nhưng khi dự án phát triển, bạn sẽ muốn quản lý tất cả các tùy chọn và làm cho việc chia sẻ code trở nên dễ dàng hơn. Tiếp theo, chúng ta sẽ giới thiệu bạn với công cụ Cargo, sẽ giúp bạn viết các chương trình Rust thực tế.

[troubleshooting]: ch01-01-installation.html#kiểm-tra-lỗi-troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
