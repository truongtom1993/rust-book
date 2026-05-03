<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## Sử Dụng Trait Objects để Trừu Tượng Hóa Hành Động Được Chia Sẻ

Trong Chương 8, chúng ta đã đề cập rằng một hạn chế của vector là chúng chỉ có thể lưu trữ các phần tử của một type duy nhất. Chúng ta đã tạo một giải pháp thay thế trong Listing 8-9 nơi chúng ta định nghĩa một enum `SpreadsheetCell` có các biến thể để giữ các số nguyên, số dấu phẩy động, và văn bản. Điều này có nghĩa là chúng ta có thể lưu trữ các type dữ liệu khác nhau trong mỗi ô và vẫn có một vector đại diện cho một hàng các ô. Đây là một giải pháp hoàn toàn tốt khi các mục có thể hoán đổi của chúng ta là một tập hợp cố định các type mà chúng ta biết khi code của chúng ta được biên dịch.

Tuy nhiên, đôi khi chúng ta muốn người dùng thư viện của chúng ta có thể mở rộng tập hợp các type hợp lệ trong một tình huống cụ thể. Để chỉ ra cách chúng ta có thể đạt được điều này, chúng ta sẽ tạo một ví dụ về công cụ giao diện người dùng đồ họa (GUI) lặp qua một danh sách các mục, gọi phương thức `draw` trên mỗi mục để vẽ nó lên màn hình—một kỹ thuật phổ biến cho các công cụ GUI. Chúng ta sẽ tạo một thư viện crate được gọi là `gui` chứa cấu trúc của một thư viện GUI. Crate này có thể bao gồm một số type cho mọi người sử dụng, chẳng hạn như `Button` hoặc `TextField`. Ngoài ra, những người dùng `gui` sẽ muốn tạo các type của riêng họ có thể được vẽ: Ví dụ, một người lập trình có thể thêm một `Image`, và một người khác có thể thêm một `SelectBox`.

Vào lúc viết thư viện, chúng ta không thể biết và định nghĩa tất cả các type mà những lập trình viên khác có thể muốn tạo. Nhưng chúng ta biết rằng `gui` cần phải theo dõi nhiều giá trị của các type khác nhau, và nó cần gọi phương thức `draw` trên mỗi giá trị được nhập theo một loại khác nhau. Nó không cần biết chính xác điều gì sẽ xảy ra khi chúng ta gọi phương thức `draw`, chỉ là giá trị sẽ có phương thức đó có sẵn để chúng ta gọi.

Để làm điều này trong một ngôn ngữ có kế thừa, chúng ta có thể định nghĩa một lớp có tên `Component` có một phương thức có tên `draw` trên nó. Các lớp khác, chẳng hạn như `Button`, `Image`, và `SelectBox`, sẽ kế thừa từ `Component` và do đó kế thừa phương thức `draw`. Chúng có thể mỗi cái ghi đè phương thức `draw` để định nghĩa hành vi tùy chỉnh của chúng, nhưng framework có thể coi tất cả các type như thể chúng là các instance `Component` và gọi `draw` trên chúng. Nhưng vì Rust không có kế thừa, chúng ta cần một cách khác để cấu trúc thư viện `gui` để cho phép người dùng tạo các type mới tương thích với thư viện.

### Định Nghĩa một Trait cho Hành Động Chung

Để triển khai hành vi mà chúng ta muốn `gui` có, chúng ta sẽ định nghĩa một trait có tên `Draw` sẽ có một phương thức có tên `draw`. Sau đó, chúng ta có thể định nghĩa một vector lấy một trait object. Một _trait object_ trỏ đến cả một instance của một type triển khai trait được chỉ định của chúng ta và một bảng được sử dụng để tra cứu các phương thức trait trên type đó tại runtime. Chúng ta tạo một trait object bằng cách chỉ định một loại con trỏ nào đó, chẳng hạn như một reference hoặc smart pointer `Box<T>`, sau đó là từ khóa `dyn`, và sau đó chỉ định trait có liên quan. (Chúng ta sẽ nói về lý do tại sao trait objects phải sử dụng một con trỏ trong ["Dynamically Sized Types and the `Sized` Trait"][dynamically-sized]<!-- ignore --> trong Chương 20.) Chúng ta có thể sử dụng trait objects thay vì một generic hoặc concrete type. Bất cứ nơi nào chúng ta sử dụng một trait object, hệ thống type của Rust sẽ đảm bảo tại compile time rằng bất kỳ giá trị nào được sử dụng trong bối cảnh đó sẽ triển khai trait của trait object. Theo đó, chúng ta không cần biết tất cả các type có thể tại compile time.

