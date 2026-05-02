## Định nghĩa và Khởi tạo Structs

Structs tương tự như tuples được thảo luận trong phần [“The Tuple Type”][tuples]<!--
ignore -->, ở chỗ cả hai đều giữ nhiều giá trị liên quan. Như tuples, các
phần của struct có thể có các kiểu khác nhau. Không giống như tuples, trong
struct bạn sẽ đặt tên cho từng phần dữ liệu để rõ ràng ý nghĩa của các giá trị.
Việc thêm các tên này có nghĩa là structs linh hoạt hơn tuples: Bạn không phải
dựa vào thứ tự của dữ liệu để chỉ định hoặc truy cập các giá trị của một instance.

Để định nghĩa một struct, chúng ta nhập keyword `struct` và đặt tên cho toàn bộ
struct. Tên của struct nên mô tả ý nghĩa của các phần dữ liệu được nhóm lại với
nhau. Sau đó, bên trong dấu ngoặc nhọn, chúng ta định nghĩa tên và kiểu của các
phần dữ liệu, được gọi là _fields_. Ví dụ, Listing 5-1 hiển thị một struct lưu trữ
thông tin về tài khoản người dùng.

<Listing number="5-1" file-name="src/main.rs" caption="A `User` struct definition">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Để sử dụng struct sau khi chúng ta đã định nghĩa nó, chúng ta tạo một _instance_
của struct đó bằng cách chỉ định các giá trị cụ thể cho mỗi field. Chúng ta tạo
một instance bằng cách nêu tên của struct và sau đó thêm dấu ngoặc nhọn chứa các
cặp _`key: value`_, trong đó các key là tên của các field và các value là dữ liệu
chúng ta muốn lưu trữ trong các field đó. Chúng ta không phải chỉ định các field
theo cùng thứ tự mà chúng ta khai báo trong struct. Nói cách khác, định nghĩa
struct giống như một template chung cho kiểu, và các instance điền vào template
đó với dữ liệu cụ thể để tạo các giá trị của kiểu. Ví dụ, chúng ta có thể khai
báo một người dùng cụ thể như hiển thị trong Listing 5-2.

<Listing number="5-2" file-name="src/main.rs" caption="Creating an instance of the `User` struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Để lấy một giá trị cụ thể từ struct, chúng ta sử dụng dot notation. Ví dụ, để
truy cập địa chỉ email của người dùng này, chúng ta sử dụng `user1.email`. Nếu
instance là mutable, chúng ta có thể thay đổi giá trị bằng cách sử dụng dot notation
và gán cho một field cụ thể. Listing 5-3 cho thấy cách thay đổi giá trị trong field
`email` của một instance `User` mutable.

<Listing number="5-3" file-name="src/main.rs" caption="Changing the value in the `email` field of a `User` instance">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Lưu ý rằng toàn bộ instance phải là mutable; Rust không cho phép chúng ta chỉ
đánh dấu các field nhất định là mutable. Như với bất kỳ expression nào, chúng
ta có thể xây dựng một instance mới của struct như expression cuối cùng trong
function body để implicitly return instance mới đó.

Listing 5-4 hiển thị một function `build_user` trả về một instance `User` với
email và username được cung cấp. Field `active` nhận giá trị `true`, và
`sign_in_count` nhận giá trị `1`.

<Listing number="5-4" file-name="src/main.rs" caption="A `build_user` function that takes an email and username and returns a `User` instance">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Có ý nghĩa để đặt tên các tham số function với cùng tên với các field của struct,
nhưng phải lặp lại tên field `email` và `username` cùng các biến là khá tẻ nhạt.
Nếu struct có nhiều field hơn, lặp lại từng tên sẽ trở nên còn khó chịu hơn. May
mắn thay, có một shorthand tiện lợi!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Sử dụng Field Init Shorthand

