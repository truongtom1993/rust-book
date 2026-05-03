## Lỗi Có Thể Phục Hồi với `Result`

Hầu hết các lỗi không nghiêm trọng đến mức cần phải dừng chương trình hoàn toàn.
Đôi khi khi một function thất bại, đó là vì một lý do mà bạn có thể dễ dàng diễn giải
và phản ứng. Ví dụ, nếu bạn cố gắng mở một file và thao tác đó thất bại
vì file không tồn tại, bạn có thể muốn tạo file thay vì
chấm dứt quá trình.

Hãy nhớ lại từ [“Handling Potential Failure with `Result`”][handle_failure]<!--
ignore --> trong Chapter 2 rằng enum `Result` được định nghĩa là có hai
variants, `Ok` và `Err`, như sau:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` và `E` là generic type parameters: Chúng ta sẽ thảo luận về generics chi tiết hơn
trong Chapter 10. Điều bạn cần biết ngay bây giờ là `T` đại diện cho
kiểu của giá trị sẽ được trả về trong trường hợp thành công trong variant `Ok`,
và `E` đại diện cho kiểu của lỗi sẽ được trả về trong trường hợp thất bại
trong variant `Err`. Vì `Result` có các generic type parameters này,
chúng ta có thể sử dụng kiểu `Result` và các function được định nghĩa trên nó
trong nhiều tình huống khác nhau nơi giá trị thành công và giá trị lỗi mà chúng ta muốn
trả về có thể khác nhau.

Hãy gọi một function trả về một giá trị `Result` vì function có thể
thất bại. Ở Listing 9-3, chúng ta cố gắng mở một file.

<Listing number="9-3" file-name="src/main.rs" caption="Mở một file">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

Kiểu return của `File::open` là `Result<T, E>`. Generic parameter `T`
đã được điền bởi implementation của `File::open` với kiểu của
giá trị thành công, `std::fs::File`, đó là một file handle. Kiểu của `E` được sử dụng
trong giá trị lỗi là `std::io::Error`. Kiểu return này có nghĩa là lệnh gọi
`File::open` có thể thành công và trả về một file handle mà chúng ta có thể đọc
hoặc ghi. Lệnh gọi function cũng có thể thất bại: Ví dụ, file có thể không
tồn tại, hoặc chúng ta có thể không có quyền truy cập file. Function
`File::open` cần phải có một cách để cho chúng ta biết liệu nó thành công hay thất bại
và cùng lúc đó cho chúng ta hoặc là file handle hoặc thông tin lỗi. Cái
thông tin này chính xác là những gì enum `Result` truyền tải.

Trong trường hợp `File::open` thành công, giá trị trong biến
`greeting_file_result` sẽ là một instance của `Ok` chứa một file handle.
Trong trường hợp nó thất bại, giá trị trong `greeting_file_result` sẽ là
một instance của `Err` chứa thêm thông tin về loại lỗi đã
xảy ra.

Chúng ta cần thêm code vào Listing 9-3 để thực hiện các hành động khác nhau tùy thuộc
vào giá trị mà `File::open` trả về. Listing 9-4 hiển thị một cách để xử lý
`Result` sử dụng một công cụ cơ bản, expression `match` mà chúng ta đã thảo luận ở
Chapter 6.

<Listing number="9-4" file-name="src/main.rs" caption="Sử dụng expression `match` để xử lý các variant `Result` có thể được trả về">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Lưu ý rằng, giống như enum `Option`, enum `Result` và các variants của nó đã được
đưa vào scope bởi prelude, vì vậy chúng ta không cần phải chỉ định `Result::`
trước các variants `Ok` và `Err` trong các arms của `match`.

Khi result là `Ok`, code này sẽ trả về giá trị `file` bên trong
variant `Ok`, và chúng ta sau đó gán giá trị file handle đó cho biến
`greeting_file`. Sau `match`, chúng ta có thể sử dụng file handle để đọc hoặc
ghi.

Arm khác của `match` xử lý trường hợp chúng ta nhận được một giá trị `Err` từ
`File::open`. Trong ví dụ này, chúng ta đã chọn gọi macro `panic!`. Nếu
không có file tên là _hello.txt_ trong thư mục hiện tại của chúng ta và chúng ta chạy
code này, chúng ta sẽ thấy output sau từ macro `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Như thường lệ, output này cho chúng ta biết chính xác những gì đã sai.

### Khớp với Các Lỗi Khác Nhau

