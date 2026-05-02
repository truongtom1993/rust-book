# Lập trình trò chơi đoán số

Hãy cùng bắt đầu với Rust qua một dự án thực hành! Chương này giới thiệu một số khái niệm phổ biến trong Rust bằng cách cho bạn thấy cách sử dụng chúng trong một chương trình thực tế. Bạn sẽ học về `let`, `match`, methods, associated functions, external crates, và nhiều hơn nữa! Ở các chương tiếp theo, chúng ta sẽ tìm hiểu sâu hơn về những khái niệm này. Trong chương này, bạn chỉ cần luyện tập các kiến thức cơ bản.

Chúng ta sẽ xây dựng một bài toán lập trình cổ điển cho người mới bắt đầu: trò chơi đoán số. Đây là cách hoạt động: Chương trình sẽ tạo ra một số nguyên ngẫu nhiên từ 1 đến 100. Sau đó nó sẽ yêu cầu người chơi nhập dự đoán. Sau khi nhập dự đoán, chương trình sẽ cho biết dự đoán đó quá thấp hay quá cao. Nếu đoán đúng, trò chơi sẽ in thông báo chúc mừng và kết thúc.

## Khởi tạo dự án mới

Để khởi tạo dự án mới, hãy vào thư mục _projects_ mà bạn đã tạo ở Chương 1 và tạo dự án mới bằng Cargo như sau:

```console
$ cargo new guessing_game
$ cd guessing_game
```

Lệnh đầu tiên, `cargo new`, nhận tên dự án (`guessing_game`) làm đối số đầu tiên. Lệnh thứ hai chuyển vào thư mục dự án mới.

Xem file _Cargo.toml_ được tạo ra:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Như bạn đã thấy ở Chương 1, `cargo new` tạo ra một chương trình "Hello, world!" cho bạn. Xem file _src/main.rs_:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Bây giờ hãy biên dịch chương trình "Hello, world!" này và chạy nó trong cùng một bước bằng lệnh `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Lệnh `run` rất tiện khi bạn cần liên tục cải tiến dự án, như chúng ta sẽ làm trong trò chơi này — kiểm tra nhanh từng vòng lặp trước khi chuyển sang vòng tiếp theo.

Mở lại file _src/main.rs_. Bạn sẽ viết toàn bộ code trong file này.

## Xử lý dự đoán

Phần đầu tiên của chương trình trò chơi đoán số sẽ yêu cầu nhập dữ liệu từ người dùng, xử lý input đó, và kiểm tra xem input có đúng định dạng không. Để bắt đầu, chúng ta sẽ cho phép người chơi nhập dự đoán. Nhập code trong Listing 2-1 vào _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Code lấy dự đoán từ người dùng và in ra">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Code này chứa nhiều thông tin, hãy đọc từng dòng một. Để nhận input từ người dùng rồi in kết quả ra output, chúng ta cần đưa thư viện input/output `io` vào scope. Thư viện `io` đến từ standard library, được biết đến là `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Theo mặc định, Rust có một tập hợp các item được định nghĩa trong standard library và tự động đưa vào scope của mọi chương trình. Tập hợp này gọi là _prelude_, và bạn có thể xem toàn bộ nội dung của nó [trong tài liệu standard library][prelude].

Nếu một type bạn muốn dùng không có trong prelude, bạn phải đưa type đó vào scope một cách tường minh bằng câu lệnh `use`. Sử dụng thư viện `std::io` cung cấp cho bạn nhiều tính năng hữu ích, bao gồm khả năng nhận input từ người dùng.

Như bạn đã thấy ở Chương 1, function `main` là điểm bắt đầu của chương trình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

Cú pháp `fn` khai báo một function mới; dấu ngoặc đơn `()` cho biết không có parameters; và dấu ngoặc nhọn `{` bắt đầu phần thân của function.

Như bạn cũng đã học ở Chương 1, `println!` là một macro in chuỗi ra màn hình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Code này in ra một prompt cho biết trò chơi là gì và yêu cầu người dùng nhập input.

### Lưu trữ giá trị với variable

Tiếp theo, chúng ta sẽ tạo một _variable_ để lưu input của người dùng, như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Bây giờ chương trình đang trở nên thú vị! Có nhiều thứ diễn ra trong dòng nhỏ này. Chúng ta dùng câu lệnh `let` để tạo variable. Đây là một ví dụ khác:

```rust,ignore
let apples = 5;
```

