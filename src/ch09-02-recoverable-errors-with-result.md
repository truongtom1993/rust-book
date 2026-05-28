## Lỗi Có Thể Phục Hồi với `Result`

Hầu hết các lỗi không nghiêm trọng đủ để yêu cầu chương trình phải dừng hoàn toàn. Đôi khi khi một function thất bại, đó là vì một lý do mà bạn có thể dễ dàng diễn giải và phản hồi. Ví dụ, nếu bạn cố gắng mở một file và hoạt động đó thất bại vì file không tồn tại, bạn có thể muốn tạo file thay vì kết thúc quy trình.

Hãy nhớ lại từ [“Xử lý Lỗi Tiềm Năng với `Result`”][handle_failure]<!-- ignore --> trong Chương 2 rằng enum `Result` được định nghĩa có hai biến thể, `Ok` và `Err`, như sau:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` và `E` là các generic type parameter: Chúng ta sẽ thảo luận về generics chi tiết hơn trong Chương 10. Những gì bạn cần biết ngay bây giờ là `T` đại diện cho kiểu của giá trị sẽ được trả về trong một trường hợp thành công trong biến thể `Ok`, và `E` đại diện cho kiểu của lỗi sẽ được trả về trong trường hợp thất bại trong biến thể `Err`. Vì `Result` có các generic type parameter này, chúng ta có thể sử dụng kiểu `Result` và các function được định nghĩa trên nó trong nhiều tình huống khác nhau nơi giá trị thành công và giá trị lỗi mà chúng ta muốn trả về có thể khác nhau.

Hãy gọi một function trả về giá trị `Result` vì function có thể thất bại. Trong Listing 9-3, chúng ta cố gắng mở một file.

<Listing number="9-3" file-name="src/main.rs" caption="Mở một file">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

Kiểu trả về của `File::open` là `Result<T, E>`. Generic parameter `T` đã được điền bởi cài đặt của `File::open` với kiểu của giá trị thành công, `std::fs::File`, đây là một file handle. Kiểu của `E` được sử dụng trong giá trị lỗi là `std::io::Error`. Kiểu trả về này có nghĩa là lệnh gọi `File::open` có thể thành công và trả về một file handle mà chúng ta có thể đọc hoặc ghi. Lệnh gọi function cũng có thể thất bại: Ví dụ, file có thể không tồn tại, hoặc chúng ta có thể không có quyền truy cập file. Function `File::open` cần có cách để cho chúng ta biết nó đã thành công hay thất bại và đồng thời cung cấp cho chúng ta file handle hoặc thông tin lỗi. Thông tin này chính xác là những gì enum `Result` truyền đạt.

Trong trường hợp `File::open` thành công, giá trị trong biến `greeting_file_result` sẽ là một instance của `Ok` chứa một file handle. Trong trường hợp nó thất bại, giá trị trong `greeting_file_result` sẽ là một instance của `Err` chứa thêm thông tin về loại lỗi đã xảy ra.

Chúng ta cần thêm vào mã trong Listing 9-3 để thực hiện các hành động khác nhau tùy thuộc vào giá trị `File::open` trả về. Listing 9-4 hiển thị một cách để xử lý `Result` bằng một công cụ cơ bản, biểu thức `match` mà chúng ta đã thảo luận trong Chương 6.

<Listing number="9-4" file-name="src/main.rs" caption="Sử dụng biểu thức `match` để xử lý các biến thể `Result` có thể được trả về">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Lưu ý rằng, giống như enum `Option`, enum `Result` và các biến thể của nó đã được đưa vào phạm vi bởi prelude, vì vậy chúng ta không cần phải chỉ định `Result::` trước các biến thể `Ok` và `Err` trong các arm của `match`.

Khi kết quả là `Ok`, mã này sẽ trả về giá trị `file` bên trong biến thể `Ok`, và sau đó chúng ta gán giá trị file handle đó cho biến `greeting_file`. Sau `match`, chúng ta có thể sử dụng file handle để đọc hoặc ghi.

Arm khác của `match` xử lý trường hợp chúng ta nhận được giá trị `Err` từ `File::open`. Trong ví dụ này, chúng ta đã chọn gọi macro `panic!`. Nếu không có file có tên _hello.txt_ trong thư mục hiện tại của chúng ta và chúng ta chạy mã này, chúng ta sẽ thấy đầu ra sau từ macro `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Như thường lệ, đầu ra này cho chúng ta biết chính xác những gì đã xảy ra sai.