Code ở Listing 9-4 sẽ `panic!` bất kể lý do tại sao `File::open` thất bại.
Tuy nhiên, chúng ta muốn thực hiện các hành động khác nhau cho các lý do thất bại khác nhau. Nếu
`File::open` thất bại vì file không tồn tại, chúng ta muốn tạo file
và trả về handle cho file mới. Nếu `File::open` thất bại vì bất kỳ lý do nào khác—ví dụ, vì chúng ta không có quyền mở file—chúng ta vẫn
muốn code `panic!` theo cách giống như ở Listing 9-4. Để làm điều này, chúng ta
thêm một expression `match` bên trong, được hiển thị ở Listing 9-5.

<Listing number="9-5" file-name="src/main.rs" caption="Xử lý các loại lỗi khác nhau theo các cách khác nhau">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

Kiểu của giá trị mà `File::open` trả về bên trong variant `Err` là
`io::Error`, đó là một struct được cung cấp bởi thư viện chuẩn. Struct này
có một method, `kind`, mà chúng ta có thể gọi để lấy một giá trị `io::ErrorKind`. Enum
`io::ErrorKind` được cung cấp bởi thư viện chuẩn và có các variants
đại diện cho các loại lỗi khác nhau có thể xảy ra từ một thao tác `io`.
Variant mà chúng ta muốn sử dụng là `ErrorKind::NotFound`, chỉ ra rằng
file mà chúng ta đang cố gắng mở không tồn tại. Vì vậy, chúng ta khớp trên
`greeting_file_result`, nhưng chúng ta cũng có một match bên trong trên `error.kind()`.

Điều kiện chúng ta muốn kiểm tra trong match bên trong là liệu giá trị được trả về
bởi `error.kind()` có phải là variant `NotFound` của enum `ErrorKind`. Nếu có,
chúng ta cố gắng tạo file với `File::create`. Tuy nhiên, vì `File::create`
cũng có thể thất bại, chúng ta cần một arm thứ hai trong expression `match` bên trong.
Khi file không thể được tạo, một thông báo lỗi khác được in. Arm thứ hai của
`match` bên ngoài vẫn giữ nguyên, vì vậy chương trình panic trên bất kỳ lỗi nào ngoài
lỗi file bị thiếu.

> #### Các Lựa Chọn Thay Thế để Sử Dụng `match` với `Result<T, E>`
>
> Đó là rất nhiều `match`! Expression `match` rất hữu ích nhưng cũng rất
> là nguyên thủy. Ở Chapter 13, bạn sẽ học về closures, được sử dụng
> với nhiều methods được định nghĩa trên `Result<T, E>`. Những methods này có thể
> ngắn gọn hơn khi sử dụng `match` khi xử lý các giá trị `Result<T, E>` trong code của bạn.
>
> Ví dụ, đây là cách khác để viết logic tương tự như được hiển thị ở Listing
> 9-5, lần này sử dụng closures và method `unwrap_or_else`:
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
> Mặc dù code này có cùng hành vi như Listing 9-5, nó không chứa
> bất kỳ expression `match` nào và dễ đọc hơn. Quay lại ví dụ này
> sau khi bạn đã đọc Chapter 13 và tìm method `unwrap_or_else` trong
> documentation của thư viện chuẩn. Nhiều methods hơn nữa có thể dọn dẹp
> những expression `match` lồng nhau khổng lồ khi bạn đang xử lý lỗi.

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### Phím Tắt cho Panic on Error

Sử dụng `match` hoạt động đủ tốt, nhưng nó có thể hơi dài dòng và không phải lúc nào
cũng truyền đạt ý định tốt. Kiểu `Result<T, E>` có nhiều helper methods
được định nghĩa trên nó để thực hiện các tác vụ khác nhau, cụ thể hơn. Method `unwrap`
là một method shortcut được triển khai giống như expression `match` chúng ta đã viết ở
Listing 9-4. Nếu giá trị `Result` là variant `Ok`, `unwrap` sẽ trả về
giá trị bên trong `Ok`. Nếu `Result` là variant `Err`, `unwrap` sẽ
gọi macro `panic!` cho chúng ta. Đây là một ví dụ về `unwrap` hoạt động:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Nếu chúng ta chạy code này mà không có file _hello.txt_, chúng ta sẽ thấy một thông báo lỗi
từ lệnh gọi `panic!` mà method `unwrap` thực hiện:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread ‘main’ panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Tương tự như vậy, method `expect` cũng cho phép chúng ta chọn thông báo lỗi `panic!`.
Sử dụng `expect` thay vì `unwrap` và cung cấp các thông báo lỗi tốt có thể truyền đạt
ý định của bạn và làm cho việc theo dõi nguồn của panic dễ dàng hơn. Cú pháp của
`expect` trông như thế này:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

