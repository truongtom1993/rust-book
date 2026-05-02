## Data Types

Mỗi giá trị trong Rust đều thuộc một _data type_ nhất định, cho Rust biết loại dữ liệu nào đang được chỉ định để biết cách làm việc với dữ liệu đó. Chúng ta sẽ xem xét hai tập con data type: scalar và compound.

Hãy nhớ rằng Rust là một ngôn ngữ _statically typed_, có nghĩa là nó phải biết kiểu của tất cả variables tại compile time. Compiler thường có thể suy luận kiểu chúng ta muốn dùng dựa trên giá trị và cách chúng ta dùng nó. Trong những trường hợp khi nhiều kiểu có thể phù hợp, chẳng hạn như khi chúng ta convert một `String` thành kiểu số dùng `parse` trong phần ["So Sánh Dự Đoán Với Số Bí Mật"][comparing-the-guess-to-the-secret-number]<!-- ignore --> trong Chương 2, chúng ta phải thêm type annotation, như sau:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Nếu chúng ta không thêm type annotation `: u32` như trong code trên, Rust sẽ hiển thị lỗi sau, có nghĩa là compiler cần thêm thông tin từ chúng ta để biết kiểu nào chúng ta muốn dùng:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Bạn sẽ thấy các type annotations khác nhau cho các data types khác.

### Scalar Types

Một _scalar_ type đại diện cho một giá trị đơn lẻ. Rust có bốn scalar types chính: integers, floating-point numbers, Booleans, và characters. Bạn có thể nhận ra những điều này từ các ngôn ngữ lập trình khác. Hãy cùng tìm hiểu cách chúng hoạt động trong Rust.

#### Integer Types

Một _integer_ là một số không có phần thập phân. Chúng ta đã dùng một integer type trong Chương 2, kiểu `u32`. Khai báo kiểu này chỉ ra rằng giá trị liên kết với nó phải là một unsigned integer (các signed integer types bắt đầu bằng `i` thay vì `u`) chiếm 32 bits. Bảng 3-1 hiển thị các integer types có sẵn trong Rust. Chúng ta có thể dùng bất kỳ biến thể nào trong số này để khai báo kiểu của một integer value.

<span class="caption">Bảng 3-1: Các Integer Types trong Rust</span>

| Độ dài  | Signed  | Unsigned |
| ------- | ------- | -------- |
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| Phụ thuộc kiến trúc | `isize` | `usize`  |

Mỗi biến thể có thể là signed hoặc unsigned và có kích thước rõ ràng. _Signed_ và _unsigned_ đề cập đến việc liệu số đó có thể âm không — nói cách khác, liệu số đó có cần dấu kèm theo (signed) hay sẽ chỉ là số dương và do đó có thể được biểu diễn không có dấu (unsigned). Giống như viết số trên giấy: Khi dấu quan trọng, một số được hiển thị với dấu cộng hoặc dấu trừ; tuy nhiên, khi an toàn để giả định số là dương, nó được hiển thị không có dấu. Các số signed được lưu trữ dùng biểu diễn [bù hai][twos-complement]<!-- ignore -->.

Mỗi biến thể signed có thể lưu các số từ −(2<sup>n − 1</sup>) đến 2<sup>n − 1</sup> − 1 (bao gồm), trong đó _n_ là số bits mà biến thể đó dùng. Vì vậy, một `i8` có thể lưu các số từ −(2<sup>7</sup>) đến 2<sup>7</sup> − 1, tương đương −128 đến 127. Các biến thể unsigned có thể lưu các số từ 0 đến 2<sup>n</sup> − 1, vì vậy một `u8` có thể lưu các số từ 0 đến 2<sup>8</sup> − 1, tương đương 0 đến 255.

Ngoài ra, các kiểu `isize` và `usize` phụ thuộc vào kiến trúc của máy tính mà chương trình của bạn đang chạy: 64 bits nếu bạn đang ở kiến trúc 64-bit và 32 bits nếu bạn đang ở kiến trúc 32-bit.

Bạn có thể viết integer literals ở bất kỳ định dạng nào được hiển thị trong Bảng 3-2. Lưu ý rằng các number literals có thể thuộc nhiều kiểu số cho phép thêm type suffix, chẳng hạn như `57u8`, để chỉ định kiểu. Number literals cũng có thể dùng `_` như dấu phân cách trực quan để đọc số dễ hơn, chẳng hạn như `1_000`, sẽ có cùng giá trị như nếu bạn đã chỉ định `1000`.