### Khớp với Các Lỗi Khác Nhau

Mã trong Listing 9-4 sẽ `panic!` bất kể lý do tại sao `File::open` thất bại. Tuy nhiên, chúng ta muốn thực hiện các hành động khác nhau cho các lý do thất bại khác nhau. Nếu `File::open` thất bại vì file không tồn tại, chúng ta muốn tạo file và trả về handle cho file mới. Nếu `File::open` thất bại vì lý do khác—ví dụ, vì chúng ta không có quyền mở file—chúng ta vẫn muốn mã `panic!` theo cách giống như trong Listing 9-4. Để làm điều này, chúng ta thêm biểu thức `match` bên trong, được hiển thị trong Listing 9-5.

<Listing number="9-5" file-name="src/main.rs" caption="Xử lý các loại lỗi khác nhau theo những cách khác nhau">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

Kiểu của giá trị mà `File::open` trả về bên trong biến thể `Err` là `io::Error`, một struct được cung cấp bởi thư viện tiêu chuẩn. Struct này có một method `kind` mà chúng ta có thể gọi để lấy giá trị `io::ErrorKind`. Enum `io::ErrorKind` được cung cấp bởi thư viện tiêu chuẩn và có các biến thể đại diện cho các loại lỗi khác nhau có thể xảy ra từ hoạt động `io`. Biến thể chúng ta muốn sử dụng là `ErrorKind::NotFound`, chỉ ra rằng file chúng ta đang cố gắng mở chưa tồn tại. Vì vậy, chúng ta khớp với `greeting_file_result`, nhưng chúng ta cũng có một match bên trong với `error.kind()`.

Điều kiện chúng ta muốn kiểm tra trong match bên trong là liệu giá trị được trả về bởi `error.kind()` có phải là biến thể `NotFound` của enum `ErrorKind` hay không. Nếu có, chúng ta cố gắng tạo file bằng `File::create`. Tuy nhiên, vì `File::create` cũng có thể thất bại, chúng ta cần một arm thứ hai trong biểu thức `match` bên trong. Khi file không thể được tạo, một thông báo lỗi khác được in ra. Arm thứ hai của `match` bên ngoài vẫn giữ nguyên, vì vậy chương trình panic trên bất kỳ lỗi nào ngoài lỗi file bị thiếu.

> #### Các Giải Pháp Thay Thế cho Việc Sử Dụng `match` với `Result<T, E>`
>
> Đó là rất nhiều `match`! Biểu thức `match` rất hữu ích nhưng cũng là một primitiv. Trong Chương 13, bạn sẽ tìm hiểu về closures, được sử dụng với nhiều methods được định nghĩa trên `Result<T, E>`. Các methods này có thể ngắn gọn hơn so với sử dụng `match` khi xử lý các giá trị `Result<T, E>` trong mã của bạn.
>
> Ví dụ, đây là một cách khác để viết logic giống như được hiển thị trong Listing 9-5, lần này sử dụng closures và method `unwrap_or_else`:
>
> <!-- CAN’T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> Mặc dù mã này có cùng hành vi với Listing 9-5, nhưng nó không chứa bất kỳ biểu thức `match` nào và dễ đọc hơn. Hãy quay lại ví dụ này sau khi bạn đã đọc Chương 13 và tìm kiếm method `unwrap_or_else` trong tài liệu thư viện tiêu chuẩn. Rất nhiều methods khác có thể dọn dẹp các biểu thức `match` lồng nhau khổng lồ khi bạn đang xử lý các lỗi.

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### Phím Tắt cho Panic Khi Xảy Ra Lỗi

