## Xác Thực References với Lifetimes

Lifetimes là một loại generic khác mà chúng ta đã sử dụng. Thay vì đảm bảo rằng
một type có hành vi mà chúng ta muốn, lifetimes đảm bảo rằng các references
vẫn hợp lệ miễn là chúng ta cần chúng.

Một chi tiết mà chúng ta không thảo luận trong phần [“References and
Borrowing”][references-and-borrowing]<!-- ignore --> ở Chương 4 là
mỗi reference trong Rust đều có một lifetime, là scope mà
reference đó hợp lệ. Hầu hết thời gian, lifetimes được ngầm định và suy luận,
giống như hầu hết thời gian, types được suy luận. Chúng ta chỉ cần
chú thích types khi có thể có nhiều types. Tương tự như vậy, chúng ta phải
chú thích lifetimes khi lifetimes của các references có thể liên quan theo một vài
cách khác nhau. Rust yêu cầu chúng ta chú thích các mối quan hệ bằng cách sử dụng generic
lifetime parameters để đảm bảo rằng các references thực tế được sử dụng khi chạy
sẽ chắc chắn hợp lệ.

Chú thích lifetimes thậm chí không phải là một khái niệm mà hầu hết các ngôn ngữ lập trình khác
có, vì vậy điều này sẽ cảm thấy lạ lẫm. Mặc dù chúng ta sẽ không bao quát lifetimes
hoàn toàn trong chương này, chúng ta sẽ thảo luận các cách phổ biến mà bạn có thể gặp
lifetime syntax để bạn có thể làm quen với khái niệm.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### Dangling References

Mục đích chính của lifetimes là ngăn chặn dangling references, mà nếu được
phép tồn tại, sẽ khiến chương trình tham chiếu đến dữ liệu khác ngoài
dữ liệu mà nó dự định tham chiếu. Xem xét chương trình trong Listing 10-16, có
một outer scope và một inner scope.

<Listing number="10-16" caption="Một nỗ lực sử dụng một reference có giá trị đã vượt ra ngoài scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> Lưu ý: Các ví dụ trong Listings 10-16, 10-17, và 10-23 khai báo các biến
> mà không cho chúng một giá trị ban đầu, vì vậy tên biến tồn tại trong outer
> scope. Thoạt nhìn, điều này có vẻ mâu thuẫn với Rust không có
> các giá trị null. Tuy nhiên, nếu chúng ta cố gắng sử dụng một biến trước khi cho nó một giá trị,
> chúng ta sẽ gặp lỗi compile-time, chứng tỏ rằng thực sự Rust không cho phép
> các giá trị null.

Outer scope khai báo một biến có tên `r` mà không có giá trị ban đầu, và
inner scope khai báo một biến có tên `x` với giá trị ban đầu là `5`. Bên trong
inner scope, chúng ta cố gắng đặt giá trị của `r` như một reference đến `x`.
Sau đó, inner scope kết thúc, và chúng ta cố gắng in giá trị trong `r`. Code này
sẽ không biên dịch được, vì giá trị mà `r` đang tham chiếu đã vượt ra ngoài scope
trước khi chúng ta cố gắng sử dụng nó. Đây là thông báo lỗi:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Thông báo lỗi nói rằng biến `x` “không sống đủ lâu.” Lý do là `x` sẽ
ra khỏi scope khi inner scope kết thúc trên dòng 7.
Nhưng `r` vẫn còn hợp lệ cho outer scope; vì scope của nó lớn hơn, chúng ta nói
rằng nó “sống lâu hơn.” Nếu Rust cho phép code này hoạt động, `r` sẽ
tham chiếu đến bộ nhớ đã bị giải phóng khi `x` vượt ra ngoài scope, và
bất kỳ điều gì chúng ta cố gắng làm với `r` sẽ không hoạt động chính xác. Vậy, Rust
xác định rằng code này không hợp lệ như thế nào? Nó sử dụng một borrow checker.

### The Borrow Checker

Trình biên dịch Rust có một _borrow checker_ so sánh các scopes để xác định
liệu tất cả các borrows có hợp lệ không. Listing 10-17 hiển thị code giống như Listing
10-16 nhưng với các chú thích hiển thị lifetimes của các biến.