<span class="caption">Bảng 3-2: Integer Literals trong Rust</span>

| Number literals  | Ví dụ         |
| ---------------- | ------------- |
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |
| Byte (chỉ `u8`) | `b'A'`        |

Vậy làm sao bạn biết nên dùng integer type nào? Nếu bạn không chắc, các giá trị mặc định của Rust thường là điểm khởi đầu tốt: Các integer types mặc định là `i32`. Tình huống chính mà bạn sẽ dùng `isize` hoặc `usize` là khi indexing một collection nào đó.

> ##### Integer Overflow
>
> Giả sử bạn có một variable kiểu `u8` có thể giữ các giá trị từ 0 đến 255. Nếu bạn cố thay đổi variable thành một giá trị nằm ngoài phạm vi đó, chẳng hạn như 256, _integer overflow_ sẽ xảy ra, có thể dẫn đến một trong hai hành vi. Khi bạn compile ở debug mode, Rust bao gồm các kiểm tra cho integer overflow khiến chương trình của bạn _panic_ tại runtime nếu hành vi này xảy ra. Rust dùng thuật ngữ _panicking_ khi một chương trình thoát với lỗi; chúng ta sẽ thảo luận về panics sâu hơn trong phần ["Unrecoverable Errors với `panic!`"][unrecoverable-errors-with-panic]<!-- ignore --> trong Chương 9.
>
> Khi bạn compile ở release mode với flag `--release`, Rust _không_ bao gồm các kiểm tra cho integer overflow gây ra panics. Thay vào đó, nếu overflow xảy ra, Rust thực hiện _two's complement wrapping_. Nói tóm lại, các giá trị lớn hơn giá trị tối đa mà kiểu có thể giữ "cuộn vòng" về giá trị tối thiểu của các giá trị mà kiểu có thể giữ. Trong trường hợp của `u8`, giá trị 256 trở thành 0, giá trị 257 trở thành 1, v.v. Chương trình sẽ không panic, nhưng variable sẽ có giá trị mà có thể không phải là điều bạn mong đợi. Dựa vào hành vi wrapping của integer overflow được coi là lỗi.
>
> Để xử lý rõ ràng khả năng overflow, bạn có thể dùng các nhóm methods sau được cung cấp bởi standard library cho các primitive numeric types:
>
> - Wrap trong tất cả các modes với các methods `wrapping_*`, chẳng hạn như `wrapping_add`.
> - Trả về giá trị `None` nếu có overflow với các methods `checked_*`.
> - Trả về giá trị và một Boolean cho biết liệu có overflow không với các methods `overflowing_*`.
> - Saturate tại giá trị tối thiểu hoặc tối đa của kiểu với các methods `saturating_*`.

#### Floating-Point Types

Rust cũng có hai primitive types cho _floating-point numbers_, là các số có dấu thập phân. Các floating-point types của Rust là `f32` và `f64`, có kích thước lần lượt là 32 bits và 64 bits. Kiểu mặc định là `f64` vì trên các CPU hiện đại, nó có tốc độ gần tương đương `f32` nhưng có độ chính xác cao hơn. Tất cả các floating-point types đều là signed.

Đây là một ví dụ minh họa floating-point numbers trong hoạt động:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Floating-point numbers được biểu diễn theo tiêu chuẩn IEEE-754.

#### Numeric Operations

Rust hỗ trợ các phép toán toán học cơ bản mà bạn mong đợi cho tất cả các kiểu số: cộng, trừ, nhân, chia và phần dư. Phép chia integer cắt về phía không tới số nguyên gần nhất. Code sau đây cho thấy cách bạn sẽ dùng mỗi numeric operation trong một `let` statement:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Mỗi expression trong các statements này dùng một toán tử toán học và đánh giá thành một giá trị duy nhất, sau đó được gán cho một variable. [Phụ lục B][appendix_b]<!-- ignore --> chứa danh sách tất cả các operators mà Rust cung cấp.

#### The Boolean Type

