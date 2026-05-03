<!-- Old headings. Do not remove or links may break. -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>
<a id="treating-smart-pointers-like-regular-references-with-deref"></a>

## Xử Lý Smart Pointers Như Regular References

Implementing trait `Deref` cho phép bạn tùy chỉnh hành vi của _dereference operator_
`*` (không nên nhầm lẫn với toán tử multiplication hoặc glob). Bằng cách implementing
`Deref` sao cho một smart pointer có thể được xử lý như một regular reference, bạn
có thể viết code hoạt động trên references và sử dụng code đó với smart pointers
cũng vậy.

Trước tiên, hãy xem cách dereference operator hoạt động với regular references.
Sau đó, chúng ta sẽ cố gắng define một custom type hoạt động giống như `Box<T>`
và xem tại sao dereference operator không hoạt động như một reference trên newly
defined type của chúng ta. Chúng ta sẽ khám phá cách implementing trait `Deref`
làm cho có thể cho smart pointers hoạt động theo cách tương tự như references.
Sau đó, chúng ta sẽ xem xét Rust's deref coercion feature và cách nó cho phép chúng
ta làm việc với cả references hoặc smart pointers.

<!-- Old headings. Do not remove or links may break. -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### Theo Dõi Reference Đến Giá Trị

Một regular reference là một kiểu pointer, và một cách để suy nghĩ về một pointer
là nó như một mũi tên chỉ tới một giá trị được lưu trữ ở nơi khác. Trong Listing
15-6, chúng ta tạo một reference tới giá trị `i32` và sau đó sử dụng dereference
operator để theo dõi reference đến giá trị.

<Listing number="15-6" file-name="src/main.rs" caption="Sử dụng dereference operator để theo dõi một reference tới giá trị `i32`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

Biến `x` giữ một giá trị `i32` là `5`. Chúng ta đặt `y` bằng một reference tới `x`.
Chúng ta có thể assert rằng `x` bằng `5`. Tuy nhiên, nếu chúng ta muốn tạo một
assertion về giá trị trong `y`, chúng ta phải sử dụng `*y` để theo dõi reference tới
giá trị nó đang chỉ tới (do đó, _dereference_) để compiler có thể compare giá trị
thực tế. Khi chúng ta dereference `y`, chúng ta có access tới giá trị integer mà `y`
đang chỉ tới để chúng ta có thể compare với `5`.

Nếu chúng ta cố gắng viết `assert_eq!(5, y);` thay vào đó, chúng ta sẽ nhận được
lỗi compilation này:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Comparing một số và một reference tới một số là không được phép vì chúng là các kiểu
khác nhau. Chúng ta phải sử dụng dereference operator để theo dõi reference tới giá
trị nó đang chỉ tới.

### Sử Dụng `Box<T>` Như Một Reference

Chúng ta có thể viết lại code trong Listing 15-6 để sử dụng một `Box<T>` thay vì
một reference; dereference operator được sử dụng trên `Box<T>` trong Listing 15-7
hoạt động theo cách tương tự như dereference operator được sử dụng trên reference
trong Listing 15-6.

<Listing number="15-7" file-name="src/main.rs" caption="Sử dụng dereference operator trên một `Box&lt;i32&gt;`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

Sự khác biệt chính giữa Listing 15-7 và Listing 15-6 là ở đây chúng ta đặt `y` là
một instance của một box chỉ tới một copied value của `x` thay vì một reference chỉ
tới giá trị của `x`. Trong assertion cuối cùng, chúng ta có thể sử dụng dereference
operator để theo dõi box's pointer theo cách tương tự như chúng ta đã làm khi `y`
là một reference. Tiếp theo, chúng ta sẽ khám phá điều gì đặc biệt về `Box<T>`
cho phép chúng ta sử dụng dereference operator bằng cách defining riêng box type của
chúng ta.

### Định Nghĩa Smart Pointer Của Chúng Ta

Hãy build một wrapper type tương tự như type `Box<T>` được cung cấp bởi thư viện
tiêu chuẩn để trải nghiệm cách smart pointer types hoạt động khác với references
theo mặc định. Sau đó, chúng ta sẽ xem cách thêm khả năng sử dụng dereference
operator.

> Lưu ý: Có một sự khác biệt lớn giữa type `MyBox<T>` chúng ta sắp build và real
> `Box<T>`: Phiên bản của chúng ta sẽ không lưu trữ dữ liệu của nó trên heap. Chúng
> ta đang tập trung ví dụ này vào `Deref`, vì vậy nơi dữ liệu thực sự được lưu trữ
> ít quan trọng hơn hành vi giống như pointer.

Type `Box<T>` cuối cùng được định nghĩa là một tuple struct với một phần tử, vì vậy
Listing 15-8 defines một type `MyBox<T>` theo cách tương tự. Chúng ta cũng sẽ định
nghĩa hàm `new` để match hàm `new` được định nghĩa trên `Box<T>`.

<Listing number="15-8" file-name="src/main.rs" caption="Định nghĩa một type `MyBox&lt;T&gt;`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