<Listing number="10-17" caption="Chú thích lifetimes của `r` và `x`, được gọi là `'a` và `'b`, tương ứng">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã chú thích lifetime của `r` với `’a` và lifetime của `x`
với `’b`. Như bạn có thể thấy, khối `’b` bên trong nhỏ hơn nhiều so với
khối lifetime `’a` bên ngoài. Tại thời biên dịch, Rust so sánh kích thước của hai
lifetimes và thấy rằng `r` có lifetime `’a` nhưng nó tham chiếu đến bộ nhớ
có lifetime `’b`. Chương trình bị từ chối vì `’b` ngắn hơn `’a`: Chủ thể của reference không sống lâu như reference.

Listing 10-18 sửa code để nó không có dangling reference và
nó biên dịch mà không có bất kỳ lỗi nào.

<Listing number="10-18" caption="Một reference hợp lệ vì dữ liệu có lifetime dài hơn reference">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Ở đây, `x` có lifetime `’b`, trong trường hợp này lớn hơn `’a`. Điều này
có nghĩa là `r` có thể tham chiếu đến `x` vì Rust biết rằng reference trong `r` sẽ
luôn hợp lệ khi `x` hợp lệ.

Bây giờ bạn biết lifetimes của các references ở đâu và cách Rust phân tích
lifetimes để đảm bảo rằng các references sẽ luôn hợp lệ, hãy khám phá generic
lifetimes trong các tham số function và giá trị trả về.

### Generic Lifetimes trong Functions

Chúng ta sẽ viết một function trả về cái dài hơn trong hai string slices. Function này
sẽ lấy hai string slices và trả về một string slice duy nhất. Sau
khi chúng ta đã triển khai function `longest`, code trong Listing 10-19 sẽ
in `The longest string is abcd`.

<Listing number="10-19" file-name="src/main.rs" caption="Một function `main` gọi function `longest` để tìm cái dài hơn trong hai string slices">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

Lưu ý rằng chúng ta muốn function lấy string slices, là các references,
thay vì strings, vì chúng ta không muốn function `longest` lấy
ownership của các tham số của nó. Tham khảo [“String Slices as
Parameters”][string-slices-as-parameters]<!-- ignore --> trong Chương 4 để có thêm
thảo luận về lý do các tham số chúng ta sử dụng trong Listing 10-19 là những cái chúng ta
muốn.

Nếu chúng ta cố gắng triển khai function `longest` như được hiển thị trong Listing 10-20, nó
sẽ không biên dịch được.

<Listing number="10-20" file-name="src/main.rs" caption="Một triển khai của function `longest` trả về cái dài hơn của hai string slices nhưng chưa biên dịch được">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

Thay vào đó, chúng ta nhận được lỗi sau nói về lifetimes:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Văn bản trợ giúp tiết lộ rằng kiểu trả về cần một generic lifetime parameter
vì Rust không thể nói liệu reference được trả về tham chiếu đến
`x` hay `y`. Trên thực tế, chúng ta cũng không biết, vì khối `if` trong thân
function này trả về một reference đến `x` và khối `else` trả về một
reference đến `y`!

Khi chúng ta đang định nghĩa function này, chúng ta không biết các giá trị cụ thể sẽ
được truyền vào function này, vì vậy chúng ta không biết liệu trường hợp `if` hay
trường hợp `else` sẽ được thực thi. Chúng ta cũng không biết lifetimes cụ thể của
các references sẽ được truyền vào, vì vậy chúng ta không thể xem scopes như chúng ta đã làm trong
Listings 10-17 và 10-18 để xác định liệu reference chúng ta trả về sẽ
luôn hợp lệ. Borrow checker cũng không thể xác định được điều này, vì nó
không biết lifetimes của `x` và `y` liên quan như thế nào đến lifetime của
giá trị trả về. Để sửa lỗi này, chúng ta sẽ thêm generic lifetime parameters
định nghĩa mối quan hệ giữa các references để borrow checker có thể
thực hiện phân tích của nó.

### Lifetime Annotation Syntax

Lifetime annotations không thay đổi thời gian sống của các references. Thay vào đó,
chúng mô tả các mối quan hệ của lifetimes của các references múl tới
nhau mà không ảnh hưởng đến lifetimes. Giống như functions có thể chấp nhận bất kỳ type
khi signature chỉ định một generic type parameter, functions có thể chấp nhận
references với bất kỳ lifetime nào bằng cách chỉ định một generic lifetime parameter.