Sử dụng `match` hoạt động tốt, nhưng nó có thể hơi dài dòng và không phải lúc nào cũng truyền đạt ý định tốt. Kiểu `Result<T, E>` có nhiều helper methods được định nghĩa trên nó để thực hiện các tác vụ khác nhau, cụ thể hơn. Method `unwrap` là một shortcut method được triển khai giống như biểu thức `match` chúng ta viết trong Listing 9-4. Nếu giá trị `Result` là biến thể `Ok`, `unwrap` sẽ trả về giá trị bên trong `Ok`. Nếu `Result` là biến thể `Err`, `unwrap` sẽ gọi macro `panic!` cho chúng ta. Dưới đây là một ví dụ về `unwrap` trong hành động:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Nếu chúng ta chạy mã này mà không có file _hello.txt_, chúng ta sẽ thấy một thông báo lỗi từ lệnh gọi `panic!` mà method `unwrap` thực hiện:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread ‘main’ panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Tương tự, method `expect` cũng cho phép chúng ta chọn thông báo lỗi `panic!`. Sử dụng `expect` thay vì `unwrap` và cung cấp các thông báo lỗi tốt có thể truyền đạt ý định của bạn và làm cho việc theo dõi nguồn gốc của panic dễ dàng hơn. Cú pháp của `expect` trông như thế này:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

Chúng ta sử dụng `expect` theo cùng cách như `unwrap`: để trả về file handle hoặc gọi macro `panic!`. Thông báo lỗi được sử dụng bởi `expect` trong lệnh gọi của nó tới `panic!` sẽ là parameter mà chúng ta truyền vào `expect`, thay vì thông báo `panic!` mặc định mà `unwrap` sử dụng. Đây là những gì nó trông giống như:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread ‘main’ panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Trong mã chất lượng sản xuất, hầu hết các Rustaceans chọn `expect` thay vì `unwrap` và cung cấp thêm ngữ cảnh về lý do tại sao hoạt động được kỳ vọng là luôn thành công. Bằng cách đó, nếu các giả định của bạn bao giờ được chứng minh là sai, bạn có thêm thông tin để sử dụng trong gỡ lỗi.

### Lan Truyền Lỗi

Khi cài đặt của một function gọi một cái gì đó có thể thất bại, thay vì xử lý lỗi trong chính function đó, bạn có thể trả về lỗi cho mã gọi để nó có thể quyết định phải làm gì. Điều này được gọi là _lan truyền_ lỗi và cung cấp nhiều kiểm soát hơn cho mã gọi, nơi có thể có thêm thông tin hoặc logic quy định cách xử lý lỗi hơn là những gì bạn có sẵn trong ngữ cảnh của mã của bạn.

Ví dụ, Listing 9-6 hiển thị một function đọc tên người dùng từ một file. Nếu file không tồn tại hoặc không thể đọc được, function này sẽ trả về các lỗi đó cho mã gọi function.

<Listing number="9-6" file-name="src/main.rs" caption="Một function trả về các lỗi cho mã gọi bằng cách sử dụng `match`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don’t want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Function này có thể được viết theo cách ngắn hơn nhiều, nhưng chúng tôi sẽ bắt đầu bằng cách thực hiện rất nhiều thủ công để khám phá xử lý lỗi; ở cuối, chúng tôi sẽ hiển thị cách ngắn hơn. Hãy xem xét kiểu trả về của function trước tiên: `Result<String, io::Error>`. Điều này có nghĩa là function trả về một giá trị của kiểu `Result<T, E>`, nơi generic parameter `T` đã được điền với kiểu cụ thể `String` và kiểu generic `E` đã được điền với kiểu cụ thể `io::Error`.