Chúng ta define một struct có tên `MyBox` và declare một generic parameter `T` vì
chúng ta muốn type của chúng ta giữ các giá trị của bất kỳ kiểu nào. Type `MyBox`
là một tuple struct với một phần tử của type `T`. Hàm `MyBox::new` lấy một tham số
của type `T` và trả về một instance `MyBox` giữ giá trị truyền vào.

Hãy cố gắng thêm hàm `main` trong Listing 15-7 vào Listing 15-8 và thay đổi nó để
sử dụng type `MyBox<T>` mà chúng ta đã defined thay vì `Box<T>`. Code trong Listing
15-9 sẽ không compile, vì Rust không biết cách dereference `MyBox`.

<Listing number="15-9" file-name="src/main.rs" caption="Cố gắng sử dụng `MyBox&lt;T&gt;` theo cách tương tự như chúng ta sử dụng references và `Box&lt;T&gt;`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

Đây là lỗi compilation kết quả:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Type `MyBox<T>` của chúng ta không thể được dereference vì chúng ta chưa implement
khả năng đó trên type của chúng ta. Để enable dereferencing với toán tử `*`, chúng
ta implement trait `Deref`.

<!-- Old headings. Do not remove or links may break. -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### Implementing Trait `Deref`

Như đã thảo luận trong ["Implementing a Trait on a Type"][impl-trait]<!-- ignore -->
trong Chương 10, để implement một trait chúng ta cần provide implementations cho các
required methods của trait. Trait `Deref`, được cung cấp bởi thư viện tiêu chuẩn,
yêu cầu chúng ta implement một method có tên `deref` mà borrow `self` và trả về một
reference đến dữ liệu bên trong. Listing 15-10 chứa một implementation của `Deref`
để thêm vào định nghĩa của `MyBox<T>`.

<Listing number="15-10" file-name="src/main.rs" caption="Implementing `Deref` trên `MyBox&lt;T&gt;`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

Syntax `type Target = T;` defines một associated type cho trait `Deref` sử dụng.
Associated types là một cách hơi khác nhau để declare một generic parameter, nhưng
bạn không cần lo lắng về chúng hiện tại; chúng ta sẽ cover chúng chi tiết hơn trong
Chương 20.