Lifetime annotations có một cú pháp hơi bất thường: Tên của lifetime
parameters phải bắt đầu bằng dấu ngoặc đơn (`’`) và thường là viết thường
và rất ngắn, giống như generic types. Hầu hết mọi người sử dụng tên `’a` cho lần đầu
lifetime annotation. Chúng ta đặt lifetime parameter annotations sau `&` của một
reference, sử dụng khoảng trắng để tách annotation khỏi type của reference.

Dưới đây là một số ví dụ—một reference đến `i32` mà không có lifetime parameter, một
reference đến `i32` có lifetime parameter được gọi là `’a`, và một mutable
reference đến `i32` cũng có lifetime `’a`:

```rust,ignore
&i32        // a reference
&’a i32     // a reference with an explicit lifetime
&’a mut i32 // a mutable reference with an explicit lifetime
```

Một lifetime annotation tự nó không có nhiều ý nghĩa, vì
annotations được cho là để nói cho Rust biết cách generic lifetime parameters của nhiều
references liên quan đến nhau. Hãy kiểm tra cách lifetime annotations
liên quan đến nhau trong bối cảnh function `longest`.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-function-signatures"></a>

### Trong Function Signatures

Để sử dụng lifetime annotations trong function signatures, chúng ta cần khai báo
generic lifetime parameters bên trong dấu ngoặc nhọn giữa tên function và
danh sách tham số, giống như chúng ta đã làm với generic type parameters.

Chúng ta muốn signature để diễn đạt ràng buộc sau: Reference được trả về
sẽ hợp lệ miễn là cả hai tham số hợp lệ. Đây là
mối quan hệ giữa lifetimes của các tham số và giá trị trả về.
Chúng ta sẽ đặt tên lifetime `’a` và sau đó thêm nó vào mỗi reference, như được hiển thị trong
Listing 10-21.

<Listing number="10-21" file-name="src/main.rs" caption="Định nghĩa function `longest` chỉ định rằng tất cả các references trong signature phải có cùng lifetime `'a`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

Code này nên biên dịch được và tạo ra kết quả chúng ta muốn khi chúng ta sử dụng nó với
function `main` trong Listing 10-19.

Function signature bây giờ nói cho Rust biết rằng với một lifetime `’a` nào đó, function
lấy hai tham số, cả hai là string slices sống ít nhất miễn là
lifetime `’a`. Function signature cũng nói cho Rust biết rằng string
slice trả về từ function sẽ sống ít nhất miễn là lifetime `’a`.
Trong thực tế, nó có nghĩa là lifetime của reference được trả về bởi
function `longest` giống như cái nhỏ hơn của lifetimes của các giá trị
được tham chiếu bởi các function arguments. Những mối quan hệ này là những gì chúng ta muốn
Rust sử dụng khi phân tích code này.

Hãy nhớ, khi chúng ta chỉ định lifetime parameters trong function signature này,
chúng ta không phải thay đổi lifetimes của bất kỳ giá trị nào được truyền vào hoặc trả về. Thay vào đó,
chúng ta đang chỉ định rằng borrow checker nên từ chối bất kỳ giá trị nào không
tuân theo các ràng buộc này. Lưu ý rằng function `longest` không cần
biết chính xác `x` và `y` sẽ sống bao lâu, chỉ cần có một scope nào đó có thể
được thay thế cho `’a` sẽ thỏa mãn signature này.

Khi chú thích lifetimes trong functions, các annotations đi vào function
signature, không phải trong function body. Lifetime annotations trở thành phần của
contract của function, giống như các types trong signature. Có
function signatures chứa lifetime contract có nghĩa là phân tích mà Rust
compiler làm có thể đơn giản hơn. Nếu có vấn đề với cách function được
chú thích hoặc cách nó được gọi, lỗi compiler có thể chỉ đến phần của
code của chúng ta và các ràng buộc chính xác hơn. Nếu, thay vào đó, Rust compiler
đã đưa ra nhiều suy luận hơn về những gì chúng ta dự định các mối quan hệ của lifetimes
là gì, compiler chỉ có thể chỉ đến một lần sử dụng code của chúng ta rất xa
từ nguyên nhân của vấn đề.