Chúng ta đã đề cập rằng, trong Rust, chúng ta tránh gọi struct và enum là "đối tượng" để phân biệt chúng với các đối tượng của những ngôn ngữ khác. Trong một struct hoặc enum, dữ liệu trong các field struct và hành động trong các khối `impl` được tách biệt, trong khi ở các ngôn ngữ khác, dữ liệu và hành động kết hợp lại thành một khái niệm thường được gọi là một đối tượng. Trait objects khác với các đối tượng trong các ngôn ngữ khác ở chỗ chúng ta không thể thêm dữ liệu vào một trait object. Trait objects không hữu ích như các đối tượng trong các ngôn ngữ khác: Mục đích cụ thể của chúng là cho phép trừu tượng hóa hành động chung.

Listing 18-3 hiển thị cách định nghĩa một trait có tên `Draw` với một phương thức có tên `draw`.

<Listing number="18-3" file-name="src/lib.rs" caption="Định nghĩa của trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Cú pháp này sẽ quen thuộc từ những cuộc thảo luận của chúng ta về cách định nghĩa các trait trong Chương 10. Tiếp theo là một số cú pháp mới: Listing 18-4 định nghĩa một struct có tên `Screen` giữ một vector có tên `components`. Vector này có type `Box<dyn Draw>`, đây là một trait object; nó là một đại diện cho bất kỳ type nào trong một `Box` triển khai trait `Draw`.

<Listing number="18-4" file-name="src/lib.rs" caption="Định nghĩa của struct `Screen` với field `components` giữ một vector của trait objects triển khai trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

Trên struct `Screen`, chúng ta sẽ định nghĩa một phương thức có tên `run` sẽ gọi phương thức `draw` trên mỗi `components` của nó, như được hiển thị trong Listing 18-5.

<Listing number="18-5" file-name="src/lib.rs" caption="Phương thức `run` trên `Screen` gọi phương thức `draw` trên mỗi component">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Điều này hoạt động khác biệt so với việc định nghĩa một struct sử dụng một tham số generic type với trait bounds. Một tham số generic type có thể được thay thế bằng một type concrete duy nhất tại một thời điểm, trong khi trait objects cho phép nhiều concrete types điền vào cho trait object tại runtime. Ví dụ, chúng ta có thể đã định nghĩa struct `Screen` bằng cách sử dụng một generic type và một trait bound, như trong Listing 18-6.

<Listing number="18-6" file-name="src/lib.rs" caption="Một cách triển khai thay thế của struct `Screen` và phương thức `run` của nó sử dụng generics và trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Điều này giới hạn chúng ta ở một instance `Screen` có một danh sách các components đều thuộc type `Button` hoặc đều thuộc type `TextField`. Nếu bạn sẽ chỉ bao giờ có các bộ sưu tập đồng nhất, sử dụng generics và trait bounds là tốt hơn vì các định nghĩa sẽ được monomorphized tại compile time để sử dụng các concrete types.

Mặt khác, với phương thức sử dụng trait objects, một instance `Screen` có thể giữ một `Vec<T>` chứa cả `Box<Button>` lẫn `Box<TextField>`. Hãy xem cách nó hoạt động, và sau đó chúng ta sẽ nói về các hàm ý hiệu suất runtime.

### Triển Khai Trait

Bây giờ chúng ta sẽ thêm một số type triển khai trait `Draw`. Chúng ta sẽ cung cấp type `Button`. Một lần nữa, việc thực sự triển khai một thư viện GUI vượt quá phạm vi của cuốn sách này, vì vậy phương thức `draw` sẽ không có bất kỳ cách triển khai hữu ích nào trong thân của nó. Để tưởng tượng cách triển khai có thể trông như thế nào, một struct `Button` có thể có các field cho `width`, `height`, và `label`, như được hiển thị trong Listing 18-7.

<Listing number="18-7" file-name="src/lib.rs" caption="Một struct `Button` triển khai trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

Các field `width`, `height`, và `label` trên `Button` sẽ khác biệt với các field trên các components khác; ví dụ, một type `TextField` có thể có những field giống nhau cộng với một field `placeholder`. Mỗi type mà chúng ta muốn vẽ trên màn hình sẽ triển khai trait `Draw` nhưng sẽ sử dụng code khác nhau trong phương thức `draw` để định nghĩa cách vẽ type cụ thể đó, như `Button` có ở đây (không có code GUI thực tế, như đã đề cập). Type `Button`, ví dụ, có thể có một khối `impl` bổ sung chứa các phương thức liên quan đến những gì xảy ra khi người dùng nhấp vào nút. Những loại phương thức này sẽ không áp dụng cho các type như `TextField`.

Nếu ai đó sử dụng thư viện của chúng ta quyết định triển khai một struct `SelectBox` có các field `width`, `height`, và `options`, họ sẽ triển khai trait `Draw` trên type `SelectBox` cũng như vậy, như được hiển thị trong Listing 18-8.

<Listing number="18-8" file-name="src/main.rs" caption="Một crate khác sử dụng `gui` và triển khai trait `Draw` trên một struct `SelectBox`">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