Chúng ta sử dụng `expect` theo cách tương tự như `unwrap`: để trả về file handle
hoặc gọi macro `panic!`. Thông báo lỗi được sử dụng bởi `expect` trong lệnh gọi `panic!`
sẽ là tham số mà chúng ta truyền đến `expect`, thay vì thông báo
mặc định `panic!` mà `unwrap` sử dụng. Đây là cách nó trông như:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread ‘main’ panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Trong code chất lượng production, hầu hết các Rustaceans chọn `expect` thay vì
`unwrap` và cung cấp thêm bối cảnh về lý do tại sao thao tác được dự kiến
sẽ luôn thành công. Theo cách đó, nếu các giả định của bạn bao giờ được chứng minh là sai,
bạn sẽ có thêm thông tin để sử dụng trong debugging.

### Lan Truyền Lỗi

Khi implementation của một function gọi một cái gì đó có thể thất bại, thay vì
xử lý lỗi trong function itself, bạn có thể trả về lỗi cho calling code
để nó có thể quyết định phải làm gì. Đây được gọi là _lan truyền_
lỗi và cung cấp thêm kiểm soát cho calling code, nơi có thể có thêm
thông tin hoặc logic quy định cách lỗi nên được xử lý hơn là những gì
bạn có sẵn trong bối cảnh code của bạn.

Ví dụ, Listing 9-6 hiển thị một function đọc một username từ một file. Nếu
file không tồn tại hoặc không thể được đọc, function này sẽ trả về những lỗi đó
cho code đã gọi function.

<Listing number="9-6" file-name="src/main.rs" caption="A function that returns errors to the calling code using `match`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Function này có thể được viết theo cách ngắn hơn, nhưng chúng ta sẽ bắt đầu bằng cách
thực hiện rất nhiều thao tác thủ công để khám phá error handling; cuối cùng,
chúng ta sẽ hiển thị cách ngắn hơn. Hãy xem xét kiểu return của function
trước tiên: `Result<String, io::Error>`. Điều này có nghĩa là function
đang trả về một giá trị của kiểu `Result<T, E>`, nơi generic parameter `T` đã được
điền vào với kiểu cụ thể `String` và generic type `E` đã được
điền vào với kiểu cụ thể `io::Error`.

Nếu function này thành công mà không gặp vấn đề gì, code gọi function này
sẽ nhận được một giá trị `Ok` chứa một `String`—`username` mà
function này đã đọc từ file. Nếu function này gặp vấn đề gì, calling
code sẽ nhận được một giá trị `Err` chứa một instance của `io::Error`
chứa thêm thông tin về những vấn đề là gì. Chúng ta chọn
`io::Error` là kiểu return của function này vì đó là
kiểu của giá trị lỗi được trả về từ cả hai thao tác mà chúng ta đang gọi trong
body của function này có thể thất bại: function `File::open` và
method `read_to_string`.

Body của function bắt đầu bằng cách gọi function `File::open`. Sau đó, chúng ta
xử lý giá trị `Result` với một `match` tương tự như `match` ở Listing 9-4.
Nếu `File::open` thành công, file handle trong pattern variable `file`
trở thành giá trị trong mutable variable `username_file` và function
tiếp tục. Trong trường hợp `Err`, thay vì gọi `panic!`, chúng ta sử dụng
keyword `return` để return sớm ra khỏi function hoàn toàn và truyền giá trị lỗi
từ `File::open`, bây giờ trong pattern variable `e`, quay trở lại calling code
như giá trị lỗi của function này.

Vì vậy, nếu chúng ta có một file handle trong `username_file`, function sau đó tạo một
`String` mới trong variable `username` và gọi method `read_to_string` trên
file handle trong `username_file` để đọc nội dung của file vào
`username`. Method `read_to_string` cũng trả về một `Result` vì nó
có thể thất bại, ngay cả khi `File::open` thành công. Vì vậy, chúng ta cần một `match`
khác để xử lý `Result` đó: Nếu `read_to_string` thành công, thì function
của chúng ta đã thành công, và chúng ta trả về username từ file hiện tại ở trong
`username` được bao bọc trong `Ok`. Nếu `read_to_string` thất bại, chúng ta trả về
giá trị lỗi theo cách tương tự mà chúng ta đã trả về giá trị lỗi trong `match`
xử lý giá trị return của `File::open`. Tuy nhiên, chúng ta không cần
phải nói rõ ràng `return`, vì đây là expression cuối cùng trong function.