Nếu function này thành công mà không gặp bất kỳ vấn đề nào, mã gọi function này sẽ nhận được giá trị `Ok` chứa một `String`—`username` mà function này đã đọc từ file. Nếu function này gặp bất kỳ vấn đề nào, mã gọi sẽ nhận được giá trị `Err` chứa một instance của `io::Error` chứa thêm thông tin về những vấn đề là gì. Chúng ta chọn `io::Error` làm kiểu trả về của function này vì đó chính là kiểu của giá trị lỗi được trả về từ cả hai hoạt động chúng ta đang gọi trong body của function này có thể thất bại: function `File::open` và method `read_to_string`.

Body của function bắt đầu bằng cách gọi function `File::open`. Sau đó, chúng ta xử lý giá trị `Result` bằng `match` tương tự như `match` trong Listing 9-4. Nếu `File::open` thành công, file handle trong biến pattern `file` trở thành giá trị trong biến mutable `username_file` và function tiếp tục. Trong trường hợp `Err`, thay vì gọi `panic!`, chúng ta sử dụng từ khóa `return` để trả về sớm khỏi function hoàn toàn và chuyển giá trị lỗi từ `File::open`, bây giờ trong biến pattern `e`, trở lại mã gọi như là giá trị lỗi của function này.

Vì vậy, nếu chúng ta có file handle trong `username_file`, function sẽ tạo một `String` mới trong biến `username` và gọi method `read_to_string` trên file handle trong `username_file` để đọc nội dung của file vào `username`. Method `read_to_string` cũng trả về một `Result` vì nó có thể thất bại, ngay cả khi `File::open` đã thành công. Vì vậy, chúng ta cần một `match` khác để xử lý `Result` đó: Nếu `read_to_string` thành công, thì function của chúng ta đã thành công, và chúng ta trả về tên người dùng từ file bây giờ trong `username` được bao bọc trong `Ok`. Nếu `read_to_string` thất bại, chúng ta trả về giá trị lỗi theo cách tương tự như chúng ta trả về giá trị lỗi trong `match` xử lý giá trị trả về của `File::open`. Tuy nhiên, chúng ta không cần phải nói `return` một cách rõ ràng, vì đây là biểu thức cuối cùng trong function.

Mã gọi mã này sẽ xử lý việc nhận giá trị `Ok` chứa tên người dùng hoặc giá trị `Err` chứa `io::Error`. Tùy thuộc vào mã gọi để quyết định phải làm gì với những giá trị đó. Nếu mã gọi nhận được giá trị `Err`, nó có thể gọi `panic!` và làm cho chương trình sập, sử dụng tên người dùng mặc định, hoặc tìm kiếm tên người dùng từ nơi khác ngoài file, ví dụ. Chúng ta không có đủ thông tin về những gì mã gọi thực sự đang cố gắng làm, vì vậy chúng ta lan truyền tất cả thông tin thành công hoặc lỗi hướng lên để nó xử lý một cách thích hợp.

Mẫu lan truyền lỗi này rất phổ biến trong Rust mà Rust cung cấp toán tử question mark `?` để giúp việc này dễ dàng hơn.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### Phím Tắt Toán Tử `?`

Listing 9-7 hiển thị một cài đặt của `read_username_from_file` có cùng chức năng như trong Listing 9-6, nhưng cài đặt này sử dụng toán tử `?`.

<Listing number="9-7" file-name="src/main.rs" caption="Một function trả về các lỗi cho mã gọi bằng cách sử dụng toán tử `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

`?` được đặt sau giá trị `Result` được định nghĩa để hoạt động theo cách gần như giống như các biểu thức `match` mà chúng ta đã định nghĩa để xử lý các giá trị `Result` trong Listing 9-6. Nếu giá trị của `Result` là `Ok`, giá trị bên trong `Ok` sẽ được trả về từ biểu thức này, và chương trình sẽ tiếp tục. Nếu giá trị là `Err`, `Err` sẽ được trả về từ toàn bộ function như thể chúng ta đã sử dụng từ khóa `return` để giá trị lỗi được lan truyền đến mã gọi.