Khi chúng ta truyền các concrete references đến `longest`, concrete lifetime được
thay thế cho `’a` là phần của scope của `x` mà trùng lặp với
scope của `y`. Nói cách khác, generic lifetime `’a` sẽ nhận được concrete
lifetime bằng với cái nhỏ hơn của lifetimes của `x` và `y`. Vì
chúng ta đã chú thích returned reference với cùng lifetime parameter `’a`,
returned reference cũng sẽ hợp lệ với độ dài của cái nhỏ hơn của
lifetimes của `x` và `y`.

Hãy xem cách lifetime annotations hạn chế function `longest` bằng cách
truyền các references có lifetimes cụ thể khác nhau. Listing 10-22 là
một ví dụ đơn giản.

<Listing number="10-22" file-name="src/main.rs" caption="Sử dụng function `longest` với các references đến `String` values có lifetimes cụ thể khác nhau">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

Trong ví dụ này, `string1` hợp lệ cho đến cuối outer scope, `string2`
hợp lệ cho đến cuối inner scope, và `result` tham chiếu đến một cái gì đó
hợp lệ cho đến cuối inner scope. Chạy code này và bạn sẽ thấy
rằng borrow checker phê duyệt; nó sẽ biên dịch và in `The longest string
is long string is long`.

Tiếp theo, hãy thử một ví dụ cho thấy rằng lifetime của reference trong
`result` phải là lifetime nhỏ hơn của hai arguments. Chúng ta sẽ di chuyển
khai báo của biến `result` ra ngoài inner scope nhưng để lại
gán giá trị cho biến `result` bên trong scope với
`string2`. Sau đó, chúng ta sẽ di chuyển `println!` sử dụng `result` ra ngoài
inner scope, sau khi inner scope đã kết thúc. Code trong Listing 10-23 sẽ
không biên dịch được.

<Listing number="10-23" file-name="src/main.rs" caption="Cố gắng sử dụng `result` sau khi `string2` đã vượt ra ngoài scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

Khi chúng ta cố gắng biên dịch code này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Lỗi cho thấy rằng để `result` hợp lệ cho statement `println!`,
`string2` sẽ cần phải hợp lệ cho đến cuối outer scope. Rust biết
điều này vì chúng ta chú thích lifetimes của function parameters và return
values bằng cách sử dụng cùng lifetime parameter `’a`.

Là con người, chúng ta có thể xem code này và thấy rằng `string1` dài hơn
`string2`, và do đó, `result` sẽ chứa một reference đến `string1`.
Vì `string1` chưa vượt ra ngoài scope, một reference đến `string1` sẽ
vẫn còn hợp lệ cho statement `println!`. Tuy nhiên, compiler không thể nhìn thấy
rằng reference hợp lệ trong trường hợp này. Chúng ta đã nói cho Rust biết rằng lifetime của
reference được trả về bởi function `longest` giống như cái nhỏ hơn của
lifetimes của các references được truyền vào. Do đó, borrow checker
không cho phép code trong Listing 10-23 vì có thể có một reference không hợp lệ.

Cố gắng thiết kế thêm các thí nghiệm thay đổi các values và lifetimes của
các references được truyền vào function `longest` và cách returned reference
được sử dụng. Đưa ra các giả thuyết về liệu các thí nghiệm của bạn có sẽ vượt qua
borrow checker hay không trước khi bạn biên dịch; sau đó, kiểm tra xem bạn có đúng không!

<!-- Old headings. Do not remove or links may break. -->

<a id="thinking-in-terms-of-lifetimes"></a>

### Relationships

Cách bạn cần chỉ định lifetime parameters phụ thuộc vào function của bạn đang làm gì.
Ví dụ, nếu chúng ta thay đổi triển khai function
`longest` để luôn trả về tham số đầu tiên thay vì longest
string slice, chúng ta sẽ không cần chỉ định lifetime cho tham số `y`. Code
sau sẽ biên dịch được:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

Chúng ta đã chỉ định một lifetime parameter `’a` cho tham số `x` và return
type, nhưng không phải cho tham số `y`, vì lifetime của `y` không có
bất kỳ mối quan hệ nào với lifetime của `x` hoặc return value.

Khi trả về một reference từ một function, lifetime parameter cho
return type cần phải khớp với lifetime parameter cho một trong các tham số. Nếu
reference được trả về _không_ tham chiếu đến một trong các tham số, nó phải tham chiếu
đến một value được tạo trong function này. Tuy nhiên, đây sẽ là một
dangling reference vì value sẽ vượt ra ngoài scope ở cuối function.
Hãy xem xét triển khai cố gắng này của function `longest` sẽ không
biên dịch được:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

