## Refutability: Liệu Pattern Có Thể Không Khớp

Pattern có hai dạng: refutable và irrefutable. Các pattern sẽ khớp với bất kỳ giá trị nào có thể được truyền vào là _irrefutable_. Một ví dụ sẽ là `x` trong statement `let x = 5;` vì `x` khớp với bất kỳ thứ gì và do đó không thể không khớp. Các pattern có thể không khớp với một số giá trị có thể là _refutable_. Một ví dụ sẽ là `Some(x)` trong expression `if let Some(x) = a_value` vì nếu giá trị trong biến `a_value` là `None` thay vì `Some`, pattern `Some(x)` sẽ không khớp.

Các tham số function, `let` statements, và `for` loops chỉ có thể chấp nhận các pattern irrefutable vì chương trình không thể làm bất cứ điều có ý nghĩa nào khi các giá trị không khớp. Các expression `if let` và `while let` và statement `let...else` chấp nhận các pattern refutable và irrefutable, nhưng compiler cảnh báo chống lại các pattern irrefutable vì theo định nghĩa, chúng được dự định để xử lý lỗi có thể xảy ra: Chức năng của một điều kiện nằm ở khả năng thực hiện khác nhau tùy thuộc vào thành công hoặc thất bại.

Nói chung, bạn không nên phải lo lắng về sự khác biệt giữa các pattern refutable và irrefutable; tuy nhiên, bạn cần phải quen thuộc với khái niệm refutability để bạn có thể đáp ứng khi bạn nhìn thấy nó trong một thông báo lỗi. Trong những trường hợp đó, bạn sẽ cần thay đổi pattern hoặc cấu trúc mà bạn đang sử dụng pattern với, tùy thuộc vào hành vi dự định của mã.

Hãy xem một ví dụ về những gì xảy ra khi chúng ta cố gắng sử dụng một pattern refutable khi Rust yêu cầu một pattern irrefutable và ngược lại. Listing 19-8 cho thấy một `let` statement, nhưng đối với pattern, chúng ta đã chỉ định `Some(x)`, một pattern refutable. Như bạn có thể mong đợi, mã này sẽ không biên dịch.

<Listing number="19-8" caption="Cố gắng sử dụng một pattern refutable với `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Nếu `some_option_value` là một giá trị `None`, nó sẽ không khớp với pattern `Some(x)`, có nghĩa là pattern là refutable. Tuy nhiên, `let` statement chỉ có thể chấp nhận một pattern irrefutable vì không có gì hợp lệ mà mã có thể làm với một giá trị `None`. Tại thời điểm compile, Rust sẽ phàn nàn rằng chúng ta đã cố gắng sử dụng một pattern refutable khi một pattern irrefutable được yêu cầu:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Vì chúng ta không bao gồm (và không thể bao gồm!) mọi giá trị hợp lệ với pattern `Some(x)`, Rust chính xác tạo ra lỗi compiler.

Nếu chúng ta có một pattern refutable khi một pattern irrefutable được cần, chúng ta có thể sửa nó bằng cách thay đổi mã sử dụng pattern: Thay vì sử dụng `let`, chúng ta có thể sử dụng `let...else`. Sau đó, nếu pattern không khớp, mã trong dấu ngoặc nhọn sẽ xử lý giá trị. Listing 19-9 cho thấy cách sửa mã trong Listing 19-8.

<Listing number="19-9" caption="Sử dụng `let...else` và một block với các pattern refutable thay vì `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Chúng ta đã cho mã một lối thoát! Mã này hoàn toàn hợp lệ, mặc dù điều đó có nghĩa là chúng ta không thể sử dụng một pattern irrefutable mà không nhận được cảnh báo. Nếu chúng ta cung cấp `let...else` một pattern sẽ luôn khớp, chẳng hạn như `x`, như được hiển thị trong Listing 19-10, compiler sẽ cảnh báo.

<Listing number="19-10" caption="Cố gắng sử dụng một pattern irrefutable với `let...else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust phàn nàn rằng nó không có ý nghĩa khi sử dụng `let...else` với một pattern irrefutable:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Vì lý do này, các match arms phải sử dụng các pattern refutable, ngoại trừ arm cuối cùng, sẽ khớp với bất kỳ giá trị còn lại nào với một pattern irrefutable. Rust cho phép chúng ta sử dụng một pattern irrefutable trong một `match` với chỉ một arm, nhưng cú pháp này không đặc biệt hữu ích và có thể được thay thế bằng một `let` statement đơn giản hơn.

Bây giờ bạn biết nơi để sử dụng pattern và sự khác biệt giữa các pattern refutable và irrefutable, hãy bao gồm tất cả cú pháp chúng ta có thể sử dụng để tạo các pattern.