Có một sự khác biệt giữa những gì biểu thức `match` từ Listing 9-6 làm và những gì toán tử `?` làm: các giá trị lỗi mà toán tử `?` được gọi trên chúng đi qua function `from`, được định nghĩa trong trait `From` trong thư viện tiêu chuẩn, được sử dụng để chuyển đổi các giá trị từ một kiểu sang kiểu khác. Khi toán tử `?` gọi function `from`, kiểu lỗi nhận được được chuyển đổi thành kiểu lỗi được định nghĩa trong kiểu trả về của function hiện tại. Điều này hữu ích khi một function trả về một kiểu lỗi để đại diện cho tất cả những cách mà một function có thể thất bại, ngay cả khi các phần có thể thất bại vì nhiều lý do khác nhau.

Ví dụ, chúng ta có thể thay đổi function `read_username_from_file` trong Listing 9-7 để trả về một custom error type có tên `OurError` mà chúng ta định nghĩa. Nếu chúng ta cũng định nghĩa `impl From<io::Error> for OurError` để xây dựng một instance của `OurError` từ `io::Error`, thì các lệnh gọi toán tử `?` trong body của `read_username_from_file` sẽ gọi `from` và chuyển đổi các kiểu lỗi mà không cần thêm bất kỳ mã nào vào function.

Trong bối cảnh của Listing 9-7, `?` ở cuối lệnh gọi `File::open` sẽ trả về giá trị bên trong `Ok` cho biến `username_file`. Nếu có lỗi xảy ra, toán tử `?` sẽ trả về sớm khỏi toàn bộ function và cung cấp bất kỳ giá trị `Err` nào cho mã gọi. Điều tương tự cũng được áp dụng cho `?` ở cuối lệnh gọi `read_to_string`.

Toán tử `?` loại bỏ rất nhiều boilerplate và làm cho cài đặt function này đơn giản hơn. Chúng ta thậm chí có thể rút ngắn mã này hơn nữa bằng cách chaining các method calls ngay sau `?`, như được hiển thị trong Listing 9-8.

<Listing number="9-8" file-name="src/main.rs" caption="Chaining các method calls sau toán tử `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don’t want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

Chúng ta đã di chuyển việc tạo `String` mới trong `username` đến phần đầu của function; phần đó chưa thay đổi. Thay vì tạo biến `username_file`, chúng ta đã chaining lệnh gọi `read_to_string` trực tiếp vào kết quả của `File::open("hello.txt")?`. Chúng ta vẫn còn `?` ở cuối lệnh gọi `read_to_string`, và chúng ta vẫn trả về giá trị `Ok` chứa `username` khi cả `File::open` và `read_to_string` đều thành công thay vì trả về các lỗi. Chức năng lại giống như trong Listing 9-6 và Listing 9-7; đây chỉ là một cách khác, ergonomic hơn để viết nó.

Listing 9-9 hiển thị một cách để làm cho nó thậm chí ngắn hơn bằng cách sử dụng `fs::read_to_string`.

<Listing number="9-9" file-name="src/main.rs" caption="Sử dụng `fs::read_to_string` thay vì mở và sau đó đọc file">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don’t want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Đọc một file vào một chuỗi là một hoạt động khá phổ biến, vì vậy thư viện tiêu chuẩn cung cấp function tiện lợi `fs::read_to_string` mở file, tạo `String` mới, đọc nội dung file, đặt nội dung vào `String` đó, và trả về nó. Tất nhiên, sử dụng `fs::read_to_string` không cho chúng ta cơ hội giải thích tất cả việc xử lý lỗi, vì vậy chúng ta đã làm nó theo cách dài hơn trước tiên.

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### Nơi Sử Dụng Toán Tử `?`