Người dùng thư viện của chúng ta bây giờ có thể viết hàm `main` của họ để tạo một instance `Screen`. Với instance `Screen`, họ có thể thêm một `SelectBox` và một `Button` bằng cách đặt mỗi cái vào một `Box<T>` để trở thành một trait object. Sau đó họ có thể gọi phương thức `run` trên instance `Screen`, sẽ gọi `draw` trên mỗi components. Listing 18-9 hiển thị cách triển khai này.

<Listing number="18-9" file-name="src/main.rs" caption="Sử dụng trait objects để lưu trữ các giá trị của các type khác nhau triển khai cùng một trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Khi chúng ta viết thư viện, chúng ta không biết rằng ai đó có thể thêm type `SelectBox`, nhưng cách triển khai `Screen` của chúng ta đã có thể hoạt động trên type mới và vẽ nó vì `SelectBox` triển khai trait `Draw`, có nghĩa là nó triển khai phương thức `draw`.

Khái niệm này—chỉ quan tâm đến các tin nhắn mà một giá trị phản hồi chứ không phải loại concrete của giá trị—tương tự như khái niệm _duck typing_ trong các ngôn ngữ được nhập động: Nếu nó bước như một con vịt và quác như một con vịt, thì nó phải là một con vịt! Trong cách triển khai `run` trên `Screen` trong Listing 18-5, `run` không cần biết loại concrete của mỗi component là gì. Nó không kiểm tra xem một component có phải là một instance của `Button` hoặc `SelectBox` hay không, nó chỉ gọi phương thức `draw` trên component. Bằng cách chỉ định `Box<dyn Draw>` là type của các giá trị trong vector `components`, chúng ta đã định nghĩa `Screen` cần các giá trị mà chúng ta có thể gọi phương thức `draw` trên.

Ưu điểm của việc sử dụng trait objects và hệ thống type của Rust để viết code tương tự như code sử dụng duck typing là chúng ta không bao giờ phải kiểm tra xem một giá trị có triển khai một phương thức cụ thể tại runtime hay lo lắng về việc nhận được lỗi nếu một giá trị không triển khai một phương thức nhưng chúng ta gọi nó. Rust sẽ không biên dịch code của chúng ta nếu các giá trị không triển khai các trait mà trait objects cần.

Ví dụ, Listing 18-10 hiển thị những gì xảy ra nếu chúng ta cố gắng tạo một `Screen` với một `String` làm component.

<Listing number="18-10" file-name="src/main.rs" caption="Cố gắng sử dụng một type không triển khai trait của trait object">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

Chúng ta sẽ nhận được lỗi này vì `String` không triển khai trait `Draw`:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Lỗi này cho chúng ta biết rằng hoặc chúng ta đang chuyển thứ gì đó cho `Screen` mà chúng ta không có ý định chuyển và vì vậy nên chuyển một type khác, hoặc chúng ta nên triển khai `Draw` trên `String` để `Screen` có thể gọi `draw` trên nó.

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### Thực Hiện Dynamic Dispatch

Hãy nhớ lại trong ["Performance of Code Using Generics"][performance-of-code-using-generics]<!-- ignore --> trong Chương 10 cuộc thảo luận của chúng ta về quá trình monomorphization được thực hiện trên generics bởi compiler: Compiler tạo ra những cách triển khai không generic của các function và phương thức cho mỗi concrete type mà chúng ta sử dụng thay vì một tham số generic type. Code kết quả từ monomorphization đang thực hiện _static dispatch_, đó là khi compiler biết phương thức bạn đang gọi tại compile time. Điều này trái ngược với _dynamic dispatch_, đó là khi compiler không thể biết tại compile time phương thức bạn đang gọi. Trong các trường hợp dynamic dispatch, compiler phát ra code mà tại runtime sẽ biết phương thức nào để gọi.

Khi chúng ta sử dụng trait objects, Rust phải sử dụng dynamic dispatch. Compiler không biết tất cả các type có thể được sử dụng với code đang sử dụng trait objects, vì vậy nó không biết phương thức nào được triển khai trên type nào để gọi. Thay vào đó, tại runtime, Rust sử dụng các con trỏ bên trong trait object để biết phương thức nào để gọi. Lần tra cứu này gây ra một chi phí runtime không xảy ra với static dispatch. Dynamic dispatch cũng ngăn compiler chọn để inline code của một phương thức, điều này lần lượt ngăn chặn một số tối ưu hóa, và Rust có một số quy tắc về nơi bạn có thể và không thể sử dụng dynamic dispatch, được gọi là _dyn compatibility_. Những quy tắc đó vượt quá phạm vi của cuộc thảo luận này, nhưng bạn có thể đọc thêm về chúng [in the reference][dyn-compatibility]<!-- ignore -->. Tuy nhiên, chúng ta đã nhận được sự linh hoạt bổ sung trong code mà chúng ta viết trong Listing 18-5 và đã có thể hỗ trợ trong Listing 18-9, vì vậy nó là một trade-off để xem xét.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