Code gọi code này sau đó sẽ xử lý nhận được một giá trị `Ok`
chứa một username hoặc một giá trị `Err` chứa một `io::Error`. Phụ thuộc vào
calling code để quyết định phải làm gì với những giá trị đó. Nếu calling
code nhận được một giá trị `Err`, nó có thể gọi `panic!` và crash chương trình, sử dụng
một username mặc định, hoặc tìm username từ một nơi khác không phải file, ví dụ.
Chúng ta không có đủ thông tin về những gì mà calling code thực sự
đang cố gắng làm, vì vậy chúng ta lan truyền tất cả thông tin thành công hoặc lỗi
lên trên để nó xử lý một cách thích hợp.

Pattern của việc lan truyền lỗi này rất phổ biến trong Rust mà Rust cung cấp
operator dấu hỏi `?` để làm điều này dễ dàng hơn.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### Phím Tắt Operator `?`

Listing 9-7 hiển thị một implementation của `read_username_from_file` có
cùng chức năng như ở Listing 9-6, nhưng implementation này sử dụng
operator `?`.

<Listing number="9-7" file-name="src/main.rs" caption="A function that returns errors to the calling code using the `?` operator">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

`?` được đặt sau một giá trị `Result` được định nghĩa để hoạt động theo cách gần
như giống hệt như các expression `match` mà chúng ta đã định nghĩa để xử lý
các giá trị `Result` ở Listing 9-6. Nếu giá trị của `Result` là `Ok`,
giá trị bên trong `Ok` sẽ được trả về từ expression này, và chương trình sẽ tiếp tục.
Nếu giá trị là `Err`, `Err` sẽ được trả về từ toàn bộ function như thể
chúng ta đã sử dụng keyword `return` để giá trị lỗi được lan truyền
đến calling code.

Có một sự khác biệt giữa những gì mà expression `match` từ Listing 9-6 làm
và những gì mà operator `?` làm: Các giá trị lỗi mà operator `?` được gọi
trên chúng sẽ đi qua function `from`, được định nghĩa trong trait `From`
trong thư viện chuẩn, được sử dụng để chuyển đổi giá trị từ kiểu này sang kiểu khác.
Khi operator `?` gọi function `from`, kiểu lỗi nhận được
được chuyển đổi thành kiểu lỗi được định nghĩa trong kiểu return của function
hiện tại. Điều này hữu ích khi một function trả về một kiểu lỗi để đại diện
cho tất cả các cách một function có thể thất bại, ngay cả khi một số phần có thể
thất bại vì nhiều lý do khác nhau.

Ví dụ, chúng ta có thể thay đổi function `read_username_from_file` ở Listing
9-7 để trả về một kiểu lỗi tùy chỉnh được đặt tên là `OurError` mà chúng ta định nghĩa.
Nếu chúng ta cũng định nghĩa `impl From<io::Error> for OurError` để tạo một instance
của `OurError` từ một `io::Error`, thì các lệnh gọi operator `?` trong body của
`read_username_from_file` sẽ gọi `from` và chuyển đổi các kiểu lỗi mà không cần
phải thêm bất kỳ code nào khác vào function.

Trong bối cảnh của Listing 9-7, `?` ở cuối lệnh gọi `File::open`
sẽ trả về giá trị bên trong `Ok` cho variable `username_file`. Nếu xảy ra lỗi,
operator `?` sẽ return sớm ra khỏi toàn bộ function và cho bất kỳ
giá trị `Err` nào tới calling code. Điều tương tự áp dụng cho `?` ở
cuối lệnh gọi `read_to_string`.

Operator `?` loại bỏ rất nhiều boilerplate và làm cho implementation function này
đơn giản hơn. Chúng ta thậm chí có thể rút ngắn code này hơn nữa bằng cách chuỗi
các method calls ngay sau `?`, như được hiển thị ở Listing 9-8.

<Listing number="9-8" file-name="src/main.rs" caption="Chaining method calls after the `?` operator">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

Chúng ta đã chuyển việc tạo `String` mới trong `username` đến đầu
function; phần đó chưa thay đổi. Thay vì tạo một variable
`username_file`, chúng ta đã chuỗi lệnh gọi `read_to_string` trực tiếp vào
kết quả của `File::open("hello.txt")?`. Chúng ta vẫn có một `?` ở cuối
lệnh gọi `read_to_string`, và chúng ta vẫn trả về một giá trị `Ok` chứa
`username` khi cả `File::open` và `read_to_string` thành công thay vì
trả về lỗi. Chức năng lại giống như ở Listing 9-6 và Listing 9-7;
đây chỉ là một cách viết khác, ergonomic hơn.