Dòng này tạo một variable mới tên `apples` và bind nó với giá trị `5`. Trong Rust, các variable mặc định là immutable, nghĩa là một khi đã gán giá trị cho variable thì giá trị đó sẽ không thay đổi. Chúng ta sẽ thảo luận khái niệm này chi tiết hơn trong phần ["Variables and Mutability"][variables-and-mutability]<!-- ignore --> ở Chương 3. Để làm variable có thể thay đổi (mutable), chúng ta thêm `mut` trước tên variable:

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> Lưu ý: Cú pháp `//` bắt đầu một comment kéo dài đến hết dòng. Rust bỏ qua mọi thứ trong comment. Chúng ta sẽ thảo luận về comment chi tiết hơn ở [Chương 3][comments]<!-- ignore -->.

Quay lại chương trình trò chơi đoán số, bạn giờ đã biết `let mut guess` sẽ tạo ra một mutable variable tên `guess`. Dấu bằng (`=`) cho Rust biết chúng ta muốn bind một thứ gì đó vào variable ngay bây giờ. Ở phía bên phải dấu bằng là giá trị mà `guess` được bind vào, đó là kết quả của việc gọi `String::new`, một function trả về một instance mới của `String`. [`String`][string]<!-- ignore --> là một string type được cung cấp bởi standard library, là một đoạn văn bản có thể mở rộng được, mã hóa UTF-8.

Cú pháp `::` trong dòng `::new` cho biết `new` là một associated function của type `String`. Một _associated function_ là một function được implement trên một type, trong trường hợp này là `String`. Function `new` này tạo ra một string mới, rỗng. Bạn sẽ thấy function `new` trên nhiều type vì đây là tên phổ biến cho một function tạo ra giá trị mới của một loại nào đó.

Tóm lại, dòng `let mut guess = String::new();` đã tạo ra một mutable variable hiện tại đang được bind với một instance mới, rỗng của `String`. Phù!

### Nhận input từ người dùng

Nhớ lại rằng chúng ta đã đưa tính năng input/output từ standard library vào với `use std::io;` ở dòng đầu tiên của chương trình. Bây giờ chúng ta sẽ gọi function `stdin` từ module `io`, cho phép chúng ta xử lý input từ người dùng:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Nếu chúng ta không import module `io` với `use std::io;` ở đầu chương trình, chúng ta vẫn có thể dùng function này bằng cách viết `std::io::stdin`. Function `stdin` trả về một instance của [`std::io::Stdin`][iostdin]<!-- ignore -->, là một type đại diện cho handle đến standard input của terminal.

Tiếp theo, dòng `.read_line(&mut guess)` gọi method [`read_line`][read_line]<!-- ignore --> trên standard input handle để lấy input từ người dùng. Chúng ta cũng truyền `&mut guess` làm đối số cho `read_line` để nói cho nó biết string nào để lưu input của người dùng vào. Nhiệm vụ đầy đủ của `read_line` là lấy bất cứ thứ gì người dùng gõ vào standard input và append vào một string (không ghi đè nội dung của nó), nên chúng ta truyền string đó làm đối số. String argument cần phải là mutable để method có thể thay đổi nội dung của string.

Ký hiệu `&` cho biết đối số này là một _reference_, cho phép nhiều phần của code cùng truy cập một phần dữ liệu mà không cần sao chép dữ liệu đó vào bộ nhớ nhiều lần. References là một tính năng phức tạp, và một trong những lợi thế lớn của Rust là cách sử dụng references an toàn và dễ dàng. Bạn không cần biết nhiều chi tiết đó để hoàn thành chương trình này. Hiện tại, bạn chỉ cần biết rằng, giống như các variable, references mặc định là immutable. Do đó, bạn cần viết `&mut guess` thay vì `&guess` để làm nó mutable. (Chương 4 sẽ giải thích references kỹ hơn.)

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Xử lý lỗi tiềm ẩn với `Result`

Chúng ta vẫn đang làm việc trên dòng code này. Bây giờ chúng ta đang thảo luận về phần thứ ba, nhưng lưu ý rằng nó vẫn là một phần của một dòng logic duy nhất. Phần tiếp theo là method này:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Chúng ta có thể viết code này như sau:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Tuy nhiên, một dòng dài khó đọc, nên tốt hơn là chia nhỏ nó ra. Thường thì nên thêm dòng mới và khoảng trắng để ngắt các dòng dài khi bạn gọi method với cú pháp `.method_name()`. Bây giờ hãy thảo luận xem dòng này làm gì.

Như đã đề cập trước đó, `read_line` đặt bất cứ thứ gì người dùng nhập vào string chúng ta truyền cho nó, nhưng nó cũng trả về một giá trị `Result`. [`Result`][result]<!-- ignore --> là một [_enumeration_][enums]<!-- ignore -->, thường được gọi là _enum_, là một type có thể ở một trong nhiều trạng thái khác nhau. Mỗi trạng thái có thể có được gọi là một _variant_.