Như trong hầu hết các ngôn ngữ lập trình khác, một Boolean type trong Rust có hai giá trị có thể: `true` và `false`. Booleans có kích thước một byte. Boolean type trong Rust được chỉ định bằng `bool`. Ví dụ:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Cách chính để dùng Boolean values là thông qua các conditionals, chẳng hạn như `if` expression. Chúng ta sẽ đề cập đến cách `if` expressions hoạt động trong Rust trong phần ["Control Flow"][control-flow]<!-- ignore -->.

#### The Character Type

Kiểu `char` của Rust là kiểu alphabetic primitive nhất của ngôn ngữ. Dưới đây là một số ví dụ khai báo `char` values:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Lưu ý rằng chúng ta chỉ định `char` literals bằng dấu nháy đơn, trái ngược với string literals dùng dấu nháy kép. Kiểu `char` của Rust có kích thước 4 bytes và đại diện cho một Unicode scalar value, có nghĩa là nó có thể biểu diễn nhiều hơn chỉ ASCII. Chữ cái có dấu; ký tự Trung Quốc, Nhật Bản và Hàn Quốc; emoji; và zero-width spaces đều là các `char` values hợp lệ trong Rust. Unicode scalar values nằm trong khoảng từ `U+0000` đến `U+D7FF` và `U+E000` đến `U+10FFFF` (bao gồm). Tuy nhiên, "character" thực ra không phải là một khái niệm trong Unicode, vì vậy trực giác của con người về "character" có thể không khớp với `char` trong Rust. Chúng ta sẽ thảo luận chủ đề này chi tiết trong ["Storing UTF-8 Encoded Text with Strings"][strings]<!-- ignore --> ở Chương 8.

### Compound Types

_Compound types_ có thể nhóm nhiều giá trị vào một kiểu. Rust có hai primitive compound types: tuples và arrays.

#### The Tuple Type

Một _tuple_ là một cách chung để nhóm một số giá trị với nhiều kiểu khác nhau thành một compound type. Tuples có độ dài cố định: Một khi được khai báo, chúng không thể tăng hay giảm kích thước.

Chúng ta tạo một tuple bằng cách viết một danh sách các giá trị phân tách bằng dấu phẩy bên trong dấu ngoặc đơn. Mỗi vị trí trong tuple có một kiểu, và các kiểu của các giá trị khác nhau trong tuple không cần phải giống nhau. Chúng ta đã thêm các type annotations tùy chọn trong ví dụ này:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

Variable `tup` gán cho toàn bộ tuple vì một tuple được coi là một phần tử compound đơn lẻ. Để lấy các giá trị riêng lẻ ra từ một tuple, chúng ta có thể dùng pattern matching để destructure một tuple value, như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Chương trình này đầu tiên tạo một tuple và gán nó cho variable `tup`. Sau đó nó dùng một pattern với `let` để lấy `tup` và biến nó thành ba variables riêng biệt, `x`, `y`, và `z`. Đây được gọi là _destructuring_ vì nó phá vỡ tuple đơn thành ba phần. Cuối cùng, chương trình in giá trị của `y`, là `6.4`.

Chúng ta cũng có thể truy cập một phần tử tuple trực tiếp bằng cách dùng dấu chấm (`.`) theo sau là index của giá trị chúng ta muốn truy cập. Ví dụ:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Chương trình này tạo tuple `x` rồi truy cập từng phần tử của tuple bằng các index tương ứng của chúng. Như với hầu hết các ngôn ngữ lập trình, index đầu tiên trong một tuple là 0.

Tuple không có bất kỳ giá trị nào có tên đặc biệt là _unit_. Giá trị này và kiểu tương ứng của nó đều được viết là `()` và đại diện cho một giá trị rỗng hoặc một kiểu trả về rỗng. Các expressions ngầm trả về unit value nếu chúng không trả về bất kỳ giá trị nào khác.

#### The Array Type

Một cách khác để có một collection của nhiều giá trị là với một _array_. Không giống như tuple, mỗi phần tử của một array phải có cùng kiểu. Không giống như arrays trong một số ngôn ngữ khác, arrays trong Rust có độ dài cố định.