Vì tên tham số và tên field của struct hoàn toàn giống nhau trong Listing 5-4,
chúng ta có thể sử dụng cú pháp _field init shorthand_ để viết lại `build_user`
sao cho nó hoạt động hoàn toàn giống nhau nhưng không có sự lặp lại của
`username` và `email`, như hiển thị trong Listing 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="A `build_user` function that uses field init shorthand because the `username` and `email` parameters have the same name as struct fields">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta đang tạo một instance mới của struct `User`, có một field
được đặt tên là `email`. Chúng ta muốn đặt giá trị của field `email` thành giá
trị trong tham số `email` của function `build_user`. Vì field `email` và tham số
`email` có cùng tên, chúng ta chỉ cần viết `email` thay vì `email: email`.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### Tạo Instances với Struct Update Syntax

Thường thì hữu ích khi tạo một instance mới của struct chứa hầu hết các giá
trị từ một instance khác của cùng loại, nhưng thay đổi một số trong số đó. Bạn
có thể làm điều này bằng cách sử dụng struct update syntax.

Đầu tiên, trong Listing 5-6 chúng ta hiển thị cách tạo một instance `User` mới
trong `user2` theo cách thông thường, không có update syntax. Chúng ta đặt một
giá trị mới cho `email` nhưng sử dụng các giá trị giống nhau từ `user1` mà chúng
ta tạo trong Listing 5-2.

<Listing number="5-6" file-name="src/main.rs" caption="Creating a new `User` instance using all but one of the values from `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Sử dụng struct update syntax, chúng ta có thể đạt được hiệu quả tương tự với ít
mã hơn, như hiển thị trong Listing 5-7. Cú pháp `..` chỉ định rằng các field còn
lại không được đặt explicitly nên có cùng giá trị với các field trong instance
được cung cấp.

<Listing number="5-7" file-name="src/main.rs" caption="Using struct update syntax to set a new `email` value for a `User` instance but to use the rest of the values from `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Mã trong Listing 5-7 cũng tạo một instance trong `user2` có giá trị khác cho
`email` nhưng có cùng các giá trị cho các field `username`, `active`, và
`sign_in_count` từ `user1`. `..user1` phải đi cuối cùng để chỉ định rằng bất
kỳ field còn lại nào nên lấy giá trị của chúng từ các field tương ứng trong
`user1`, nhưng chúng ta có thể chọn chỉ định giá trị cho bao nhiêu field tùy
ý theo bất kỳ thứ tự nào, bất kể thứ tự của các field trong định nghĩa struct.

Lưu ý rằng struct update syntax sử dụng `=` như một assignment; điều này là
vì nó move dữ liệu, giống như chúng ta đã thấy trong phần [“Variables and Data
Interacting with Move”][move]<!-- ignore -->. Trong ví dụ này, chúng ta không
thể sử dụng `user1` nữa sau khi tạo `user2` vì `String` trong field `username`
của `user1` đã được move vào `user2`. Nếu chúng ta đã cung cấp các giá trị
`String` mới cho cả `email` và `username` cho `user2`, và do đó chỉ sử dụng
các giá trị `active` và `sign_in_count` từ `user1`, thì `user1` vẫn sẽ hợp lệ
sau khi tạo `user2`. Cả `active` và `sign_in_count` đều là các kiểu implement
trait `Copy`, vì vậy behavior chúng ta thảo luận trong phần [“Stack-Only Data:
Copy”][copy]<!-- ignore --> sẽ áp dụng. Chúng ta cũng vẫn có thể sử dụng
`user1.email` trong ví dụ này, vì giá trị của nó không được move ra khỏi
`user1`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### Tạo Các Kiểu Khác nhau với Tuple Structs

Rust cũng hỗ trợ structs giống như tuples, được gọi là _tuple structs_. Tuple
structs có ý nghĩa bổ sung được cung cấp bởi tên struct nhưng không có tên liên
kết với các field của chúng; thay vào đó, chúng chỉ có các kiểu của các field.
Tuple structs hữu ích khi bạn muốn cung cấp một tên cho toàn bộ tuple và làm cho
tuple trở thành một kiểu khác với các tuple khác, và khi đặt tên cho từng field
như trong một struct thông thường sẽ quá dài dòng hoặc dư thừa.

Để định nghĩa một tuple struct, hãy bắt đầu với keyword `struct` và tên struct
theo sau là các kiểu trong tuple. Ví dụ, ở đây chúng ta định nghĩa và sử dụng
hai tuple structs được đặt tên `Color` và `Point`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