[Chương 6][enums]<!-- ignore --> sẽ đề cập đến enum chi tiết hơn. Mục đích của các type `Result` này là để mã hóa thông tin xử lý lỗi.

Các variant của `Result` là `Ok` và `Err`. Variant `Ok` cho biết thao tác đã thành công, và nó chứa giá trị được tạo ra thành công. Variant `Err` có nghĩa là thao tác đã thất bại, và nó chứa thông tin về lý do thất bại.

Các giá trị của type `Result`, giống như các giá trị của bất kỳ type nào, có các method được định nghĩa trên chúng. Một instance của `Result` có method [`expect`][expect]<!-- ignore --> mà bạn có thể gọi. Nếu instance `Result` này là giá trị `Err`, `expect` sẽ làm chương trình crash và hiển thị thông báo mà bạn truyền làm đối số cho `expect`. Nếu method `read_line` trả về `Err`, đó có thể là kết quả của lỗi từ hệ điều hành bên dưới. Nếu instance `Result` này là giá trị `Ok`, `expect` sẽ lấy giá trị trả về mà `Ok` đang giữ và trả về chỉ giá trị đó cho bạn để bạn có thể sử dụng. Trong trường hợp này, giá trị đó là số byte trong input của người dùng.

Nếu bạn không gọi `expect`, chương trình vẫn biên dịch được, nhưng bạn sẽ nhận được một cảnh báo:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust cảnh báo rằng bạn chưa sử dụng giá trị `Result` được trả về từ `read_line`, cho biết chương trình chưa xử lý một lỗi có thể xảy ra.

Cách đúng để tắt cảnh báo là thực sự viết code xử lý lỗi, nhưng trong trường hợp này chúng ta chỉ muốn crash chương trình khi xảy ra vấn đề, nên chúng ta có thể dùng `expect`. Bạn sẽ học về việc recover từ lỗi ở [Chương 9][recover]<!-- ignore -->.

### In giá trị với `println!` Placeholders

Ngoài dấu ngoặc nhọn đóng, chỉ còn một dòng nữa cần thảo luận trong code cho đến nay:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Dòng này in ra string hiện đang chứa input của người dùng. Cặp dấu ngoặc nhọn `{}` là một placeholder: Hãy nghĩ `{}` như cái kẹp cua nhỏ giữ một giá trị tại chỗ. Khi in giá trị của một variable, tên variable có thể đặt bên trong dấu ngoặc nhọn. Khi in kết quả của việc tính toán một expression, đặt dấu ngoặc nhọn rỗng vào trong format string, sau đó theo sau format string bằng danh sách các expression phân cách bởi dấu phẩy để in trong mỗi placeholder dấu ngoặc nhọn rỗng theo thứ tự tương ứng. Việc in một variable và kết quả của một expression trong một lần gọi `println!` sẽ trông như sau:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Code này sẽ in ra `x = 5 and y + 2 = 12`.

### Kiểm tra phần đầu tiên

Hãy kiểm tra phần đầu tiên của trò chơi đoán số. Chạy nó bằng `cargo run`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Đến đây, phần đầu tiên của trò chơi đã xong: Chúng ta đang lấy input từ bàn phím và in nó ra.

## Tạo số bí mật

Tiếp theo, chúng ta cần tạo một số bí mật mà người dùng sẽ cố đoán. Số bí mật phải khác nhau mỗi lần để trò chơi thú vị khi chơi nhiều lần. Chúng ta sẽ dùng một số ngẫu nhiên từ 1 đến 100 để trò chơi không quá khó. Rust chưa có tính năng tạo số ngẫu nhiên trong standard library. Tuy nhiên, team Rust cung cấp một [`rand` crate][randcrate] với tính năng đó.

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### Tăng cường tính năng với Crate

Nhớ rằng một crate là một tập hợp các file source code Rust. Dự án chúng ta đang xây dựng là một binary crate, tức là một file thực thi. Crate `rand` là một library crate, chứa code được thiết kế để sử dụng trong các chương trình khác và không thể chạy độc lập.

Khả năng điều phối các external crate của Cargo là điểm mạnh thực sự của Cargo. Trước khi có thể viết code sử dụng `rand`, chúng ta cần sửa file _Cargo.toml_ để thêm crate `rand` làm dependency. Mở file đó ngay bây giờ và thêm dòng sau vào cuối, bên dưới phần header `[dependencies]` mà Cargo đã tạo cho bạn. Hãy chắc chắn chỉ định `rand` chính xác như chúng ta có ở đây, với số version này, hoặc các ví dụ code trong tutorial này có thể không hoạt động:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