Chúng ta fill body của method `deref` với `&self.0` để `deref` trả về một reference
đến giá trị chúng ta muốn access với toán tử `*`; recall từ ["Creating Different
Types with Tuple Structs"][tuple-structs]<!-- ignore --> trong Chương 5 rằng `.0`
truy cập giá trị đầu tiên trong một tuple struct. Hàm `main` trong Listing 15-9 gọi
`*` trên giá trị `MyBox<T>` bây giờ compiles, và các assertions pass!

Mà không có trait `Deref`, compiler chỉ có thể dereference `&` references. Method
`deref` cho compiler khả năng lấy một giá trị của bất kỳ type nào implementing
`Deref` và gọi method `deref` để lấy một reference mà nó biết cách dereference.

Khi chúng ta entered `*y` trong Listing 15-9, behind the scenes Rust thực sự chạy
code này:

```rust,ignore
*(y.deref())
```

Rust substitutes toán tử `*` với một lệnh gọi tới method `deref` và sau đó một plain
dereference để chúng ta không phải suy nghĩ về việc liệu chúng ta có cần gọi method
`deref` hay không. Rust feature này cho phép chúng ta viết code hoạt động giống hệt
nhau cho dù chúng ta có một regular reference hoặc một type implementing `Deref`.

Lý do mà method `deref` trả về một reference đến một giá trị, và mà plain dereference
bên ngoài parentheses trong `*(y.deref())` vẫn còn cần thiết, có liên quan đến
ownership system. Nếu method `deref` trả về giá trị trực tiếp thay vì một reference
đến giá trị, giá trị sẽ được moved out của `self`. Chúng ta không muốn take ownership
của inner value bên trong `MyBox<T>` trong trường hợp này hoặc trong hầu hết các trường
hợp mà chúng ta sử dụng dereference operator.

Lưu ý rằng toán tử `*` được thay thế bằng một lệnh gọi tới method `deref` và sau đó
một lệnh gọi tới toán tử `*` chỉ một lần, mỗi khi chúng ta sử dụng một `*` trong code
của chúng ta. Vì substitution của toán tử `*` không recurse vô hạn, chúng ta kết thúc
với dữ liệu của type `i32`, phù hợp với `5` trong `assert_eq!` trong Listing 15-9.

<!-- Old headings. Do not remove or links may break. -->

<a id="implicit-deref-coercions-with-functions-and-methods"></a>
<a id="using-deref-coercions-in-functions-and-methods"></a>

### Sử Dụng Deref Coercion Trong Functions và Methods

_Deref coercion_ converts một reference tới một type implementing trait `Deref` thành
một reference tới một type khác. Ví dụ, deref coercion có thể convert `&String` thành
`&str` vì `String` implements trait `Deref` sao cho nó trả về `&str`. Deref coercion
là một convenience mà Rust performs trên arguments cho functions và methods, và nó hoạt
động chỉ trên types implementing trait `Deref`. Nó xảy ra tự động khi chúng ta truyền
một reference tới giá trị của một type cụ thể làm argument tới một function hoặc method
không match parameter type trong định nghĩa function hoặc method. Một sequence của các
lệnh gọi tới method `deref` converts type chúng ta cung cấp thành type mà parameter cần.

Deref coercion được thêm vào Rust để các lập trình viên viết function và method calls
không cần thêm nhiều explicit references và dereferences với `&` và `*`. Feature deref
coercion cũng cho phép chúng ta viết more code có thể hoạt động cho cả references hoặc
smart pointers.

Để xem deref coercion hoạt động, hãy sử dụng type `MyBox<T>` chúng ta defined trong
Listing 15-8 cũng như implementation của `Deref` mà chúng ta thêm trong Listing 15-10.
Listing 15-11 cho thấy định nghĩa của một function có một string slice parameter.

<Listing number="15-11" file-name="src/main.rs" caption="Một hàm `hello` có parameter `name` của type `&str`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

Chúng ta có thể gọi hàm `hello` với một string slice làm argument, chẳng hạn như
`hello("Rust");`, ví dụ. Deref coercion làm cho có thể gọi `hello` với một reference
tới một giá trị của type `MyBox<String>`, như hiển thị trong Listing 15-12.

<Listing number="15-12" file-name="src/main.rs" caption="Gọi `hello` với một reference tới giá trị `MyBox&lt;String&gt;`, hoạt động vì deref coercion">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

Ở đây chúng ta đang gọi hàm `hello` với argument `&m`, là một reference tới giá
trị `MyBox<String>`. Vì chúng ta implemented trait `Deref` trên `MyBox<T>` trong
Listing 15-10, Rust có thể turn `&MyBox<String>` thành `&String` bằng cách gọi
`deref`. Thư viện tiêu chuẩn cung cấp một implementation của `Deref` trên `String`
mà trả về một string slice, và điều này nằm trong API documentation cho `Deref`.
Rust gọi `deref` lại để turn `&String` thành `&str`, phù hợp với định nghĩa hàm
`hello`.

Nếu Rust không implement deref coercion, chúng ta sẽ phải viết code trong Listing
15-13 thay vì code trong Listing 15-12 để gọi `hello` với một giá trị của type
`&MyBox<String>`.

<Listing number="15-13" file-name="src/main.rs" caption="Code chúng ta sẽ phải viết nếu Rust không có deref coercion">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)` dereferences `MyBox<String>` thành `String`. Sau đó, `&` và `[..]` lấy một
string slice của `String` bằng toàn bộ string để match signature của `hello`. Code
này không có deref coercions khó đọc, viết, và hiểu hơn với tất cả các symbols này.
Deref coercion cho phép Rust xử lý những conversions này cho chúng ta tự động.

Khi trait `Deref` được defined cho các types liên quan, Rust sẽ analyze các types
và sử dụng `Deref::deref` bao nhiêu lần cần thiết để lấy một reference để match
type của parameter. Số lần mà `Deref::deref` cần được inserted được resolved tại
compile time, vì vậy không có runtime penalty cho việc tận dụng deref coercion!

<!-- Old headings. Do not remove or links may break. -->

<a id="how-deref-coercion-interacts-with-mutability"></a>

### Handling Deref Coercion Với Mutable References

Tương tự như cách bạn sử dụng trait `Deref` để override toán tử `*` trên immutable
references, bạn có thể sử dụng trait `DerefMut` để override toán tử `*` trên mutable
references.

Rust làm deref coercion khi nó tìm thấy types và trait implementations trong ba
trường hợp:

1. Từ `&T` tới `&U` khi `T: Deref<Target=U>`
2. Từ `&mut T` tới `&mut U` khi `T: DerefMut<Target=U>`
3. Từ `&mut T` tới `&U` khi `T: Deref<Target=U>`

Hai trường hợp đầu tiên giống nhau ngoại trừ trường hợp thứ hai implements mutability.
Trường hợp đầu tiên nói rằng nếu bạn có một `&T`, và `T` implements `Deref` tới một
type `U`, bạn có thể lấy một `&U` một cách transparent. Trường hợp thứ hai nói rằng
deref coercion tương tự xảy ra cho mutable references.

Trường hợp thứ ba khó khăn hơn: Rust sẽ cũng coerce một mutable reference thành
một immutable. Nhưng reverse là _không_ có thể: Immutable references sẽ không bao
giờ coerce thành mutable references. Vì borrowing rules, nếu bạn có một mutable
reference, mutable reference đó phải là cách duy nhất reference tới dữ liệu đó
(ngược lại, chương trình sẽ không compile). Converting một mutable reference thành
một immutable reference sẽ không bao giờ break borrowing rules. Converting một
immutable reference thành một mutable reference sẽ yêu cầu rằng initial immutable
reference là immutable reference duy nhất tới dữ liệu đó, nhưng borrowing rules
không guarantee điều đó. Do đó, Rust không thể tạo giả định rằng converting một
immutable reference thành một mutable reference là có thể.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