Listing 9-9 hiển thị một cách để làm cái này thậm chí còn ngắn hơn sử dụng
`fs::read_to_string`.

<Listing number="9-9" file-name="src/main.rs" caption="Sử dụng `fs::read_to_string` thay vì mở và sau đó đọc file">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don’t want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Đọc một file vào một string là một thao tác khá phổ biến, vì vậy thư viện
chuẩn cung cấp function `fs::read_to_string` thuận tiện mở file,
tạo một `String` mới, đọc nội dung của file, đặt nội dung
vào `String` đó, và trả về nó. Tất nhiên, sử dụng `fs::read_to_string`
không cho chúng ta cơ hội để giải thích tất cả error handling, vì vậy
chúng ta đã làm nó theo cách dài hơn trước tiên.

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### Nơi Sử Dụng Operator `?`

Operator `?` chỉ có thể được sử dụng trong các function có kiểu return
tương thích với giá trị mà `?` được sử dụng trên. Điều này là vì operator `?`
được định nghĩa để thực hiện một early return của một giá trị ra khỏi function,
theo cách tương tự như expression `match` mà chúng ta đã định nghĩa ở Listing 9-6.
Ở Listing 9-6, `match` đã sử dụng một giá trị `Result`, và arm early return
trả về một giá trị `Err(e)`. Kiểu return của function phải là `Result`
để nó tương thích với `return` này.

Ở Listing 9-10, hãy xem xét lỗi mà chúng ta sẽ nhận được nếu chúng ta sử dụng
operator `?` trong function `main` với kiểu return không tương thích
với kiểu của giá trị mà chúng ta sử dụng `?` trên.

<Listing number="9-10" file-name="src/main.rs" caption="Cố gắng sử dụng `?` trong function `main` trả về `()` sẽ không biên dịch.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Code này mở một file, có thể thất bại. Operator `?` theo sau giá trị `Result`
được trả về bởi `File::open`, nhưng function `main` này có kiểu return
là `()`, không phải `Result`. Khi chúng ta biên dịch code này, chúng ta
nhận được thông báo lỗi sau:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Lỗi này chỉ ra rằng chúng ta chỉ được phép sử dụng operator `?` trong một
function trả về `Result`, `Option`, hoặc một kiểu khác
triển khai `FromResidual`.

Để sửa lỗi, bạn có hai lựa chọn. Một lựa chọn là thay đổi kiểu return
của function của bạn để tương thích với giá trị bạn đang sử dụng operator `?`
trên miễn là bạn không có hạn chế nào ngăn chặn điều đó. Lựa chọn khác
là sử dụng một `match` hoặc một trong các methods của `Result<T, E>`
để xử lý `Result<T, E>` theo bất kỳ cách nào thích hợp.

Thông báo lỗi cũng đề cập rằng `?` có thể được sử dụng với các giá trị
`Option<T>` cũng. Cũng giống như sử dụng `?` trên `Result`, bạn chỉ
có thể sử dụng `?` trên `Option` trong một function trả về `Option`. Hành vi
của operator `?` khi được gọi trên `Option<T>` tương tự như hành vi của nó
khi được gọi trên `Result<T, E>`: Nếu giá trị là `None`, `None` sẽ
được trả về sớm từ function tại điểm đó. Nếu giá trị là `Some`,
giá trị bên trong `Some` là giá trị kết quả của expression, và function tiếp tục.
Listing 9-11 có một ví dụ về một function tìm ký tự cuối cùng của dòng
đầu tiên trong text được cung cấp.

<Listing number="9-11" caption="Sử dụng operator `?` trên một giá trị `Option<T>`">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Function này trả về `Option<char>` vì nó có thể có một
ký tự ở đó, nhưng nó cũng có thể không. Code này lấy argument
string slice `text` và gọi method `lines` trên nó, trả về
một iterator trên các dòng trong string. Vì function này muốn
xem xét dòng đầu tiên, nó gọi `next` trên iterator để lấy giá trị đầu tiên
từ iterator. Nếu `text` là string trống, lệnh gọi `next` này
sẽ trả về `None`, trong đó chúng ta sử dụng `?` để dừng và trả về `None`
từ `last_char_of_first_line`. Nếu `text` không phải là string trống,
`next` sẽ trả về một giá trị `Some` chứa một string slice của dòng đầu tiên
trong `text`.

