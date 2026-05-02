## References và Borrowing

Vấn đề với code tuple trong Listing 4-5 là chúng ta phải trả `String` lại cho function gọi để vẫn có thể sử dụng `String` sau khi gọi `calculate_length`, vì `String` đã được moved vào `calculate_length`. Thay vào đó, chúng ta có thể cung cấp một reference đến giá trị `String`. Một reference giống như một pointer ở chỗ nó là một địa chỉ chúng ta có thể theo để truy cập dữ liệu được lưu trữ tại địa chỉ đó; dữ liệu đó được sở hữu bởi một biến khác. Không giống như pointer, một reference được đảm bảo trỏ đến một giá trị hợp lệ của một kiểu cụ thể trong suốt vòng đời của reference đó.

Đây là cách bạn sẽ định nghĩa và sử dụng một function `calculate_length` có reference đến một object làm tham số thay vì lấy ownership của giá trị:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

Đầu tiên, lưu ý rằng tất cả code tuple trong khai báo biến và giá trị return của function đã biến mất. Thứ hai, lưu ý rằng chúng ta truyền `&s1` vào `calculate_length` và trong định nghĩa của nó, chúng ta nhận `&String` thay vì `String`. Các dấu ampersand này đại diện cho references, và chúng cho phép bạn tham chiếu đến một giá trị nào đó mà không lấy ownership của nó. Hình 4-6 mô tả khái niệm này.

<img alt="Ba bảng: bảng cho s chỉ chứa một pointer đến bảng
cho s1. Bảng cho s1 chứa dữ liệu stack cho s1 và trỏ đến
dữ liệu string trên heap." src="img/trpl04-06.svg" class="center" />

<span class="caption">Hình 4-6: Sơ đồ của `&String` `s` trỏ vào
`String` `s1`</span>

> Lưu ý: Ngược lại với việc referencing bằng cách sử dụng `&` là _dereferencing_,
> được thực hiện bằng toán tử dereference, `*`. Chúng ta sẽ thấy một số cách
> sử dụng toán tử dereference trong Chương 8 và thảo luận chi tiết về
> dereferencing trong Chương 15.

Hãy xem xét kỹ hơn lệnh gọi function ở đây:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

Cú pháp `&s1` cho phép chúng ta tạo một reference _tham chiếu đến_ giá trị của `s1` nhưng không sở hữu nó. Vì reference không sở hữu nó, giá trị mà nó trỏ đến sẽ không bị dropped khi reference ngừng được sử dụng.

Tương tự, signature của function sử dụng `&` để chỉ ra rằng kiểu của tham số `s` là một reference. Hãy thêm một số annotation giải thích:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

Scope mà biến `s` có hiệu lực giống như bất kỳ scope tham số function nào, nhưng giá trị được trỏ đến bởi reference không bị dropped khi `s` ngừng được sử dụng, vì `s` không có ownership. Khi các functions có references làm tham số thay vì các giá trị thực sự, chúng ta sẽ không cần trả lại các giá trị để trao trả ownership, vì chúng ta chưa bao giờ có ownership.

Chúng ta gọi hành động tạo một reference là _borrowing_. Như trong cuộc sống thực, nếu một người sở hữu gì đó, bạn có thể mượn nó từ họ. Khi bạn xong, bạn phải trả lại. Bạn không sở hữu nó.

Vậy, điều gì xảy ra nếu chúng ta cố gắng sửa đổi thứ gì đó chúng ta đang borrowing? Hãy thử code trong Listing 4-6. Cảnh báo trước: Nó sẽ không hoạt động!

<Listing number="4-6" file-name="src/main.rs" caption="Cố gắng sửa đổi một giá trị đang được borrowed">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Cũng như các biến mặc định là immutable, references cũng vậy. Chúng ta không được phép sửa đổi thứ gì đó chúng ta có một reference đến.

### Mutable References

Chúng ta có thể sửa code từ Listing 4-6 để cho phép chúng ta sửa đổi một giá trị đang được borrowed chỉ với một vài chỉnh sửa nhỏ sử dụng thay vào đó một _mutable reference_:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

Đầu tiên, chúng ta thay đổi `s` thành `mut`. Sau đó chúng ta tạo một mutable reference với `&mut s` khi chúng ta gọi function `change` và cập nhật signature của function để chấp nhận một mutable reference với `some_string: &mut String`. Điều này làm rõ rằng function `change` sẽ mutate giá trị mà nó borrowing.

Mutable references có một hạn chế lớn: Nếu bạn có một mutable reference đến một giá trị, bạn không thể có bất kỳ reference nào khác đến giá trị đó. Code này cố gắng tạo hai mutable references đến `s` sẽ thất bại:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Lỗi này nói rằng code này không hợp lệ vì chúng ta không thể borrow `s` là mutable nhiều hơn một lần tại một thời điểm. Mutable borrow đầu tiên nằm trong `r1` và phải tồn tại cho đến khi nó được sử dụng trong `println!`, nhưng giữa việc tạo mutable reference đó và việc sử dụng nó, chúng ta đã cố gắng tạo một mutable reference khác trong `r2` borrow cùng dữ liệu với `r1`.