Toán tử `?` chỉ có thể được sử dụng trong các function có kiểu trả về tương thích với giá trị mà `?` được sử dụng trên. Điều này là vì toán tử `?` được định nghĩa để thực hiện sự trả về sớm của một giá trị khỏi function, theo cách tương tự như biểu thức `match` chúng ta đã định nghĩa trong Listing 9-6. Trong Listing 9-6, `match` sử dụng giá trị `Result`, và arm trả về sớm trả về giá trị `Err(e)`. Kiểu trả về của function phải là `Result` để nó tương thích với `return` này.

Trong Listing 9-10, hãy xem xét lỗi chúng ta sẽ nhận được nếu chúng ta sử dụng toán tử `?` trong function `main` có kiểu trả về không tương thích với kiểu của giá trị chúng ta sử dụng `?` trên.

<Listing number="9-10" file-name="src/main.rs" caption="Cố gắng sử dụng `?` trong function `main` trả về `()` sẽ không biên dịch được.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Mã này mở một file, có thể thất bại. Toán tử `?` theo sau giá trị `Result` được trả về bởi `File::open`, nhưng function `main` này có kiểu trả về `()`, không phải `Result`. Khi chúng ta biên dịch mã này, chúng ta nhận được thông báo lỗi sau:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Lỗi này chỉ ra rằng chúng ta chỉ được phép sử dụng toán tử `?` trong một function trả về `Result`, `Option`, hoặc một kiểu khác triển khai `FromResidual`.

Để sửa lỗi, bạn có hai lựa chọn. Một lựa chọn là thay đổi kiểu trả về của function của bạn để tương thích với giá trị bạn đang sử dụng toán tử `?` trên miễn là bạn không có những hạn chế nào ngăn cản điều đó. Lựa chọn khác là sử dụng `match` hoặc một trong các methods của `Result<T, E>` để xử lý `Result<T, E>` theo bất kỳ cách nào phù hợp.

Thông báo lỗi cũng đề cập rằng `?` cũng có thể được sử dụng với các giá trị `Option<T>`. Cũng giống như sử dụng `?` trên `Result`, bạn chỉ có thể sử dụng `?` trên `Option` trong một function trả về `Option`. Hành vi của toán tử `?` khi được gọi trên `Option<T>` tương tự như hành vi của nó khi được gọi trên `Result<T, E>`: Nếu giá trị là `None`, `None` sẽ được trả về sớm từ function tại điểm đó. Nếu giá trị là `Some`, giá trị bên trong `Some` là giá trị kết quả của biểu thức, và function tiếp tục. Listing 9-11 có một ví dụ về một function tìm ký tự cuối cùng của dòng đầu tiên trong văn bản cho trước.

<Listing number="9-11" caption="Sử dụng toán tử `?` trên giá trị `Option<T>`">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Function này trả về `Option<char>` vì có thể có một ký tự ở đó, nhưng cũng có thể là không có. Mã này lấy argument string slice `text` và gọi method `lines` trên nó, trả về một iterator trên các dòng trong chuỗi. Vì function này muốn kiểm tra dòng đầu tiên, nó gọi `next` trên iterator để lấy giá trị đầu tiên từ iterator. Nếu `text` là chuỗi rỗng, lệnh gọi `next` này sẽ trả về `None`, trong trường hợp đó chúng ta sử dụng `?` để dừng lại và trả về `None` từ `last_char_of_first_line`. Nếu `text` không phải là chuỗi rỗng, `next` sẽ trả về giá trị `Some` chứa một string slice của dòng đầu tiên trong `text`.

