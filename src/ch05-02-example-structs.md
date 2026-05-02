## Một Ví dụ Program Sử dụng Structs

Để hiểu khi nào chúng ta có thể muốn sử dụng structs, hãy viết một program tính
diện tích của một hình chữ nhật. Chúng ta sẽ bắt đầu bằng cách sử dụng các biến
đơn lẻ và sau đó refactor program cho đến khi chúng ta sử dụng structs.

Hãy tạo một project binary mới với Cargo gọi là _rectangles_ sẽ lấy chiều rộng
và chiều cao của một hình chữ nhật được chỉ định trong pixels và tính diện tích
của hình chữ nhật. Listing 5-8 hiển thị một program ngắn với một cách để làm
điều đó chính xác trong _src/main.rs_ của project của chúng ta.

<Listing number="5-8" file-name="src/main.rs" caption="Calculating the area of a rectangle specified by separate width and height variables">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Bây giờ, hãy chạy program này bằng `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Mã này thành công trong việc tính diện tích của hình chữ nhật bằng cách gọi
function `area` với mỗi kích thước, nhưng chúng ta có thể làm nhiều hơn để làm
cho mã này rõ ràng và dễ đọc hơn.

Vấn đề với mã này rõ ràng trong signature của `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Function `area` được cho là tính diện tích của một hình chữ nhật, nhưng function
chúng ta viết có hai tham số, và không rõ ràng ở bất kỳ nơi nào trong program
của chúng ta rằng các tham số có liên quan. Sẽ dễ đọc hơn và quản lý hơn để
nhóm width và height lại với nhau. Chúng ta đã thảo luận một cách chúng ta có
thể làm điều đó trong phần [“The Tuple Type”][the-tuple-type]<!-- ignore -->
của Chương 3: bằng cách sử dụng tuples.

### Refactoring với Tuples

Listing 5-9 hiển thị một phiên bản khác của program của chúng ta sử dụng tuples.

<Listing number="5-9" file-name="src/main.rs" caption="Specifying the width and height of the rectangle with a tuple">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

Theo một cách, program này tốt hơn. Tuples cho phép chúng ta thêm một chút
structure, và bây giờ chúng ta chỉ truyền một argument. Nhưng theo cách khác,
phiên bản này kém rõ ràng hơn: Tuples không đặt tên cho các phần tử của chúng,
vì vậy chúng ta phải index vào các phần của tuple, làm cho calculation của chúng
ta kém rõ ràng hơn.

Mixing up width và height sẽ không quan trọng cho area calculation, nhưng nếu
chúng ta muốn draw rectangle trên screen, nó sẽ quan trọng! Chúng ta sẽ phải
ghi nhớ rằng `width` là tuple index `0` và `height` là tuple index `1`. Điều
này sẽ còn khó hơn để người khác tìm ra và ghi nhớ nếu họ sử dụng mã của chúng
ta. Vì chúng ta chưa truyền đạt ý nghĩa của dữ liệu của chúng ta trong mã,
bây giờ dễ dàng hơn để introduce errors.

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### Refactoring với Structs

Chúng ta sử dụng structs để thêm ý nghĩa bằng cách gắn nhãn dữ liệu. Chúng ta
có thể transform tuple chúng ta đang sử dụng thành một struct với một tên cho
toàn bộ cũng như tên cho các phần, như hiển thị trong Listing 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Defining a `Rectangle` struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã định nghĩa một struct và đặt tên nó là `Rectangle`. Bên
trong dấu ngoặc nhọn, chúng ta đã định nghĩa các field là `width` và `height`,
cả hai đều có kiểu `u32`. Sau đó, trong `main`, chúng ta đã tạo một instance
cụ thể của `Rectangle` có width là `30` và height là `50`.

Function `area` của chúng ta bây giờ được định nghĩa với một tham số, được
đặt tên `rectangle`, có kiểu là một immutable borrow của một instance struct
`Rectangle`. Như đã đề cập trong Chương 4, chúng ta muốn borrow struct thay vì
take ownership của nó. Bằng cách này, `main` giữ ownership của nó và có thể
tiếp tục sử dụng `rect1`, đó là lý do chúng ta sử dụng `&` trong function
signature và nơi chúng ta gọi function.

Function `area` truy cập các field `width` và `height` của instance `Rectangle`
(lưu ý rằng truy cập các field của một borrowed struct instance không move các
field values, đó là lý do bạn thường thấy borrows của structs). Function
signature của chúng ta cho `area` bây giờ nói chính xác ý nghĩa của chúng ta:
Tính diện tích của `Rectangle`, sử dụng các field `width` và `height` của nó.
Điều này truyền đạt rằng width và height liên quan đến nhau, và nó cung cấp
các tên mô tả cho các giá trị thay vì sử dụng các giá trị tuple index `0` và
`1`. Đây là một thắng lợi cho sự rõ ràng.

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### Thêm Functionality với Derived Traits

Sẽ hữu ích khi có thể print một instance của `Rectangle` trong khi chúng ta
đang debugging program của chúng ta và thấy các giá trị cho tất cả các field
của nó. Listing 5-11 thử sử dụng macro [`println!`][println]<!-- ignore -->
khi chúng ta đã sử dụng trong các chương trước. Tuy nhiên, điều này sẽ không
hoạt động.