Lưu ý rằng các giá trị `black` và `origin` là các kiểu khác nhau vì chúng là
các instance của các tuple structs khác nhau. Mỗi struct bạn định nghĩa là
kiểu của nó, dù các field trong struct có thể có các kiểu giống nhau. Ví dụ,
một function lấy tham số của kiểu `Color` không thể lấy `Point` làm argument,
dù cả hai kiểu được tạo thành từ ba giá trị `i32`. Ngoài ra, các tuple struct
instances tương tự như tuples ở chỗ bạn có thể destructure chúng thành các
phần riêng lẻ của chúng, và bạn có thể sử dụng `.` theo sau là index để truy
cập một giá trị riêng lẻ. Không giống như tuples, tuple structs yêu cầu bạn
đặt tên cho kiểu của struct khi bạn destructure chúng. Ví dụ, chúng ta sẽ viết
`let Point(x, y, z) = origin;` để destructure các giá trị trong điểm `origin`
vào các biến được đặt tên `x`, `y`, và `z`.

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### Định nghĩa Unit-Like Structs

Bạn cũng có thể định nghĩa structs không có bất kỳ field nào! Chúng được gọi
là _unit-like structs_ vì chúng hoạt động tương tự như `()`, kiểu unit mà chúng
ta đã đề cập trong phần [“The Tuple Type”][tuples]<!-- ignore -->. Unit-like
structs có thể hữu ích khi bạn cần implement một trait trên một số kiểu nhưng
không có dữ liệu nào bạn muốn lưu trữ trong chính kiểu đó. Chúng ta sẽ thảo
luận traits trong Chương 10. Dưới đây là một ví dụ về khai báo và khởi tạo một
unit struct được đặt tên `AlwaysEqual`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

Để định nghĩa `AlwaysEqual`, chúng ta sử dụng keyword `struct`, tên chúng ta
muốn, và sau đó là một dấu chấm phẩy. Không cần dấu ngoặc nhọn hoặc dấu ngoặc
đơn! Sau đó, chúng ta có thể lấy một instance của `AlwaysEqual` trong biến
`subject` theo cách tương tự: sử dụng tên chúng ta đã định nghĩa, không có bất
kỳ dấu ngoặc nhọn hoặc dấu ngoặc đơn nào. Hãy tưởng tượng rằng sau đó chúng ta
sẽ implement behavior cho kiểu này sao cho mỗi instance của `AlwaysEqual` luôn
bằng mỗi instance của bất kỳ kiểu nào khác, có lẽ để có kết quả đã biết cho
mục đích test. Chúng ta không cần bất kỳ dữ liệu nào để implement behavior đó!
Bạn sẽ thấy trong Chương 10 cách định nghĩa traits và implement chúng trên bất
kỳ kiểu nào, bao gồm unit-like structs.

> ### Ownership của Struct Data
>
> Trong định nghĩa struct `User` trong Listing 5-1, chúng ta đã sử dụng kiểu
> `String` được owned thay vì kiểu string slice `&str`. Đây là một lựa chọn
> cố ý vì chúng ta muốn mỗi instance của struct này sở hữu tất cả dữ liệu của
> nó và dữ liệu đó hợp lệ miễn là toàn bộ struct hợp lệ.
>
> Cũng có thể structs lưu trữ references tới dữ liệu được sở hữu bởi cái gì
> đó khác, nhưng để làm như vậy cần sử dụng _lifetimes_, một tính năng Rust
> mà chúng ta sẽ thảo luận trong Chương 10. Lifetimes đảm bảo rằng dữ liệu
> được referenced bởi struct hợp lệ miễn là struct hợp lệ. Hãy nói rằng bạn
> cố gắng lưu trữ một reference trong struct mà không chỉ định lifetimes, như
> sau trong *src/main.rs*; điều này sẽ không hoạt động:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> The compiler will complain that it needs lifetime specifiers:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> Trong Chương 10, chúng ta sẽ thảo luận cách sửa các lỗi này để bạn có thể lưu
> trữ references trong structs, nhưng bây giờ, chúng ta sẽ sửa các lỗi như
> vậy bằng cách sử dụng các kiểu owned như `String` thay vì references như
> `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
