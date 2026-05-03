## Xác thực tham chiếu bằng lifetime

Lifetime là một dạng generic khác mà chúng ta đã sử dụng. Thay vì đảm bảo một kiểu dữ liệu có hành vi mà chúng ta mong muốn, lifetime đảm bảo rằng các tham chiếu (reference) vẫn hợp lệ trong suốt khoảng thời gian chúng ta cần dùng đến chúng.

Có một chi tiết mà chúng ta chưa bàn đến trong phần [“Tham chiếu và Borrowing”][references-and-borrowing]<!-- ignore --> ở Chương 4, đó là mọi tham chiếu trong Rust đều có một lifetime, tức là phạm vi (scope) mà trong đó tham chiếu đó còn hợp lệ. Phần lớn thời gian, lifetime là ẩn và được suy luận, tương tự như hầu hết kiểu dữ liệu cũng được suy luận. Chúng ta chỉ bắt buộc phải ghi chú (annotate) kiểu khi có nhiều kiểu khả dĩ. Tương tự, chúng ta phải ghi chú lifetime khi các lifetime của các tham chiếu có thể liên hệ với nhau theo một vài cách khác nhau. Rust yêu cầu chúng ta chú thích các mối quan hệ đó bằng các tham số lifetime generic (generic lifetime parameter) để đảm bảo rằng các tham chiếu thực tế được sử dụng lúc runtime chắc chắn sẽ hợp lệ.

Việc chú thích lifetime thậm chí còn không phải là một khái niệm mà đa số các ngôn ngữ lập trình khác có, nên điều này sẽ gây cảm giác xa lạ. Mặc dù trong chương này chúng ta sẽ không bao quát toàn bộ lifetime, chúng ta sẽ thảo luận các cách thường gặp mà bạn có thể bắt gặp cú pháp lifetime để bạn dần quen với khái niệm này.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### Tham chiếu treo (Dangling References)

Mục tiêu chính của lifetime là ngăn chặn các tham chiếu treo (dangling reference), vốn — nếu được phép tồn tại — sẽ khiến chương trình tham chiếu đến dữ liệu khác với dữ liệu mà nó dự định tham chiếu. Hãy xem chương trình trong Liệt kê 10-16, chương trình này có một phạm vi ngoài (outer scope) và một phạm vi trong (inner scope).

<Listing number="10-16" caption="Một nỗ lực sử dụng tham chiếu đến một giá trị đã ra khỏi phạm vi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> Lưu ý: Các ví dụ trong Liệt kê 10-16, 10-17, và 10-23 khai báo biến mà không gán giá trị khởi tạo, do đó tên biến tồn tại trong phạm vi ngoài. Thoạt nhìn, điều này có vẻ mâu thuẫn với việc Rust không có giá trị null. Tuy nhiên, nếu chúng ta cố gắng sử dụng một biến trước khi gán giá trị cho nó, chúng ta sẽ nhận lỗi tại thời điểm biên dịch, điều này chứng tỏ rằng Rust thực sự không cho phép giá trị null.

Phạm vi ngoài khai báo một biến tên `r` mà không có giá trị khởi tạo, và phạm vi trong khai báo một biến tên `x` với giá trị khởi tạo là `5`. Bên trong phạm vi trong, chúng ta cố gắng gán giá trị của `r` thành tham chiếu đến `x`. Sau đó, phạm vi trong kết thúc, và chúng ta cố gắng in giá trị trong `r`. Đoạn mã này sẽ không biên dịch, vì giá trị mà `r` tham chiếu tới đã ra khỏi phạm vi trước khi chúng ta cố gắng sử dụng nó. Đây là thông báo lỗi:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Thông báo lỗi cho biết rằng biến `x` “does not live long enough” (không sống đủ lâu). Nguyên nhân là `x` sẽ ra khỏi phạm vi khi phạm vi trong kết thúc ở dòng 7. Nhưng `r` vẫn còn hợp lệ trong phạm vi ngoài; vì phạm vi của nó lớn hơn, chúng ta nói rằng nó “sống lâu hơn”. Nếu Rust cho phép đoạn mã này chạy, `r` sẽ tham chiếu đến vùng nhớ đã bị giải phóng khi `x` ra khỏi phạm vi, và bất cứ điều gì chúng ta cố gắng làm với `r` đều sẽ không hoạt động đúng. Vậy Rust xác định đoạn mã này không hợp lệ như thế nào? Nó dùng một bộ kiểm tra borrow (borrow checker).

### Borrow Checker

Trình biên dịch Rust có một _borrow checker_ (bộ kiểm tra borrow) dùng để so sánh các phạm vi nhằm xác định liệu tất cả các borrow có hợp lệ hay không. Liệt kê 10-17 cho thấy cùng đoạn mã như Liệt kê 10-16 nhưng có thêm chú thích về lifetime của các biến.

<Listing number="10-17" caption="Chú thích lifetime của `r` và `x`, lần lượt là `'a` và `'b`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Tại đây, chúng ta đã chú thích lifetime của `r` là `'a` và lifetime của `x` là `'b`. Như bạn có thể thấy, khối `'b` bên trong nhỏ hơn nhiều so với khối lifetime `'a` bên ngoài. Tại thời điểm biên dịch, Rust so sánh kích thước của hai lifetime này và thấy rằng `r` có lifetime `'a` nhưng nó lại tham chiếu đến vùng nhớ có lifetime `'b`. Chương trình bị từ chối vì `'b` ngắn hơn `'a`: đối tượng được tham chiếu không sống lâu bằng tham chiếu.

Liệt kê 10-18 sửa đoạn mã để nó không còn tham chiếu treo và có thể biên dịch mà không gặp lỗi.

<Listing number="10-18" caption="Một tham chiếu hợp lệ vì dữ liệu có lifetime dài hơn tham chiếu">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Ở đây, `x` có lifetime `'b`, trong trường hợp này lại lớn hơn `'a`. Điều này có nghĩa là `r` có thể tham chiếu đến `x` vì Rust biết rằng tham chiếu trong `r` sẽ luôn hợp lệ trong khi `x` còn hợp lệ.

Bây giờ, khi bạn đã biết lifetime của tham chiếu nằm ở đâu và Rust phân tích lifetime như thế nào để đảm bảo rằng các tham chiếu sẽ