Hạn chế ngăn chặn nhiều mutable references đến cùng một dữ liệu vào cùng một lúc cho phép mutation nhưng theo một cách rất có kiểm soát. Đây là điều mà những Rustaceans mới gặp khó khăn vì hầu hết các ngôn ngữ cho phép bạn mutate bất cứ lúc nào bạn muốn. Lợi ích của việc có hạn chế này là Rust có thể ngăn chặn data race tại compile time. Một _data race_ tương tự như một race condition và xảy ra khi ba hành vi này xảy ra:

- Hai hoặc nhiều pointer truy cập cùng một dữ liệu cùng một lúc.
- Ít nhất một trong các pointer đang được sử dụng để ghi vào dữ liệu.
- Không có cơ chế nào được sử dụng để đồng bộ hóa quyền truy cập vào dữ liệu.

Data races gây ra hành vi không xác định và có thể khó chẩn đoán và sửa chữa khi bạn đang cố gắng theo dõi chúng tại runtime; Rust ngăn chặn vấn đề này bằng cách từ chối compile code có data races!

Như thường lệ, chúng ta có thể sử dụng dấu ngoặc nhọn để tạo một scope mới, cho phép nhiều mutable references, chỉ là không _đồng thời_:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust thực thi một quy tắc tương tự cho việc kết hợp mutable và immutable references. Code này dẫn đến lỗi:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Ôi! Chúng ta _cũng_ không thể có một mutable reference trong khi chúng ta có một immutable reference đến cùng một giá trị.

Người dùng của immutable reference không mong đợi giá trị bỗng dưng thay đổi trước mắt họ! Tuy nhiên, nhiều immutable references được phép vì không ai chỉ đọc dữ liệu có khả năng ảnh hưởng đến việc đọc dữ liệu của người khác.

Lưu ý rằng scope của một reference bắt đầu từ nơi nó được giới thiệu và tiếp tục đến lần cuối cùng reference đó được sử dụng. Ví dụ, code này sẽ compile vì lần sử dụng cuối cùng của immutable references là trong `println!`, trước khi mutable reference được giới thiệu:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Các scope của immutable references `r1` và `r2` kết thúc sau `println!` nơi chúng được sử dụng lần cuối, trước khi mutable reference `r3` được tạo. Các scope này không chồng lên nhau, vì vậy code này được phép: Compiler có thể biết rằng reference không còn được sử dụng tại một điểm trước khi scope kết thúc.

Mặc dù các lỗi borrowing đôi khi có thể frustrating, hãy nhớ rằng đó là Rust compiler chỉ ra một bug tiềm năng sớm (tại compile time thay vì runtime) và chỉ cho bạn chính xác vấn đề ở đâu. Sau đó, bạn không phải theo dõi tại sao dữ liệu của bạn không như bạn nghĩ.

### Dangling References

Trong các ngôn ngữ có pointers, dễ dàng vô tình tạo ra một _dangling pointer_ — một pointer tham chiếu đến một vị trí trong bộ nhớ có thể đã được trao cho ai đó khác — bằng cách giải phóng một số bộ nhớ trong khi giữ lại một pointer đến bộ nhớ đó. Trong Rust, ngược lại, compiler đảm bảo rằng references sẽ không bao giờ là dangling references: Nếu bạn có một reference đến một số dữ liệu, compiler sẽ đảm bảo rằng dữ liệu sẽ không ra khỏi scope trước khi reference đến dữ liệu đó.

Hãy cố gắng tạo một dangling reference để xem Rust ngăn chặn chúng với lỗi compile-time như thế nào:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Thông báo lỗi này đề cập đến một tính năng chúng ta chưa đề cập: lifetimes. Chúng ta sẽ thảo luận về lifetimes chi tiết trong Chương 10. Nhưng nếu bạn bỏ qua các phần về lifetimes, thông báo có chứa chìa khóa cho lý do tại sao code này là vấn đề:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

Hãy xem xét chính xác điều gì đang xảy ra ở mỗi giai đoạn của code `dangle`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

Vì `s` được tạo bên trong `dangle`, khi code của `dangle` kết thúc, `s` sẽ được deallocated. Nhưng chúng ta đã cố gắng trả về một reference đến nó. Điều đó có nghĩa là reference này sẽ trỏ đến một `String` không hợp lệ. Không ổn! Rust sẽ không cho phép chúng ta làm điều này.

Giải pháp ở đây là trả về `String` trực tiếp:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Điều này hoạt động mà không có bất kỳ vấn đề nào. Ownership được moved ra, và không có gì bị deallocated.

### Các quy tắc của References

Hãy tóm tắt những gì chúng ta đã thảo luận về references:

- Tại bất kỳ thời điểm nào, bạn có thể có _hoặc_ một mutable reference _hoặc_ bất kỳ số lượng immutable references nào.
- References phải luôn hợp lệ.

Tiếp theo, chúng ta sẽ xem xét một loại reference khác: slices.