Chúng ta viết các giá trị trong một array dưới dạng danh sách phân tách bằng dấu phẩy bên trong dấu ngoặc vuông:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Arrays hữu ích khi bạn muốn dữ liệu của mình được cấp phát trên stack, giống như các kiểu khác chúng ta đã thấy cho đến nay, thay vì trên heap (chúng ta sẽ thảo luận về stack và heap nhiều hơn trong [Chương 4][stack-and-heap]<!-- ignore -->) hoặc khi bạn muốn đảm bảo rằng bạn luôn có một số lượng phần tử cố định. Tuy nhiên, một array không linh hoạt như kiểu vector. Một vector là một collection type tương tự được cung cấp bởi standard library mà _được phép_ tăng hoặc giảm kích thước vì nội dung của nó nằm trên heap. Nếu bạn không chắc nên dùng array hay vector, có lẽ bạn nên dùng vector. [Chương 8][vectors]<!-- ignore --> thảo luận về vectors chi tiết hơn.

Tuy nhiên, arrays hữu ích hơn khi bạn biết số lượng phần tử sẽ không cần thay đổi. Ví dụ, nếu bạn dùng tên các tháng trong một chương trình, bạn có thể sẽ dùng một array thay vì một vector vì bạn biết nó sẽ luôn chứa 12 phần tử:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Bạn viết kiểu của một array bằng cách dùng dấu ngoặc vuông với kiểu của mỗi phần tử, một dấu chấm phẩy, và sau đó là số lượng phần tử trong array, như sau:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Ở đây, `i32` là kiểu của mỗi phần tử. Sau dấu chấm phẩy, số `5` chỉ ra array chứa năm phần tử.

Bạn cũng có thể khởi tạo một array để chứa cùng một giá trị cho mỗi phần tử bằng cách chỉ định giá trị ban đầu, theo sau là dấu chấm phẩy, và sau đó là độ dài của array trong dấu ngoặc vuông, như ví dụ sau:

```rust
let a = [3; 5];
```

Array tên `a` sẽ chứa `5` phần tử đều được đặt ban đầu thành giá trị `3`. Điều này tương đương với `let a = [3, 3, 3, 3, 3];` nhưng ngắn gọn hơn.

<!-- Old headings. Do not remove or links may break. -->
<a id="accessing-array-elements"></a>

#### Truy Cập Phần Tử Array

Một array là một khối bộ nhớ đơn có kích thước cố định đã biết, có thể được cấp phát trên stack. Bạn có thể truy cập các phần tử của một array dùng indexing, như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Trong ví dụ này, variable tên `first` sẽ nhận giá trị `1` vì đó là giá trị tại index `[0]` trong array. Variable tên `second` sẽ nhận giá trị `2` từ index `[1]` trong array.

#### Truy Cập Phần Tử Array Không Hợp Lệ

Hãy xem điều gì xảy ra nếu bạn cố truy cập một phần tử của array vượt qua cuối array. Giả sử bạn chạy code này, tương tự như guessing game trong Chương 2, để lấy một array index từ người dùng:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Code này compile thành công. Nếu bạn chạy code này dùng `cargo run` và nhập `0`, `1`, `2`, `3`, hoặc `4`, chương trình sẽ in ra giá trị tương ứng tại index đó trong array. Nếu bạn nhập một số vượt qua cuối array, chẳng hạn như `10`, bạn sẽ thấy output như sau:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Chương trình đã dẫn đến runtime error tại điểm dùng giá trị không hợp lệ trong indexing operation. Chương trình thoát với thông báo lỗi và không thực thi câu lệnh `println!` cuối cùng. Khi bạn cố truy cập một phần tử dùng indexing, Rust sẽ kiểm tra rằng index bạn đã chỉ định nhỏ hơn độ dài array. Nếu index lớn hơn hoặc bằng độ dài, Rust sẽ panic. Kiểm tra này phải xảy ra tại runtime, đặc biệt trong trường hợp này, vì compiler không thể biết trước giá trị nào người dùng sẽ nhập khi họ chạy code sau.

Đây là ví dụ về các nguyên tắc memory safety của Rust trong hoạt động. Trong nhiều ngôn ngữ low-level, loại kiểm tra này không được thực hiện, và khi bạn cung cấp một index không đúng, bộ nhớ không hợp lệ có thể được truy cập. Rust bảo vệ bạn khỏi loại lỗi này bằng cách thoát ngay lập tức thay vì cho phép truy cập bộ nhớ và tiếp tục. Chương 9 thảo luận thêm về error handling của Rust và cách bạn có thể viết code có thể đọc được, an toàn mà không panic và cũng không cho phép truy cập bộ nhớ không hợp lệ.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