Ở đây, mặc dù chúng ta đã chỉ định một lifetime parameter `’a` cho return
type, triển khai này sẽ không biên dịch được vì return value
lifetime hoàn toàn không liên quan đến lifetime của các tham số. Đây là
thông báo lỗi chúng ta nhận được:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Vấn đề là `result` vượt ra ngoài scope và bị dọn dẹp ở cuối
function `longest`. Chúng ta cũng đang cố gắng trả về một reference đến `result`
từ function. Không có cách nào chúng ta có thể chỉ định lifetime parameters
sẽ thay đổi dangling reference, và Rust sẽ không cho phép chúng ta tạo một dangling
reference. Trong trường hợp này, sửa chữa tốt nhất sẽ là trả về một owned data type
thay vì một reference để function gọi thì chịu trách nhiệm
dọn dẹp value.

Cuối cùng, lifetime syntax là về kết nối lifetimes của các
tham số khác nhau và return values của functions. Một khi chúng được kết nối, Rust có
đủ thông tin để cho phép các hoạt động an toàn bộ nhớ và không cho phép các hoạt động
sẽ tạo dangling pointers hoặc vi phạm memory safety theo cách khác.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-struct-definitions"></a>

### Trong Struct Definitions

Cho đến nay, các structs mà chúng ta đã định nghĩa đều giữ các owned types. Chúng ta có thể định nghĩa structs
để giữ references, nhưng trong trường hợp đó, chúng ta sẽ cần thêm một lifetime
annotation trên mỗi reference trong struct definition. Listing 10-24 có một
struct được gọi là `ImportantExcerpt` giữ một string slice.

<Listing number="10-24" file-name="src/main.rs" caption="Một struct giữ một reference, yêu cầu lifetime annotation">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

Struct này có trường duy nhất `part` giữ một string slice, là một
reference. Như với generic data types, chúng ta khai báo tên của generic
lifetime parameter bên trong dấu ngoặc nhọn sau tên của struct để
chúng ta có thể sử dụng lifetime parameter trong body của struct definition. Annotation này
có nghĩa là một instance của `ImportantExcerpt` không thể sống lâu hơn reference
nó giữ trong trường `part` của nó.

Function `main` ở đây tạo ra một instance của struct `ImportantExcerpt`
giữ một reference đến câu đầu tiên của `String` được sở hữu bởi
biến `novel`. Dữ liệu trong `novel` tồn tại trước khi `ImportantExcerpt`
instance được tạo. Ngoài ra, `novel` không vượt ra ngoài scope cho đến sau khi
`ImportantExcerpt` vượt ra ngoài scope, vì vậy reference trong
instance `ImportantExcerpt` hợp lệ.

### Lifetime Elision

Bạn đã học rằng mỗi reference có một lifetime và bạn cần chỉ định
lifetime parameters cho functions hoặc structs sử dụng references. Tuy nhiên, chúng ta
có một function trong Listing 4-9, được hiển thị lại trong Listing 10-25, được biên dịch
mà không có lifetime annotations.

<Listing number="10-25" file-name="src/lib.rs" caption="Một function chúng ta định nghĩa trong Listing 4-9 được biên dịch mà không có lifetime annotations, mặc dù tham số và return type là references">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

Lý do function này biên dịch mà không có lifetime annotations là vì lịch sử:
Trong các phiên bản đầu tiên (pre-1.0) của Rust, code này sẽ không được biên dịch, vì
mỗi reference cần một explicit lifetime. Tại thời điểm đó, function
signature sẽ được viết như thế này:

```rust,ignore
fn first_word<’a>(s: &’a str) -> &’a str {
```

Sau khi viết rất nhiều Rust code, nhóm Rust phát hiện rằng các Rust programmers
đang nhập các lifetime annotations giống nhau đi và lại trong những
tình huống cụ thể. Những tình huống này có thể dự đoán được và tuân theo một vài
patterns xác định. Các developers đã lập trình các patterns này vào mã trình biên dịch để
borrow checker có thể suy luận lifetimes trong những tình huống này và
không cần explicit annotations.