`?` trích xuất string slice, và chúng ta có thể gọi `chars` trên string slice đó để lấy một iterator của các ký tự của nó. Chúng ta quan tâm đến ký tự cuối cùng trong dòng đầu tiên này, vì vậy chúng ta gọi `last` để trả về item cuối cùng trong iterator. Đây là `Option` vì có thể dòng đầu tiên là chuỗi rỗng; ví dụ, nếu `text` bắt đầu với dòng trống nhưng có các ký tự trên các dòng khác, như trong `"\nhi"`. Tuy nhiên, nếu có ký tự cuối cùng trên dòng đầu tiên, nó sẽ được trả về trong biến thể `Some`. Toán tử `?` ở giữa cung cấp cho chúng ta một cách ngắn gọn để diễn đạt logic này, cho phép chúng ta triển khai function trong một dòng. Nếu chúng ta không thể sử dụng toán tử `?` trên `Option`, chúng ta sẽ phải triển khai logic này bằng các lệnh gọi method hoặc biểu thức `match` nhiều hơn.

Lưu ý rằng bạn có thể sử dụng toán tử `?` trên `Result` trong một function trả về `Result`, và bạn có thể sử dụng toán tử `?` trên `Option` trong một function trả về `Option`, nhưng bạn không thể trộn lẫn. Toán tử `?` sẽ không tự động chuyển đổi `Result` thành `Option` hoặc ngược lại; trong những trường hợp đó, bạn có thể sử dụng các methods như method `ok` trên `Result` hoặc method `ok_or` trên `Option` để thực hiện việc chuyển đổi một cách rõ ràng.

Cho đến nay, tất cả các function `main` chúng ta sử dụng đều trả về `()`. Function `main` là đặc biệt vì nó là điểm vào và điểm thoát của một chương trình có thể thực thi được, và có những hạn chế đối với kiểu trả về của nó để chương trình hoạt động như mong đợi.

May mắn thay, `main` cũng có thể trả về `Result<(), E>`. Listing 9-12 có mã từ Listing 9-10, nhưng chúng tôi đã thay đổi kiểu trả về của `main` thành `Result<(), Box<dyn Error>>` và thêm giá trị trả về `Ok(())` ở cuối. Mã này bây giờ sẽ biên dịch được.

<Listing number=”9-12” file-name=”src/main.rs” caption=”Thay đổi `main` để trả về `Result<(), E>` cho phép sử dụng toán tử `?` trên các giá trị `Result`.”>

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

Kiểu `Box<dyn Error>` là một trait object, mà chúng ta sẽ nói về nó trong [“Sử dụng Trait Objects để Trừu Tượng Hóa Hành Vi Chung”][trait-objects]<!-- ignore --> trong Chương 18. Hiện tại, bạn có thể đọc `Box<dyn Error>` có nghĩa là “bất kỳ loại lỗi nào.” Sử dụng `?` trên giá trị `Result` trong function `main` với kiểu lỗi `Box<dyn Error>` là được cho phép vì nó cho phép bất kỳ giá trị `Err` nào được trả về sớm. Mặc dù body của function `main` này sẽ chỉ bao giờ trả về các lỗi của kiểu `std::io::Error`, bằng cách chỉ định `Box<dyn Error>`, signature này sẽ tiếp tục đúng ngay cả khi thêm nhiều mã trả về các lỗi khác vào body của `main`.

Khi function `main` trả về `Result<(), E>`, executable sẽ thoát với giá trị `0` nếu `main` trả về `Ok(())` và sẽ thoát với giá trị khác 0 nếu `main` trả về giá trị `Err`. Các executable được viết bằng C trả về các số nguyên khi chúng thoát: các chương trình thoát thành công trả về số nguyên `0`, và các chương trình có lỗi trả về một số nguyên khác `0`. Rust cũng trả về các số nguyên từ các executable để tương thích với quy ước này.

Function `main` có thể trả về bất kỳ kiểu nào triển khai [trait `std::process::Termination`][termination]<!-- ignore -->, chứa function `report` trả về `ExitCode`. Hãy tham khảo tài liệu thư viện tiêu chuẩn để biết thêm thông tin về việc triển khai trait `Termination` cho các kiểu của chính bạn.

Bây giờ chúng ta đã thảo luận chi tiết về việc gọi `panic!` hoặc trả về `Result`, hãy quay lại chủ đề về cách quyết định cái nào là thích hợp để sử dụng trong những trường hợp nào.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html