Trong file _Cargo.toml_, mọi thứ theo sau một header đều là phần của section đó và tiếp tục cho đến khi section khác bắt đầu. Trong `[dependencies]`, bạn cho Cargo biết các external crate mà dự án phụ thuộc vào và phiên bản nào của các crate đó bạn yêu cầu. Trong trường hợp này, chúng ta chỉ định crate `rand` với semantic version specifier `0.8.5`. Cargo hiểu [Semantic Versioning][semver]<!-- ignore --> (đôi khi gọi là _SemVer_), là một tiêu chuẩn để viết số version. Specifier `0.8.5` thực ra là viết tắt của `^0.8.5`, nghĩa là bất kỳ version nào ít nhất là 0.8.5 nhưng dưới 0.9.0.

Cargo coi các version này có public API tương thích với version 0.8.5, và specification này đảm bảo bạn sẽ nhận được bản patch mới nhất vẫn biên dịch được với code trong chương này. Bất kỳ version 0.9.0 hoặc lớn hơn đều không được đảm bảo có cùng API như những gì các ví dụ sau đây sử dụng.

Bây giờ, mà không thay đổi bất kỳ code nào, hãy build dự án, như được hiển thị trong Listing 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="Output từ việc chạy `cargo build` sau khi thêm crate `rand` làm dependency">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

Bạn có thể thấy số version khác nhau (nhưng tất cả sẽ tương thích với code, nhờ SemVer!) và các dòng khác nhau (tùy thuộc vào hệ điều hành), và các dòng có thể theo thứ tự khác nhau.

Khi chúng ta bao gồm một external dependency, Cargo lấy các version mới nhất của mọi thứ mà dependency đó cần từ _registry_, là một bản sao dữ liệu từ [Crates.io][cratesio]. Crates.io là nơi mọi người trong hệ sinh thái Rust đăng các dự án Rust mã nguồn mở để người khác sử dụng.

Sau khi cập nhật registry, Cargo kiểm tra section `[dependencies]` và tải xuống các crate được liệt kê mà chưa được tải về. Trong trường hợp này, mặc dù chúng ta chỉ liệt kê `rand` làm dependency, Cargo cũng lấy các crate khác mà `rand` phụ thuộc vào để hoạt động. Sau khi tải về các crate, Rust biên dịch chúng rồi biên dịch dự án với các dependency có sẵn.

Nếu bạn chạy lại `cargo build` ngay lập tức mà không thay đổi gì, bạn sẽ không nhận được output nào ngoài dòng `Finished`. Cargo biết rằng nó đã tải và biên dịch các dependency, và bạn không thay đổi gì về chúng trong file _Cargo.toml_ của mình. Cargo cũng biết rằng bạn không thay đổi gì về code, nên nó cũng không biên dịch lại. Không có gì để làm, nó chỉ đơn giản là thoát ra.

Nếu bạn mở file _src/main.rs_, thực hiện một thay đổi nhỏ, lưu lại và build lại, bạn sẽ chỉ thấy hai dòng output:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Những dòng này cho thấy Cargo chỉ cập nhật bản build với thay đổi nhỏ của bạn trong file _src/main.rs_. Các dependency của bạn không thay đổi, nên Cargo biết nó có thể tái sử dụng những gì đã tải về và biên dịch cho chúng.

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### Đảm bảo builds có thể tái tạo lại

Cargo có một cơ chế đảm bảo rằng bạn có thể rebuild cùng một artifact mỗi lần bạn hoặc bất kỳ ai khác build code của bạn: Cargo sẽ chỉ sử dụng các version của các dependency bạn đã chỉ định cho đến khi bạn chỉ định khác. Ví dụ, giả sử tuần tới version 0.8.6 của crate `rand` ra mắt, và version đó chứa một bản sửa lỗi quan trọng, nhưng cũng chứa một regression sẽ làm hỏng code của bạn. Để xử lý điều này, Rust tạo file _Cargo.lock_ lần đầu tiên bạn chạy `cargo build`, do đó bây giờ chúng ta có file này trong thư mục _guessing_game_.

Khi bạn build một dự án lần đầu tiên, Cargo tính toán tất cả các version của các dependency phù hợp với tiêu chí và sau đó ghi chúng vào file _Cargo.lock_. Khi bạn build dự án trong tương lai, Cargo sẽ thấy rằng file _Cargo.lock_ tồn tại và sẽ sử dụng các version được chỉ định ở đó thay vì làm lại toàn bộ công việc tính toán version. Điều này cho phép bạn có một reproducible build tự động. Nói cách khác, dự án của bạn sẽ vẫn ở 0.8.5 cho đến khi bạn nâng cấp một cách tường minh, nhờ file _Cargo.lock_. Vì file _Cargo.lock_ quan trọng cho reproducible builds, nó thường được commit vào source control cùng với phần còn lại của code trong dự án của bạn.

#### Cập nhật crate để nhận version mới