Phần lịch sử Rust này có liên quan vì có khả năng sẽ xuất hiện thêm các
patterns xác định được thêm vào trình biên dịch. Trong tương lai,
thậm chí có thể cần ít lifetime annotations hơn.

Các patterns được lập trình vào phân tích references của Rust được gọi là
_lifetime elision rules_. Đây không phải là các quy tắc để programmers tuân theo; chúng
là một tập hợp các trường hợp cụ thể mà trình biên dịch sẽ xem xét, và nếu code của bạn
phù hợp với các trường hợp này, bạn không cần viết lifetimes một cách rõ ràng.

Các elision rules không cung cấp full inference. Nếu vẫn còn sự mơ hồ
về lifetimes của các references sau khi Rust áp dụng các quy tắc, trình biên dịch
sẽ không đoán lifetime của các references còn lại phải là gì.
Thay vì đoán, trình biên dịch sẽ cho bạn một lỗi mà bạn có thể giải quyết
bằng cách thêm lifetime annotations.

Lifetimes trên function hoặc method parameters được gọi là _input lifetimes_, và
lifetimes trên return values được gọi là _output lifetimes_.

Trình biên dịch sử dụng ba quy tắc để tìm ra lifetimes của các references
khi không có explicit annotations. Quy tắc đầu tiên áp dụng cho input
lifetimes, và quy tắc thứ hai và thứ ba áp dụng cho output lifetimes. Nếu
trình biên dịch đến cuối ba quy tắc và vẫn còn các references
mà nó không thể tìm ra lifetimes, trình biên dịch sẽ dừng lại với một lỗi.
Các quy tắc này áp dụng cho cả `fn` definitions cũng như `impl` blocks.

Quy tắc đầu tiên là trình biên dịch gán một lifetime parameter cho mỗi
tham số là một reference. Nói cách khác, một function có một tham số
nhận một lifetime parameter: `fn foo<’a>(x: &’a i32)`; một function có hai
tham số nhận hai lifetime parameters riêng biệt: `fn foo<’a, ‘b>(x: &’a i32,
y: &’b i32)`; v.v.

Quy tắc thứ hai là, nếu có chính xác một input lifetime parameter, thì
lifetime đó được gán cho tất cả output lifetime parameters: `fn foo<’a>(x: &’a i32)
-> &’a i32`.

Quy tắc thứ ba là, nếu có nhiều input lifetime parameters, nhưng
một trong số chúng là `&self` hoặc `&mut self` vì đây là một method, lifetime của
`self` được gán cho tất cả output lifetime parameters. Quy tắc thứ ba này
làm cho methods dễ đọc và viết hơn nhiều vì cần ít kí hiệu hơn.
Hãy giả sử chúng ta là compiler. Ta sẽ áp dụng các quy tắc để xác định lifetime của các tham chiếu trong chữ ký của hàm `first_word` ở Listing 10-25. Ban đầu, chữ ký không có lifetime nào được gắn với các tham chiếu:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Sau đó, compiler áp dụng quy tắc thứ nhất: mỗi tham số sẽ có một lifetime riêng. Ta gọi nó là `'a` như thường lệ, và chữ ký trở thành:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Tiếp theo, quy tắc thứ hai được áp dụng vì chỉ có đúng một lifetime đầu vào. Quy tắc này nói rằng lifetime của tham số đầu vào duy nhất sẽ được gán cho lifetime của giá trị trả về, nên chữ ký giờ là:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Giờ đây tất cả các tham chiếu trong chữ ký hàm đều đã có lifetime, và compiler có thể tiếp tục phân tích mà không cần lập trình viên phải tự annotate lifetime trong trường hợp này.

***

Hãy xem một ví dụ khác, lần này với hàm `longest` (ban đầu không có tham số lifetime ở Listing 10-20):

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Áp dụng quy tắc thứ nhất: mỗi tham số có lifetime riêng. Vì có hai tham số, ta có hai lifetime:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Bạn có thể thấy quy tắc thứ hai không áp dụng vì có nhiều hơn một lifetime đầu vào. Quy tắc thứ ba cũng không áp dụng vì `longest` là một hàm (function), không phải method, nên không có tham số `self`.

Sau khi áp dụng cả ba quy tắc, ta vẫn không xác định được lifetime của kiểu trả về. Đây chính là lý do code ở Listing 10-20 bị lỗi khi compile: compiler đã thử áp dụng lifetime elision rules nhưng vẫn không suy ra được đầy đủ lifetime.


