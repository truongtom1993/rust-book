## Packages và Crates

Phần đầu tiên của module system mà chúng ta sẽ tìm hiểu là packages và crates.

**Crate** là lượng code nhỏ nhất mà Rust compiler xem xét tại một thời điểm. Ngay cả khi bạn chạy `rustc` thay vì `cargo` và truyền một single source code file (như chúng ta đã làm từ đầu trong phần ["Rust Program Basics"][basics]<!-- ignore --> ở Chapter 1), compiler vẫn coi file đó là một crate. Crates có thể chứa modules, và các modules này có thể được định nghĩa trong các file khác được compile cùng với crate, như chúng ta sẽ thấy ở các phần tiếp theo.

Một crate có thể có một trong hai dạng: binary crate hoặc library crate.

**Binary crates** là các chương trình mà bạn có thể compile thành một executable để chạy được, chẳng hạn như một command line program hay một server. Mỗi binary crate phải có một function gọi là `main` để định nghĩa những gì sẽ xảy ra khi executable chạy. Tất cả các crates mà chúng ta đã tạo cho đến nay đều là binary crates.

**Library crates** không có function `main`, và chúng không được compile thành executable. Thay vào đó, chúng định nghĩa functionality dùng để chia sẻ với nhiều projects. Chẳng hạn, `rand` crate mà chúng ta sử dụng trong [Chapter 2][rand]<!-- ignore --> cung cấp functionality để tạo random numbers. Hầu hết thời gian khi các Rustaceans nói "crate", họ muốn nói đến library crate, và họ sử dụng từ "crate" để chỉ khái niệm chung trong lập trình về một "library".

**Crate root** là một source file mà Rust compiler bắt đầu từ đó và tạo nên root module của crate của bạn (chúng ta sẽ giải thích modules chi tiết hơn trong ["Control Scope and Privacy with Modules"][modules]<!-- ignore -->).

**Package** là một bundle của một hoặc nhiều crates cung cấp một tập hợp functionality. Một package chứa một **Cargo.toml** file mô tả cách build các crates đó. Cargo thực chất là một package mà chứa binary crate cho command line tool mà bạn đã dùng để build code. Package Cargo cũng chứa một library crate mà binary crate phụ thuộc vào. Các projects khác có thể phụ thuộc vào Cargo library crate để sử dụng cùng logic mà Cargo command line tool sử dụng.

Một package có thể chứa bao nhiêu binary crates tùy ý, nhưng tối đa chỉ có một library crate. Một package phải chứa ít nhất một crate, dù đó là library hay binary crate.

Hãy cùng xem những gì sẽ xảy ra khi chúng ta tạo một package. Trước tiên, chúng ta nhập command `cargo new my-project`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Sau khi chúng ta chạy `cargo new my-project`, chúng ta sử dụng `ls` để xem Cargo đã tạo gì. Trong thư mục **my-project**, có một file **Cargo.toml**, cho chúng ta một package. Cũng có một thư mục **src** chứa **main.rs**. Mở **Cargo.toml** trong text editor của bạn và lưu ý rằng không có đề cập đến **src/main.rs**. Cargo tuân theo convention là **src/main.rs** là crate root của một binary crate có tên trùng với package. Tương tự, Cargo biết rằng nếu package directory chứa **src/lib.rs**, package chứa một library crate có tên trùng với package, và **src/lib.rs** là crate root của nó. Cargo truyền các crate root files cho `rustc` để build library hoặc binary.

Ở đây, chúng ta có một package chỉ chứa **src/main.rs**, nghĩa là nó chỉ chứa một binary crate có tên là `my-project`. Nếu một package chứa **src/main.rs** và **src/lib.rs**, nó sẽ có hai crates: một binary và một library, cả hai đều có tên trùng với package. Một package có thể có nhiều binary crates bằng cách đặt files trong thư mục **src/bin**: Mỗi file sẽ là một separate binary crate.

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number