Khi bạn _muốn_ cập nhật một crate, Cargo cung cấp lệnh `update`, sẽ bỏ qua file _Cargo.lock_ và tính toán tất cả các version mới nhất phù hợp với specifications của bạn trong _Cargo.toml_. Cargo sau đó sẽ ghi các version đó vào file _Cargo.lock_. Nếu không, theo mặc định, Cargo sẽ chỉ tìm kiếm các version lớn hơn 0.8.5 và nhỏ hơn 0.9.0. Nếu crate `rand` đã phát hành hai version mới 0.8.6 và 0.999.0, bạn sẽ thấy kết quả sau nếu chạy `cargo update`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.999.0)
```

Cargo bỏ qua bản release 0.999.0. Lúc này, bạn cũng sẽ nhận thấy một thay đổi trong file _Cargo.lock_ ghi lại rằng version của crate `rand` bạn đang sử dụng bây giờ là 0.8.6. Để sử dụng `rand` version 0.999.0 hoặc bất kỳ version nào trong series 0.999._x_, bạn phải cập nhật file _Cargo.toml_ thành như sau (đừng thực sự thực hiện thay đổi này vì các ví dụ sau giả định bạn đang sử dụng `rand` 0.8):

```toml
[dependencies]
rand = "0.999.0"
```

Lần tiếp theo bạn chạy `cargo build`, Cargo sẽ cập nhật registry của các crate có sẵn và đánh giá lại các yêu cầu `rand` của bạn theo version mới bạn đã chỉ định.

Còn nhiều điều để nói về [Cargo][doccargo]<!-- ignore --> và [hệ sinh thái của nó][doccratesio]<!-- ignore -->, mà chúng ta sẽ thảo luận ở Chương 14, nhưng hiện tại, đó là tất cả những gì bạn cần biết. Cargo làm cho việc tái sử dụng thư viện rất dễ dàng, vì vậy Rustaceans có thể viết các dự án nhỏ hơn được lắp ráp từ nhiều package.

### Tạo số ngẫu nhiên

Hãy bắt đầu sử dụng `rand` để tạo một số để đoán. Bước tiếp theo là cập nhật _src/main.rs_, như được hiển thị trong Listing 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Thêm code để tạo số ngẫu nhiên">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Đầu tiên, chúng ta thêm dòng `use rand::Rng;`. Trait `Rng` định nghĩa các method mà các random number generator implement, và trait này phải nằm trong scope để chúng ta có thể sử dụng những method đó. Chương 10 sẽ đề cập đến trait chi tiết.

Tiếp theo, chúng ta thêm hai dòng ở giữa. Trong dòng đầu tiên, chúng ta gọi function `rand::thread_rng` cho chúng ta random number generator cụ thể mà chúng ta sẽ sử dụng: một cái là local với thread thực thi hiện tại và được seeded bởi hệ điều hành. Sau đó, chúng ta gọi method `gen_range` trên random number generator. Method này được định nghĩa bởi trait `Rng` mà chúng ta đã đưa vào scope với câu lệnh `use rand::Rng;`. Method `gen_range` nhận một range expression làm đối số và tạo ra một số ngẫu nhiên trong khoảng đó. Loại range expression chúng ta đang dùng ở đây có dạng `start..=end` và bao gồm cả biên dưới và biên trên, nên chúng ta cần chỉ định `1..=100` để yêu cầu một số từ 1 đến 100.

> Lưu ý: Bạn sẽ không chỉ biết ngay trait nào cần dùng và method, function nào cần gọi từ một crate, vì vậy mỗi crate đều có tài liệu với hướng dẫn sử dụng. Một tính năng hay khác của Cargo là chạy lệnh `cargo doc --open` sẽ build tài liệu được cung cấp bởi tất cả các dependency cục bộ và mở nó trong trình duyệt của bạn. Nếu bạn quan tâm đến các tính năng khác trong crate `rand`, ví dụ, hãy chạy `cargo doc --open` và nhấp vào `rand` trong sidebar bên trái.

Dòng mới thứ hai in ra số bí mật. Điều này hữu ích khi chúng ta đang phát triển chương trình để có thể kiểm tra nó, nhưng chúng ta sẽ xóa nó khỏi version cuối cùng. Nó không còn là trò chơi nếu chương trình in ra câu trả lời ngay khi khởi động!

Thử chạy chương trình vài lần:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Bạn sẽ nhận được các số ngẫu nhiên khác nhau, và tất cả chúng đều phải là số từ 1 đến 100. Tuyệt vời!

## So sánh dự đoán với số bí mật

Bây giờ chúng ta đã có input của người dùng và một số ngẫu nhiên, chúng ta có thể so sánh chúng. Bước đó được hiển thị trong Listing 2-4. Lưu ý rằng code này chưa biên dịch được, như chúng ta sẽ giải thích.

<Listing number="2-4" file-name="src/main.rs" caption="Xử lý các giá trị trả về có thể có khi so sánh hai số">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Đầu tiên, chúng ta thêm một câu lệnh `use` khác, đưa một type gọi là `std::cmp::Ordering` vào scope từ standard library. Type `Ordering` là một enum khác và có các variant `Less`, `Greater`, và `Equal`. Đây là ba kết quả có thể xảy ra khi bạn so sánh hai giá trị.

Sau đó, chúng ta thêm năm dòng mới ở cuối sử dụng type `Ordering`. Method `cmp` so sánh hai giá trị và có thể được gọi trên bất cứ thứ gì có thể so sánh. Nó nhận một reference đến bất cứ thứ gì bạn muốn so sánh với: Ở đây, nó đang so sánh `guess` với `secret_number`. Sau đó, nó trả về một variant của enum `Ordering` mà chúng ta đã đưa vào scope với câu lệnh `use`. Chúng ta sử dụng một expression [`match`][match]<!-- ignore --> để quyết định phải làm gì tiếp theo dựa trên variant nào của `Ordering` được trả về từ lần gọi `cmp` với các giá trị trong `guess` và `secret_number`.

Một expression `match` được tạo thành từ các _arm_. Một arm bao gồm một _pattern_ để khớp với, và code sẽ được chạy nếu giá trị được đưa cho `match` phù hợp với pattern của arm đó. Rust lấy giá trị được đưa cho `match` và kiểm tra qua từng pattern của arm. Patterns và cấu trúc `match` là các tính năng mạnh mẽ của Rust: Chúng cho phép bạn diễn đạt nhiều tình huống khác nhau mà code có thể gặp phải, và đảm bảo bạn xử lý tất cả chúng. Những tính năng này sẽ được đề cập chi tiết trong Chương 6 và Chương 19.

Hãy đi qua một ví dụ với expression `match` chúng ta sử dụng ở đây. Giả sử người dùng đã đoán 50 và số bí mật được tạo ngẫu nhiên lần này là 38.

Khi code so sánh 50 với 38, method `cmp` sẽ trả về `Ordering::Greater` vì 50 lớn hơn 38. Expression `match` nhận giá trị `Ordering::Greater` và bắt đầu kiểm tra từng pattern của arm. Nó xem pattern của arm đầu tiên, `Ordering::Less`, và thấy rằng giá trị `Ordering::Greater` không khớp với `Ordering::Less`, nên nó bỏ qua code trong arm đó và chuyển sang arm tiếp theo. Pattern của arm tiếp theo là `Ordering::Greater`, khớp với `Ordering::Greater`! Code liên quan trong arm đó sẽ thực thi và in `Too big!` ra màn hình. Expression `match` kết thúc sau lần khớp thành công đầu tiên, nên nó sẽ không xem arm cuối cùng trong kịch bản này.

Tuy nhiên, code trong Listing 2-4 chưa biên dịch được. Hãy thử nó:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Cốt lõi của lỗi cho biết có _mismatched types_ (kiểu không khớp). Rust có một hệ thống type mạnh, tĩnh. Tuy nhiên, nó cũng có type inference. Khi chúng ta viết `let mut guess = String::new()`, Rust có thể suy ra rằng `guess` phải là một `String` và không yêu cầu chúng ta viết type. Mặt khác, `secret_number` là một number type. Một số number type của Rust có thể có giá trị từ 1 đến 100: `i32`, số 32-bit; `u32`, số 32-bit không dấu; `i64`, số 64-bit; và nhiều type khác. Trừ khi được chỉ định khác, Rust mặc định là `i32`, đó là type của `secret_number` trừ khi bạn thêm thông tin type ở nơi khác khiến Rust suy ra một numerical type khác. Lý do của lỗi là Rust không thể so sánh một string với một number type.

Cuối cùng, chúng ta muốn chuyển đổi `String` mà chương trình đọc làm input thành một number type để chúng ta có thể so sánh nó số học với số bí mật. Chúng ta thực hiện điều này bằng cách thêm dòng này vào phần thân của function `main`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Dòng đó là:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Chúng ta tạo một variable tên `guess`. Nhưng khoan, chương trình đã có một variable tên `guess` rồi? Có, nhưng Rust cho phép chúng ta shadow giá trị trước đó của `guess` với một giá trị mới. _Shadowing_ cho phép chúng ta tái sử dụng tên variable `guess` thay vì buộc chúng ta tạo hai variable riêng biệt, chẳng hạn như `guess_str` và `guess`. Chúng ta sẽ đề cập điều này chi tiết hơn ở [Chương 3][shadowing]<!-- ignore -->, nhưng hiện tại, biết rằng tính năng này thường được sử dụng khi bạn muốn chuyển đổi một giá trị từ một type sang type khác.

Chúng ta bind variable mới này với expression `guess.trim().parse()`. `guess` trong expression đề cập đến variable `guess` gốc chứa input dưới dạng string. Method `trim` trên một instance `String` sẽ loại bỏ bất kỳ khoảng trắng nào ở đầu và cuối, điều này chúng ta phải làm trước khi có thể chuyển đổi string thành `u32`, chỉ có thể chứa dữ liệu số. Người dùng phải nhấn <kbd>enter</kbd> để thỏa mãn `read_line` và nhập dự đoán của họ, điều này thêm một ký tự newline vào string. Ví dụ, nếu người dùng gõ <kbd>5</kbd> và nhấn <kbd>enter</kbd>, `guess` trông như thế này: `5\n`. `\n` đại diện cho "newline." (Trên Windows, nhấn <kbd>enter</kbd> kết quả trong một carriage return và một newline, `\r\n`.) Method `trim` loại bỏ `\n` hoặc `\r\n`, chỉ còn lại `5`.

Method [`parse` trên strings][parse]<!-- ignore --> chuyển đổi một string sang type khác. Ở đây, chúng ta dùng nó để chuyển từ string sang number. Chúng ta cần cho Rust biết chính xác number type chúng ta muốn bằng cách sử dụng `let guess: u32`. Dấu hai chấm (`:`) sau `guess` cho Rust biết chúng ta sẽ annotate type của variable. Rust có một vài kiểu số tích hợp; `u32` ở đây là một unsigned, 32-bit integer. Đây là lựa chọn mặc định tốt cho một số dương nhỏ. Bạn sẽ học về các number type khác ở [Chương 3][integers]<!-- ignore -->.

Ngoài ra, annotation `u32` trong chương trình ví dụ này và việc so sánh với `secret_number` có nghĩa là Rust sẽ suy ra rằng `secret_number` cũng phải là `u32`. Vì vậy, bây giờ việc so sánh sẽ là giữa hai giá trị cùng type!

Method `parse` sẽ chỉ hoạt động trên các ký tự có thể chuyển đổi hợp lý thành số và vì vậy có thể dễ dàng gây ra lỗi. Nếu, ví dụ, string chứa `A👍%`, sẽ không có cách nào để chuyển đổi thành số. Vì nó có thể thất bại, method `parse` trả về một type `Result`, giống như method `read_line` (đã thảo luận trước đó trong ["Xử lý lỗi tiềm ẩn với `Result`"](#handling-potential-failure-with-result)<!-- ignore -->). Chúng ta sẽ xử lý `Result` này theo cách tương tự bằng cách sử dụng lại method `expect`. Nếu `parse` trả về variant `Err` của `Result` vì không thể tạo số từ string, lần gọi `expect` sẽ crash trò chơi và in thông báo chúng ta đưa cho nó. Nếu `parse` có thể chuyển đổi thành công string thành số, nó sẽ trả về variant `Ok` của `Result`, và `expect` sẽ trả về số mà chúng ta muốn từ giá trị `Ok`.

Hãy chạy chương trình ngay bây giờ:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Tuyệt! Mặc dù có khoảng trắng trước dự đoán, chương trình vẫn xác định được rằng người dùng đã đoán 76. Chạy chương trình vài lần để kiểm tra các hành vi khác nhau với các loại input khác nhau: Đoán đúng số, đoán số quá cao, và đoán số quá thấp.

Chúng ta đã có hầu hết trò chơi hoạt động rồi, nhưng người dùng chỉ có thể đoán một lần. Hãy thay đổi điều đó bằng cách thêm một vòng lặp!

## Cho phép đoán nhiều lần với vòng lặp

Từ khóa `loop` tạo một vòng lặp vô hạn. Chúng ta sẽ thêm một vòng lặp để cho người dùng có thêm cơ hội đoán số:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Như bạn thấy, chúng ta đã di chuyển mọi thứ từ phần prompt nhập dự đoán trở đi vào trong một vòng lặp. Hãy chắc chắn indent các dòng bên trong vòng lặp thêm bốn khoảng trắng nữa và chạy lại chương trình. Chương trình bây giờ sẽ yêu cầu đoán thêm mãi mãi, điều này thực ra tạo ra một vấn đề mới. Có vẻ như người dùng không thể thoát!

Người dùng luôn có thể ngắt chương trình bằng cách sử dụng phím tắt <kbd>ctrl</kbd>-<kbd>C</kbd>. Nhưng có một cách khác để thoát khỏi con quái vật không thỏa mãn này, như đã đề cập trong phần thảo luận về `parse` trong ["So sánh dự đoán với số bí mật"](#comparing-the-guess-to-the-secret-number)<!-- ignore -->: Nếu người dùng nhập một câu trả lời không phải số, chương trình sẽ crash. Chúng ta có thể tận dụng điều đó để cho phép người dùng thoát, như được hiển thị ở đây:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Gõ `quit` sẽ thoát trò chơi, nhưng như bạn sẽ nhận thấy, nhập bất kỳ input không phải số nào cũng vậy. Điều này không tối ưu, nói thẳng ra; chúng ta muốn trò chơi cũng dừng lại khi đoán đúng số.

### Thoát sau khi đoán đúng

Hãy lập trình trò chơi thoát khi người dùng thắng bằng cách thêm câu lệnh `break`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Thêm dòng `break` sau `You win!` làm cho chương trình thoát khỏi vòng lặp khi người dùng đoán đúng số bí mật. Thoát khỏi vòng lặp cũng có nghĩa là thoát khỏi chương trình, vì vòng lặp là phần cuối cùng của `main`.

### Xử lý input không hợp lệ

Để tinh chỉnh thêm hành vi của trò chơi, thay vì crash chương trình khi người dùng nhập một số không phải số, hãy làm cho trò chơi bỏ qua nó để người dùng có thể tiếp tục đoán. Chúng ta có thể làm điều đó bằng cách thay đổi dòng mà `guess` được chuyển đổi từ `String` sang `u32`, như được hiển thị trong Listing 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Bỏ qua dự đoán không phải số và yêu cầu đoán khác thay vì crash chương trình">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Chúng ta chuyển từ lần gọi `expect` sang expression `match` để chuyển từ crash khi có lỗi sang xử lý lỗi. Nhớ rằng `parse` trả về type `Result` và `Result` là một enum có các variant `Ok` và `Err`. Chúng ta đang sử dụng expression `match` ở đây, như chúng ta đã làm với kết quả `Ordering` của method `cmp`.

Nếu `parse` có thể chuyển đổi thành công string thành số, nó sẽ trả về giá trị `Ok` chứa số kết quả. Giá trị `Ok` đó sẽ khớp với pattern của arm đầu tiên, và expression `match` sẽ chỉ trả về giá trị `num` mà `parse` tạo ra và đặt vào bên trong giá trị `Ok`. Số đó sẽ kết thúc đúng chỗ chúng ta muốn trong variable `guess` mới mà chúng ta đang tạo.

Nếu `parse` _không thể_ chuyển đổi string thành số, nó sẽ trả về giá trị `Err` chứa thêm thông tin về lỗi. Giá trị `Err` không khớp với pattern `Ok(num)` trong arm đầu tiên của `match`, nhưng nó khớp với pattern `Err(_)` trong arm thứ hai. Dấu gạch dưới `_` là một catch-all value; trong ví dụ này, chúng ta đang nói chúng ta muốn khớp với tất cả các giá trị `Err`, bất kể thông tin nào chúng có bên trong. Vì vậy, chương trình sẽ thực thi code của arm thứ hai, `continue`, cho chương trình biết hãy chuyển sang iteration tiếp theo của `loop` và yêu cầu đoán thêm. Vì vậy, thực tế là chương trình bỏ qua tất cả các lỗi mà `parse` có thể gặp!

Bây giờ mọi thứ trong chương trình sẽ hoạt động như mong đợi. Hãy thử nó:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Tuyệt vời! Với một chỉnh sửa nhỏ cuối cùng, chúng ta sẽ hoàn thành trò chơi đoán số. Nhớ lại rằng chương trình vẫn đang in số bí mật. Điều đó hoạt động tốt cho việc kiểm tra, nhưng nó làm hỏng trò chơi. Hãy xóa câu lệnh `println!` in số bí mật. Listing 2-6 hiển thị code cuối cùng.

<Listing number="2-6" file-name="src/main.rs" caption="Code hoàn chỉnh của trò chơi đoán số">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Đến đây, bạn đã xây dựng thành công trò chơi đoán số. Xin chúc mừng!

## Tóm tắt

Dự án này là cách thực hành để giới thiệu cho bạn nhiều khái niệm mới của Rust: `let`, `match`, functions, việc sử dụng external crates, và nhiều hơn nữa. Trong vài chương tiếp theo, bạn sẽ tìm hiểu về những khái niệm này chi tiết hơn. Chương 3 đề cập đến các khái niệm mà hầu hết các ngôn ngữ lập trình đều có, chẳng hạn như variables, data types, và functions, và cho thấy cách sử dụng chúng trong Rust. Chương 4 khám phá ownership, một tính năng làm cho Rust khác biệt với các ngôn ngữ khác. Chương 5 thảo luận về structs và method syntax, và Chương 6 giải thích cách enum hoạt động.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