<Listing number="5-11" file-name="src/main.rs" caption="Attempting to print a `Rectangle` instance">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Khi chúng ta compile code này, chúng ta nhận được một lỗi với message cốt lõi
này:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Macro `println!` có thể thực hiện nhiều loại formatting, và theo mặc định, các
dấu ngoặc nhọn cho `println!` biết sử dụng formatting được gọi là `Display`:
output dự định cho direct end user consumption. Các primitive types chúng ta đã
thấy cho đến nay implement `Display` theo mặc định vì chỉ có một cách bạn muốn
hiển thị `1` hoặc bất kỳ primitive type nào khác cho user. Nhưng với structs,
cách `println!` nên format output kém rõ ràng hơn vì có nhiều display
possibilities hơn: Bạn có muốn commas không? Bạn có muốn print các dấu ngoặc
nhọn không? Tất cả các field nên được hiển thị không? Do ambiguity này, Rust
không cố gắng đoán những gì chúng ta muốn, và structs không có một provided
implementation của `Display` để sử dụng với `println!` và placeholder `{}`.

Nếu chúng ta tiếp tục đọc các lỗi, chúng ta sẽ tìm thấy note hữu ích này:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Hãy thử nó! Cuộc gọi macro `println!` sẽ bây giờ trông như `println!("rect1 is
{rect1:?}");`. Đặt specifier `:?` bên trong dấu ngoặc nhọn cho `println!` biết
chúng ta muốn sử dụng một output format được gọi là `Debug`. Trait `Debug` cho
phép chúng ta print struct của chúng ta theo một cách hữu ích cho developers để
chúng ta có thể thấy giá trị của nó trong khi chúng ta đang debugging mã.

Compile code với thay đổi này. Drat! Chúng ta vẫn nhận được một lỗi:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Nhưng một lần nữa, compiler cung cấp cho chúng ta một note hữu ích:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _có_ bao gồm functionality để print debugging information, nhưng chúng ta
phải explicitly opt in để làm cho functionality đó khả dụng cho struct của
chúng ta. Để làm điều đó, chúng ta thêm outer attribute `#[derive(Debug)]` ngay
trước định nghĩa struct, như hiển thị trong Listing 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Adding the attribute to derive the `Debug` trait and printing the `Rectangle` instance using debug formatting">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Bây giờ khi chúng ta chạy program, chúng ta sẽ không nhận được bất kỳ lỗi nào, và
chúng ta sẽ thấy output sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Tốt! Nó không phải là output đẹp nhất, nhưng nó hiển thị các giá trị của tất cả
các field cho instance này, điều này chắc chắn sẽ giúp trong debugging. Khi chúng
ta có các struct lớn hơn, thật hữu ích khi có output dễ đọc hơn một chút; trong
những trường hợp đó, chúng ta có thể sử dụng `{:#?}` thay vì `{:?}` trong string
`println!`. Trong ví dụ này, sử dụng style `{:#?}` sẽ output như sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Một cách khác để print một giá trị sử dụng format `Debug` là sử dụng macro
[`dbg!`][dbg]<!-- ignore -->, lấy ownership của một expression (trái ngược với
`println!`, lấy một reference), print file và line number nơi mà cuộc gọi macro
`dbg!` xảy ra trong mã của bạn cùng với resultant value của expression đó, và
trả về ownership của giá trị.

> Lưu ý: Gọi macro `dbg!` print tới standard error console stream (`stderr`),
> trái ngược với `println!`, in tới standard output console stream (`stdout`).
> Chúng ta sẽ nói thêm về `stderr` và `stdout` trong phần [“Redirecting Errors
> to Standard Error” trong Chương 12][err]<!-- ignore -->.

Dưới đây là một ví dụ nơi chúng ta quan tâm đến giá trị được gán cho field
`width`, cũng như giá trị của toàn bộ struct trong `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Chúng ta có thể đặt `dbg!` xung quanh expression `30 * scale` và, vì `dbg!`
trả về ownership của giá trị expression, field `width` sẽ nhận cùng giá trị
như nếu chúng ta không có cuộc gọi `dbg!` ở đó. Chúng ta không muốn `dbg!`
take ownership của `rect1`, vì vậy chúng ta sử dụng một reference tới `rect1`
trong cuộc gọi tiếp theo. Dưới đây là output của ví dụ này trông như thế nào:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Chúng ta có thể thấy phần đầu tiên của output đến từ line 10 của _src/main.rs_
nơi chúng ta đang debugging expression `30 * scale`, và resultant value của
nó là `60` (debug formatting được implement cho integers là print chỉ có giá
trị của chúng). Cuộc gọi `dbg!` trên line 14 của _src/main.rs_ output giá trị
của `&rect1`, đó là struct `Rectangle`. Output này sử dụng pretty `Debug`
formatting của kiểu `Rectangle`. Macro `dbg!` có thể thực sự hữu ích khi bạn
đang cố gắng tìm ra code của bạn đang làm gì!

Ngoài trait `Debug`, Rust đã cung cấp một số traits để chúng ta sử dụng với
attribute `derive` có thể thêm useful behavior vào các custom types của chúng
ta. Các traits đó và behaviors của chúng được liệt kê trong [Appendix C]
[app-c]<!-- ignore -->. Chúng ta sẽ cover cách implement các traits này với
custom behavior cũng như cách tạo traits của riêng bạn trong Chương 10. Cũng
có nhiều attributes khác ngoài `derive`; để biết thêm thông tin, hãy xem
[phần “Attributes” của Rust Reference][attributes].

Function `area` của chúng ta rất cụ thể: Nó chỉ tính diện tích của rectangles.
Sẽ hữu ích khi tie behavior này gần hơn tới struct `Rectangle` của chúng ta
vì nó sẽ không hoạt động với bất kỳ loại nào khác. Hãy xem cách chúng ta có
thể tiếp tục refactor code này bằng cách turn function `area` thành một method
`area` được định nghĩa trên kiểu `Rectangle` của chúng ta.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