Vì quy tắc thứ ba chủ yếu áp dụng cho method, nên tiếp theo ta sẽ xem xét lifetime trong ngữ cảnh method để hiểu tại sao trong method thường không cần annotate lifetime.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-method-definitions"></a>

### Trong định nghĩa method

Khi implement method cho một struct có lifetime, ta dùng cùng cú pháp như generic type parameters (như ở Listing 10-11). Việc khai báo và sử dụng lifetime phụ thuộc vào việc chúng liên quan đến field của struct hay tham số/giá trị trả về của method.

Lifetime của field trong struct luôn phải được khai báo sau từ khóa `impl`, rồi dùng sau tên struct, vì chúng là một phần của kiểu struct.

Trong chữ ký method bên trong khối `impl`, các tham chiếu có thể:
- Gắn với lifetime của field trong struct, hoặc
- Hoàn toàn độc lập

Ngoài ra, lifetime elision rules thường giúp ta không cần annotate lifetime trong method.

Hãy xem ví dụ với struct `ImportantExcerpt` (Listing 10-24).

Đầu tiên là method `level`, chỉ nhận `&self` và trả về `i32` (không phải tham chiếu):

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

Việc khai báo lifetime sau `impl` và sử dụng sau tên type là bắt buộc. Nhưng nhờ quy tắc elision thứ nhất, ta không cần annotate lifetime cho `&self`.


Ví dụ sau áp dụng quy tắc elision thứ ba:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Có hai lifetime đầu vào, nên Rust áp dụng quy tắc thứ nhất và gán mỗi tham số (`&self` và `announcement`) một lifetime riêng. Sau đó, vì có `&self`, quy tắc thứ ba nói rằng lifetime của `&self` sẽ được dùng cho giá trị trả về. Như vậy tất cả lifetime đã được xác định.

### Lifetime `'static`

Một lifetime đặc biệt là `'static`, biểu thị rằng tham chiếu có thể tồn tại trong suốt thời gian chạy của chương trình.

Tất cả string literal đều có lifetime `'static`:

```rust
let s: &'static str = "I have a static lifetime.";
```

Nội dung của chuỗi này được lưu trực tiếp trong binary của chương trình và luôn tồn tại, nên lifetime của nó là `'static`.

Bạn có thể thấy compiler gợi ý dùng `'static` trong thông báo lỗi. Tuy nhiên, trước khi làm vậy, hãy cân nhắc:
- Tham chiếu đó có thực sự sống suốt chương trình không?
- Bạn có thực sự muốn như vậy không?

Trong đa số trường hợp, việc compiler gợi ý `'static` là do:
- Bạn tạo dangling reference, hoặc
- Lifetime bị mismatch

Khi đó, giải pháp đúng là sửa logic code, không phải thêm `'static`.

<!-- Old headings. Do not remove or links may break. -->

<a id="generic-type-parameters-trait-bounds-and-lifetimes-together"></a>

## Generic type, trait bound và lifetime

Hãy xem cú pháp khi kết hợp cả ba:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Đây là hàm `longest` từ Listing 10-21, trả về chuỗi dài hơn trong hai chuỗi. Nhưng giờ nó có thêm tham số `ann` kiểu generic `T`, với ràng buộc trait `Display` (qua `where`).

Tham số này sẽ được in bằng `{}`, nên cần trait `Display`.

Vì lifetime cũng là một dạng generic, nên `'a` và `T` được khai báo chung trong dấu `<...>` sau tên hàm.

## Tổng kết

Trong chương này, bạn đã học:

- Generic type parameters: giúp code dùng được cho nhiều kiểu.
- Traits và trait bounds: đảm bảo kiểu generic có hành vi cần thiết.
- Lifetime annotations: đảm bảo không có dangling reference.

Quan trọng là tất cả phân tích này diễn ra ở compile-time, không ảnh hưởng đến hiệu năng runtime.

Tuy nhiên, vẫn còn nhiều điều nâng cao: Chapter 18 nói về trait objects. Một số trường hợp lifetime phức tạp hơn chỉ xuất hiện trong tình huống nâng cao (có thể tham khảo [Rust Reference][reference]). Tiếp theo, bạn sẽ học cách viết test trong Rust để đảm bảo code hoạt động đúng như mong đợi.

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/trait-bounds.html