`?` trích xuất string slice, và chúng ta có thể gọi `chars` trên string slice
đó để lấy một iterator các ký tự của nó. Chúng ta quan tâm đến ký tự cuối cùng
trong dòng đầu tiên này, vì vậy chúng ta gọi `last` để trả về item cuối cùng
trong iterator. Đây là `Option` vì có thể dòng đầu tiên là
string trống; ví dụ, nếu `text` bắt đầu bằng một dòng trống nhưng có ký tự trên
các dòng khác, như `"\nhi"`. Tuy nhiên, nếu có một ký tự cuối cùng trên dòng đầu tiên,
nó sẽ được trả về trong variant `Some`. Operator `?` ở giữa
cho chúng ta một cách ngắn gọn để biểu diễn logic này, cho phép chúng ta triển khai
function trong một dòng. Nếu chúng ta không thể sử dụng operator `?` trên
`Option`, chúng ta phải triển khai logic này sử dụng nhiều lệnh gọi method hơn
hoặc một expression `match`.

Lưu ý rằng bạn có thể sử dụng operator `?` trên một `Result` trong một
function trả về `Result`, và bạn có thể sử dụng operator `?` trên một `Option`
trong một function trả về `Option`, nhưng bạn không thể trộn lẫn chúng.
Operator `?` sẽ không tự động chuyển đổi `Result` thành `Option` hoặc ngược lại;
trong những trường hợp đó, bạn có thể sử dụng các methods như method `ok` trên
`Result` hoặc method `ok_or` trên `Option` để thực hiện chuyển đổi một cách rõ ràng.

Cho đến bây giờ, tất cả các function `main` mà chúng ta đã sử dụng trả về `()`. Function
`main` là đặc biệt vì nó là điểm vào và điểm thoát của một chương trình
executable, và có những hạn chế về những gì kiểu return của nó có thể
để chương trình hoạt động như mong đợi.

May mắn thay, `main` cũng có thể trả về `Result<(), E>`. Listing 9-12 có
code từ Listing 9-10, nhưng chúng ta đã thay đổi kiểu return của `main`
thành `Result<(), Box<dyn Error>>` và thêm một giá trị return `Ok(())` ở cuối.
Code này sẽ biên dịch được ngay bây giờ.

<Listing number=”9-12” file-name=”src/main.rs” caption=”Thay đổi `main` để trả về `Result<(), E>` cho phép sử dụng operator `?` trên các giá trị `Result`.”>

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

Kiểu `Box<dyn Error>` là một trait object, mà chúng ta sẽ nói về trong [“Using
Trait Objects to Abstract over Shared Behavior”][trait-objects]<!-- ignore -->
ở Chapter 18. Bây giờ, bạn có thể đọc `Box<dyn Error>` có nghĩa là “bất kỳ
loại lỗi nào.” Sử dụng `?` trên một giá trị `Result` trong function `main`
với kiểu lỗi `Box<dyn Error>` được phép vì nó cho phép bất kỳ giá trị `Err`
nào được trả về sớm. Mặc dù body của function `main` này chỉ sẽ trả về
các lỗi của kiểu `std::io::Error`, bằng cách chỉ định `Box<dyn Error>`,
signature này sẽ tiếp tục được chính xác ngay cả khi thêm code trả về
các lỗi khác vào body của `main`.

Khi function `main` trả về `Result<(), E>`, executable sẽ thoát với
một giá trị `0` nếu `main` trả về `Ok(())` và sẽ thoát với một giá trị
khác không nếu `main` trả về một giá trị `Err`. Executables được viết bằng C
trả về integers khi chúng thoát: Các chương trình thoát thành công
trả về integer `0`, và các chương trình lỗi trả về một integer khác `0`.
Rust cũng trả về integers từ executables để tương thích với quy ước này.

Function `main` có thể trả về bất kỳ kiểu nào triển khai [trait
`std::process::Termination`][termination]<!-- ignore -->, chứa
một function `report` trả về `ExitCode`. Tham khảo documentation
của thư viện chuẩn để biết thêm thông tin về việc triển khai trait
`Termination` cho các kiểu của bạn.

Bây giờ chúng ta đã thảo luận chi tiết về việc gọi `panic!` hoặc trả về
`Result`, hãy quay lại chủ đề cách quyết định cái nào thích hợp
để sử dụng trong những trường hợp nào.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html
